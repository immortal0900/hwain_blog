---
생성날짜:
  - 2026-04-16 04:24
마지막수정날짜:
  - 2026-04-16-목요일 04:24
tags:
  - GraphRAG
  - Neo4j
  - Text2Cypher
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## 저장할 때 (Indexing Phase)

착각했던 부분: LLM이 Cypher(Neo4j 전용 그래프 질의어)를 직접 짜서 저장하는 게 아니다. 프로덕션 파이프라인은 이렇게 돌아간다.

LLM은 상세한 프롬프트(지시사항, 예시, 스키마, 기존 엔티티, 출력 포맷)를 받아서 비구조화 텍스트로부터 엔티티와 관계를 추출하고, **구조화된 JSON 포맷**으로 반환한다. 이걸 Neo4j 같은 그래프 데이터베이스에 저장하고 원본 문서에 다시 연결한다. [Neo4j](https://neo4j.com/developer/genai-ecosystem/importing-graph-from-unstructured-data/)

흐름을 한 줄로 정리하면 이렇다.

```
텍스트 → LLM (엔티티/관계 추출) → JSON 출력 → 코드가 Cypher MERGE문으로 변환 → Neo4j 저장
```

LangChain의 `llm-graph-transformer` 모듈을 쓰면 텍스트에서 엔티티와 관계를 뽑고, 그래프에 저장한 뒤 원본 청크와 연결해준다. [Neo4j](https://neo4j.com/labs/genai-ecosystem/llm-graph-builder/) 즉 LLM은 "이 텍스트에서 엔티티와 관계를 뽑아줘" 역할만 하고, 실제 `MERGE (e:Entity {name: ...})` 같은 Cypher는 **코드가 프로그래밍적으로 생성**한다. 이래야 출력이 결정론적이라 재현 가능하다.

---

## 검색할 때 (Retrieval Phase)

검색 단계에서 LLM 관여 수준에 따라 세 방식으로 갈린다.

### 방식 A: Text2Cypher
사용자 질문 → LLM이 Cypher 쿼리를 직접 생성 → Neo4j에서 실행. "LLM이 쿼리를 만들게 하는 방식"이 이거다. 가장 유연하지만 가장 신뢰도가 낮은 패턴이다. [GraphRAG](https://graphrag.com/reference/graphrag/text2cypher/)

### 방식 B: 벡터 매칭 + 프로그래밍적 순회
쿼리를 임베딩(embedding, 의미 벡터화) → 엔티티 임베딩과 유사도 검색 → 매칭된 노드에서 **미리 정의된 Cypher 템플릿**으로 순회. LLM이 Cypher를 짜는 게 아니라 고정된 패턴을 실행한다. 신뢰도와 성능 모두 A보다 안정적이다.

### 방식 C: Graphiti 스타일
BM25 + 벡터 인덱스로 노드/엣지를 찾고 그래프를 순회하며, 검색 시점에 LLM 호출을 전혀 하지 않는다. [Neo4j](https://neo4j.com/blog/developer/graphiti-knowledge-graph-memory/) 검색 레이턴시(latency, 응답 지연)와 비용 모두 최소화하는 구조.

| | 방식 A (Text2Cypher) | 방식 B (벡터+템플릿) | 방식 C (Graphiti) |
|---|---|---|---|
| 검색 시 LLM 호출 | O | X (템플릿) | X |
| 유연성 | 높음 | 중간 | 낮음 |
| 신뢰도 | 낮음 | 중간 | 높음 |
| 비용/속도 | 비쌈/느림 | 저렴 | 가장 저렴 |

---

## 내가 궁금했던 질문에 다시 답하면

> "벡터 기반으로 엔티티를 매칭하고 LLM이 Cypher 쿼리를 만드는 방식(Text2Cypher)"을 쓴다면?

- 엔티티 진입점에서 벡터를 이미 쓰고 있으니 별도 벡터 검색은 중복이 맞다.
- **키워드 검색을 추가하는 건 엔티티 별칭(alias) 불일치 fallback 용도**로 맞다.

다만 현업에서는 Text2Cypher 자체의 불안정성 때문에 방식 B나 C를 더 선호하는 추세다. LLM이 만든 Cypher가 빈 결과를 뱉거나 엉뚱한 경로를 타는 케이스가 운영에서 계속 문제가 된다.

## 한마디 요약
저장은 LLM → JSON → 코드가 Cypher 생성이 표준이고, 검색은 유연성(A)보다 신뢰성(B, C)을 택하는 게 현업 트렌드다.

## 연관노트
- [[hybride graph rag시 vector 기반 엔티티 추출이면 벡터 검색을 안해도 되는가]]
- [[neo4j에 벡터 저장이 가능한지]]
