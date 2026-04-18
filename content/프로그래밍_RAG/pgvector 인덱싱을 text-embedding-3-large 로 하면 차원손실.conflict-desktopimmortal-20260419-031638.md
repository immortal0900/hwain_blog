---
생성날짜:
  - 2026-04-04 03:34
마지막수정날짜:
  - 2026-04-04-토요일 03:34
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## pgvector 인덱싱 방법 전체 비교

### 1. 사용 가능한 인덱싱 방법

|방법|차원 제한 (vector/halfvec)|정밀도 손실|빌드 속도|검색 속도|데이터 없이 생성|
|---|---|---|---|---|---|
|**Flat Scan (인덱스 없음)**|제한 없음|없음 (정확한 결과)|-|느림 (전수조사)|-|
|**HNSW + halfvec**|4,000|float32→float16 (~1-2% recall 손실)|느림|빠름|가능|
|**HNSW + vector**|**2,000 (초과 불가)**|없음|느림|빠름|가능|
|**IVFFlat + halfvec**|4,000|float32→float16|빠름|HNSW보다 느림|**불가** (데이터 필요)|
|**IVFFlat + vector**|**2,000 (초과 불가)**|없음|빠름|HNSW보다 느림|**불가**|

### 2. 3072차원에서 손실 없이 인덱싱하는 방법은?

**결론: pgvector에서는 불가능합니다.**

근본적 제약이 PostgreSQL 페이지 크기(8KB)에서 옵니다:

```
vector(3072) = 3072 x 4byte(float32) = 12,288byte > 8KB 페이지 크기
```

한 벡터가 한 페이지에 안 들어가므로, pgvector가 인덱스 구조를 만들 수 없습니다. `halfvec`은 `3072 x 2byte = 6,144byte < 8KB`로 페이지 안에 들어가서 가능한 것입니다.

### 3. 손실 없이 가능한 대안: 차원 축소 (Dimension Shortening)

OpenAI `text-embedding-3-large`는 API 호출 시 `dimensions` 파라미터로 **출력 차원을 줄일 수 있습니다**:

```python
# 현재: 3072차원 (HNSW vector 인덱스 불가)
OpenAIEmbeddings(model="text-embedding-3-large")

# 대안: 1536차원으로 축소 (HNSW vector 인덱스 가능, 손실 없는 float32)
OpenAIEmbeddings(model="text-embedding-3-large", dimensions=1536)
```

OpenAI 공식 벤치마크에 따르면:

- **3072차원**: MTEB 64.6%
- **1536차원**: MTEB ~62-63% (약 2-3% 하락)
- **256차원**: 여전히 `text-embedding-ada-002`(1536차원)보다 우수

### 4. 최종 비교: 3가지 현실적 선택지

|선택지|검색 정밀도|인덱스 속도|구현 난이도|비고|
|---|---|---|---|---|
|**A. HNSW + halfvec(3072)** (현재 적용)|높음 (float16 근사로 ~1-2% recall 손실)|빠름|낮음 (완료됨)|기존 데이터 재인덱싱 불필요|
|**B. HNSW + vector(1536)** (차원 축소)|높음 (float32 정확, 차원 축소로 ~2-3% 정보 손실)|빠름|**높음** (전체 재임베딩 필요)|임베딩 모델 설정 변경 + 모든 컬렉션 재적재|
|**C. Flat Scan (인덱스 없음)** (기존 방식)|최고 (정확한 전수조사)|-|없음|데이터가 수천 건이면 속도 차이 미미|

### 결론

현재 데이터 규모(350건 + 기존 컬렉션 합쳐도 수천 건)에서는 **A(현재 적용된 HNSW + halfvec)가 가장 합리적**입니다. halfvec의 ~1-2% recall 손실과 차원 축소의 ~2-3% 정보 손실은 비슷한 수준이고, A는 기존 데이터를 건드리지 않아도 됩니다.

데이터가 수만 건 이상으로 늘어나서 Flat Scan이 느려지는 시점이 오면, 그때 B(차원 축소 + 재임베딩)를 검토하는 것이 순서입니다.

Sources:

- [pgvector README - 인덱스 차원 제한표](https://github.com/pgvector/pgvector/blob/master/README.md)
- [pgvector Issue #461 - 2000 차원 제한 논의](https://github.com/pgvector/pgvector/issues/461)
- [Supabase HNSW Indexes Docs](https://supabase.com/docs/guides/ai/vector-indexes/hnsw-indexes)
- [OpenAI Embeddings - dimensions 파라미터](https://platform.openai.com/docs/guides/embeddings)