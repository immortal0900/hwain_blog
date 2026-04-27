---
생성날짜:
  - 2026-04-24 11:17
마지막수정날짜:
  - 2026-04-24-금요일 11:17
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
# RAG Pipeline 전체 흐름 정리: Keyword RAG vs Vector RAG

> 각 단계를 순서대로 나열하고, 2026년 현재 각 단계에서 많이 쓰이는 best-in-class 방법을 함께 정리한다. 

---

## 공통 전제: 왜 두 방식을 나눠서 봐야 하는가

Keyword RAG(키워드 기반 검색)와 Vector RAG(벡터 기반 검색)는 **인덱싱 산출물**과 **유사도 정의**가 근본적으로 다르다.

|항목|Keyword RAG|Vector RAG|
|---|---|---|
|인덱스 산출물|Inverted Index(역색인, 단어 -> 문서 ID 매핑)|Dense Vector Index(밀집 벡터 인덱스, 임베딩 공간)|
|유사도 정의|Term Frequency(단어 빈도) 기반 스코어링|Cosine/Dot Product(코사인/내적) 벡터 거리|
|강점|고유명사, 코드, 숫자, 약어 같은 exact match(정확 일치)에 강함|의미적 유사성, paraphrase(표현이 다른 같은 의미)에 강함|
|약점|동의어, 표현 차이에 약함|고유명사 오탐, 추상적 유사성으로 오매칭|
|대표 엔진|Elasticsearch, OpenSearch, PGroonga, BM25S, Tantivy|Qdrant, Weaviate, Milvus, pgvector, Neo4j Vector Index|

결국 실전에서는 두 결과를 **RRF(Reciprocal Rank Fusion, 순위 역수 결합)** 등으로 합치는 Hybrid RAG가 표준이지만, 각 파이프라인의 단계는 전혀 다르다.

---

## Part 1. Keyword RAG (BM25 계열) 전체 순서

### Indexing Phase (인덱싱 단계, 오프라인)

**1. Document Collection (문서 수집)** 원본 소스(PDF, HTML, Confluence, Notion, DB, Obsidian Vault 등)에서 문서를 모은다.

- 현재 best: 소스별 official connector(공식 커넥터)를 쓰되, 변경 감지(file hash, updated_at)를 함께 기록해서 incremental indexing(증분 인덱싱)을 지원하는 구조로 설계한다.

**2. Document Parsing (문서 파싱, 구조 복원)** 레이아웃, 표, 헤딩 계층을 살리면서 텍스트를 뽑는다.

- 현재 best: **Docling**(IBM, 2024년 말 공개, PDF/DOCX/PPTX 구조 보존 탁월), **Unstructured**, **LlamaParse**. 표가 많은 문서는 Docling이 사실상 표준으로 굳어가고 있다.

**3. Preprocessing (전처리)** Keyword RAG에서 **가장 중요한 단계**다. 여기서 토큰화 품질이 검색 품질을 좌우한다.

- 현재 best:
    - 한국어: **Kiwi**(최근 Okt/Mecab보다 성능·속도 우위로 평가받음), 그다음 Mecab-ko, Okt 순.
    - 영어: **Snowball stemmer**(스노볼 어간 추출), Lucene analyzer.
    - 다국어 혼합: language detection(언어 감지) 후 언어별 analyzer 라우팅.
- 처리 내용: 형태소 분석(morphological analysis), 불용어 제거(stopword removal), 소문자화, stemming/lemmatization(어간/표제어 추출).

**4. Chunking (청킹, 문서 분할)** Keyword RAG는 벡터와 달리 긴 chunk도 BM25 정규화 덕에 잘 작동하지만, 결국 LLM context에 넣을 단위이므로 적절히 쪼갠다.

- 현재 best: **Recursive Character Splitting**(재귀적 문자 분할, LangChain 기본)을 heading-aware(헤딩 인식)하게 변형. Markdown/HTML은 **Document-aware chunking**(구조 기반 분할)이 가장 안정적이다. 크기는 보통 512~1024 토큰 + 10~20% overlap(중첩).

**5. Metadata Extraction (메타데이터 추출)** source, title, section path, updated_at, author, tags를 각 chunk에 붙인다.

- 현재 best: 파싱 단계에서 heading hierarchy(헤딩 계층)를 그대로 `section_path: ["2. Method", "2.1 Retriever"]` 형태로 저장한다. filtering(필터링)과 citation(인용)에 필수.

**6. Inverted Index 구축 (역색인)** 각 token이 어느 문서의 어느 위치에 등장하는지를 매핑한다.

- 현재 best:
    - 대규모: **Elasticsearch / OpenSearch**(BM25 기본 내장).
    - 한국어 특화: **PGroonga**(PostgreSQL 확장, 한국어·일본어·중국어 n-gram+형태소 지원).
    - 가볍게: **BM25S**(Python 네이티브, 최근 속도 최적화로 인기).
    - Rust 기반: **Tantivy**.
- BM25 파라미터: k1=1.2~2.0, b=0.75가 일반적 기본값이며, 도메인에 맞춰 튜닝한다.

### Retrieval Phase (검색 단계, 온라인)

**7. Query Input (쿼리 입력)** 사용자 질의를 받는다.

**8. Query Preprocessing (쿼리 전처리)** **인덱싱 때 썼던 것과 완전히 동일한 analyzer**를 적용한다. 이게 안 맞으면 검색이 통째로 망가진다.

- 현재 best: Elasticsearch의 `search_analyzer`와 `analyzer`를 동일하게 설정하거나, PGroonga의 tokenizer를 인덱싱·쿼리 모두 같은 값으로 맞춘다.

**9. Query Expansion (쿼리 확장, 선택적)** 동의어·약어·오타를 보강한다.

- 현재 best: **LLM 기반 query rewriting**(LLM이 동의어·관련어 3~5개를 생성해 OR 결합). 도메인별 synonym dictionary(동의어 사전)를 함께 쓰면 정밀도가 더 올라간다.

**10. BM25 Retrieval (BM25 검색)** 역색인을 타고 top-k 문서를 뽑는다. 보통 k=50~100으로 넉넉히 뽑고 뒤 단계에서 줄인다.

**11. Re-ranking (재순위화)** BM25는 recall(재현율)은 좋지만 precision(정밀도)이 떨어지므로 반드시 재정렬한다.

- 현재 best: **Cohere Rerank 3.5** 또는 **bge-reranker-v2-m3**(다국어 지원, 오픈소스 중 최상위). 한국어 포함 multilingual이라면 bge-reranker-v2-m3가 사실상 표준.

**12. Filtering & Deduplication (필터링·중복 제거)** metadata filter(날짜 범위, 권한, 카테고리), score threshold(점수 임계값), near-duplicate 제거를 적용한다.

### Generation Phase (생성 단계)

**13. Context Assembly (컨텍스트 조립)** 선택된 chunk를 prompt에 넣는다.

- 현재 best: **Lost in the Middle(중간 망각, 긴 컨텍스트에서 가운데 정보가 잘 안 쓰이는 현상)** 대응을 위해 중요 chunk를 맨 앞/뒤에 배치, 각 chunk 앞에 source metadata를 명시, citation 토큰(`[1]`, `[2]`)을 부여한다.

**14. LLM Generation (답변 생성)**

- 현재 best: **Claude Opus/Sonnet 4.x**, **GPT-4.1/GPT-5**, **Gemini 2.x Pro** 중 비용·지연시간·품질 trade-off로 선택.

**15. Post-processing (후처리)** citation 매핑, hallucination check(환각 검증), guardrail(가드레일) 적용.

---

## Part 2. Vector RAG (Dense Retrieval) 전체 순서

### Indexing Phase

**1. Document Collection (문서 수집)** Keyword RAG와 동일.

**2. Document Parsing (문서 파싱)** Keyword RAG와 동일. **Docling** 권장.

**3. Preprocessing (전처리)** Vector RAG는 임베딩 모델이 noise(잡음)에 강하므로 Keyword RAG만큼 공격적으로 정제하지 않는다. 오히려 문맥을 보존해야 임베딩이 제대로 된다.

- 현재 best: 헤더/푸터·네비게이션 같은 구조 노이즈만 제거하고, 본문 표현은 그대로 둔다. 형태소 분석은 **하지 않는다**(임베딩 모델이 내부에서 처리).

**4. Chunking (청킹)** Vector RAG에서는 chunking 전략이 검색 품질에 Keyword RAG보다 **훨씬 더 크게** 영향을 미친다.

- 현재 best 3가지 중 선택:
    - **Contextual Retrieval**(Anthropic, 2024년 말 제안): 각 chunk 앞에 "이 chunk가 전체 문서에서 어떤 맥락인지"를 LLM이 50~100 토큰으로 요약해 붙인 뒤 임베딩한다. 실험적으로 retrieval failure를 35~49% 감소시켰다고 보고됨. **현재 사실상 SOTA(state-of-the-art, 최고 성능)**.
    - **Late Chunking**(Jina AI): 문서 전체를 먼저 long-context 임베딩 모델로 인코딩한 뒤, 토큰 임베딩을 chunk 단위로 pooling(풀링, 집계)한다. 문맥 정보를 살리면서도 chunk 단위 검색이 가능.
    - **Semantic Chunking**(의미 기반 분할): 문장 임베딩 유사도가 급락하는 지점에서 경계를 나눈다. LlamaIndex `SemanticSplitterNodeParser`.
- 크기: 256~512 토큰이 임베딩 품질 피크. 2048 넘으면 임베딩이 "평균화(smoothing)"돼서 성능 하락.

**5. Metadata Extraction (메타데이터 추출)** Keyword RAG와 동일. Vector DB에서는 payload/attribute 형태로 저장된다.

**6. Embedding (임베딩 생성)** chunk를 dense vector(밀집 벡터)로 변환하는 단계. 모델 선택이 곧 검색 품질을 결정한다.

- 현재 best (2026년 4월 기준):
    - **다국어/한국어**: **BGE-M3**(BAAI, multilingual 100+ 언어, dense+sparse+ColBERT multi-vector 동시 지원), **Voyage-3-large**, **Cohere embed-multilingual-v3**.
    - **한국어 특화 오픈소스**: **KURE-v1**(한국어 검색 튜닝 모델), **ko-sroberta-multitask**.
    - **영어 전용 상용 최고치**: **Voyage-3-large**, **OpenAI text-embedding-3-large**.
    - MTEB 리더보드에서 모델을 비교하되, **자기 도메인 데이터로 직접 평가**하는 게 가장 정확하다.
- 선택 기준: (1) 지원 언어, (2) 최대 context length, (3) 벡터 차원(스토리지 비용), (4) 라이선스, (5) 비용.

```python
# BGE-M3 예시
from FlagEmbedding import BGEM3FlagModel  # 모델 클래스를 불러와라
model = BGEM3FlagModel('BAAI/bge-m3', use_fp16=True)  # 모델을 로드해라: fp16으로 메모리 절약
embeddings = model.encode(chunks, batch_size=32)['dense_vecs']  # chunk를 인코딩해라: dense 벡터 반환
```

**7. Vector Indexing (벡터 인덱스 구축)** 수백만 벡터를 빠르게 탐색하기 위한 ANN(Approximate Nearest Neighbor, 근사 최근접 이웃) 인덱스를 만든다.

- 현재 best 알고리즘:
    - **HNSW**(Hierarchical Navigable Small World, 계층적 근접 그래프): 거의 모든 실전 서비스의 기본 선택. recall·속도 균형이 가장 좋다.
    - **IVF-PQ**(Inverted File + Product Quantization): 메모리가 극단적으로 빡빡한 대규모(10억+ 벡터)에서 선택.
    - **DiskANN**(Microsoft): SSD 기반 수십억 벡터 스케일.
- 현재 best 저장소:
    - **Qdrant**: Rust 기반, payload filtering 성능과 운영 편의성으로 최근 가장 많이 채택됨.
    - **Weaviate**: hybrid search와 모듈 생태계가 풍부.
    - **Milvus / Zilliz**: 초대규모(수십억 벡터)에서 강함.
    - **pgvector + pgvectorscale**: 기존 PostgreSQL 스택에 붙이기 좋음. 2025년 pgvectorscale로 HNSW 성능이 크게 개선됨.
    - **Neo4j Vector Index**: 그래프와 함께 쓸 때(Graph RAG) 선택.

### Retrieval Phase

**8. Query Input (쿼리 입력)**

**9. Query Transformation (쿼리 변환)** Vector RAG에서 이 단계가 품질에 미치는 영향이 매우 크다.

- 현재 best:
    - **HyDE**(Hypothetical Document Embeddings, 가상 문서 임베딩): LLM에게 "이 질문에 대한 가상의 답변"을 짧게 생성시키고, 그 답변을 임베딩해서 검색한다. 질문-문서 간 분포 불일치(asymmetric retrieval) 문제를 완화.
    - **Multi-Query**: LLM이 원 질문을 3~5개 변형 질문으로 확장, 각각 검색 후 결과를 RRF로 합침.
    - **Step-back Prompting**: 구체 질문을 일반화된 질문으로 한 번 추상화해서 검색.
    - **Query Decomposition**: 복합 질문(multi-hop question)을 sub-query로 쪼갬. Agentic RAG의 기본.

**10. Query Embedding (쿼리 임베딩)** **인덱싱 때와 반드시 동일한 임베딩 모델**을 쓴다. 차원이 다르면 검색 자체가 불가능하고, 같은 차원이어도 모델이 다르면 의미 공간이 달라 성능이 붕괴된다.

**11. Vector Retrieval (벡터 검색)** ANN 인덱스에서 top-k를 뽑는다. 보통 k=20~100.

- 현재 best: HNSW의 `ef_search` 파라미터를 runtime에 조절해서 latency(지연시간) vs recall(재현율) trade-off를 제어한다.

**12. Re-ranking (재순위화)** Bi-encoder(bi-인코더, 쿼리·문서를 각각 독립 인코딩)의 한계를 Cross-encoder(교차 인코더, 쿼리·문서를 함께 입력해 정밀 스코어링)로 보완한다.

- 현재 best: **Cohere Rerank 3.5**(상용 최고치), **bge-reranker-v2-m3**(오픈소스 최고치, 다국어), **Jina Reranker v2**.
- 비용 측면: top-100을 re-rank해서 top-5만 쓰는 게 일반적인 recipe.

**13. Filtering & Deduplication (필터링·중복 제거)** metadata filter, MMR(Maximal Marginal Relevance, 최대 한계 관련도)로 다양성 확보.

### Generation Phase

**14. Context Assembly (컨텍스트 조립)** Keyword RAG와 동일. Lost in the Middle 대응 필수.

**15. LLM Generation (답변 생성)** Keyword RAG와 동일.

**16. Post-processing (후처리)** citation 매핑, faithfulness(충실도) 검증, guardrail.

---

## Part 3. 두 파이프라인을 결합하는 Hybrid RAG

실무에서는 어느 한쪽만 쓰지 않는다. 두 결과를 합친다.

- 현재 best 결합 방법: **RRF(Reciprocal Rank Fusion, 순위 역수 결합)**.
    - 공식: `RRF_score(d) = Σ 1 / (k + rank_i(d))`, 보통 `k=60`.
    - 장점: 점수 스케일이 다른 BM25와 cosine similarity를 스케일 정규화 없이 자연스럽게 합친다.
- 가중치 기반 결합도 쓰이지만(`score = α * normalized_bm25 + (1-α) * cosine`), 데이터셋마다 α 튜닝이 필요해 RRF가 기본값으로 쓰인다.
- **BGE-M3의 장점**: dense + sparse(lexical) + ColBERT-style multi-vector를 **한 모델이 동시에 생성**하므로, 하이브리드 검색을 단일 모델 호출로 구성할 수 있다. 최근 1년간 한국어 하이브리드 RAG의 표준 선택지로 자리 잡았다.

---

## Part 4. 평가·관측 (Evaluation & Observability)

RAG는 "만든다"보다 "측정하고 튜닝한다"가 훨씬 어려운 영역이다. 처음부터 관측 가능하게 짜야 한다.

- 현재 best:
    - **RAGAS**: faithfulness(충실도), answer relevance(답변 관련성), context precision/recall(컨텍스트 정밀도/재현율) 지표.
    - **DeepEval**: pytest 스타일로 CI에 통합 가능.
    - **TruLens**: 실시간 프로덕션 모니터링.
    - **Langfuse**: distributed tracing(분산 추적), 각 단계의 latency·cost·입출력을 추적. Self-hosted 가능. 최근 RAG 파이프라인 관측의 de facto standard(사실상의 표준).

---

## 요약 비교표

|단계|Keyword RAG|Vector RAG|
|---|---|---|
|전처리|형태소 분석 필수 (Kiwi/Mecab)|최소한만 (임베딩 모델이 처리)|
|Chunking|Recursive + heading-aware|Contextual Retrieval / Late Chunking|
|인덱스 빌드|Inverted Index (BM25)|HNSW (Vector)|
|저장소|Elasticsearch, PGroonga, BM25S|Qdrant, pgvector, Weaviate|
|모델 의존도|analyzer 품질이 핵심|임베딩 모델 품질이 핵심|
|Query 변환|Synonym expansion|HyDE / Multi-Query|
|Re-ranker|bge-reranker-v2-m3 / Cohere Rerank|동일|
|강점|고유명사·코드·숫자|의미적 유사성·paraphrase|
|비용|낮음 (CPU만)|높음 (임베딩 API/GPU)|

---

## 다음에 볼 만한 인접 주제

- **Contextual Retrieval의 실제 구현**(Anthropic 블로그 + prompt caching으로 비용 절감).
- **ColBERT / late-interaction retrieval**: bi-encoder와 cross-encoder의 중간 지점. 최근 다시 주목받음.
- **Graph RAG**: chunk가 아닌 entity-relationship graph 위에서의 검색. Neo4j + Text2Cypher 조합이 유력.
- **Agentic RAG**: Self-RAG, CRAG(Corrective RAG), Adaptive RAG. 검색 여부·재검색·쿼리 재작성을 LLM이 스스로 판단.
- **Self-hosted 임베딩 추론 최적화**: Text Embeddings Inference(TEI, HuggingFace) + ONNX/TensorRT.