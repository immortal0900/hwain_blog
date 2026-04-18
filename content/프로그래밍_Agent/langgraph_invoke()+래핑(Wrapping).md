---
생성날짜:
- 2026-01-27 01:54
마지막수정날짜:
- 2026-01-27-화요일 01:53
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## invoke wrapping
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

# 내부:
# agent.graph.invoke() 
# → 그래프 노드들 순회 (agent → tools → agent → END)
# → 최종 결과 반환
```

## invoke()가 특별한 이유

**Python의 매직 메서드는 아니지만**, LangGraph의 **표준 인터페이스**:[](https://github.com/langchain-ai/langchain/discussions/24292)

- 모든 compiled graph는 `invoke()` 메서드 제공
- 동기 실행 (비동기는 `ainvoke()`)
- 입력 → 그래프 실행 → 출력의 표준화된 방식

```python
# LangChain Runnable 인터페이스 (표준 패턴)
chain = prompt | llm | parser
chain.invoke({"input": "질문"})  # 동기 실행
await chain.ainvoke({"input": "질문"})  # 비동기 실행

# LangGraph도 동일한 패턴
graph.invoke({"messages": [...]})  # 동기 실행
await graph.ainvoke({"messages": [...]})  # 비동기 실행
```

**결론**:

- `invoke()`는 **compiled graph 객체의 메서드**[](https://github.com/langchain-ai/langchain/discussions/20201)
- 내부적으로 **그래프 노드들을 순회하며 실행** (엣지 조건 따라)
- "쿼리 넣으면 바로 실행"이 맞지만, 중간에 **여러 노드를 거치며 상태를 업데이트**함
- 너 코드에선 보통 `self.graph.invoke()`를 래핑해서 사용

**내부 흐름**:
1. `agent.invoke("질문")` 호출
2. → `RealEstateAgent.invoke()` 메서드 실행
3. → 내부에서 `self.graph.invoke()` 호출 (실제 작업은 여기서)
4. → 결과 반환

## 왜 래핑하나?

## 1. 인터페이스 단순화
```python
# 래핑 안 하면 (사용자가 복잡한 구조 알아야 함)
agent = RealEstateAgent(llm, tools)
result = agent.graph.invoke({  # ← graph에 직접 접근
    "messages": [{"role": "user", "content": "질문"}]
})

# 래핑하면 (간단!)
agent = RealEstateAgent(llm, tools)
result = agent.invoke("질문")  # ← 쉬운 인터페이스
```