---
생성날짜:
- 2026-04-21 13:00
마지막수정날짜:
- 2026-04-21-화요일 13:00
tags:
- 설계원칙
- 디자인패턴
- 생성패턴
- AI에이전트
별칭:
- Builder
- 빌더 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# Builder 패턴

**"복잡한 객체를 단계별로 조립해서 만드는 생성 패턴."**

Builder 패턴(빌더 패턴, 복합 객체 단계별 생성 규약)은 GoF(Gang of Four, 디자인 패턴 원전 4인 저자)의 생성(Creational) 패턴 중 하나다. 생성자 인자가 너무 많거나 선택 필드가 많을 때 `new Xxx(a, b, c, d, e, f, g, ...)` 같은 텔레스코핑(telescoping, 길어지는 망원경) 생성자를 피하고, 체인 방식으로 필요한 필드만 쌓아 올리게 해준다.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 생성(Creational) 패턴 |
| 목적 | 복잡한 객체의 생성 과정과 표현을 분리 |
| 핵심 질문 | "이 객체, 인자 10개짜리 생성자로 만들고 있나?" |
| 대안 | 팩토리(Factory), 정적 생성 메서드(Static Factory Method) |
| 트레이드오프 | 클래스 하나 늘어남. 대신 가독성과 유연성은 크게 개선 |

> **한마디 요약**: "선택지 많고 조합이 복잡하면 단계별로 쌓아라."

---

## 언제 쓰는가

다음 신호가 보이면 Builder 도입을 고려한다.

1. 생성자 인자가 5개 이상, 그중 절반 이상이 선택적(Optional)
2. 같은 클래스인데 다른 조합의 인스턴스가 많이 필요함
3. 생성 도중 검증(Validation)이 여러 단계에 걸쳐 필요함
4. 불변 객체(Immutable Object, 한 번 만들면 내부 상태가 바뀌지 않는 객체)를 만들어야 하는데 필드가 많음

### 일상 비유
햄버거 주문을 생각하면 된다. "빅맥 주세요" 한 마디면 끝이 아니라, 수제 버거 집에서는 `빵 선택 → 패티 굽기 정도 → 치즈 → 토마토 → 양파 → 소스 → 사이드`를 하나씩 고른다. 중간에 양파는 빼고 소스는 두 배로 같은 커스터마이징을 자연스럽게 받을 수 있게 한 것이 Builder다.

---

## 구조

Builder 패턴의 구성 요소는 3개(+1)다.

| 역할 | 설명 |
| --- | --- |
| Product | 최종 결과물. 생성자가 복잡한 대상 |
| Builder | 단계별 메서드(`with_x`, `add_y`) 제공. 마지막에 `build()`로 Product 반환 |
| Director (선택) | 특정 조립 레시피를 외부에 숨김. "스탠다드 버거 만드는 법" 같은 프리셋 |
| Client | Builder를 들고 메서드 체인으로 원하는 Product 완성 |

단계 분해:
1. **1단계**: Client가 Builder 인스턴스 생성
2. **2단계**: `with_llm()`, `add_tool()` 같은 단계 메서드를 원하는 만큼 호출(각 메서드는 `self` 반환 → 체인 가능)
3. **3단계**: `build()` 호출 시 내부 검증 후 Product 인스턴스 반환

---

## AI 에이전트 개발 예시: Agent 조립

실제 AI 에이전트 개발에서는 LLM, Tool 목록, Memory, System Prompt, 안전 설정 등 조합이 많다. 일반 생성자로 받으면 금방 뚱뚱해진다.

### 나쁜 예: 텔레스코핑 생성자

```python
class Agent:
    def __init__(
        self,
        llm,
        tools=None,
        memory=None,
        system_prompt=None,
        temperature=0.7,
        max_iterations=10,
        safety_filter=None,
        cost_tracker=None,
        trace_callback=None,
        retry_policy=None,
    ):
        ...

agent = Agent(
    OpenAILLM(),
    [SearchTool(), CalcTool()],
    BufferMemory(),
    "You are helpful",
    0.3,
    15,
    None,
    CostTracker(),
    None,
    RetryPolicy(3),
)
# 어느 위치가 뭐였더라?
```

### 좋은 예: Builder 도입

```python
class AgentBuilder:
    def __init__(self):
        self._llm = None
        self._tools = []
        self._memory = None
        self._system_prompt = None
        self._temperature = 0.7
        self._max_iterations = 10

    def with_llm(self, llm):
        self._llm = llm
        return self

    def add_tool(self, tool):
        self._tools.append(tool)
        return self

    def with_memory(self, memory):
        self._memory = memory
        return self

    def with_prompt(self, prompt: str):
        self._system_prompt = prompt
        return self

    def with_temperature(self, t: float):
        if not 0 <= t <= 2:
            raise ValueError("temperature must be in [0, 2]")
        self._temperature = t
        return self

    def build(self) -> "Agent":
        if self._llm is None:
            raise ValueError("LLM is required")
        if self._system_prompt is None:
            self._system_prompt = "You are a helpful assistant."
        return Agent(
            llm=self._llm,
            tools=self._tools,
            memory=self._memory,
            system_prompt=self._system_prompt,
            temperature=self._temperature,
            max_iterations=self._max_iterations,
        )

# 클라이언트
agent = (
    AgentBuilder()
    .with_llm(OpenAILLM(model="gpt-4o"))
    .add_tool(SearchTool())
    .add_tool(CalcTool())
    .with_memory(BufferMemory(max_turns=20))
    .with_prompt("You are a research assistant.")
    .with_temperature(0.3)
    .build()
)
```

필요한 것만 지정, 읽기도 편함. `build()` 시점에 필수 필드 검증까지 몰아서 한다.

### Director로 프리셋 만들기

같은 Builder로 "연구용 에이전트"와 "고객응대 에이전트"를 자주 만들어야 한다면 Director를 둔다.

```python
class AgentPresets:
    @staticmethod
    def research_agent(llm) -> AgentBuilder:
        return (AgentBuilder()
                .with_llm(llm)
                .add_tool(SearchTool())
                .add_tool(ArxivTool())
                .with_temperature(0.2)
                .with_prompt("You are a rigorous researcher."))

    @staticmethod
    def support_agent(llm) -> AgentBuilder:
        return (AgentBuilder()
                .with_llm(llm)
                .add_tool(KnowledgeBaseTool())
                .with_temperature(0.7)
                .with_prompt("You are a friendly support agent."))

# 프리셋에서 파생해서 커스터마이즈
agent = AgentPresets.research_agent(llm).add_tool(CalcTool()).build()
```

Director는 "기본 레시피"를 숨기고, Builder는 "마무리 커스터마이징"을 열어두는 조합이다.

---

## 실제 라이브러리에서의 Builder

| 라이브러리 | Builder 사례 |
| --- | --- |
| LangChain | `ChatPromptTemplate.from_messages([...])` + `.partial()` 체인 |
| OpenAI SDK | Assistant API의 `tools=[...]`, `tool_resources={...}` 누적 구성 |
| SQLAlchemy | `select(User).where(...).order_by(...).limit(10)` 쿼리 빌더 |
| Pydantic | `model_validate()` 전 단계 상태 구성 (간접 활용) |
| Rust | `std::process::Command::new("ls").arg("-l").arg("/tmp").spawn()` |

위 라이브러리 모두 "체인 메서드 → 최종 호출에서 결과물 반환" 구조를 따른다.

---

## 직접 확인해 보기

자기 코드에서 인자 5개 이상 받는 생성자가 있는 파일을 찾아본다.

```bash
# 파이썬 기준
grep -rn "def __init__" src/ | awk -F',' '{ if (NF > 5) print $0 }'
```

나오는 후보들이 Builder 도입 대상이다.

---

## 왜 이렇게 하는가

Builder가 해결하는 문제와 대안 대비 장점:

- **가독성**: 인자 순서 외울 필요 없음. 메서드 이름이 곧 문서
- **불변성(Immutability) 확보**: Product 자체는 `frozen=True`/불변으로 두고, 변경 가능한 상태는 Builder가 전담
- **점진적 검증**: 각 `with_x` 메서드에서 개별 검증, `build()`에서 종합 검증 가능
- **조합 폭발 제어**: 선택 필드 2^n 조합을 생성자 오버로딩으로 다 풀지 않아도 됨

트레이드오프:
- 클래스가 Product + Builder 로 최소 2개로 늘어남
- 단순한 객체에 Builder를 붙이면 오버엔지니어링(Over-engineering, 과잉설계). 필드 3개 이하 객체에는 굳이 쓸 필요 없음

---

## 다른 생성 패턴과의 비교

| 패턴 | 언제 쓰나 | 생성 단계 |
| --- | --- | --- |
| Builder | 필드 많고 조립 과정이 의미 있음 | 여러 단계 |
| Factory Method | 어떤 구체 클래스를 만들지 결정이 핵심 | 한 번 호출 |
| Abstract Factory | 관련된 객체군을 한꺼번에 생성 | 한 번 호출 |
| Prototype | 기존 인스턴스를 복제해서 변형 | 복제 + 수정 |

AI 에이전트 개발에서 가장 자주 쓰는 조합은 `Factory + Builder`다. Factory로 "어떤 LLM Provider"를 뽑고, Builder로 "그 LLM으로 어떤 Agent"를 조립한다.

---

## 한마디 요약

> **"생성자 인자가 망원경처럼 길어진다 싶으면, 단계별 메서드 체인으로 쌓아 올리는 Builder를 도입하라."**

관련 문서:
- [[Adapter 패턴]]
- [[Facade 패턴]]
- [[SOLID원칙]]
