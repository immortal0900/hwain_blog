---
생성날짜:
- 2026-04-21 15:34
마지막수정날짜:
- 2026-04-21-화요일 15:34
tags:
- 설계원칙
- 디자인패턴
- Repository
- DDD
- AI에이전트
별칭:
- Repository
- 레포지토리 패턴
- 저장소 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# Repository 패턴 (데이터 접근 통일)

Repository(리포지토리, 저장소)는 도메인 로직과 데이터 저장소(DB, 파일, 외부 API) 사이에 **컬렉션처럼 생긴 중간 계층**을 두는 패턴이다. 상위 코드는 "엔티티를 넣어라, 꺼내라, 지워라"만 알면 되고, 실제 SQL/NoSQL/HTTP 호출은 Repository 구현체가 숨긴다.

> GoF 23패턴에는 없지만, Martin Fowler의 PoEAA(Patterns of Enterprise Application Architecture, 2002)와 DDD(Domain-Driven Design, 도메인 주도 설계)에서 정립된 아키텍처 패턴.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 아키텍처/도메인 패턴 (엄밀히 GoF 아님) |
| 핵심 아이디어 | "메모리 속 컬렉션"처럼 쓰는 데이터 접근 인터페이스 |
| 풀려는 문제 | SQL/ORM/API 호출이 도메인 코드에 섞임 |
| 상위 계층 체감 | `users.get_by_id(1)` 수준의 단순함 |
| 연관 개념 | DAO(Data Access Object), Unit of Work, ORM |

## 일상 비유

도서관 사서가 Repository다. 책이 서가 어디에 꽂혀 있는지, 지하 보관소에 있는지, 다른 도서관에서 대출해 오는지 이용자는 몰라도 된다. "이 책 주세요" 하면 사서가 알아서 찾아준다. 도서관이 디지털 시스템으로 바뀌어도 이용자 창구 동작(제목/저자로 요청)은 그대로다.

## 구조

```
도메인 계층 (비즈니스 로직)
       │
       ▼
  Repository (인터페이스)
       ▲           ▲
       │           │
 SQLRepository   InMemoryRepository   VectorRepository
   (운영용)       (테스트용)           (RAG용)
```

상위 계층은 인터페이스에만 의존. 구현 교체는 DI(Dependency Injection, 의존성 주입)로 처리. [[SOLID원칙]]의 DIP가 그대로 적용되는 구조.

## 표준 메서드 시그니처

Repository는 보통 컬렉션 느낌의 메서드를 제공한다.

| 메서드 | 역할 |
| --- | --- |
| `add(entity)` / `save(entity)` | 새 엔티티 저장 |
| `get_by_id(id)` | 식별자로 하나 조회 |
| `find(query)` / `list(...)` | 조건으로 여러 개 조회 |
| `delete(entity)` | 삭제 |

쿼리 메서드가 많아질 것 같으면 Specification 패턴과 결합하거나 별도 QueryService를 분리하는 게 정석.

## AI 에이전트 개발 예시 1: Conversation Repository

에이전트가 유저와 주고받은 대화를 저장/조회할 때, 저장소 선택(Postgres, SQLite, Redis, 파일)을 도메인 로직에서 떼어낸다.

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Message:
    id: str
    conversation_id: str
    role: str        # "user" or "assistant"
    content: str
    created_at: datetime

class ConversationRepository(ABC):
    @abstractmethod
    def add(self, msg: Message) -> None: ...
    @abstractmethod
    def get_history(self, conversation_id: str, limit: int = 20) -> list[Message]: ...
    @abstractmethod
    def delete_conversation(self, conversation_id: str) -> None: ...

# 구현 1: Postgres
class PostgresConversationRepository(ConversationRepository):
    def __init__(self, conn):
        self.conn = conn

    def add(self, msg):
        self.conn.execute(
            "INSERT INTO messages (id, conv_id, role, content, created_at) VALUES (%s,%s,%s,%s,%s)",
            (msg.id, msg.conversation_id, msg.role, msg.content, msg.created_at),
        )

    def get_history(self, conversation_id, limit=20):
        rows = self.conn.execute(
            "SELECT * FROM messages WHERE conv_id=%s ORDER BY created_at DESC LIMIT %s",
            (conversation_id, limit),
        ).fetchall()
        return [self._row_to_msg(r) for r in rows]

    def delete_conversation(self, conversation_id):
        self.conn.execute("DELETE FROM messages WHERE conv_id=%s", (conversation_id,))

# 구현 2: 테스트용 인메모리
class InMemoryConversationRepository(ConversationRepository):
    def __init__(self):
        self._store: list[Message] = []

    def add(self, msg):
        self._store.append(msg)

    def get_history(self, conversation_id, limit=20):
        filtered = [m for m in self._store if m.conversation_id == conversation_id]
        return sorted(filtered, key=lambda m: m.created_at, reverse=True)[:limit]

    def delete_conversation(self, conversation_id):
        self._store = [m for m in self._store if m.conversation_id != conversation_id]
```

도메인 로직은 구현체가 바뀌어도 흔들리지 않음.

```python
class ChatService:
    def __init__(self, repo: ConversationRepository, llm: LLMClient):
        self.repo = repo
        self.llm = llm

    def reply(self, conv_id: str, user_msg: str) -> str:
        history = self.repo.get_history(conv_id)
        answer = self.llm.chat(history + [user_msg])
        self.repo.add(Message(...))
        return answer

# 프로덕션
service = ChatService(PostgresConversationRepository(conn), OpenAIClient())
# 테스트
service = ChatService(InMemoryConversationRepository(), FakeLLM())
```

## AI 에이전트 개발 예시 2: Vector Repository (RAG)

RAG 파이프라인에서 벡터 저장소(Pinecone, Weaviate, pgvector, FAISS)가 자주 바뀜. Repository로 감싸 두면 교체가 한 줄이다.

```python
@dataclass
class Document:
    id: str
    content: str
    embedding: list[float]
    metadata: dict

class VectorRepository(ABC):
    @abstractmethod
    def upsert(self, doc: Document) -> None: ...
    @abstractmethod
    def search(self, query_vec: list[float], k: int = 5) -> list[Document]: ...
    @abstractmethod
    def delete(self, doc_id: str) -> None: ...

class PineconeRepository(VectorRepository):
    def __init__(self, index):
        self.index = index
    def upsert(self, doc):
        self.index.upsert([(doc.id, doc.embedding, doc.metadata)])
    def search(self, query_vec, k=5):
        res = self.index.query(vector=query_vec, top_k=k, include_metadata=True)
        return [self._hit_to_doc(h) for h in res.matches]
    def delete(self, doc_id):
        self.index.delete(ids=[doc_id])

class FAISSRepository(VectorRepository):
    # 동일 인터페이스, 다른 구현
    ...
```

검색 엔진을 FAISS에서 Pinecone으로 이사 가도 RAG 파이프라인 코드는 한 줄도 안 바뀜.

## AI 에이전트 개발 예시 3: 프롬프트 Repository

프롬프트(prompt, LLM에게 주는 입력 템플릿)를 코드에 하드코딩하면 바꿀 때마다 배포 필요. Repository로 빼면 DB/파일/Git에서 읽어오게 할 수 있음.

```python
class PromptRepository(ABC):
    @abstractmethod
    def get(self, name: str, version: str = "latest") -> str: ...

class FilePromptRepository(PromptRepository):
    def __init__(self, root: str):
        self.root = root
    def get(self, name, version="latest"):
        path = f"{self.root}/{name}/{version}.txt"
        return open(path, encoding="utf-8").read()

class LangfusePromptRepository(PromptRepository):
    # Langfuse 같은 프롬프트 관리 플랫폼에서 가져오기
    ...
```

## 왜 이렇게 하는가

1. **도메인과 저장소 해방**: 도메인 로직이 SQL, HTTP, ORM 세부를 몰라도 됨. 책임 분리(SRP) 실현.
2. **테스트 용이성**: 인메모리 구현만 하나 두면 DB 없이도 비즈니스 로직 단위 테스트 가능. CI 속도 수십 배 빨라짐.
3. **저장소 교체 비용 최소화**: 스타트업에서 SQLite → Postgres → MongoDB 식으로 이사 가더라도 Repository 구현체만 갈아끼우면 됨.
4. **ORM의 독성 완화**: ORM 쿼리가 도메인 코드 곳곳에 박혀 있으면 나중에 ORM 이주(SQLAlchemy → Tortoise 등)가 지옥. Repository가 이걸 한 겹 가려 줌.

### 대안 대비 트레이드오프

| 접근 | 장점 | 단점 |
| --- | --- | --- |
| ORM 직접 호출 | 초반 간단, 쿼리 자유도 높음 | 도메인 코드에 쿼리 섞임, 교체 불가 |
| Repository | 책임 분리, 테스트/교체 쉬움 | 래핑 계층 추가, 단순 쿼리도 메서드화 필요 |
| DAO(Data Access Object) | 테이블 단위 접근 | 도메인 엔티티 개념이 약함 |
| Active Record (Rails 스타일) | 코드 짧음 | 도메인과 저장소 강결합 |

**Repository vs DAO 차이**: DAO는 "테이블 한 개 to 메서드들" 성격이 강함. Repository는 "애그리거트(도메인 객체 묶음) 한 단위 to 컬렉션 인터페이스" 성격. 쉽게 말해 Repository가 더 도메인 친화적.

## 흔한 함정

1. **Repository 안에 비즈니스 로직 넣기**: 예를 들어 "활성 유저만 조회" 같은 정책을 Repository에 박으면 Repository가 뚱뚱해진다. 쿼리는 제공하되, 정책 판단은 도메인 서비스로 빼라.
2. **N+1 쿼리**: `get_by_id`를 루프에서 N번 호출하지 않도록, `get_many(ids)` 같은 배치 메서드를 같이 두는 게 좋다.
3. **트랜잭션 경계**: 여러 Repository를 같은 트랜잭션으로 묶으려면 Unit of Work 패턴이 필요. Repository만으로는 부족함.

## 직접 확인

파이썬 생태계에서 Repository 스타일 구현은 SQLAlchemy 위에 얇은 래퍼를 두는 방식이 흔함. Django ORM은 모델이 `Manager`로 Repository 역할을 일부 수행하지만, 도메인/저장소 분리에는 약한 편.

```python
# 아주 단순한 Repository 예시
class UserRepository:
    def __init__(self, session):
        self.session = session
    def get_by_id(self, user_id):
        return self.session.query(User).get(user_id)
    def add(self, user):
        self.session.add(user)
```

## 포함 관계 및 관련 패턴

- Repository의 성립 근거는 [[SOLID원칙]]의 **DIP(의존성 역전)** + **SRP(단일 책임)**. 도메인 to 저장소 의존을 추상화로 뒤집는 게 핵심.
- Repository 위에서 "이 작업 전체를 하나의 단위로 커밋/롤백"하려면 Unit of Work 패턴과 함께 쓴다.
- 쿼리 조건을 객체화하려면 Specification 패턴 결합.
- [[Command 패턴(요청의 객체화)]]에서 실행되는 Command가 Repository를 써서 엔티티를 꺼내 조작하는 식으로 자주 결합.
- [[Proxy 패턴(대리 처리)]]로 Repository를 감싸면 캐시/감사 로그/리트라이를 공짜로 얹을 수 있음.

## 한마디 요약

> **"도메인 코드는 '저 책 좀요'만 말하게 해라. 사서(Repository)가 SQL이든 벡터 DB든 알아서 찾아온다. 사서를 교체하는 건 건물 기둥 바꾸는 일이 아니어야 한다."**

관련 문서:
- [[SOLID원칙]]
- [[Proxy 패턴(대리 처리)]]
- [[Command 패턴(요청의 객체화)]]
- [[Mediator 패턴(중재자)]]
