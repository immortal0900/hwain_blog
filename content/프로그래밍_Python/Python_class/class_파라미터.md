---
생성날짜:
- 2026-01-26 19:30
마지막수정날짜:
- 2026-01-26-월요일 19:30
tags:
- python
- class
- 파라미터
- parameter
- 의존성주입
- AI_Agent
별칭:
- 매개변수
- Parameter
- Argument
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python
Project:
---

# 클래스 파라미터와 의존성 주입

## 한눈에 보기

**클래스의 `__init__` 파라미터는 "**이 객체가 태어날 때 반드시 받아야 할 재료**" 목록**. 이걸 통해 외부에서 의존성(다른 객체)을 주입받는 패턴이 **DI(Dependency Injection, 의존성 주입)**다.

## 포함 관계

**함수 시그니처 ⊃ 파라미터(정의) → 호출 시 인자(Argument, 실제 값) 전달**

| 용어 | 영문 | 위치 | 예시 |
|---|---|---|---|
| 파라미터 | Parameter | 함수 정의 쪽 | `def __init__(self, payment_method):` |
| 인자 | Argument | 함수 호출 쪽 | `PaymentProcessor(credit_payment)` |
| 타입 힌트 | Type Hint | 파라미터 옆 `:` | `payment_method: PaymentStrategy` |
| 기본값 | Default Value | 파라미터 `=` 뒤 | `region: str = "서울"` |

**용어 정리**: 엄밀히는 정의 쪽이 "파라미터", 호출 쪽이 "인자"지만 실무에선 섞어 쓴다. 중요한 건 **값이 호출 쪽에서 정의 쪽으로 흘러들어간다**는 방향성.

## 전체 예시: OCP 준수한 Strategy 패턴

```python
# Good - OCP 준수 (Strategy 패턴)

from abc import ABC, abstractmethod


class PaymentStrategy(ABC):
    """결제 전략 인터페이스"""

    @abstractmethod
    def process(self, amount):
        pass


class CreditCardPayment(PaymentStrategy):
    """신용카드 결제"""

    def process(self, amount):
        # 신용카드 처리 로직
        return f"Credit card payment: ${amount}"


class PayPalPayment(PaymentStrategy):
    """PayPal 결제"""

    def process(self, amount):
        # PayPal 처리 로직
        return f"PayPal payment: ${amount}"


class BankTransferPayment(PaymentStrategy):
    """계좌이체 결제"""

    def process(self, amount):
        # 계좌이체 처리 로직
        return f"Bank transfer: ${amount}"


class PaymentProcessor:
    """결제 처리기 - 기존 코드 수정 없이 확장 가능"""

    def __init__(self, strategy: PaymentStrategy):
        self.strategy = strategy

    def process(self, amount):
        return self.strategy.process(amount)


# 사용
processor = PaymentProcessor(CreditCardPayment())
processor.process(100)


# 새로운 결제 수단 추가 시 기존 코드 수정 불필요!
class CryptoPayment(PaymentStrategy):
    def process(self, amount):
        return f"Crypto payment: ${amount}"
```

**여기서 일어나는 일**: `PaymentProcessor`는 자기가 어떤 결제 방식을 쓸지 **몰라도 된다**. `__init__` 파라미터로 전략 객체를 주입받아서, 그걸 그냥 호출만 한다. 결제 수단을 추가/교체하려면 `PaymentProcessor` 코드는 건드리지 않고 새 전략 클래스만 만들면 된다. 이게 **OCP(Open-Closed Principle, 개방-폐쇄 원칙)**의 실제 구현.

## 질문1: 오른쪽 인자가 파라미터로 들어가는가?

**정답: YES, 정확히 맞음**

```python
class PaymentProcessor:
    def __init__(self, payment_method: PaymentStrategy):
        self.payment_method = payment_method
```

```python
# 사용 시나리오
credit_payment = CreditCardPayment()  # 여기서 객체 생성
processor = PaymentProcessor(credit_payment)  # 이게 아래로 전달됨
```

### 내부 흐름 단계 분해

```python
def __init__(self, payment_method: PaymentStrategy):
    # 1. credit_payment 객체가 payment_method 파라미터로 들어옴 (오른쪽)
    # 2. 그걸 self.payment_method에 저장 (왼쪽)
    self.payment_method = payment_method
    #    ↑ 인스턴스 속성   ↑ 파라미터로 받은 객체
```

**실행 순서**:

1. `CreditCardPayment()` 객체 생성 → 메모리 어딘가에 존재(예: 주소 `0x1A2B`)
2. 그 객체 참조를 `PaymentProcessor(여기)` 괄호 안에 넣음
3. `__init__` 파라미터 `payment_method`가 그 객체 참조를 받음(같은 주소 `0x1A2B` 가리킴)
4. `self.payment_method = payment_method`로 인스턴스 속성에 저장([[class에서 self. 필요성]] 참고)
5. 이제 `processor.payment_method`로 언제든 접근 가능(주소 `0x1A2B`의 객체)

**일상 비유**: 택배 물류센터(클래스)에 "내용물"이라는 빈 상자 라벨(파라미터)이 있고, 발송자가 실제 물건(인자, `credit_payment`)을 넣으면, 라벨에 따라 `self.payment_method`라는 선반에 얹힌다.

### 직접 확인: 같은 객체를 가리키는지 검증

```python
credit = CreditCardPayment()
processor = PaymentProcessor(credit)

print(id(credit))                     # 예: 140234567891232
print(id(processor.payment_method))   # 동일! 같은 객체 참조
print(credit is processor.payment_method)  # True
```

## 파라미터의 4가지 종류

**파이썬 파라미터는 전달 방식에 따라 분류**된다.

```python
class Agent:
    def __init__(
        self,
        name,                    # 1. 위치 인자(positional, 필수)
        model="gpt-4o-mini",     # 2. 키워드 인자 + 기본값
        *tools,                  # 3. 가변 위치 인자(variadic positional)
        system_prompt=None,      # 4. 키워드 전용(keyword-only)
        **config                 # 5. 가변 키워드 인자(variadic keyword)
    ):
        self.name = name
        self.model = model
        self.tools = tools
        self.system_prompt = system_prompt
        self.config = config
```

### 호출 예시

```python
agent = Agent(
    "researcher",              # name (위치)
    "gpt-4o",                  # model (위치 또는 키워드)
    tool1, tool2, tool3,       # *tools로 묶임 → (tool1, tool2, tool3)
    system_prompt="...",       # 키워드
    temperature=0.7,           # **config로 묶임 → {"temperature": 0.7}
    max_tokens=1000,
)

print(agent.tools)    # (tool1, tool2, tool3)
print(agent.config)   # {"temperature": 0.7, "max_tokens": 1000}
```

**왜 이렇게 쪼개나**: AI Agent는 설정 옵션이 많아서 모든 걸 위치 인자로 받으면 호출부가 해독 불가능해진다. `**kwargs`로 "나머지 설정"을 한 번에 받는 패턴이 표준.

## 기본값과 타입 힌트

```python
class ResearchAgent:
    def __init__(
        self,
        llm,                                    # 필수
        region: str = "전국",                   # 기본값
        max_iterations: int = 5,                # 기본값 + 타입 힌트
        tools: list | None = None,              # None 기본값 패턴
    ):
        self.llm = llm
        self.region = region
        self.max_iterations = max_iterations
        self.tools = tools if tools is not None else []  # None 처리
```

**왜 `tools=None`으로 하고 안에서 `[]`를 넣나**: 가변 객체(list, dict)를 기본값으로 직접 쓰면 **모든 인스턴스가 같은 리스트를 공유하는 버그** 발생. [[class에서 self. 필요성]] 문서의 "실무 주의점" 참고.

## AI Agent 실무 예시: 의존성 주입 패턴

**LangChain/LangGraph Agent에서 파라미터로 받아야 하는 것들**:

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import BaseTool
from langgraph.checkpoint.memory import MemorySaver

class AgentExecutor:
    def __init__(
        self,
        llm: ChatOpenAI,                  # LLM 객체 주입
        tools: list[BaseTool],            # 툴 목록 주입
        checkpointer: MemorySaver | None = None,  # 체크포인터 주입
        system_prompt: str = "You are a helpful assistant.",
        max_iterations: int = 10,
    ):
        self.llm = llm
        self.tools = tools
        self.checkpointer = checkpointer or MemorySaver()
        self.system_prompt = system_prompt
        self.max_iterations = max_iterations

# 사용
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
search_tool = TavilySearch()
executor = AgentExecutor(
    llm=llm,
    tools=[search_tool],
    max_iterations=5,
)
```

**왜 이렇게 하나**:
1. **테스트 용이성**: 테스트 시 진짜 `ChatOpenAI` 대신 mock LLM을 주입 가능
2. **교체 유연성**: GPT 대신 Claude로 바꾸려면 `llm` 파라미터만 교체
3. **명시적 의존성**: 클래스 시그니처만 봐도 "이 Agent가 뭘 필요로 하는지" 한눈에 파악

## 파라미터 설계 원칙

**좋은 `__init__` 시그니처를 위한 체크리스트**:

| 원칙 | 설명 | 예시 |
|---|---|---|
| 필수는 위치, 선택은 키워드 | 꼭 필요한 건 앞, 옵션은 뒤에 `=기본값` | `(self, llm, temperature=0)` |
| 타입 힌트 필수 | IDE 자동완성 + 문서화 | `llm: ChatOpenAI` |
| 가변 객체 기본값 금지 | `[]`, `{}` 직접 쓰지 말고 `None` 후 내부에서 할당 | `tools=None` → `self.tools = tools or []` |
| 파라미터 3개 이상이면 키워드 강제 | `*` 사용해 키워드 전용으로 | `def __init__(self, llm, *, region, max_iter)` |
| 부울 파라미터는 키워드로 | `True/False`만 보면 의미 불명 | `Agent(verbose=True)` |

### 키워드 전용 파라미터 강제

```python
class Agent:
    def __init__(self, llm, *, region: str, max_iter: int = 5):
        #                 ↑ 이 뒤는 반드시 키워드로만
        self.llm = llm
        self.region = region

Agent(my_llm, "서울", 10)       # TypeError! 위치로 못 넘김
Agent(my_llm, region="서울", max_iter=10)  # OK
```

**왜 이렇게 하나**: 호출부 가독성을 높이고, 파라미터 순서 바꿔도 기존 호출부가 안 깨지게 하려고.

## 실무 주의점

### 1. 파라미터 이름 = 속성 이름 관례

```python
# 좋음: 일관성
def __init__(self, name, age):
    self.name = name
    self.age = age

# 나쁨: 이름 다르게 쓰면 추적 어려움
def __init__(self, n, a):
    self.name = n
    self.age = a
```

### 2. `__init__`에 너무 많은 파라미터

5~6개 넘으면 설정 객체(config object)로 묶는 걸 고려.

```python
# 전
class Agent:
    def __init__(self, llm, tools, region, max_iter, temp, top_p, verbose, checkpointer):
        ...

# 후: 설정 객체로 묶기
from dataclasses import dataclass

@dataclass
class AgentConfig:
    region: str = "전국"
    max_iter: int = 5
    temperature: float = 0.0
    top_p: float = 1.0
    verbose: bool = False

class Agent:
    def __init__(self, llm, tools, config: AgentConfig):
        self.llm = llm
        self.tools = tools
        self.config = config
```

## 정리

- 클래스 파라미터 = `__init__`에서 받는 "이 객체가 태어날 재료"
- 호출 쪽 인자 → 정의 쪽 파라미터로 값이 흐름(방향성 기억)
- `self.x = x` 패턴으로 [[class_인스턴스]] 속성에 저장
- AI Agent에선 **LLM, 툴, 설정**을 파라미터로 주입하는 DI 패턴이 표준
- 기본값에 가변 객체(`[]`, `{}`) 직접 쓰지 말기

## 관련 문서

- [[class_`__init__`]]: 파라미터를 받는 초기화 메서드
- [[class에서 self. 필요성]]: 파라미터를 `self.`에 저장하는 이유
- [[class_타입힌트]]: 파라미터 옆 `:` 로 타입 명시
- [[class_인스턴스]]: 파라미터로 만들어지는 결과물
