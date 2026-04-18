---
생성날짜:
  - 2026-04-16 04:08
마지막수정날짜:
  - 2026-04-16-목요일 04:07
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
**"벡터 기반 엔티티 추출 → Text2Cypher"가 별도 벡터 검색을 완전히 대체하지는 않는다.** 그리고 프로덕션에서는 거의 예외 없이 **vector + keyword + graph traversal(그래프 순회)** 3중 하이브리드를 쓴다.

## 왜 대체가 안 되는가

여기서 구분해야 할 게 두 가지 "벡터 검색"이야.

**① 쿼리 시점 엔티티 매칭용 벡터 검색** — 사용자 질문에서 엔티티를 뽑고, 그래프 DB에 있는 엔티티 임베딩(entity embedding)과 유사도 비교해서 "이 질문이 어떤 노드에 대한 건지" 찾는 것. 이건 **진입점(entry point)을 찾는 과정**이지, 답변에 필요한 컨텍스트를 수집하는 과정이 아니야.

**② 청크/문서 수준 벡터 검색** — 실제 텍스트 청크를 임베딩해서, 쿼리와 의미적으로 가까운 텍스트 조각을 가져오는 것. 이건 **컨텍스트 수집 과정**이야.

①만 하고 Text2Cypher(LLM이 Cypher 쿼리를 생성하는 방식)로 그래프를 탐색하면, 그래프에 명시적으로 저장된 관계만 가져올 수 있어. Text2Cypher 패턴은 가장 유연하지만, 동시에 가장 신뢰도가 낮은 패턴이기도 하다. [GraphRAG](https://graphrag.com/reference/graphrag/text2cypher/) 특히 사용자 질문에서 추출된 엔티티 별칭(alias)이 DB에 저장된 값과 정확히 일치하지 않으면 쿼리가 빈 결과를 반환하는 문제 [Neo4j](https://neo4j.com/blog/developer/graphrag-field-guide-rag-patterns/)가 있어.