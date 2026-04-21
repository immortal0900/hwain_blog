---
생성날짜:
- 2026-04-21 15:32
마지막수정날짜:
- 2026-04-21-화요일 15:32
tags:
- 설계원칙
- 디자인패턴
- Command
- GoF
- AI에이전트
별칭:
- Command
- 커맨드 패턴
- 명령 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# Command 패턴 (요청의 객체화)

Command(커맨드, 명령)는 "무엇을 해라"는 요청 자체를 **객체로 포장**하는 행위 패턴(Behavioral Pattern)이다. 요청이 객체가 되는 순간 큐에 쌓기, 로그에 남기기, 되돌리기(Undo), 다시 실행하기(Redo), 원격 전송 같은 일들이 다 가능해진다.

> GoF 23패턴 중 행위 패턴. "요청을 쏘고 끝내지 말고, 요청을 찍어서 보관하라"가 핵심.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 행위 패턴(Behavioral Pattern) |
| 핵심 아이디어 | 요청 = 객체, `execute()` 메서드로 실행 |
| 풀려는 문제 | 요청을 기록/큐잉/취소/재실행하고 싶다 |
| 요청자(Invoker) | Command를 들고 실행 타이밍만 결정 |
| 수신자(Receiver) | 실제 일을 하는 객체, Command가 참조 |

## 일상 비유

식당 주문서가 커맨드다. 손님이 "스테이크 미디움"이라고 말하면 웨이터가 주문서(Command 객체)에 적음. 주문서는 주방 칸(큐)에 꽂혀서 순서대로 처리되고, 실수가 있으면 주문서를 뽑아 취소(Undo)하고, 동일 주문이 필요하면 복사해서 다시 넣는다(Redo). 손님은 요리사와 직접 얘기하지 않음. 종이 한 장이 요청을 들고 다니는 것.

## 구조

```
Client ─생성─▶ Command(execute, undo)
                    │
Invoker ─호출─▶ execute()
                    │
                    ▼
               Receiver (실제로 일을 함)
```

- **Command**: 실행 메서드(`execute`) 와 선택적 `undo` 를 정의한 객체
- **Receiver**: 실제 작업을 수행하는 도메인 객체
- **Invoker**: Command를 호출하는 쪽 (버튼, 스케줄러, 큐 워커)
- **Client**: Command를 만들어서 Invoker에 끼워 주는 쪽

## AI 에이전트 개발 예시 1: Tool 실행의 커맨드화

에이전트가 LLM에게 받은 Tool 호출을 그때그때 실행하는 대신, Command 객체로 찍어 둔다. 이러면 감사 로그, 재실행, 배치 처리, 드라이 런(dry-run, 실행 없이 시뮬레이션)이 공짜로 따라온다.

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from datetime import datetime

class ToolCommand(ABC):
    @abstractmethod
    def execute(self) -> str: ...
    def undo(self) -> None:     # 선택, 되돌릴 수 있으면 구현
        raise NotImplementedError

@dataclass
class SendEmailCommand(ToolCommand):
    to: str
    subject: str
    body: str
    sent_id: str | None = None      # undo용 저장

    def execute(self):
        self.sent_id = mail_client.send(self.to, self.subject, self.body)
        return f"sent: {self.sent_id}"

    def undo(self):
        if self.sent_id:
            mail_client.recall(self.sent_id)     # Gmail의 "전송 취소" 같은 거

@dataclass
class SearchCommand(ToolCommand):
    query: str
    def execute(self):
        return google_search(self.query)
    # 검색은 부작용 없음, undo 필요 없음

# Invoker: 커맨드 실행과 이력 관리
class AgentInvoker:
    def __init__(self):
        self.history: list[ToolCommand] = []

    def run(self, cmd: ToolCommand) -> str:
        result = cmd.execute()
        self.history.append(cmd)
        return result

    def undo_last(self):
        if self.history:
            self.history.pop().undo()

# Client
invoker = AgentInvoker()
invoker.run(SearchCommand(query="커맨드 패턴"))
invoker.run(SendEmailCommand(to="a@b.c", subject="hi", body="..."))
invoker.undo_last()  # 방금 보낸 메일 취소 시도
```

AgentInvoker 코드는 어떤 도구가 들어오든 같은 방식으로 다룸. [[SOLID원칙]]의 OCP, SRP를 동시에 챙긴 모양.

## AI 에이전트 개발 예시 2: 작업 큐와 비동기 실행

요청이 객체라서 큐에 직렬화해 넣을 수 있음. LLM 호출, 임베딩, 파인튜닝 작업 같은 무거운 일을 큐 워커가 집어서 돌린다.

```python
import json
from queue import Queue

@dataclass
class EmbedCommand(ToolCommand):
    doc_id: str
    text: str
    def execute(self):
        vec = model.embed(self.text)
        vector_store.upsert(self.doc_id, vec)
        return self.doc_id

    def to_json(self):
        return json.dumps({"type": "embed", "doc_id": self.doc_id, "text": self.text})

queue: Queue[ToolCommand] = Queue()
queue.put(EmbedCommand(doc_id="d1", text="..."))
queue.put(EmbedCommand(doc_id="d2", text="..."))

# 워커 프로세스
while not queue.empty():
    cmd = queue.get()
    cmd.execute()
```

큐에 쌓인 순간 "언제" 실행할지, "어디서"(원격 워커, 로컬) 실행할지 자유. 요청을 객체로 만들지 않았다면 이게 불가능하다.

## AI 에이전트 개발 예시 3: Undo/Redo 이력

에이전트가 파일을 여러 개 수정하는 Task를 돌릴 때, 중간에 실패하면 지금까지 한 걸 되돌려야 한다.

```python
class WriteFileCommand(ToolCommand):
    def __init__(self, path, new_content):
        self.path = path
        self.new_content = new_content
        self.old_content: str | None = None

    def execute(self):
        self.old_content = read(self.path)       # 되돌리기 위해 백업
        write(self.path, self.new_content)

    def undo(self):
        if self.old_content is not None:
            write(self.path, self.old_content)

# 트랜잭션처럼 사용
executed: list[ToolCommand] = []
try:
    for cmd in plan:
        cmd.execute()
        executed.append(cmd)
except Exception:
    for cmd in reversed(executed):
        cmd.undo()                                # 실패 시 역순 롤백
    raise
```

## 왜 이렇게 하는가

1. **요청을 1급 객체로 다룸**: 저장, 비교, 복사, 전송이 가능. 로그, 재생, 시뮬레이션이 쉬워짐.
2. **Invoker와 Receiver 분리**: 버튼(Invoker)은 Command만 실행함. 무슨 Receiver에 어떤 작업이 가는지 모름. 덕분에 단축키, CLI, 스케줄러 같은 여러 Invoker가 동일 Command를 재사용 가능.
3. **Undo/Redo/감사 로그가 거의 공짜**: 실행한 Command를 리스트에 넣으면 그게 곧 히스토리.

### 대안 대비 트레이드오프

| 접근 | 장점 | 단점 |
| --- | --- | --- |
| 함수/메서드 직접 호출 | 간단, 코드 짧음 | 이력/큐/취소 전부 수제 구현 필요 |
| Command 객체화 | 이력, 큐잉, 취소, 직렬화가 구조적으로 풀림 | 클래스 수 증가, 간단한 앱에는 과함 |
| 이벤트 드리븐/Pub-Sub | 느슨한 결합, 비동기 | 흐름 추적 어려움, 디버깅 까다로움 |

## Command vs 다른 패턴 헷갈림 정리

| 패턴 | 다루는 것 | 핵심 질문 |
| --- | --- | --- |
| Command | 요청 자체(동사) | "이 요청을 객체로 찍어 보관할까?" |
| Strategy | 알고리즘 교체 | "이 단계 알고리즘을 갈아끼울까?" |
| Observer | 사건 알림 | "이 일이 일어나면 누구에게 알릴까?" |

## 직접 확인

버튼 UI가 있는 데스크톱 앱이든, LangChain 같은 에이전트 프레임워크든, 거의 다 Command의 변형을 쓴다. LangChain의 `AgentAction`은 (tool, tool_input) 쌍을 들고 있는 명령 객체나 다름없음.

```python
# LangChain 예시(개념)
from langchain.schema import AgentAction
action = AgentAction(tool="search", tool_input={"q": "서울 날씨"}, log="...")
# 이 객체가 바로 커맨드 역할. executor가 이걸 받아서 실행함.
```

## 포함 관계 및 관련 패턴

- Command를 담는 리스트 위에 [[Iterator 패턴(순회)]]를 얹으면 "명령 히스토리 순회" 가 자연스럽게 구성됨.
- [[Proxy 패턴(대리 처리)]]로 Command 실행을 감싸면 "실행 전 권한 체크", "실행 후 비용 기록" 같은 횡단 관심사(cross-cutting concern, 여러 기능에 걸쳐 반복되는 관심사)를 넣을 수 있음.
- 여러 Command를 오케스트레이션(orchestration, 여러 요청을 엮어 흐름을 조율)할 주체가 필요하면 [[Mediator 패턴(중재자)]] 등장.
- [[SOLID원칙]]의 SRP(Invoker는 실행 타이밍만, Command는 요청 내용만)와 OCP(새 명령 추가 시 기존 Invoker 수정 없음)를 잘 살리는 구조.

## 한마디 요약

> **"요청을 바로 실행하지 말고, 일단 종이에 적어서 들고 다녀라. 그러면 언제 돌릴지, 되돌릴지, 누가 대신 돌릴지 네가 정할 수 있다."**

관련 문서:
- [[SOLID원칙]]
- [[Iterator 패턴(순회)]]
- [[Proxy 패턴(대리 처리)]]
- [[Mediator 패턴(중재자)]]
