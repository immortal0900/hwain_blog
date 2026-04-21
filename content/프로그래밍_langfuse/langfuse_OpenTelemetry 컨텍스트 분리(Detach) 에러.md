---
생성날짜:
  - 2026-03-12 13:28
마지막수정날짜:
  - 2026-03-12-목요일 13:27
tags:
  - langfuse
  - opentelemetry
  - contextvars
  - 비동기
  - 에러분석
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
Langfuse 3.x로 비동기 LangChain을 돌리다가 아래 로그가 계속 찍혀서 원인 정리.

```
Failed to detach context Traceback (most recent call last): File "/usr/local/lib/python3.12/site-packages/opentelemetry/context/__init__.py", line 155, in detach _RUNTIME_CONTEXT.detach(token) File "/usr/local/lib/python3.12/site-packages/opentelemetry/context/contextvars_context.py", line 53, in detach self._current_context.reset(token) ValueError: <Token var=<ContextVar name='current_context' default={} at 0x7f099723b1a0> at 0x7f08a31d7f80> was created in a different Context
```

## 한 줄 요약

비동기 환경에서 **트레이싱(tracing, LLM 호출 구간을 기록해 성능/비용을 추적하는 기술) 컨텍스트 토큰이 만들어진 Task와 다른 Task에서 해제**되어서 나는 에러다.

## 근본 원인

Python `contextvars`(컨텍스트 변수, 비동기 Task마다 독립된 저장소를 갖는 표준 라이브러리)는 Task 단위로 상태를 격리한다.

```
Task A에서 토큰 생성:  token = context.attach(span)
     │
     └──→ Task B에서 해제 시도:  context.detach(token)  ← ValueError!
```

토큰은 **생성된 동일 컨텍스트에서만 `reset`** 가능한 규칙이 있는데, Langfuse/OpenTelemetry가 span을 시작한 코루틴과 종료하는 코루틴이 다른 경우 이 규칙이 깨진다.

## 이 프로젝트에서 벌어진 경로

Langfuse 3.x 내부는 OpenTelemetry(오픈텔레메트리, 분산 시스템 관측성을 위한 표준 SDK)를 사용한다. LangChain 비동기 콜백 체인에서 이런 흐름이 생긴다.

1. **Span 시작**: 코루틴 A가 `context.attach()` 호출
2. **Span 종료**: 코루틴 B가 `context.detach()` 호출
3. 두 코루틴의 `contextvars.Context`가 다르므로 `ValueError`

## 심각도

**낮음.** 실제 기능에는 영향 없음. 트레이싱 데이터가 일부 누락될 수는 있지만 앱은 정상 동작한다. OpenTelemetry 내부에서 `try/except`로 잡아 로그만 남기는 구조.

## 해결 방법

| 방법 | 설명 |
| --- | --- |
| **무시** | 기능 영향 없으므로 로그 레벨 조정으로 숨김 |
| **로그 억제** | `logging.getLogger("opentelemetry.context").setLevel(logging.CRITICAL)` |
| **근본 해결** | `propagate_attributes`로 컨텍스트를 명시적으로 전파 |

`propagate_attributes` 근본 해결 예시:

```python
from langfuse import propagate_attributes

# 비동기 태스크 생성 시 컨텍스트를 명시적으로 전파
with propagate_attributes():
    await some_async_langchain_call()
```

이 블록 안에서 생성된 자식 Task는 부모의 트레이싱 컨텍스트를 상속받아 토큰 불일치가 사라진다.

---

## 쉽게 풀어서

### 식당 번호표 비유

```
1. 내가 직접 줄 서서 번호표 37번을 받음
2. 식사 끝나고 번호표를 반납해야 함
3. 그런데 내 친구가 대신 반납하려고 함
4. 식당 직원: "이거 당신이 받은 게 아닌데요?" → 거부!
```

이 에러는 딱 이 상황이다. 발급자와 반납자가 달라서 거절당하는 것.

### 실제 상황에 대입

프로그램에는 **여러 작업(Task)**이 동시에 돈다.

```
[작업 A] "나 지금 LLM 호출 시작한다!" → 추적 시스템에서 출입증(토큰) 발급받음
    │
    │  ... 시간이 흐르고 ...
    │
[작업 B] "LLM 호출 끝났으니 출입증 반납할게!" → 추적 시스템: "너한테 발급한 적 없는데?" → 에러!
```

A가 받은 출입증을 B가 반납하려다 막힌 것.

### 왜 이런 일이 생기나

Langfuse가 LLM 호출을 모니터링할 때
1. **시작과 끝을 기록**해서 구간 시간/비용을 계산한다
2. 비동기(async) 환경에서는 "시작"과 "끝" 담당 작업자가 다를 수 있다
3. Python은 **"네가 시작했으면 네가 끝내라"** 규칙을 강제한다
4. 규칙 위반 시 로그가 찍힘

### 그래서 문제가 되나

되지 않는다. 추적 시스템 내부 cleanup에서 나는 로그일 뿐, 실제 LLM 호출이나 애플리케이션 동작은 멀쩡하다. 번호표 반납이 안 됐을 뿐, 식사는 이미 끝난 상태.

로그가 거슬리면 이 한 줄로 숨김 가능.

```python
import logging
logging.getLogger("opentelemetry.context").setLevel(logging.CRITICAL)
```

---

## 해결 방법 3가지

### 방법 1: 무시하기

번호표 반납이 안 됐을 뿐, 식사(앱 동작)는 정상. 그대로 두면 된다.

### 방법 2: 에러 로그 숨기기

에러 자체를 안 보이게 한다.

```
평소: "경고", "알림", "에러" 전부 출력
변경: "심각한 에러"만 출력
```

```python
import logging
logging.getLogger("opentelemetry.context").setLevel(logging.CRITICAL)
```

귀마개를 끼는 식이다. 원인을 없앤 게 아니라 들리지 않게 한 것.

### 방법 3: 근본 해결 (`propagate_attributes`)

비유로 다시 풀면

```
문제 상황:
  작업 A가 출입증을 받음 → 작업 B가 반납 시도 → 거부!

해결:
  작업 A가 출입증을 받을 때, "이건 우리 팀 공용이야"라고 등록
  → 작업 B도 같은 팀이니까 반납 가능!
```

`propagate_attributes()`가 하는 일이 바로 이것. **"이 안에서 생기는 모든 작업은 같은 팀이다"** 라고 선언하는 구문이다.

```python
from langfuse import propagate_attributes

# 이 블록 안의 모든 작업은 같은 컨텍스트(팀)를 공유
with propagate_attributes():
    await some_async_langchain_call()
```

#### 적용 전후

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

## 한마디 요약

| 방법 | 비유 | 효과 |
| --- | --- | --- |
| 무시 | 그냥 냅두기 | 앱 정상, 로그만 지저분 |
| 로그 숨기기 | 귀마개 | 로그 깔끔, 원인은 그대로 |
| `propagate_attributes` | 팀 등록 | 원인 자체를 제거 |

선택 기준: 로그가 거슬리지 않으면 방법 1, 거슬리면 방법 2, Langfuse 트레이싱 데이터 누락이 확인되면 방법 3.

[[langfuse_propagate_attributes()의 원래 역할]]
