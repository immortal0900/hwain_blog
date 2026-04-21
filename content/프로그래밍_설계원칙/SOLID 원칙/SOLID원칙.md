---
생성날짜:
- 2026-01-16 00:32
마지막수정날짜:
- 2026-01-16-금요일 00:31
tags:
- 설계원칙
- SOLID
- OOP
- AI에이전트
별칭:
- SOLID
type:
- 자료수집
Area/Reasource:
Project:
---
# SOLID 원칙이란?

SOLID는 객체지향 설계를 할 때 **"코드를 읽기 쉽고, 수정하기 편하게"** 만들기 위한 5가지 약속이다. 다섯 글자는 Robert C. Martin(일명 Uncle Bob)이 1990년대 말 정리한 다섯 원칙의 머리글자를 모은 것이다.

## 한눈에 보는 5원칙

| 약자  | 풀네임                                                      | 한국어             | 한 줄 정의                              | 핵심 질문                 |
| --- | -------------------------------------------------------- | --------------- | ----------------------------------- | --------------------- |
| S   | Single Responsibility Principle                          | 단일 책임 원칙        | 클래스는 한 가지 변경 이유만 가져야 한다             | "이 클래스, 누구 때문에 바뀌나?"  |
| O   | Open-Closed Principle                                    | 개방 폐쇄 원칙        | 확장에는 열려 있고, 수정에는 닫혀 있어야 한다          | "새 기능 추가할 때 기존 코드 건드리나?" |
| L   | Liskov Substitution Principle                            | 리스코프 치환 원칙      | 자식 클래스는 부모 자리를 대신해도 동작이 깨지면 안 된다    | "자식을 부모로 바꿔 껴도 멀쩡한가?" |
| I   | Interface Segregation Principle                          | 인터페이스 분리 원칙     | 쓰지도 않는 메서드를 억지로 구현하게 만들지 마라         | "이 인터페이스, 너무 뚱뚱하지 않나?" |
| D   | Dependency Inversion Principle                           | 의존성 역전 원칙       | 고수준도 저수준도 둘 다 추상화에 의존해야 한다          | "구체 클래스를 new 하고 있나?"  |

> **한마디 요약**: S는 "나눠라", O는 "갈아끼워라", L은 "약속 지켜라", I는 "잘게 쪼개라", D는 "거꾸로 의존해라".

---

## 1. S: Single Responsibility Principle (단일 책임 원칙)

**"하나의 클래스는 하나의 일만 해야 한다."**

한 클래스가 바뀌어야 할 이유는 오직 하나여야 한다. "일"을 기능으로 보지 말고 "변경 이유(actor, 요구를 내는 사람)"로 보는 게 정확하다.

### 일상 비유
식당 주방에서 한 사람이 요리 + 홀 서빙 + 회계 + 식자재 주문까지 전부 하면, 손님 항의가 들어와도 (회계 사장/매니저/셰프) 중 누가 움직여야 하는지 헷갈린다. 각자 전담을 두면 수정할 사람이 명확해진다.

### AI 에이전트 개발 예시

```python
# 나쁜 예: 하나의 클래스가 너무 많은 일
class ChatAgent:
    def call_llm(self, messages): ...       # LLM 호출
    def save_to_db(self, message): ...      # DB 저장
    def format_response(self, text): ...    # 포맷팅
    def send_slack(self, text): ...         # 외부 알림
    def log_cost(self, tokens): ...         # 비용 로깅
```

바뀌는 이유가 5개이다. LLM 공급자 교체, DB 스키마 변경, 포맷 규칙 변경, 알림 채널 추가, 비용 정책 변경. 하나 건드리면 다른 기능 테스트까지 돌려야 한다.

```python
# 좋은 예: 책임 분리
class LLMClient: ...          # LLM 호출만
class MessageRepository: ...  # 영속화만
class ResponseFormatter: ...  # 포맷팅만
class NotificationService: ...# 알림만
class CostTracker: ...        # 비용 기록만

class ChatAgent:
    def __init__(self, llm, repo, formatter, notifier, tracker):
        self.llm = llm
        self.repo = repo
        ...
```

### 직접 확인하기
자기가 짠 클래스를 열고 `git blame`으로 최근 수정 커밋을 뽑아봐라. 수정 이유가 3개 이상 섞여 있으면 SRP 위반 신호이다.

---

## 2. O: Open-Closed Principle (개방 폐쇄 원칙)

**"확장에는 열려 있고, 수정에는 닫혀 있어야 한다."**

새 기능을 추가할 때 기존 코드(특히 이미 운영 중인 검증된 코드)는 건드리지 않고, 새로운 클래스/모듈을 "갖다 붙이는" 것으로 해결되어야 한다.

### OCP 위반 신호 3종

1. 타입 분기 체인이 계속 자람 (`if type == "A" elif type == "B" ...`)
2. 새 기능 추가 시 기존 클래스를 수정해야 함
3. 새 기능을 넣으려고 기존 테스트가 줄줄이 깨짐

### AI 에이전트 개발 예시: Tool 추가

에이전트에 새 Tool(웹 검색, 코드 실행, 계산기 등)을 붙이는 상황을 보자.

```python
# 나쁜 예: if-elif 체인
class Agent:
    def use_tool(self, name, args):
        if name == "search":
            return search_web(args["query"])
        elif name == "calc":
            return eval(args["expr"])
        elif name == "code":
            return run_python(args["code"])
        # 새 Tool 추가 = Agent 클래스 수정
```

```python
# 좋은 예: Tool 추상화 + 레지스트리
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

    def use_tool(self, name, args):
        return self.registry[name].run(args)

# 새 Tool 추가 = 새 클래스만 만들고 주입
agent = Agent([SearchTool(), CalcTool(), CodeTool()])
```

> Agent 코드는 고정, Tool만 새로 만들어 주입. 이것이 "확장에는 열리고 수정에는 닫힌" 상태이다.

---

## 3. L: Liskov Substitution Principle (리스코프 치환 원칙)

**"자식 클래스는 언제나 부모 클래스를 대신할 수 있어야 한다."**

Barbara Liskov(1987)의 논문에서 따온 원칙이다. 상속을 받았으면 부모가 약속한 동작(사전 조건, 사후 조건, 불변식)을 깨뜨려서는 안 된다.

### 위반 신호
- 자식에서 예외를 새로 던진다 (`NotImplementedError`)
- 부모가 받는 인자 타입을 자식이 좁게 제한한다
- 부모 리턴 타입을 자식이 더 넓힌다 (호출자 입장에서 당황스러움)

### AI 에이전트 개발 예시

```python
# 부모: "embed는 float 벡터 리스트를 돌려준다" 약속
class Embedder(ABC):
    @abstractmethod
    def embed(self, text: str) -> list[float]: ...

# 나쁜 자식: 어떤 입력에서 None을 돌려줌
class FlakyEmbedder(Embedder):
    def embed(self, text):
        if len(text) > 1000:
            return None  # ❌ 약속 파기, 호출자가 list[float] 기대
        return openai_embed(text)
```

호출자 입장에서 `results = [e.embed(t) for t in texts]` 했을 때 `None`이 섞이면 후속 코드가 터진다. 이런 자식은 LSP 위반이다.

```python
# 좋은 자식: 약속 유지, 문제 상황은 예외로
class ChunkedEmbedder(Embedder):
    def embed(self, text):
        if len(text) > 1000:
            chunks = self._split(text)
            vecs = [openai_embed(c) for c in chunks]
            return self._avg(vecs)  # 여전히 list[float] 반환
        return openai_embed(text)
```

### 한마디 요약
"is-a 관계"만 있으면 상속이 되는 게 아니다. "행동 관계"까지 맞아야 LSP를 지킨 것이다.

---

## 4. I: Interface Segregation Principle (인터페이스 분리 원칙)

**"클라이언트가 자기가 쓰지도 않는 메서드에 의존하도록 강요하지 마라."**

하나의 거대한 인터페이스보다 작고 구체적인 여러 인터페이스가 낫다.

### 위반 신호
- 어떤 구현체가 특정 메서드에서 `raise NotImplementedError`를 던진다
- 인터페이스가 "AllInOne"처럼 비대하다
- 일부 메서드만 쓰는데 전체 인터페이스를 import 해야 한다

### AI 에이전트 개발 예시: 거대한 LLMProvider

```python
# 나쁜 예: 모든 LLM 기능을 한 인터페이스에
class LLMProvider(ABC):
    @abstractmethod
    def chat(self, messages): ...
    @abstractmethod
    def embed(self, text): ...
    @abstractmethod
    def moderate(self, text): ...
    @abstractmethod
    def image_generate(self, prompt): ...
    @abstractmethod
    def speech_to_text(self, audio): ...

# 임베딩만 제공하는 업체는?
class CohereEmbedProvider(LLMProvider):
    def embed(self, text): return cohere_embed(text)
    def chat(self, *a): raise NotImplementedError  # ❌ 강요당함
    def image_generate(self, *a): raise NotImplementedError
    ...
```

```python
# 좋은 예: 역할별 분리
class ChatModel(ABC):
    @abstractmethod
    def chat(self, messages): ...

class EmbeddingModel(ABC):
    @abstractmethod
    def embed(self, text) -> list[float]: ...

class Moderator(ABC):
    @abstractmethod
    def moderate(self, text) -> bool: ...

# 필요한 것만 구현
class CohereEmbedProvider(EmbeddingModel):
    def embed(self, text): return cohere_embed(text)
```

### 포함 관계
LSP(행동 약속)를 지키려면 인터페이스가 작을수록 유리하다. 즉, ISP는 LSP를 쉽게 해주는 전제 조건이 된다.

---

## 5. D: Dependency Inversion Principle (의존성 역전 원칙)

**"고수준 모듈이 저수준 모듈에 의존하지 말고, 둘 다 추상화에 의존해야 한다."**

관련 심화 자료는 [[고수준 모듈과 저수준 모듈(SOLID DIP)]] 참고. [[SOLID_ O와 D의 차이]]에서 OCP와 구분법을 정리함.

### 그림으로 이해하기

- **나쁜 예 (상위 모듈이 하위 모듈에 직접 의존)**: 로봇 손이 '망치'라는 특정 도구에 딱딱하게 고정된 상태. 망치 대신 드라이버를 쓰려면 로봇 손 전체를 수술해서 바꿔야 한다.
- **좋은 예 (의존성 역전)**: 로봇 손은 '도구'를 끼울 수 있는 **소켓(인터페이스)** 만 가지고 있다. 그 소켓에 망치를 끼우든 드라이버를 끼우든 로봇 손 코드는 바뀔 필요가 없다.

### 왜 '역전'이라고 부르는가?

보통은 '상위(나)'가 '하위(도구)'를 선택하고 의존하는 것이 자연스러워 보인다. DIP를 적용하면, **상위 모듈이 정의한 인터페이스(약속)에 하위 모듈이 맞춰야 하는 상황**이 된다.

즉, 주도권이 '도구'에서 '사용자'로 넘어가서 의존 화살표 방향이 뒤집혔다는 의미에서 **'역전'** 이라고 부른다.

### AI 에이전트 개발 예시: Vector Store 교체

```python
# 나쁜 예
class RAGAgent:
    def __init__(self):
        self.store = PineconeStore(api_key=...)  # 구체 의존
```

```python
# 좋은 예
class VectorStore(ABC):
    @abstractmethod
    def upsert(self, vecs): ...
    @abstractmethod
    def search(self, query_vec, k): ...

class RAGAgent:
    def __init__(self, store: VectorStore):  # 추상 의존
        self.store = store

# 프로덕션
agent = RAGAgent(PineconeStore(...))
# 테스트
agent = RAGAgent(InMemoryStore())
# 마이그레이션
agent = RAGAgent(WeaviateStore(...))
```

RAGAgent 코드는 한 줄도 바뀌지 않는다.

---

## SOLID 원칙 간의 포함 관계

다섯 원칙은 독립된 규칙이 아니라 서로를 받쳐준다.

- **SRP → OCP**: 책임이 분리돼 있어야 새 책임을 독립 클래스로 얹을 수 있다
- **ISP → LSP**: 인터페이스가 작을수록 약속을 지키기 쉽다
- **DIP → OCP**: 추상화에 의존해야 구현을 갈아끼우는 "확장"이 성립한다
- **LSP → OCP**: 자식이 부모 자리를 대신할 수 있어야 다형성 기반 확장이 안전하다

```
     SRP (나누기)
        │
        ▼
     ISP (인터페이스 쪼개기)
        │
        ▼
     LSP (행동 약속 지키기)
        │
        ▼
     DIP (추상화에 의존)
        │
        ▼
     OCP (확장에 열리고 수정에 닫힘)
```

> **한마디 요약**: OCP는 목표, DIP는 달성 수단, SRP와 ISP는 재료 준비, LSP는 안전장치.

---

## AI Agent Developer 체크리스트

에이전트 코드를 짜면서 아래 질문을 스스로 던져보자.

- [ ] LLM Provider를 바꾸려면 몇 개 파일을 수정해야 하나? (DIP)
- [ ] 새 Tool을 붙일 때 Agent 클래스를 수정하는가? (OCP)
- [ ] Memory/Vector Store 구현체 바꿀 때 테스트는 그대로 돌아가는가? (DIP + LSP)
- [ ] "채팅 + 임베딩 + 모더레이션"을 한 인터페이스에 몰아넣었는가? (ISP)
- [ ] Agent 클래스가 "LLM 호출 + DB 저장 + 비용 로깅 + Slack 알림"을 모두 하는가? (SRP)

---

## 요약

> **"구체적인 물건(망치, 아반떼, GPT-4)에 집착하지 말고, 추상적인 역할(연장, 자동차, LLM)에 집중하라. 그래야 나중에 부품만 슥 갈아끼우기 편하다."**

관련 문서:
- [[SOLID_ O와 D의 차이]]
- [[고수준 모듈과 저수준 모듈(SOLID DIP)]]
