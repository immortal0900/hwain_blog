# 코딩 & 앱 설계 원칙 — AI Agent 개발자에게 필요한 것들

> SOLID가 "클래스를 어떻게 설계하냐"라면, 여기는 "코드를 어떤 철학으로 짜냐"다.
> 별도 노트 만들 정도는 아니지만 전부 알고 있어야 하는 것들.

---

## 코딩 원칙

| 원칙 | 한 줄 설명 | AI Agent에서 언제? |
|------|-----------|-------------------|
| **DRY** (Don't Repeat Yourself) | 같은 코드 반복하지 마라 | LLM 호출 로직이 여러 곳에 복붙되어 있으면 모델 바꿀 때 전부 수정해야 함 |
| **KISS** (Keep It Simple, Stupid) | 단순하게 만들어라 | Agent에 불필요한 추상화 레이어 쌓지 않기. 클래스 3개면 될 걸 10개로 만들지 마라 |
| **YAGNI** (You Aren't Gonna Need It) | 지금 안 쓰는 거 미리 만들지 마라 | "나중에 10개 모델 지원할 수도 있으니까" 하고 미리 인터페이스 만들지 마라. 필요할 때 리팩토링 |
| **Separation of Concerns** | 관심사를 분리해라 | Prompt 관리 / LLM 호출 / 결과 파싱 / DB 저장을 각각 분리. SOLID의 상위 개념 |
| **Composition over Inheritance** | 상속보다 조합을 써라 | LangChain/LangGraph가 전부 조합 기반. Agent = Tool + Memory + LLM을 끼워 맞추는 구조 |
| **Law of Demeter** | 직접 아는 객체하고만 대화해라 | `agent.memory.db.connection.cursor.execute()` 이렇게 체인하지 마라. `agent.save()` 하나로 끝내라 |
| **Fail Fast** | 에러는 빨리 터뜨려라 | API 키 없으면 Agent 실행 시작할 때 바로 에러. 10분 돌다가 터지면 디버깅 지옥 |

---

## 앱 설계 원칙

| 원칙 | 한 줄 설명 | AI Agent에서 언제? |
|------|-----------|-------------------|
| **12-Factor App** | 클라우드 배포용 앱 설계 12가지 규칙 | .env로 설정 분리, 로그를 stdout 스트림으로, 프로세스 무상태 유지 등. 배포할 때 필수 |
| **Defensive Programming** | 입력을 믿지 마라, 항상 검증해라 | LLM 응답이 예상 형식 안 지킬 수 있음. JSON 파싱 전에 항상 검증. 사용자 입력도 마찬가지 |

---

## 특히 중요한 3개

1. **Composition over Inheritance** — LangChain/LangGraph 구조 자체가 조합 기반. 이거 모르면 프레임워크 설계 의도를 이해 못 함
2. **YAGNI** — Agent 만들 때 과설계 함정에 빠지기 쉬움. "일단 동작하게 → 필요할 때 리팩토링"이 정답
3. **12-Factor App** — 배포 안 하면 포트폴리오가 안 됨. .env 분리, 로그 관리, 포트 바인딩 등 기본기

---



- SOLID 원칙 — 클래스 레벨 설계 원칙
- 디자인 패턴 — 원칙을 구현하는 구체적 방법
- 아키텍처 패턴 — 시스템 전체 구조 레벨
