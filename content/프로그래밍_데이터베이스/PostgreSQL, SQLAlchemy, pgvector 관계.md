---
생성날짜:
  - 2026-01-27 02:59
마지막수정날짜:
  - 2026-01-27-화요일 02:58
tags:
  - 데이터베이스
  - PostgreSQL
  - SQLAlchemy
  - pgvector
  - ORM
별칭:
type:
Area/Reasource:
Project:
---
## 한 줄 핵심

**PostgreSQL**은 데이터가 저장되는 "창고 그 자체"이고, **SQLAlchemy**는 그 창고를 Python 언어로 편하게 관리하게 해주는 "통역사"이며, **pgvector**는 창고에 벡터 전용 선반을 추가해 주는 "확장 부품"이다. 세 가지는 경쟁 관계가 아니라 **층이 다른 협력 도구**.

## 포함 관계

```
Python 애플리케이션
   │
   └─ SQLAlchemy (ORM 라이브러리)
        │
        └─ psycopg2 같은 드라이버
             │
             └─ PostgreSQL (DB 서버)
                  └─ pgvector (확장 기능, 벡터 타입/인덱스 추가)
```

위에서 아래로 내려갈수록 "더 낮은 층, DB에 가까운 층"이다. SQLAlchemy는 PostgreSQL을 포함하는 게 아니라 **PostgreSQL 위에서 대화하는 Python 쪽 도구**임.

---

## 1. PostgreSQL (DBMS, Database Management System)

데이터를 실제로 저장/수정/삭제하는 **프로그램 그 자체**다.

- **역할**: 하드 드라이브에 데이터를 안전하게 보관하고 관리한다
- **언어**: 데이터 조작을 위해 **SQL(Structured Query Language, 표준 데이터베이스 언어)**을 사용
- **실행 위치**: 데이터베이스 서버에서 독립 프로세스로 상주
- **예시 쿼리**: `SELECT * FROM users;`

## 2. SQLAlchemy (ORM, Object Relational Mapping)

Python 객체와 DB 테이블을 자동 매핑해 주는 **라이브러리(파이썬 패키지)**다.

- **역할**: SQL을 직접 쓰지 않고 **Python 클래스/객체**를 다루듯 데이터를 처리하게 해준다
- **장점**: 코드가 간결해지고, DB를 PostgreSQL → MySQL 등으로 교체해도 수정 범위가 작음
- **실행 위치**: 애플리케이션(Python) 프로세스 내부
- **예시 코드**: `session.query(User).all()`

---

## 주요 차이 비교

| 구분 | PostgreSQL | SQLAlchemy |
|---|---|---|
| 정체 | 데이터베이스 서버/소프트웨어 | Python 라이브러리 |
| 주요 언어 | SQL | Python |
| 실행 위치 | 별도 DB 서버 프로세스 | 애플리케이션 프로세스 |
| 설치 | 서버 설치(예: apt install postgresql) | `pip install sqlalchemy` |
| 비유 | 책이 꽂혀 있는 도서관 | 책을 대신 찾아오는 사서 |

---

## 둘이 같이 동작하는 순서

```
[개발자]
  user.save()  ← Python 객체 조작
     │
     ▼
[SQLAlchemy]
  "저장해달라는 뜻이구나"
  → INSERT INTO users ... 로 자동 번역
     │
     ▼ (TCP 연결로 전송)
     │
[PostgreSQL]
  SQL 수신 → 디스크에 실제 저장
  → 결과 반환
```

1단계: 개발자가 Python으로 `user.save()` 호출
2단계: SQLAlchemy가 그 호출을 적절한 SQL 문자열로 번역
3단계: 드라이버(psycopg2 등)가 SQL을 TCP 패킷으로 감싸 전송
4단계: PostgreSQL이 실제 디스크 I/O 수행 후 결과 반환

---

## pgvector (PostgreSQL의 벡터 확장)

**PostgreSQL이라는 창고에 "벡터(Vector, 숫자 배열 형태의 의미 좌표) 데이터"를 저장할 수 있는 선반을 추가해 주는 Extension(확장 기능)**이다.

### pgvector가 하는 구체적 일

일반 컬럼은 "값이 같은가?"를 묻지만, pgvector는 **"얼마나 비슷한가?"**를 묻게 해준다.

- **벡터 타입(Type) 제공**: `[0.1, -0.2, 0.5, ...]` 같은 실수 배열을 `vector(1536)` 같은 컬럼에 저장 가능
- **유사도 계산 연산자**: 아래 표의 연산자로 "가장 가까운 이웃(Nearest Neighbor)" 검색
- **인덱싱(Indexing, 검색 지름길)**: HNSW, IVFFlat 인덱스로 수백만 건이어도 순식간에 탐색

### pgvector 거리 연산자

| 연산자 | 거리 종류 | 주 용도 |
|---|---|---|
| `<->` | L2 거리(유클리드) | 기본 거리 측정 |
| `<#>` | 음의 내적(Inner Product) | 정규화 벡터 비교 |
| `<=>` | 코사인 거리(Cosine) | OpenAI 임베딩 등 기본값 |
| `<+>` | L1 거리(맨해튼) | 드물게 사용 |

### 인덱스 방식 비교

| 관점 | HNSW | IVFFlat |
|---|---|---|
| 검색 속도 | 빠름 | 보통 |
| 빌드 시간 | 오래 걸림 | 빠름 |
| 메모리 사용 | 많음 | 적음 |
| 데이터 추가 시 | 자동 반영 | 리빌드 권장 |
| 추천 규모 | 대규모 프로덕션 | 중소규모 |

### 왜 별도 벡터 DB(Pinecone, Milvus) 대신 pgvector를 쓰나

- **통합 관리**: 사용자 이름(문자), 가입일(날짜), AI 임베딩(벡터)을 **한 테이블**에 넣고 **한 쿼리**로 조회 가능
- **학습 비용 절감**: 이미 아는 SQLAlchemy를 그대로 재활용
- **운영 단순화**: 관리할 DB 서버 개수가 하나로 유지됨

---

## SQLAlchemy와 pgvector가 함께 동작할 때

```
[AI 임베딩 단계]
  "사과" → OpenAI API → [0.12, 0.85, ...] (1536차원 벡터)
     │
     ▼
[SQLAlchemy]
  Vector 타입 컬럼으로 감싸서
  INSERT INTO items (name, embedding) VALUES ('사과', '[0.12, 0.85, ...]')
     │
     ▼
[PostgreSQL + pgvector]
  디스크에 저장
  → 검색 시 "<=>" 연산자로 코사인 거리 계산
  → "포도"보다 "배"가 더 가깝다고 판단
```

### SQLAlchemy에서 pgvector 쓰는 최소 코드

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import declarative_base
from pgvector.sqlalchemy import Vector

Base = declarative_base()

class Item(Base):
    __tablename__ = "items"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    embedding = Column(Vector(1536))  # OpenAI text-embedding-3-small 차원
```

### 직접 확인

```sql
-- PostgreSQL에 접속 후 확장 활성화 (관리자 권한 필요)
CREATE EXTENSION IF NOT EXISTS vector;

-- 확장 설치 확인
SELECT * FROM pg_extension WHERE extname = 'vector';
```

---

## 한마디 요약

> PostgreSQL은 서버, SQLAlchemy는 Python 쪽 통역사, pgvector는 서버 안에 덧붙는 벡터 선반이다. 세 가지가 층층이 협력해 "AI 벡터 검색 + 기존 RDB 기능"을 한 번에 제공함.

## 관련 노트

- [[AI_DB선택 참고]]
- [[TCP 연결과 IDLE 커넥션 pgBouncer psycopg2 SQLAlchemy]]
