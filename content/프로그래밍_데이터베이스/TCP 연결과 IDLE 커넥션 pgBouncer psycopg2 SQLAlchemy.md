---
생성날짜:
  - 2026-03-20 00:31
마지막수정날짜:
  - 2026-03-20-금요일 00:30
tags:
  - 데이터베이스
  - 커넥션풀
  - pgBouncer
  - SQLAlchemy
  - psycopg2
  - TCP
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## TCP 연결(TCP Connection)이란

두 컴퓨터가 데이터를 주고받기 위해 만드는 **통신 회선**이다.

```
[Railway 서버]  ←――― TCP 연결 ―――→  [Supabase DB]
   IP: 1.2.3.4                        IP: 5.6.7.8
   Port: 54321                         Port: 5432
```

- TCP 연결이 성립되면 양쪽이 서로의 IP와 포트(Port, 프로세스별 통신 창구 번호)를 기억한다
- 이 회선을 통해 SQL 쿼리를 보내고 결과를 받음
- 웹 브라우저 접속이든 앱의 DB 접속이든 모두 TCP 연결이 기반

### TCP 연결은 만드는 데 비용이 든다

- **3-way handshake(세 번의 손짓, 연결 수립 절차)**: SYN → SYN-ACK → ACK 3단계 확인을 거친다
- DB의 경우 TLS 협상과 인증까지 포함하면 수십 ~ 수백 ms가 걸림
- 그래서 SQLAlchemy는 한 번 만든 TCP 연결을 **풀(Pool, 여러 개 모아둔 대기 주머니)**에 보관해 두고 재사용한다

일상 비유: 택배 기사와 매번 처음부터 인사하고 주소 확인하는 대신, 자주 오는 기사는 "단골 기사" 목록에 올려놓고 바로 맡기는 것과 같다.

---

## idle 커넥션(유휴 커넥션)

**TCP 연결은 살아 있지만 아무 데이터도 오가지 않는 상태**다.

```
시간    상태                  TCP 연결    SQL 쿼리
00:00   검색 실행             살아있음     SELECT ... → 결과 반환
00:01   결과 받음             살아있음     (없음) ← idle 시작
00:02   다른 에이전트 작업 중  살아있음     (없음)
  :
13:00   13분째 쿼리 없음      살아있음     (없음) ← 13분간 idle
        pgBouncer: "이 커넥션 끊겠다"
```

### pgBouncer가 idle 커넥션을 끊는 이유

- DB 서버의 동시 접속 수에는 한계가 있다 (PostgreSQL 기본 max_connections=100 수준)
- 놀고 있는 커넥션이 자리를 차지하면 다른 클라이언트가 새로 접속 못 함
- 일정 시간(보통 10 ~ 15분) 쿼리가 없으면 "이 클라이언트는 더 이상 안 쓰나 보다"로 판단해 회수한다

---

## 각 역할 한 줄 정의

| 이름 | 한 줄 정의 | 층 위치 | 비유 |
|---|---|---|---|
| PostgreSQL | 데이터를 실제 저장하는 **DB 서버** | 가장 안쪽 | 은행 금고 |
| pgvector | PostgreSQL에 벡터 유사도 기능을 더한 **확장** | PostgreSQL 내부 | 금고에 설치한 특수 검색 장치 |
| pgBouncer | DB 앞에서 커넥션을 관리하는 **중개자(Connection Pooler)** | DB 앞단 | 금고실 앞 접수 창구 |
| SQLAlchemy | Python 코드에서 SQL을 만들어 보내는 **ORM/엔진** | 애플리케이션 내부 | 요청서를 작성하는 도구 |
| psycopg2 | SQLAlchemy가 PostgreSQL과 실제 TCP 통신할 때 쓰는 **드라이버(Driver)** | SQLAlchemy 내부 | 요청서를 창구까지 배달하는 배달원 |

### 포함 관계

```
Python 애플리케이션
├─ SQLAlchemy (ORM 엔진 + 커넥션 풀)
│   └─ psycopg2 (DB-API 드라이버, 실제 TCP 통신 담당)
│
└─ TCP 연결 ────→ pgBouncer (커넥션 중개) ────→ PostgreSQL + pgvector
```

- **SQLAlchemy는 psycopg2를 포함함**: SQLAlchemy 자체는 SQL을 만들 뿐이고, 실제 네트워크 송신은 psycopg2가 맡는다
- **pgBouncer는 PostgreSQL 앞에 서 있음**: 애플리케이션 입장에서는 pgBouncer가 PostgreSQL처럼 보인다

---

## 데이터가 흐르는 순서

```
[Python 코드]
  "대치동 관련 정책 문서 검색해줘"
     │
     ▼
[SQLAlchemy]  (1단계: ORM → SQL 변환)
  → "SELECT * FROM vectors WHERE similarity(...) > 0.8"
     │
     ▼
[psycopg2]  (2단계: SQL → TCP 패킷 직렬화)
  → TCP 소켓으로 네트워크 전송
     │
     ▼ (TCP 연결)
     │
[pgBouncer]  (3단계: 커넥션 검증/중개)
  → "이 클라이언트 세션이 유효한가?"
  → 유효하면 뒤쪽 PostgreSQL로 전달
     │
     ▼
[PostgreSQL + pgvector]  (4단계: 실제 실행)
  → SQL 실행 → 유사도 높은 문서 반환
```

각 단계가 "상위 층 → 하위 층"으로 내려가며 책임이 좁아지고, 결과는 반대 방향으로 올라간다.

---

## 대표적인 "죽은 커넥션" 에러 시나리오

### 상황 재연

| 이름 | 이 에러에서 한 일 |
|---|---|
| SQLAlchemy | 풀에 커넥션이 있으니 "유효함"으로 판단 → **실제로는 죽은 커넥션을 꺼냄** (원인 제공) |
| psycopg2 | TCP keepalive 패킷(연결 유지용 ping)을 30초마다 보냄 → pgBouncer에는 의미가 없었다 (방어 실패) |
| pgBouncer | 13분간 SQL 쿼리가 없음 → 세션 종료 (직접 원인) |
| pgvector | `engine_args` 없이 엔진 생성 → SQLAlchemy 기본값(`pre_ping=False`) 적용 (설정 누락) |

### 왜 TCP keepalive로는 부족한가

- TCP keepalive는 **OS 레벨**의 "이 소켓 살아 있나?" ping이다
- pgBouncer는 **애플리케이션 레벨**에서 "실제 SQL이 13분간 없었다"로 판단함
- 두 레이어가 달라서 keepalive가 통해도 pgBouncer는 세션을 종료할 수 있다

### 해결책: `pool_pre_ping=True`

SQLAlchemy 엔진 생성 시 옵션 한 줄이면 방어 가능함.

```python
from sqlalchemy import create_engine

engine = create_engine(
    DATABASE_URL,
    pool_pre_ping=True,     # 커넥션 꺼낼 때마다 SELECT 1로 생존 확인
    pool_recycle=300,       # 300초마다 커넥션 강제 재생성
    pool_size=5,
    max_overflow=10,
)
```

- `pool_pre_ping=True`: 풀에서 커넥션을 꺼내기 직전 **가벼운 쿼리로 살아있는지 검사**한다. 죽었으면 버리고 새로 생성
- `pool_recycle=300`: pgBouncer의 idle 타임아웃(보통 600초)보다 짧게 설정해 만료 전에 재사용을 중단시킴

### 직접 확인

```python
# 풀 상태 찍어보기
print(engine.pool.status())
# "Pool size: 5  Connections in pool: 3 ..." 형태로 출력된다
```

---

## 한마디 요약

> TCP 연결은 만드는 비용이 크니 풀에 보관한다. 그런데 pgBouncer는 idle(쿼리 없음) 커넥션을 정리하기 때문에, SQLAlchemy가 이를 모르고 죽은 커넥션을 꺼내면 에러가 난다. `pool_pre_ping`과 `pool_recycle`로 방어하면 끝.

## 관련 노트

- [[PostgreSQL, SQLAlchemy, pgvector 관계]]
- [[AI_DB선택 참고]]
- [[redis 와 celery]]
