---
생성날짜:
- 2026-01-28 20:02
마지막수정날짜:
- 2026-01-28-수요일 20:01
tags:
- 설계원칙
- SOLID
- OCP
- DIP
- AI에이전트
별칭:
- OCP vs DIP
type:
- 자료수집
Area/Reasource:
Project:
---
# OCP와 DIP의 차이

SOLID 중에서도 OCP(Open-Closed Principle)와 DIP(Dependency Inversion Principle)는 코드에서 증상이 비슷하게 보여서 헷갈리기 쉽다. 같은 리팩토링(추상화 + 전략 패턴)으로 둘 다 해결되는 경우가 많기 때문이다. 그래도 **질문 각도가 다르다**.

상위 개념은 [[SOLID원칙]], 의존성 역전 심화는 [[고수준 모듈과 저수준 모듈(SOLID DIP)]] 참고.

---

## 한 줄 정의 복습

| 원칙  | 한국어       | 한 줄 정의                            |
| --- | --------- | --------------------------------- |
| OCP | 개방 폐쇄 원칙  | 확장에는 열려 있고, 수정에는 닫혀 있어야 한다        |
| DIP | 의존성 역전 원칙 | 고수준도 저수준도 둘 다 추상화에 의존해야 한다        |

---

## O: Open-Closed Principle (개방 폐쇄 원칙)

**"확장에는 열려있고, 수정에는 닫혀있어야 한다."**

기능을 추가할 때 **기존 클래스를 수정하지 말고, 새 클래스를 얹어서** 해결하는 것이 목표이다.

### OCP 위반 신호

```python
# if-elif 체인이 계속 늘어남
def route(msg_type, payload):
    if msg_type == "chat":
        return handle_chat(payload)
    elif msg_type == "embed":
        return handle_embed(payload)
    elif msg_type == "moderate":   # ← 추가할 때마다 함수 수정
        return handle_moderate(payload)
```

- 새 타입이 생길 때마다 기존 함수 수정
- 기존 테스트가 영향을 받음
- merge conflict 자석

---

## D: Dependency Inversion Principle (의존성 역전 원칙)

**"고수준 모듈이 저수준 모듈에 의존하지 말고, 둘 다 추상화에 의존해야 한다."**

클래스가 어떤 "도구"를 사용할지 자기 안에서 new 하지 않고, **외부에서 주입받는다**.

### DIP 위반 신호

```python
# 클래스 내부에서 직접 생성
class Service:
    def __init__(self):
        self.db = MySQLDatabase()       # ← 구체 클래스 직접 생성
        self.llm = OpenAIClient(api_key=os.getenv("OPENAI_KEY"))  # ← 환경에 강결합
```

- 구체 클래스 이름이 `__init__` 안에 박혀 있음
- 테스트할 때 진짜 DB/진짜 LLM을 써야 함 (비용 + 속도 + 불안정)
- DB나 LLM Provider 교체 = 클래스 내부 수정

---

## 둘을 동시에 해결하는 리팩토링: 결제 예시

```python
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        pass

class CreditPayment(PaymentStrategy):
    def process(self, amount):
        return f"신용카드: {amount}원"

class PayPalPayment(PaymentStrategy):
    def process(self, amount):
        return f"PayPal: {amount}원"

class KakaoPayPayment(PaymentStrategy):
    def process(self, amount):
        return f"카카오페이: {amount}원"

# OCP + DIP 둘 다 준수
class PaymentProcessor:
    def __init__(self, strategy: PaymentStrategy):  # DIP: 추상화 의존
        self.strategy = strategy

    def process(self, amount):
        return self.strategy.process(amount)        # OCP: 확장 가능

# 실행
print("=== 신용카드 ===")
p1 = PaymentProcessor(CreditPayment())
print(p1.process(1000))  # 신용카드: 1000원

print("\n=== PayPal ===")
p2 = PaymentProcessor(PayPalPayment())
print(p2.process(2000))  # PayPal: 2000원

print("\n=== 카카오페이 (새로 추가) ===")
p3 = PaymentProcessor(KakaoPayPayment())
print(p3.process(3000))  # 카카오페이: 3000원
# PaymentProcessor 코드 수정 없이 추가!
```

---

## AI Agent Developer 관점: LLM Provider 예시

에이전트 개발에서 가장 자주 맞닥뜨리는 상황이다. OpenAI → Anthropic → Gemini 옮겨다니거나, 여러 Provider를 A/B 테스트하거나, 테스트 시에는 가짜 응답을 써야 할 때.

### 위반 버전

```python
import openai

class ChatAgent:
    def __init__(self):
        self.client = openai.OpenAI(api_key=os.getenv("OPENAI_KEY"))  # DIP 위반

    def ask(self, provider: str, question: str):
        # OCP 위반: 새 provider 추가마다 수정
        if provider == "openai":
            resp = self.client.chat.completions.create(
                model="gpt-4o", messages=[{"role":"user","content":question}]
            )
            return resp.choices[0].message.content
        elif provider == "anthropic":
            import anthropic
            client = anthropic.Anthropic(api_key=os.getenv("ANTHROPIC_KEY"))
            resp = client.messages.create(
                model="claude-opus-4-7",
                messages=[{"role":"user","content":question}],
                max_tokens=1024,
            )
            return resp.content[0].text
        elif provider == "gemini":
            ...
```

문제:
- Provider 추가할 때마다 `ask` 메서드 수정 (OCP 위반)
- 클래스 안에서 직접 클라이언트 생성 (DIP 위반)
- 테스트에서 가짜 응답을 끼우기 어려움

### 개선 버전

```python
from abc import ABC, abstractmethod

class ChatModel(ABC):
    @abstractmethod
    def generate(self, prompt: str) -> str: ...

class OpenAIChat(ChatModel):
    def __init__(self, client, model="gpt-4o"):
        self.client = client
        self.model = model
    def generate(self, prompt):
        resp = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role":"user","content":prompt}],
        )
        return resp.choices[0].message.content

class AnthropicChat(ChatModel):
    def __init__(self, client, model="claude-opus-4-7"):
        self.client = client
        self.model = model
    def generate(self, prompt):
        resp = self.client.messages.create(
            model=self.model,
            messages=[{"role":"user","content":prompt}],
            max_tokens=1024,
        )
        return resp.content[0].text

class FakeChat(ChatModel):
    """테스트용"""
    def __init__(self, canned_response: str):
        self.canned = canned_response
    def generate(self, prompt):
        return self.canned

class ChatAgent:
    def __init__(self, model: ChatModel):  # DIP: 추상화 의존
        self.model = model

    def ask(self, question: str) -> str:
        return self.model.generate(question)  # OCP: 새 Provider 추가해도 여기 안 건드림

# 실행 환경별 주입만 바꿈
prod  = ChatAgent(AnthropicChat(anthropic.Anthropic(api_key=...)))
test  = ChatAgent(FakeChat("항상 같은 응답"))
local = ChatAgent(OpenAIChat(openai.OpenAI(api_key=...), model="gpt-4o-mini"))
```

Provider 하나 더 붙이고 싶으면? `ChatModel`을 구현한 새 클래스만 만들고 주입하면 된다. `ChatAgent` 코드는 한 글자도 안 바뀐다.

---

## 정리표

| | **OCP**                  | **DIP**                  |
| --- | ------------------------ | ------------------------ |
| **질문** | 새 기능 추가 시?               | 의존 대상은?                  |
| **나쁨** | 기존 코드 수정                 | 구체 클래스 의존                |
| **좋음** | 새 클래스 추가                 | 추상화 의존 + 외부 주입           |
| **키워드** | 확장, 수정                   | 고수준, 저수준, 추상             |
| **예시** | if-elif 체인 제거            | 내부 `new` 제거              |
| **목적** | 변경 파급 범위 축소              | 결합도 낮춤, 테스트/교체 용이        |
| **리팩토링 수단** | 다형성 (Strategy, Template) | 생성자 주입, 팩토리, DI 컨테이너     |

---

## 둘의 관계 (포함 관계)

- **OCP를 달성하려면 보통 DIP가 필요함**: 추상화 없이는 "새 클래스 갖다 끼우기" 자체가 불가능하다
- 하지만 **목적이 다름**: OCP는 "확장성(수정 없이 기능 추가)", DIP는 "의존 방향(고수준이 저수준에 끌려다니지 않기)"
- **DIP만 있고 OCP가 안 보이는 경우**: 의존성 주입은 했지만 분기 체인이 여전히 있는 경우 (DIP ○, OCP ×)
- **OCP만 있고 DIP가 안 보이는 경우**: 다형성으로 확장은 되지만, 생성은 여전히 구현체 안에서 직접 하는 경우 (OCP ○, DIP △)

```
     [DIP: 추상에 의존]
            │
            │ 이게 있어야
            ▼
     [OCP: 수정 없이 확장]
```

---

## 쉬운 구분법

| 증상                        | 어떤 원칙 문제? |
| ------------------------- | --------- |
| 새 기능 넣을 때마다 기존 함수 `if/elif` 추가 | **OCP**   |
| 테스트할 때 진짜 API/DB 안 쓰면 안 됨      | **DIP**   |
| 클래스 안에서 `OpenAI()` 같은 new를 하고 있음 | **DIP**   |
| `handler_type` 같은 분기 파라미터가 늘어남 | **OCP**   |
| Provider 스위치가 `main.py` 꼭대기 `if`로 있음 | **OCP + DIP 둘 다** |

---

## 직접 확인하기

내가 짠 에이전트 코드에서 이 두 개만 찾아봐라.

1. `grep -rn "if provider" src/` 또는 `grep -rn "elif.*type" src/` : 분기 체인 흔적 (OCP 체크)
2. `grep -rn "openai.OpenAI(" src/` 또는 `grep -rn "anthropic.Anthropic(" src/` : 구체 클라이언트 직접 생성 (DIP 체크)

두 검색 결과가 비즈니스 로직 쪽 파일에 있으면 리팩토링 후보이다. Factory/Composition Root(main.py, DI 설정 파일) 같은 "조립 전용" 모듈에만 있으면 건강한 상태이다.

---

## 한마디 요약

> **OCP**는 "새 기능 추가할 때 기존 코드 건드리지 마라", **DIP**는 "구체적인 것 말고 약속(추상)에 의존해라". DIP를 지켜야 OCP가 자연스럽게 따라온다.

관련 문서:
- [[SOLID원칙]]
- [[고수준 모듈과 저수준 모듈(SOLID DIP)]]
