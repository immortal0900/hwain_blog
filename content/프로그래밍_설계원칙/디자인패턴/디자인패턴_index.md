# 디자인 패턴 — AI Agent 개발자에게 필요한 것들

> SOLID가 "왜"라면, 디자인 패턴은 "어떻게"다.
> GoF 23개 다 외울 필요 없음. 아래만 알면 실무 커버.

---

## 거의 매일 마주치는 패턴

| 패턴 | 한 줄 설명 | AI Agent에서 언제? |
|------|-----------|-------------------|
| **Strategy** | 알고리즘(행동)을 갈아끼울 수 있게 분리 | LLM 모델 교체, 검색 전략 교체 (BM25 ↔ 벡터 ↔ 하이브리드) |
| **Observer** | 이벤트 발생 → 구독자들에게 알림 | Langfuse 트레이싱, Telegram 알림, 로그 수집 |
| **Chain of Responsibility** | 요청을 처리기 체인에 순서대로 넘김 | Planner → Generator → Evaluator, 미들웨어 체인 |
| **Factory** | 객체 생성을 한 곳에서 관리 | Agent 종류별 생성, Tool 인스턴스 생성 |
| **Singleton** | 앱 전체에서 딱 하나만 존재 | DB 연결 풀, 설정 객체, LLM 클라이언트 인스턴스 |
| **Decorator** | 기존 기능에 추가 기능을 감싸서 덧붙임 | Python `@decorator`, retry/로깅/캐싱 래핑 |

---

## 자주 등장하는 패턴

| 패턴 | 한 줄 설명 | AI Agent에서 언제? |
|------|-----------|-------------------|
| **Builder** | 복잡한 객체를 단계별로 조립 | Prompt 조립 (system + context + user + few-shot), LangChain chain 구성 |
| **Adapter** | 호환 안 되는 인터페이스를 연결 | 서로 다른 LLM API 응답 형식을 통일된 포맷으로 변환 |
| **Facade** | 복잡한 내부를 간단한 인터페이스로 감춤 | 여러 API/DB/도구를 하나의 `agent.run()`으로 묶기 |
| **Template Method** | 전체 흐름은 고정, 세부 단계만 교체 | Agent 실행 루프 (init → plan → act → reflect) 중 act만 커스텀 |
| **State** | 상태에 따라 행동이 달라짐 | Agent 상태 관리 (idle → thinking → acting → waiting_approval) |
| **Pub/Sub** | Observer 확장판. 메시지 큐 기반 비동기 통신 | 멀티 Agent 간 메시지 전달, 이벤트 버스 |

---

## 알아두면 좋은 패턴

| 패턴 | 한 줄 설명 | AI Agent에서 언제? |
|------|-----------|-------------------|
| **Proxy** | 대리인이 대신 처리 (접근 제어, 캐싱) | LLM 응답 캐싱 프록시, API rate limit 관리 |
| **Iterator** | 컬렉션 내부 구조 몰라도 순회 가능 | 스트리밍 응답 처리, 청크 단위 데이터 순회 |
| **Command** | 요청 자체를 객체로 만듦 (실행취소 가능) | Agent 액션을 객체화 → 로깅/되돌리기/재실행 |
| **Mediator** | 객체들이 직접 통신 안 하고 중재자를 통함 | Orchestrator Agent가 sub-agent 간 통신 중재 |
| **Repository** | 데이터 접근 로직을 한 곳에 모음 | 메모리/벡터DB/그래프DB 접근을 통일된 인터페이스로 |

