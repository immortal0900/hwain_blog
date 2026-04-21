---
생성날짜:
- 2026-04-21 12:16
마지막수정날짜:
- 2026-04-21-화요일 12:16
tags:
- 설계원칙
- 디자인패턴
- GoF
- 생성패턴
- AI에이전트
별칭:
- Factory
- 팩토리
- Factory Method
- Abstract Factory
- Simple Factory
type:
- 자료수집
Area/Reasource:
Project:
---
# Factory 패턴이란?

Factory(팩토리, 공장 패턴, 객체 생성 로직을 별도 모듈로 옮겨 클라이언트가 `new`를 직접 부르지 않게 만드는 생성 디자인 패턴)는 **"무엇을 만들지 결정하는 책임을 따로 떼어내, 사용자는 '만들어 달라'고만 요청하는"** 패턴이다. GoF 분류상 **생성(Creational) 패턴**에 속함.

세 가지 변형이 섞여 쓰여 혼동되지만, 공통 목표는 **"객체 생성과 사용의 분리"** 하나임.

## 한눈에 보기

| 항목     | 내용                                                     |
| ------ | ------------------------------------------------------ |
| 분류     | 생성 패턴(Creational)                                      |
| 한 줄 정의 | 객체 생성 로직을 캡슐화해 사용자가 구체 클래스에 결합되지 않게 함                   |
| 핵심 구성  | Product(추상), ConcreteProduct(구현), Factory(생성 책임)       |
| 해결 문제  | `new Pinecone()` 같은 구체 의존, 조건 분기로 객체 만드는 문제, 생성 로직 중복 |
| 관련 원칙  | DIP, OCP, SRP                                          |

> **한마디 요약**: "뭘 만들지 물어보지 말고 그냥 달라고 요청해라. 결정은 공장이 한다."

---

## 일상 비유

카페에서 주문할 때를 생각해보자.

손님(클라이언트)이 "아메리카노"라고 말하면 바리스타(Factory)가 원두 종류, 추출 시간, 사이즈 같은 세부 결정을 알아서 한다. 손님은 커피 추출기 조작법을 몰라도 된다. 내부 절차가 바뀌어도(원두 공급사 교체, 기계 업그레이드) 주문 방법은 그대로 임.

다른 비유:
- 택배 주문(보내는 사람은 상자 내용만 알려주고, 송장 생성/분류/배송 결정은 택배사)
- 게임 캐릭터 생성(종족/직업만 고르면 스탯 초기화는 시스템이)
- 자동차 공장(고객은 "SUV"를 고르고, 부품 조립은 공장이)

---

## 세 가지 변형 비교

| 변형                        | 특징                         | 예시                     |
| ------------------------- | -------------------------- | ---------------------- |
| **Simple Factory**(GoF 정식 아님) | 하나의 팩토리 함수/클래스가 타입 인자로 분기 | `create_llm("openai")` |
| **Factory Method**        | 서브클래스가 어떤 Product를 만들지 결정  | 게임 스테이지별 몬스터 생성        |
| **Abstract Factory**      | 관련된 Product 군(패밀리)을 통째로 생성 | 테마별 UI 위젯 세트, DB 방언 세트 |

---

## Simple Factory (실무 기본형)

가장 간단하면서 실무에서 압도적으로 많이 쓰는 형태. GoF 원전 23개에는 없지만 "Factory"라고 하면 보통 이걸 가리킴.

### 나쁜 예: 사용자가 직접 구체 클래스 생성

```python
class RAGAgent:
    def __init__(self, backend: str):
        if backend == "pinecone":
            self.store = PineconeStore(api_key=os.getenv("PINECONE_KEY"))
        elif backend == "weaviate":
            self.store = WeaviateStore(url=os.getenv("WEAVIATE_URL"))
        elif backend == "qdrant":
            self.store = QdrantStore(host=os.getenv("QDRANT_HOST"))
```

문제:
- RAGAgent가 Pinecone, Weaviate, Qdrant API를 전부 알아야 함
- 새 벡터스토어 추가 시 RAGAgent 수정 (OCP 위반)
- 환경변수 이름/연결 파라미터가 Agent에 섞임

### 좋은 예: Factory로 분리

```python
from abc import ABC, abstractmethod

class VectorStore(ABC):
    @abstractmethod
    def upsert(self, vecs): ...
    @abstractmethod
    def search(self, query, k): ...

class PineconeStore(VectorStore): ...
class WeaviateStore(VectorStore): ...
class QdrantStore(VectorStore): ...

class VectorStoreFactory:
    @staticmethod
    def create(backend: str) -> VectorStore:
        if backend == "pinecone":
            return PineconeStore(api_key=os.getenv("PINECONE_KEY"))
        if backend == "weaviate":
            return WeaviateStore(url=os.getenv("WEAVIATE_URL"))
        if backend == "qdrant":
            return QdrantStore(host=os.getenv("QDRANT_HOST"))
        raise ValueError(f"Unknown backend: {backend}")

class RAGAgent:
    def __init__(self, store: VectorStore):
        self.store = store

# 사용
store = VectorStoreFactory.create("pinecone")
agent = RAGAgent(store)
```

### 레지스트리 Factory (권장)

`if/elif` 체인도 OCP 위반 신호다. 레지스트리(registry, 이름-구현 매핑 딕셔너리)로 개선 가능.

```python
class VectorStoreFactory:
    _registry: dict[str, type[VectorStore]] = {}

    @classmethod
    def register(cls, name: str):
        def decorator(klass):
            cls._registry[name] = klass
            return klass
        return decorator

    @classmethod
    def create(cls, name: str, **kwargs) -> VectorStore:
        if name not in cls._registry:
            raise ValueError(f"Unknown: {name}. Available: {list(cls._registry)}")
        return cls._registry[name](**kwargs)

@VectorStoreFactory.register("pinecone")
class PineconeStore(VectorStore): ...

@VectorStoreFactory.register("weaviate")
class WeaviateStore(VectorStore): ...

store = VectorStoreFactory.create("pinecone", api_key="...")
```

새 벡터스토어 추가 = 클래스 만들고 `@register` 한 줄. Factory 코드는 영구 동결됨.

---

## Factory Method (GoF 원전)

**Factory Method는 "생성 메서드를 서브클래스가 오버라이드"** 하는 구조. 부모 클래스는 "뭘 만드는지" 모르고 "언제 만드는지"만 안다.

```python
from abc import ABC, abstractmethod

class Creator(ABC):
    @abstractmethod
    def create_product(self) -> "Product": ...

    def operation(self):
        # 템플릿 메서드처럼 동작
        product = self.create_product()
        return product.do_something()

class ConcreteCreatorA(Creator):
    def create_product(self):
        return ProductA()

class ConcreteCreatorB(Creator):
    def create_product(self):
        return ProductB()
```

### AI 에이전트 예시: Agent 종류별 Tool 세트

```python
class BaseAgent(ABC):
    def __init__(self):
        self.tools = self.create_tools()  # 서브클래스가 결정

    @abstractmethod
    def create_tools(self) -> list[Tool]: ...

    def run(self, task):
        # 공통 실행 흐름
        ...

class ResearchAgent(BaseAgent):
    def create_tools(self):
        return [WebSearchTool(), SummarizeTool(), CiteTool()]

class CodingAgent(BaseAgent):
    def create_tools(self):
        return [FileReadTool(), CodeExecTool(), GitTool()]
```

BaseAgent의 `run` 흐름은 동일하고, 에이전트 종류별로 Tool 세트만 달라진다.

---

## Abstract Factory

**"관련된 객체 패밀리"를 통째로 생성**하는 Factory. 각 Factory가 "일관된 세트"를 만드는 책임을 짐.

### 예시: LLM 스택 패밀리

```python
class LLMStackFactory(ABC):
    @abstractmethod
    def create_chat(self) -> ChatModel: ...
    @abstractmethod
    def create_embedder(self) -> Embedder: ...
    @abstractmethod
    def create_tokenizer(self) -> Tokenizer: ...

class OpenAIStack(LLMStackFactory):
    def create_chat(self): return OpenAIChat()
    def create_embedder(self): return OpenAIEmbedder()
    def create_tokenizer(self): return TiktokenTokenizer()

class AnthropicStack(LLMStackFactory):
    def create_chat(self): return AnthropicChat()
    def create_embedder(self): return VoyageEmbedder()
    def create_tokenizer(self): return AnthropicTokenizer()

# 한 번 정하면 전체 스택이 맞춰짐
stack = OpenAIStack()
chat = stack.create_chat()
emb = stack.create_embedder()
```

한 스택 안에서 서로 짝이 맞는 객체들만 생성되므로 "OpenAI용 토크나이저에 Anthropic 응답 넣기" 같은 실수가 구조적으로 막힌다.

---

## 실전 AI 에이전트 개발 예시: Provider 스위칭

```python
import os
from abc import ABC, abstractmethod

class ChatModel(ABC):
    @abstractmethod
    def chat(self, messages, **kwargs) -> str: ...

class OpenAIChat(ChatModel):
    def __init__(self, model="gpt-4o-mini"):
        self.client = OpenAI()
        self.model = model
    def chat(self, messages, **kwargs):
        return self.client.chat.completions.create(
            model=self.model, messages=messages, **kwargs
        ).choices[0].message.content

class AnthropicChat(ChatModel):
    def __init__(self, model="claude-sonnet-4-6"):
        self.client = Anthropic()
        self.model = model
    def chat(self, messages, **kwargs):
        return self.client.messages.create(
            model=self.model, messages=messages, **kwargs
        ).content[0].text

class LLMFactory:
    @staticmethod
    def create(provider: str = None) -> ChatModel:
        provider = provider or os.getenv("LLM_PROVIDER", "openai")
        if provider == "openai":
            return OpenAIChat(model=os.getenv("OPENAI_MODEL", "gpt-4o-mini"))
        if provider == "anthropic":
            return AnthropicChat(model=os.getenv("ANTHROPIC_MODEL", "claude-sonnet-4-6"))
        raise ValueError(provider)

# 어디서든 동일한 호출
llm = LLMFactory.create()
answer = llm.chat([{"role": "user", "content": "hi"}])
```

프로덕션/개발/테스트에서 환경변수만 바꾸면 LLM 공급자가 통째로 교체됨.

---

## 언제 쓰면 좋은가

1. **구체 클래스 선택을 런타임/설정으로 결정**하고 싶을 때
2. 생성 로직이 **복잡하거나 반복**될 때 (설정 파일 읽기, 인증, 연결 풀 초기화 등)
3. 생성될 객체 **타입이 앞으로 늘어날 것**이 예상될 때
4. 관련된 객체 패밀리를 **일관되게 묶어야** 할 때 (Abstract Factory)

## 트레이드오프

| 장점                     | 단점                                  |
| ---------------------- | ----------------------------------- |
| 사용자가 구체 클래스 몰라도 됨      | 추상화 계층이 늘어 코드량 증가                   |
| 생성 로직 한 곳에 모여 유지보수 쉬움   | 간단한 생성에는 과한 오버엔지니어링               |
| DIP 자연스럽게 실현           | Factory가 "신(god) 객체"로 비대해질 위험         |
| 새 Product 추가 시 사용자 수정 없음 | Simple Factory는 내부 if 체인 확장 시 OCP 위반 소지 |

---

## 의존성 주입과의 관계

Factory가 필요한 상황 중 상당수는 **DI 컨테이너(Dependency Injection Container, 객체 생성/조립을 설정 기반으로 자동 처리하는 프레임워크)** 로 대체 가능함. Python이면 `dependency-injector`, `punq`, `inject`, FastAPI의 `Depends` 같은 것들.

작은 프로젝트는 Factory로 충분, 큰 프로젝트는 DI 컨테이너 고려.

---

## 실전 주의점

### 1. Factory를 남발하지 말 것
`new SomethingFactory().create()` 가 그저 `new Something()`의 포장이라면 필요 없다. **분기/설정/조합이 있을 때만** Factory의 값어치가 있음.

### 2. Factory 자체의 테스트
Factory에서 나오는 객체가 올바른 타입, 올바른 초기 상태인지 확인하는 단위 테스트를 따로 둘 것.

### 3. Abstract Factory의 폭발
Product가 N개, Factory가 M개면 N * M개의 구체 클래스가 생긴다. 늘어나는 속도가 빨라 관리 부담 커짐. 작게 시작할 것.

---

## 다른 패턴과의 관계 (포함 관계)

- **[[SOLID원칙|DIP]]와 한 몸**: 추상 인터페이스 의존을 실현하려면 Factory가 필요
- **[[Singleton 패턴|Singleton]]과 자주 결합**: Factory 자체를 Singleton으로 만들어 전역 접근
- **[[Strategy 패턴|Strategy]]와 조합**: 어떤 Strategy를 쓸지 Factory가 결정
- **Builder와 구분**: Builder는 "단계별로 복잡한 객체 하나 짓기", Factory는 "어떤 타입을 만들지 고르기"
- **Prototype과 구분**: Prototype은 "기존 객체 복제", Factory는 "새로 만들기"

---

## 직접 확인하기

자기 코드에서 `Class(...)` 직접 호출이 여러 곳에 퍼져 있고, 생성 인자가 중복되거나 환경에 따라 바뀐다면 Factory 후보임. `grep -n "PineconeStore(" .` 같이 하나의 구체 클래스가 몇 군데서 직접 생성되는지 세어볼 것. 3군데 이상이면 경보.

---

## 요약

> **"어떤 물건을 만들지 판단하는 책임을 공장에 넘겨라. 사용자는 '필요하다'만 말하면 된다."**

관련 문서:
- [[SOLID원칙]]
- [[Strategy 패턴]]
- [[Singleton 패턴]]
