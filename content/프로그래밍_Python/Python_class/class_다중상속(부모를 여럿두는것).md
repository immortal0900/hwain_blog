---
생성날짜:
- 2026-01-28 02:42
마지막수정날짜:
- 2026-01-28-수요일 02:42
tags:
- python
- class
- multiple_inheritance
- MRO
- mixin
- AI_Agent
별칭:
- Multiple Inheritance
- 다중 상속
- MRO
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python
Project:
---

# 다중 상속(Multiple Inheritance)과 MRO

## 한마디 요약

**다중 상속(Multiple Inheritance)**: 자식 클래스가 **여러 부모를 동시에 상속**받는 기능. 부모들이 같은 이름의 메서드를 가질 때 파이썬은 **MRO(Method Resolution Order, 메서드 탐색 순서)**에 따라 누구 걸 쓸지 결정한다. 실무에선 **Mixin 패턴**(기능 조립식 블록)으로 자주 사용.

## 포함 관계

**다중 상속 ⊃ MRO(탐색 순서) ⊃ C3 linearization(MRO 계산 알고리즘)**

| 용어 | 영문 | 의미 |
|---|---|---|
| 다중 상속 | Multiple Inheritance | 부모를 여럿 두는 것 |
| MRO | Method Resolution Order | 메서드 찾아가는 순서 |
| C3 선형화 | C3 Linearization | MRO를 계산하는 알고리즘 |
| Mixin | Mixin | 기능만 제공하는 얇은 부모 클래스 |
| 다이아몬드 상속 | Diamond Inheritance | A를 B와 C가 상속, D가 B/C 상속하는 모양 |

## 일상 비유: 레고 블록 조립

- **단일 상속**: 레고 완성품 세트(기본 베이스) 한 개만 씀
- **다중 상속**: 여러 미니 블록 세트(로깅, 캐싱, 메시지 발송)를 한 자식에 합쳐 조립
- **Mixin**: "이 블록은 독립 완성품이 아님, 다른 것과 붙어야 의미 있음"

## 다중 상속 기본

**여러 부모 클래스를 동시에 상속**

```python
class Parent1:
    def method1(self):
        return "Parent1 메서드"

class Parent2:
    def method2(self):
        return "Parent2 메서드"

class Child(Parent1, Parent2):  # 2개 부모 상속
    def child_method(self):
        return "Child 메서드"

# 사용
obj = Child()
print(obj.method1())      # Parent1 메서드
print(obj.method2())      # Parent2 메서드
print(obj.child_method()) # Child 메서드
```

## 메서드 순서 (MRO: Method Resolution Order)

**왼쪽에서 오른쪽 순서로 메서드 찾음**

```python
class A:
    def greet(self):
        return "A"

class B:
    def greet(self):
        return "B"

class C(A, B):  # A가 먼저
    pass

obj = C()
print(obj.greet())  # "A" ← A가 우선!
print(C.mro())      # 순서 확인
# [<class 'C'>, <class 'A'>, <class 'B'>, <class 'object'>]
```

### 단계 분해: `obj.greet()` 호출 시

1. **1단계**: 파이썬이 `C` 클래스에서 `greet` 찾음 → 없음
2. **2단계**: MRO 순서대로 다음 후보 `A`로 이동 → `greet` 있음! 실행
3. **3단계**: `B`, `object`는 더 볼 필요 없음(`A`에서 끝남)

## MRO를 직접 확인하는 법

**`ClassName.mro()` 또는 `ClassName.__mro__`로 탐색 순서 확인**

```python
class Animal:
    def speak(self):
        return "동물 소리"

class Flyable:
    def move(self):
        return "날기"

class Swimmable:
    def move(self):
        return "헤엄치기"

class Duck(Animal, Flyable, Swimmable):
    pass

print(Duck.mro())
# [<class 'Duck'>, <class 'Animal'>, <class 'Flyable'>, <class 'Swimmable'>, <class 'object'>]

duck = Duck()
print(duck.move())  # "날기" (Flyable이 Swimmable보다 왼쪽)
```

**한눈에 읽는 법**: `Duck.mro()`의 리스트 순서대로 속성/메서드를 찾는다. 왼쪽부터 차례대로.

## 다이아몬드 상속 문제

**A를 B, C가 상속 → D가 B, C 모두 상속하는 다이아몬드 모양**

```
    A
   / \
  B   C
   \ /
    D
```

```python
class A:
    def hi(self): return "A"

class B(A):
    def hi(self): return "B"

class C(A):
    def hi(self): return "C"

class D(B, C):
    pass

d = D()
print(d.hi())      # "B" (MRO: D → B → C → A → object)
print(D.mro())
# [D, B, C, A, object]
```

**왜 "B"가 나오나**: 파이썬 3의 **C3 선형화 알고리즘**이 "왼쪽 부모부터 깊이 우선, 하지만 공통 조상은 가장 뒤로"라는 규칙으로 순서를 만든다. 덕분에 A는 중복 호출되지 않고 한 번만 등장.

### C3 선형화 규칙 (간단 버전)

1. 자식은 항상 부모보다 앞
2. 여러 부모 중 왼쪽에 쓴 게 오른쪽보다 앞
3. 위 규칙들을 일관되게 만족시키는 유일한 순서가 있으면 그걸 MRO로

**만약 일관된 순서가 불가능하면?** `TypeError: Cannot create a consistent method resolution order (MRO)` 발생, 클래스 선언 자체가 실패한다.

```python
class A: pass
class B(A): pass
class C(A, B): pass  # TypeError! B가 A보다 앞인데 A가 B보다 앞이어야 해서 모순
```

## Mixin 패턴: 다중 상속의 실무 활용

**Mixin**: "이 클래스는 단독으로 인스턴스화하지 않고, **다른 클래스에 기능을 추가하는 용도**"로만 쓰는 얇은 클래스. 관례상 이름 끝에 `Mixin`을 붙인다.

```python
class LoggableMixin:
    def log(self, message):
        print(f"[LOG] {message}")

class CacheableMixin:
    def __init__(self):
        self._cache = {}

    def cache_get(self, key):
        return self._cache.get(key)

    def cache_set(self, key, value):
        self._cache[key] = value

class TimestampMixin:
    def timestamp(self):
        from datetime import datetime
        return datetime.now().isoformat()

# 조립식 클래스
class SmartAgent(LoggableMixin, CacheableMixin, TimestampMixin):
    def __init__(self, name):
        super().__init__()  # Mixin들의 __init__ 호출
        self.name = name

    def work(self, task):
        ts = self.timestamp()
        self.log(f"[{ts}] {self.name}이 {task} 시작")
        cached = self.cache_get(task)
        if cached:
            self.log("캐시 히트")
            return cached
        result = f"{task} 결과"
        self.cache_set(task, result)
        return result

agent = SmartAgent("리서치봇")
agent.work("부동산 분석")
```

**왜 Mixin을 쓰나**:
- **단일 책임 분리**: 로깅/캐싱/타임스탬프가 독립적으로 교체 가능
- **조합 자유**: `class BasicAgent(LoggableMixin)` 처럼 필요한 것만 골라 쓰기
- **상속 깊이 얕음**: 한 줄로 여러 기능 획득

**트레이드오프**: MRO가 복잡해지면 디버깅 난이도 상승. 5개 이상의 Mixin 조합은 주의.

## AI Agent 실무 예시: LangChain Callback Mixin

**LangChain은 Mixin 스타일로 Callback 기능을 조립**한다.

```python
from langchain.callbacks.base import BaseCallbackHandler

class LoggingCallbackMixin(BaseCallbackHandler):
    def on_llm_start(self, serialized, prompts, **kwargs):
        print(f"[LLM 시작] prompts={prompts}")

    def on_llm_end(self, response, **kwargs):
        print(f"[LLM 종료] response={response}")

class MetricsCallbackMixin(BaseCallbackHandler):
    def __init__(self):
        super().__init__()
        self.call_count = 0

    def on_llm_start(self, serialized, prompts, **kwargs):
        self.call_count += 1

class CombinedCallback(LoggingCallbackMixin, MetricsCallbackMixin):
    """로깅 + 메트릭 동시에"""
    pass

callback = CombinedCallback()
# 양쪽 기능 모두 활성화
```

### 직접 확인: MRO 검증

```python
print(CombinedCallback.mro())
# [CombinedCallback, LoggingCallbackMixin, MetricsCallbackMixin, BaseCallbackHandler, object]
```

## 부모 초기화 문제: 여러 부모가 `__init__`을 가질 때

**가장 흔한 함정**. `super().__init__()` 한 번 호출로 모든 부모 초기화를 체인처럼 전달하는 패턴이 표준. [[class_상속시`super()` 사용]] 참고.

```python
class A:
    def __init__(self):
        print("A.__init__")
        super().__init__()  # 다음 MRO로 전달 필수

class B:
    def __init__(self):
        print("B.__init__")
        super().__init__()

class C(A, B):
    def __init__(self):
        print("C.__init__")
        super().__init__()  # MRO 순서대로 A → B → object 호출

c = C()
# 출력:
# C.__init__
# A.__init__
# B.__init__

print(C.mro())
# [C, A, B, object]
```

**왜 A.__init__이 B.__init__을 호출하게 되나**: `A.__init__` 안의 `super().__init__()`은 **MRO의 다음 클래스**를 호출함. `C`의 MRO에서 A 다음은 B이기 때문. 단일 상속만 생각하면 "A의 super는 object"라고 착각하기 쉬운데, MRO는 **자식 클래스가 뭘 상속하느냐에 따라 동적으로 결정**된다.

## 실무 주의점

### 1. 상속 순서가 동작을 바꾼다

```python
class Duck(Animal, Flyable):  # 날기 우선
class Duck(Flyable, Animal):  # Animal 우선
```

**같은 부모 목록인데 순서만 달라져도 MRO가 다르게 나옴**. 의도한 순서로 명시.

### 2. `super()`를 모두 호출해야 체인 유지

**하나라도 `super().__init__()`를 빼면 그 뒤 부모들은 초기화되지 않음**.

```python
class A:
    def __init__(self):
        print("A")
        # super().__init__() 호출 안 함 ← 체인 끊김!

class B:
    def __init__(self):
        print("B")
        super().__init__()

class C(A, B):
    def __init__(self):
        super().__init__()

C()
# 출력: A  ← B 초기화 안 됨!
```

### 3. 다이아몬드가 아닌 경우엔 문제없음

두 부모가 **공통 조상을 공유하지 않으면** MRO 복잡도가 낮고 버그 확률도 낮다. Mixin이 보통 이 케이스.

## Mixin 설계 팁

| 팁 | 이유 |
|---|---|
| 이름에 `Mixin` 접미사 | 단독 사용 안 함을 명시 |
| 상태(`self.x`) 최소화 | 단순 기능 제공이 원칙 |
| `__init__`에서 `super().__init__()` 필수 | 체인 유지 |
| 하나의 책임만 가짐 | SRP 준수 |
| Mixin끼리 서로 의존 금지 | 조합 자유도 보존 |

## 정리

- **다중 상속** = 여러 부모 클래스 동시 상속
- **MRO** = 메서드 탐색 순서 (`.mro()`로 확인)
- **왼쪽 부모 우선** + **C3 선형화**로 일관된 순서 보장
- **Mixin 패턴**이 다중 상속의 실무 활용 핵심(조립식 기능 추가)
- `super().__init__()`으로 부모 체인 유지가 필수
- AI Agent에선 Callback, Logging, Caching 같은 횡단 관심사(cross-cutting concerns)를 Mixin으로

## 관련 문서

- [[class_상속시`super()` 사용]]: 다중 상속에서 `super()`의 작동 원리
- [[class_오버라이딩]]: 부모 메서드를 자식에서 재정의할 때 MRO의 영향
- [[class_@abstractmethod_추상 클래스]]: 다중 상속으로 여러 인터페이스 구현
- [[class_`__init__`]]: 다중 상속 시 초기화 체인
