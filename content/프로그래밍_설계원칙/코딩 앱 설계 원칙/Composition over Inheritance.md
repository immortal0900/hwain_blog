---
tags:
  - 설계원칙
  - 코딩원칙
  - AI_Agent
  - LangChain
  - LangGraph
  - OOP
생성날짜: 2026-04-21
마지막수정날짜: 2026-04-21
상위문서: "[[코딩 앱 설계 원칙_index]]"
---

# Composition over Inheritance (조합 > 상속)

> "A is a B" 대신 "A has a B"로 설계. 기능 확장을 상속 트리(class hierarchy)로 풀지 않고, 작은 부품을 끼워 맞춰 처리함.

LangChain의 LCEL(LangChain Expression Language, Runnable을 파이프로 이어 붙이는 표현식)이나 LangGraph의 StateGraph(노드와 엣지로 구성한 실행 그래프) 전체가 이 원칙 위에 세워져 있음. 이 철학을 이해 못 하면 프레임워크 코드가 왜 저런 모양인지 끝내 납득이 안 감.

---

## 1. 두 전략이 뭐가 다른가

**핵심 질문: "기능을 어디서 가져올 것인가?"**

- **Inheritance (상속, 부모 클래스의 속성·메서드를 자식이 물려받는 방식)**: 부모 class에 정의된 동작을 `class Child(Parent)` 선언 한 줄로 끌어온다. "is-a" 관계.
- **Composition (조합, 여러 독립 객체를 필드로 가지고 위임해서 기능을 얻는 방식)**: 필요한 기능을 객체로 주입받아 내부에 저장하고 호출. "has-a" 관계.

### 나란히 비교표

| 관점 | 상속 (Inheritance) | 조합 (Composition) |
|------|-------------------|-------------------|
| 관계 | A is a B (로봇청소기는 청소기다) | A has a B (로봇청소기는 모터를 가진다) |
| 결합도 | 컴파일 타임 고정 (Tight Coupling, 강한 결합) | 런타임 교체 가능 (Loose Coupling, 느슨한 결합) |
| 재사용 단위 | 클래스 계층 전체 | 객체 하나 |
| 변경 비용 | 부모 수정이 자식 전부에 파급 | 주입만 바꾸면 끝 |
| 다형성 구현 | `override` | 의존성 주입 (Dependency Injection, 외부에서 필요한 객체를 넣어주는 기법) |
| 대표 문제 | Fragile Base Class, Diamond Problem | 객체 수 증가, 초기 설계 비용 |

> 일상 비유: 상속은 "유전자를 물려받는 가족 관계"임. 아버지가 바뀌면 자식 형질이 전부 바뀜. 조합은 "부품을 사서 조립하는 레고"임. 바퀴가 마음에 안 들면 바퀴만 교체.

---

## 2. 왜 상속이 위험한가

**상속이 낳는 고질병 3가지.**

### 2-1. Fragile Base Class Problem (깨지기 쉬운 부모 클래스)
부모 class의 메서드 하나 수정했는데 자식 수십 개가 동시에 터지는 현상. 부모를 만진 사람은 의도가 없어도, 자식들의 숨은 가정(assumption)을 깨뜨릴 수 있음.

### 2-2. Diamond Problem (다이아몬드 문제)
`class D(B, C)`인데 `B`와 `C`가 둘 다 `A`를 상속하는 상황. `D`가 호출한 메서드를 누가 제공한 건지 모호해짐. Python은 MRO(Method Resolution Order, 메서드 해결 순서)로 억지로 해결하지만 복잡해짐.

### 2-3. 런타임 교체 불가
`Robot(Vehicle)`로 선언한 순간 "이 로봇은 Vehicle이다"가 코드에 박힘. 실행 중에 Drone으로 바꾸고 싶어도 못 바꿈. 반면 `Robot(vehicle=Car())`이면 `robot.vehicle = Drone()` 한 줄로 교체됨.

---

## 3. AI Agent 개발에서의 조합 패턴

**Agent 프레임워크는 전부 조합 기반임.** `class MyAgent(BaseAgent)` 같은 상속 스타일이 거의 안 보이는 이유.

### 3-1. Agent = 부품 조립

```
Agent = LLM + Tool List + Memory + Prompt + State Schema
```

각 부품이 독립 객체로 존재하고, Agent는 그걸 "가진다(has-a)". LLM을 `claude-sonnet-4-6`에서 `gpt-5`로 바꿔도 Tool, Memory 코드는 한 줄도 안 건드림.

### 3-2. LangChain LCEL: 파이프로 Runnable 이어붙이기

LCEL에선 모든 구성요소가 `Runnable`(동일 인터페이스 `.invoke()`, `.stream()`, `.batch()`를 구현한 실행 단위)임. `|` 연산자로 왼쪽 출력을 오른쪽 입력으로 넘겨 파이프라인을 구성함.

```python
from langchain_core.runnables import RunnableGenerator

pipeline = (
    RunnableGenerator(stt_stream)      # 음성 → 텍스트
    | RunnableGenerator(agent_stream)  # 텍스트 → Agent 응답
    | RunnableGenerator(tts_stream)    # Agent 응답 → 음성
)
```

(출처: LangChain 공식문서 voice-agent 페이지)

**포함 관계**: `pipeline`은 세 Runnable을 조합한 `RunnableSequence`(연속 실행용 Runnable). 이 Sequence 자체도 Runnable이라 다른 Runnable과 또 조합 가능. 재귀적 조합.

### 3-3. LangGraph StateGraph: 노드와 엣지로 워크플로우 조립

```python
from langgraph.graph import StateGraph, START, END

agent_builder = StateGraph(MessagesState)

agent_builder.add_node("llm_call", llm_call)        # 노드 1: LLM 호출
agent_builder.add_node("tool_node", tool_node)       # 노드 2: 툴 실행

agent_builder.add_edge(START, "llm_call")
agent_builder.add_conditional_edges(
    "llm_call",
    should_continue,
    ["tool_node", END],
)
agent_builder.add_edge("tool_node", "llm_call")

agent = agent_builder.compile()
```

(출처: LangGraph 공식문서 quickstart/workflows-agents 페이지)

**포함 관계**: `StateGraph`는 `Node`(함수 또는 Runnable)와 `Edge`(전이 규칙)를 담는 컨테이너. `compile()` 결과물은 그 자체로 `Runnable`임. 그러니까 `agent_builder.compile()`을 또 다른 StateGraph의 노드로 집어넣을 수 있음. Supergraph 안에 Subgraph 넣기.

### 3-4. Subgraph 조합: 도메인별 전문 Agent 합치기

```python
def create_sub_agent(model, *, name, **kwargs):
    agent = create_agent(model=model, name=name, **kwargs)
    return (
        StateGraph(MessagesState)
        .add_node(name, agent)
        .add_edge("__start__", name)
        .compile()
    )

fruit_agent = create_sub_agent("gpt-4.1-mini", name="fruit_agent", tools=[fruit_info], ...)
veggie_agent = create_sub_agent("gpt-4.1-mini", name="veggie_agent", tools=[veggie_info], ...)
```

(출처: LangGraph 공식문서 use-subgraphs 페이지)

과일 전문가 Agent + 채소 전문가 Agent를 상위 Supervisor Agent가 "가진다". 각 sub-agent는 독립 네임스페이스와 독립 체크포인트(checkpoint, 대화 상태 스냅샷)를 유지함. 상속으로는 이런 구조 못 만듦.

---

## 4. 단계 분해: 상속을 조합으로 바꾸는 과정

**상속 지옥에 빠진 예제를 조합으로 리팩토링.**

### 1단계: 상속 스타일 (안티패턴)

```python
class BaseAgent:
    def call_llm(self, prompt): ...
    def save_memory(self, msg): ...

class ClaudeAgent(BaseAgent):
    def call_llm(self, prompt):  # Claude용으로 override
        ...

class ClaudeAgentWithRedis(ClaudeAgent):
    def save_memory(self, msg):  # Redis용으로 override
        ...

class ClaudeAgentWithRedisAndTools(ClaudeAgentWithRedis):
    ...
```

→ 조합 폭발(combinatorial explosion). `{모델} x {메모리} x {툴}` 경우의 수마다 class를 새로 만들어야 함.

### 2단계: 조합 스타일로 변환

```python
class Agent:
    def __init__(self, llm, memory, tools):
        self.llm = llm          # has-a LLM
        self.memory = memory    # has-a Memory
        self.tools = tools      # has-a Tools

    def run(self, user_input):
        self.memory.save(user_input)
        response = self.llm.invoke(user_input, tools=self.tools)
        self.memory.save(response)
        return response

agent = Agent(
    llm=ChatAnthropic(model="claude-sonnet-4-6"),
    memory=RedisMemory(),
    tools=[search_tool, calc_tool],
)
```

→ class 하나로 끝남. 부품만 바꾸면 다른 조합 완성.

### 3단계: 런타임 교체

```python
agent.llm = ChatOpenAI(model="gpt-5")       # 모델 교체
agent.memory = PostgresMemory()              # 메모리 교체
```

실행 중에 교체 가능. 상속 구조에선 class 자체를 새로 만들어야 됨.

---

## 5. 트레이드오프

**조합이 만능은 아님.** 알고 써야 됨.

| 항목 | 조합의 단점 | 대응 |
|------|-----------|------|
| 객체 수 증가 | 작은 객체가 많아져 추적이 어려워짐 | DI 컨테이너(Dependency Injection Container, 객체 생성·조립을 중앙에서 관리하는 라이브러리) 사용 |
| 인터페이스 설계 부담 | 교체 가능하게 만들려면 공통 Protocol 필요 | `typing.Protocol`, 추상 base class 활용 |
| 보일러플레이트 | 위임 메서드(`self.llm.invoke()` 등)가 반복됨 | `__getattr__` 위임, Runnable 같은 표준 인터페이스 |
| 성능 | 간접 호출 한 단계 더 | 거의 무시 가능 (LLM 호출 비용이 수천 배 큼) |

**언제 상속이 그래도 나은가**: 프레임워크가 요구하는 base class가 있을 때(`class MyModel(pydantic.BaseModel)`), 진짜로 타입 계층이 자연스러울 때(`class Square(Shape)`). 그 외엔 조합이 기본값.

---

## 6. 직접 확인해 보기

**설치한 LangChain 폴더를 뒤져서 조합 흔적을 직접 보면 감이 옴.**

```bash
# LangChain 설치 경로 찾기
python -c "import langchain_core; print(langchain_core.__file__)"
# → .../site-packages/langchain_core/__init__.py

# Runnable base 정의 확인
python -c "from langchain_core.runnables import RunnableSequence; help(RunnableSequence.__init__)"
```

볼 포인트:
1. `RunnableSequence.__init__`가 `first: Runnable`, `middle: list[Runnable]`, `last: Runnable`을 필드로 받음 → 완벽한 has-a 구조
2. `__or__` 매직 메서드가 `|` 연산자를 오버로딩해 새 `RunnableSequence`를 만들어 반환 → 파이프라인이 불변(immutable)
3. 어떤 Runnable도 다른 Runnable을 상속하지 않음. 전부 동일 Protocol을 구현한 평평한 구조.

---

## 7. 왜 하필 AI Agent에서 조합이 필수인가

**설계 의도 해석.**

1. **모델 교체 빈도가 높음**: Claude → GPT → Gemini 전환이 주 단위로 일어남. 상속으로 묶어두면 교체할 때마다 class 수정.
2. **툴 조합이 케이스마다 다름**: 고객 A는 검색+DB, 고객 B는 계산기+파일시스템. 런타임 주입이 자연스러움.
3. **그래프로 사고하는 게 자연스러움**: Agent 실행 흐름이 "노드 → 노드 → 조건 분기"임. 이걸 상속 트리로 표현하면 왜곡됨.
4. **테스트 용이성**: Mock LLM을 주입해 테스트하려면 조합 구조가 필수. 상속이면 전체 class를 Mock class로 바꿔야 함.

---

## 8. 한마디 요약

> "has-a"로 부품을 끼우면 런타임에 갈아 끼울 수 있음. LangChain의 `|`, LangGraph의 `add_node`는 전부 이 원칙의 문법적 표현. 상속은 진짜 타입 계층일 때만.

---

## 관련 문서

- [[코딩 앱 설계 원칙_index]] (상위 인덱스)
- [[SOLID 원칙]] 특히 LSP(Liskov Substitution Principle)와 DIP(Dependency Inversion Principle)가 조합을 뒷받침함
- [[디자인 패턴]] Strategy, Decorator, Adapter 전부 조합 기반
- [[Dependency Injection]] 조합을 실행하는 대표 기법
- [[DRY]] 조합 부품을 재사용해 반복 제거

## 참고 자료

- LangGraph 공식문서 Quickstart: https://docs.langchain.com/oss/python/langgraph/quickstart
- LangGraph Workflows and Agents: https://docs.langchain.com/oss/python/langgraph/workflows-agents
- LangGraph Subgraphs: https://docs.langchain.com/oss/python/langgraph/use-subgraphs
- LangChain Voice Agent (LCEL 조합 예): https://docs.langchain.com/oss/python/langchain/voice-agent
- 원전: Gang of Four, "Design Patterns", 1994, "Favor object composition over class inheritance" 원칙
