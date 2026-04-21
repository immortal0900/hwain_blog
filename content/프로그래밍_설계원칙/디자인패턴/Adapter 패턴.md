---
생성날짜:
- 2026-04-21 13:05
마지막수정날짜:
- 2026-04-21-화요일 13:05
tags:
- 설계원칙
- 디자인패턴
- 구조패턴
- AI에이전트
별칭:
- Adapter
- 어댑터 패턴
- Wrapper
type:
- 자료수집
Area/Reasource:
Project:
---
# Adapter 패턴

**"인터페이스(Interface, 외부와 주고받는 약속된 신호 규격)가 서로 다른 두 코드를 그대로 이어붙이는 구조 패턴."**

Adapter 패턴(어댑터 패턴, 인터페이스 변환기)은 GoF 구조(Structural) 패턴 중 하나다. 이미 존재하지만 내가 원하는 모양이 아닌 클래스를, 수정하지 않고 내 시스템이 기대하는 인터페이스로 감싸서 쓰게 해준다. Wrapper(래퍼, 포장) 패턴이라는 별칭으로도 자주 불린다.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 구조(Structural) 패턴 |
| 목적 | 호환되지 않는 인터페이스를 원하는 인터페이스로 변환 |
| 핵심 질문 | "이미 잘 동작하는 라이브러리인데, 시그니처가 안 맞아서 못 쓰고 있나?" |
| 변형 | 클래스 어댑터(상속 기반), 객체 어댑터(합성 기반) |
| 트레이드오프 | 래퍼 클래스 한 겹 더 생김. 대신 원본을 건드리지 않음 |

> **한마디 요약**: "콘센트가 다르면 콘센트를 갈지 말고, 어댑터를 끼워라."

---

## 일상 비유: 해외여행 전원 어댑터

한국 110/220V 기기를 들고 영국(BS1363 플러그) 호텔에 갔다고 하자. 드라이어의 콘센트 모양을 깎아서 바꿀 수는 없다. 대신 "한쪽은 한국 플러그, 반대쪽은 영국 플러그"인 어댑터 젠더(Adapter Gender)를 끼운다. 드라이어 내부는 하나도 안 바꾸고, 호텔 콘센트도 안 바꿨다. 가운데 껴 있는 어댑터가 모양 변환을 담당한다.

코드 세계도 똑같다. "라이브러리의 인터페이스"를 못 바꾸고 "내 애플리케이션의 인터페이스"도 이미 결정됐을 때, 가운데에 Adapter 클래스를 둬서 변환한다.

---

## 구조

| 역할 | 설명 |
| --- | --- |
| Target | 클라이언트가 기대하는 인터페이스 (우리 쪽 규격) |
| Adaptee | 가져다 쓰고 싶은 기존 클래스 (상대 쪽 규격) |
| Adapter | Target을 구현하면서 내부적으로 Adaptee를 호출해 변환 |
| Client | Target 인터페이스만 바라봄. Adaptee의 존재는 모름 |

단계 분해:
1. **1단계**: Client 코드는 `Target` 인터페이스에만 의존
2. **2단계**: 실제로 쓰고 싶은 외부 클래스(Adaptee)의 메서드 이름, 인자 구조는 다름
3. **3단계**: Adapter가 `Target`을 구현하고, 메서드 내부에서 Adaptee의 호출로 변환해 돌려줌

### 클래스 어댑터 vs 객체 어댑터

| 구분 | 클래스 어댑터 | 객체 어댑터 |
| --- | --- | --- |
| 결합 방식 | Adaptee를 상속 | Adaptee를 필드로 가짐 (합성) |
| 유연성 | 런타임 교체 불가 | 런타임 교체 가능 |
| 다중 상속 | 언어 지원 필요(Python 가능, Java 불가) | 어디서나 가능 |
| 권장도 | 낮음 | 높음 (대부분 경우 합성 선호) |

---

## AI 에이전트 개발 예시 1: LLM Provider 통일

OpenAI, Anthropic, Cohere, 로컬 Ollama마다 SDK 호출 시그니처가 다 다르다. 애플리케이션 안에서 "LLM을 바꿔도 호출부는 똑같이 쓰고 싶다"면 Adapter가 정답이다.

### 상황: 시그니처가 제각각

```python
# OpenAI
openai.ChatCompletion.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "hi"}],
)

# Anthropic
anthropic.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    messages=[{"role": "user", "content": "hi"}],
)

# Cohere
cohere.chat(
    model="command-r",
    message="hi",
    chat_history=[],
)
```

호출 함수명도, 인자 이름도, 응답 객체 구조도 제각각. Client 코드가 이걸 다 알면 LLM 교체가 지옥이 된다.

### Target 정의

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class ChatMessage:
    role: str
    content: str

@dataclass
class ChatResponse:
    text: str
    input_tokens: int
    output_tokens: int

class ChatModel(ABC):
    @abstractmethod
    def chat(self, messages: list[ChatMessage]) -> ChatResponse: ...
```

### Adapter 구현

```python
class OpenAIAdapter(ChatModel):
    def __init__(self, client, model: str):
        self._client = client
        self._model = model

    def chat(self, messages):
        raw_messages = [{"role": m.role, "content": m.content} for m in messages]
        resp = self._client.chat.completions.create(
            model=self._model,
            messages=raw_messages,
        )
        return ChatResponse(
            text=resp.choices[0].message.content,
            input_tokens=resp.usage.prompt_tokens,
            output_tokens=resp.usage.completion_tokens,
        )


class AnthropicAdapter(ChatModel):
    def __init__(self, client, model: str, max_tokens: int = 1024):
        self._client = client
        self._model = model
        self._max_tokens = max_tokens

    def chat(self, messages):
        raw_messages = [{"role": m.role, "content": m.content} for m in messages]
        resp = self._client.messages.create(
            model=self._model,
            max_tokens=self._max_tokens,
            messages=raw_messages,
        )
        return ChatResponse(
            text=resp.content[0].text,
            input_tokens=resp.usage.input_tokens,
            output_tokens=resp.usage.output_tokens,
        )
```

### 클라이언트 코드는 깨끗

```python
def run_agent(llm: ChatModel, question: str) -> str:
    response = llm.chat([ChatMessage(role="user", content=question)])
    print(f"cost tokens: in={response.input_tokens}, out={response.output_tokens}")
    return response.text

# 전환이 자유로움
llm = OpenAIAdapter(openai_client, "gpt-4o")
# 또는
llm = AnthropicAdapter(anthropic_client, "claude-opus-4-7")

print(run_agent(llm, "Hello"))
```

`run_agent` 코드는 LLM 업체와 무관하다. 이것이 Adapter의 진가다.

---

## AI 에이전트 개발 예시 2: Vector Store 통일

Pinecone, Weaviate, Chroma, FAISS도 같은 문제. 각자 `upsert`, `insert`, `add`, `index_put` 등 이름도 다르다.

```python
class VectorStore(ABC):
    @abstractmethod
    def upsert(self, vectors: list[tuple[str, list[float], dict]]) -> None: ...
    @abstractmethod
    def search(self, query: list[float], k: int = 5) -> list[str]: ...


class PineconeAdapter(VectorStore):
    def __init__(self, index):
        self._index = index

    def upsert(self, vectors):
        self._index.upsert(vectors=[
            {"id": id_, "values": vec, "metadata": meta}
            for id_, vec, meta in vectors
        ])

    def search(self, query, k=5):
        res = self._index.query(vector=query, top_k=k)
        return [m["id"] for m in res["matches"]]


class ChromaAdapter(VectorStore):
    def __init__(self, collection):
        self._collection = collection

    def upsert(self, vectors):
        ids = [id_ for id_, _, _ in vectors]
        embs = [vec for _, vec, _ in vectors]
        metas = [meta for _, _, meta in vectors]
        self._collection.upsert(ids=ids, embeddings=embs, metadatas=metas)

    def search(self, query, k=5):
        res = self._collection.query(query_embeddings=[query], n_results=k)
        return res["ids"][0]
```

---

## 실제 라이브러리에서의 Adapter

| 라이브러리 | 적용 |
| --- | --- |
| LangChain | `ChatOpenAI`, `ChatAnthropic` 모두 `BaseChatModel`이라는 Target을 구현. 내부는 각 SDK Adaptee를 감쌈 |
| LlamaIndex | `LLM`, `Embedding`, `VectorStore` 인터페이스에 각 업체별 Adapter 구현체 배치 |
| Python `os.path` | 운영체제별(POSIX/Windows) 경로 처리 차이를 추상화 |
| Java `Arrays.asList()` | 배열(Adaptee)을 `List` 인터페이스(Target)로 변환 |

---

## 직접 확인해 보기

프로젝트에서 외부 SDK 호출이 여러 파일에 흩어져 있는지 확인한다.

```bash
grep -rn "openai\." src/ | wc -l
grep -rn "anthropic\." src/ | wc -l
```

두 SDK 호출이 비즈니스 로직 여기저기에 섞여 있다면 Adapter 도입 신호다. 이상적으로는 Adapter 파일 안에서만 외부 SDK 이름이 등장해야 한다.

---

## 왜 이렇게 하는가

Adapter의 설계 의도와 대안 대비 장점:

- **기존 코드 무수정(Open/Closed)**: Adaptee도 Client도 건드리지 않음. SOLID의 OCP(개방 폐쇄 원칙)에 정확히 부합
- **테스트 용이**: Target 인터페이스에 대해 `FakeAdapter`를 만들면 네트워크 없이 테스트 가능
- **의존성 역전(DIP) 달성 수단**: Client가 추상(Target)에만 의존하게 하는 실천 방법
- **관심사 격리**: "호출 변환"이라는 저수준 작업이 Adapter 안에 갇혀 있음

트레이드오프:
- 클래스 개수 증가. LLM 업체 N개면 Adapter N개
- 모든 업체의 최소 공통 분모(LCD, Least Common Denominator)만 노출하게 되면 고급 기능을 잃을 수 있음. 이 경우 `extra_params: dict`를 Target에 받아 Adapter마다 해석하도록 하는 절충안을 쓴다

---

## 포함 관계

- **Adapter ⊂ DIP(의존성 역전)의 실천 수단**: Client가 Target이라는 추상에 의존하고, 구체는 Adapter들이 구현
- **Adapter vs Facade**: Adapter는 "모양 변환"이 목적, Facade는 "복잡도 숨김"이 목적. 겉보기가 비슷하지만 의도가 다름. [[Facade 패턴]] 참고
- **Adapter vs Decorator**: Adapter는 인터페이스를 바꾸고, Decorator는 인터페이스를 유지한 채 책임을 추가

---

## 한마디 요약

> **"원본을 못 고치거나 고치기 싫으면, 사이에 어댑터를 끼워 인터페이스만 맞춰라."**

관련 문서:
- [[Builder 패턴]]
- [[Facade 패턴]]
- [[SOLID원칙]]
- [[고수준 모듈과 저수준 모듈(SOLID DIP)]]
