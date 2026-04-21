---
생성날짜:
  - 2026-03-30 04:31
마지막수정날짜:
  - 2026-03-30-월요일 04:31
tags:
  - python
  - basic
  - introspection
  - agent
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---

> LangGraph나 LangChain 돌리다 보면 `msgs` 안에 `HumanMessage`, `AIMessage`, `ToolMessage`가 섞여 온다. "이 중에 ToolMessage만 골라서 그 안의 name이 'think_tool'인 것만 꺼내"같은 일을 매일 한다. 그걸 안전하게 하는 두 함수가 `isinstance`와 `getattr`다. 관련 [[파이썬에서 `.`, `대괄호`의 쓰임(내부에 있는 무언가를 지칭한다)]]도 참고.

## 1. `isinstance(obj, Type)` : "이 객체가 이 타입이야?"

**`isinstance(obj, Type)`(is instance of, 어떤 클래스의 인스턴스인지 확인, 상속 관계까지 True로 인정)** 은 타입 체크 함수다.

```python
isinstance(m, ToolMessage)   # m이 ToolMessage 클래스의 인스턴스인지 True/False
isinstance(3, int)            # True
isinstance("hi", int)         # False
isinstance(3, (int, float))   # 튜플로 여러 타입 한 번에 검사
```

### `type(x) == Cls` 와의 차이 (상속 인정)

| 비교 방식 | 상속 관계 인정 | 여러 타입 한 번에 | 추천도 |
|-----------|---------------|------------------|--------|
| `type(m) == ToolMessage` | X (정확히 같은 클래스여야 True) | X | 낮음 |
| `isinstance(m, ToolMessage)` | O (부모/자식 클래스도 True) | O (튜플 전달) | 높음 |

LangChain의 경우 `ToolMessage`가 `BaseMessage`를 상속하므로 `isinstance(m, BaseMessage)`도 True가 된다. 필터링할 때 "이 계열 전부" 잡고 싶으면 부모 클래스로 검사.

### 비유

공항 출입국 심사와 같다. `type` 비교는 "이 사람 대한민국 국적자 맞음?" 정확히 묻는 것이고, `isinstance`는 "동아시아권에서 왔음?"처럼 상위 카테고리까지 포함해 묻는 것.

---

## 2. `getattr(obj, "attr", default)` : "속성 꺼내줘, 없으면 기본값"

**`getattr(obj, "attr", default)`(get attribute, 속성 이름을 문자열로 받아 꺼냄, 없으면 기본값 반환)** 은 안전한 속성 접근 함수다.

```python
getattr(m, "name", "")        # m.name이 있으면 그 값, 없으면 빈 문자열
getattr(m, "tool_call_id", None)
```

### `m.name` 직접 접근과의 차이

| 접근 방식 | 속성이 없을 때 | 속성 이름을 런타임에 결정 | 추천 상황 |
|-----------|---------------|-----------------------|----------|
| `m.name` | `AttributeError` 발생 | 불가 (코드에 하드코딩) | 속성이 반드시 있다고 보장될 때 |
| `getattr(m, "name", "")` | 기본값 반환 | 가능 (문자열 변수로 넘김) | 메시지 타입이 섞여 있을 때 |

### 동적 속성 접근 예시

```python
# 사용자가 지정한 필드를 꺼내는 경우
field_name = user_input["field"]   # 예: "content"
value = getattr(msg, field_name, None)
```

이게 `m.field_name`으로는 안 된다. 그건 글자 그대로 `field_name`이라는 속성을 찾으러 감.

---

## 3. 친척 함수들 한눈에

| 함수 | 용도 | 예시 | 없을 때 |
|------|------|------|---------|
| `getattr(obj, "x", default)` | 속성 읽기 | `getattr(m, "name", "")` | 기본값 반환 |
| `setattr(obj, "x", value)` | 속성 쓰기 | `setattr(m, "name", "tool")` | 새로 만듦 |
| `hasattr(obj, "x")` | 있는지 확인 | `if hasattr(m, "name"):` | False 반환 |
| `delattr(obj, "x")` | 속성 삭제 | `delattr(m, "name")` | AttributeError |

`hasattr`은 내부적으로 `getattr` 후 예외 잡는 형태. 딱 존재 여부만 알고 싶을 때.

---

## 4. 실제 Agent 코드 예시

### 예시 1: 특정 툴 메시지 골라내기

```python
# msgs를 뒤에서부터 순회하며 think_tool의 마지막 출력 찾기
for m in reversed(msgs):
    if isinstance(m, ToolMessage) and getattr(m, "name", "") == "think_tool":
        feedback = m.content or ""
        break
```

**한 줄로 읽기:** msgs를 역순으로 보면서 **ToolMessage이면서 name 속성이 "think_tool"인** 첫 메시지를 찾아 content를 feedback에 저장하고 멈춤.

### 예시 2: 다양한 메시지 타입을 한 리스트에서 나누기

```python
from langchain_core.messages import HumanMessage, AIMessage, ToolMessage

humans = [m for m in msgs if isinstance(m, HumanMessage)]
ais    = [m for m in msgs if isinstance(m, AIMessage)]
tools  = [m for m in msgs if isinstance(m, ToolMessage)]
```

### 예시 3: 응답 구조가 스펙에 따라 다를 때

```python
# OpenAI는 choices[0].message, Anthropic은 content[0].text
text = (
    getattr(response, "content", None)
    or getattr(response.choices[0].message, "content", "")
)
```

---

## 5. 단계 분해: Agent가 메시지 필터링할 때

1단계 → 상태에서 `messages` 리스트 꺼냄 (`state["messages"]`)
2단계 → `isinstance`로 원하는 메시지 타입만 걸러냄
3단계 → `getattr`로 각 메시지에서 필요한 필드(name, tool_call_id, content) 안전하게 추출
4단계 → 비어 있을 수 있으니 `or` 기본값으로 방어
5단계 → 다음 노드로 넘길 입력 구성

---

## 6. 왜 이렇게 쓰는가

- **Duck typing(덕 타이핑, 파이썬 기본 철학, "오리처럼 생겼고 오리처럼 울면 오리다")**: 원래는 타입 검사 없이 메서드 호출해도 됨
- 하지만 **Agent 같이 여러 라이브러리가 섞이는 환경**에선 타입이 꼬이기 쉬움
- `isinstance`로 "이 계열 메시지만" 명확히 거르는 게 안전
- `getattr`로 기본값 처리해두면 라이브러리 버전 올라가며 필드가 바뀌어도 AttributeError로 파이프라인이 죽지 않음

### 트레이드오프

- 과도한 `isinstance` 체크는 OOP(객체 지향) 설계 원칙 위배. 진짜 해결책은 공통 인터페이스(`BaseMessage.content`) 쓰거나 다형성(polymorphism)으로 처리
- `getattr` 남용은 "어떤 필드가 올지 모르겠는데 일단 꺼내자"식 코드로 이어져 버그 원인이 됨. [[Pydantic]]이나 [[dataclass]]로 구조를 확정짓는 게 원칙

---

## 7. 직접 확인

```python
>>> class A: pass
>>> class B(A): pass
>>> b = B()
>>> type(b) == A              # False (정확히 같은 클래스여야)
>>> isinstance(b, A)          # True (상속 관계 인정)
>>> getattr(b, "x", "없음")    # '없음'
>>> hasattr(b, "x")            # False
>>> setattr(b, "x", 42)
>>> getattr(b, "x", "없음")    # 42
```

---

## 8. 한마디 요약

`isinstance`는 "이 객체가 그 계열인지" 상속 포함해서 묻고, `getattr`은 "있으면 꺼내고 없으면 기본값"으로 안전하게 속성 접근하는 함수다.
