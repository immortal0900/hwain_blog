---
생성날짜:
  - 2026-03-20 21:16
마지막수정날짜:
  - 2026-03-20-금요일 21:16
tags:
  - langfuse
  - contextvars
  - asyncio
  - session
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
FastAPI로 동시에 여러 요청을 받는 환경에서 **Session ID(요청/파이프라인 한 번을 식별하는 ID)**를 하위 LLM 호출까지 안전하게 내려주려고 `contextvars.ContextVar`(파이썬 표준 컨텍스트 저장소)를 채택했다. 선정 이유 정리.

## 왜 `contextvars.ContextVar`인가

- **비동기 Task 단위 격리**: `asyncio.Task`마다 독립 컨텍스트를 가지므로 데이터 오염이 원천 차단된다.
- **자동 상속**: 파이썬 3.7+에서 `asyncio.create_task()` 호출 시 부모 컨텍스트가 자식에게 자동 복사된다.
- **코드 변경 최소**: 하위 에이전트 함수 시그니처를 고칠 필요 없이 값이 그대로 따라간다.

## "asyncio.create_task() 경계를 넘는다"는 표현의 의미

### 경계(boundary)가 뭔지

`asyncio.create_task()`를 호출하면 **새로운 독립 실행 단위**가 만들어진다.

```
부모 코루틴 (run_graph_task)
    │
    │  await graph.ainvoke(...)  ← 여기서 내부적으로
    │
    ├── asyncio.create_task(policy_agent())     ← 새 Task 1 (독립)
    ├── asyncio.create_task(location_agent())   ← 새 Task 2 (독립)
    ├── asyncio.create_task(nearby_market())    ← 새 Task 3 (독립)
    └── ... 7개
```

이 `create_task()` 호출 지점이 **"경계"**. 부모와 자식이 갈라지는 분기점이다.

### 문제: 일반 변수는 이 경계를 못 넘는다

```python
# 부모 코루틴
session_id = "job_123"

async def policy_agent():
    print(session_id)  # ← 이건 클로저라서 되긴 하지만...

# 실제로는 policy_agent가 별도 모듈에 있으므로
# 부모의 지역 변수를 직접 참조할 방법이 없음
```

비유

```
[부모 사무실]  session_id = "job_123" (책상 위 메모)
      │
      ├── create_task() ──→ [자식 사무실 1]  "메모가 없다... job_id가 뭐지?"
      ├── create_task() ──→ [자식 사무실 2]  "메모가 없다..."
      └── create_task() ──→ [자식 사무실 3]  "메모가 없다..."

※ 각 사무실은 독립 공간이라 부모 책상 위 메모를 볼 수 없음
```

### 해결: ContextVar는 이 경계를 자동으로 넘는다

Python 3.7+에서 `asyncio.create_task()`는 **부모의 ContextVar 스냅샷을 자식에게 자동 복사**한다. 런타임 차원에서 보장되는 동작이다.

```
[부모 사무실]  _active_session_id.set("job_123")  ← ContextVar에 저장
      │
      │  ※ create_task() 시 Python이 자동으로 ContextVar 스냅샷을 복사
      │
      ├── create_task() ──→ [자식 사무실 1]  _active_session_id.get() = "job_123" ✓
      ├── create_task() ──→ [자식 사무실 2]  _active_session_id.get() = "job_123" ✓
      └── create_task() ──→ [자식 사무실 3]  _active_session_id.get() = "job_123" ✓

※ 메모를 책상에 놓는 게 아니라 "사원증에 새긴" 것
※ 사원증은 어느 사무실에 가든 몸에 붙어 다님
```

## 핵심 한 줄 정리

> **"경계를 넘는다"** = `create_task()`로 만든 **별도의 독립 Task에서도** 부모가 설정한 ContextVar 값을 읽을 수 있다

이게 없었다면 7개 에이전트 함수 전부에 `session_id` 인자를 추가하고, 그 안에서 호출하는 LLM 함수에도 또 넘겨야 했을 것. ContextVar 덕분에 **진입점 1곳에서 설정 → 하위 모든 Task가 자동으로 읽음** 구조가 가능해진다.

## 포함 관계 정리

- `contextvars` (모듈) ⊃ `ContextVar` (클래스) ⊃ `set()` / `get()` (메서드)
- `asyncio.Task` ⊃ 각자의 `Context` ⊃ 여러 `ContextVar` 값
- Langfuse의 trace context 전파도 내부적으로 이 `contextvars` 위에서 동작함 → [[langfuse_propagate_attributes()의 원래 역할]]과 같은 뿌리
