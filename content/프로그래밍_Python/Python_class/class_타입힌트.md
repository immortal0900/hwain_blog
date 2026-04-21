---
생성날짜:
- 2026-01-26 19:36
마지막수정날짜:
- 2026-01-26-월요일 19:35
tags:
- python
- class
- 타입힌트
- type_hint
- typing
- AI_Agent
별칭:
- Type Hint
- Type Annotation
- 타입 어노테이션
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python
Project:
---

# 타입 힌트(Type Hint) 완전 정리

## 한마디 요약

**타입 힌트(Type Hint, PEP 484)**: 변수/파라미터/반환값의 타입을 코드에 **주석처럼 적어두는 표기**. 실행엔 영향 없음(0 런타임 비용), 하지만 IDE 자동완성과 정적 분석기(mypy, pyright)가 버그를 미리 잡아준다.

## 질문: `: PaymentStrategy`는 왜 쓰나?

```python
def __init__(self, payment_method: PaymentStrategy):
    #                             ↑ 이 부분이 타입 힌트
    self.payment_method = payment_method
```

**역할**:
- "이 파라미터는 `PaymentStrategy` 타입이어야 합니다" 라고 **문서화**
- **런타임에는 아무 영향 없음**(파이썬은 실행 시 타입 체크 안 함)
- IDE(VS Code, Cursor, PyCharm)가 **자동완성/에러 검출**에 활용
- mypy/pyright 같은 **정적 분석기**가 타입 불일치 미리 경고

## 타입 힌트 있음 vs 없음

| 구분 | 없음 | 있음 |
|---|---|---|
| 실행 속도 | 동일 | 동일(런타임 영향 0) |
| IDE 자동완성 | 약함 | 강함(`.` 치면 메서드 목록) |
| 버그 조기 발견 | 실행해봐야 앎 | mypy가 미리 잡음 |
| 코드 가독성 | 파라미터 용도 추측 | 시그니처만 봐도 파악 |
| 리팩토링 안전성 | 위험 | IDE가 타입 기반 추적 |

### 비교 예시

```python
# 타입 힌트 없음 (Python 3.4 이하 스타일)
def __init__(self, payment_method):
    self.payment_method = payment_method

# 타입 힌트 있음 (Python 3.5+ 권장)
def __init__(self, payment_method: PaymentStrategy):
    self.payment_method = payment_method
```

```python
# 타입 힌트 없으면
processor = PaymentProcessor("잘못된 문자열")  # 실행됨, 나중에 에러
processor.payment_method.process(100)  # AttributeError!

# 타입 힌트 있으면
processor = PaymentProcessor("잘못된 문자열")  # IDE가 즉시 경고 표시
                                              # 하지만 실행은 됨
```

**왜 실행은 되나**: 파이썬은 **동적 타입 언어(Dynamic Typing)**. 타입 힌트는 "강제가 아닌 권고"다. `isinstance()` 같은 명시적 체크를 따로 하지 않는 한, 잘못된 타입이 들어와도 런타임에는 안 막는다.

## 기본 타입 힌트 문법

```python
name: str = "김철수"
age: int = 30
price: float = 9.99
is_active: bool = True
data: list = [1, 2, 3]
config: dict = {"key": "value"}
tags: tuple = ("a", "b")
unique_ids: set = {1, 2, 3}
nothing: None = None
```

### 제네릭(Generic) 타입: 컨테이너 안의 타입까지 명시

```python
# Python 3.9+ (권장)
names: list[str] = ["김철수", "이영희"]
scores: dict[str, int] = {"김철수": 95, "이영희": 88}
coordinates: tuple[float, float] = (37.5, 127.0)

# Python 3.8 이하 (구버전)
from typing import List, Dict, Tuple
names: List[str] = ["김철수", "이영희"]
```

**일상 비유**: `list[str]`은 "문자열만 담는 바구니", `dict[str, int]`는 "문자열 열쇠와 정수 값으로 된 사물함".

### Optional과 Union (Python 3.10+ 파이프 문법)

```python
# 값이 있거나 None일 수 있음
region: str | None = None  # Python 3.10+
# 구버전: from typing import Optional; region: Optional[str] = None

# 여러 타입 중 하나
value: int | str = 42  # Python 3.10+
# 구버전: from typing import Union; value: Union[int, str] = 42
```

### Callable, Any

```python
from typing import Callable, Any

# 함수 타입: (int, int) -> int
add_fn: Callable[[int, int], int] = lambda x, y: x + y

# 아무 타입이나 OK(최후의 수단, 남용 금지)
metadata: Any = {"anything": "goes"}
```

## 클래스 속성에 타입 힌트 붙이기

### `__init__` 밖에서 선언

```python
class Agent:
    name: str        # 클래스 선언부에 타입만
    tools: list
    max_iter: int = 5  # 기본값과 함께

    def __init__(self, name: str, tools: list):
        self.name = name
        self.tools = tools
```

### `__init__` 안에서 선언

```python
class Agent:
    def __init__(self, name: str, tools: list):
        self.name: str = name              # 속성에도 타입 힌트 가능
        self.tools: list[BaseTool] = tools
        self.history: list[str] = []       # 초기화값과 함께
```

**어느 쪽이 낫나**: 둘 다 유효. 클래스 상단에 선언하면 전체 속성을 한눈에 볼 수 있어 문서화 효과가 크다. `@dataclass`를 쓸 땐 상단 선언이 필수.

## AI Agent 실무 예시

**LangChain/LangGraph 코드는 타입 힌트 없으면 거의 사용 불가능**. 객체 간 계약이 복잡해서 IDE 도움 없이는 어떤 메서드를 호출해야 할지 추측이 힘들다.

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import BaseTool
from langchain_core.messages import BaseMessage
from langgraph.checkpoint.memory import MemorySaver

class ResearchAgent:
    def __init__(
        self,
        llm: ChatOpenAI,
        tools: list[BaseTool],
        checkpointer: MemorySaver | None = None,
        system_prompt: str = "You are a helpful assistant.",
        max_iterations: int = 10,
    ):
        self.llm: ChatOpenAI = llm
        self.tools: list[BaseTool] = tools
        self.checkpointer: MemorySaver = checkpointer or MemorySaver()
        self.system_prompt: str = system_prompt
        self.max_iterations: int = max_iterations
        self.history: list[BaseMessage] = []

    def invoke(self, query: str) -> dict:
        # 반환 타입도 명시
        response = self.llm.invoke(query)
        return {"result": response.content}
```

**여기서 IDE가 해주는 일**:
- `self.llm.` 타이핑하면 `ChatOpenAI`의 모든 메서드 자동완성
- `tools`에 `str`을 넣으려 하면 빨간 밑줄
- `response.content`는 `str`임을 추론해서 `.upper()` 같은 문자열 메서드 제안

## 반환 타입 힌트

```python
def process(self, amount: int) -> str:
    #                           ↑ 반환 타입
    return f"${amount} 결제 완료"

def get_config(self) -> dict[str, Any]:
    return self.config

def create_agent(self) -> "Agent":  # 문자열로 감싸면 "전방 참조"(forward reference)
    return Agent()

def nothing(self) -> None:  # 반환값 없음 명시
    print("side effect only")
```

## 커스텀 클래스를 타입으로

```python
class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount: int) -> str: ...

class PaymentProcessor:
    def __init__(self, strategy: PaymentStrategy):  # 추상 클래스도 타입으로
        self.strategy = strategy
```

이 코드가 자식 클래스도 허용하는 이유: **리스코프 치환 원칙(LSP)** 덕분에 `PaymentStrategy`의 자식(`CreditCardPayment` 등)도 같은 자리에 넣을 수 있다. [[class_오버라이딩]] 참고.

## 타입 힌트가 런타임에 안 막는 증거

**직접 확인**: 아래 코드 실행해보면 에러 없이 그냥 돈다.

```python
def greet(name: str) -> str:
    return f"hi {name}"

# 타입 힌트 거슬러 int 넘기기
result = greet(123)
print(result)  # "hi 123" ← 작동함!
print(type(result))  # <class 'str'>
```

**왜 이렇게 설계됐나**: 기존 파이썬 코드와 100% 호환을 유지하려고. 타입 힌트를 강제하면 과거 수백만 줄의 코드가 깨진다. 대신 **mypy** 같은 외부 도구에 검증 책임을 위임.

### 런타임 타입 강제가 필요하면 Pydantic

**AI Agent의 API 입출력 스키마는 Pydantic으로 런타임 검증**한다.

```python
from pydantic import BaseModel

class QueryInput(BaseModel):
    question: str
    max_results: int = 5

# 잘못된 타입 주면 즉시 에러
QueryInput(question="hi", max_results="many")  # ValidationError!
```

## 단계 분해: mypy로 타입 검증 돌리기

1. **1단계**: `pip install mypy`
2. **2단계**: 코드 작성(타입 힌트 포함)
3. **3단계**: 터미널에서 `mypy my_agent.py` 실행
4. **4단계**: 리포트 확인, 타입 불일치 위치와 이유 출력

```bash
my_agent.py:15: error: Argument 1 to "PaymentProcessor" has incompatible type "str"; expected "PaymentStrategy"
```

## 실무 주의점

### 1. `Any` 남용 금지

```python
def process(data: Any) -> Any:  # 타입 힌트 효과 0
    ...
```

`Any`는 "타입 체크 포기" 선언. 정말 타입을 모를 때만 쓰고, 가능한 한 구체적으로 적어야 한다.

### 2. `TYPE_CHECKING`으로 순환 참조 회피

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from agent_module import Agent  # 런타임엔 import 안 함

class Tool:
    def set_agent(self, agent: "Agent") -> None:  # 문자열로 감싸기
        self.agent = agent
```

**왜 필요**: A 모듈이 B를 import하고 B가 A를 import하면 순환 참조 에러. 타입 힌트용이라면 `TYPE_CHECKING` 블록에서만 import해서 피함.

### 3. Python 버전별 문법 차이

| 버전 | 문법 예시 |
|---|---|
| 3.8 이하 | `from typing import List, Optional; List[str], Optional[str]` |
| 3.9+ | `list[str]`(List import 불필요) |
| 3.10+ | `str \| None`(Optional import 불필요) |

**실무 팁**: 프로젝트의 `pyproject.toml`에 명시된 최소 파이썬 버전에 맞춰 문법 선택.

## 정리

- 타입 힌트 = 문서화 + IDE 도움 + 정적 분석(런타임 영향 0)
- `변수: 타입` 또는 `def f(x: 타입) -> 반환타입` 패턴
- AI Agent 코드에선 **복잡한 객체 간 계약을 명시하는 필수 도구**
- 런타임 강제는 Pydantic 같은 별도 라이브러리 사용
- `Any` 남용은 타입 힌트의 의미를 지움

## 관련 문서

- [[class_파라미터]]: 타입 힌트를 가장 많이 붙이는 자리
- [[class_`__init__`]]: 속성 초기화 시 타입 명시
- [[class에서 self. 필요성]]: `self.x: int = x` 형태로 속성 타입
- [[class_@abstractmethod_추상 클래스]]: 추상 클래스를 타입으로 쓰는 패턴
