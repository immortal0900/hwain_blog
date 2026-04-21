---
생성날짜:
- 2026-01-28 03:19
마지막수정날짜:
- 2026-01-28-수요일 03:19
tags:
- python
- class
- super
- 상속
- MRO
- mixin
- AI_Agent
별칭:
- super()
- 부모 호출
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python
Project:
---

# 상속 시 `super()` 사용법

## 한마디 요약

**`super()`**: 상속받은 부모 클래스를 **MRO 순서대로 자동 탐색**해서 메서드를 호출하는 빌트인 함수. 다중 상속에서 **중복 호출 방지**와 **부모 체인 유지**를 동시에 해준다.

## 포함 관계

**상속(Inheritance) ⊃ 부모 메서드 호출 ⊃ super() 또는 직접 호출**

| 호출 방식 | 문법 | MRO 반영 | 중복 호출 위험 | 추천 상황 |
|---|---|---|---|---|
| `super()` | `super().method()` | 자동 | 없음(체인 안전) | 대부분의 경우 |
| 직접 호출 | `Parent.method(self)` | 무시 | 있음 | 특정 부모 골라 호출 |

## `super()` 사용 (추천)

**작동 원리**:
- 파이썬이 **MRO(Method Resolution Order)** 따라 순서대로 호출
- 중복 호출 방지(다이아몬드 상속 문제 해결)

```python
class RealEstateTool(BaseTool, LoggableMixin, CacheableMixin):
    def __init__(self):
        super().__init__()  # MRO 순서대로 자동 호출
```

### `super()`가 실제로 하는 일 단계 분해

1. **1단계**: `super()`는 "현재 클래스의 MRO에서 **다음 클래스**"를 찾음
2. **2단계**: 그 클래스의 메서드를 바인딩된 상태(`self` 포함)로 반환
3. **3단계**: 해당 메서드가 호출됨
4. **4단계**: 그 메서드 안에서 또 `super()`를 호출하면, **MRO의 그 다음 클래스**로 이동

**일상 비유**: 회사 결재 라인에서 "윗사람에게 올려"라고 할 때, 정확히 누구인지 상황(MRO)에 따라 자동 결정. 중복 결재는 없음.

## `ClassName.__init__(self)` 직접 호출

**차이점**:
- 특정 클래스만 골라서 호출 가능
- 하지만 MRO(실행순서) 무시 → 중복 호출 위험

```python
class RealEstateTool(BaseTool, LoggableMixin, CacheableMixin):
    def __init__(self):
        BaseTool.__init__(self)       # 명시적 호출
        CacheableMixin.__init__(self) # 명시적 호출
```

## 실제 실행 예시

```python
class A:
    def __init__(self):
        print("A 초기화")

class B(A):
    def __init__(self):
        print("B 초기화")
        super().__init__()

class C(A):
    def __init__(self):
        print("C 초기화")
        super().__init__()

# super() 사용
class D1(B, C):
    def __init__(self):
        print("D1 초기화")
        super().__init__()

print("=== super() 사용 ===")
d1 = D1()
# 출력:
# D1 초기화
# B 초기화
# C 초기화
# A 초기화  ← A는 1번만! (부모들인 B와 C 가 상속받은 부모)

print("\n=== MRO 확인 ===")
print(D1.mro())
# [D1, B, C, A, object]

# 직접 호출
class D2(B, C):
    def __init__(self):
        print("D2 초기화")
        B.__init__(self)
        C.__init__(self)

print("\n=== 직접 호출 ===")
d2 = D2()
# 출력:
# D2 초기화
# B 초기화
# A 초기화  ← A 호출 1번 (D2의 부모인 B의 부모)
# C 초기화
# A 초기화  ← A 호출 2번! (중복) (D2의 부모인 C의 부모)
```

**포인트**: `super()`는 MRO 덕분에 A를 한 번만 호출. 직접 호출은 "B → A"와 "C → A"가 각각 실행돼서 A가 두 번 호출됨. 초기화가 멱등(idempotent)이 아니라면 **상태 오염** 발생.

## 왜 `super()`가 MRO를 따라가나?

**설계 의도**: 파이썬 2의 `super`는 부모 클래스 이름을 명시해야 했고, 다중 상속 시 체인 관리가 악몽이었다. Python 3에서 "현재 클래스 기준 MRO의 다음 클래스"를 자동으로 찾도록 재설계. 이 덕분에 Mixin 패턴([[class_다중상속(부모를 여럿두는것)]])이 실용적으로 가능해졌다.

**트레이드오프**: 디버깅 시 "지금 super()가 누구를 가리키지?" 추적이 어려울 수 있음. `ClassName.mro()`로 먼저 확인하는 습관 필요.

## 언제 뭘 쓰나?

### `super()` 써야 할 때
- **다중 상속**(대부분의 경우)
- Mixin 패턴 활용 시
- 부모 체인 전체 초기화 필요할 때
- 유연한 구조 원할 때

### 직접 호출 써야 할 때
- **특정 부모만 골라서 호출**
- Mixin이 `__init__` 없을 때 필요한 것만
- 레거시 코드 디버깅 중 특정 로직 추적

```python
class LoggableMixin:
    def log(self, message):
        print(f"[LOG] {message}")  # 인자가 message로 들어옴

class CacheableMixin:
    def __init__(self):
        print("CacheableMixin 초기화")
        self.cache = {}

    def get_cached(self, key):
        return self.cache.get(key)  # 딕셔너리 key 값을 넣어서 value를 가져오게 하는 것

class BaseTool:
    def __init__(self):
        print("BaseTool 초기화")
        self.name = ""

# 올바른 방식
class RealEstateTool(BaseTool, LoggableMixin, CacheableMixin):
    name = "real_estate_search"
    description = "부동산 검색"

    def __init__(self):
        print("RealEstateTool 초기화")
        super().__init__()  # MRO 순서대로 모두 호출

    def _run(self, query: str):
        self.log(f"검색 시작: {query}")  # f"[LOG] {message}"의 message로 들어옴
        cached = self.get_cached(query)
        if cached:
            return cached  # 맨처음엔 저장된 캐시가 없으나, 그 후로 같은 질문이 들어오면 query를 key로 value 반환
        result = f"{query} 검색 결과"
        self.cache[query] = result  # f"{query} 검색 결과"가 value, query가 key
        return result

print("=== 객체 생성 ===")
tool = RealEstateTool()
# 출력:
# RealEstateTool 초기화
# BaseTool 초기화
# CacheableMixin 초기화

print("\n=== 실행 ===")
result1 = tool._run("강남 아파트")
# 출력: [LOG] 검색 시작: 강남 아파트
print(result1)  # 강남 아파트 검색 결과

result2 = tool._run("강남 아파트")  # 캐시 사용
print(result2)  # 강남 아파트 검색 결과 (캐시에서)
```

## AI Agent 실무 예시: LangChain Tool 상속

**LangChain `BaseTool`을 상속해 커스텀 툴 만들 때 `super().__init__()` 호출이 필수**. 안 하면 Pydantic 검증, 툴 등록 로직 등 내부 초기화가 건너뛰어진다.

```python
from langchain_core.tools import BaseTool
from pydantic import Field

class RealEstateSearchTool(BaseTool):
    name: str = "real_estate_search"
    description: str = "지역 부동산 시세 검색"

    # 추가 속성
    api_endpoint: str = Field(default="https://api.example.com")
    cache: dict = Field(default_factory=dict)

    def __init__(self, **kwargs):
        super().__init__(**kwargs)  # BaseTool의 Pydantic 초기화 실행
        # 커스텀 초기화 로직은 super() 호출 이후

    def _run(self, query: str) -> str:
        if query in self.cache:
            return self.cache[query]
        result = f"{query} 시세"
        self.cache[query] = result
        return result
```

**주의**: LangChain의 `BaseTool`은 Pydantic 기반이라 `super().__init__(**kwargs)`를 빠뜨리면 `name`, `description` 검증이 안 되고 엉뚱한 에러가 난다.

## `super().method()` vs `super().__init__()`

**`super()`는 `__init__`만 호출하는 게 아니다**. 어떤 메서드든 부모 것을 호출 가능.

```python
class Logger:
    def log(self, message):
        print(f"[Parent] {message}")

class DetailedLogger(Logger):
    def log(self, message):
        super().log(message)  # 부모 log도 실행
        print(f"[Child] 추가 정보: {len(message)} chars")

logger = DetailedLogger()
logger.log("테스트")
# 출력:
# [Parent] 테스트
# [Child] 추가 정보: 3 chars
```

[[class_오버라이딩]]의 "기능 확장 패턴"이 바로 이것.

## `super()`의 두 가지 형태

```python
class Child(Parent):
    def method(self):
        super().method()               # Python 3: 간단 문법(현대적)
        super(Child, self).method()    # Python 2 호환: 명시적 문법
```

**Python 3부터는 인자 없는 `super()`가 표준**. 내부적으로 `super(__class__, <first arg>)`로 변환됨.

## `.get()`이 뭐하는 용도?

**딕셔너리에서 안전하게 값 가져오기** (키 없어도 에러 안 남)

```python
# 일반 접근 vs .get() 비교
cache = {"강남": "강남 데이터"}

# 1. 대괄호 [] 접근
print(cache["강남"])  # "강남 데이터"
print(cache["서초"])  # KeyError 발생!

# 2. .get() 메서드
print(cache.get("강남"))  # "강남 데이터"
print(cache.get("서초"))  # None (에러 안 남!)
print(cache.get("서초", "기본값"))  # "기본값"
```

**왜 `.get()`을 선호**: 검색 결과 없을 때 예외가 아니라 `None`으로 처리하는 게 흔함. 위 예제의 캐시 조회가 대표적 사용처.

## 실무 주의점

### 1. 체인이 끊기면 뒤 부모 초기화 안 됨

```python
class A:
    def __init__(self):
        print("A")
        super().__init__()

class B:
    def __init__(self):
        print("B")
        # super() 빠짐 ← 체인 끊김!

class C(A, B):
    def __init__(self):
        super().__init__()

C()
# 출력: A  ← B, object 초기화 안 됨
```

**규칙**: 다중 상속 체인에선 **모든 `__init__`에 `super().__init__()`을 넣어라**. 단일 상속이라도 넣는 습관이 미래의 다중 상속 전환을 쉽게 만든다.

### 2. `super()`의 인자 전달

**자식의 `__init__`이 추가 인자를 받지만 부모엔 안 넘겨야 할 때**:

```python
class Base:
    def __init__(self, name):
        self.name = name

class Extended(Base):
    def __init__(self, name, extra):
        super().__init__(name)  # name만 부모에 전달
        self.extra = extra       # extra는 자식만 씀
```

### 3. 클래스 메서드에서 `super()` 사용

```python
class Parent:
    @classmethod
    def create(cls):
        return cls()

class Child(Parent):
    @classmethod
    def create(cls):
        instance = super().create()  # 부모 classmethod 호출
        instance.extra = "자식 초기화"
        return instance
```

## `super()` vs 직접 호출 정리표

| 관점 | `super()` | 직접 호출 `Parent.__init__(self)` |
|---|---|---|
| MRO 반영 | O (자동) | X (무시) |
| 중복 호출 방지 | O | X (직접 관리 필요) |
| 다중 상속 친화 | 매우 좋음 | 나쁨 |
| 특정 부모 골라 호출 | 불가 | 가능 |
| 코드 변경 영향 | 부모 바뀌면 자동 적응 | 하드코딩, 수동 업데이트 필요 |
| 실무 표준 | O | 예외적 경우만 |

## 정리

- `super()` = MRO 순서로 부모 메서드 자동 호출
- 다중 상속에서 **중복 호출 방지**가 핵심 장점
- 직접 호출(`Parent.__init__(self)`)은 특수 상황용
- **다중 상속의 모든 `__init__`에 `super().__init__()` 필수**
- `super()`는 `__init__`만이 아니라 **모든 메서드** 호출에 쓸 수 있음
- AI Agent 실무: LangChain `BaseTool` 상속 시 `super().__init__(**kwargs)` 필수

## 관련 문서

- [[class_다중상속(부모를 여럿두는것)]]: `super()`가 따라가는 MRO의 실체
- [[class_오버라이딩]]: `super()`로 부모 로직 유지하며 기능 확장
- [[class_`__init__`]]: `super()`가 가장 많이 쓰이는 장면
- [[class_@abstractmethod_추상 클래스]]: 추상 부모의 기본 구현을 `super()`로 활용
