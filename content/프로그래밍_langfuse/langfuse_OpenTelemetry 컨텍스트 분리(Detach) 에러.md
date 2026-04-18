---
생성날짜:
  - 2026-03-12 13:28
마지막수정날짜:
  - 2026-03-12-목요일 13:27
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
```
Failed to detach context Traceback (most recent call last): File "/usr/local/lib/python3.12/site-packages/opentelemetry/context/__init__.py", line 155, in detach _RUNTIME_CONTEXT.detach(token) File "/usr/local/lib/python3.12/site-packages/opentelemetry/context/contextvars_context.py", line 53, in detach self._current_context.reset(token) ValueError: <Token var=<ContextVar name='current_context' default={} at 0x7f099723b1a0> at 0x7f08a31d7f80> was created in a different Context
```

## OpenTelemetry 컨텍스트 분리(Detach) 에러

**한 줄 요약**: 비동기 환경에서 **트레이싱 컨텍스트 토큰이 생성된 곳과 다른 곳에서 해제**하려고 해서 발생하는 에러입니다.

---

### 근본 원인

Python의 `contextvars`는 각 async Task마다 독립된 컨텍스트를 갖습니다:

```
Task A에서 토큰 생성:  token = context.attach(span)
     │
     └──→ Task B에서 해제 시도:  context.detach(token)  ← ValueError!
```

토큰은 **생성된 동일한 컨텍스트에서만 `reset`** 할 수 있는데, Langfuse/OpenTelemetry가 span을 시작한 코루틴과 종료하는 코루틴이 다르기 때문에 발생합니다.

---

### 이 프로젝트에서 원인

Langfuse 3.x가 내부적으로 OpenTelemetry를 사용하는데, LangChain의 비동기 콜백 체인에서:

1. **Span 시작** → 코루틴 A에서 `context.attach()` 호출
2. **Span 종료** → 코루틴 B에서 `context.detach()` 호출
3. 서로 다른 `contextvars.Context`이므로 → `ValueError`

---

### 심각도

**낮음** — 실제 기능에 영향 없습니다. 트레이싱 데이터가 일부 누락될 수 있지만 앱 자체는 정상 동작합니다. OpenTelemetry가 내부적으로 `try/except`로 잡아서 로그만 남깁니다.

---

### 해결 방법

|방법|설명|
|---|---|
|**무시**|기능에 영향 없으므로 로그 레벨 조정으로 숨기기|
|**로그 억제**|`logging.getLogger("opentelemetry.context").setLevel(logging.CRITICAL)`|
|**근본 해결**|`propagate_attributes`로 컨텍스트를 명시적으로 전파|

`propagate_attributes`를 사용한 근본 해결 예시:

```python
from langfuse import propagate_attributes

# 비동기 태스크 생성 시 컨텍스트를 명시적으로 전파
with propagate_attributes():
    await some_async_langchain_call()
```

이렇게 하면 자식 태스크가 부모의 트레이싱 컨텍스트를 올바르게 상속받아서 토큰 불일치가 사라집니다.


---

## 쉽게 풀어서 설명

### 비유: 식당 번호표

식당에서 **번호표**를 받는다고 생각하세요.

```
1. 내가 직접 줄 서서 번호표 37번을 받음
2. 식사 끝나고 번호표를 반납해야 함
3. 그런데 내 친구가 대신 반납하려고 함
4. 식당 직원: "이거 당신이 받은 게 아닌데요?" → 거부!
```

이게 이 에러의 전부입니다.

---

### 실제 상황에 대입

프로그램에는 **여러 작업(Task)**이 동시에 돌아갑니다.

```
[작업 A] "나 지금 LLM 호출 시작한다!" → 추적 시스템에서 출입증(토큰) 발급받음
    │
    │  ... 시간이 흐르고 ...
    │
[작업 B] "LLM 호출 끝났으니 출입증 반납할게!" → 추적 시스템: "너한테 발급한 적 없는데?" → 에러!
```

**작업 A**가 받은 출입증을 **작업 B**가 반납하려니까 거부당한 겁니다.

---

### 왜 이런 일이 생기나?

Langfuse(추적 도구)가 LLM 호출을 모니터링할 때:

1. **"시작"과 "끝"을 기록**해서 얼마나 걸렸는지 추적합니다
2. 그런데 비동기(async) 환경에서는 "시작"을 담당한 작업자와 "끝"을 담당한 작업자가 **다른 사람**일 수 있습니다
3. Python은 보안상 **"니가 시작한 건 니가 끝내라"** 규칙을 강제합니다
4. 규칙 위반 → 이 에러 로그가 찍힘

---

### 그래서 문제가 되나?

**아닙니다.** 추적 시스템 내부의 정리(cleanup) 과정에서 나는 에러일 뿐, 실제 LLM 호출이나 앱 동작에는 영향이 없습니다. 번호표 반납이 안 됐을 뿐, 식사는 이미 잘 마친 상태입니다.

로그가 거슬리면 이 한 줄로 숨길 수 있습니다:

```python
import logging
logging.getLogger("opentelemetry.context").setLevel(logging.CRITICAL)
```

---

## 해결 방법 3가지

### 방법 1: 무시하기

번호표 반납이 안 됐을 뿐, 식사(앱 동작)는 정상입니다. 그냥 두면 됩니다.

---

### 방법 2: 에러 로그 숨기기

에러 자체를 안 보이게 하는 겁니다.

```
평소: "경고", "알림", "에러" 전부 출력
변경: "심각한 에러"만 출력
```

```python
import logging
logging.getLogger("opentelemetry.context").setLevel(logging.CRITICAL)
```

이건 **귀마개를 끼는 것**과 같습니다. 문제가 사라진 게 아니라 안 들리게 한 것뿐입니다.

---

### 방법 3: 근본 해결 (`propagate_attributes`)

아까 비유로 돌아가면:

```
문제 상황:
  작업 A가 출입증을 받음 → 작업 B가 반납 시도 → 거부!

해결:
  작업 A가 출입증을 받을 때, "이건 우리 팀 공용이야"라고 등록
  → 작업 B도 같은 팀이니까 반납 가능!
```

`propagate_attributes()`가 하는 일이 정확히 이겁니다. **"이 안에서 생기는 모든 작업은 같은 팀이다"** 라고 선언하는 것입니다.

```python
from langfuse import propagate_attributes

# 이 블록 안의 모든 작업은 같은 컨텍스트(팀)를 공유
with propagate_attributes():
    await some_async_langchain_call()
```

#### 적용 전 vs 후

```
적용 전:
  작업 A (컨텍스트 #1) → 출입증 발급
  작업 B (컨텍스트 #2) → 반납 시도 → "넌 다른 팀인데?" → 에러

적용 후:
  propagate_attributes() 선언
    └─ 작업 A (공유 컨텍스트) → 출입증 발급
    └─ 작업 B (공유 컨텍스트) → 반납 → "같은 팀이네, OK" → 성공
```

---

### 정리

|방법|비유|효과|
|---|---|---|
|무시|그냥 냅두기|앱 정상, 로그만 지저분|
|로그 숨기기|귀마개|로그 깔끔, 원인은 그대로|
|`propagate_attributes`|팀 등록|원인 자체를 제거|

**추천**: 로그가 거슬리지 않으면 방법 1, 거슬리면 방법 2, Langfuse 트레이싱 데이터가 누락되는 게 확인되면 방법 3입니다.
[[langfuse_propagate_attributes()의 원래 역할]]