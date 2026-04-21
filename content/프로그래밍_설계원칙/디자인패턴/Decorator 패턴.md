---
생성날짜:
- 2026-04-21 12:20
마지막수정날짜:
- 2026-04-21-화요일 12:20
tags:
- 설계원칙
- 디자인패턴
- GoF
- 구조패턴
- AI에이전트
별칭:
- Decorator
- 데코레이터
- Wrapper
- 래퍼 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# Decorator 패턴이란?

Decorator(데코레이터, 장식자 패턴, 원본 객체를 같은 인터페이스의 래퍼(wrapper)로 감싸 기능을 덧붙이는 구조 디자인 패턴)는 **"원래 객체는 그대로 두고, 바깥에 층층이 껍질을 입혀 새 기능을 추가하는"** 패턴이다. GoF 분류상 **구조(Structural) 패턴**에 속함.

Python의 `@decorator` 문법과 이름은 같지만 구현 방식이 조금 다르다. 개념은 같은 뿌리임.

## 한눈에 보기

| 항목     | 내용                                   |
| ------ | ------------------------------------ |
| 분류     | 구조 패턴(Structural)                    |
| 한 줄 정의 | 같은 인터페이스를 유지한 채 객체에 책임을 동적으로 추가      |
| 핵심 구성  | Component(인터페이스), 원본, Decorator(래퍼) |
| 해결 문제  | 기능 조합마다 서브클래스 폭발, 상속으로 풀면 유연성 부족     |
| 관련 원칙  | OCP, SRP, 컴포지션(상속 대신 조합)             |

> **한마디 요약**: "원본은 그대로, 바깥을 감싸서 기능 추가. 양파 껍질 쌓듯이."

---

## 일상 비유

햄버거로 생각해보자. 기본 패티가 Component고, 치즈, 베이컨, 양파, 토마토가 데코레이터임.

```
패티 → [치즈 추가] → [베이컨 추가] → [양파 추가] = 최종 햄버거
```

각 껍질은 "햄버거"라는 공통 타입을 유지하면서 자기 기능(비용, 설명)만 덧붙인다. 손님 입장에서는 여전히 "햄버거" 하나를 들고 먹음.

다른 비유:
- 양파 껍질, 선물 포장(포장지 안에 또 포장지)
- 방한 복장(속옷 → 티셔츠 → 셔츠 → 패딩, 각 층이 보온 기능 추가)
- 주택 리모델링(벽은 그대로 두고 외장재만 바꿈)

---

## 구조

```
┌────────────────┐
│  <<interface>> │
│   Component    │
│  + operation() │
└───────┬────────┘
        │ implements
    ┌───┴────────────────┐
    ▼                    ▼
┌─────────┐    ┌──────────────────┐
│Concrete │    │    Decorator     │ ◇── Component (wrapped)
│Component│    │  + operation()   │
└─────────┘    └────────┬─────────┘
                        │ extends
               ┌────────┴─────────┐
               ▼                  ▼
         ConcreteDecoratorA  ConcreteDecoratorB
```

핵심 트릭:
- Decorator도 Component를 구현함 (같은 인터페이스)
- Decorator가 내부에 또 다른 Component를 들고 있음 (composition)
- 그래서 Decorator 안에 또 Decorator를 넣을 수 있음 (층층이)

---

## 나쁜 예 vs 좋은 예 (Python)

### 나쁜 예: 상속 조합 폭발

커피에 우유, 시럽, 휘핑 옵션을 상속으로 표현하면 이렇게 된다.

```python
class Coffee: pass
class CoffeeWithMilk(Coffee): pass
class CoffeeWithSyrup(Coffee): pass
class CoffeeWithMilkAndSyrup(Coffee): pass
class CoffeeWithMilkAndWhip(Coffee): pass
class CoffeeWithSyrupAndWhip(Coffee): pass
class CoffeeWithMilkSyrupAndWhip(Coffee): pass
# 옵션 4개면 16개 클래스, 5개면 32개...
```

옵션이 n개면 2^n 조합. 유지 불가.

### 좋은 예: Decorator

```python
from abc import ABC, abstractmethod

class Beverage(ABC):
    @abstractmethod
    def cost(self) -> int: ...
    @abstractmethod
    def desc(self) -> str: ...

class Espresso(Beverage):
    def cost(self): return 3000
    def desc(self): return "에스프레소"

class BeverageDecorator(Beverage):
    def __init__(self, wrapped: Beverage):
        self._wrapped = wrapped

class Milk(BeverageDecorator):
    def cost(self): return self._wrapped.cost() + 500
    def desc(self): return self._wrapped.desc() + " + 우유"

class Syrup(BeverageDecorator):
    def cost(self): return self._wrapped.cost() + 700
    def desc(self): return self._wrapped.desc() + " + 시럽"

class Whip(BeverageDecorator):
    def cost(self): return self._wrapped.cost() + 800
    def desc(self): return self._wrapped.desc() + " + 휘핑"

# 조립
drink = Whip(Syrup(Milk(Espresso())))
print(drink.desc(), drink.cost())
# 에스프레소 + 우유 + 시럽 + 휘핑 5000
```

새 토핑 추가 = 새 Decorator 클래스 하나. 기존 코드 무수정.

---

## AI 에이전트 개발 예시: LLM 호출 래핑

이게 실무에서 가장 강력하게 쓰이는 케이스다. LLM 호출에 **재시도, 캐싱, 로깅, 관측성, 속도 제한** 같은 교차 관심사(cross-cutting concerns, 여러 모듈에 흩어져 붙는 공통 관심사)를 덧붙이는 상황.

```python
from abc import ABC, abstractmethod
import time

class ChatModel(ABC):
    @abstractmethod
    def chat(self, messages: list) -> str: ...

class OpenAIChat(ChatModel):
    def chat(self, messages):
        return openai.chat(messages)

# 이하 모두 Decorator
class ChatDecorator(ChatModel):
    def __init__(self, wrapped: ChatModel):
        self._wrapped = wrapped

class RetryingChat(ChatDecorator):
    def __init__(self, wrapped, max_retries=3):
        super().__init__(wrapped)
        self.max_retries = max_retries

    def chat(self, messages):
        for attempt in range(self.max_retries):
            try:
                return self._wrapped.chat(messages)
            except RateLimitError:
                time.sleep(2 ** attempt)
        raise

class CachingChat(ChatDecorator):
    def __init__(self, wrapped, cache):
        super().__init__(wrapped)
        self.cache = cache

    def chat(self, messages):
        key = hash_messages(messages)
        if key in self.cache:
            return self.cache[key]
        result = self._wrapped.chat(messages)
        self.cache[key] = result
        return result

class LoggingChat(ChatDecorator):
    def chat(self, messages):
        start = time.time()
        result = self._wrapped.chat(messages)
        elapsed = time.time() - start
        logger.info(f"chat took {elapsed:.2f}s, {len(result)} chars")
        return result

class RateLimitedChat(ChatDecorator):
    def __init__(self, wrapped, rps: int):
        super().__init__(wrapped)
        self.limiter = TokenBucket(rps)

    def chat(self, messages):
        self.limiter.acquire()
        return self._wrapped.chat(messages)

# 조립 (순서가 곧 동작 순서)
llm = LoggingChat(
    CachingChat(
        RetryingChat(
            RateLimitedChat(
                OpenAIChat(),
                rps=10
            ),
            max_retries=3
        ),
        cache={}
    )
)

llm.chat([{"role": "user", "content": "hi"}])
```

호출 흐름:
`Logging → Caching → Retrying → RateLimit → OpenAI → (응답) → RateLimit → Retrying → Caching → Logging`

각 층이 자기 일만 하고 원본 `OpenAIChat` 코드는 한 줄도 건드릴 필요가 없다.

### 왜 순서가 중요한가

`Caching(Retrying(...))` 과 `Retrying(Caching(...))` 은 결과가 다르다.

- `Caching(Retrying(...))`: 캐시 확인 → 없으면 재시도 포함 호출 → 결과 캐시
- `Retrying(Caching(...))`: 재시도 루프 안에서 캐시 확인 → 첫 시도 실패 후 재시도 때 캐시 히트 가능성

요구사항에 맞춰 조립 순서를 결정해야 함.

---

## Python 함수 데코레이터와의 관계

Python `@decorator` 문법은 **함수 버전의 Decorator 패턴**이다. 개념은 같음.

```python
def retry(max_retries=3):
    def wrap(fn):
        def wrapper(*args, **kwargs):
            for i in range(max_retries):
                try:
                    return fn(*args, **kwargs)
                except Exception:
                    if i == max_retries - 1:
                        raise
        return wrapper
    return wrap

def cache(fn):
    store = {}
    def wrapper(*args):
        if args not in store:
            store[args] = fn(*args)
        return store[args]
    return wrapper

@retry(max_retries=3)
@cache
def chat(messages_key):
    return openai.chat(messages_key)
```

`@retry`와 `@cache`가 GoF Decorator의 `RetryingChat`, `CachingChat`에 대응됨. 함수 레벨 vs 객체 레벨의 차이.

---

## 언제 쓰면 좋은가

1. **같은 인터페이스에 기능을 층층이 더하고** 싶을 때
2. **상속 조합이 폭발**할 때 (옵션 n개면 2^n 클래스)
3. **런타임에 기능 조합을 바꾸고** 싶을 때
4. **교차 관심사 주입**(로깅, 캐싱, 재시도, 인증, 트레이싱)이 필요할 때

## 트레이드오프

| 장점                  | 단점                                |
| ------------------- | --------------------------------- |
| 기능 조합을 런타임 결정       | 층이 많으면 디버깅 시 스택 추적이 복잡            |
| 원본 코드 수정 없음 (OCP)   | 객체 수가 많아져 메모리/성능 고려 필요            |
| 재사용성 높음(각 Decorator 독립) | 순서 의존성 존재, 잘못 조립하면 의도와 다른 동작      |
| 상속 폭발 방지            | 간단한 경우엔 상속이나 함수로 충분한데 과하게 쓸 위험 |

---

## 실전 주의점

### 1. 인터페이스 깨뜨리지 말기
Decorator는 **원본과 완전히 동일한 인터페이스**를 유지해야 함. 메서드 추가는 괜찮지만, 원래 있던 메서드의 시그니처/반환 타입을 바꾸면 [[SOLID원칙|LSP]] 위반임.

### 2. 순서 문서화
어떤 순서로 감싸야 하는지가 동작에 크게 영향을 준다. 조립부에 주석으로 의도 남기거나 조립 팩토리 함수로 표준 순서 고정.

```python
def build_llm(base: ChatModel) -> ChatModel:
    """표준 조립 순서: 관측성(외부) → 캐싱 → 재시도 → 속도제한 → 원본(내부)"""
    return LoggingChat(CachingChat(RetryingChat(RateLimitedChat(base))))
```

### 3. 예외 전파 정책
Decorator 층에서 예외를 삼켜버리면 바깥 층이 문제를 감지하지 못함. 처리 가능한 예외만 잡고 나머지는 통과시킬 것.

### 4. 성능
각 층마다 함수 호출 비용이 있음. 깊게 중첩하면 지연이 쌓인다. 핫 패스(hot path, 초당 수천 번 실행되는 코드)에는 주의.

---

## 다른 패턴과의 관계 (포함 관계)

- **[[Chain of Responsibility 패턴|Chain of Responsibility]]와 구조 비슷**: 둘 다 "다음"을 가리키며 위임. 차이: Decorator는 "인터페이스 유지 + 기능 덧붙임", CoR은 "처리 책임 넘김"
- **[[Strategy 패턴|Strategy]]와 구분**: Strategy는 "알고리즘 자체 교체", Decorator는 "알고리즘에 기능 추가"
- **Proxy와 구분**: 구조는 매우 유사하지만 의도 차이. Proxy는 "접근 제어/지연 로딩/원격 호출 대리", Decorator는 "기능 확장"
- **Composite와 자주 조합**: Composite 트리의 각 노드에 Decorator를 덧붙여 기능 강화
- **상속의 대안**: 상속 대신 컴포지션(composition)을 택한 대표 사례. "Favor composition over inheritance" 원칙의 구현

---

## 직접 확인하기

자기 코드에서 다음을 찾아봐라.

- 같은 작업의 변형마다 서브클래스를 만들고 있는가? (상속 폭발 신호)
- LLM 호출, HTTP 클라이언트, DB 접근에 `try/except` 재시도, 수동 캐시, 수동 로깅이 흩어져 있는가? (교차 관심사 신호)
- 한 클래스가 "핵심 로직 + 로깅 + 캐싱 + 재시도"를 모두 하고 있는가? ([[SOLID원칙|SRP]] 위반 + Decorator 후보)

위 신호가 보이면 Decorator로 풀어낼 가치가 있음.

---

## 요약

> **"원본은 건드리지 말고 껍질을 씌워라. 껍질마다 한 가지 기능만. 필요한 만큼 층층이 조립해서 써라."**

관련 문서:
- [[SOLID원칙]]
- [[Strategy 패턴]]
- [[Chain of Responsibility 패턴]]
