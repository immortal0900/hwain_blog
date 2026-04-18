---
생성날짜:
  - 2026-04-16 04:24
마지막수정날짜:
  - 2026-04-16-목요일 04:24
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## 저장할 때 (Indexing Phase)

LLM이 Cypher를 직접 짜는 게 아니야. 프로덕션 파이프라인은 이렇게 동작해:

LLM은 상세한 프롬프트(지시사항, 예시, 스키마, 기존 엔티티, 출력 포맷)를 받아서 비구조화 텍스트로부터 엔티티와 관계를 추출하고, **구조화된 JSON 포맷**으로 반환한다. 이걸 Neo4j 같은 그래프 데이터베이스에 저장하고 원본 문서에 다시 연결한다. [Neo4j](https://neo4j.com/developer/genai-ecosystem/importing-graph-from-unstructured-data/)

즉 흐름은:

```
텍스트 → LLM (엔티티/관계 추출) → JSON 출력 → 코드가 Cypher MERGE문으로 변환 → Neo4j 저장
```

LangChain의 llm-graph-transformer 모듈을 사용해서 텍스트로부터 엔티티와 관계를 추출하고, 이를 그래프에 저장한 뒤 원본 청크와 연결한다. [Neo4j](https://neo4j.com/labs/genai-ecosystem/llm-graph-builder/) LLM은 "이 텍스트에서 엔티티와 관계를 뽑아줘"라는 역할만 하고, 실제 `MERGE (e:Entity {name: ...})` 같은 Cypher는 **코드가 프로그래밍적으로 생성**하는 거야.

## 검색할 때 (Retrieval Phase)

여기서 두 가지 방식이 갈려:

**방식 A — Text2Cypher**: 사용자 질문 → LLM이 Cypher 쿼리를 생성 → Neo4j에서 실행. 이게 화인이 말한 "LLM이 쿼리를 만들게 하는 방식"이야. 가장 유연하지만 가장 신뢰도가 낮은 패턴이다. [GraphRAG](https://graphrag.com/reference/graphrag/text2cypher/)

**방식 B — 벡터 매칭 + 프로그래밍적 순회**: 쿼리를 임베딩 → 엔티티 임베딩과 유사도 검색 → 매칭된 노드에서 **미리 정의된 Cypher 템플릿**으로 순회. LLM이 Cypher를 짜는 게 아니라 고정된 패턴을 실행하는 거야.

**방식 C — Graphiti 스타일**: BM25 + 벡터 인덱스로 노드/엣지를 찾고 그래프를 순회하며, 검색 시점에 LLM 호출을 전혀 하지 않는다. [Neo4j](https://neo4j.com/blog/developer/graphiti-knowledge-graph-memory/)

## 그래서 화인의 원래 질문에 다시 답하면

"벡터 기반으로 엔티티를 매칭하고 LLM이 Cypher 쿼리를 만드는 방식(Text2Cypher)"을 쓴다면 — 엔티티 진입점에서 벡터를 이미 쓰고 있으니 별도 벡터 검색은 중복이 맞고, **키워드 검색을 추가하는 건 엔티티 별칭 불일치 fallback 용도로 맞다.**

다만 현업에서는 Text2Cypher 자체의 불안정성 때문에 방식 B나 C를 더 선호하는 추세야.