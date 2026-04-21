---
생성날짜:
- 2026-01-28 02:06
마지막수정날짜:
- 2026-01-28-수요일 02:06
tags:
- python
- class
- overriding
- LSP
- 상속
- AI_Agent
별칭:
- Method Override
- 메서드 재정의
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python
Project:
---

# 오버라이딩(Overriding) 완전 정리

## 한마디 요약

**오버라이딩(Overriding, 메서드 재정의)**: 자식 클래스가 부모의 메서드를 **같은 이름으로 다시 써서 덮어쓰기**. 단, 덮어쓸 때 **부모의 계약(입력/출력 약속)을 깨면 안 된다**(리스코프 치환 원칙, LSP).

## 포함 관계

**상속(Inheritance) ⊃ 오버라이딩(Overriding) ⊃ 리스코프 원칙(LSP) 준수**

| 개념 | 설명 | 예시 |
|---|---|---|
| 상속 | 부모 클래스의 속성/메서드를 물려받음 | `class Child(Parent):` |
| 오버라이딩 | 물려받은 메서드를 자식에서 재정의 | `def greet(self): return "안녕"` |
| 오버로딩(Overloading) | 같은 이름 다른 시그니처 메서드 여러 개(파이썬엔 없음) | `void f(int)` vs `void f(str)` |
| LSP | 자식은 부모를 대체 가능해야 함 | 부모 자리에 자식 넣어도 깨지지 않음 |

**주의**: 파이썬은 오버로딩(같은 이름으로 시그니처만 다르게)을 기본 지원하지 않는다. `@singledispatch`로 흉내는 가능.

## 오버라이딩 기본 컨셉

**자식 클래스가 부모 클래스의 메서드를 재정의하는 것**

```python
class Parent:
    def greet(self):
        return "안녕하세요"

class Child(Parent):
    def greet(self):  # 오버라이딩 (재정의)
        return "안녕"

p = Parent()
c = Child()
print(p.greet())  # "안녕하세요"
print(c.greet())  # "안녕" ← 부모 메서드 무시, 자식 메서드 사용
```

### 단계 분해: 메서드 해석 순서

1. **1단계**: `c.greet()` 호출 시 파이썬이 먼저 `Child` 클래스에서 `greet` 찾음
2. **2단계**: 있으면 그걸 실행(부모 메서드는 가려짐)
3. **3단계**: `Child`에 없으면 MRO(Method Resolution Order) 따라 부모로 올라가며 찾음

**일상 비유**: 회사 매뉴얼(부모)에 "고객 응대 시 '안녕하세요' 인사"라고 써 있어도, 신입팀 매뉴얼(자식)에서 "우린 그냥 '안녕'"이라고 덮어쓰면 신입팀은 '안녕'만 씀. 단, 고객 기대(계약)는 어기면 안 됨.

## 나쁜 구현

### 1. 빈 구현으로 오버라이드 (나쁨)

**부모 기능을 아예 없애버림**

**왜 나쁜가**:
- 부모 클래스의 처리(결제 처리)를 무시함
- 사용자는 결제될 것이라 예상했지만 실제론 아무 일도 안 일어남
- 버그 발생

```python
class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        """결제를 처리합니다"""
        pass

class CreditCardPayment(PaymentStrategy):
    def process(self, amount):
        pass  # 빈 구현! 아무것도 안 함 (나쁨!)

# 사용
payment = CreditCardPayment()
result = payment.process(100)  # None 반환, 아무 일도 안 일어남!
```

**올바른 방식**:

```python
class CreditCardPayment(PaymentStrategy):
    def process(self, amount):
        # 실제 로직 구현
        return f"신용카드 {amount}원 결제 완료"
```

### 2. 약한 전제조건 / 강한 후속조건 (나쁨)

**리스코프 치환 원칙(Liskov Substitution Principle, LSP)의 핵심 규칙**:
- 자식은 **전제조건(precondition)을 더 약하게**(더 관대하게)
- 자식은 **후속조건(postcondition)을 더 강하게**(더 엄격한 보장)

이 방향을 뒤집으면 "부모 자리에 자식을 넣으면 깨지는" 상황이 생긴다.

#### 전제조건(Precondition): 함수 실행 전 요구사항

**원칙 위반**: 자식은 부모보다 **더 관대**해야 함(덜 까다로워야 함)

```python
class FileProcessor:
    def read_file(self, filepath):
        """파일을 읽습니다. filepath는 문자열이어야 함"""
        if not isinstance(filepath, str):
            raise TypeError("filepath는 문자열이어야 함")
        with open(filepath) as f:
            return f.read()

# 나쁜 자식: 전제조건을 더 강하게 만듦
class StrictFileProcessor(FileProcessor):
    def read_file(self, filepath):
        # 부모: str만 요구
        # 자식: str + 확장자까지 요구 (더 까다로움!)
        if not filepath.endswith('.txt'):
            raise ValueError("txt 파일만 가능")  # 추가 제약!
        return super().read_file(filepath)

# 문제 발생
processor = FileProcessor()
processor.read_file("data.json")  # 작동

strict = StrictFileProcessor()
strict.read_file("data.json")  # 에러! (자식이 더 까다로움)
```

#### 후속조건(Postcondition): 함수 실행 후 보장 사항

**원칙 위반**: 자식은 부모보다 **더 강한 보장**을 해야 함

```python
class DataFetcher:
    def fetch(self):
        """데이터를 반환합니다. 반환값: list"""
        return [1, 2, 3]

# 나쁜 자식: 후속조건을 약하게 만듦
class WeakDataFetcher(DataFetcher):
    def fetch(self):
        # 부모: list 반환 보장
        # 자식: None 반환 가능 (보장 약화!)
        if random.random() < 0.5:
            return None  # 부모 약속 위반!
        return [1, 2, 3]

# 문제 발생
fetcher = DataFetcher()
data = fetcher.fetch()
print(len(data))  # 3 (list 보장)

weak = WeakDataFetcher()
data = weak.fetch()
print(len(data))  # TypeError! (None일 수 있음)
```

### 3. 예외를 던지는 오버라이드 (나쁨)

**왜 나쁜가**:
- 부모를 사용하던 코드에 자식을 넣으면 갑자기 에러 발생
- **리스코프 치환 원칙(LSP) 위반**

```python
class Calculator:
    def add(self, a, b):
        """두 수를 더합니다. 예외 없음"""
        return a + b

# 나쁜 자식: 갑자기 예외 던짐
class StrictCalculator(Calculator):
    def add(self, a, b):
        if a < 0 or b < 0:
            raise ValueError("음수는 안 됨!")  # 부모는 안 던지는 예외!
        return a + b

# 문제 발생
calc = Calculator()
result = calc.add(-5, 10)  # 5 (작동)

strict = StrictCalculator()
result = strict.add(-5, 10)  # ValueError! (예상 못 한 에러)
```

## 올바른 오버라이딩 패턴

### 1. 기능 확장 (좋음)

```python
class Logger:
    def log(self, message):
        print(message)

class FileLogger(Logger):
    def log(self, message):
        super().log(message)  # 부모 기능 유지
        with open("log.txt", "a") as f:
            f.write(message + "\n")  # 추가 기능
```

**포인트**: `super().log(message)`로 부모 로직을 보존하고, 거기에 파일 쓰기 동작을 **추가**. 부모의 계약("메시지 출력")은 그대로 지킴. [[class_상속시`super()` 사용]] 참고.

### 2. 구체적 구현 (좋음)

```python
class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        pass

class CreditCardPayment(PaymentStrategy):
    def process(self, amount):
        # 구체적 구현 (빈 구현 아님!)
        return f"신용카드 {amount}원 결제"
```

추상 메서드([[class_@abstractmethod_추상 클래스]])를 자식에서 실제 로직으로 채우는 패턴. **Strategy 디자인 패턴**의 핵심.

### 3. 더 관대한 입력 허용 (좋음)

```python
class Greeter:
    def greet(self, name: str):
        return f"안녕, {name}"

class FlexibleGreeter(Greeter):
    def greet(self, name):  # str뿐만 아니라 모든 타입 허용
        if name is None:
            name = "손님"  # None도 처리
        return f"안녕, {name}"
```

## AI Agent 실무 예시: LangChain 커스텀 Retriever

**LangChain의 `BaseRetriever`를 상속해 커스텀 검색기를 만들 때, `_get_relevant_documents` 메서드를 오버라이딩하는 게 표준**.

```python
from langchain_core.retrievers import BaseRetriever
from langchain_core.documents import Document
from langchain_core.callbacks import CallbackManagerForRetrieverRun

class RealEstateRetriever(BaseRetriever):
    """부동산 데이터 전용 Retriever"""

    def _get_relevant_documents(
        self,
        query: str,
        *,
        run_manager: CallbackManagerForRetrieverRun,
    ) -> list[Document]:
        # LSP 준수:
        # - 부모가 list[Document] 반환 약속 → 자식도 list[Document] 반환
        # - 부모가 받는 query: str → 자식도 str 받음
        # - 추가 예외 던지지 않음

        results = self._search_real_estate_db(query)
        return [
            Document(page_content=r["content"], metadata=r["meta"])
            for r in results
        ]

    def _search_real_estate_db(self, query: str) -> list[dict]:
        # 실제 DB 검색 로직
        return [{"content": f"{query} 결과", "meta": {"source": "real_estate"}}]
```

**왜 이렇게 해야 하나**: LangChain의 Agent는 `retriever.invoke(query)`를 호출할 때 **항상 `list[Document]`를 받을 것으로 기대**. 우리가 이 계약을 깨면(예: `None` 반환, 다른 예외 던짐) Agent 전체 파이프라인이 터진다.

### 잘못된 커스텀 Retriever 예시

```python
class BadRetriever(BaseRetriever):
    def _get_relevant_documents(self, query, *, run_manager):
        if len(query) < 5:
            raise ValueError("쿼리가 너무 짧음")  # LSP 위반! 부모는 이 예외 안 던짐
        if random.random() < 0.1:
            return None  # LSP 위반! list 반환 약속 깸
        return []
```

## 파이썬 오버라이딩 vs 자바의 차이

| 관점 | 파이썬 | 자바 |
|---|---|---|
| `@Override` 애노테이션 | 없음(선택적으로 `@override` Python 3.12+) | 필수 권장 |
| 컴파일 시점 검증 | 없음(런타임에야 앎) | `@Override` 붙이면 컴파일러가 검증 |
| 시그니처 일치 강제 | 없음(이름만 같아도 OK) | 엄격함 |
| 오버로딩(같은 이름 다른 시그니처) | 없음 | 기본 지원 |

**Python 3.12+**: `typing.override` 데코레이터 추가. `@override` 붙이면 정적 분석기가 "정말 부모 메서드를 오버라이드 중인지" 검증.

```python
from typing import override  # Python 3.12+

class Child(Parent):
    @override  # 부모에 greet이 없으면 mypy가 경고
    def greet(self):
        return "안녕"
```

## 실무 예시

### 나쁜 예: FastAPI

```python
class BaseAPI:
    def get_data(self):
        return {"status": "ok", "data": [1, 2, 3]}

class BrokenAPI(BaseAPI):
    def get_data(self):
        pass  # 빈 구현! None 반환

@app.get("/data")
def endpoint():
    api = BrokenAPI()
    return api.get_data()  # None 반환, 클라이언트 에러!
```

## 요약 표

| 나쁜 패턴 | 설명 | 예시 |
|---|---|---|
| **빈 구현** | `pass`만 쓰고 아무것도 안 함 | `def process(self): pass` |
| **약한 전제조건** | 자식이 더 까다로운 입력 요구 | 부모: str / 자식: `.txt`만 |
| **강한 후속조건** | 자식이 더 약한 결과 반환 | 부모: list / 자식: None 가능 |
| **예외 추가** | 부모는 안 던지는 예외 던짐 | 부모: 정상 / 자식: ValueError |

## 정리

- 오버라이딩 = 자식이 부모 메서드 재정의
- 빈 구현(`pass`) = 기능 없앰 (나쁨)
- 자식은 부모보다 **더 관대하게 입력**, **더 강하게 보장** 해야 함
- 예외 추가 = 사용자 예상 깸 (나쁨)
- **LSP(리스코프 치환 원칙) 준수**가 올바른 오버라이딩의 핵심
- 부모 로직 유지하고 싶으면 `super().method()`로 호출 후 기능 추가
- AI Agent 실무: LangChain Retriever, Tool, Callback 상속 시 반드시 LSP 지킬 것

## 관련 문서

- [[class_@abstractmethod_추상 클래스]]: 오버라이딩을 강제하는 장치
- [[class_상속시`super()` 사용]]: 부모 메서드 호출해서 기능 확장
- [[class_다중상속(부모를 여럿두는것)]]: 여러 부모의 메서드가 충돌할 때 MRO
- [[SOLID원칙]]: LSP(리스코프 치환 원칙)는 SOLID의 L
- [[class_타입힌트]]: 오버라이딩 시 타입 일관성 유지
