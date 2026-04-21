---
생성날짜:
- 2026-02-23 23:08
마지막수정날짜:
- 2026-02-23-월요일 23:08
tags:
  - LLM
  - Prompting
  - Agent_Pattern
  - CoT
  - ReAct
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## CoT는 "생각만", ReAct는 "생각 + 행동 + 관찰"

두 용어가 혼용되어 쓰이지만 구조적으로 다른 패턴임. 가장 큰 차이는 **외부 행동(Act)의 유무**.

---

## 개념 비교

| 구분 | CoT (Chain of Thought) | ReAct (Reasoning + Acting) |
|---|---|---|
| 핵심 | 생각 단계를 명시적으로 나열 | 추론 → 외부 행동 → 관찰의 루프 |
| 외부 액션 | 없음 (순수 추론) | 필수 (Tool call, API 호출 등) |
| 구조 | Think → Answer | Think → Act → Observe → Think → ... |
| 상태 | Stateless(무상태) | Stateful(외부 관찰 결과를 누적) |
| 논문 | Wei et al. (2022) | Yao et al. (2022) |
| 대표 구현 | "Let's think step by step" 프롬프트 | LangGraph `create_react_agent`, AutoGPT |

---

## CoT(Chain of Thought, 사고 사슬)의 동작

**한 번의 LLM 호출 안에서 중간 생각을 텍스트로 풀어내는 패턴**. 외부 도구 호출 없이 "생각을 말로 풀면 답이 더 잘 나오더라"는 경험적 관찰에서 출발함.

```
[프롬프트] "17 × 24는 얼마인가? 단계별로 생각해보자."

[LLM 내부 출력]
먼저 17 × 20 = 340을 계산한다.
다음으로 17 × 4 = 68을 계산한다.
둘을 더하면 340 + 68 = 408이다.

[최종 답변] 408
```

- 장점: 추가 인프라 없이 프롬프트만으로 추론력 향상
- 단점: LLM 내부 지식에만 의존함. 실시간 데이터(주가, 날씨)나 외부 DB는 모름

---

## ReAct(Reasoning and Acting, 추론과 행동)의 동작

**LLM이 생각(Thought) → 행동(Action) → 관찰(Observation)을 반복하는 루프 패턴**. 외부 세계에 영향을 주거나 외부 정보를 끌어와야 풀리는 문제에 쓰인다.

```
[질문] "강남역 근처 오늘 영업 중인 피자집은?"

[Thought] 지도 API를 호출해야 한다.
[Action] search_places(query="강남역 피자", open_now=True)
[Observation] [피자헛 강남점, 도미노 강남역점, ...]

[Thought] 영업시간 재확인이 필요하다.
[Action] get_business_hours(place_id="피자헛 강남점")
[Observation] {"open": "11:00", "close": "23:00"}

[Thought] 충분한 정보를 모았다.
[Answer] 지금 영업 중인 곳은 피자헛 강남점과 도미노 강남역점이다.
```

- 장점: 실시간 정보 반영, 외부 시스템 조작 가능
- 단점: Tool 호출 실패 / Observation 파싱 오류 같은 실전 이슈가 많음. Latency 증가

---

## 판단 기준: NPC 내면 독백은 어느 쪽?

게임 NPC가 응답 전에 자신의 생각을 먼저 쓰는 패턴을 보면:

```
[NPC의 생각]
"나는 지금 주인공에게 화가 나있다. 
하지만 감정을 드러내면 안 된다. 
차갑게 대응해야 한다."

[실제 응답]
"...별로 상관없어요."
```

이것은 **외부 도구 호출이나 관찰 루프가 없는**, 순수한 내면 추론 → 응답 패턴임.

따라서 이것은 **CoT**다. ReAct가 되려면 반드시 "행동(Act)" 단계(메모리 검색, DB 조회 등)가 있어야 함.

---

## 더 정밀한 표현들

상황에 따라 더 구체적인 용어를 쓰기도 한다:

- **CoT (Chain of Thought)**: 일반적으로 가장 맞는 표현
- **Inner Monologue**: "내면의 독백", 캐릭터 / 페르소나 맥락에서 자주 쓰임
- **Scratchpad**: "초안 공간" 개념, LLM이 답을 내기 전 자유롭게 메모하는 영역
- **Persona-conditioned CoT**: 포트폴리오 / 논문에서 더 명확히 설명할 때

---

## 포함 관계

```
Agent Prompting Patterns
│
├── CoT (순수 추론만)
│   └── Inner Monologue, Scratchpad
│
└── ReAct (CoT + 외부 Action)
    └── 각종 Tool-use agent들 (LangGraph ReAct agent 등)
```

ReAct는 **"CoT + 외부 Action 루프"** 로 볼 수 있다. CoT가 ReAct의 Think 단계에 들어가는 셈.

---

### 한마디 요약

**"생각만 하면 CoT, 생각하고 도구도 쓰면 ReAct."**

관련: [[AI AGENT TASK 분해 기준 단위]], [[langgraph_invoke()+래핑(Wrapping)]]
