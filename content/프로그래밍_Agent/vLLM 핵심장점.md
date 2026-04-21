---
생성날짜:
  - 2026-03-26 01:15
마지막수정날짜:
  - 2026-03-26-목요일 01:14
tags:
  - vLLM
  - LLM_Serving
  - PagedAttention
  - Continuous_Batching
  - Guided_Decoding
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## vLLM 없이 로컬 모델 돌리면 뭐가 문제인가

일반적으로 HuggingFace `transformers`로 모델을 로드하면, 요청을 **하나 처리하고 → 끝나면 → 다음 요청** 순서로 돈다. 순차 처리라 느리고, KV Cache(Key-Value 캐시, 이전 토큰들의 어텐션 계산 결과를 저장해 재사용하는 메모리) 공간도 **최대 시퀀스 길이만큼 통째로 예약**해서 낭비가 심함.

vLLM은 이 둘을 모두 해결하는 LLM 서빙 엔진임. UC Berkeley에서 출발한 오픈소스이고, 지금은 실무 표준 중 하나.

---

## vLLM의 핵심 장점 4가지

### 1. 속도: Continuous Batching

여러 요청이 동시에 들어와도 하나씩 기다리지 않고 묶어서 처리한다. 일반 Batching과의 차이가 중요함.

| 방식 | 동작 | 문제 |
|---|---|---|
| Static Batching | N개 요청을 모아서 한 번에 처리. **전원이 끝날 때까지 대기** | 짧은 요청이 긴 요청에 묶여서 같이 기다림 |
| Continuous Batching | 한 요청이 끝나는 **즉시 빈 슬롯에 새 요청 투입** | 대기 시간 최소화, 처리량 최대화 |

[[ALL_FOR_ONE]]처럼 7개 에이전트가 병렬로 도는 구조라면 동시 요청이 많을 텐데, 이때 처리량(Throughput) 차이가 크게 벌어진다.

```
Static Batching:   [req1 ----][req2 --][req3 ------]  ← 다같이 끝날 때까지 대기
Continuous Batching: [req1 ----]
                     [req2 --][req4 -][req6 ---]      ← req2 끝나는 즉시 req4 투입
                     [req3 ------][req5 --]
```

---

### 2. 메모리 효율: PagedAttention

vLLM의 시그니처 기술임. 2023년 논문 "Efficient Memory Management for Large Language Model Serving with PagedAttention"에서 공개됨.

**비유**: 운영체제(OS)의 Virtual Memory(가상 메모리) + Paging(페이징)을 GPU의 KV Cache 관리에 그대로 옮긴 것.

- 기존 방식: "이 요청은 최대 4096 토큰 쓸 수 있으니 4096토큰 분량 메모리를 처음부터 통째로 잡아둠" → 실제로 500토큰만 쓰면 3596토큰 분량이 낭비됨
- PagedAttention: KV Cache를 **블록(Page) 단위**로 쪼개서, **필요한 만큼만** 할당. OS가 프로세스 메모리를 페이지 단위로 관리하는 것과 동일한 원리

결과: 같은 VRAM으로 **더 큰 모델을 올리거나 더 많은 동시 요청**을 처리할 수 있음. RTX 4090 24GB 같은 제한된 환경에서 특히 체감이 큼. 공식 벤치마크에서 기존 `transformers` 대비 2배 이상의 처리량을 낸다고 보고됨.

---

### 3. OpenAI 호환 API 엔드포인트 자동 제공

vLLM 서버를 띄우면 바로 `/v1/chat/completions`, `/v1/completions`, `/v1/embeddings` 같은 OpenAI 호환 엔드포인트가 생긴다.

```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --port 8000
```

기존에 OpenAI나 다른 API를 쓰던 LangChain / LangGraph / OpenAI SDK 코드에서 **base_url만** `http://localhost:8000/v1` 로 바꾸면 끝이다. 코드 수정이 거의 없음.

```python
# 기존 OpenAI 코드
client = OpenAI(api_key="sk-...", base_url="https://api.openai.com/v1")

# vLLM으로 전환
client = OpenAI(api_key="EMPTY", base_url="http://localhost:8000/v1")
```

---

### 4. 다중 클라이언트 접속 가능

한 번 서버로 띄워놓으면 **여러 프로세스, 여러 에이전트**가 동시에 같은 모델에 요청을 보낼 수 있음. `transformers`로 직접 로드하면 프로세스마다 모델 가중치를 중복 로드해야 해서 VRAM이 금방 터짐. vLLM은 서버 하나 / 가중치 한 번 로드 / 클라이언트 N개 접속 구조라 VRAM을 아낀다.

---

## 상세 개념 정리

### PagedAttention

LLM 추론 시 KV Cache를 OS의 가상 메모리 페이징처럼 **블록 단위로 관리하는 방식**임. 기존에는 최대 시퀀스 길이만큼 메모리를 통째로 예약해야 해서 낭비가 심했는데, PagedAttention은 필요한 만큼만 할당하므로 **GPU 메모리를 훨씬 효율적으로 사용**할 수 있다. 그 결과 같은 GPU에서 더 빠르고 더 많은 요청을 처리할 수 있게 됨.

### Guided Decoding

모델이 토큰을 하나씩 생성할 때마다, 다음에 올 수 있는 토큰을 JSON 스키마 / 정규식 / 문법에 맞는 것들로만 **제한하는 기법**임. 예를 들어 `{"score":` 까지 생성했으면 다음 토큰은 숫자만 허용하는 식. 그래서 **출력이 항상 유효한 JSON이 되도록 강제**한다.

vLLM은 `guided_json`, `guided_regex`, `guided_choice`, `guided_grammar` 파라미터를 지원함:

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct",
    messages=[{"role": "user", "content": "사용자 정보를 JSON으로 추출해줘"}],
    extra_body={
        "guided_json": {
            "type": "object",
            "properties": {
                "name": {"type": "string"},
                "age": {"type": "integer"}
            },
            "required": ["name", "age"]
        }
    }
)
```

이게 [[langchain_with_structured_output의 강제력|LangChain의 with_structured_output]]의 **self-hosted 실제 구현**에 해당함. LangChain은 위 파라미터를 대신 꽂아주는 래퍼고, 진짜 토큰 레벨 제약은 vLLM이 수행.

### Continuous Batching

일반적인 Batching은 여러 요청을 모아서 한 번에 처리하되, 모든 요청이 끝날 때까지 기다려야 한다. Continuous Batching은 **요청이 끝나는 즉시 새 요청을 바로 투입**하는 방식임. 보이스피싱 탐지처럼 여러 통화를 동시에 분석해야 하는 상황에서, 한 통화 분석이 끝나면 바로 다음 통화를 GPU에 넣을 수 있어 **처리량(throughput)이 크게 향상**된다.

---

## 포함 관계

```
LLM Serving(LLM을 여러 클라이언트에 노출시키는 인프라 영역)
│
├── 단순 로드 방식 (transformers)  ← 개발/실험용
│
└── 전용 서빙 엔진
    │
    ├── vLLM  ← PagedAttention + Continuous Batching + Guided Decoding
    ├── TGI (HuggingFace Text Generation Inference)
    ├── TensorRT-LLM (NVIDIA 전용)
    └── llama.cpp / Ollama (경량화, CPU/Mac 친화)
```

vLLM은 이 중 **GPU 서빙 처리량 최적화** 관점에서 가장 널리 쓰이는 옵션임.

---

### 한마디 요약

**"vLLM은 로컬 모델을 '그냥 돌리는 것'에서 '제대로 서빙하는 것'으로 바꿔주는 도구."**

관련: [[LLM Integration(통합)과 Inference Pipeline(추론 파이프라인)]], [[langchain_with_structured_output의 강제력]], [[with_structured_output]]
