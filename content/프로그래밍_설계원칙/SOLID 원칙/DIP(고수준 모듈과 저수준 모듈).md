---
생성날짜:
- 2026-01-28 04:35
마지막수정날짜:
- 2026-01-28-수요일 04:34
tags:
- 설계원칙
- SOLID
- DIP
- 의존성주입
- AI에이전트
별칭:
- DIP
- 의존성 역전 원칙
type:
- 자료수집
Area/Reasource:
Project:
---
# 고수준 모듈과 저수준 모듈 (SOLID - DIP)

SOLID의 마지막 글자 D에 해당하는 DIP(Dependency Inversion Principle, 의존성 역전 원칙)를 정리한다. SOLID 전체는 [[SOLID원칙]], OCP와의 차이는 [[SOLID_ O와 D의 차이]] 참고.

## 용어 정리 (포함 관계)

| 용어 (영문 원어) | 한 줄 풀이 | 무엇 | AI 에이전트 예시 |
| --- | --- | --- | --- |
| **고수준 모듈 (High-level Module)** | 정책·의사결정 코드 | 비즈니스 로직, "왜"를 담당 | "사용자 질문에 답하는 에이전트", "결제 처리", "주문 처리" |
| **저수준 모듈 (Low-level Module)** | 구체 동작 코드 | 구체 구현, "어떻게"를 담당 | OpenAI SDK 호출, PostgreSQL 쿼리, Pinecone API |
| **추상화 (Abstraction / Interface)** | 약속(메서드 시그니처) 묶음 | 고수준이 요구하는 계약 | `ChatModel`, `VectorStore`, `Memory` ABC 클래스 |
| **의존성 주입 (Dependency Injection, DI)** | 외부에서 객체를 넣어 주는 기법 | DIP를 실제로 적용하는 수단 | `__init__(model: ChatModel)`, FastAPI `Depends`, `dependency-injector` |
| **제어 역전 (Inversion of Control, IoC)** | 객체 생성·연결의 주도권을 외부로 넘김 | DIP의 상위 개념 | DI 컨테이너, 프레임워크 콜백 구조 |

포함 관계로 보면:
```
 고수준 모듈  ──의존──▶  추상화 인터페이스  ◀──구현──  저수준 모듈
                              ▲
                              │ 외부에서 주입(DI)
                              │
                       Composition Root
```

고수준과 저수준이 **같은 추상화를 바라보고** 있어서 직접 서로를 모른다. 이것이 '의존성 역전'의 정체이다. DIP는 원칙(원칙: 추상에 의존), DI는 그 원칙을 적용하는 기법(주입 행위), IoC는 더 큰 개념(객체 결합 주도권의 역전)이라는 3층 구조로 본다.

---

# DIP (Dependency Inversion Principle)

## 핵심 개념

> **"고수준 모듈은 저수준 모듈에 의존하지 말고, 둘 다 추상화에 의존해야 한다."**

일상 비유로 표현하면, **전기 콘센트(인터페이스)** 와 **가전 제품(구체 구현)** 의 관계이다. 벽(고수준: 건물 전력 시스템)은 "220V 둥근 두 구멍"이라는 약속만 노출하고, 제품(저수준: 청소기, 드라이기, 노트북 충전기)이 거기에 맞춘다. 벽이 "청소기 전용 구멍"과 "드라이기 전용 구멍"을 따로 가지면 새 제품 나올 때마다 벽을 뚫어야 한다.

---

## 왜 '역전'인가?

자연스러운 흐름은 "고수준이 필요한 저수준을 직접 고른다"이다. DIP를 적용하면 고수준이 **인터페이스를 먼저 정의**하고, 저수준이 거기에 맞춰 만들어진다. 결과적으로 의존 방향이 뒤집힌다.

```
[자연스러운 흐름 - 위반]
OrderService  ──의존──▶  MySQLDatabase
(고수준)                  (저수준)

[역전된 흐름 - DIP 준수]
OrderService  ──의존──▶  Database(인터페이스)
                              ▲
                              │ 구현
                        MySQLDatabase
                        (저수준이 인터페이스에 맞춤)
```

주도권이 '도구'에서 '사용자(고수준)'로 넘어간다.

---

# 나쁜 예시 (DIP 위반)

## 패턴 1: 클래스 내부에서 직접 객체 생성

```python
class PaymentService:
    def __init__(self):
        self.payment = CreditCardPayment()  # ❌ 내부 생성
    
    def process(self, amount):
        return self.payment.process(amount)

service = PaymentService()
result = service.process(100)
print(result)  # 신용카드 결제 100원
```

**문제점**:
1. PayPal로 바꾸려면 `PaymentService` 코드 수정 필요
2. 테스트할 때 진짜 결제가 일어남 (가짜 결제로 테스트 불가)
3. `PaymentService`가 `CreditCardPayment`에 강하게 결합

## 패턴 2: 구체 클래스에 직접 의존

```python
class OrderProcessor:
    def __init__(self):
        self.db = MySQLDatabase()        # ❌ MySQL에 강결합
        self.notifier = EmailNotifier()  # ❌ Email에 강결합
    
    def process_order(self, order):
        self.db.save(order)
        self.notifier.send(f"주문 완료: {order}")

# PostgreSQL로 바꾸고 싶다면?  →  OrderProcessor 코드 수정
# Slack 알림을 추가하고 싶다면?  →  OrderProcessor 코드 수정
```

## 패턴 3: 하드코딩된 의존성

```python
class ReportGenerator:
    def generate(self):
        db = MySQLDatabase("localhost", "root", "password")  # ❌ 하드코딩
        data = db.query("SELECT * FROM reports")
        return data

# 테스트 환경에서는?  →  코드 수정해야 함
# 다른 DB 쓰고 싶다면?  →  코드 수정해야 함
```

---

# 좋은 예시 (DIP + 의존성 주입)

## 추상화에 의존 + 외부 주입

```python
from abc import ABC, abstractmethod

# 1. 추상화 정의
class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        pass

# 2. 구체 클래스들 (저수준)
class CreditCardPayment(PaymentStrategy):
    def process(self, amount):
        return f"신용카드 결제: {amount}원"

class PayPalPayment(PaymentStrategy):
    def process(self, amount):
        return f"PayPal 결제: {amount}원"

class KakaoPayPayment(PaymentStrategy):
    def process(self, amount):
        return f"카카오페이 결제: {amount}원"

# 3. 고수준 모듈 (의존성 주입)
class PaymentService:
    def __init__(self, payment: PaymentStrategy):  # ← 외부에서 주입
        self.payment = payment
    
    def process(self, amount):
        return self.payment.process(amount)

# 실행
print("=== 신용카드 ===")
service1 = PaymentService(CreditCardPayment())
print(service1.process(100))  # 신용카드 결제: 100원

print("\n=== PayPal ===")
service2 = PaymentService(PayPalPayment())
print(service2.process(200))  # PayPal 결제: 200원

print("\n=== 카카오페이 ===")
service3 = PaymentService(KakaoPayPayment())
print(service3.process(300))  # 카카오페이 결제: 300원

# PaymentService 코드 수정 없이 결제 수단 변경!
```

---

## 위반에서 준수로 가는 4단계 리팩토링

DIP를 어디서부터 적용해야 하는지 막막할 때 쓸 수 있는 단계 분해이다. 결제 예시를 그대로 써서 단계별로 어떻게 코드가 바뀌는지 추적한다.

| 단계 | 무엇을 하는가 | 코드에 일어나는 일 |
| --- | --- | --- |
| **1단계** | 구체 클래스 사용 지점 식별 | `self.payment = CreditCardPayment()` 같은 직접 생성 줄을 모두 찾음 |
| **2단계** | 인터페이스(추상 타입) 정의 | `class PaymentStrategy(ABC)`로 메서드 시그니처만 뽑아 약속을 만듦 |
| **3단계** | 구체 클래스를 인터페이스 구현체로 변환 | `class CreditCardPayment(PaymentStrategy)`처럼 상속/구현 선언 추가 |
| **4단계** | 생성자 인자로 인터페이스 주입 | `def __init__(self, payment: PaymentStrategy):`로 외부에서 받게 함 |

각 단계가 끝났을 때 코드가 어떤 상태인지 확인할 수 있다.

```
[1단계 종료]  생성 위치 N개를 메모해 둔 상태. 코드는 그대로
[2단계 종료]  ABC 클래스 1개 추가. 기존 코드는 아직 그대로
[3단계 종료]  구체 클래스가 ABC를 상속. 호출부는 아직 그대로
[4단계 종료]  호출부의 `new` 제거, Composition Root에서만 생성
```

**조립 위치**(어디서 구체 클래스를 생성하는가)는 4단계 이후 자연스럽게 `main.py`나 `bootstrap.py` 같은 진입점 한 곳으로 모인다.

---

# AI 에이전트 개발에서의 DIP

AI Agent Developer로서 DIP가 가장 크게 효과를 내는 4가지 지점이다.

## 1. LLM Provider 추상화

여러 Provider(OpenAI, Anthropic, Gemini, Groq, 로컬 Ollama)를 갈아 끼우거나 A/B 테스트할 때 필수.

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class Message:
    role: str   # "system", "user", "assistant"
    content: str

class ChatModel(ABC):
    @abstractmethod
    def generate(self, messages: list[Message], **kwargs) -> str: ...

class OpenAIChat(ChatModel):
    def __init__(self, client, model: str):
        self.client = client
        self.model = model
    def generate(self, messages, **kwargs):
        resp = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": m.role, "content": m.content} for m in messages],
            **kwargs,
        )
        return resp.choices[0].message.content

class AnthropicChat(ChatModel):
    def __init__(self, client, model: str, max_tokens: int = 1024):
        self.client = client
        self.model = model
        self.max_tokens = max_tokens
    def generate(self, messages, **kwargs):
        system = next((m.content for m in messages if m.role == "system"), None)
        user_msgs = [{"role": m.role, "content": m.content} for m in messages if m.role != "system"]
        resp = self.client.messages.create(
            model=self.model,
            system=system,
            messages=user_msgs,
            max_tokens=self.max_tokens,
            **kwargs,
        )
        return resp.content[0].text

class FakeChat(ChatModel):
    """단위 테스트용"""
    def __init__(self, response: str):
        self.response = response
    def generate(self, messages, **kwargs):
        return self.response

# 고수준 모듈
class ResearchAgent:
    def __init__(self, model: ChatModel):
        self.model = model

    def research(self, topic: str) -> str:
        return self.model.generate([
            Message("system", "너는 조사 전문가다."),
            Message("user", f"{topic}에 대해 5줄로 요약해줘."),
        ])
```

## 2. Vector Store 추상화 (RAG)

Pinecone → Weaviate → Qdrant → pgvector 이사를 6개월에 한 번씩 하는 게 업계 표준 이벤트이다.

```python
class VectorStore(ABC):
    @abstractmethod
    def upsert(self, id: str, vector: list[float], metadata: dict) -> None: ...
    @abstractmethod
    def search(self, query: list[float], k: int = 5) -> list[dict]: ...

class PineconeStore(VectorStore): ...
class QdrantStore(VectorStore): ...
class InMemoryStore(VectorStore):
    """테스트 및 로컬 개발용"""
    def __init__(self):
        self.data = {}
    def upsert(self, id, vector, metadata):
        self.data[id] = (vector, metadata)
    def search(self, query, k=5):
        import numpy as np
        items = [(id, np.dot(query, v), meta) for id, (v, meta) in self.data.items()]
        items.sort(key=lambda x: -x[1])
        return [{"id": i, "score": s, "metadata": m} for i, s, m in items[:k]]
```

## 3. Memory Backend 추상화

에이전트의 대화 기록을 어디에 저장할지(Redis, Postgres, SQLite, DynamoDB) 바꿔치기 가능.

```python
class Memory(ABC):
    @abstractmethod
    def append(self, session_id: str, message: Message) -> None: ...
    @abstractmethod
    def history(self, session_id: str, limit: int = 20) -> list[Message]: ...

class RedisMemory(Memory): ...
class SQLiteMemory(Memory): ...
class InMemory(Memory):
    def __init__(self):
        self.store: dict[str, list[Message]] = {}
    def append(self, session_id, message):
        self.store.setdefault(session_id, []).append(message)
    def history(self, session_id, limit=20):
        return self.store.get(session_id, [])[-limit:]
```

## 4. Tool Execution 추상화

외부 부수 효과(파일 IO, HTTP 호출, Shell 명령)를 추상화하면 실수로 `rm -rf`가 실행되는 악몽을 막을 수 있다.

```python
class ToolRunner(ABC):
    @abstractmethod
    def run(self, tool_name: str, args: dict) -> str: ...

class ProductionToolRunner(ToolRunner): ...
class SandboxedToolRunner(ToolRunner):
    """개발/테스트에서 안전장치 적용"""
    BLOCKED = {"rm", "drop_table", "send_email"}
    def __init__(self, inner: ToolRunner):
        self.inner = inner
    def run(self, tool_name, args):
        if tool_name in self.BLOCKED:
            return f"[DRY RUN] {tool_name}({args}) blocked"
        return self.inner.run(tool_name, args)
```

## 5. Embedding Model 추상화 (RAG 입력 단)

벡터 검색의 정확도는 **임베딩 모델**(Embedding Model: 텍스트를 고정 길이 숫자 벡터로 변환하는 신경망)에 좌우된다. OpenAI `text-embedding-3-small`로 시작했다가 비용/성능 트레이드오프 때문에 Cohere `embed-multilingual-v3`나 BGE 로컬 모델로 갈아 끼우는 일이 흔하다.

```python
class Embedder(ABC):
    @abstractmethod
    def embed(self, text: str) -> list[float]: ...
    @abstractmethod
    def dimension(self) -> int: ...   # 1536, 1024, 768 등 모델별 차원

class OpenAIEmbedder(Embedder):
    def __init__(self, client, model="text-embedding-3-small"):
        self.client, self.model = client, model
    def embed(self, text):
        return self.client.embeddings.create(model=self.model, input=text).data[0].embedding
    def dimension(self):
        return 1536

class LocalBGEEmbedder(Embedder):
    """sentence-transformers로 로컬 임베딩"""
    def __init__(self, model_name="BAAI/bge-small-en-v1.5"):
        from sentence_transformers import SentenceTransformer
        self.model = SentenceTransformer(model_name)
    def embed(self, text):
        return self.model.encode(text).tolist()
    def dimension(self):
        return 384
```

`RAGPipeline` 같은 고수준 모듈은 `Embedder`만 알면 된다. 모델을 갈아 끼울 때 비즈니스 로직 코드는 한 줄도 안 바뀐다.

## 6. Observability(관측 가능성) 추상화

LLM 호출은 비싸고 느리다. 그래서 **Tracer**(Tracer: 호출의 입출력·지연·토큰 사용량을 기록하는 도구)와 **TokenCounter**(요청별 토큰 사용량 집계기)가 필수다. Langfuse, LangSmith, OpenTelemetry, 자체 로거 사이를 갈아 끼우려면 추상화가 답이다.

```python
class Tracer(ABC):
    @abstractmethod
    def start_span(self, name: str, **attrs): ...
    @abstractmethod
    def end_span(self, span, output=None, error=None): ...

class LangfuseTracer(Tracer): ...
class OTELTracer(Tracer): ...
class NoopTracer(Tracer):
    """관측이 꺼진 환경(로컬 디버그)에서 사용"""
    def start_span(self, name, **attrs): return None
    def end_span(self, span, output=None, error=None): return None

# 고수준은 Tracer 인터페이스만 본다
class TracedAgent:
    def __init__(self, model: ChatModel, tracer: Tracer):
        self.model, self.tracer = model, tracer

    def ask(self, question: str) -> str:
        span = self.tracer.start_span("agent.ask", question=question)
        try:
            answer = self.model.generate([Message("user", question)])
            self.tracer.end_span(span, output=answer)
            return answer
        except Exception as e:
            self.tracer.end_span(span, error=str(e))
            raise
```

비용이 부담되는 환경에서는 `NoopTracer`로 끄고, 운영에서는 `LangfuseTracer`를 주입한다. 고수준 코드는 무엇을 쓰는지 모른다.

---

## 조립 지점: Composition Root

DIP를 지키면 "누가 구체 클래스를 만들어서 주입하느냐"라는 질문이 남는다. 답은 **Composition Root** 라고 부르는 프로그램 진입점 한 곳에서만 구체 클래스를 조립하는 것이다.

```python
# main.py (Composition Root)
def build_agent(env: str) -> ResearchAgent:
    if env == "prod":
        import anthropic
        client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
        return ResearchAgent(AnthropicChat(client, "claude-opus-4-7"))
    elif env == "test":
        return ResearchAgent(FakeChat("테스트 응답"))
    elif env == "local":
        import openai
        client = openai.OpenAI(api_key=os.environ["OPENAI_API_KEY"])
        return ResearchAgent(OpenAIChat(client, "gpt-4o-mini"))
    else:
        raise ValueError(env)

if __name__ == "__main__":
    agent = build_agent(os.environ.get("ENV", "local"))
    print(agent.research("양자컴퓨팅"))
```

나머지 비즈니스 로직 파일은 `OpenAI`, `Anthropic`, `Pinecone` 같은 이름을 **전혀 import하지 않는다**. 이게 DIP를 지킨 상태의 특징이다.

---

## DIP 체크리스트

내 코드를 살펴보며 스스로 답해보자.

- [ ] 비즈니스 로직 파일이 `import openai`, `import pinecone` 같은 걸 직접 하고 있나? (있으면 DIP 위반)
- [ ] 클래스 `__init__`에서 구체 클래스를 `new`하고 있나? (`self.db = MySQLDatabase()`)
- [ ] 테스트 시 진짜 LLM/DB를 써야 하나? (Fake/Stub 주입이 안 되면 DIP 위반 신호)
- [ ] Provider 교체하려면 몇 개 파일을 수정해야 하나? (1개가 아니면 위험 신호)

---

## 직접 확인하기 (자가 진단 grep)

리포지토리 루트에서 다음 명령으로 1분 안에 DIP 위반 후보를 추출할 수 있다. macOS/Linux는 `grep -rn`, Windows PowerShell은 `Select-String -Pattern ... -Path src\* -Recurse` 형태로 바꿔서 쓴다.

```bash
# 1) 비즈니스 로직 폴더에서 SDK 직수입을 찾는다 (DIP 위반 신호)
grep -rn "^import openai\|^from openai\|import anthropic\|from anthropic" src/agents/

# 2) __init__ 안에서 구체 클라이언트를 직접 생성하는 줄을 찾는다
grep -rn "self\.\(client\|llm\|db\|store\)\s*=\s*[A-Z]" src/

# 3) Provider 분기(if provider == "openai") 흔적
grep -rn "if provider\s*==" src/
grep -rn "elif.*type\s*==" src/
```

**해석 기준**:
- `src/agents/` 같은 비즈니스 로직 디렉터리에서 1번이 잡히면 → 추상화 후보
- 2번이 잡히면 → 생성자 주입 대상
- 3번이 잡히면 → [[SOLID원칙|OCP]]까지 같이 깨지고 있을 가능성 큼

진입점(`main.py`, `bootstrap.py`, `composition_root.py`)이나 DI 설정 파일에서만 잡히는 것은 **건강한 상태**이다. 거기서만 구체 클래스를 알아야 한다.

---

## 의존성 주입 방법 3가지

| 방식                   | 문법                                    | 장점                  | 단점                  |
| -------------------- | ------------------------------------- | ------------------- | ------------------- |
| **생성자 주입 (권장)**      | `def __init__(self, dep):`            | 불변, 테스트 쉬움, 누락 시 즉시 실패 | 생성자 인자가 많아질 수 있음    |
| **세터 주입**            | `obj.set_dep(dep)`                    | 런타임 교체 가능           | 미설정 상태 가능, 깜빡하기 쉬움  |
| **메서드 주입**           | `def run(self, dep):`                 | 사용 시점에만 의존          | 호출자가 매번 넘겨야 함       |

파이썬에서는 생성자 주입이 기본이다. 더 복잡해지면 DI 컨테이너(예: `dependency-injector`, `punq`, FastAPI의 `Depends`)를 쓸 수도 있지만, 작은 에이전트는 생성자 주입만으로 충분하다.

---

## 트레이드오프: 언제 DIP를 "안" 써도 되는가

DIP는 강력하지만 무료가 아니다. 추상화 레이어를 한 겹 깐다는 것은 **읽는 사람이 따라가야 할 점프 횟수가 늘어난다**는 뜻이다. 다음 상황에서는 일부러 DIP를 적용하지 않는 편이 깔끔하다.

| 상황 | 추천 | 이유 |
| --- | --- | --- |
| 1회용 스크립트, PoC, 노트북 코드 | **적용 X** | 오버엔지니어링, 유지보수 대상이 아님 |
| 표준 라이브러리(예: `json`, `re`, `pathlib`) | **적용 X** | 안정적이고 거의 교체되지 않음 |
| 구현체가 영원히 1개일 게 확실 | 적용 X | 가짜 추상이 더 혼란스러움 |
| LLM Provider, DB, Vector Store | **적용 O** | 갈아 끼움/테스트 대체가 잦음 |
| 외부 부수 효과(파일/HTTP/Shell) | **적용 O** | 테스트와 안전장치(Sandbox) 필요 |
| 비용·과금이 발생하는 호출 | **적용 O** | Fake로 우회해야 단위 테스트 가능 |

핵심 질문은 **"이 의존성을 6개월 안에 갈아 끼울 가능성이 있는가, 또는 테스트에서 가짜로 바꿔야 하는가?"** 이다. 둘 중 하나라도 "그렇다"면 추상화 비용이 회수된다. 둘 다 "아니다"면 그냥 직접 호출이 더 정직한 코드이다.

---

## 한마디 요약

> **"내가 만든 것(고수준)이 남이 만든 것(저수준)에 끌려다니지 않게, 내가 먼저 약속(인터페이스)을 정하고 남이 맞추게 하라."**

관련 문서:
- [[SOLID원칙]]
- [[SOLID_ O와 D의 차이]]
- [[역할별 인터페이스 분리(SOLID ISP)]]
