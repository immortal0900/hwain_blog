---
생성날짜:
  - 2026-03-21 17:36
마지막수정날짜:
  - 2026-03-21-토요일 17:36
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
Langfuse에서 특정 시점 이후의 토큰/호출량을 집계하는 방법을 찾아볼게요.HWAIN, 좋은 질문이에요. Langfuse에서 특정 시점 이후의 토큰/호출량을 일괄 집계하는 방법이 크게 3가지 있어요.

---

**방법 1: Metrics API (가장 추천)**

Langfuse의 Metrics API를 사용하면 `fromTimestamp`/`toTimestamp`로 시간 범위를 지정하고, `totalCost`, `count` 등의 measure를 aggregation해서 한 번에 받을 수 있어요. Hobby 플랜이니 Cloud API 사용 가능합니다.

Python SDK로 하면 이렇게 됩니다:

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

토큰 관련 measure로는 `totalTokens`(sum), `count`(count), `totalCost`(sum), `latency`(avg/p95) 등을 사용할 수 있어요. 다만 v2에서 `totalTokens`가 observations view에서 지원되는지는 v1의 traces view에서 확인된 것이라, 안 되면 v1 legacy를 쓰면 됩니다:

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

---

**방법 2: Observations API로 raw 데이터 가져와서 합산**

SDK의 `api.observations.get_many()`를 사용하면 individual observation 단위로 usage 정보를 가져올 수 있어요. Metrics API가 안 되는 경우 이걸 쓰면 됩니다:

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

---

**방법 3: UI에서 Export**

스크린샷에서 보이는 Tracing 화면 우측 상단에 다운로드 아이콘이 있어요. 시간 필터를 "Past 1 day" 대신 커스텀으로 설정한 후 CSV로 export하면 스프레드시트에서 합산할 수 있습니다. 다만 이건 수동에 가까워서 반복하기엔 비효율적이에요.

---

**세션 미분리 문제에 대한 제안**

현재 DeepEval 테스트를 세션 없이 돌리고 있다면, 앞으로는 테스트 실행 시 trace에 **tag**나 **metadata**를 붙이는 걸 추천합니다. 예를 들어:

```python
from langfuse import get_client

langfuse = get_client()
langfuse.update_current_trace(
    tags=["deepeval-test", "batch-20260321-1545"],
    session_id="deepeval-run-001"  # 이후 세션별 추적 가능
)
```

이렇게 해두면 Metrics API에서 tag 기반 필터링도 가능하고, 대시보드에서도 분리해서 볼 수 있어요.

가장 빠르게 결과를 얻으려면 **방법 1 (Metrics API)**를 먼저 시도해보세요. 환경변수에 `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`, `LANGFUSE_BASE_URL`만 설정돼 있으면 바로 돌릴 수 있습니다.