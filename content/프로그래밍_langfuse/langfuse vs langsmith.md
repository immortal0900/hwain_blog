---
생성날짜:
- 2026-01-29 19:44
마지막수정날짜:
- 2026-01-29-목요일 19:44
tags:
  - langfuse
  - langsmith
  - 관측성
  - self-hosting
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
LangFuse와 LangSmith 둘 중 뭘 쓸지 정리하면서 Self-Hosting이 뭔지부터 명확히 잡아둔다.

참고: langfuse 공식문서 비교 페이지 - https://langfuse.com/faq/all/langsmith-alternative

## 1. LangFuse Self-Hosting이 뭔가

**Self-Hosting(셀프 호스팅, 자체 서버에 직접 설치해 운영하는 방식) = LangFuse 소프트웨어를 내 서버에 올려서 쓰는 것.**

```
┌─────────────────────────────────────────────────────────────┐
│                   LangFuse 사용 방식 비교                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [방법 1] LangFuse Cloud (권장)                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  내 애플리케이션                                       │  │
│  │  └─> LangFuse SDK                                     │  │
│  │       └─> https://cloud.langfuse.com (LangFuse 서버) │  │
│  │            └─> 데이터 저장 (LangFuse 관리)           │  │
│  └──────────────────────────────────────────────────────┘  │
│  장점: 설정 간단, 유지보수 불필요                           │
│  단점: 데이터가 LangFuse 서버에 저장됨                      │
│                                                             │
│  [방법 2] Self-Hosting (자체 호스팅)                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  내 애플리케이션                                       │  │
│  │  └─> LangFuse SDK                                     │  │
│  │       └─> http://내서버:3000 (내가 설치한 LangFuse)  │  │
│  │            └─> 내 데이터베이스에 저장                 │  │
│  └──────────────────────────────────────────────────────┘  │
│  장점: 데이터 주권 확보, 완전한 통제                        │
│  단점: 서버 설치/관리 필요, Docker 필요                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

핵심: **Self-Hosting을 쓰면 LangFuse 오픈소스 코드는 그대로 쓰지만, LangFuse Cloud 서비스는 거치지 않는다.** 즉 소프트웨어는 쓰되 클라우드는 안 쓰는 것.

### Self-Hosting 설정 예시

```yaml
# docker-compose.yml
services:
  # 기존 서비스들...
  postgres:
    image: postgres:15
    # ...
  
  langfuse-server:
    image: langfuse/langfuse:latest  # LangFuse 오픈소스 이미지
    depends_on:
      - postgres
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/langfuse
      - NEXTAUTH_SECRET=your-random-secret-key
      - NEXTAUTH_URL=http://localhost:3000
```

`.env`에서 엔드포인트만 교체하면 끝.
```bash
# Cloud 대신 내 서버 사용
LANGFUSE_HOST=http://localhost:3000  # 또는 http://my-server.com
```

---

## 2. LangSmith의 한계

### 왜 LangSmith는 내 환경에 맞지 않는가

| 항목 | LangSmith | LangFuse |
|------|-----------|----------|
| **Self-Hosting** | ❌ 불가능 | ✅ 가능 (오픈소스) |
| **클라우드 종속** | ✅ 필수 (LangChain 운영) | ⚠️ 선택사항 |
| **데이터 주권** | ❌ 불가능 | ✅ Self-hosting 시 가능 |
| **무료 티어** | 5,000 traces/월 | 50,000 observations/월 |
| **가격** | $39/월 (50K traces) | 무료 (Self-hosting) |
| **오픈소스** | ❌ 아님 | ✅ MIT 라이선스 |

### LangSmith를 쓰면 걸리는 지점

```python
# LangSmith는 환경 변수만 설정하면 자동 추적
LANGCHAIN_TRACING_V2=true  # 이미 .env에 있음
LANGCHAIN_API_KEY=lsv2_...
LANGCHAIN_PROJECT=memory_labyrinth

# 장점: 설정 매우 간단
# 단점:
# 1. 데이터가 무조건 LangChain 서버로 전송됨 (선택 불가)
# 2. Self-hosting 불가능
# 3. 비싼 가격 ($39/월 vs LangFuse 무료)
```

---

## 3. 내 선택

### 현재 프로젝트에 최적인 방법

```
┌─────────────────────────────────────────────────────────┐
│              1단계: LangFuse Cloud로 시작                │
├─────────────────────────────────────────────────────────┤
│  1. 빠른 시작 (5분 내 설정 완료)                        │
│  2. 무료 티어 충분 (50,000 observations/월)             │
│  3. 유지보수 불필요                                      │
│                                                         │
│  나중에 필요하면 Self-Hosting으로 전환 가능             │
│  (코드 변경 없이 LANGFUSE_HOST만 변경)                  │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│          2단계: 필요 시 Self-Hosting으로 전환            │
├─────────────────────────────────────────────────────────┤
│  언제 필요한가?                                          │
│  - 민감한 대화 데이터를 외부에 보내고 싶지 않을 때      │
│  - 완전한 데이터 통제가 필요할 때                        │
│  - 무료 티어 한도를 초과할 때                            │
│                                                         │
│  단점:                                                   │
│  - Docker Compose 설정 필요                             │
│  - PostgreSQL 추가 설치                                 │
│  - 서버 관리 필요                                        │
└─────────────────────────────────────────────────────────┘
```

---

## 4. 실전 적용 순서

### Phase 1: LangFuse Cloud로 시작

```bash
# .env 설정
LANGFUSE_SECRET_KEY=sk-lf-...  # cloud.langfuse.com에서 발급
LANGFUSE_PUBLIC_KEY=pk-lf-...
LANGFUSE_HOST=https://cloud.langfuse.com
```

장점
- 5분 내 바로 사용 가능
- 서버 관리 불필요
- 무료 (50,000 observations/월)

### Phase 2: Self-Hosting 전환 (필요할 때)

```yaml
# docker-compose.yml에 추가
services:
  langfuse:
    image: langfuse/langfuse:latest
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/langfuse
      - NEXTAUTH_SECRET=${LANGFUSE_NEXTAUTH_SECRET}
      - NEXTAUTH_URL=http://localhost:3000
    depends_on:
      - postgres
```

```bash
# .env 변경 (이것만!)
LANGFUSE_HOST=http://localhost:3000  # Cloud → Self-hosted

# 코드 변경 불필요!
```

---

## 5. LangSmith vs LangFuse 최종 비교

| 기준 | LangSmith | LangFuse |
|------|-----------|----------|
| 설정 난이도 | ⭐ (환경변수만) | ⭐⭐ (콜백 추가) |
| Self-Hosting | ❌ | ✅ |
| 무료 한도 | 5,000/월 | 50,000/월 |
| 가격 (유료) | $39/월 | 무료 (Self-host) |
| 데이터 주권 | ❌ | ✅ (Self-host) |
| 오픈소스 | ❌ | ✅ |
| LangGraph 지원 | ✅ 네이티브 | ✅ 콜백 |
| 커스텀 모델 가격 | ⚠️ 제한적 | ✅ 완전 지원 |

결론: 유연성과 비용 모두 LangFuse가 낫다.

---

## 한마디 요약

1. **Self-Hosting** = LangFuse 소프트웨어를 내 서버에 설치하는 것 (LangFuse Cloud 미사용)
2. **LangSmith 문제점** = Self-hosting 불가능, 비쌈, 오픈소스 아님
3. **내 선택** = LangFuse Cloud로 시작 → 필요 시 Self-hosting 전환

핵심 정리
- Self-hosting은 LangFuse Cloud를 안 쓰고 내 서버에 설치하는 방식
- LangSmith는 Self-hosting이 아예 불가능 (이게 가장 큰 단점)

## 연관노트
- [[langfuse_한번 요청시 나온 토큰등 추적 정보를 묶어서 보기]]
- [[langfuse_한번 호출한 분량 분리해서 호출]]
