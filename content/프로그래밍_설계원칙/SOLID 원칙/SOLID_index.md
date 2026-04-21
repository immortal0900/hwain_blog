# SOLID 원칙: AI Agent 개발자에게 필요한 것들

> 디자인 패턴이 "어떻게"라면, SOLID는 "왜 그렇게 해야 하는가"다.
> 5개 전부 필수. 서로 연결되어 있어서 하나만 빼기 어렵다.

---

## 5가지 원칙

| 원칙 | 정식 명칭 | 한 줄 설명 | AI Agent에서 언제? |
|------|----------|-----------|-------------------|
| [[SRP (단일 책임 원칙)\|**SRP**]] | Single Responsibility | 하나의 클래스는 하나의 책임만 | Agent, Tool, Prompt, Memory 각각 역할 분리. 하나의 클래스가 LLM 호출 + DB 저장 + 로깅을 다 하면 안 됨 |
| [[OCP (개방 폐쇄 원칙)\|**OCP**]] | Open-Closed | 확장에는 열려있고, 수정에는 닫혀있어야 함 | 새 LLM 추가할 때 기존 코드 안 건드리고 클래스 하나만 추가 |
| **LSP** | Liskov Substitution | 하위 클래스는 상위 클래스를 대체할 수 있어야 함 | OpenAIAgent를 ClaudeAgent로 바꿔도 시스템이 똑같이 동작해야 함 |
| [[역할별 인터페이스 분리(SOLID ISP)\|**ISP**]] | Interface Segregation | 인터페이스를 작게 쪼개라 | Agent에게 불필요한 메서드 강제하지 않기. "검색만 하는 Agent"에 generate() 메서드 강제 X |
| **DIP** | Dependency Inversion | 구체 클래스가 아니라 추상에 의존해라 | `agent.llm = OpenAI()`가 아니라 `agent.llm: LLMInterface`로 선언 |

---

## 원칙 간 연결 관계

- **Strategy 패턴** = OCP + DIP + LSP를 동시에 지키는 구현 방법
- **Factory 패턴** = DIP를 지키면서 객체를 생성하는 방법
- **Decorator 패턴** = OCP + SRP를 지키면서 기능을 추가하는 방법
- **Adapter 패턴** = LSP + DIP를 지키면서 인터페이스를 통일하는 방법

→ 디자인 패턴이 "왜 이렇게 생겼는지"는 전부 SOLID로 설명된다.

---

## 원칙별 심화 노트

- [[SRP (단일 책임 원칙)]]: actor(변경 요구자) 단위로 책임 나누기, AI Agent의 ChatAgent 쪼개기 실전 예시
- [[OCP (개방 폐쇄 원칙)]]: Tool/LLM Provider/Vector Store 확장 방법, Strategy·Decorator·Plugin 패턴 활용
- [[역할별 인터페이스 분리(SOLID ISP)]]: ISP 심화, Tool·LLM·Agent·Memory 능력별 인터페이스 분리
- [[고수준 모듈과 저수준 모듈(SOLID DIP)]]: DIP 심화, 의존 방향 역전의 구체적 구현
- [[SOLID_ O와 D의 차이]]: OCP와 DIP 증상이 비슷할 때 구분하는 법

## 전체 정리

- [[SOLID원칙]]: 5원칙 전체 개요와 AI Agent 체크리스트

