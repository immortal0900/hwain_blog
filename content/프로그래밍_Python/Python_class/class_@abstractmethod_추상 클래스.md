---
생성날짜:
- 2026-01-26 23:42
마지막수정날짜:
- 2026-01-26-월요일 23:42
tags:
- python
- class
- abstract_class
- ABC
- 디자인패턴
- AI_Agent
별칭:
- Abstract Base Class
- 추상 클래스
- 인터페이스
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python
Project:
---

# 추상 클래스(Abstract Class)와 `@abstractmethod`

## 한마디 요약

**추상 클래스(Abstract Class, 메서드 목록만 갖는 미완성 클래스)**: 직접 인스턴스화 불가능한 "설계 계약서". 상속받은 자식 클래스가 지정된 메서드를 **반드시 구현**하도록 강제하는 파이썬의 **인터페이스 메커니즘**.

## 포함 관계

**`abc` 모듈 ⊃ `ABC`/`ABCMeta` ⊃ `@abstractmethod` 데코레이터**

| 개념 | 역할 | 필수 여부 |
|---|---|---|
| `abc`(abstract base class) 모듈 | 추상 클래스 기능 제공 | 필수 import |
| `ABC` 클래스 또는 `ABCMeta` 메타클래스 | "이 클래스는 추상" 표식 | 둘 중 하나 필수 |
| `@abstractmethod` 데코레이터 | "이 메서드는 구현 강제" 표식 | 추상 메서드마다 필요 |

## 일상 비유

**건축 설계도**: 집을 지으려면 "침실 1개, 화장실 1개 필수" 같은 **최소 요구사항 명세**가 있다. 추상 클래스는 이 명세. 실제 집(자식 클래스)은 침실/화장실을 반드시 만들어야 준공 허가(인스턴스화)가 난다.

파이썬은 추상 클래스(abstract class)라는 기능을 제공한다. 추상 클래스는 **메서드의 목록만 가진 클래스**이며 **상속받는 클래스에서 메서드 구현을 강제하기 위해 사용**한다.

## 질문1: abc가 뭐지?

**`abc`(Abstract Base Class, 추상 기본 클래스) 모듈**은 파이썬 표준 라이브러리. 추상 클래스를 만들려면 **import로 abc 모듈을 가져와야 한다**.

```python
from abc import ABCMeta, abstractmethod

class StudentBase(metaclass=ABCMeta):
    @abstractmethod
    def study(self):
        pass

    @abstractmethod
    def go_to_school(self):
        pass

class Student(StudentBase):  # 상속받고 go_to_school()을 구현 안 함
    def study(self):
        print('공부하기')

james = Student()
james.study()
```

### 실행결과

```
Traceback (most recent call last):
  File "C:\project\class_abc_error.py", line 16, in <module>
    james = Student()
TypeError: Can't instantiate abstract class Student with abstract methods go_to_school
```

실행하면 에러 발생. 추상 클래스 `StudentBase`에서 추상 메서드로 `study`와 `go_to_school`을 정의했는데, `Student`는 `study`만 구현하고 `go_to_school`은 안 구현해서 에러가 난다.

따라서 **추상 클래스를 상속받았다면 `@abstractmethod`가 붙은 추상 메서드를 모두 구현해야 한다**.

## 현대적 문법: `metaclass=ABCMeta` 대신 `ABC` 상속

**Python 3.4부터는 더 간단한 문법 제공**:

```python
from abc import ABC, abstractmethod

# 현대적 방식 (권장)
class StudentBase(ABC):
    @abstractmethod
    def study(self):
        pass

# 구버전 방식 (호환용, 동작은 동일)
class StudentBase(metaclass=ABCMeta):
    @abstractmethod
    def study(self):
        pass
```

| 비교 관점 | `metaclass=ABCMeta` | `ABC` 상속 |
|---|---|---|
| 문법 | 메타클래스 지정 | 일반 상속 |
| Python 버전 | 3.0+ | 3.4+ |
| 다중 상속 조합 | 까다로움 | 자연스러움 |
| 가독성 | 약함 | 강함 |
| 현대 코드베이스 | 드묾 | 표준 |

## 질문2: `@abstractmethod`는 뭐지?

**핵심**: "이 메서드는 **반드시 구현해야 합니다**"라고 강제하는 장치
상속받아놓고 구현 안 하면 ==인스턴스화 시점에 에러 발생==

```python
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        pass  # 구현 안 함, 껍데기만
```

### `@abstractmethod` 효과

```python
# 1. PaymentStrategy를 직접 인스턴스화 시도
strategy = PaymentStrategy()  # TypeError 발생!
# "추상 클래스는 직접 객체 만들 수 없음"

# 2. process()를 구현 안 한 채 상속
class BadPayment(PaymentStrategy):
    pass  # process() 구현 안 함

bad = BadPayment()  # TypeError 발생!
# "추상 메서드 process()를 구현하지 않음"

# 3. process()를 제대로 구현
class GoodPayment(PaymentStrategy):
    def process(self, amount):  # 구현함
        return f"결제: {amount}원"

good = GoodPayment()  # OK 실행됨
```

## 단계 분해: 추상 클래스가 작동하는 순서

1. **1단계**: `class PaymentStrategy(ABC):` 선언 시 파이썬이 `ABCMeta`를 통해 이 클래스를 "추상"으로 표시
2. **2단계**: `@abstractmethod` 달린 메서드마다 "미구현" 플래그 설정
3. **3단계**: 자식 클래스(`GoodPayment`)가 상속받으면 파이썬이 **모든 추상 메서드가 오버라이드됐는지 검사**
4. **4단계**: 하나라도 안 됐으면 `GoodPayment()` 호출 순간 `TypeError` 발생
5. **5단계**: 전부 구현했으면 정상적으로 [[class_인스턴스]] 생성

## 왜 이렇게 설계됐나?

**설계 의도**: 파이썬은 자바처럼 `interface` 키워드가 없다. 대신 **덕 타이핑(Duck Typing, "오리처럼 생기고 오리처럼 울면 오리")**이 기본. 하지만 큰 프로젝트에서 "이 메서드는 꼭 구현해야 해"라는 계약이 필요할 때가 있다. 그래서 `abc` 모듈을 제공.

**대안 대비 장점**:
- 컴파일 시점이 아니어도 **인스턴스화 시점에** 즉시 오류(지연 감지보다 빠름)
- IDE가 "이 메서드 구현 안 했음" 자동 표시
- 팀 협업 시 "이 인터페이스 따르세요"라는 강한 시그널

**트레이드오프**: 런타임 오버헤드 약간(무시할 수준). 문법이 살짝 복잡.

## AI Agent 실무 예시: LangChain `BaseTool`

**LangChain은 추상 클래스로 모든 툴의 공통 인터페이스를 정의**한다. 직접 `BaseTool`을 상속해 커스텀 툴을 만드는 게 표준 패턴.

```python
from abc import ABC, abstractmethod
from langchain_core.tools import BaseTool
# BaseTool 자체가 내부에서 abstractmethod를 쓰고 있음

class RealEstateSearchTool(BaseTool):
    name: str = "real_estate_search"
    description: str = "지역 부동산 시세를 검색합니다"

    def _run(self, query: str) -> str:
        # 실제 검색 로직
        return f"{query} 시세: 평당 X만원"

    async def _arun(self, query: str) -> str:
        # 비동기 버전
        return f"{query} 시세: 평당 X만원"

tool = RealEstateSearchTool()
result = tool.invoke({"query": "강남 아파트"})
```

**`BaseTool`이 강제하는 것**: `_run`, `_arun`, `name`, `description`. 구현 안 하면 인스턴스화 시 에러. 이 덕분에 LangGraph는 어떤 툴이든 같은 인터페이스(`invoke`, `batch`, `stream`)로 호출 가능.

### 직접 설계: LLM 백엔드 교체 가능한 구조

```python
from abc import ABC, abstractmethod

class LLMBackend(ABC):
    """LLM 공급사를 추상화"""

    @abstractmethod
    def generate(self, prompt: str, max_tokens: int = 1000) -> str:
        """텍스트 생성"""
        pass

    @abstractmethod
    def embed(self, text: str) -> list[float]:
        """임베딩 생성"""
        pass

class OpenAIBackend(LLMBackend):
    def __init__(self, api_key: str):
        from openai import OpenAI
        self.client = OpenAI(api_key=api_key)

    def generate(self, prompt: str, max_tokens: int = 1000) -> str:
        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}],
            max_tokens=max_tokens,
        )
        return response.choices[0].message.content

    def embed(self, text: str) -> list[float]:
        response = self.client.embeddings.create(
            model="text-embedding-3-small",
            input=text,
        )
        return response.data[0].embedding

class AnthropicBackend(LLMBackend):
    def __init__(self, api_key: str):
        from anthropic import Anthropic
        self.client = Anthropic(api_key=api_key)

    def generate(self, prompt: str, max_tokens: int = 1000) -> str:
        response = self.client.messages.create(
            model="claude-opus-4-7",
            max_tokens=max_tokens,
            messages=[{"role": "user", "content": prompt}],
        )
        return response.content[0].text

    def embed(self, text: str) -> list[float]:
        # Anthropic은 자체 임베딩 없음, Voyage 등 별도 필요
        raise NotImplementedError("Anthropic 임베딩은 Voyage AI 사용")

# Agent는 LLMBackend 타입으로만 받음
class Agent:
    def __init__(self, backend: LLMBackend):
        self.backend = backend

    def answer(self, question: str) -> str:
        return self.backend.generate(question)

# 백엔드 교체 자유
agent_openai = Agent(OpenAIBackend(api_key="..."))
agent_anthropic = Agent(AnthropicBackend(api_key="..."))
```

**여기서 얻는 것**: Agent 코드는 OpenAI든 Anthropic이든 몰라도 됨. 테스트 시 `MockBackend`를 주입해서 LLM 호출 없이 로직만 검증 가능.

## 클래스 속성도 추상화 가능

**Python 3.3+에서 `@abstractmethod`를 `@property`와 조합해 속성도 강제**.

```python
from abc import ABC, abstractmethod

class Tool(ABC):
    @property
    @abstractmethod
    def name(self) -> str:
        """툴 이름"""
        pass

class SearchTool(Tool):
    @property
    def name(self) -> str:
        return "search"

# 속성 구현 안 하면?
class BadTool(Tool):
    pass

BadTool()  # TypeError: Can't instantiate abstract class BadTool with abstract methods name
```

## 실무 주의점

### 1. `@abstractmethod`에 구현 넣어도 됨(공통 로직)

**오해 주의**: `pass`만 써야 한다고 생각하기 쉬운데, **실제 코드를 넣고 자식이 `super()`로 호출 가능**.

```python
class BaseAgent(ABC):
    @abstractmethod
    def run(self, query: str) -> str:
        # 공통 로깅 로직
        print(f"[BaseAgent] 실행: {query}")
        # 자식이 super().run(query) 호출해서 이 로깅 활용 가능

class MyAgent(BaseAgent):
    def run(self, query: str) -> str:
        super().run(query)  # 부모 로깅 실행
        return f"처리 결과: {query}"
```

### 2. 인터페이스만 원한다면 `typing.Protocol` 고려

**Python 3.8+에서는 `Protocol`로 덕 타이핑 기반 "구조적 서브타입"이 가능**. `ABC`보다 유연함.

```python
from typing import Protocol

class SupportsRun(Protocol):
    def run(self, query: str) -> str: ...

def execute(tool: SupportsRun, query: str) -> str:
    return tool.run(query)  # run 메서드만 있으면 OK (상속 불필요!)
```

| 비교 관점 | `ABC` | `Protocol` |
|---|---|---|
| 상속 필요 | 필수 | 불필요 |
| 런타임 검증 | 강함(인스턴스화 시 체크) | 약함(mypy 정적 검증만) |
| 외부 라이브러리 클래스 | 상속해야 적용 | 상속 없이 적용 가능 |
| 용도 | 계약을 강하게 강제 | 덕 타이핑 타입화 |

### 3. 추상 클래스는 `@dataclass`와 조합 가능

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class BaseAgent(ABC):
    name: str
    max_iter: int = 5

    @abstractmethod
    def run(self, query: str) -> str: ...

@dataclass
class SearchAgent(BaseAgent):
    def run(self, query: str) -> str:
        return f"{self.name}이 {query} 검색"

agent = SearchAgent(name="서치봇", max_iter=10)
```

## 정리

- **추상 클래스** = 메서드 목록만 가진 미완성 클래스, 직접 인스턴스화 불가
- **`@abstractmethod`** = "자식이 반드시 구현해야 함" 강제
- **현대 문법**: `ABC` 상속(권장), `metaclass=ABCMeta`(레거시)
- **AI Agent 실무**: LangChain `BaseTool`, 백엔드 추상화, 전략 패턴(Strategy Pattern)
- 인터페이스만 원한다면 `Protocol`도 고려

## 관련 문서

- [[class_오버라이딩]]: 추상 메서드를 자식이 구현하는 행위
- [[class_상속시`super()` 사용]]: 추상 클래스의 공통 로직을 자식이 `super()`로 호출
- [[class_타입힌트]]: 추상 클래스를 타입으로 명시
- [[class_파라미터]]: Strategy 패턴에서 추상 클래스를 파라미터 타입으로
- [[SOLID원칙]]: DIP(의존성 역전 원칙)의 구현 도구
