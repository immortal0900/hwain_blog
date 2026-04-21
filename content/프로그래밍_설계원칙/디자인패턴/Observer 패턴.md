---
생성날짜:
- 2026-04-21 12:12
마지막수정날짜:
- 2026-04-21-화요일 12:12
tags:
- 설계원칙
- 디자인패턴
- GoF
- 행위패턴
- AI에이전트
- 이벤트드리븐
별칭:
- Observer
- 옵저버
- 관찰자 패턴
- Pub-Sub
type:
- 자료수집
Area/Reasource:
Project:
---
# Observer 패턴이란?

Observer(옵저버, 관찰자 패턴, 한 객체의 상태 변화를 관심 있는 다른 객체들에게 자동 통보하는 행위 디자인 패턴)는 **"어떤 일이 일어났을 때, 그 일에 관심 있는 모두에게 한 번에 알려주는"** 패턴이다. GoF 분류상 **행위(Behavioral) 패턴**에 속함.

일상에서 Pub-Sub(퍼블리시-서브스크라이브, 발행-구독 모델, Kafka/Redis Pub-Sub 같은 분산 메시징 시스템의 기초 개념)이라고 부르는 구조의 객체지향 버전이라고 봐도 된다.

## 한눈에 보기

| 항목     | 내용                                                    |
| ------ | ----------------------------------------------------- |
| 분류     | 행위 패턴(Behavioral)                                     |
| 한 줄 정의 | Subject 상태 변화를 등록된 Observer들에게 자동 통보                  |
| 핵심 구성  | Subject(발행자), Observer(구독자), 등록/해지 API                |
| 해결 문제  | 주체가 "누가 듣고 있는지" 알 필요 없음. 한쪽이 바뀌면 여러 쪽 자동 반영 |
| 관련 원칙  | OCP, SRP, 느슨한 결합(loose coupling)                      |

> **한마디 요약**: "유튜브 구독 알림. 채널은 올리기만 하고 누가 받는지는 신경 안 씀."

---

## 일상 비유

유튜브 채널(Subject)이 새 영상을 올리면, 구독 버튼을 누른 시청자들(Observer)에게 알림이 자동으로 간다. 채널은 "내 구독자가 누구누구인지" 일일이 외워두지 않고, 구독자는 언제든 구독/구독 해지가 가능함. 채널과 시청자가 서로 약하게 연결된 상태(loose coupling)임.

다른 비유:
- 신문 구독(발행자와 구독자, 우체국이 중간 매개)
- 알람 시계(시계는 울리기만 하고 누가 일어나는지 모름)
- 경매장의 입찰 호가 전광판(경매사가 가격을 바꾸면 입찰자 모두가 동시에 봄)

---

## 구조

```
┌─────────────────┐   notify()    ┌──────────────────┐
│    Subject      │ ────────────▶ │   <<interface>>  │
│ + attach(obs)   │               │     Observer     │
│ + detach(obs)   │               │   + update(data) │
│ + notify()      │               └─────────┬────────┘
│ - observers[]   │                          │
└─────────────────┘                          │ implements
        ▲                                    ▼
        │                        ┌──────────────────────┐
 ConcreteSubject                 │  ConcreteObserverA   │
 (상태 변경 시 notify 호출)         │  ConcreteObserverB   │
                                 └──────────────────────┘
```

- **Subject**: 상태 변화가 일어나는 쪽. `attach/detach/notify` 메서드 보유
- **Observer**: `update(data)` 메서드만 약속한 쪽. 누가 구독했는지는 Subject가 리스트로 관리
- **notify()**: 내부 루프로 등록된 Observer 모두에게 `update` 호출

---

## 나쁜 예 vs 좋은 예 (Python)

### 나쁜 예: 직접 결합

```python
class Order:
    def __init__(self):
        self.email = EmailSender()
        self.slack = SlackNotifier()
        self.audit_db = AuditLogger()

    def complete(self):
        # 주문 완료 처리
        self._save()
        # 알림 받을 곳을 모두 직접 호출
        self.email.send("주문 완료")
        self.slack.post("주문 완료")
        self.audit_db.log("order_completed")
```

문제:
- 새 알림(SMS, 푸시)이 추가되면 `Order.complete`를 수정해야 함 (OCP 위반)
- Order가 "알림 대상"을 전부 알고 있어 SRP 위반
- 테스트할 때 Slack, Email 목 객체를 전부 준비해야 함

### 좋은 예: Observer 패턴

```python
from abc import ABC, abstractmethod

class OrderObserver(ABC):
    @abstractmethod
    def on_order_completed(self, order_id: str): ...

class Order:  # Subject
    def __init__(self):
        self._observers: list[OrderObserver] = []

    def attach(self, obs: OrderObserver):
        self._observers.append(obs)

    def detach(self, obs: OrderObserver):
        self._observers.remove(obs)

    def _notify(self, order_id: str):
        for obs in self._observers:
            obs.on_order_completed(order_id)

    def complete(self, order_id: str):
        self._save(order_id)
        self._notify(order_id)

# 구체 Observer들
class EmailObserver(OrderObserver):
    def on_order_completed(self, order_id):
        send_mail(order_id)

class SlackObserver(OrderObserver):
    def on_order_completed(self, order_id):
        post_slack(order_id)

# 조립
order = Order()
order.attach(EmailObserver())
order.attach(SlackObserver())
order.complete("O-1234")
```

새 알림 추가는 `Observer` 하나 만들고 `attach` 한 줄이면 끝. Order 코드는 변경 없음.

---

## AI 에이전트 개발 예시: LLM 스트리밍 토큰 옵저빙

에이전트에서 LLM 응답을 스트리밍(streaming, 토큰 단위로 생성되는 대로 하나씩 내려받는 방식)으로 받을 때 여러 곳에서 동시에 처리하고 싶은 경우가 많다. UI 렌더링, 토큰 카운팅, 비용 로깅, Langfuse(랭퓨즈, LLM 관측성 플랫폼) 트레이싱 등.

```python
from abc import ABC, abstractmethod

class StreamObserver(ABC):
    @abstractmethod
    def on_token(self, token: str): ...
    @abstractmethod
    def on_complete(self, full_text: str): ...

class UIRenderer(StreamObserver):
    def on_token(self, token):
        print(token, end="", flush=True)
    def on_complete(self, full_text):
        print("\n[완료]")

class TokenCounter(StreamObserver):
    def __init__(self):
        self.count = 0
    def on_token(self, token):
        self.count += 1
    def on_complete(self, full_text):
        log_cost(self.count)

class LangfuseTracer(StreamObserver):
    def __init__(self, trace):
        self.trace = trace
        self.buffer = []
    def on_token(self, token):
        self.buffer.append(token)
    def on_complete(self, full_text):
        self.trace.end(output=full_text)

class StreamingLLM:  # Subject
    def __init__(self):
        self._observers: list[StreamObserver] = []

    def subscribe(self, obs):
        self._observers.append(obs)

    def stream(self, messages):
        buf = []
        for token in openai_stream(messages):
            buf.append(token)
            for obs in self._observers:
                obs.on_token(token)
        full = "".join(buf)
        for obs in self._observers:
            obs.on_complete(full)
        return full

# 조립
llm = StreamingLLM()
llm.subscribe(UIRenderer())
llm.subscribe(TokenCounter())
llm.subscribe(LangfuseTracer(trace))
llm.stream(messages)
```

이 구조를 쓰면 "토큰이 들어올 때마다 해야 할 일"을 한 군데에 엮지 않고, 관심사별로 Observer를 따로 두어 관리한다.

### 에이전트 이벤트 버스

더 큰 에이전트 시스템에서는 이벤트 종류별로 다양하게 발행된다.

| 이벤트                  | 발행 시점           | 전형적 Observer         |
| -------------------- | --------------- | -------------------- |
| `tool_called`        | Tool 호출 시작      | 로거, 비용 추적기           |
| `tool_result`        | Tool 결과 수신      | 메모리 저장소, UI          |
| `llm_token`          | 토큰 스트리밍 중       | UI 렌더러, 안전 필터        |
| `agent_finished`     | Agent 실행 완료     | Slack 알림, DB 기록      |
| `error`              | 예외 발생           | Sentry, 알람           |

---

## 언제 쓰면 좋은가

1. **한 사건에 반응해야 할 대상이 여러 개**일 때
2. **주체가 구독자 수를 알 필요 없이** 느슨하게 연결하고 싶을 때
3. **런타임에 구독자가 늘거나 줄어야** 할 때
4. **이벤트 기반 UI**, 게임 루프, GUI 프레임워크, 에이전트 워크플로 등

## 트레이드오프

| 장점                 | 단점                                        |
| ------------------ | ----------------------------------------- |
| Subject와 Observer 분리 | 디버깅 시 "누가 이걸 호출했는지" 추적이 어려움 (암묵적 흐름)      |
| 런타임 구독 가능          | Observer가 많아지면 순서 의존/성능 문제 발생 가능          |
| 새 Observer 추가에 기존 코드 수정 없음 | 메모리 누수 위험 (detach 안 하면 Observer가 GC 안 됨) |
| 테스트가 쉬움(Subject 단독 테스트 가능) | 동기/비동기 섞이면 예외 전파 정책이 까다로워짐                |

> 알림 순서가 중요하거나 Observer 간 의존성이 있다면 Observer보다 Mediator 패턴이 맞을 수 있음.

---

## 실전 주의점

### 1. 순환 참조 주의
Observer가 Subject를 가리키고 Subject가 Observer를 리스트로 들고 있으면 순환 참조가 됨. Python은 GC가 처리하지만, 명시적으로 `detach`를 해주는 게 안전함.

### 2. 예외 전파 정책
한 Observer에서 예외가 나도 다른 Observer는 실행되어야 하는지 결정해야 함.

```python
def _notify(self, data):
    for obs in self._observers:
        try:
            obs.on_event(data)
        except Exception as e:
            logger.error(f"Observer {obs} failed: {e}")
            # 계속 진행
```

### 3. 동기 vs 비동기
Observer가 오래 걸리는 작업(네트워크 호출)을 한다면 Subject가 막힘. 비동기 큐(`asyncio`, `Celery`)를 사이에 둘지 검토할 것.

---

## 다른 패턴과의 관계 (포함 관계)

- **Pub-Sub의 소규모 객체지향판**: Observer는 보통 같은 프로세스 안, Pub-Sub은 분산 환경
- **[[SOLID원칙|OCP]] 실현 수단**: 새 Observer 추가가 Subject 수정 없이 가능
- **Mediator와 구분**: Mediator는 "모든 객체가 하나의 중재자를 통해 대화", Observer는 "1:N 브로드캐스트"
- **[[Chain of Responsibility 패턴|Chain of Responsibility]]와 구분**: CoR은 "한 명이 처리하면 멈춤", Observer는 "등록된 모두에게 전달"
- **이벤트 드리븐 아키텍처의 기본 벽돌**: 큰 시스템에서는 이벤트 버스(Event Bus) 형태로 확장됨

---

## 직접 확인하기

자기 코드에서 "A가 바뀌면 B, C, D를 호출"하는 구조를 찾아봐라. 호출 대상이 앞으로 늘어날 가능성이 있고, 관심사가 제각각(로깅, 알림, UI 업데이트)이라면 Observer 패턴 후보임.

Python 표준 라이브러리 `logging` 모듈의 Handler 구조, `blinker` 패키지, FastAPI의 이벤트 훅 등이 모두 Observer 패턴의 변형이다.

---

## 요약

> **"한쪽이 바뀌었을 때 관심 있는 모두에게 자동으로 알려라. 주체는 자기 구독자가 누구인지 몰라도 된다."**

관련 문서:
- [[SOLID원칙]]
- [[Chain of Responsibility 패턴]]
- [[Strategy 패턴]]
