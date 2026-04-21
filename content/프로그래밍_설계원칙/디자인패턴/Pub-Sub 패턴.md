---
생성날짜:
- 2026-04-21 13:25
마지막수정날짜:
- 2026-04-21-화요일 13:25
tags:
- 설계원칙
- 디자인패턴
- 행동패턴
- 아키텍처패턴
- AI에이전트
- 비동기
별칭:
- Pub/Sub
- Pub-Sub
- Publish/Subscribe
- 발행-구독
- Event Bus
type:
- 자료수집
Area/Reasource:
Project:
---
# Pub/Sub 패턴

**"발행자(Publisher)와 구독자(Subscriber)가 서로를 모른 채, 브로커(Broker, 중개자)를 통해 주제(Topic, 채널)별 메시지를 비동기(Asynchronous, 호출자가 결과를 기다리지 않고 계속 진행)로 주고받는 패턴."**

Pub/Sub 패턴(펍섭, 발행-구독 패턴)은 GoF의 Observer(관찰자) 패턴을 분산/비동기로 확장한 형태다. Observer가 "한 객체가 자기 구독자를 직접 알고 호출"하는 것이었다면, Pub/Sub는 "중간에 브로커를 두고 서로 몰라도 되게" 만든 구조다. 메시징 미들웨어(Kafka, Redis Pub/Sub, RabbitMQ, AWS SNS, GCP Pub/Sub)의 핵심 모델이자, 프로세스 내부 이벤트 버스의 기본형이기도 하다.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 행동(Behavioral) 패턴 / 메시징 아키텍처 패턴 |
| 목적 | 송신과 수신 사이 결합도 제거, 비동기 처리 |
| 핵심 질문 | "한 이벤트에 여러 후속 동작이 동시에 필요한가? 서로 모르게 하고 싶은가?" |
| 대안 | Observer(동기, 인프로세스), Request/Response, Message Queue(점대점) |
| 트레이드오프 | 디버깅과 순서 보장이 까다로움. 대신 확장성은 탁월 |

> **한마디 요약**: "신호를 쏘면 관심 있는 놈이 각자 알아서 듣는다."

---

## 일상 비유: 유튜브 구독

크리에이터가 영상을 업로드(publish)한다. 유튜브라는 플랫폼(broker)이 그 영상을 구독자 목록(subscriptions)에 따라 알림으로 뿌린다. 크리에이터는 구독자가 누군지 모르고, 구독자는 크리에이터에게 직접 연락하지 않는다. 구독 취소도 크리에이터 몰래 일어난다.

라디오 방송도 똑같다. 방송국(publisher)이 특정 주파수(topic)로 송출하면, 그 주파수에 맞춘 라디오(subscriber) 아무거나 듣는다. 방송국은 청취자 수나 이름을 모른다.

코드 세계에서는 이 "중간 플랫폼" 역할을 브로커(Broker) 또는 이벤트 버스(Event Bus)가 맡는다.

---

## Observer와 Pub/Sub의 차이

둘 다 "한 이벤트에 여러 반응"을 엮는다. 하지만 다음이 다르다.

| 항목 | Observer | Pub/Sub |
| --- | --- | --- |
| 중개자 | 없음 (Subject가 구독자 목록 직접 관리) | 있음 (Broker / Event Bus) |
| 결합도 | Subject가 Observer를 참조 | Publisher와 Subscriber 완전 분리 |
| 실행 | 동기, 같은 프로세스 | 보통 비동기, 다른 프로세스/서버도 가능 |
| 필터링 | 없음 (모두에게 통지) | 토픽/라우팅 키로 필터 |
| 배달 보장 | 호출로 끝 | 정책에 따라 at-most-once / at-least-once / exactly-once |

AI 에이전트 개발에서는 인프로세스 이벤트 버스를 "Pub/Sub 스타일로 쓴다"고 말하기도 한다. 둘 사이 경계가 실무에서는 흐릿하다.

---

## 구조

| 역할 | 설명 |
| --- | --- |
| Publisher | 특정 Topic에 메시지를 발행 |
| Topic (채널) | 메시지 분류 키. `llm.token.stream`, `user.created` 등 |
| Broker | Topic별 구독자 목록 관리, 메시지 라우팅, 버퍼링 |
| Subscriber | Topic을 구독하고 메시지 수신 시 처리 |
| Message | 직렬화된 페이로드(payload, 실어 나르는 데이터 본문) + 메타데이터 |

단계 분해:
1. **1단계**: Subscriber가 Broker에 `subscribe(topic, handler)` 등록
2. **2단계**: Publisher가 Broker에 `publish(topic, message)` 호출
3. **3단계**: Broker가 해당 Topic 구독자들의 handler를 비동기 실행 (큐에 적재 후 워커가 소비)
4. **4단계**: Subscriber가 수신 후 ACK(Acknowledgement, 수신 확인) 전송. 실패하면 재시도 또는 Dead Letter Queue(DLQ, 실패 메시지 격리 큐)로 이동

---

## AI 에이전트 개발 예시 1: LLM 토큰 스트리밍

LLM 응답 토큰을 받는 순간 여러 곳에서 동시에 듣고 싶다.

- UI: WebSocket으로 사용자 화면에 즉시 출력
- Logger: 파일/DB에 전체 응답 기록
- Cost Tracker: 토큰 수 누적 계산
- Moderation: 유해성 검출
- Trace: Langfuse/OpenTelemetry로 전송

Publisher가 이 모든 수신자를 직접 알고 호출하면 결합이 지옥. Pub/Sub가 답이다.

### 간단한 인프로세스 Event Bus

```python
from collections import defaultdict
from dataclasses import dataclass
import asyncio
from typing import Callable, Awaitable

@dataclass
class Event:
    topic: str
    payload: dict


class EventBus:
    def __init__(self):
        self._subs: dict[str, list[Callable[[Event], Awaitable[None]]]] = defaultdict(list)

    def subscribe(self, topic: str, handler):
        self._subs[topic].append(handler)

    async def publish(self, event: Event):
        handlers = self._subs.get(event.topic, [])
        # 구독자들을 병렬로 실행. 한쪽이 실패해도 다른 쪽은 진행
        await asyncio.gather(*(h(event) for h in handlers), return_exceptions=True)
```

### Publisher: LLM Streaming Loop

```python
async def stream_llm(bus: EventBus, prompt: str):
    async for chunk in openai_stream(prompt):
        await bus.publish(Event(
            topic="llm.token",
            payload={"text": chunk.text, "model": "gpt-4o"},
        ))
    await bus.publish(Event(topic="llm.done", payload={}))
```

### Subscribers

```python
async def ui_handler(event):
    await websocket.send_text(event.payload["text"])

async def logger_handler(event):
    await db.log(event.payload["text"])

async def cost_handler(event):
    cost_tracker.add_tokens(count_tokens(event.payload["text"]))

async def moderation_handler(event):
    if is_unsafe(event.payload["text"]):
        await bus.publish(Event("llm.unsafe", event.payload))


bus = EventBus()
bus.subscribe("llm.token", ui_handler)
bus.subscribe("llm.token", logger_handler)
bus.subscribe("llm.token", cost_handler)
bus.subscribe("llm.token", moderation_handler)

await stream_llm(bus, "Hello")
```

새 수신자(예: 실시간 요약기)가 필요하면 `bus.subscribe("llm.token", summarizer_handler)` 한 줄 추가. Publisher 코드는 한 글자도 바뀌지 않는다.

---

## AI 에이전트 개발 예시 2: 멀티 에이전트 협업

여러 에이전트가 `agent.message`, `tool.result`, `plan.updated` 같은 토픽으로 소통하면, 에이전트를 추가/제거해도 다른 에이전트 코드를 건드리지 않아도 된다. Coordinator 에이전트, Researcher, Critic, Writer가 각자 다른 토픽을 구독/발행하는 구조는 오픈소스 멀티에이전트 프레임워크(AutoGen, CrewAI, LangGraph)의 기본 설계다.

### 토픽 설계 예시

| Topic | Publisher | Subscriber |
| --- | --- | --- |
| `task.assigned` | Coordinator | Researcher |
| `research.completed` | Researcher | Writer, Critic |
| `draft.created` | Writer | Critic |
| `critic.feedback` | Critic | Writer, Coordinator |
| `task.done` | Coordinator | UI, Logger |

---

## 실제 시스템에서의 Pub/Sub

| 시스템 | 특징 |
| --- | --- |
| Apache Kafka | 분산 로그 기반. 파티셔닝, 리플레이 가능, 스루풋 극강 |
| Redis Pub/Sub | 인메모리, 빠름, 버퍼링 없음(구독자가 놓치면 사라짐) |
| RabbitMQ | AMQP 기반, 다양한 라우팅(fanout, topic, direct) |
| AWS SNS + SQS | SNS로 팬아웃, SQS로 수신자별 큐 |
| GCP Pub/Sub | 서버리스 Pub/Sub, 자동 스케일 |
| LangChain Callbacks | 인프로세스 Pub/Sub의 사실상 구현. on_llm_start, on_tool_end 등 |

선택 기준:
- 단일 프로세스 내부에서만 필요: 인프로세스 EventBus
- 같은 서버 내 여러 프로세스: Redis Pub/Sub
- 메시지 손실 불가 + 리플레이 필요: Kafka
- 복잡한 라우팅: RabbitMQ
- 완전 관리형: AWS SNS+SQS, GCP Pub/Sub

---

## 전달 보장(Delivery Guarantee)

Pub/Sub를 쓰려면 반드시 이해해야 하는 개념.

| 모드 | 의미 | 예시 |
| --- | --- | --- |
| At-most-once | 잘해야 한 번. 누락 가능 | UDP, Redis Pub/Sub |
| At-least-once | 최소 한 번. 중복 가능 | Kafka 기본, SQS |
| Exactly-once | 정확히 한 번. 어렵고 비쌈 | Kafka Transactions + idempotent producer |

중복 수신이 문제라면 Subscriber를 **멱등(Idempotent, 같은 입력을 여러 번 줘도 결과가 같은)** 하게 설계한다. AI 에이전트라면 "같은 tool call id면 결과를 캐시하고 재실행하지 않음" 같은 방식.

---

## 직접 확인해 보기

현재 프로젝트에서 이벤트 또는 콜백을 찾는다.

```bash
grep -rn "callback" src/
grep -rn "on_token\|on_event\|dispatch" src/
```

한 이벤트를 처리하는 함수가 여러 비즈니스 관심사(UI + 로깅 + 과금)를 섞고 있다면 토픽 분리 신호다. 수신자별로 토픽을 나누고, Publisher는 이벤트만 쏘도록 리팩터링.

---

## 왜 이렇게 하는가

Pub/Sub의 설계 의도와 장점:

- **송수신 결합 제거**: Publisher는 누가 듣는지 몰라도 됨. 수신자 추가/제거 자유
- **비동기 확장성**: 느린 Subscriber가 빠른 Publisher를 막지 않음. 큐로 버퍼링
- **팬아웃(Fan-out) 자연스러움**: 한 이벤트 → 여러 처리. 분기 코드 없이
- **장애 격리**: 한 Subscriber가 죽어도 Publisher와 다른 Subscriber는 멀쩡
- **이벤트 소싱(Event Sourcing) 기반**: 토픽 로그가 시스템 상태의 감사 기록(audit trail)이 됨

트레이드오프:
- **디버깅 난이도**: 누가 어떤 이벤트 때문에 실행됐는지 추적이 어려움. 분산 트레이싱(Distributed Tracing, OpenTelemetry 등)이 필수
- **순서 보장**: 기본적으로 순서 없음. 필요하면 파티셔닝 키를 설계해야 함 (Kafka)
- **메시지 스키마 진화**: 버전 관리가 어려움. Schema Registry 같은 도구 필요
- **중복/누락 처리**: 멱등성을 수신자에 강제해야 함
- **테스트 복잡**: 비동기 흐름을 어떻게 단언(assert)할지 전략 필요

---

## 포함 관계와 다른 패턴

| 패턴 | 관계 |
| --- | --- |
| Observer | Pub/Sub의 인프로세스 동기 단순 버전 |
| Mediator | 객체들 사이 중재. Pub/Sub는 Mediator의 메시징 전용 일반화 |
| Chain of Responsibility | 체인으로 전달. Pub/Sub는 브로드캐스트 |
| Event Sourcing | Pub/Sub로 쌓인 이벤트 로그 자체를 상태의 원천으로 삼는 패턴 |
| CQRS(Command Query Responsibility Segregation) | 쓰기 측 결과를 이벤트로 흘려 읽기 측이 구독. Pub/Sub가 뼈대 |

- **Pub/Sub + State**: 상태 변화(state_changed) 이벤트를 발행해 여러 관심사가 관찰. [[State 패턴]] 참고
- **Pub/Sub + Facade**: Facade 내부 로직이 Pub/Sub로 느슨 결합되어 구성되는 경우도 흔함. [[Facade 패턴]] 참고

---

## 한마디 요약

> **"한 이벤트에 관심을 보이는 구성원이 둘 이상이면, 서로 모르는 채로 만나게 해줄 브로커를 세워라."**

관련 문서:
- [[State 패턴]]
- [[Facade 패턴]]
- [[SOLID원칙]]
