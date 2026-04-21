---
생성날짜:
- 2026-04-21 13:20
마지막수정날짜:
- 2026-04-21-화요일 13:20
tags:
- 설계원칙
- 디자인패턴
- 행동패턴
- AI에이전트
별칭:
- State
- 상태 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# State 패턴

**"객체가 내부 상태(State, 현재 처한 단계 또는 모드)에 따라 행동 자체를 바꾸게 하는 행동 패턴."**

State 패턴(상태 패턴, 상태 기반 행동 전환)은 GoF 행동(Behavioral) 패턴 중 하나다. `if status == "A": ... elif status == "B": ...` 같은 거대 분기를 상태별 클래스로 분해해, 마치 "상태가 바뀌면 객체의 클래스가 바뀐 것처럼" 동작하게 한다. 유한 상태 기계(FSM, Finite State Machine, 유한한 상태와 상태 사이 전이 규칙으로 동작을 정의하는 모델)의 OOP 구현 형태다.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 행동(Behavioral) 패턴 |
| 목적 | 상태에 따라 동작이 달라지는 객체를, 상태별 클래스로 분해 |
| 핵심 질문 | "상태 플래그를 보고 분기하는 거대한 if/elif가 있는가?" |
| 대안 | 단순 enum + 분기, Strategy (상태 개념 없는 교체) |
| 트레이드오프 | 상태 클래스 수가 늘어남. 대신 상태별 로직이 깔끔히 분리됨 |

> **한마디 요약**: "상태별 if문을 상태별 클래스로 바꿔라."

---

## 일상 비유: 자판기

자판기는 같은 "동전 투입구 누름" 동작에도 상태에 따라 다르게 반응한다.

| 현재 상태 | 동전 투입 시 반응 | 버튼 누름 시 반응 |
| --- | --- | --- |
| IDLE (대기) | 금액 누적, 상태 → HAS_COIN | "동전을 넣으세요" 경고 |
| HAS_COIN | 금액 추가 누적 | 상품 배출, 상태 → DISPENSING |
| DISPENSING | 무시 | 무시 |
| SOLD_OUT (매진) | 즉시 반환 | "매진입니다" |

버튼 하나에도 상태에 따라 4가지 다른 반응이 있다. if/else로 쓰면 `handle_button()` 안에 4분기, `handle_coin()` 안에 4분기, 다른 메서드에도 4분기, 상태 하나 늘리면 전 메서드 건드려야 한다.

State 패턴은 이걸 상태별 클래스로 분해해 "동전 투입"이라는 메서드를 상태 객체가 자기 식대로 해석하게 만든다.

---

## 구조

| 역할 | 설명 |
| --- | --- |
| Context | 외부에서 보는 객체. 내부에 현재 State 참조를 들고 있음 |
| State (인터페이스) | 상태가 응답할 수 있는 메서드 목록 (`on_event_x`, `on_event_y`) |
| ConcreteState | 각 상태별 실제 동작 구현. 다음 상태로의 전이도 담당 |

단계 분해:
1. **1단계**: Client가 `context.do_something(event)` 호출
2. **2단계**: Context가 `self._state.do_something(self, event)`로 위임
3. **3단계**: 현재 State 객체가 동작을 수행하고, 필요하면 `context.set_state(NextState())`로 상태 전이

전이(Transition)는 누가 결정하는가에 따라 두 스타일이 있다.
- **State 주도(권장)**: 각 State가 다음 State를 결정. 상태 객체의 응집도가 높음
- **Context 주도**: Context의 전이 테이블이 결정. 전이 로직이 한 곳에 집중됨

---

## AI 에이전트 개발 예시 1: 대화 플로우 상태

Task-Oriented Agent(작업 지향 에이전트, 특정 업무를 단계별로 수행하는 봇)는 보통 아래 상태를 오간다.

```
COLLECT_INPUT → CONFIRM → EXECUTE → REPORT → DONE
          ↑                ↓
          └─ REJECT ───────┘
```

### 나쁜 예: if-elif 지옥

```python
class BookingAgent:
    def __init__(self):
        self.status = "COLLECT_INPUT"
        self.data = {}

    def handle(self, user_msg):
        if self.status == "COLLECT_INPUT":
            if "from" in user_msg and "to" in user_msg:
                self.data = parse_route(user_msg)
                self.status = "CONFIRM"
                return f"Book {self.data['from']} → {self.data['to']}?"
            return "Tell me route."

        elif self.status == "CONFIRM":
            if user_msg.strip().lower() in ("yes", "y"):
                self.status = "EXECUTE"
                return self._execute()
            elif user_msg.strip().lower() in ("no", "n"):
                self.status = "COLLECT_INPUT"
                self.data = {}
                return "Okay, tell me new route."
            return "Please answer yes or no."

        elif self.status == "EXECUTE":
            return "Processing..."

        elif self.status == "REPORT":
            ...
```

상태 하나 추가하면 모든 메서드에 분기 추가. 상태 수 × 이벤트 수 크기의 잠재적 버그 매트릭스가 생긴다.

### 좋은 예: State 패턴 적용

```python
from abc import ABC, abstractmethod

class BookingState(ABC):
    @abstractmethod
    def handle(self, context: "BookingAgent", user_msg: str) -> str: ...


class CollectInputState(BookingState):
    def handle(self, context, user_msg):
        if "from" in user_msg and "to" in user_msg:
            context.data = parse_route(user_msg)
            context.set_state(ConfirmState())
            return f"Book {context.data['from']} → {context.data['to']}?"
        return "Tell me route (from / to)."


class ConfirmState(BookingState):
    def handle(self, context, user_msg):
        ans = user_msg.strip().lower()
        if ans in ("yes", "y"):
            context.set_state(ExecuteState())
            return context.state.handle(context, "")  # 바로 실행 트리거
        if ans in ("no", "n"):
            context.data = {}
            context.set_state(CollectInputState())
            return "Okay, tell me new route."
        return "Please answer yes or no."


class ExecuteState(BookingState):
    def handle(self, context, user_msg):
        result = booking_api.reserve(**context.data)
        context.result = result
        context.set_state(ReportState())
        return context.state.handle(context, "")


class ReportState(BookingState):
    def handle(self, context, user_msg):
        context.set_state(DoneState())
        return f"Confirmed. Ticket: {context.result.ticket_id}"


class DoneState(BookingState):
    def handle(self, context, user_msg):
        return "Booking already done. Start a new conversation."


class BookingAgent:
    def __init__(self):
        self.state: BookingState = CollectInputState()
        self.data: dict = {}
        self.result = None

    def set_state(self, state: BookingState):
        self.state = state

    def handle(self, user_msg: str) -> str:
        return self.state.handle(self, user_msg)
```

상태 하나 추가 = 클래스 하나 추가. 기존 State 클래스는 건드릴 필요 없다. OCP(개방 폐쇄 원칙)에 정확히 부합.

---

## AI 에이전트 개발 예시 2: Tool 실행 상태

LLM이 Tool을 호출하는 단일 실행 단위도 상태를 가진다.

```
PENDING → RUNNING → SUCCESS
             │
             └──────→ FAILURE → RETRYING → SUCCESS | FAILED
```

재시도 정책이 복잡한 Tool 실행기는 이걸 FSM으로 모델링하면 버그가 훨씬 줄어든다.

```python
class ToolExecutionContext:
    def __init__(self, tool, args, max_retries=3):
        self.tool = tool
        self.args = args
        self.max_retries = max_retries
        self.attempt = 0
        self.result = None
        self.error = None
        self.state = PendingState()

    def tick(self):
        self.state.tick(self)


class PendingState:
    def tick(self, ctx):
        ctx.state = RunningState()
        ctx.state.tick(ctx)


class RunningState:
    def tick(self, ctx):
        try:
            ctx.result = ctx.tool.run(ctx.args)
            ctx.state = SuccessState()
        except Exception as e:
            ctx.error = e
            ctx.state = FailureState()


class FailureState:
    def tick(self, ctx):
        if ctx.attempt < ctx.max_retries:
            ctx.attempt += 1
            ctx.state = RetryingState()
            ctx.state.tick(ctx)
        else:
            ctx.state = FailedState()


class RetryingState:
    def tick(self, ctx):
        time.sleep(2 ** ctx.attempt)  # exponential backoff
        ctx.state = RunningState()
        ctx.state.tick(ctx)


class SuccessState:
    def tick(self, ctx): pass  # terminal


class FailedState:
    def tick(self, ctx): pass  # terminal
```

---

## 실제 라이브러리에서의 State

| 시스템 | State 적용 |
| --- | --- |
| LangGraph | 대화 흐름을 노드(State)와 엣지(Transition)로 모델링 |
| XState (JS) | 선언적으로 FSM을 정의 |
| AWS Step Functions | 서버리스 워크플로를 JSON FSM으로 기술 |
| TCP 소켓 | CLOSED/LISTEN/SYN_SENT/ESTABLISHED/FIN_WAIT 등 11개 상태 |
| Git | 파일이 Untracked → Staged → Committed 상태를 오감 |

---

## 직접 확인해 보기

프로젝트에서 "거대 상태 분기"를 찾는다.

```bash
# status, state, mode 변수를 검사하는 if/elif 체인
grep -rn "self.status" src/ | wc -l
grep -rn "self.state ==" src/
```

한 메서드 안에 `elif state == ...`가 3개 이상이면 State 패턴 도입 후보.

---

## 왜 이렇게 하는가

State 패턴의 설계 의도와 장점:

- **상태 로직의 응집**: 상태별 로직이 한 클래스에 모여 읽기 쉬움
- **전이 그래프의 가시화**: 각 State의 메서드에서 `set_state(...)` 호출만 모아보면 FSM 다이어그램이 만들어짐
- **OCP 준수**: 새 상태 추가가 새 클래스 하나 추가로 끝남
- **테스트 용이**: 상태 클래스 단위로 독립 테스트 가능

트레이드오프:
- 상태 수가 적고 전이가 단순하면 과한 도입. 상태 2~3개면 enum + switch가 낫다
- 상태끼리 공유해야 하는 데이터가 많으면 Context가 뚱뚱해짐. 이때는 Context에 도메인 책임을 적절히 분배
- 상태 전이가 매우 복잡하면 별도의 "상태 기계 라이브러리"(Python `transitions`, JS XState) 도입을 고려

---

## 포함 관계와 다른 패턴

| 패턴 | 공통점 | 차이 |
| --- | --- | --- |
| State | 객체가 "상태별 다른 클래스"처럼 행동 | 상태 객체끼리 다음 상태를 알고 전이 |
| Strategy | 객체가 주입된 전략의 동작 수행 | 상태 개념 없음, 외부에서 전략 교체 |
| Template Method | 절차 공통, 디테일 변형 | 상태 전이 개념 없음 |

- **State + Template Method**: 템플릿의 특정 단계에서 `self.state.do_x()`로 위임하는 조합이 자주 나옴. [[Template Method 패턴]] 참고
- **State + Pub/Sub**: 상태가 바뀔 때 `state_changed` 이벤트를 발행해 UI, 로거, 메트릭이 구독하는 구조가 실무에서 흔함. [[Pub-Sub 패턴]] 참고

---

## 한마디 요약

> **"`if state == ...` 체인이 두 함수 이상에 퍼지기 시작했다면, 상태를 클래스로 승격시켜라."**

관련 문서:
- [[Template Method 패턴]]
- [[Pub-Sub 패턴]]
- [[SOLID원칙]]
