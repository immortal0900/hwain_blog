---
생성날짜:
- 2026-04-21 12:10
마지막수정날짜:
- 2026-04-21-화요일 12:10
tags:
- 설계원칙
- 디자인패턴
- GoF
- 행위패턴
- AI에이전트
별칭:
- Strategy
- 전략 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# Strategy 패턴이란?

Strategy(스트래티지, 전략 패턴, 알고리즘을 객체로 감싸 런타임에 갈아끼우게 만드는 행위 디자인 패턴)는 **"같은 문제를 푸는 여러 방법을 서로 다른 클래스로 분리해두고, 호출 시점에 원하는 방법을 골라 끼우는"** 패턴이다. GoF(Gang of Four, 1994년 Erich Gamma 등 4인이 정리한 23개 디자인 패턴 모음집) 분류상 **행위(Behavioral) 패턴**에 속한다.

## 한눈에 보기

| 항목     | 내용                                                    |
| ------ | ----------------------------------------------------- |
| 분류     | 행위 패턴(Behavioral)                                     |
| 한 줄 정의 | 알고리즘군(family of algorithms)을 캡슐화해 상호 교체 가능하게 만듦       |
| 핵심 구성  | Context(사용자), Strategy(추상 인터페이스), ConcreteStrategy(구현) |
| 해결 문제  | `if/elif` 타입 분기로 알고리즘 선택, 새 방법 추가 시 기존 코드 수정          |
| 관련 원칙  | OCP(확장 개방 수정 폐쇄), DIP(추상화 의존)                         |

> **한마디 요약**: "if문으로 고르던 걸, 객체로 주입해서 고르게 바꾼 것."

---

## 일상 비유

길찾기 앱(네이버 지도, 카카오맵)을 떠올려보면 된다. 출발지와 도착지는 그대로인데 "최단 거리", "대중교통", "도보", "자동차" 버튼을 눌러 경로 계산 방식만 바꿀 수 있다. 앱 본체(Context)는 그대로 있고, 경로 계산 알고리즘(Strategy)만 교체되는 구조임.

---

## 구조

```
┌──────────────┐   strategy    ┌────────────────┐
│   Context    │ ────────────▶ │  <<interface>> │
│              │               │    Strategy    │
│ execute()─┐  │               │  + run(data)   │
└──────────┼──┘               └────────┬───────┘
           │                            │ implements
           ▼                            ▼
      self.strategy.run()   ┌──────────────────────┐
                            │  ConcreteStrategyA   │
                            │  ConcreteStrategyB   │
                            │  ConcreteStrategyC   │
                            └──────────────────────┘
```

- **Context**: 알고리즘을 사용하는 주체. Strategy 인터페이스만 알고, 구체 구현은 모름
- **Strategy**: 모든 알고리즘이 지켜야 할 공통 약속(메서드 시그니처)
- **ConcreteStrategy**: 실제 알고리즘 구현체. 자유롭게 추가 가능

---

## 나쁜 예 vs 좋은 예 (Python)

### 나쁜 예: 타입 분기로 전략 선택

```python
class RouteFinder:
    def find(self, start, end, mode):
        if mode == "car":
            # 복잡한 자동차 경로 계산 로직
            return calc_car_route(start, end)
        elif mode == "bike":
            return calc_bike_route(start, end)
        elif mode == "walk":
            return calc_walk_route(start, end)
        else:
            raise ValueError(f"Unknown mode: {mode}")
```

문제:
- 새 운송수단(전동킥보드)을 추가하려면 `RouteFinder` 본체를 열어 분기를 추가해야 함 (OCP 위반)
- `find` 메서드 안이 계속 비대해짐
- 각 알고리즘을 독립적으로 테스트하기 불편함

### 좋은 예: Strategy 패턴

```python
from abc import ABC, abstractmethod

class RouteStrategy(ABC):
    @abstractmethod
    def calculate(self, start, end) -> list[tuple]: ...

class CarStrategy(RouteStrategy):
    def calculate(self, start, end):
        return calc_car_route(start, end)

class BikeStrategy(RouteStrategy):
    def calculate(self, start, end):
        return calc_bike_route(start, end)

class WalkStrategy(RouteStrategy):
    def calculate(self, start, end):
        return calc_walk_route(start, end)

class RouteFinder:  # Context
    def __init__(self, strategy: RouteStrategy):
        self.strategy = strategy

    def find(self, start, end):
        return self.strategy.calculate(start, end)

# 사용
finder = RouteFinder(CarStrategy())
finder.find(seoul, busan)

finder.strategy = BikeStrategy()  # 런타임 교체 가능
```

---

## AI 에이전트 개발 예시: 프롬프트 전략 교체

LLM 에이전트에서 "같은 질문에 대해 프롬프트 구성 방법을 여러 개 두고 골라 쓰고 싶은 경우"가 자주 생긴다. Zero-shot, Few-shot, Chain-of-Thought(CoT, 단계별 사고 유도 기법) 같은 것들임.

```python
from abc import ABC, abstractmethod

class PromptStrategy(ABC):
    @abstractmethod
    def build(self, question: str) -> list[dict]: ...

class ZeroShotStrategy(PromptStrategy):
    def build(self, question):
        return [{"role": "user", "content": question}]

class FewShotStrategy(PromptStrategy):
    def __init__(self, examples: list[tuple[str, str]]):
        self.examples = examples

    def build(self, question):
        msgs = []
        for q, a in self.examples:
            msgs.append({"role": "user", "content": q})
            msgs.append({"role": "assistant", "content": a})
        msgs.append({"role": "user", "content": question})
        return msgs

class CoTStrategy(PromptStrategy):
    def build(self, question):
        prefix = "단계별로 생각한 뒤 최종 답을 내주세요.\n\n"
        return [{"role": "user", "content": prefix + question}]

class Agent:
    def __init__(self, llm, strategy: PromptStrategy):
        self.llm = llm
        self.strategy = strategy

    def ask(self, question: str) -> str:
        messages = self.strategy.build(question)
        return self.llm.chat(messages)

# 사용
agent = Agent(llm, ZeroShotStrategy())
agent.strategy = CoTStrategy()  # 어려운 추론 문제면 교체
```

같은 원리로 LLM 라우팅 전략(저렴한 모델 vs 고성능 모델 선택), 샘플링 전략(greedy, top-p, temperature 변화), 청킹 전략(고정 크기 vs 문단 단위 vs 의미 기반) 등에 모두 적용 가능하다.

---

## 언제 쓰면 좋은가

1. **같은 작업의 변형(variant)이 3개 이상**일 때
2. **런타임에 알고리즘을 바꿔야** 할 때 (A/B 테스트, 설정 기반 분기)
3. **타입 분기문이 자라는 게 보일 때** (`if mode == ...` 체인)
4. 각 알고리즘을 **독립적으로 테스트**하고 싶을 때

## 트레이드오프

| 장점                 | 단점                                |
| ------------------ | --------------------------------- |
| 새 알고리즘 추가가 수정 없이 가능 | 전략 수가 적으면 과한 추상화가 됨               |
| 런타임 교체 가능          | Context가 어떤 Strategy를 고를지 알아야 함   |
| 알고리즘 단위 테스트 쉬움     | 전략마다 필요한 입력이 다르면 인터페이스 설계가 까다로워짐 |

> 2개밖에 없는 분기라면 차라리 함수 매개변수 하나로 두는 게 나을 수 있다. 과도한 패턴 적용을 경계할 것.

---

## 함수형 스타일 대안

Python 같은 동적 언어에서는 클래스 없이 **함수 자체를 Strategy로** 넘길 수도 있음.

```python
def zero_shot(q): return [{"role": "user", "content": q}]
def cot(q): return [{"role": "user", "content": f"단계별로 생각하세요.\n\n{q}"}]

class Agent:
    def __init__(self, llm, build_prompt):
        self.llm = llm
        self.build = build_prompt

    def ask(self, q):
        return self.llm.chat(self.build(q))

agent = Agent(llm, cot)
```

상태를 가진 전략(예: Few-shot의 예제 목록)이 필요하면 클래스로, 순수 함수로 충분하면 함수로 가는 식이 실무적임.

---

## 다른 패턴과의 관계 (포함 관계)

- **[[SOLID원칙|OCP]]의 대표적 구현 수단**: Strategy를 쓰면 새 알고리즘 추가가 "수정 없이 확장"이 된다
- **[[SOLID원칙|DIP]]와 쌍을 이룸**: Context가 추상 Strategy에 의존하고 구체 구현은 주입받음
- **[[Factory 패턴|Factory]]와 자주 함께 쓰임**: 어떤 Strategy를 만들지 결정하는 책임을 Factory가 맡음
- **[[Decorator 패턴|Decorator]]와 구분**: Strategy는 "알고리즘 자체를 교체", Decorator는 "알고리즘에 기능을 덧붙임"
- **State 패턴과 구조적 쌍둥이**: 구현은 거의 같지만 의도가 다름. State는 내부 상태에 따라 자동 전환, Strategy는 외부에서 명시적으로 선택

---

## 직접 확인하기

자기 코드에서 `if mode ==` 또는 `switch(type)` 패턴을 `grep`으로 뽑아봐라. 같은 변수로 3번 이상 분기하고 있다면 Strategy 후보임. 다음과 같이 리팩토링해볼 것.

1단계: 공통 메서드 시그니처를 뽑아 추상 인터페이스 만들기
2단계: 각 분기를 클래스로 이동
3단계: Context에 전략 주입
4단계: `if` 체인 제거

---

## 요약

> **"알고리즘을 객체로 만들어 플러그처럼 뺐다 꽂았다 하는 패턴. `if/elif` 대신 클래스로 분리하라."**

관련 문서:
- [[SOLID원칙]]
- [[Factory 패턴]]
- [[Decorator 패턴]]
