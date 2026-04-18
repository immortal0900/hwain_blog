---
생성날짜:
  - 2026-03-12 13:31
마지막수정날짜:
  - 2026-03-12-목요일 13:31
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---

### 문제 배경부터

Python 비동기 프로그래밍에서 각 Task는 **독립된 주머니(Context)**를 갖고 있습니다.

```
메인 함수 (주머니 A: session_id="abc", user_id="kim")
    │
    ├─→ asyncio.create_task(작업1)  → 새 주머니 B (비어있음!)
    ├─→ asyncio.create_task(작업2)  → 새 주머니 C (비어있음!)
    └─→ asyncio.create_task(작업3)  → 새 주머니 D (비어있음!)
```

자식 Task가 생성되면 **부모의 주머니 내용물이 자동으로 복사되지 않습니다.** 이게 Python `contextvars`의 기본 동작입니다.

---

### Langfuse 관점에서 문제

Langfuse는 추적 정보(어떤 세션인지, 어떤 trace인지)를 이 주머니(Context)에 넣어둡니다.

```
@observe() 데코레이터가 실행됨
  └─ 주머니에 넣음: trace_id="tr_123", span_id="sp_456"
      │
      ├─→ 비동기 작업 1: 주머니가 비어있음 → "나 어떤 trace에 속한 거지?" → 추적 끊김
      └─→ 비동기 작업 2: 주머니가 비어있음 → 같은 문제
```

LLM 호출이 여러 비동기 작업으로 나뉘면, 자식 작업들이 **"나는 어떤 trace의 일부인지"를 모르는 상태**가 됩니다.

---

### `propagate_attributes()`가 하는 일

**"부모 주머니 내용물을 자식들에게 복사해라"** 라는 명령입니다.

```
with propagate_attributes():
    # 이 블록 안에서 생성되는 모든 자식 Task는
    # 부모의 주머니 내용물을 복사받음

    await async_llm_call()
```

```
적용 후:

메인 함수 (주머니 A: trace_id="tr_123")
    │
    │  with propagate_attributes():
    │
    ├─→ 작업1 (주머니 B: trace_id="tr_123")  ← 복사됨!
    ├─→ 작업2 (주머니 C: trace_id="tr_123")  ← 복사됨!
    └─→ 작업3 (주머니 D: trace_id="tr_123")  ← 복사됨!
```

모든 자식이 **"나는 trace tr_123의 일부야"**를 알게 되니까:

- 추적이 끊기지 않고
- 출입증(토큰)도 같은 컨텍스트에서 발급/반납되므로 아까 에러도 사라짐

---

### 정리

|없을 때|있을 때|
|---|---|
|자식 Task는 빈 주머니로 시작|부모 주머니 내용물을 복사받음|
|추적 데이터 끊김|하나의 trace로 연결됨|
|토큰 컨텍스트 불일치 → 에러|같은 컨텍스트 → 정상|

**한 마디로**: 비동기 환경에서 **추적 정보가 자식 Task로 전파되도록 보장**하는 함수입니다. 이름 그대로 "attributes를 propagate(전파)하라"는 뜻이죠.

[[langfuse_span_id attributes 뜻]]