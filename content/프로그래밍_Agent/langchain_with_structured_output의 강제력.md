---
생성날짜:
- 2026-03-01 18:42
마지막수정날짜:
- 2026-03-01-일요일 18:42
tags:
  - LangChain
  - Structured_Output
  - LLM
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## 왜 프롬프트로 "JSON만 줘"라고 하면 못 믿나

일반 프롬프트로 "JSON 형식으로 답해줘"라고 지시하면, LLM이 가끔 앞뒤에 설명 텍스트를 덧붙이거나 마크다운 코드펜스(\`\`\`json ... \`\`\`)로 감싸는 실수를 함. 프로덕션 파싱에서는 이게 곧 버그가 된다. 그래서 프롬프트 레벨이 아니라 **API 레벨에서 출력 형태를 강제하는 장치**가 필요함. 그게 LangChain의 `with_structured_output()`임.

---

## 실제 동작 방식: 3가지 method 중 하나

공식 문서(LangChain Docs) 기준으로 `with_structured_output()`은 모델에 따라 아래 3가지 method 중 하나를 사용한다. 프롬프트 한 줄에 의존하는 게 아니라 **API 호출 시 구조 강제 파라미터를 함께 넘긴다**.

| method | 설명 | 강제 강도 | 대표 공급자 |
|---|---|---|---|
| `function_calling` | Tool/Function Calling 스펙으로 변환해 tool call 강제 | 강 | OpenAI, Anthropic, 대부분의 provider (기본값) |
| `json_schema` | 모델이 지원하는 공식 JSON Schema 파라미터로 API에 직접 주입 | 가장 강 (API 레벨 grammar 제약) | OpenAI Structured Outputs, Gemini `response_schema` |
| `json_mode` | "JSON만 반환" 플래그만 켬. 스키마는 프롬프트로 기술 | 약 (형식만 JSON, 필드는 보장 안됨) | OpenAI `response_format={"type":"json_object"}` |

```python
from langchain_openai import ChatOpenAI
from pydantic import BaseModel

class Movie(BaseModel):
    title: str
    year: int

llm = ChatOpenAI(model="gpt-4.1")

# method 지정 없이 쓰면 provider별 기본값 (OpenAI면 function_calling)
structured_llm = llm.with_structured_output(Movie)

# strict=True로 스키마 완전 일치 강제 (json_schema / function_calling에서만 지원, json_mode는 미지원)
strict_llm = llm.with_structured_output(Movie, method="json_schema", strict=True)
```

---

## 왜 `function_calling`이 기본값인가

Tool Calling(도구 호출, LLM이 미리 정의된 함수를 호출하도록 학습된 동작)은 **대부분의 상업 LLM이 학습 단계에서 이미 익힌 스펙**임. 그래서 스키마 준수율이 매우 높다. LangChain 입장에서는 "스키마를 tool 정의로 변환 → bind_tools 호출 → 응답에서 tool_call 추출 → Pydantic 객체로 파싱" 이 일련의 과정을 `with_structured_output()` 한 줄로 추상화한 것.

```
사용자 코드: llm.with_structured_output(Movie).invoke(prompt)
     ↓
LangChain 내부:
   1. Movie(Pydantic) → OpenAI function schema 변환
   2. llm.bind_tools([movie_fn], tool_choice="movie")  ← 강제 호출
   3. API 응답의 tool_call 인자를 추출
   4. Movie(**args)로 파싱하여 반환
```

위 4단계 중 2번의 `tool_choice` 강제가 "Structured Output의 강제력"의 실체다. 모델이 일반 텍스트로 답하는 경로 자체를 닫아버리는 셈.

---

## `strict=True`는 한 단계 더 강함

OpenAI의 **Structured Outputs** 기능(2024년 8월 공개)은 API 단에서 **constrained decoding(제약 디코딩, 스키마를 벗어나는 토큰을 샘플링 후보에서 제외)** 을 수행함. `strict=True`로 켜면 이걸 활성화한다. 토큰 레벨에서 스키마에 맞지 않는 글자는 아예 생성 확률을 0으로 만드므로, 출력이 **100% 스키마 준수** 보장됨.

- `strict=False`: 모델이 스키마에 맞게 답하도록 유도만. 어긋나면 validation error 가능성 있음
- `strict=True`: 토큰 샘플링 단계에서 제약. 단, 지원 method는 `function_calling` 또는 `json_schema`만 해당. `json_mode`는 미지원

Gemini의 `json_schema` method도 `response_schema` 파라미터로 유사한 제약 디코딩을 수행함.

---

## "Grammar-guided decoding"은 어디서 오는가

초반에 헷갈렸던 부분: "LangChain이 토큰 확률을 0으로 만든다"는 설명은 **정확히는 LangChain이 아니라 그 아래 layer(OpenAI Structured Outputs, vLLM Guided Decoding, Outlines 라이브러리 등)의 역할**임. LangChain은 이 기능들을 쓸 수 있도록 **API 파라미터를 꽂아주는 래퍼(wrapper)** 역할.

- LangChain: 스키마 → API 파라미터 변환, 응답 파싱
- OpenAI / vLLM / Outlines: 실제 token-level 제약 디코딩 수행

[[vLLM 핵심장점|vLLM의 Guided Decoding]]도 이 layer에 해당함. 같은 개념을 self-hosted 환경에서 구현한 것.

---

## 직접 확인하는 법

```python
llm = ChatOpenAI(model="gpt-4.1")
structured = llm.with_structured_output(Movie)

# 내부 호출 구조 확인: bind_tools가 걸린 chain이 나옴
print(structured)
# → RunnableBinding(bound=..., kwargs={'tools': [...], 'tool_choice': {...}})
```

`tool_choice`에 우리가 준 스키마가 강제 선택되어 있는 걸 눈으로 확인 가능함.

---

### 한마디 요약

**"프롬프트가 아니라 API의 tool_choice / json_schema 파라미터로 LLM의 출력 경로 자체를 구조화된 경로만 열어두는 것."**

관련: [[with_structured_output]], [[vLLM 핵심장점]], [[LLM Integration(통합)과 Inference Pipeline(추론 파이프라인)]]
