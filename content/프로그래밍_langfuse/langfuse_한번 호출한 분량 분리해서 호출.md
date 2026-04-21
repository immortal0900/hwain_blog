---
생성날짜:
  - 2026-03-21 17:36
마지막수정날짜:
  - 2026-03-21-토요일 17:36
tags:
  - langfuse
  - metrics-api
  - observations-api
  - 토큰집계
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
Langfuse에서 **특정 시점 이후의 토큰/호출량만 일괄 집계**하는 방법 3가지. Hobby 플랜이면 Cloud API 그대로 사용 가능.

## 방법 1: Metrics API (가장 추천)

Langfuse **Metrics API**(시간 범위별 집계 지표를 한 번에 반환하는 엔드포인트)로 `fromTimestamp`/`toTimestamp` 지정 + `totalCost`/`count` 같은 measure를 aggregation해서 한 방에 받을 수 있다.

Python SDK 기준:

```python
from langfuse import get_client
import json

langfuse = get_client()

# v2 Metrics API (observations view 사용 - traces view는 v2에서 제거됨)
query_v2 = json.dumps({
    "view": "observations",
    "metrics": [
        {"measure": "totalCost", "aggregation": "sum"},
        {"measure": "count", "aggregation": "count"}     # LLM 호출 수
    ],
    "dimensions": [{"field": "providedModelName"}],       # 모델별 분류
    "filters": [],
    "fromTimestamp": "2026-03-21T15:45:07Z",
    "toTimestamp": "2026-03-21T23:59:59Z"
})

result = langfuse.api.metrics.get(query=query_v2)
print(result)
```

사용 가능한 measure 예시: `totalTokens`(sum), `count`(count), `totalCost`(sum), `latency`(avg/p95).

v2 `observations` view에서 `totalTokens`가 막힐 경우 **v1 legacy**로 폴백:

```python
# v1 Legacy (totalTokens 확실히 지원)
query_v1 = json.dumps({
    "view": "traces",
    "metrics": [
        {"measure": "totalTokens", "aggregation": "sum"},
        {"measure": "totalCost", "aggregation": "sum"},
        {"measure": "count", "aggregation": "count"}
    ],
    "dimensions": [{"field": "name"}],
    "filters": [],
    "fromTimestamp": "2026-03-21T15:45:07Z",
    "toTimestamp": "2026-03-21T23:59:59Z"
})

result = langfuse.api.legacy.metrics_v1.metrics(query=query_v1)
print(result)
```

## 방법 2: Observations API로 raw 데이터 긁어와 합산

SDK의 `api.observations.get_many()`는 **개별 observation**(span 단위) 레벨의 usage 정보를 반환. Metrics API가 안 되는 경우 수동 집계용으로 쓴다.

```python
from langfuse import get_client

langfuse = get_client()

total_input = 0
total_output = 0
call_count = 0
cursor = None

while True:
    kwargs = {
        "type": "GENERATION",
        "limit": 100,
        "fields": "core,basic,usage",
        "from_start_time": "2026-03-21T15:45:07Z",
    }
    if cursor:
        kwargs["cursor"] = cursor
    
    obs = langfuse.api.observations.get_many(**kwargs)
    
    for o in obs.data:
        call_count += 1
        if o.usage_details:
            total_input += o.usage_details.get("input", 0)
            total_output += o.usage_details.get("output", 0)
    
    cursor = obs.meta.cursor if hasattr(obs, 'meta') and obs.meta else None
    if not cursor:
        break

print(f"LLM 호출 수: {call_count}")
print(f"Input tokens: {total_input}")
print(f"Output tokens: {total_output}")
print(f"Total tokens: {total_input + total_output}")
```

## 방법 3: UI에서 Export

Tracing 화면 우측 상단 다운로드 아이콘 → 시간 필터를 커스텀 구간으로 설정한 뒤 CSV 내보내기 → 스프레드시트에서 합산.

수동에 가까워서 반복 작업에는 부적합. 1회성 확인용.

## 세션 미분리 문제에 대한 제안

DeepEval 테스트를 세션 없이 돌리는 중이라면, 앞으로 trace에 **tag** 또는 **metadata**, **session_id**를 명시해 두면 집계/필터링이 훨씬 편해진다.

```python
from langfuse import get_client

langfuse = get_client()
langfuse.update_current_trace(
    tags=["deepeval-test", "batch-20260321-1545"],
    session_id="deepeval-run-001"  # 이후 세션별 추적 가능
)
```

이렇게 박아두면 Metrics API에서 tag 필터가 가능하고, 대시보드에서도 깔끔하게 분리된다.

## 포함 관계 & 선택 기준

- **Metrics API** ⊃ 여러 `measure`(`totalCost`, `totalTokens`, `count`, `latency`) ⊃ `aggregation`(`sum`, `count`, `avg`, `p95`)
- **Observations API**는 Metrics API가 숨기는 **raw row**(개별 span)를 그대로 노출 → 더 유연하지만 느림
- 포함 관계 관점: Metrics API = 집계된 결과, Observations API = 그 집계의 원천 데이터

선택 기준
1. 빠른 합산이 필요하면 **방법 1 (Metrics API)**
2. measure가 안 맞거나 커스텀 계산이 필요하면 **방법 2 (Observations API)**
3. 1회성 공유/엑셀 가공이면 **방법 3 (UI Export)**

환경변수 `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`, `LANGFUSE_BASE_URL` 세팅돼 있으면 방법 1은 바로 돌릴 수 있음.

## 연관노트
- [[langfuse_한번 요청시 나온 토큰등 추적 정보를 묶어서 보기]]
- [[langfuse_SessionID_ContextVar]]
