# 아키텍처 패턴 — AI Agent 개발자에게 필요한 것들

> 디자인 패턴이 "코드를 어떻게 짜냐"라면, 아키텍처 패턴은 "시스템 전체를 어떻게 구성하냐"다.

---

## 거의 매일 마주치는 패턴

| 패턴 | 한 줄 설명 | AI Agent에서 언제? |
|------|-----------|-------------------|
| **Event-Driven** | 이벤트 발생 → 반응 | Telegram 승인 게이트, sprint FAIL 시 자동 중단, Agent의 "관찰→행동" 루프 전체 |
| **Pipeline** | 데이터가 단계별로 흘러감 | Planner→Generator→Evaluator, RAG의 Query→Retrieval→Reranking→Generation |
| **Orchestrator-Worker** | 조율자 하나가 여러 작업자를 관리 | THE_FORGE 구조, LangGraph supervisor agent가 sub-agent 지시 |

---

## 자주 등장하는 패턴

| 패턴 | 한 줄 설명 | AI Agent에서 언제? |
|------|-----------|-------------------|
| **Pub/Sub (시스템 레벨)** | 메시지 큐 기반 비동기 통신 | 멀티 Agent 간 메시지 전달, Langfuse 로그 발행→수집 |
| **API Gateway** | 외부 요청을 하나의 입구에서 받아서 라우팅 | 여러 LLM API를 하나의 인터페이스로 묶기, 인증/rate limit 중앙 관리 |

---

## 알아두면 좋은 패턴

| 패턴 | 한 줄 설명 | AI Agent에서 언제? |
|------|-----------|-------------------|
| **MVC** | Model-View-Controller. 데이터/화면/로직 분리 | FastAPI로 API 서버 만들 때 자연스럽게 적용 |
| **Microservices vs Monolith** | 서비스를 하나로 묶을지 쪼갤지 | 프로젝트 규모 커질 때 판단 (Agent 서버 / RAG 서버 / DB 서버 분리 여부) |
| **CQRS** | 읽기와 쓰기를 분리 | 데이터 많은 RAG 시스템에서 검색(읽기)과 인덱싱(쓰기) 분리 |

