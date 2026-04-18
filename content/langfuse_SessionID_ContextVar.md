---
생성날짜:
  - 2026-03-20 21:16
마지막수정날짜:
  - 2026-03-20-금요일 21:16
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
**`contextvars.ContextVar` (채택):** 비동기 작업(`asyncio.Task`) 단위로 데이터를 격리합니다. 파이썬 3.7 이상에서는 `asyncio.create_task()`가 실행될 때 부모의 컨텍스트를 자식에게 자동으로 복사합니다. 따라서 FastAPI 동시 요청 간 데이터 오염이 원천 차단되며, 코드 수정 없이 하위 에이전트로 값이 안전하게 자동 상속됩니다.

## "asyncio.create_task() 경계를 넘는다"는 말의 의미

### 먼저, 경계(boundary)가 뭔지부터

`asyncio.create_task()`를 호출하면 **새로운 독립 실행 단위**가 만들어집니다.

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

이 `create_task()` 호출 지점이 바로 **"경계"**입니다. 부모와 자식이 갈라지는 분기점이죠.

---

### 문제: 일반 변수는 이 경계를 못 넘는다

```python
# 부모 코루틴
session_id = "job_123"

async def policy_agent():
    print(session_id)  # ← 이건 클로저라서 되긴 하지만...

# 실제로는 policy_agent가 별도 모듈에 있으므로
# 부모의 지역 변수를 직접 참조할 방법이 없음
```

비유하면:

```
[부모 사무실]  session_id = "job_123" (책상 위 메모)
      │
      ├── create_task() ──→ [자식 사무실 1]  "메모가 없다... job_id가 뭐지?"
      ├── create_task() ──→ [자식 사무실 2]  "메모가 없다..."
      └── create_task() ──→ [자식 사무실 3]  "메모가 없다..."

※ 각 사무실은 독립 공간이라 부모 책상 위 메모를 볼 수 없음
```

---

### 해결: ContextVar는 이 경계를 자동으로 넘는다

Python 3.7+에서 `asyncio.create_task()`는 **부모의 ContextVar를 자식에게 자동 복사**합니다. 이것이 Python 런타임이 보장하는 동작입니다.

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

---

### 핵심 한 줄 정리

> **"경계를 넘는다"** = `create_task()`로 만든 **별도의 독립 Task에서도** 부모가 설정한 ContextVar 값을 읽을 수 있다

이것이 없었다면, 7개 에이전트 함수에 전부 `session_id` 인자를 추가해야 했고, 그 안에서 호출하는 LLM 함수에도 또 넘겨야 했을 것입니다. ContextVar 덕분에 **진입점 1곳에서 설정 → 하위 모든 Task가 자동으로 읽음** 이 가능한 것입니다.