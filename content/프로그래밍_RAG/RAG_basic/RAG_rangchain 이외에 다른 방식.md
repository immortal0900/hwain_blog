---
생성날짜:
  - 2025-10-14 23:59
마지막수정날짜:
  - 2025-10-14-화요일 23:59
tags:
  - RAG/rangchain이외
  - RAG/프레임워크비교
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---

## 한 줄 정리

**LangChain은 RAG의 유일한 선택지가 아님.** LangChain이 담당하는 두 축(접착제 + 프레임워크) 중 하나 또는 전부를 대체하는 대안이 여럿 존재함. 실무에서 고려할 만한 4가지가 **LlamaIndex / 직접구현(DIY) / Haystack / LangChain 본인**임.

---

## LangChain의 두 가지 역할부터 분해

```
LangChain이 하는 일
  ├─ 접착제(Glue): LLM + 벡터 DB + API + 툴을 엮어주는 연결고리
  └─ 프레임워크(Framework): RAG·Agent·Memory 등 패턴을 미리 구현한 도구 모음
```

### 포함 관계

```
LLM 애플리케이션 (넓은 범주)
  └─ RAG (한 축)
       ├─ LangChain          (RAG 포함 모든 LLM 앱 커버)
       ├─ LlamaIndex         (RAG에 집중, 데이터 인덱싱 깊이 팜)
       ├─ Haystack           (검색 파이프라인 기반, LLM은 얹어 씀)
       └─ 직접구현(DIY)      (프레임워크 없이 라이브러리 조합)
```

**LangChain은 외연이 가장 넓고, LlamaIndex/Haystack은 검색 쪽으로 더 깊게 팜, DIY는 맨손.**

---

## 1. LlamaIndex: 데이터 중심의 RAG 특화 프레임워크

**LangChain과 가장 직접적인 경쟁자임.** 초기부터 "데이터 인덱싱과 검색(Retrieval)"에 집중하며 시작했기 때문에 RAG 구현 품질 자체는 매우 강함.

```
[LlamaIndex 파이프라인]
데이터 로딩(Data Loaders / LlamaHub)
    → 인덱싱(Vector/KG/Summary Index)
    → 쿼리 엔진(Query Engine)
    → 응답 합성(Response Synthesizer)
```

2024 이후 최신 구조에선 **Query Engine / Chat Engine / Agent / Workflows** 가 상위 추상화로 올라가고, 하위에 Retriever와 Response Synthesizer가 붙는 식임.

### 장점

- **강력한 데이터 처리·인덱싱**: PDF, 노션, Slack, API, SQL, 이미지 같은 비정형 데이터를 인덱스로 만들어주는 LlamaHub 커넥터가 방대함
- **고급 검색 전략**: Sub-Question, Recursive Retrieval, Knowledge Graph Index, Router Query Engine 같은 정교한 전략이 기본 내장
- **명확한 파이프라인**: "로딩 → 인덱싱 → 쿼리" 흐름이 직관적이라 RAG 핵심 로직에 집중하기 좋음

### 단점

- **에이전트 워크플로우 약함**: 여러 도구를 연쇄로 쓰는 범용 에이전트는 LangChain보다 유연성이 떨어짐 (보완책으로 LlamaIndex Workflows 추가됨)
- **좁은 범위**: 핵심 초점이 검색이라 LLM 앱 전반을 묶기엔 기능이 제한적
- **학습 곡선**: 내부 자료구조(Node, Document, Index)에 대한 이해가 선행되어야 함

> **한마디 요약**: "검색 정확도 극한"을 원하면 LlamaIndex.

---

## 2. 직접 구현(DIY): 근본 라이브러리 조합

**프레임워크 없이 기능별 라이브러리를 직접 엮는 방식임.** 진정한 엔지니어의 길.

```python
# [DIY 파이프라인 예시]
from pypdf import PdfReader
from openai import OpenAI
import faiss, numpy as np

client = OpenAI()
reader = PdfReader("docs.pdf")
text = "\n".join(p.extract_text() for p in reader.pages)

chunks = [text[i:i+500] for i in range(0, len(text), 500)]
embeds = [
    client.embeddings.create(model="text-embedding-3-small", input=c).data[0].embedding
    for c in chunks
]

index = faiss.IndexFlatL2(len(embeds[0]))
index.add(np.array(embeds, dtype="float32"))
# 이하 검색/프롬프트 조립/호출도 전부 직접 작성
```

주로 쓰는 라이브러리는 `openai`, `faiss-cpu` 또는 `pgvector`, `huggingface-hub`, `tiktoken` 같은 원시 도구들임.

### 장점

- **완벽한 통제권과 투명성**: 내부 동작이 전부 내 코드 안에 있음. 디버깅 쉽고 성능 최적화 극한까지 가능
- **가벼움(Lightweight)**: 쓸 기능만 설치, 배포 용량 작음, 의존성 지옥 없음
- **가장 깊은 학습 효과**: 프롬프트 구성, 컨텍스트 주입, API 호출, 파싱을 뼛속까지 이해하게 됨

### 단점

- **높은 초기 개발 비용**: 프롬프트 템플릿, LLM API 연동, 벡터 DB 검색, 결과 파싱을 전부 직접 짜야 함. 보일러플레이트가 쌓임
- **느린 실험 속도**: LLM/벡터 DB/프롬프트 전략 교체마다 코드 수술 필요. 프로토타입 속도가 느림
- **오케스트레이션 부재**: Retry, Fallback, Tracing, Observability 기능은 직접 얹어야 함

> **한마디 요약**: "투명성과 최적화"가 필요하면 DIY, 다만 속도 포기 각오 필요.

---

## 3. Haystack: 검색 시스템 뿌리의 프레임워크

**`deepset.ai`에서 만든 프레임워크임.** 전통적인 NLP 검색(Search)/질의응답(QA) 시스템에서 출발해 LLM 기능을 얹은 케이스라 검색 파이프라인 설계가 단단함.

2024년 이후 **Haystack 2.x** 가 릴리즈되며 구조가 바뀜 (v1의 DocumentStore + Node 체제 → v2의 Pipeline + Component DAG 체제).

```
[Haystack 2.x 파이프라인 예시]
Fetcher → Converter → Splitter → Embedder → Retriever
       → PromptBuilder → Generator → Answer
```

각 Component는 입력/출력 포트가 명시된 노드이고, Pipeline 이 DAG(Directed Acyclic Graph, 방향 비순환 그래프) 로 이들을 엮음.

### 장점

- **강력한 파이프라인 개념**: Retriever, Reader, Generator 같은 검색 구성 요소를 명시적으로 조립 가능. 대규모 문서 검색 시스템에 강점
- **뛰어난 확장성**: Elasticsearch, OpenSearch, Weaviate, Qdrant, pgvector 등 다양한 DocumentStore 공식 지원
- **성숙도와 안정성**: LLM 유행 이전부터 존재한 프레임워크라 자체 안정성과 문서화가 뛰어남
- **Production-ready**: REST API, Docker 배포, 모니터링 관련 툴링이 탄탄함

### 단점

- **상대적으로 높은 복잡도**: LangChain 대비 선행 지식이 필요하고 간단한 RAG 예제도 장황해질 수 있음
- **LLM 중심 생태계 인지도 부족**: LangChain/LlamaIndex 대비 최신 LLM 기능 지원·예제 수가 상대적으로 적을 수 있음

> **한마디 요약**: "기존 검색 인프라 + LLM 결합"이면 Haystack이 자연스러움.

---

## 4. 최종 요약 테이블

| 구분 | LangChain | LlamaIndex | 직접 구현 (DIY) | Haystack |
| --- | --- | --- | --- | --- |
| **핵심 철학** | LLM 앱 개발 만능 프레임워크 | 데이터 중심 RAG 특화 | 필요한 것만 조립 | 검색 파이프라인 기반 |
| **주요 장점** | 빠른 프로토타이핑, 방대한 기능, 거대 생태계 | 고급 검색, 복잡한 데이터 처리 | 완벽한 통제, 성능 최적화 | 안정성, 검색 엔진 연동, 명확한 구조 |
| **주요 단점** | 추상화 때문에 디버깅 어려움, 잦은 업데이트 | 범용성 부족, 높은 학습 곡선 | 높은 개발 비용, 느린 실험 | 복잡도, 최신 LLM 지원 속도 |
| **추천 시점** | **빠른 아이디어 검증** | **검색 정확도 극한** | **프로덕션 최적화** | **기존 검색 시스템 확장** |
| **대표 상위 추상화** | LCEL, LangGraph | Query Engine, Workflows | 없음 | Pipeline, Component |

---

## 그래서 LangChain을 왜 이렇게 많이 쓰는가

**LangChain 인기의 본질은 "RAG가 가능해서"가 아니라, "LLM 중심 애플리케이션 개발 생태계 그 자체"임.**

### 1) RAG를 넘어 전체 워크플로우를 아우름

LangChain의 철학은 LLM을 단독으로 쓰는 게 아니라, 외부 데이터·API·계산기 같은 **"도구(Tool)"와 연결**하는 것. RAG는 이 철학의 한 구현체일 뿐임.

```
LlamaIndex: "검색 성능을 어떻게 올릴까" 에 집중
LangChain : "검색된 정보로 다음에 뭘 할지" 까지 커버
            (API 호출, 다른 LLM에 위임, 코드 실행, DB 쓰기 등)
```

### 2) 압도적인 통합 생태계와 모듈성

LangChain은 LLM 앱 제작에 필요한 거의 모든 것을 **부품**으로 제공함.

- **통합 폭**: 수십 개 벡터 저장소, 수백 개 문서 로더, 각종 LLM 벤더, 툴 직결기(Slack, Gmail, SQL 등) 내장
- **모듈식 설계**: 각 컴포넌트(로더, 스플리터, 임베더)가 독립이라 FAISS → Pinecone 전환이 몇 줄 수정으로 가능
- **LCEL(LangChain Expression Language)**: 파이프 연산자로 체인 조립 (`retriever | prompt | llm | parser`)

### 3) 빠른 프로토타이핑과 거대한 커뮤니티

높은 추상화 덕분에 복잡한 아이디어도 몇 줄로 돌려볼 수 있음. 사용자 커뮤니티 규모 덕에 문서·튜토리얼·스택오버플로 답변이 풍부함.

### 4) 복잡한 AI 에이전트로의 확장성

단순 질의응답을 넘어 **LLM이 스스로 판단하고 여러 도구를 연쇄 사용해 문제를 푸는 에이전트**를 만들 때 진가가 드러남.

```
예시: LLM이 RAG로 정보 찾고
      → 인터넷 검색으로 최신 뉴스 보강
      → 코드 실행로 수치 계산
      → 최종 답변 조립
```

이런 체이닝이 LCEL 또는 **LangGraph (상태 기반 DAG 에이전트 프레임워크)** 로 자연스럽게 조립됨.

---

## 직접 확인 가이드

빠르게 감을 잡으려면 동일 RAG를 네 방식으로 돌려보면 좋음:

1. **LangChain**: `langchain` + `langchain-community` + FAISS → 10~20줄
2. **LlamaIndex**: `VectorStoreIndex.from_documents(...).as_query_engine()` → 5~10줄
3. **Haystack**: `Pipeline()` 에 Component 네다섯 개 엮기 → 20~30줄
4. **DIY**: `openai` + `faiss-cpu` 직접 → 50줄 내외

같은 질문/같은 데이터로 돌려 보면 **추상화 층이 얼마나 다른지** 감각으로 체감됨.

---

## 결론 (선택 가이드)

- **LlamaIndex**: RAG **검색 성능** 자체를 극한으로 끌어올려야 할 때 최선
- **LangChain**: RAG 포함 **다양한 도구·데이터 오케스트레이션** 필요할 때 최강
- **DIY**: 프로덕션 최적화 또는 학습 목적
- **Haystack**: 기존 검색 인프라(Elasticsearch 등) 위에 LLM을 얹어야 할 때

많은 개발자가 RAG를 단독 기능이 아닌 **더 큰 애플리케이션의 일부**로 바라보기 때문에, LangChain의 "통합·오케스트레이션" 능력이 대중적 선택지를 만든 것임.

> **한마디 요약**: 검색만 극한이면 **LlamaIndex**, 전체 오케스트레이션이면 **LangChain**.

관련: [[retriever_추천]], [[BM25와 PGroonga의 관계]], [[벡터유사도에 관하여]], [[BGE-M3]]
