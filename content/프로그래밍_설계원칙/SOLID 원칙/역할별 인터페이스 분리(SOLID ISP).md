---
생성날짜:
- 2026-04-21 14:30
마지막수정날짜:
- 2026-04-21-화요일 14:30
tags:
- 설계원칙
- SOLID
- ISP
- 인터페이스
- AI에이전트
별칭:
- ISP
- 인터페이스 분리 원칙
type:
- 자료수집
Area/Reasource:
Project:
---
# 역할별 인터페이스 분리 (SOLID - ISP)

SOLID의 네 번째 글자 I에 해당하는 ISP(Interface Segregation Principle, 인터페이스 분리 원칙)를 정리한다. SOLID 전체는 [[SOLID원칙]], 의존성 역전과의 관계는 [[고수준 모듈과 저수준 모듈(SOLID DIP)]] 참고.

## 용어 정리 (포함 관계)

| 용어 | 무엇 | AI 에이전트 예시 |
| --- | --- | --- |
| **인터페이스(추상 타입)** | "어떤 메서드를 제공한다"라는 약속 묶음 | `ChatModel`, `Embedder`, `ToolRunner` ABC |
| **클라이언트** | 인터페이스를 통해 객체를 사용하는 쪽 | 인터페이스를 받아 쓰는 `ResearchAgent`, `RAGPipeline` |
| **구현체** | 인터페이스 약속을 실제로 채우는 클래스 | `OpenAIChat`, `PineconeStore`, `BashToolRunner` |
| **뚱뚱한 인터페이스(Fat Interface)** | 한 인터페이스에 모든 메서드를 다 모아 둔 형태 | "에이전트는 검색·생성·계획·평가를 다 한다"는 한 덩어리 ABC |

포함 관계로 보면:
```
 클라이언트 ──의존──▶ [작은 인터페이스 A]
                       ◀──구현── 구현체 1, 구현체 2

 클라이언트 ──의존──▶ [작은 인터페이스 B]
                       ◀──구현── 구현체 3
```

ISP의 요지는 **클라이언트가 자기에게 필요한 약속만 들고 있게 하라**는 것이다. SRP가 "클래스를 책임 단위로 쪼갠다"라면, ISP는 "**인터페이스를 역할 단위로 쪼갠다**"라는 짝꿍 원칙으로 본다.

---

# ISP (Interface Segregation Principle)

## 핵심 개념

> **"클라이언트는 자신이 사용하지 않는 메서드에 의존하도록 강요받지 않아야 한다."**
> Robert C. Martin이 1990년대 말 Xerox 프린터 소프트웨어 리팩토링 사례에서 도출한 원칙이다. 거대한 단일 인터페이스에 묶여 있던 Job 클래스가 모든 클라이언트에 영향을 주던 문제를 인터페이스를 작게 분리해 해결한 것이 시초이다.

일상 비유로 표현하면, **만능 리모컨 대신 용도별 작은 리모컨**이다. 거실에 TV용/에어컨용/조명용 리모컨이 따로 있으면 손님이 TV만 켜고 싶을 때 에어컨 버튼을 헷갈릴 일이 없다. 만약 리모컨 하나에 50개 버튼이 있다면, 손님(클라이언트)은 자기가 안 쓸 45개의 버튼 배치까지 외워야 한다. 그리고 에어컨 모델이 바뀌어 버튼 위치가 변하면, TV만 보던 손님 손도 같이 영향을 받는다.

---

## 왜 분리해야 하는가

뚱뚱한 인터페이스 하나가 가져오는 부작용은 다음과 같다.

1. **불필요한 결합**: 클라이언트가 안 쓰는 메서드 시그니처가 바뀌어도 같이 영향을 받음
2. **빈 구현 강요**: 일부 구현체가 `raise NotImplementedError`나 `pass`로 메서드를 비워두게 됨 (이 순간 [[SOLID원칙|LSP]]까지 같이 깨짐)
3. **테스트 부담**: Mock/Fake를 만들 때 안 쓰는 메서드까지 다 stub 처리해야 함
4. **변경 파급 범위 확대**: 인터페이스에 메서드 하나만 추가해도 모든 구현체와 모든 클라이언트가 다시 컴파일/검증 대상이 됨

---

# 나쁜 예시 (ISP 위반)

## 패턴 1: 만능 Worker 인터페이스

```python
from abc import ABC, abstractmethod

class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...
    @abstractmethod
    def sleep(self): ...

class HumanWorker(Worker):
    def work(self):  return "코드를 작성한다"
    def eat(self):   return "점심을 먹는다"
    def sleep(self): return "8시간 잔다"

class RobotWorker(Worker):
    def work(self):  return "조립 라인에서 작업한다"
    def eat(self):   raise NotImplementedError("로봇은 안 먹음")  # 빈 구현
    def sleep(self): raise NotImplementedError("로봇은 안 잠")    # 빈 구현
```

**문제점**:
1. `RobotWorker`가 `eat`, `sleep`을 강제로 받음 (사용 X인데도 시그니처에 묶임)
2. `Worker`만 받는 함수가 모든 워커에 대해 `eat()`을 호출하면 로봇에서 깨짐 ([[SOLID원칙|LSP 위반]] 동시 발생)
3. `Worker`에 `take_break()`가 추가되면 `RobotWorker`도 또 `NotImplementedError`를 채워야 함

## 패턴 2: 만능 Tool 인터페이스 (AI Agent에서 자주 보임)

```python
class Tool(ABC):
    @abstractmethod
    def read(self, path: str) -> str: ...
    @abstractmethod
    def write(self, path: str, content: str) -> None: ...
    @abstractmethod
    def execute(self, command: str) -> str: ...
    @abstractmethod
    def http_get(self, url: str) -> str: ...

class FileReaderTool(Tool):
    def read(self, path):           return open(path).read()
    def write(self, path, content): raise NotImplementedError  # 안 씀
    def execute(self, command):     raise NotImplementedError  # 안 씀
    def http_get(self, url):        raise NotImplementedError  # 안 씀
```

`Tool` 한 인터페이스가 파일 IO + Shell + HTTP를 다 떠안고 있어, 단순한 도구도 4개 메서드 다 채워야 한다. 진짜로 그 메서드를 호출하는 에이전트 코드는 `tool.write()`가 터질지 안 터질지 알 수 없다.

## 패턴 3: 만능 LLM 인터페이스

```python
class LLM(ABC):
    @abstractmethod
    def generate(self, prompt: str) -> str: ...
    @abstractmethod
    def stream(self, prompt: str): ...
    @abstractmethod
    def embed(self, text: str) -> list[float]: ...
    @abstractmethod
    def call_with_tools(self, prompt, tools) -> dict: ...
    @abstractmethod
    def fine_tune(self, dataset) -> str: ...
```

임베딩 전용 모델(예: `text-embedding-3-small`)을 감싸려고 하면 `generate`, `stream`, `call_with_tools`, `fine_tune`을 전부 `NotImplementedError`로 막아야 한다. 만약 새로 `vision()` 메서드가 추가되면 텍스트 전용 모델까지 영향을 받는다.

---

# 좋은 예시 (ISP 적용)

## 능력별로 인터페이스 쪼개기

```python
from abc import ABC, abstractmethod

class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Eatable(ABC):
    @abstractmethod
    def eat(self): ...

class Sleepable(ABC):
    @abstractmethod
    def sleep(self): ...

# 사람은 셋 다 함
class HumanWorker(Workable, Eatable, Sleepable):
    def work(self):  return "코드를 작성한다"
    def eat(self):   return "점심을 먹는다"
    def sleep(self): return "8시간 잔다"

# 로봇은 일만 함 (강제로 채우는 빈 메서드 없음)
class RobotWorker(Workable):
    def work(self):  return "조립 라인에서 작업한다"

# 클라이언트는 자기가 필요한 능력만 받음
def assign_shift(worker: Workable):
    return worker.work()

def schedule_lunch(worker: Eatable):
    return worker.eat()

print(assign_shift(HumanWorker()))   # "코드를 작성한다"
print(assign_shift(RobotWorker()))   # "조립 라인에서 작업한다"
print(schedule_lunch(HumanWorker())) # "점심을 먹는다"
# schedule_lunch(RobotWorker())      # 타입 단계에서 거부됨
```

`schedule_lunch`는 `Eatable`만 의존한다. `RobotWorker`는 애초에 `Eatable`을 구현하지 않으므로 잘못 호출하는 사고가 타입 체커 수준에서 막힌다.

---

# AI 에이전트 개발에서의 ISP

AI Agent Developer 관점에서 ISP가 가장 크게 효과를 내는 4가지 지점이다.

## 1. Tool 인터페이스를 능력별로 쪼개기

도구마다 사용 가능한 동작이 다르다. 읽기 전용 도구가 `write`, `execute`까지 들고 있으면 권한 누수와 빈 구현이 동시에 생긴다.

```python
from abc import ABC, abstractmethod

class Readable(ABC):
    @abstractmethod
    def read(self, path: str) -> str: ...

class Writable(ABC):
    @abstractmethod
    def write(self, path: str, content: str) -> None: ...

class Executable(ABC):
    @abstractmethod
    def execute(self, command: str) -> str: ...

class HttpFetchable(ABC):
    @abstractmethod
    def http_get(self, url: str) -> str: ...

# 도구는 자기가 할 수 있는 능력만 구현
class FileReadTool(Readable):
    def read(self, path):
        return open(path, encoding="utf-8").read()

class FileEditTool(Readable, Writable):
    def read(self, path):
        return open(path, encoding="utf-8").read()
    def write(self, path, content):
        with open(path, "w", encoding="utf-8") as f:
            f.write(content)

class BashTool(Executable):
    def execute(self, command):
        import subprocess
        return subprocess.check_output(command, shell=False, text=True)

class WebSearchTool(HttpFetchable):
    def http_get(self, url):
        import urllib.request
        return urllib.request.urlopen(url).read().decode()

# 클라이언트는 필요한 능력만 받음
class ReadOnlyAgent:
    def __init__(self, reader: Readable):  # write 능력은 받지도 않음
        self.reader = reader
```

이렇게 두면 "읽기만 하는 에이전트"에 실수로 쓰기 도구를 주입하는 사고를 타입 단계에서 거른다.

## 2. LLM 능력을 인터페이스별로 분리

LLM Provider마다 지원 범위가 다르다 (텍스트 전용 / 임베딩 전용 / 비전 / 함수 호출 등). 한 인터페이스에 다 묶지 말고 능력별로 쪼개 두면, RAG 파이프라인은 임베딩 능력만, 챗봇은 텍스트 생성 능력만 의존하면 된다.

```python
class TextGenerator(ABC):
    @abstractmethod
    def generate(self, messages: list[dict]) -> str: ...

class TextStreamer(ABC):
    @abstractmethod
    def stream(self, messages: list[dict]): ...

class Embedder(ABC):
    @abstractmethod
    def embed(self, text: str) -> list[float]: ...

class ToolCaller(ABC):
    @abstractmethod
    def call_with_tools(self, messages: list[dict], tools: list[dict]) -> dict: ...

# 일부 능력만 가진 모델
class OpenAIEmbeddingModel(Embedder):
    """text-embedding-3-small 전용 래퍼"""
    def __init__(self, client, model="text-embedding-3-small"):
        self.client, self.model = client, model
    def embed(self, text):
        return self.client.embeddings.create(model=self.model, input=text).data[0].embedding

# 여러 능력을 가진 모델
class AnthropicChatModel(TextGenerator, TextStreamer, ToolCaller):
    def __init__(self, client, model="claude-opus-4-7", max_tokens=1024):
        self.client, self.model, self.max_tokens = client, model, max_tokens
    def generate(self, messages):
        resp = self.client.messages.create(
            model=self.model, messages=messages, max_tokens=self.max_tokens,
        )
        return resp.content[0].text
    def stream(self, messages):
        with self.client.messages.stream(
            model=self.model, messages=messages, max_tokens=self.max_tokens,
        ) as s:
            for chunk in s.text_stream:
                yield chunk
    def call_with_tools(self, messages, tools):
        return self.client.messages.create(
            model=self.model, messages=messages, tools=tools, max_tokens=self.max_tokens,
        )

# 클라이언트는 필요한 능력만 의존
class RAGPipeline:
    def __init__(self, embedder: Embedder, generator: TextGenerator):
        self.embedder = embedder
        self.generator = generator
    # stream, call_with_tools, fine_tune은 알 필요 없음
```

`RAGPipeline`이 `Embedder`와 `TextGenerator` 두 개에만 의존하므로, 나중에 `ToolCaller`에 시그니처 변경이 일어나도 RAG 파이프라인은 영향이 없다.

## 3. Agent 역할(Role) 인터페이스 분리

에이전트 자체도 역할별로 쪼개면, 오케스트레이터(Orchestrator)가 "이 에이전트에게는 이 능력만 기대하면 된다"라고 명확하게 알 수 있다.

```python
class Searcher(ABC):
    @abstractmethod
    def search(self, query: str) -> list[dict]: ...

class Generator(ABC):
    @abstractmethod
    def generate(self, prompt: str) -> str: ...

class Planner(ABC):
    @abstractmethod
    def plan(self, goal: str) -> list[str]: ...

class Critic(ABC):
    @abstractmethod
    def critique(self, output: str) -> str: ...

class ResearchAgent(Searcher, Generator):
    """검색 + 응답 생성"""
    def search(self, query):  ...
    def generate(self, prompt): ...

class CodingAgent(Planner, Generator):
    """계획 수립 + 코드 생성"""
    def plan(self, goal):     ...
    def generate(self, prompt): ...

class ReviewerAgent(Critic):
    """리뷰 전용"""
    def critique(self, output): ...

# 오케스트레이터는 역할만 본다
def run_research_workflow(searcher: Searcher, writer: Generator) -> str:
    docs = searcher.search("retrieval-augmented generation")
    return writer.generate(f"다음 문서를 요약: {docs}")

# Agent를 역할에 끼워 넣을 수 있음 (둘 다 ResearchAgent가 될 수도, 둘이 다를 수도)
agent = ResearchAgent()
print(run_research_workflow(searcher=agent, writer=agent))
```

오케스트레이터 함수는 `Searcher`와 `Generator`만 본다. 새 메서드 `Critic.critique()`가 추가되어도 영향이 없다.

## 4. Memory의 읽기/쓰기 인터페이스 분리

장기 메모리는 보통 "읽기 전용 검색기"와 "쓰기 가능한 저장기"로 역할이 갈린다. 추론 단계에서는 읽기만 필요하지, 쓰기는 학습/업데이트 단계에서만 필요한 경우가 많다.

```python
class MemoryReader(ABC):
    @abstractmethod
    def history(self, session_id: str, limit: int = 20) -> list[dict]: ...

class MemoryWriter(ABC):
    @abstractmethod
    def append(self, session_id: str, message: dict) -> None: ...

class FullMemory(MemoryReader, MemoryWriter):
    """양쪽 다 지원"""
    def __init__(self):
        self.store: dict[str, list[dict]] = {}
    def history(self, session_id, limit=20):
        return self.store.get(session_id, [])[-limit:]
    def append(self, session_id, message):
        self.store.setdefault(session_id, []).append(message)

# 추론 전용 에이전트는 읽기만 받음
class InferenceAgent:
    def __init__(self, memory: MemoryReader):  # 쓰기 권한 없음
        self.memory = memory
```

`InferenceAgent`는 메모리에 함부로 쓸 수 없다. 권한을 타입 단계에서 분리한 것이다.

---

## ISP vs SRP (헷갈리기 쉬움)

ISP는 SRP와 헷갈리기 쉽다. 둘 다 "잘게 쪼개라"라고 말하기 때문이다. 그러나 적용 대상이 다르다.

| 관점 | **SRP (단일 책임)** | **ISP (인터페이스 분리)** |
| --- | --- | --- |
| 적용 대상 | 클래스/모듈 (구현체) | 인터페이스 (추상 타입) |
| 핵심 질문 | "이 클래스, 누구 때문에 바뀌나?" | "이 인터페이스, 너무 뚱뚱하지 않나?" |
| 위반 신호 | 한 클래스가 여러 이유로 수정됨 | 구현체에 `NotImplementedError`나 빈 메서드가 있음 |
| 해결 방법 | 책임별로 클래스 분리 | 역할별로 인터페이스 분리 |
| 보호 대상 | 변경 비용 | 사용자(클라이언트) |

쉽게 말해 SRP는 **"내가 변경되는 이유"**, ISP는 **"내가 강요받는 의무"**의 문제이다.

---

## ISP 체크리스트

내 코드를 살펴보며 스스로 답해보자.

- [ ] 어떤 구현체가 인터페이스의 메서드 중 일부를 `raise NotImplementedError`나 `pass`로 두고 있나? (ISP 위반의 직접 신호)
- [ ] 인터페이스에 메서드가 5개 이상 있는데, 모든 구현체가 5개를 다 의미 있게 채우나? (안 그러면 분할 후보)
- [ ] 클라이언트 함수의 시그니처에 인터페이스를 받는데, 그 함수가 실제로 호출하는 메서드는 1~2개뿐인가? (그렇다면 더 작은 인터페이스로 좁힐 여지)
- [ ] 인터페이스에 메서드 하나 추가했더니 무관한 구현체 N개를 다 고쳐야 했나? (ISP가 깨져 있다는 증상)
- [ ] 테스트용 Mock을 만들 때 "안 쓰는 메서드인데 어쩔 수 없이 stub"가 자주 나오나?

---

## 직접 확인하기

내가 짠 에이전트 코드에서 다음 두 가지를 검색해 보자.

1. `grep -rn "NotImplementedError" src/` : 빈 구현은 ISP/LSP 위반의 흔한 흔적이다. 이게 보이면 그 인터페이스를 더 잘게 쪼갤 수 있는지 의심할 차례다.
2. `grep -rn "raise NotImplemented" src/` : 위와 동일한 신호. 일부 구현체가 약속을 지킬 수 없다는 뜻이므로, 그 약속(인터페이스)을 분리하는 것이 정공법이다.

추가로, IDE에서 인터페이스를 정의한 ABC 파일을 열어 메서드 수를 세 보자. 평균 3개를 넘어가면 "이게 정말 한 역할인가?"를 다시 묻는다.

---

## 한마디 요약

> **"하나의 큰 약속에 모두 끌려가지 말고, 각자 필요한 약속만 따로 묶어라. 클라이언트가 안 쓰는 메서드 시그니처에 발목 잡히지 않게 하는 것이 ISP의 본질이다."**

관련 문서:
- [[SOLID원칙]]
- [[고수준 모듈과 저수준 모듈(SOLID DIP)]]
- [[SOLID_ O와 D의 차이]]
