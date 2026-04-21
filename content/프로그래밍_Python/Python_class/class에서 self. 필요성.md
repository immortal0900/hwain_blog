---
생성날짜:
- 2026-01-26 19:18
마지막수정날짜:
- 2026-01-26-월요일 19:18
tags:
- python
- class
- self
- 객체지향
- AI_Agent
별칭:
- self
- 인스턴스 참조
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python
Project:
---

# `self.`는 왜 붙이나? 언제 붙이나?

## 한마디 요약

**`self`(이 인스턴스 자기 자신을 가리키는 특수 변수)**는 "**이 객체에 속한 상자**" 라벨이다. 라벨(`self.`) 붙이면 [[class_인스턴스]] 속에 영구 저장, 안 붙이면 함수 안에서만 살다가 버려지는 로컬 변수다.

## 포함 관계 한눈에 보기

**인스턴스(Instance) ⊃ 인스턴스 변수(self.x) ⊃ 로컬 변수(temp)는 포함되지 않음**

| 구분 | 위치 | 수명 | 접근 범위 | 메모리 저장 |
|---|---|---|---|---|
| 로컬 변수(local variable) | 함수 스택 | 함수 종료 시 소멸 | 그 함수 내부만 | 스택(Stack) |
| 인스턴스 변수(`self.x`) | 인스턴스의 `__dict__` | 인스턴스 살아있는 동안 유지 | 모든 메서드에서 `self.x`로 접근 | 힙(Heap) |
| 클래스 변수(`ClassName.x`) | 클래스의 `__dict__` | 프로세스 종료까지 | 모든 인스턴스 공유 | 힙(클래스 객체) |

## 왜 필요한가?

```python
class PaymentProcessor:
    def __init__(self, payment_method):
        # self.를 안 붙이면?
        payment_method = payment_method  # 로컬 변수, 함수 끝나면 사라짐

        # self.를 붙이면?
        self.payment_method = payment_method  # 인스턴스 변수, 객체에 저장됨
```

### 실제 차이

```python
class Bad:
    def __init__(self, value):
        value = value  # 로컬 변수

    def get_value(self):
        return self.value  # AttributeError! (저장 안 했으니)

class Good:
    def __init__(self, value):
        self.value = value  # 인스턴스 변수

    def get_value(self):
        return self.value  # OK
```

### 직접 확인: `__dict__`로 인스턴스 속성 보기

**설명한 것을 눈으로 확인**해보자. 파이썬의 모든 인스턴스는 자기 속성을 `__dict__`라는 내부 딕셔너리에 저장한다.

```python
class Agent:
    def __init__(self, name):
        self.name = name      # 인스턴스 변수
        temp = "초기화 완료"  # 로컬 변수

agent = Agent("리서치봇")
print(agent.__dict__)
# 출력: {'name': '리서치봇'}
# temp는 없음! 로컬 변수라 이미 증발
```

**일상 비유**: `self.`는 "이 택배 상자에 포스트잇으로 라벨 붙이기" 같은 행위. 라벨 없이 그냥 책상에 둔 물건(로컬 변수)은 퇴근하면 청소부(GC)가 치워간다. 라벨 붙여서 개인 사물함(인스턴스)에 넣으면 다음 날에도 꺼낼 수 있다.

## 언제 `self.`를 붙이나?

**규칙**: "**다른 메서드에서도 접근해야 하는 데이터**"면 `self.` 붙임

```python
class PaymentProcessor:
    def __init__(self, payment_method):
        # 1. 인스턴스 변수 (self. 필수)
        self.payment_method = payment_method  # 다른 메서드에서 사용

        # 2. 로컬 변수 (self. 안 붙임)
        temp_value = "초기화 중..."  # __init__에서만 사용
        print(temp_value)

    def process(self, amount):
        # 3. 인스턴스 변수 접근 (self. 필수)
        return self.payment_method.process(amount)

        # 4. 로컬 변수 (self. 안 붙임)
        result = "처리 완료"  # 이 메서드에서만 사용
        return result
```

## 판단 기준 치트시트

| 상황 | self. 붙일까? | 이유 |
|---|---|---|
| 다른 메서드에서 써야 함 | 붙임 | 인스턴스에 저장해야 접근 가능 |
| 외부에서 `obj.x`로 읽을 것 | 붙임 | 공개 API는 인스턴스 속성 |
| 초기화 중 1회만 쓸 임시값 | 안 붙임 | 로컬 변수로 충분 |
| for 루프 카운터 | 안 붙임 | 메서드 내부에서만 유효 |
| 자식 클래스가 오버라이드해도 접근 가능해야 함 | 붙임 | [[class_오버라이딩]] 대비 |

## 왜 이렇게 설계됐나?

**설계 의도**: 파이썬은 "명시적이 암묵적보다 낫다"(Explicit is better than implicit, PEP 20 Zen of Python) 원칙을 따른다. 자바/C++처럼 `this`를 자동 제공하지 않고, 개발자가 직접 `self`를 첫 파라미터로 받고 `self.`로 명시하게 만든 이유:

1. 인스턴스 변수인지 로컬 변수인지 **코드만 봐도 구분 가능**
2. 메서드와 일반 함수의 경계가 명확(메서드도 결국 `self`를 받는 함수일 뿐)
3. 데코레이터(`@staticmethod`, `@classmethod`)로 쉽게 변형 가능

**트레이드오프**: 매번 `self`를 쓰는 게 귀찮다. 하지만 "암묵적 this"로 인한 버그(변수 스코프 혼동)를 줄이는 쪽을 택했다.

## 실무 패턴

```python
class RealEstateAgent:
    def __init__(self, api_key: str, region: str):
        # 인스턴스 변수 (self. 붙임): 여러 메서드에서 재사용
        self.api_key = api_key
        self.region = region
        self.client = APIClient(api_key)  # 다른 메서드에서 사용

        # 로컬 변수 (self. 안 붙임): 초기화 시에만 사용
        welcome_msg = f"{region} 분석 시작"
        print(welcome_msg)

    def fetch_data(self):
        # self.client 사용 (위에서 저장했으니 접근 가능)
        data = self.client.get(f"/region/{self.region}")
        return data
```

```python
processor1 = PaymentProcessor(CreditCardPayment())
processor2 = PaymentProcessor(PayPalPayment())

# 각 인스턴스는 독립적인 self.payment_method를 가짐
processor1.payment_method  # CreditCardPayment 객체
processor2.payment_method  # PayPalPayment 객체
```

## AI Agent 개발 실무 예시

**LangGraph Agent에서 상태를 유지해야 하는 것은 `self.`, 한 번 쓰고 버리는 것은 로컬 변수.**

```python
from langchain_openai import ChatOpenAI

class ConversationalAgent:
    def __init__(self, llm: ChatOpenAI, system_prompt: str):
        # 재사용 (self. 필수)
        self.llm = llm
        self.system_prompt = system_prompt
        self.conversation_history = []  # 대화 누적
        self.tool_call_count = 0         # 툴 호출 횟수 추적

    def chat(self, user_message: str):
        # 로컬 변수 (이 메서드 안에서만 사용)
        timestamp = time.time()
        formatted_prompt = f"[{timestamp}] {user_message}"

        # self. 사용: 누적 업데이트
        self.conversation_history.append({"role": "user", "content": user_message})

        # self.llm 호출
        response = self.llm.invoke(self.conversation_history)

        self.conversation_history.append({"role": "assistant", "content": response.content})
        return response.content

    def get_stats(self):
        # 다른 메서드에서 self.conversation_history 접근 가능
        return {
            "messages": len(self.conversation_history),
            "tool_calls": self.tool_call_count,
        }
```

**포인트**:
- `self.conversation_history`는 `chat()`과 `get_stats()` 두 메서드에서 공유 → `self.` 필수
- `timestamp`, `formatted_prompt`는 `chat()` 내부에서만 써서 버려짐 → 로컬 변수

## 메서드/속성 접근 시 `self.`

**메서드 호출 시에도 `self.` 붙여야 한다** (같은 클래스 안이라도).

```python
class LoggableMixin:
    def log(self, message):  # 인스턴스 메서드
        print(f"[LOG] {message}")

class Tool(LoggableMixin):
    def run(self):
        self.log("실행")  # self. 필수! (인스턴스 메서드 호출)
        # log("실행")  ← 에러! log는 전역 함수가 아님

tool = Tool()
result = tool.run()
# 출력: [LOG] 실행
```

**왜 이렇게 되나**: 파이썬은 메서드를 함수와 동일하게 다룬다. `self.log(...)`는 실제로 `Tool.log(self, "실행")`로 변환된다. `self.`를 빼면 파이썬이 `log`라는 이름을 로컬 스코프 → 전역 스코프에서 찾는데, 거기에 없으니 `NameError`.

## 실무 주의점

### 1. `self` 대신 다른 이름 쓰지 말기

**문법적으론 첫 파라미터 이름은 자유지만, 관례상 무조건 `self`**. 다른 이름 쓰면 린터(linter)가 경고하고, 동료가 코드 리뷰에서 즉시 지적한다.

```python
class Bad:
    def method(this, value):  # 작동은 함, 하지만 관례 위반
        this.value = value
```

### 2. 클래스 변수 덮어쓰기 주의

```python
class Agent:
    count = 0  # 클래스 변수

    def __init__(self):
        self.count += 1  # 함정!
```

`self.count += 1`은 `self.count = self.count + 1`로 풀리는데, 이 순간 **인스턴스 변수 `self.count`가 새로 생기면서 클래스 변수 `Agent.count`를 가린다**. 전체 인스턴스 수를 세고 싶다면 `Agent.count += 1`로 명시해야 한다.

### 3. 가변 객체 기본값 주의

```python
class BadAgent:
    def __init__(self, history=[]):  # 위험!
        self.history = history

a1 = BadAgent()
a2 = BadAgent()
a1.history.append("hi")
print(a2.history)  # ["hi"] 공유됨!

class GoodAgent:
    def __init__(self, history=None):
        self.history = history if history is not None else []
```

## 결론

- `self.` = "이 객체의 속성으로 저장"(다른 메서드에서 접근 가능)
- `self.` 안 붙이면 = 로컬 변수(함수 끝나면 사라짐)
- 메서드 호출도 `self.메서드()` 형태로
- AI Agent에서 **상태 유지 필요 = `self.`, 임시 계산 = 로컬** 으로 구분

## 관련 문서

- [[class_인스턴스]]: `self`가 가리키는 대상(인스턴스 자체)
- [[class_`__init__`]]: `self.x = x` 패턴이 가장 많이 등장하는 곳
- [[class_파라미터]]: `__init__`의 파라미터와 `self.속성` 매핑
- [[class_타입힌트]]: `self.x: int` 형태로 속성 타입 명시
