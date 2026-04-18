---
생성날짜:
  - 2026-03-30 04:31
마지막수정날짜:
  - 2026-03-30-월요일 04:31
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
**`isinstance(obj, Type)`** — "이 객체가 이 타입이야?"

python

```python
isinstance(m, ToolMessage)  # m이 ToolMessage 클래스의 인스턴스인지 True/False
isinstance(3, int)          # True
isinstance("hi", int)       # False
```

`type(m) == ToolMessage`과 비슷하지만, `isinstance`는 **상속 관계도 인정**해준다는 차이가 있어요. `ToolMessage`가 `BaseMessage`를 상속했다면 `isinstance(m, BaseMessage)`도 `True`가 됩니다.

**`getattr(obj, "attr", default)`** — "이 객체에서 속성 꺼내줘, 없으면 기본값"

python

```python
getattr(m, "name", "")  # m.name이 있으면 그 값, 없으면 빈 문자열 ""
```

`m.name`을 직접 쓰면 속성이 없을 때 `AttributeError`가 터지는데, `getattr`은 세 번째 인자(default)를 돌려주니까 안전하게 접근할 수 있어요.

**Step 1** — ToolMessage에서 피드백 추출 (Line 220-227):

```python
for m in reversed(msgs):
    if isinstance(m, ToolMessage) and getattr(m, "name", "") == "think_tool":
        feedback = m.content or ""
        break
```

**코드 전체 흐름을 한 줄로 읽으면:**

> msgs를 뒤에서부터 순회하면서, **ToolMessage 타입이면서 name 속성이 "think_tool"인** 첫 번째 메시지를 찾아 그 content를 feedback에 담고 멈춰라.