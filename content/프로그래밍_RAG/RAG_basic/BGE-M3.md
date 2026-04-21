---
생성날짜:
  - 2026-04-16 00:38
마지막수정날짜:
  - 2026-04-16-목요일 00:37
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---



## Self-Knowledge Distillation?

**먼저 Knowledge Distillation(지식 증류)부터**

원래 Knowledge Distillation은 **두 모델** 사이에서 일어나는 과정임:

```
Teacher 모델 (크고 똑똑함)
    │
    │ "이 문서와 이 쿼리의 관련도는 0.85야"  ← teacher의 판단
    │
    ▼
Student 모델 (작고 가벼움)
    → "아, 0.85에 가깝게 학습해야겠구나"
```

큰 모델의 지식을 작은 모델에 "증류(distill)"해서 옮기는 것. 소주 증류할 때 원액에서 핵심만 뽑아내는 것과 같은 원리임.

**Self-Knowledge Distillation: 자기 자신이 teacher**

BGE-M3는 teacher 모델이 따로 없음. **자기 자신의 여러 출력을 합쳐서 teacher를 만듦.**

BGE-M3는 하나의 모델인데 출력이 세 개임:

```
입력: "asyncio에서 에러 발생" ↔ "gather에 sync 함수 넘기면 TypeError"

BGE-M3 모델 (하나의 모델)
    │
    ├─ Dense 출력:  "이 두 문장의 관련도는 0.72야"
    │               (의미적으로 비슷한지 판단)
    │
    ├─ Sparse 출력: "이 두 문장의 관련도는 0.65야"
    │               (키워드가 얼마나 겹치는지 판단)
    │
    └─ Multi-vec 출력: "이 두 문장의 관련도는 0.80이야"
                       (토큰별 세밀한 매칭으로 판단)
```

세 가지가 각각 다른 관점에서 관련도를 판단 중. 여기서 핵심 아이디어가 도출됨:

**"세 명의 심사위원이 각자 점수를 줬으면, 그 평균이 한 명보다 더 정확하지 않을까?"**

```
Teacher Signal (종합 점수) = Dense(0.72) + Sparse(0.65) + Multi-vec(0.80)
                           ─────────────────────────────────────────────
                                              3
                         = 0.723 (대략)

이 종합 점수가 "정답"이 됨
```

그 다음, 이 종합 점수를 목표로 삼아 **각 개별 출력을 다시 학습**시킴:

```
[학습 과정]

종합 점수: 0.723  ← "이게 정답"
    │
    ├─→ Dense(0.72):    "난 0.72인데 0.723에 맞춰야지"    → 약간 올림
    ├─→ Sparse(0.65):   "난 0.65인데 0.723에 맞춰야지"    → 좀 더 올림
    └─→ Multi-vec(0.80): "난 0.80인데 0.723에 맞춰야지"   → 약간 내림
```

이 과정을 반복하면:

```
[학습 전]
Dense: 0.72  |  Sparse: 0.65  |  Multi-vec: 0.80
      (각자 따로 놀고 있음)

[학습 후: 서로를 강화]  
Dense: 0.78  |  Sparse: 0.76  |  Multi-vec: 0.82
      (서로의 판단을 참고해서 각자 더 정확해짐)
```

**특히 Sparse가 가장 큰 수혜자.** 원래 키워드 매칭만으로는 의미를 못 잡지만, Dense와 Multi-vec이 "이 두 문장은 의미적으로 관련 있다"고 알려주니까, Sparse도 그걸 반영해서 키워드 가중치를 더 잘 조절하게 됨. 논문에서도 self-knowledge distillation이 특히 sparse retrieval과 multi-vector retrieval의 성능을 크게 향상시켰다고 명시 [Hugging Face](https://huggingface.co/BAAI/bge-m3).

**비유로 정리**

```
[일반 BM25]
국어 선생님 혼자 채점 → 국어 관점에서만 판단

[일반 Dense]  
수학 선생님 혼자 채점 → 수학 관점에서만 판단

[BGE-M3 Self-Knowledge Distillation]
국어+수학+영어 선생님이 각자 채점한 후
→ 세 점수를 평균 내서 "종합 정답"을 만들고
→ 각 선생님이 그 종합 정답에 맞춰서 자기 채점 기준을 보정
→ 결과: 국어 선생님도 수학적 논리를, 수학 선생님도 언어적 맥락을 반영하게 됨
```

그래서 BGE-M3의 sparse가 단순 BM25보다 강한 것.


## sparse 검색이 키워드 검색을 대체 가능한 이유 

**같은 토큰이 양쪽에 다 있을 때만 값이 나오고, 한쪽이라도 0이면 기여하지 않음.** 결국 "같은 키워드가 있으면 점수가 올라간다"는 것이고, 키워드 매칭과 본질적으로 동일함.

**그러면 BM25와 뭐가 다른지?**

```
문서: "파이썬 비동기 처리에서 문제 발생"
쿼리: "asyncio 에러"

[BM25]
"asyncio" 있나? → ❌ 없음 → 점수 0
"에러" 있나?    → ❌ 없음 ("문제"는 있지만 "에러"는 없음) → 점수 0
최종: 0점 (매칭 실패) ❌

[BGE-M3 sparse]
모델이 문맥을 이해하고 가중치를 줌:
"비동기"  → Transformer가 "이건 async 관련이다"라고 학습했기 때문에
              async 관련 토큰에도 약간의 가중치를 부여할 수 있음
"문제"    → "에러"와 비슷한 맥락이라는 걸 학습에서 배움
최종: 낮지만 0은 아닌 점수 (약한 매칭) ⚠️
```

이게 앞서 말한 **Self-Knowledge Distillation의 효과**임. Dense(의미 검색)가 "비동기 = asyncio"라는 걸 알고 있으니까, 학습 과정에서 sparse 쪽도 그 정보를 간접적으로 흡수한 것. 그래서 BGE-M3 sparse가 순수 BM25보다 성능이 높음.

다만 어차피 하이브리드 검색을 사용할 것이므로, 키워드 검색은 정확한 키워드를 가져오는 역할만 수행하면 됨.

## 핵심 연구: IBM Blended RAG

IBM 연구에서 BM25, dense vector, BM25 + dense, dense + sparse vector, BM25 + dense + sparse vector 조합을 비교한 결과, **세 가지를 다 쓰는 three-way retrieval이 최적**이라는 결론이 나옴 [Infiniflow](https://infiniflow.org/blog/best-hybrid-search-solution).

```
[IBM 연구 결과, nDCG 기준]

Dense only           ████████░░  (기본)
Dense + Sparse       █████████░  (약간 개선)
Dense + BM25         ██████████  (큰 개선)
Dense + Sparse + BM25 ██████████▌ (최고 성능)
```

---

## 왜 Sparse vector가 BM25를 대체 못 하는지

sparse vector는 3만 차원으로 모든 키워드를 커버할 수 없고, 다국어 상황에서는 더 심함. 또한 구문 검색(phrase query)에서 심각한 정보 손실이 발생함. 이런 작업은 full-text search(BM25)로 처리해야 함 [Infiniflow](https://infiniflow.org/blog/best-hybrid-search-solution).

구체적으로 보면:

**문제 1. 토크나이저가 다르다**

```
검색어: "run_in_executor"

[BM25: 단어 단위]
→ "run_in_executor" 통째로 매칭
→ 문서에 "run_in_executor"가 있으면 100% 매칭 ✅

[SPLADE/BGE-M3 sparse: subword 단위]
→ ["run", "_", "in", "_", "ex", "ecu", "tor"]으로 쪼개짐
→ "executor"가 포함된 다른 문서도 부분 매칭 ⚠️
→ "run_in_executor" 자체의 정확 매칭은 보장되지 않음
```

Qdrant 팀 자체도 이 문제를 인정함. SPLADE 같은 학습된 sparse 모델은 transformer 토크나이저를 쓰는데, 이 토크나이저가 retrieval 용도로 설계된 게 아니라는 것 [Qdrant](https://qdrant.tech/articles/bm42/).

**문제 2. 학습 데이터 편향**

SPLADE 같은 learned sparse 모델의 transformer 토크나이저는 영어 자연어용으로 만들어진 것. 제품 코드, 정책 번호 같은 특수 식별자를 처리하도록 설계된 게 아님 [Minimalistinnovation](https://www.minimalistinnovation.co/post/learned-sparse-retrieval-splade-entity-resolution).

내 지식 DB에 들어갈 `QLoRA`, `pgvector`, `astream_events`, `ContextVar` 같은 기술 용어가 정확히 이 케이스에 해당함.

**문제 3. 용어 확장이 양날의 검**

"study"라는 키워드를 BM25로 변환하면 딱 하나의 non-zero 값만 생기지만, SPLADE로 변환하면 "learn", "research", "investigate" 등 유사 단어에도 non-zero 값이 생김 [Medium](https://medium.com/@zilliz_learn/comparing-splade-sparse-vectors-with-bm25-53368877359f).

이 용어 확장(term expansion)이 일반 검색에서는 장점이지만, 지금 유스케이스에서는:

```
쿼리: "QLoRA"

[BM25]
→ "QLoRA"만 찾음 → 정확 ✅

[Sparse vector (SPLADE/BGE-M3)]
→ "QLoRA" + "LoRA" + "adapter" + "quantize" 등에도 가중치 부여
→ "LoRA" 문서도 점수가 높게 나옴 → 노이즈 ⚠️
```

"QLoRA"를 검색했는데 "LoRA" 문서가 같이 나오는 건 Dense의 역할. 키워드 검색까지 이러면 정확한 매칭을 하는 곳이 아무 데도 없어짐.
