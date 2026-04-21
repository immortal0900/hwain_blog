---
생성날짜:
- 2026-04-21 12:45
마지막수정날짜:
- 2026-04-21-화요일 12:45
tags:
- 설계원칙
- SOLID
- OCP
- OOP
- AI에이전트
별칭:
- OCP
- Open-Closed Principle
- 개방 폐쇄 원칙
type:
- 자료수집
Area/Reasource:
Project:
---
# OCP (Open-Closed Principle, 개방 폐쇄 원칙)

> **"Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification."**
> 소프트웨어 요소(클래스, 모듈, 함수 등)는 **확장에는 열려 있고, 수정에는 닫혀 있어야** 한다.
> Bertrand Meyer, *Object-Oriented Software Construction* (1988)

상위 개념: [[SOLID원칙]]
관련: [[SRP (단일 책임 원칙)]], [[SOLID_ O와 D의 차이]], [[고수준 모듈과 저수준 모듈(SOLID DIP)]]

---

## 1. 두 가지 버전의 OCP

OCP는 사실 두 번 정의되었다. 이걸 구분 못 하면 책마다 설명이 달라서 혼란스럽다.

| 버전 | 제안자 | 년도 | 핵심 수단 |
| --- | --- | --- | --- |
| **Meyer 버전** | Bertrand Meyer | 1988 | **구현 상속(implementation inheritance)**. 기존 클래스를 상속해서 새 동작을 덧붙임. 기존 클래스는 손대지 않음 |
| **Martin 버전** | Robert C. Martin | 2000년대 | **추상 인터페이스 + 다형성(polymorphism)**. 추상 기반 클래스에 의존하고, 구현체를 갈아끼움 |

현대 객체지향 커뮤니티는 거의 전부 **Martin 버전**을 쓴다. 상속은 결합도를 너무 높이고 LSP 위반을 부르기 쉽기 때문에, "상속 대신 합성(composition over inheritance)" 흐름으로 옮겨갔다.

> **한마디 요약**: OCP는 "상속"이 아니라 "추상에 기대서 갈아끼우기"로 이해하자.

---

## 2. "확장에 열려 있고, 수정에 닫혀 있다"는 말의 뜻

두 단어의 주어(subject)가 다르다는 점이 핵심이다.

| 구절 | 주어 | 의미 |
| --- | --- | --- |
| "확장에 열려 있다" | **시스템의 동작(behavior)** | 새 기능을 추가할 수 있어야 함 |
| "수정에 닫혀 있다" | **기존 소스 코드(source)** | 이미 작성·검증된 코드는 건드리지 않아야 함 |

**동작은 확장되지만, 코드는 수정되지 않는다.** 모순처럼 들리지만, 다형성(polymorphism, 같은 인터페이스를 여러 구현체가 구현하는 성질)을 이용하면 성립한다.

---

## 3. 일상 비유: 스마트폰 앱스토어

스마트폰 OS(안드로이드/iOS)는 OCP의 모범 사례다.

| 영역 | 동작 방식 |
| --- | --- |
| **수정에 닫힘** | OS 코어는 신규 앱이 출시될 때마다 커널이 바뀌지 않음. 매일 수백 개 앱이 나와도 OS는 그대로 |
| **확장에 열림** | 새 앱을 설치하면 기능이 늘어남. 삭제하면 줄어듦 |
| **열쇠** | OS가 정의한 **API(Application Programming Interface)** 를 앱이 구현. OS는 API만 호출하고, 구현체는 앱 쪽에 위임 |

OCP가 지켜진 시스템의 공통점: **코어는 안정, 주변은 교체 가능한 플러그인**.

---

## 4. OCP 위반 신호

자기 코드에 아래 증상이 보이면 OCP 위반이다.

### 4.1 타입 분기 체인이 자람

```python
# ❌ 새 타입 추가할 때마다 함수 수정
def route(msg_type: str, payload: dict):
    if msg_type == "chat":
        return handle_chat(payload)
    elif msg_type == "embed":
        return handle_embed(payload)
    elif msg_type == "moderate":
        return handle_moderate(payload)
    elif msg_type == "rerank":   # 새 타입 = 기존 함수 수정
        return handle_rerank(payload)
```

### 4.2 설정 스위치가 비즈니스 로직 안에 박힘

```python
# ❌ Agent 내부에 LLM 스위치
class Agent:
    def ask(self, q, provider="openai"):
        if provider == "openai":
            ...
        elif provider == "anthropic":
            ...
```

### 4.3 새 기능 하나 넣으려고 기존 테스트가 줄줄이 깨짐

회귀(regression) 테스트가 실패하는 게 당연해 보이면 OCP가 깨진 것이다. 기존 코드를 수정했기 때문에 기존 테스트 전제가 무너진 것.

### 4.4 merge conflict가 같은 파일에서 반복적으로 난다

여러 팀이 같은 if-elif 체인 아래에 각자 분기를 추가하느라 같은 줄을 건드리면 conflict가 끊이지 않는다.

---

## 5. AI Agent 개발에서의 OCP

에이전트 프로젝트에서 OCP를 가장 강하게 요구받는 지점 세 가지.

### 5.1 Tool 추가

새 Tool(웹 검색, 코드 실행, 파일 읽기, DB 쿼리 등)을 붙일 때 Agent 본체를 수정하지 않는다.

```python
# ❌ 위반
class Agent:
    def use_tool(self, name, args):
        if name == "search":
            return search_web(args["query"])
        elif name == "calc":
            return eval(args["expr"])
        elif name == "read_file":
            return open(args["path"]).read()
```

```python
# ✅ OCP 준수: Tool 추상화 + 레지스트리
from abc import ABC, abstractmethod

class Tool(ABC):
    name: str

    @abstractmethod
    def run(self, args: dict) -> str: ...

class SearchTool(Tool):
    name = "search"
    def run(self, args): return search_web(args["query"])

class CalcTool(Tool):
    name = "calc"
    def run(self, args): return eval(args["expr"])

class Agent:
    def __init__(self, tools: list[Tool]):
        self.registry = {t.name: t for t in tools}

    def use_tool(self, name: str, args: dict) -> str:
        return self.registry[name].run(args)

# 새 Tool 추가: 새 클래스만 만들고 주입
agent = Agent([SearchTool(), CalcTool(), CodeExecTool()])
```

**Agent 클래스 코드는 한 글자도 바뀌지 않는다.** Tool 수가 2개든 200개든 동일.

### 5.2 LLM Provider 교체

OpenAI, Anthropic, Gemini, 로컬 Llama, vLLM 같은 공급자 간 A/B 테스트는 에이전트 개발의 일상이다.

```python
# ❌ 위반: provider 문자열로 분기
def generate(provider: str, prompt: str) -> str:
    if provider == "openai":
        return openai_call(prompt)
    elif provider == "anthropic":
        return anthropic_call(prompt)
    elif provider == "gemini":
        return gemini_call(prompt)
```

```python
# ✅ OCP 준수: ChatModel 인터페이스
class ChatModel(ABC):
    @abstractmethod
    def generate(self, prompt: str) -> str: ...

class OpenAIChat(ChatModel): ...
class AnthropicChat(ChatModel): ...
class GeminiChat(ChatModel): ...
class FakeChat(ChatModel): ...   # 테스트용

def run_agent(model: ChatModel, prompt: str) -> str:
    return model.generate(prompt)   # 새 Provider 추가해도 이 함수는 불변
```

상세 예시는 [[SOLID_ O와 D의 차이]] 참고.

### 5.3 Vector Store / Memory 교체

RAG(Retrieval-Augmented Generation, 검색 기반 생성) 에이전트는 Pinecone, Weaviate, Qdrant, pgvector, 인메모리 구현체를 환경별로 바꿔 쓴다.

```python
# ✅ VectorStore 추상화
class VectorStore(ABC):
    @abstractmethod
    def upsert(self, vecs: list[Vector]) -> None: ...
    @abstractmethod
    def search(self, query_vec: Vector, k: int) -> list[Hit]: ...

class PineconeStore(VectorStore): ...
class QdrantStore(VectorStore): ...
class InMemoryStore(VectorStore): ...   # 테스트용

class RAGAgent:
    def __init__(self, store: VectorStore): self.store = store
```

환경별 주입 변경만으로 개발/스테이징/프로덕션을 분리할 수 있다.

---

## 6. OCP를 달성하는 3가지 수단

### 6.1 Strategy 패턴 (전략)

동작(알고리즘)을 인터페이스로 추상화하고, 구현체를 갈아끼운다. 위에서 본 `ChatModel`, `Tool`, `VectorStore`가 전부 Strategy이다.

### 6.2 Decorator 패턴 (장식)

기존 동작을 감싸서 부가 기능을 덧붙인다. **원본 클래스를 수정하지 않고 기능을 확장**하는 OCP의 핵심 실천 수단.

```python
# 원본 Tool은 그대로, 캐싱/로깅을 데코레이터로 덧붙임
class CachedTool(Tool):
    def __init__(self, inner: Tool, cache: dict):
        self.inner = inner
        self.name = inner.name
        self.cache = cache

    def run(self, args: dict) -> str:
        key = json.dumps(args, sort_keys=True)
        if key not in self.cache:
            self.cache[key] = self.inner.run(args)
        return self.cache[key]

class LoggedTool(Tool):
    def __init__(self, inner: Tool):
        self.inner = inner
        self.name = inner.name

    def run(self, args):
        print(f"[{self.name}] args={args}")
        result = self.inner.run(args)
        print(f"[{self.name}] result={result[:50]}...")
        return result

# 조립
tool = LoggedTool(CachedTool(SearchTool(), cache={}))
```

### 6.3 Plugin / Registry 패턴

런타임에 구현체를 등록해서 조회한다. Python의 `entry_points`, `importlib.metadata`를 이용하면 외부 패키지가 자동으로 Tool을 등록하게 만들 수도 있다.

```python
# pyproject.toml (외부 플러그인 패키지)
# [project.entry-points."myagent.tools"]
# wiki = "wiki_plugin:WikiTool"

import importlib.metadata

def load_plugins() -> list[Tool]:
    return [ep.load()() for ep in importlib.metadata.entry_points(group="myagent.tools")]

agent = Agent(tools=load_plugins())
```

플러그인을 `pip install`만 하면 에이전트 본체 코드 수정 없이 기능이 확장된다.

---

## 7. OCP를 적용하는 판단 기준

OCP를 모든 곳에 적용하면 오히려 **추상화 지옥(abstraction hell)** 에 빠진다. 적용 여부를 판단하는 기준.

| 상황 | OCP 적용? |
| --- | --- |
| 이미 2번 이상 변경 요구가 들어옴 | 적용 |
| 앞으로 교체될 가능성이 확실 (LLM Provider, DB 등) | 적용 |
| 외부 팀/사용자가 확장할 여지가 있음 | 적용 |
| 한 번 쓰고 버릴 스크립트 | 적용 X |
| "언젠가 바뀔 수도 있으니까" | 적용 X (YAGNI 원칙 위반) |

> **경험칙: "두 번째로 같은 분기가 필요해질 때" 추상화한다.** 처음엔 구체 코드로 쓰고, 같은 구조가 반복되면 그때 추상화하라. Meyer의 OCP는 원래 "미리 설계하라"였지만, 실전에선 "Rule of Three(세 번 반복되면 추상화)"가 더 안전하다.

---

## 8. OCP와 다른 SOLID 원칙

### 8.1 OCP ← DIP (의존성 역전)

OCP를 달성하려면 거의 항상 DIP가 먼저 깔려야 한다. 추상 인터페이스에 의존해야 구현체를 갈아끼울 수 있기 때문.

```
DIP (추상에 의존)
   │
   ▼
OCP (수정 없이 확장)
```

DIP와 OCP를 혼동하지 않는 법은 [[SOLID_ O와 D의 차이]] 참고.

### 8.2 OCP ← LSP (리스코프 치환)

새 구현체가 기존 구현체 자리를 문제없이 차지할 수 있어야 진짜 "확장"이다. LSP 위반이 있으면 OCP는 겉모습만 준수된 상태.

### 8.3 OCP ← SRP (단일 책임)

책임이 뭉쳐 있으면 "새 책임 하나 확장" 자체가 불가능하다. SRP로 책임이 나뉘어 있어야 OCP 단위가 생긴다.

```
SRP (책임 분리)
   │
   ▼
OCP (확장 가능)
```

### 8.4 OCP의 구현 경로 요약

```
    SRP (나눠두기) ─┐
                    ├─→ OCP (확장 가능)
    DIP (추상 의존) ─┤
                    │
    LSP (약속 지키기) ┘
```

---

## 9. 실전 체크리스트

- [ ] 새 LLM Provider를 붙일 때, Agent 본체 파일을 수정해야 하는가? (수정해야 하면 OCP 위반)
- [ ] Tool을 하나 추가할 때 영향받는 기존 테스트가 있는가? (있으면 OCP 위반)
- [ ] 소스 코드에 `if provider ==`, `elif type ==` 같은 분기가 3개 이상 연속으로 있는가? (위반 신호)
- [ ] 플러그인처럼 외부에서 기능을 추가할 수 있는 구조인가? (아니면 확장성 0점)
- [ ] 기존 코드를 건드리지 않고 기능을 추가한 경험이 최근에 있었는가? (없다면 설계 재검토)

### 코드베이스 진단 명령

```bash
# OCP 위반 냄새: if-elif 체인
grep -rn "elif.*provider\|elif.*type\|elif.*name" src/

# OCP 위반 냄새: 하드코딩된 구현체 이름
grep -rn "OpenAI(\|Anthropic(\|Pinecone(" src/ | grep -v "factory\|main\|__init__"
```

두 검색 결과가 비즈니스 로직 파일(Agent 본체, Use Case 계층)에 있으면 리팩토링 후보.

---

## 10. 흔한 오해와 함정

### 오해 1: "모든 클래스를 열어둬야 한다"

아니다. **핵심 비즈니스 로직**과 **변경 빈도가 높은 지점**만 열어둔다. 상수, DTO, 단순 유틸은 OCP 대상이 아니다.

### 오해 2: "상속을 쓰면 OCP다"

Meyer 버전의 낡은 해석. 현대 Martin 버전에서는 **추상 인터페이스 + 합성(composition)** 을 선호한다. 상속은 결합도가 너무 높아 오히려 OCP를 깨뜨리기 쉽다.

### 오해 3: "확장 가능 = 설정값 늘리기"

`if config.use_new_flow:` 같은 feature flag는 OCP가 아니라 **수정**이다. 기존 코드 경로에 새 조건문을 추가하는 것이기 때문. 진짜 OCP는 **새 코드 경로**를 만들고 조립 시점에 고르는 것.

---

## 11. 한마디 요약

> **"OCP는 '상속해서 덧붙여라'가 아니라 '추상에 기대서 갈아끼워라'. 소스는 얼어 있고, 동작은 플러그인처럼 자란다."**

관련 문서:
- [[SOLID원칙]]
- [[SRP (단일 책임 원칙)]]
- [[SOLID_ O와 D의 차이]]
- [[고수준 모듈과 저수준 모듈(SOLID DIP)]]

참고 원문:
- Robert C. Martin, [The Open-Closed Principle (cleancoder.com, 2014)](https://blog.cleancoder.com/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html)
- Bertrand Meyer, *Object-Oriented Software Construction* (1988), Chapter "Open-Closed Principle"
- [Wikipedia: Open-closed principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)
