---
생성날짜:
- 2026-02-03 00:50
마지막수정날짜:
- 2026-02-03-화요일 00:49
tags:
- 파인튜닝
- PEFT
- LoRA
- DoRA
- QLoRA
별칭:
- LoRA DoRA 비교
- Low-Rank Adaptation
type:
- 자료수집
Area/Reasource:
Project:
---

# LoRA vs DoRA

파인튜닝의 근간인 **LoRA**(Low-Rank Adaptation, 저차원 행렬 추가로 일부만 학습하는 기법)와 그 진화형 **DoRA**(Weight-Decomposed LoRA, 가중치를 크기/방향으로 분해하는 기법)를 비교 정리함.

관련 노트: [[파인튜닝 종합정리]], [[GGUF란_가중치,토크나이저, 모델구조를 파일 하나로]]

**포함 관계**: PEFT(Parameter-Efficient Fine-Tuning) ⊃ LoRA 계열 ⊃ {LoRA, QLoRA, DoRA}. DoRA는 LoRA를 감싸는 상위 호환이라 `use_dora=True` 한 줄로 전환됨.

---

## 1. LoRA: "본체는 놔두고 옆길만 학습"

### 설계 의도
거대 LLM의 수십억 파라미터 전체를 업데이트하려면 VRAM(Video RAM, GPU 메모리) 수백GB가 필요함. LoRA는 "핵심 지능(Pre-trained Weights)은 동결(Freeze)하고, 작업별 변화량만 작은 행렬로 학습하자"는 아이디어임.

### 원리

원래 가중치 행렬 $W$(보통 $d \times d$, 예: 4096 x 4096) 옆에 두 개의 작은 행렬을 덧붙임.

- **$W$ (동결, Frozen)**: 원래 모델 지식. 학습 안 함.
- **$A$ ($d \times r$)**: 입력을 낮은 차원 $r$(보통 8 or 16)로 눌러줌.
- **$B$ ($r \times d$)**: 다시 원래 차원으로 복원.
- **최종 출력**: $h = Wx + BAx = (W + BA)x$

즉 원본 출력에 "학습된 변화량 $BA$"를 더하는 구조.

### 일상 비유
두꺼운 전공 서적($W$)에 포스트잇($A, B$) 붙여서 새 메모 추가하는 느낌임. 원본 책은 건드리지 않고, 포스트잇만 교체하면 여러 작업에 같은 본체 재사용 가능.

### 학습 파라미터 수 비교 (실측)

| 모델 | 전체 파라미터 | LoRA 학습 파라미터 (r=16) | 비율 |
|---|---|---|---|
| Gemma 2 9B | 9.2B | 약 40M | 약 0.4% |
| Llama 3 8B | 8.0B | 약 21M | 약 0.26% |
| Mistral 7B | 7.2B | 약 19M | 약 0.26% |

→ 99.7%는 얼려두고 0.3%만 학습. VRAM 부담 극적 감소.

---

## 2. DoRA: "크기와 방향을 분리"

### 설계 의도 (LoRA의 한계)
LoRA는 가중치의 **크기(Magnitude, 벡터의 세기)**와 **방향(Direction, 벡터가 가리키는 각도)**을 한꺼번에 업데이트함. 실제로 Full Fine-tuning과 LoRA의 가중치 변화 패턴을 분석하면 이 두 요소의 업데이트 양상이 다름(논문: NVIDIA DoRA 2024). 그래서 때때로 정교함이 떨어지는 현상이 관찰됨.

DoRA는 이 둘을 분리해서 **전체 파인튜닝 성능에 근접**하도록 설계됨.

### 원리

가중치 $W$를 수학적으로 분해함.
$$W = m \cdot \frac{V}{\|V\|}$$
- **$m$ (Magnitude)**: 가중치의 크기 (스칼라 또는 벡터)
- **$V$** : 방향 성분 (정규화 전)
- **$V / \|V\|$** : 방향 (정규화된 단위 벡터)

**DoRA의 핵심**:
- 방향 $V$에만 LoRA 적용 (작은 $A, B$ 행렬로 학습)
- 크기 $m$은 별도 학습 가능 파라미터로 따로 세밀 조정

### 일상 비유
LoRA가 "스피커 볼륨과 음질을 한 번에 조절하는 다이얼"이라면, DoRA는 "볼륨 다이얼($m$)과 이퀄라이저($V$)를 따로 조작"하는 느낌임. 두 축 분리하니 미세 조정이 정교해짐.

---

## 3. LoRA vs DoRA 비교표

| 구분 | LoRA | DoRA |
|---|---|---|
| **핵심 개념** | 저차원 행렬 추가로 변화량 학습 | 가중치를 크기/방향으로 분해 후 학습 |
| **학습 파라미터** | $A, B$ | $A, B$ + $m$ (크기 벡터) |
| **성능** | 훌륭하나 Full FT보다 약간 낮음 | **Full FT에 근접** (벤치마크 평균 +1~3점) |
| **학습 속도** | 매우 빠름 | LoRA 대비 약 10~15% 느림 (분해 연산) |
| **VRAM** | 매우 적음 | LoRA와 거의 동일 (m은 작음) |
| **안정성** | 가끔 불안정 | **더 안정적** (방향/크기 분리 덕) |
| **설정** | `use_dora=False` (기본) | `use_dora=True` |

### 벤치마크 요약 (DoRA 논문 Table 1 기준)
- LLaMA 7B 상식 추론 8개 태스크 평균:
  - Full FT: 82.4
  - LoRA: 80.8
  - **DoRA: 82.8** (Full FT 수준)
- MMLU:
  - LoRA: 44.6
  - **DoRA: 45.9**

---

## 4. 실제 설정 (PEFT 라이브러리)

HuggingFace `peft`에서 `LoraConfig` 하나로 둘 다 제어 가능. 차이는 한 줄.

### LoRA 설정
```python
from peft import LoraConfig, get_peft_model

config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM",
)
model = get_peft_model(base_model, config)
```

### DoRA 설정
```python
config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    use_dora=True,  # ← 이 한 줄만 추가하면 DoRA
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM",
)
model = get_peft_model(base_model, config)
```

### 직접 확인
학습 가능한 파라미터 수 찍어보기.
```python
model.print_trainable_parameters()
# trainable params: 41,943,040 || all params: 9,241,845,760 || trainable%: 0.45
```

---

## 5. 파라미터별 의미와 튜닝 포인트

조절 나사 하나씩 뜯어봄.

### 5.1 `r=16` (Rank, LoRA 행렬의 중간 차원)
- **의미**: $A(d \times r)$와 $B(r \times d)$ 사이 병목 차원. LoRA의 "용량".
- **영향**: 크면 복잡한 패턴 학습 가능, 대신 VRAM 증가.
- **권장값**: 분류/간단 작업 → 8, 일반 도메인 적응 → 16, 복잡한 스타일 학습 → 32~64.
- **비유**: 연습장 두께. 얇으면 필기 용량 작고, 두꺼우면 많이 쓸 수 있음.

### 5.2 `lora_alpha=32` (Scaling Factor)
- **의미**: 학습된 변화량에 곱하는 스케일. 실제 계산식은 $\alpha / r$ 배율로 적용됨.
- **권장값**: 보통 `r`의 2배. `r=16`이면 `alpha=32`.
- **영향**: 크면 새 지식이 기존 지식 대비 강하게 반영됨. "학습 강도" 다이얼.
- **비유**: 새 메모를 원본 책 위에 **얼마나 굵은 펜으로** 덧씌울지.

### 5.3 `target_modules=["q_proj", "v_proj"]` (학습 대상 지정)
- **의미**: Transformer 내부 어느 선형층(Linear Layer)에 LoRA를 끼울지.
- **주요 후보**:

| 모듈 | 역할 |
|---|---|
| `q_proj`, `k_proj`, `v_proj`, `o_proj` | Attention(토큰 간 관계 계산)의 Query/Key/Value/Output 투영 |
| `gate_proj`, `up_proj`, `down_proj` | FFN(Feed-Forward Network, 지식 저장/변환)의 세 선형층 |
| `embed_tokens`, `lm_head` | 입출력 임베딩/언어모델 헤드 |

- **권장 조합**:
  - 최소: `["q_proj", "v_proj"]` (표준 LoRA 논문)
  - 적극: `["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"]` (all-linear, Unsloth 기본)
- **영향**: 많이 지정할수록 성능 ↑, VRAM ↑.

### 5.4 `use_dora=True`
- **의미**: DoRA 활성화 스위치.
- **영향**: 성능 +1~3점, 속도 약간 저하.
- **트레이드오프**: 실험 반복이 많으면 False, 최종 학습엔 True 권장.

### 5.5 `lora_dropout=0.05`
- **의미**: 학습 중 LoRA 노드 5%를 무작위 비활성화.
- **존재 이유**: 과적합(Overfitting, 훈련 데이터만 달달 외우는 현상) 방지.
- **권장값**: 0.05~0.1. 데이터 적으면 0.1, 많으면 0.05 이하.

### 5.6 `bias="none"`
- **의미**: 편향(bias, 선형층 $y=Wx+b$의 $b$) 학습 여부.
- **선택지**: `none` / `all` / `lora_only`.
- **권장**: 대부분 `none`. 특수 상황 아니면 건드릴 이유 없음.

### 5.7 `task_type="CAUSAL_LM"`
- **의미**: 작업 유형 선언.
- **주요 값**:

| 값 | 용도 |
|---|---|
| `CAUSAL_LM` | 다음 토큰 예측 (GPT, Llama, Gemma 등) |
| `SEQ_2_SEQ_LM` | 입력→출력 변환 (T5, BART) |
| `TOKEN_CLS` | 토큰 단위 분류 (NER) |
| `SEQ_CLS` | 문장 분류 (감정분석 등) |

- **영향**: 내부적으로 손실 함수와 모델 래퍼가 결정됨.

---

## 6. 한눈에 보는 파라미터 요약

| 파라미터 | 비유 | 핵심 역할 | VRAM 영향 |
|---|---|---|---|
| `r` | 연습장 두께 | 학습 용량 결정 | **매우 큼** |
| `lora_alpha` | 펜 굵기 | 반영 강도 | 없음 |
| `target_modules` | 공부할 과목 | 학습 부위 | **매우 큼** |
| `use_dora` | 공부 방법론 | 정밀도 향상 | 소폭 |
| `lora_dropout` | 가끔 한눈팔기 | 과적합 방지 | 없음 |
| `bias` | 잡소음 조정 | 편향 학습 | 미미 |
| `task_type` | 시험 종류 | 손실 함수 결정 | 없음 |

**VRAM 부족 시 우선 조정 순서**: `r` 낮추기 → `target_modules` 축소 → `use_dora=False`.

---

## 7. 언제 무엇을 쓸까

| 상황 | 추천 |
|---|---|
| 빠른 프로토타이핑, 실험 반복 | **LoRA** (r=8~16) |
| 최종 성능 뽑아야 할 때 | **DoRA** (r=16, all-linear) |
| VRAM 극도로 부족 | **QLoRA** (4bit NF4 + LoRA) |
| 정확도 한 끗이 아쉬울 때 | LoRA → `use_dora=True` 전환 |

2026년 기준 트렌드: **DoRA가 사실상 기본 옵션**으로 자리잡음. 연산 오버헤드 10% 정도는 감수하고 쓰는 분위기.

---

## 8. 한마디 요약

**LoRA = 본체 동결 + 얇은 어댑터 학습. DoRA = 그 어댑터를 "크기/방향"으로 쪼개 더 정교하게 학습**. 코드상 차이는 `use_dora=True` 한 줄. 성능 +1~3점, 속도 -10~15%.

---

## 참고 자료
- LoRA 논문 (Hu et al., 2021): https://arxiv.org/abs/2106.09685
- DoRA 논문 (Liu et al., 2024, NVIDIA): https://arxiv.org/abs/2402.09353
- PEFT 공식 문서: https://huggingface.co/docs/peft
- `LoraConfig` API: https://huggingface.co/docs/peft/package_reference/lora
