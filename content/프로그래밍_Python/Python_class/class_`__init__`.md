---
생성날짜:
- 2026-01-27 00:24
마지막수정날짜:
- 2026-01-27-화요일 00:23
tags:
- python
- class
- 생성자
- constructor
- AI_Agent
별칭:
- 생성자
- Constructor
- initializer
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python
Project:
---

# `__init__` 있는 클래스 vs 없는 클래스

## 한눈에 보기

**`__init__`(인스턴스 초기화 메서드, Dunder method의 일종)**: 클래스에서 [[class_인스턴스]]가 생성되는 순간 **자동 실행되는 초기화 훅**. 없어도 클래스는 작동하지만, 속성 설정을 수동으로 해야 해서 실수가 쌓인다.

| 구분 | `__init__` 없음 | `__init__` 있음 |
|---|---|---|
| 객체 생성 | `Person()` OK | `Person("김철수", 30)` 필수 인자 강제 |
| 속성 설정 | 수동(`person.name = ...`) | 자동(생성 시 주입) |
| 필수 데이터 강제 | 불가능 | 가능 |
| 실수 가능성 | 높음(속성 누락 시 AttributeError) | 낮음 |
| 용도 | 단순 컨테이너(거의 안 씀) | 대부분의 실무 클래스 |

## 용어 풀이

- **`__init__`(double underscore init, 던더 이닛)**: "dunder"는 double underscore의 줄임말. 파이썬이 특별한 타이밍에 자동 호출하는 메서드는 `__이름__` 패턴을 쓴다(`__init__`, `__str__`, `__call__` 등). 묶어서 **특수 메서드(Special Method)** 또는 **매직 메서드(Magic Method)**라고 부른다.
- **초기화(Initialization)**: 객체가 태어나자마자 속성에 초기값을 채우는 단계. "생성자(constructor)"라고도 부르는데, 엄밀히는 파이썬의 진짜 생성자는 `__new__`이고 `__init__`은 초기화기(initializer)다. 실무에선 섞어 쓴다.

## `__init__` 없는 클래스

```python
class Person:
    pass  # __init__ 없음

# 사용
person1 = Person()  # 객체 생성은 됨
person1.name = "김철수"  # 수동으로 속성 추가
person1.age = 30       # 수동으로 속성 추가

person2 = Person()
person2.name = "이영희"
person2.age = 25
```

**문제점**:
- 객체마다 속성을 일일이 설정해야 함, 반복 작업 증가
- 실수로 속성 안 넣으면 `AttributeError` 발생
- 필수 데이터 강제 불가(나중에 추가하면 되니까)
- 다른 개발자가 "이 클래스에 어떤 속성이 있어야 하는지" 한눈에 파악 불가

## `__init__` 있는 클래스

```python
class Person:
    def __init__(self, name, age):  # 초기화 메서드
        self.name = name
        self.age = age

# 사용
person1 = Person("김철수", 30)  # 생성과 동시에 초기화
person2 = Person("이영희", 25)  # 간결함!

# name, age 안 넣으면?
person3 = Person()  # TypeError: 필수 인자 누락!
```

**장점**:
- 객체 생성 시 필수 데이터 강제
- 코드 중복 제거
- 일관된 초기화 보장
- 클래스 시그니처만 봐도 "이 객체는 name과 age를 갖는다"가 명확

## 왜 `__init__`이 자동 실행되나?

**단계 분해**: `Person("김철수", 30)` 호출 시

1. **1단계**: 파이썬이 `Person.__new__(Person)`를 호출 → 빈 객체 껍데기 메모리 할당
2. **2단계**: 그 껍데기를 `self`로 넘기며 `Person.__init__(self, "김철수", 30)` 호출
3. **3단계**: `__init__` 안에서 `self.name`, `self.age`에 값 채움
4. **4단계**: 완성된 객체를 반환, `person1`이라는 이름에 바인딩

**설계 의도**: "객체가 태어나는 순간 반드시 거쳐야 하는 초기화 경로"를 클래스 작성자가 강제할 수 있게 하려고. C++의 생성자와 같은 역할이지만, 파이썬은 `__new__`와 `__init__`을 분리해서 **메모리 할당**과 **속성 채우기**를 따로 커스터마이징할 수 있게 했다.

## 실제 차이 비교

### 시나리오: 결제 클래스

```python
# Good: __init__ 있음
class PaymentProcessor:
    def __init__(self, payment_method, api_key):
        self.payment_method = payment_method
        self.api_key = api_key

# 생성 시 강제로 필수 데이터 받음
processor1 = PaymentProcessor(CreditCardPayment(), "secret123")
processor1.process(100)  # 안전


# Bad: __init__ 없음
class PaymentProcessor:
    pass

# 사용할 때마다 수동 설정
processor1 = PaymentProcessor()
processor1.payment_method = CreditCardPayment()  # 깜빡하면 끝
processor1.api_key = "secret123"                 # 이것도 깜빡하면 끝

processor1.process(100)  # AttributeError 위험!
```

## `__init__`이 하는 일

```python
class Car:
    def __init__(self, brand, color):
        # 1. 객체 생성되자마자 자동 실행
        self.brand = brand
        self.color = color

        # 2. 초기 계산/설정도 가능
        self.mileage = 0
        self.is_running = False

        # 3. 초기화 시 로직 실행
        print(f"{brand} {color} 차량 등록 완료")

# 호출
car = Car("Tesla", "Red")  # __init__ 자동 실행됨
# 출력: "Tesla Red 차량 등록 완료"
```

## `__init__` 없으면 무슨 일이?

**파이썬은 자동으로 기본 `__init__`을 제공한다**. 단, 기본 `__init__`은 아무것도 안 함 → 속성 초기화 X

```python
class Empty:
    pass

# 위 코드는 실제로는 이렇게 작동:
class Empty:
    def __init__(self):  # 자동 생성됨 (보이지 않음)
        pass

empty = Empty()  # 작동함! 기본 __init__ 호출됨
```

## 실무 패턴

### 간단한 클래스 (데이터만)

```python
class Point:
    pass

# 이 정도면 __init__ 없어도 OK
point = Point()
point.x = 10
point.y = 20
```

**하지만 실무에선 `dataclass`를 쓰는 게 더 낫다** (아래 참고).

### 복잡한 클래스 (로직 + 데이터)

```python
class DatabaseConnection:
    def __init__(self, host, port, username, password):
        # 초기화 시 연결 생성
        self.host = host
        self.port = port
        self.connection = self._create_connection(username, password)

    def _create_connection(self, username, password):
        # 복잡한 초기화 로직
        return f"Connected to {self.host}:{self.port}"

# __init__ 필수! 연결 설정 자동화
db = DatabaseConnection("localhost", 5432, "admin", "pw123")
```

## AI Agent 개발 실무 예시

**LangGraph 기반 Agent를 클래스로 감쌀 때 `__init__`에서 그래프를 컴파일해두는 패턴이 표준**이다. 왜냐하면 그래프 빌드 비용이 크기 때문에, 인스턴스를 만드는 순간 1회만 빌드하고 이후 `invoke()`마다 재사용하는 게 효율적.

```python
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, END

class ResearchAgent:
    def __init__(self, llm: ChatOpenAI, tools: list):
        # 1단계: 의존성 주입(DI, Dependency Injection)
        self.llm = llm
        self.tools = tools

        # 2단계: 상태 초기화
        self.memory = []

        # 3단계: 그래프 1회 빌드 후 컴파일 결과 저장
        self.graph = self._build_graph()

    def _build_graph(self):
        builder = StateGraph(dict)
        # 노드 추가, 엣지 연결 등
        builder.add_node("research", self._research_node)
        builder.set_entry_point("research")
        builder.add_edge("research", END)
        return builder.compile()  # compile()이 핵심

    def _research_node(self, state):
        response = self.llm.invoke(state["query"])
        return {"result": response.content}

    def invoke(self, input_data):
        # self.graph가 이미 invoke()를 가지고 있으니 위임
        return self.graph.invoke(input_data)

# 사용
llm = ChatOpenAI(model="gpt-4o-mini")
agent = ResearchAgent(llm, tools=[])
agent.invoke({"query": "부동산 분석"})
```

**포인트**:
- `__init__`에서 `llm`, `tools` 같은 무거운 의존성을 **1회만 주입**
- `self.graph = self._build_graph()`로 **컴파일 결과 캐싱**(매번 invoke 시 재빌드 안 함)
- 외부 호출자는 `agent.invoke(...)`만 신경 쓰면 됨(내부 구현 은닉)

## 현대적 대안: `dataclass`

**단순히 속성만 저장하는 클래스라면 `__init__`을 직접 쓰지 말고 `@dataclass` 데코레이터를 쓰는 게 훨씬 깔끔하다** (Python 3.7+).

```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int

# __init__ 자동 생성됨
person = Person("김철수", 30)
print(person)  # Person(name='김철수', age=30) ← __repr__도 자동!
```

**왜 이게 나은가**:
- 보일러플레이트(boilerplate, 반복되는 뻔한 코드) 제거
- `__repr__`, `__eq__` 자동 생성
- 타입 힌트가 필수라 문서화 강제

**언제 `__init__`을 직접 쓰나**: 초기화 시 계산/검증/리소스 연결 같은 **로직**이 필요할 때.

## 실무 주의점

### 1. `__init__`에서 무거운 작업 주의

```python
# 나쁨: 인스턴스 생성마다 네트워크 호출
class BadAgent:
    def __init__(self, model_name):
        self.model = download_model_from_hub(model_name)  # 느림!

# 좋음: 외부에서 받아서 주입
class GoodAgent:
    def __init__(self, model):
        self.model = model  # 이미 로드된 객체 받기
```

**왜**: 테스트 시 매번 네트워크 타기 때문. 의존성 주입(DI) 패턴을 쓰면 테스트에서 mock 객체를 쉽게 주입할 수 있다.

### 2. `__init__`은 반환값이 없다

```python
class Foo:
    def __init__(self):
        return "hello"  # TypeError: __init__() should return None
```

반환값을 주려면 `__new__`를 오버라이드해야 한다(드문 경우).

## 정리

- `__init__`은 **인스턴스 생성 시 자동 실행되는 초기화 메서드**
- 없어도 되지만, 필수 속성 강제와 일관된 초기화를 위해 대부분 작성
- AI Agent 클래스에선 **의존성 주입 + 그래프 컴파일 캐싱** 패턴이 표준
- 단순 데이터 컨테이너는 `@dataclass`가 더 나음
- 무거운 I/O는 `__init__` 밖에서 처리 후 주입하는 게 테스트 친화적

## 관련 문서

- [[class_인스턴스]]: `__init__`이 초기화하는 대상, 메모리에 찍힌 객체
- [[class에서 self. 필요성]]: `__init__` 안의 `self.속성` 저장 방식
- [[class_파라미터]]: `__init__` 파라미터 전달 흐름
- [[class_타입힌트]]: `__init__` 파라미터에 타입 붙이는 법
- [[class_상속시`super()` 사용]]: 자식 클래스의 `__init__`에서 부모 `__init__` 호출
