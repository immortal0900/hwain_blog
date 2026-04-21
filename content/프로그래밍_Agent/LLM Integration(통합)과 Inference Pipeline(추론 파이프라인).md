---
생성날짜:
- 2026-02-11 20:09
마지막수정날짜:
- 2026-02-11-수요일 20:08
tags:
  - LLM
  - LLM_Integration
  - Inference_Pipeline
  - RAG
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## 같이 쓰이지만 층위가 다른 두 용어

LLM 프로젝트 문서에서 자주 같이 나오는 두 단어인데, 구조(Integration)와 흐름(Pipeline)로 층위가 다름. 헷갈려서 정리.

---

## 1. LLM Integration (LLM 통합)

**"우리가 만든 서비스(앱/웹)에 거대 언어 모델(LLM)을 연결하는 행위와 구조"** 를 말한다.

단순히 챗봇을 만드는 것을 넘어, LLM이 우리 서비스의 DB / 외부 API / UI와 상호작용하도록 만드는 모든 과정을 포함함. 관점은 **"어떻게 붙일 것인가"** 에 있음.

### 주요 구성 요소

- **Model Serving(모델 서빙, LLM을 실제 호출 가능한 상태로 띄우는 것)**
    - API 방식: OpenAI(GPT-4), Anthropic(Claude), Google(Gemini) 같은 외부 API 호출
    - 로컬 호스팅: [[vLLM 핵심장점|vLLM]], Ollama 등으로 내 서버(GPU)에 Llama 3 같은 모델을 직접 띄워놓고 연결

- **Orchestration(오케스트레이션, LLM + 외부 도구의 실행 순서 관리)**
    - LLM이 혼자서는 못 하는 일(검색, 계산, DB 조회 등)을 연결해주는 도구들
    - 대표 프레임워크: LangChain, LangGraph, LlamaIndex
    - 사용자의 질문을 분석해서 검색이 필요한지, 계산기가 필요한지 판단하고 연결

- **Tool Use / Function Calling(함수 호출)**
    - LLM에게 "너는 계산기랑 날씨 API를 쓸 수 있다"고 알려주고, 필요할 때 함수를 실행하게 만드는 것
    - MCP(Model Context Protocol, Anthropic이 만든 LLM-외부도구 연결 표준 프로토콜)도 여기에 해당함

---

## 2. Inference Pipeline (추론 파이프라인)

**"사용자가 질문을 입력하고(Input), 최종 답변(Output)을 받기까지 데이터가 흐르는 일련의 처리 과정"** 이다.

LLM은 텍스트를 그대로 이해하지 못한다. 숫자로 바꾸고 계산한 뒤 다시 텍스트로 바꾸는 과정이 필요함. 이 전체 흐름을 파이프라인이라고 부름. 관점은 **"데이터가 어떻게 흐르는가"** 에 있음.

### 파이프라인의 4단계 (End-to-End)

1. **Pre-processing / Tokenization(전처리 / 토큰화)**
    - 사용자의 질문("안녕?")을 모델이 이해할 수 있는 숫자(Token ID)로 변환
    - 프롬프트 템플릿을 입히는 과정도 여기 포함됨 (예: "너는 친절한 AI야" + "안녕?")

2. **Retrieval + Context Injection(검색 + 컨텍스트 주입, RAG 사용 시)**
    - RAG(Retrieval-Augmented Generation, 외부 지식을 검색해서 프롬프트에 끼워넣는 기법)
    - 질문과 관련된 데이터를 Vector DB(벡터 DB, 의미 기반 검색을 위해 임베딩 벡터를 저장하는 DB)에서 찾아와 프롬프트에 끼워 넣음
    - 선택 단계임 (RAG 안 쓰면 스킵)

3. **Model Inference(모델 추론)**
    - 가장 무거운 단계임. GPU가 행렬 연산을 수행하여 다음 토큰(글자)을 확률적으로 예측
    - KV Cache(Key-Value 캐시, 이전 토큰들의 어텐션 계산 결과를 저장해 재사용하는 메모리) 같은 기술로 속도를 높임
    - [[vLLM 핵심장점|vLLM의 PagedAttention]]이 이 단계의 메모리 효율을 끌어올리는 기술

4. **Post-processing / Detokenization(후처리 / 역토큰화)**
    - 모델이 뱉어낸 숫자들을 다시 사람이 읽을 수 있는 글자로 변환
    - 욕설 필터링, 포맷팅(JSON 변환) 같은 마무리 작업 수행
    - [[with_structured_output|Structured Output]] 도 여기서 강제됨

---

## 두 용어의 포함 관계

```
LLM Integration (구조 설계, "어떻게 붙일 것인가")
│
├── Model Serving
├── Orchestration
├── Tool Use
│
└── Inference Pipeline (데이터 흐름, "어떻게 처리되는가")
    ├── Pre-processing
    ├── Retrieval
    ├── Model Inference
    └── Post-processing
```

Integration이 **큰 그림의 구조 설계**, Pipeline은 그 안에서 **한 번의 요청이 흐르는 경로**로 이해하면 됨. Pipeline은 Integration의 일부.

---

## 실무 대화에서의 쓰임

| 상황 | 뜻 | 성격 |
|---|---|---|
| "나 이번에 LLM Integration 작업해야 해" | "우리 앱에서 버튼 누르면 GPT-4가 동작하게 API 연결하고, DB에서 정보 가져오도록 LangChain 코드 짜야 해" | 구조 설계 |
| "Inference Pipeline 최적화가 필요해" | "사용자가 질문하고 답변받는 데 5초나 걸리네? 전처리 과정을 줄이거나 vLLM을 써서 GPU 연산 속도를 높여야겠어" | 성능 튜닝 |

---

### 한마디 요약

**"Integration은 LLM을 서비스에 붙이는 설계도, Pipeline은 한 요청이 안에서 흐르는 경로."**

관련: [[vLLM 핵심장점]], [[with_structured_output]], [[AI AGENT TASK 분해 기준 단위]]
