---
생성날짜:
- 2026-01-27 01:54
마지막수정날짜:
- 2026-01-27-화요일 01:53
tags:
  - LangGraph
  - LangChain
  - Runnable
  - Agent
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## 왜 `agent.invoke()`로 감싸서 쓰는가

LangGraph로 에이전트를 만들면 **compiled graph 객체**(compile() 호출 결과)가 나온다. 이 객체에 `.invoke()`를 호출하면 그래프 전체가 실행됨. 그런데 실무에서는 이 그래프를 **클래스로 한 겹 감싸서** 쓰는 경우가 많다. 이게 "Wrapping(래핑)" 이고, 이유가 명확함.

---

## Invoke Wrapping 예시 코드

```python
class RealEstateAgent:
    def __init__(self, llm, tools):
        self.graph = self._build_graph(llm, tools)
    
    def _build_graph(self, llm, tools):
        from langgraph.prebuilt import create_react_agent
        
        # create_react_agent가 내부적으로 그래프 빌드
        # compile() 된 객체 반환 (invoke() 가능)
        return create_react_agent(llm, tools)
    
    def invoke(self, query):
        # 실제로는 self.graph.invoke() 호출
        
        # ↓ 이게 래핑 (self.graph.invoke를 감쌈)
        return self.graph.invoke({
            "messages": [{"role": "user", "content": query}]
        })

# 사용
agent = RealEstateAgent(llm, tools)
result = agent.invoke("강남 아파트 가격 추이 분석")

# 내부 흐름:
# agent.graph.invoke() 
# → 그래프 노드들 순회 (agent → tools → agent → END)
# → 최종 결과 반환
```

---

## `.invoke()`가 뭐기에 모든 LangChain 객체가 이걸 가지는가

Python의 매직 메서드(`__call__` 같은 언어 레벨 특수 메서드)는 아니다. **LangChain의 Runnable(실행 가능 객체) 표준 인터페이스**임. LangChain / LangGraph의 핵심 설계 원칙이 "모든 구성요소는 Runnable이어야 하고, Runnable은 같은 인터페이스를 제공한다"는 것.

| 메서드 | 용도 | 성격 |
|---|---|---|
| `.invoke(input)` | 동기 단일 실행 | 입력 → 출력 하나 |
| `.ainvoke(input)` | 비동기 단일 실행 | async/await로 병렬 처리 |
| `.stream(input)` | 동기 스트리밍 | 토큰 / 청크 단위로 yield |
| `.astream(input)` | 비동기 스트리밍 | async generator |
| `.batch(inputs)` | 동기 배치 실행 | 여러 입력 한번에 |

모든 compiled graph / chain / LLM / retriever는 이 5개 메서드를 동일하게 제공함. 그래서 다음이 가능함:

```python
# LangChain Runnable 체인 (표준 패턴)
chain = prompt | llm | parser
chain.invoke({"input": "질문"})           # 동기
await chain.ainvoke({"input": "질문"})    # 비동기

# LangGraph도 동일한 인터페이스
graph.invoke({"messages": [...]})
await graph.ainvoke({"messages": [...]})
```

Chain이든 Graph든 **호출하는 쪽이 구현 차이를 몰라도 됨**. 일상 비유로 하면 **리모컨의 전원 버튼** 같은 것. TV든 에어컨이든 선풍기든, 리모컨의 "전원" 버튼 위치는 같음. 내부 회로는 달라도.

---

## `self.graph.invoke()` 내부에서 일어나는 일

1. `agent.invoke("질문")` 호출
2. → `RealEstateAgent.invoke()` 메서드 실행 (겉껍질)
3. → 내부에서 `self.graph.invoke({...})` 호출 (실제 작업은 여기서)
4. → 그래프 노드 순회: `agent 노드` → `tools 노드` → `agent 노드` → `END`
5. → 각 노드마다 state(상태) 업데이트 누적
6. → 최종 state 반환

즉 **"쿼리 넣으면 즉시 답"** 처럼 보이지만, 중간에 **여러 노드를 거치며 state를 업데이트**하는 과정이 숨어있음.

---

## 왜 래핑하나: 3가지 이유

### 1. 인터페이스 단순화

래핑 없이 쓰면 사용자가 LangGraph 내부 구조(메시지 리스트 형태, state 스키마)를 알아야 함.

```python
# 래핑 안 하면 (사용자가 복잡한 구조 알아야 함)
agent = RealEstateAgent(llm, tools)
result = agent.graph.invoke({
    "messages": [{"role": "user", "content": "질문"}]  # 내부 포맷 노출
})

# 래핑하면 (간단)
agent = RealEstateAgent(llm, tools)
result = agent.invoke("질문")  # 문자열만 주면 끝
```

### 2. 전처리 / 후처리 삽입 지점 확보

```python
def invoke(self, query):
    # ① 전처리: 쿼리 정제, 로깅, 토큰 수 체크
    query = self._sanitize(query)
    
    # ② 실제 실행
    result = self.graph.invoke({"messages": [{"role": "user", "content": query}]})
    
    # ③ 후처리: 결과 파싱, 민감정보 마스킹, 메트릭 기록
    return self._format_output(result)
```

순수 `graph.invoke()`는 이 3단계가 노출되어 있음. 래퍼가 있으면 감춰짐.

### 3. 여러 그래프 / 폴백 그래프 조합

```python
class RealEstateAgent:
    def invoke(self, query):
        try:
            return self.main_graph.invoke(...)   # 메인 그래프
        except Exception:
            return self.fallback_graph.invoke(...)  # 장애 시 fallback
```

래퍼가 있으면 여러 그래프를 합쳐서 하나의 "에이전트"로 제공할 수 있음.

---

## 포함 관계

```
LangChain Runnable(표준 인터페이스)
│
├── LLM, Prompt, Parser, Retriever ...
├── Chain (prompt | llm | parser)
│
└── LangGraph compiled graph
    │
    └── 실무에서 래퍼 클래스로 한번 더 감싼 "Agent"
        (RealEstateAgent, SupportAgent 등)
```

즉 `RealEstateAgent` 같은 래퍼는 **Runnable 인터페이스를 재노출하는 얇은 어댑터**. 내부는 그대로 Runnable 생태계에 속해 있음.

---

### 한마디 요약

**"graph.invoke()가 실제 실행 진입점이고, 래퍼 클래스는 인터페이스 단순화 + 전후 처리 훅을 위한 얇은 껍질."**

관련: [[with_structured_output]], [[ReAct vs CoT의 핵심 차이]], [[AI AGENT TASK 분해 기준 단위]]
