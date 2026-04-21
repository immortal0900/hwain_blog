---
생성날짜:
  - 2026-03-12 13:33
마지막수정날짜:
  - 2026-03-12-목요일 13:33
tags:
  - langfuse
  - opentelemetry
  - span
  - attributes
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## span_id

### Span이란

**하나의 작업 단위**를 뜻한다.

```
사용자가 질문을 보냄
  │
  ├─ [Span 1] 질문 분석         (0.1초)
  ├─ [Span 2] 구글 검색 호출     (1.2초)
  ├─ [Span 3] LLM에게 답변 요청  (3.5초)
  └─ [Span 4] 결과 포맷팅        (0.05초)
```

각 Span에 붙인 고유 ID가 `span_id`.

- **span_id가 없으면**: "LLM 호출이 느렸다"는 건 알지만 **어느 단계에서** 느렸는지 구분 불가.
- **span_id가 있으면**: "span_id=sp_456(구글 검색)에서 1.2초 걸림" 처럼 **정확히 어디가 병목인지** 특정 가능.

## Attributes

**각 Span에 붙이는 부가 정보(꼬리표)**다.

```
[Span 3] LLM 호출
  ├─ trace_id:    "tr_123"        ← 이 요청 전체의 ID
  ├─ span_id:     "sp_456"        ← 이 단계의 ID
  ├─ model:       "gpt-4o"        ← 어떤 모델?
  ├─ tokens:      1523            ← 토큰 몇 개?
  ├─ session_id:  "user_kim_001"  ← 누구의 세션?
  └─ cost:        0.0045          ← 비용 얼마?
```

이것들이 전부 **attributes**.

## 다시 `propagate_attributes()`에 연결하면

```
부모 Task의 attributes:
  trace_id = "tr_123"
  session_id = "user_kim_001"
      │
      │  propagate_attributes()
      │  = "이 꼬리표들을 자식에게도 복사해라"
      ▼
자식 Task도 동일한 꼬리표를 가짐
  → Langfuse가 "아, 이것들은 전부 같은 요청의 일부구나" 인식
```

## 포함 관계 한눈에

- **Trace**(요청 1회 단위) ⊃ **Span**(작업 구간 단위, 여러 개) ⊃ **Attributes**(각 Span의 부가 정보)
- `trace_id`는 Trace 단위 ID, `span_id`는 Span 단위 ID → 같은 `trace_id`를 공유하는 여러 `span_id`가 한 Trace에 묶임

## 한마디로

- **Span** = 작업 단위 (구간)
- **span_id** = 그 구간의 고유 이름표
- **Attributes** = 구간에 붙은 부가 정보 (모델명, 비용, 세션 등)
- **propagate** = 이 부가 정보들을 자식 Task로 물려주는 것

[[langfuse_propagate_attributes()의 원래 역할]]
