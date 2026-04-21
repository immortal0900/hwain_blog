---
생성날짜:
- 2026-04-21 15:33
마지막수정날짜:
- 2026-04-21-화요일 15:33
tags:
- 설계원칙
- 디자인패턴
- Mediator
- GoF
- AI에이전트
- MultiAgent
별칭:
- Mediator
- 중재자 패턴
- 미디에이터
type:
- 자료수집
Area/Reasource:
Project:
---
# Mediator 패턴 (중재자)

Mediator(미디에이터, 중재자)는 서로 얽힌 객체들이 **직접 참조하지 않도록**, 가운데에 중재자 객체 하나를 두고 그를 통해서만 소통하게 만드는 행위 패턴(Behavioral Pattern)이다. 객체 N개가 서로 물려 있으면 연결선이 N x (N-1) / 2 개 생기지만, Mediator를 끼우면 N개로 줄어든다.

> GoF 23패턴 중 행위 패턴. "객체끼리 직접 말 걸지 말고, 관제탑을 거치게 하라"가 핵심.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 행위 패턴(Behavioral Pattern) |
| 핵심 아이디어 | 객체 간 상호작용을 중재자 하나로 모음 |
| 풀려는 문제 | N:N 직접 참조로 결합도 폭발 |
| 결합 방향 | 객체 to 객체 대신, 객체 to Mediator |
| 대표 실생활 예 | 관제탑, 카톡 단체방 서버, 이벤트 버스 |

## 일상 비유

공항 관제탑이 전형적 Mediator다. 비행기끼리 무전기로 "저쪽 활주로 좀 비켜줘" 하면 사고난다. 비행기들은 관제탑에만 말한다. 관제탑이 누구를 어느 활주로로, 몇 시에 띄울지 결정해서 각 비행기에 지시를 내린다. 비행기 수가 늘어도 관제탑만 똑똑하면 된다.

## 구조

```
  직접 결합 (문제)              Mediator 적용 (해결)

  A ──┬──┬── B                  A       B
  │   X  X   │                   \     /
  C ──┴──┴── D                    Mediator
                                 /     \
                                C       D
```

- **Mediator** 인터페이스: 공통 통신 규약을 정의(`notify(sender, event)` 같은 메서드)
- **ConcreteMediator**: 실제 조율 로직을 담음(누가 누구에게 무엇을 전달할지)
- **Colleague(동료 객체)**: Mediator 참조만 들고, 서로는 모름

## AI 에이전트 개발 예시 1: Multi-Agent Orchestrator

AI Agent(에이전트, 스스로 도구를 써가며 목표를 달성하는 LLM 기반 프로그램)를 여러 개 엮는 멀티에이전트 시스템에서 Mediator가 빛난다. Researcher, Writer, Reviewer 에이전트가 서로 직접 호출하면 의존이 엉킴.

```python
from abc import ABC, abstractmethod

class AgentMediator(ABC):
    @abstractmethod
    def notify(self, sender: "Agent", event: str, payload: dict): ...

class Agent(ABC):
    def __init__(self, mediator: AgentMediator, name: str):
        self.mediator = mediator
        self.name = name

    @abstractmethod
    def handle(self, event: str, payload: dict): ...

class Researcher(Agent):
    def handle(self, event, payload):
        if event == "research":
            facts = llm_search(payload["topic"])
            self.mediator.notify(self, "research_done", {"facts": facts})

class Writer(Agent):
    def handle(self, event, payload):
        if event == "research_done":
            draft = llm_draft(payload["facts"])
            self.mediator.notify(self, "draft_done", {"draft": draft})

class Reviewer(Agent):
    def handle(self, event, payload):
        if event == "draft_done":
            score = llm_review(payload["draft"])
            self.mediator.notify(self, "review_done", {"score": score})

class BlogOrchestrator(AgentMediator):
    def __init__(self):
        self.researcher = Researcher(self, "researcher")
        self.writer = Writer(self, "writer")
        self.reviewer = Reviewer(self, "reviewer")

    def notify(self, sender, event, payload):
        # 중재자가 누구에게 넘길지 결정
        if event == "research_done":
            self.writer.handle(event, payload)
        elif event == "draft_done":
            self.reviewer.handle(event, payload)
        elif event == "review_done":
            print("done:", payload)

    def start(self, topic: str):
        self.researcher.handle("research", {"topic": topic})

BlogOrchestrator().start("디자인 패턴")
```

Researcher는 Writer가 존재하는지도 모른다. Writer를 2명으로 늘리거나 Editor를 중간에 끼워도 Orchestrator(Mediator)만 고치면 됨. [[SOLID원칙]]의 OCP, SRP를 매우 깔끔하게 실현.

## AI 에이전트 개발 예시 2: Tool 조정자

에이전트가 여러 Tool을 조합해 복합 질문에 답할 때, Tool끼리 서로 "너 이거 다음에 해" 같은 결합이 생기면 안 된다. Mediator가 Tool 간 실행 순서를 잡아 준다.

```python
class ToolMediator:
    def __init__(self, search, calc, memory):
        self.search = search
        self.calc = calc
        self.memory = memory

    def answer(self, question: str):
        hits = self.search.run(question)              # 1단계: 검색
        summary = llm_summarize(hits)
        if needs_math(summary):
            summary = self.calc.run(summary)          # 2단계: 수치 계산
        self.memory.save(question, summary)           # 3단계: 기억
        return summary
```

`search`, `calc`, `memory`는 서로를 전혀 모름. `ToolMediator`만 사라지거나 바뀌어도 각 Tool은 영향 없음.

## AI 에이전트 개발 예시 3: 이벤트 버스(Event Bus) 변형

Mediator의 느슨한 버전이 Event Bus다. 중재자가 "누가 뭘 원하는지"를 명시적으로 알지 않고, 이벤트 타입 기반 구독만 관리함. Observer 패턴과 섞인 형태.

```python
class EventBus:
    def __init__(self):
        self._subs: dict[str, list] = {}

    def subscribe(self, event: str, handler):
        self._subs.setdefault(event, []).append(handler)

    def publish(self, event: str, payload: dict):
        for h in self._subs.get(event, []):
            h(payload)

bus = EventBus()
bus.subscribe("user_message", lambda p: researcher.handle("research", p))
bus.subscribe("research_done", lambda p: writer.handle("research_done", p))
bus.publish("user_message", {"topic": "디자인 패턴"})
```

완전한 Mediator는 중재 로직을 들고, Event Bus는 단순 라우팅만 함. 프로젝트 복잡도에 따라 선택.

## 왜 이렇게 하는가

1. **N:N 결합 폭발 해결**: Agent/Module 수가 늘면 직접 참조 방식은 유지보수 지옥. Mediator를 두면 연결선이 선형으로 늘어남.
2. **정책을 한곳에 집중**: "Researcher 끝나면 Writer" 같은 흐름 정책이 Mediator 한 파일에 모이고, 여기만 보면 전체 흐름을 파악 가능.
3. **동료 객체 재사용 쉬움**: Researcher는 Mediator 인터페이스만 알면 다른 프로젝트에서도 재사용.

### 대안 대비 트레이드오프

| 접근 | 장점 | 단점 |
| --- | --- | --- |
| 객체끼리 직접 호출 | 초반 짧은 코드 | N 커지면 유지보수 불가 |
| Mediator | 결합 낮음, 정책 한곳 | Mediator가 비대해질 위험(God Object) |
| Event Bus(Pub/Sub) | 극도로 느슨함 | 흐름 추적 어려움, 디버깅 난이도 |

God Object(신 객체, 너무 많은 책임을 떠안은 거대 객체) 위험이 Mediator 패턴의 주요 함정. 중재자가 뚱뚱해지면 내부를 다시 서브 Mediator로 나누든, Event Bus로 일부 흐름을 위임해야 함.

## Mediator vs 다른 패턴 헷갈림 정리

| 패턴 | 주 목적 | 차이 |
| --- | --- | --- |
| Mediator | 다대다 통신 중재 | 중재자가 흐름 정책을 들고 있음 |
| Observer | 상태 변화 알림 | 구독자는 특정 이벤트만 받고 싶음 |
| Facade | 복잡한 하위 시스템 간단 인터페이스 | 일방향, 하위 시스템은 Facade 모름 |
| Proxy | 본체 대리 | 1대1, 같은 인터페이스 |

## 직접 확인

- LangGraph의 Graph/Supervisor 노드가 Mediator 역할을 한다. 노드들은 그래프가 어디로 보낼지 결정하는 정책에만 의존하고, 서로를 직접 부르지 않음.
- CrewAI의 Crew 객체도 Agent들을 중재하는 Mediator의 구체 구현.

## 포함 관계 및 관련 패턴

- Mediator 내부에서 요청을 일단 큐에 쌓는 구조라면 [[Command 패턴(요청의 객체화)]]이 같이 등장.
- 중재자로 들어오는 메시지 흐름을 시간순으로 훑으려면 [[Iterator 패턴(순회)]]이 유용.
- Mediator가 하위 구성 요소를 인터페이스로만 참조해야 [[SOLID원칙]]의 DIP(의존성 역전)가 성립. 즉 DIP가 전제되는 패턴.

## 한마디 요약

> **"모두가 서로에게 전화 걸면 난장판이다. 관제탑 하나 세우고 모두 거기로만 말하게 하라. 관제탑이 뚱뚱해지거든 다시 쪼개라."**

관련 문서:
- [[SOLID원칙]]
- [[Command 패턴(요청의 객체화)]]
- [[Iterator 패턴(순회)]]
- [[Proxy 패턴(대리 처리)]]
