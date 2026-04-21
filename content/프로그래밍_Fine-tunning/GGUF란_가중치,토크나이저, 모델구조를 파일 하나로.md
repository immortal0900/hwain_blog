---
생성날짜:
- 2026-02-03 00:32
마지막수정날짜:
- 2026-02-03-화요일 00:32
tags:
- 파인튜닝
- 양자화
- 추론엔진
- GGUF
- llama.cpp
- Ollama
별칭:
- GGUF 포맷
- GGML 후속
type:
- 자료수집
Area/Reasource:
Project:
---

# GGUF: 가중치 + 토크나이저 + 모델구조를 파일 하나로

GGUF(GPT-Generated Unified Format, 로컬 LLM 추론을 위해 설계된 단일 바이너리 포맷)는 한마디로 **"내 PC에서 LLM을 가장 쉽고 빠르게 돌리기 위한 통합 파일 포맷"**임.

관련 노트: [[LoRA vs DoRA]], [[파인튜닝 종합정리]]

---

## 1. 왜 만들어졌나 (설계 의도)

### 기존 방식의 문제점
HuggingFace 표준으로 학습시키면 결과물이 여러 조각으로 흩어짐.
- `pytorch_model.bin` 또는 `model.safetensors` (가중치)
- `config.json` (모델 구조 설정)
- `tokenizer.json`, `tokenizer_config.json`, `special_tokens_map.json` (토크나이저)
- 어댑터를 쓰면 `adapter_model.safetensors` 까지 추가

→ 파일 하나만 빠져도 실행 안 됨. 용량도 Gemma 2 9B 기준 FP16이 약 17GB 수준.

### GGUF의 해결책
위 모든 정보를 **`.gguf` 단일 파일**에 집어넣고, 동시에 2~8비트로 양자화(Quantization, 고정밀 수치를 적은 비트로 압축) 가능하게 설계됨. 파일 하나만 전달하면 로컬 환경에서 바로 실행됨.

**일상 비유**: 예전엔 "본체 + 설명서 + 리모컨 + 건전지"를 따로 챙겨야 했다면, GGUF는 "전원만 꽂으면 바로 켜지는 일체형 기기"임.

---

## 2. 원리: 파일 내부에 뭐가 들어있나

GGUF는 크게 세 영역으로 구성됨.

| 영역 | 내용 | 역할 |
|---|---|---|
| **Header** | 매직 넘버(`GGUF`), 버전, 텐서 개수, 메타데이터 개수 | 파일 식별 및 전체 크기 파악 |
| **Metadata (KV Store)** | 모델 구조, 토크나이저 정보, 양자화 방식, prompt template | 실행 엔진이 추론할 때 참조 |
| **Tensor Data** | 실제 가중치(양자화 적용됨) | 모델의 "지식 본체" |

### 핵심 최적화 기술 3가지

1. **단일 파일 구조**: 가중치, 설정, 토크나이저가 한 파일에 포함됨. 누락 위험 제로.

2. **mmap(Memory Mapping, 파일을 가상 메모리 공간에 직접 매핑해 필요한 부분만 읽는 OS 기능) 최적화**: 파일 전체를 RAM에 올리지 않고, OS의 페이지 캐시를 이용해 접근하는 부분만 즉시 로드. 9B 모델을 수 초 내에 기동 가능.

3. **다단계 양자화**: K-quant(Q2_K, Q4_K_M, Q5_K_M 등) 계열로 중요도에 따라 비트 수를 다르게 할당. 핵심 레이어는 정밀하게, 덜 중요한 레이어는 과감히 압축.

---

## 3. 양자화 레벨 선택 가이드

파일 이름에 붙는 접미사(`Q4_K_M` 등)가 양자화 방식을 나타냄. 실사용에서 가장 자주 선택하는 조합만 정리.

| 양자화 | 비트(평균) | 7B 기준 용량 | 품질 손실 | 추천 상황 |
|---|---|---|---|---|
| **Q2_K** | 약 2.6 bpw | 약 2.8GB | 큼 | 메모리 극도 부족 |
| **Q4_K_M** | 약 4.5 bpw | 약 4.1GB | 거의 없음 | **기본 추천** (속도/품질 균형) |
| **Q5_K_M** | 약 5.5 bpw | 약 4.8GB | 미미 | 품질 우선 |
| **Q8_0** | 8.0 bpw | 약 7.2GB | 없음 | FP16 대비 2배 압축만 필요 |
| **F16** | 16 bpw | 약 13GB | 원본 그대로 | 양자화 전 기준 모델 |

**bpw**(bits per weight, 가중치 하나당 평균 비트 수): 양자화 효율의 표준 지표.

**한마디 요약**: Q4_K_M 쓰면 대부분 맞음. VRAM 여유 있으면 Q5_K_M.

---

## 4. 언제 쓰나 (사용 시나리오)

| 상황 | 이유 |
|---|---|
| **CPU만으로 LLM 돌려야 할 때** | GPU 없어도 RAM 16GB면 7B급 실행 가능 |
| **모델을 배포/공유할 때** | 파일 하나만 전달하면 끝. 의존성 지옥 없음 |
| **로컬 PC 실시간 챗봇** | 5GB 내외로 압축, 지능 손실 최소 |
| **오프라인 환경** | 인터넷 없이 완전 로컬 동작 |

포함 관계: GGUF는 **양자화 포맷**의 하위 분류 중 하나. 전체 계층은 `추론 포맷 > 양자화 포맷 > GGUF / AWQ / GPTQ / EXL2`. 이 중 GGUF만 CPU 추론에 특화됨.

---

## 5. 만드는 법 (SafeTensors → GGUF 변환)

파인튜닝 결과물은 보통 HuggingFace 표준인 SafeTensors 형식임. 이를 GGUF로 바꾸려면 `llama.cpp` 저장소의 스크립트 사용.

### 1단계: llama.cpp 준비
```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
pip install -r requirements.txt
make  # llama-quantize 등 바이너리 빌드
```

### 2단계: FP16 통합 변환
먼저 흩어진 HuggingFace 파일을 FP16 `.gguf` 하나로 합침.

```bash
python convert_hf_to_gguf.py /path/to/my-model \
    --outfile my_model_f16.gguf \
    --outtype f16
```

### 3단계: 양자화 적용
```bash
./llama-quantize my_model_f16.gguf my_model_q4_k_m.gguf Q4_K_M
```

### 직접 확인
변환 후 크기를 비교해 보면 체감됨.
```bash
ls -lh my_model_*.gguf
# my_model_f16.gguf     -> 약 13GB
# my_model_q4_k_m.gguf  -> 약 4.1GB
```

---

## 6. 실행하는 법 (추론 엔진 선택)

같은 `.gguf` 파일을 다양한 엔진에서 재사용 가능함.

| 엔진 | 장점 | 용도 |
|---|---|---|
| **llama.cpp** | C++ 네이티브, 가장 가볍고 빠름 | 터미널/임베디드 |
| **Ollama** | `ollama run model` 한 줄 실행 | 맥/윈도우 데스크톱 |
| **LM Studio** | GUI, 코딩 불필요 | 비개발자/테스트 |
| **vLLM** | PagedAttention 기반 고성능 서빙 | 프로덕션 API 서버 |
| **text-generation-webui** | 웹 UI + 다양한 파라미터 | 실험/튜닝 |

### Ollama 등록 예시 (가장 실용적)
`Modelfile` 작성 후 import.
```
FROM ./my_model_q4_k_m.gguf
PARAMETER temperature 0.7
TEMPLATE """{{ .System }}\n\nUser: {{ .Prompt }}\nAssistant:"""
```
```bash
ollama create my-model -f Modelfile
ollama run my-model
```

---

## 7. GGUF vs 다른 포맷 (왜 GGUF를 골랐나)

| 비교축 | SafeTensors | GGUF | AWQ/GPTQ |
|---|---|---|---|
| **파일 구성** | 가중치만 + 별도 설정 여러 개 | 단일 파일 통합 | 가중치 + 설정 분리 |
| **CPU 추론** | 매우 느림 | **최적화됨** | GPU 전용 |
| **양자화 종류** | 없음(원본) | Q2~Q8 다양 | 4bit 주력 |
| **주 런타임** | transformers, vLLM | llama.cpp 계열 | vLLM, AutoAWQ |
| **속도(동일 GPU)** | 기준 | 중 | **빠름** |

**한마디 요약**: GPU 있으면 AWQ/GPTQ, CPU나 로컬 편의성 우선이면 GGUF.

---

## 8. 전체 파이프라인에서의 위치

학습부터 서비스까지 흐름 중 GGUF가 들어가는 위치는 명확함.

```
[데이터 준비]
    ↓
[Unsloth + QLoRA/NF4 학습]  ← 학습 단계
    ↓
[어댑터 병합 → SafeTensors]
    ↓
[GGUF 변환 + 양자화]        ← 여기
    ↓
[Ollama/vLLM 서빙]          ← 배포 단계
    ↓
[FastAPI 연결 → 사용자]
```

- **학습용 포맷**: Unsloth, QLoRA (본체는 여전히 HuggingFace 형식)
- **배포용 포맷**: GGUF (압축 + 단일 파일화)

---

## 9. 한마디 요약

**GGUF = "학습 끝난 모델을 내 PC나 저사양 환경에서 가장 빠르고 가볍게 돌리기 위한 단일 파일 포맷"**. 본체 + 설정 + 토크나이저를 한 파일에 몰아넣고, 2~8비트 양자화로 용량까지 줄인 배포용 결정판임.

---

## 참고 자료
- llama.cpp 공식 저장소: https://github.com/ggerganov/llama.cpp
- GGUF 스펙 문서: https://github.com/ggerganov/ggml/blob/master/docs/gguf.md
- Ollama Modelfile 문서: https://github.com/ollama/ollama/blob/main/docs/modelfile.md
