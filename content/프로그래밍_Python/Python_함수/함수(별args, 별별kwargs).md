---
생성날짜:
- 2026-01-27 02:56
마지막수정날짜:
- 2026-01-27-화요일 02:55
tags:
- Python
- 함수
- 가변인자
- args
- kwargs
- AI_Agent
별칭:
- star args
- double star kwargs
- variable arguments
- unpacking
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python/함수
Project:
---

## 한마디 요약

**`*args` / `**kwargs`** 는 "몇 개 들어올지 모르는 인자를 받아두는 바구니"다. 함수 **정의부**에서는 **묶기(packing)**, **호출부**에서는 **풀기(unpacking)** 로 방향이 반대로 쓰인다. AI Agent 코드에서는 데코레이터, 콜백 전달, LLM 응답의 동적 필드 처리에 거의 매번 등장.

---

## 가변 인자(Variable Arguments) 가 무엇인가

**가변 인자** 는 "개수가 고정되지 않은 인자" 를 받는 Python 문법. 크게 두 종류.

| 문법 | 데이터 형태 | 의미 | 비유 |
|---|---|---|---|
| `*args` | 튜플 `(1, 2, 3)` | 위치 인자(positional) 를 통째로 | "장바구니에 순서대로 담아와" |
| `**kwargs` | 딕셔너리 `{"a": 1}` | 키워드 인자(keyword, `이름=값`) 를 통째로 | "포스트잇 붙여서 박스에 담아와" |

이름 `args`, `kwargs` 는 관례일 뿐 필수 아님. **핵심은 별표 `*`, `**` 개수.**

---

## 1단계: `*args` 기초

```python
def add_all(*args):
    # 호출부에서 넘긴 값들이 튜플로 묶인다
    print(type(args), args)
    return sum(args)

add_all(1, 2, 3)
# <class 'tuple'> (1, 2, 3)
# 반환값: 6

add_all()         # 0  (빈 튜플도 OK)
add_all(10)       # 10
```

## 2단계: `**kwargs` 기초

```python
def print_info(**kwargs):
    print(type(kwargs), kwargs)
    for key, value in kwargs.items():
        print(f"  {key} = {value}")

print_info(name="Gemini", age=2)
# <class 'dict'> {'name': 'Gemini', 'age': 2}
#   name = Gemini
#   age = 2
```

## 3단계: 둘 다 동시에

```python
def flexible(*args, **kwargs):
    print("positional:", args)
    print("keyword:",    kwargs)

flexible(1, 2, 3, name="test", value=100)
# positional: (1, 2, 3)
# keyword:    {'name': 'test', 'value': 100}
```

### 파라미터 선언 순서 (Python 문법 강제)

```python
def f(일반필수, 일반기본값=1, *args, 키워드전용, 키워드기본값=2, **kwargs):
    ...
#     ^^^^^^    ^^^^^^^^^^  ^^^^  ^^^^^^^^^^  ^^^^^^^^^^^^^  ^^^^^^^^
#     1         2           3     4(키워드전용)  5            6
```

순서는 반드시 `일반 → *args → 키워드전용 → **kwargs`. 어기면 `SyntaxError`.

---

## 방향이 뒤집힌다: 정의 vs 호출

같은 `*` 를 써도 **함수 정의부** 와 **함수 호출부** 에서 정반대 방향으로 동작한다. 이게 초보자가 가장 헷갈리는 지점.

| 위치 | 방향 | 하는 일 | 이름 |
|---|---|---|---|
| `def f(*args)` | 묶기 | 여러 값을 튜플로 **packing** | 가변 인자 수신 |
| `f(*some_list)` | 풀기 | 튜플/리스트를 여러 인자로 **unpacking** | 가변 인자 송신 |

### 직접 확인

```python
def greet(a, b, c):
    return f"{a}/{b}/{c}"

nums = [1, 2, 3]
print(greet(*nums))    # "1/2/3"   ← 리스트를 풀어서 세 인자로
                        
info = {"a": 1, "b": 2, "c": 3}
print(greet(**info))   # "1/2/3"   ← 딕셔너리를 풀어서 키워드 인자로
```

### 단계 분해

```
1단계: greet(*nums)
2단계: 내부적으로 greet(nums[0], nums[1], nums[2])  
3단계: greet(1, 2, 3) 실행
```

---

## 4단계: 데코레이터의 핵심 구조

데코레이터는 원본 함수가 어떤 인자를 받는지 모르는 상태에서 감싼다. 그래서 wrapper 는 "모든 인자를 다 받을 수 있어야" 한다.

```python
def decorator(func):
    def wrapper(*args, **kwargs):        # 모든 인자 바구니에 담고
        print(f"[call] {func.__name__}")
        result = func(*args, **kwargs)   # 그대로 풀어서 원본에 전달
        print(f"[done] {func.__name__}")
        return result
    return wrapper

@decorator
def search(region, limit=10, sort="asc"):
    return f"{region} {limit}개 ({sort})"

search("강남", limit=20, sort="desc")
# [call] search
# [done] search
# '강남 20개 (desc)'
```

`wrapper` 내부 흐름:
```
1. 호출부:     search("강남", limit=20, sort="desc")
2. 실제 호출:  wrapper("강남", limit=20, sort="desc")   ← 데코레이터로 대체됨
3. 바구니:     args = ("강남",)   kwargs = {"limit": 20, "sort": "desc"}
4. 풀어 전달:  func("강남", limit=20, sort="desc")
```

*args / **kwargs 없이 `def wrapper()` 로 만들면 `TypeError: wrapper() takes 0 positional arguments but 1 was given` 발생.

---

## AI Agent 개발에서 자주 등장하는 패턴

### 패턴 1: LangChain 콜백 / 리트라이 데코레이터

```python
import asyncio
from functools import wraps

def with_retry(max_attempts: int = 3):
    def deco(func):
        @wraps(func)                          # [[@데코레이터]] 참고
        async def wrapper(*args, **kwargs):   # async 도 가변 인자 동일
            last_exc = None
            for i in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    last_exc = e
                    await asyncio.sleep(2 ** i)
            raise last_exc
        return wrapper
    return deco

@with_retry(max_attempts=3)
async def call_llm(prompt: str, model: str = "gpt-4o"):
    ...
```

`*args, **kwargs` 덕분에 `call_llm` 이 어떤 인자를 받든 재시도 로직이 그대로 동작한다.

### 패턴 2: LangGraph 노드 설정 전달

```python
from langgraph.graph import StateGraph

def build_graph(**node_configs):
    graph = StateGraph(AgentState)
    for name, cfg in node_configs.items():
        graph.add_node(name, make_node(**cfg))    # 설정 풀어서 전달
    return graph

# 호출 시 유연하게
graph = build_graph(
    planner={"model": "gpt-4o", "temperature": 0},
    executor={"model": "gpt-4o-mini", "tools": [search, calc]},
)
```

### 패턴 3: LLM 응답의 자유 필드 받아 포워딩

```python
def log_to_langfuse(trace_name: str, **metadata):
    # metadata 에 뭐가 들어올지 모름. session_id, user_id, tags, ...
    langfuse.trace(name=trace_name, metadata=metadata)

log_to_langfuse(
    "agent_run",
    session_id="abc",
    user_id="u123",
    tools_used=["search", "calc"],
)
```

---

## 타입 힌트 붙이기

`*args`, `**kwargs` 에도 타입 힌트를 달 수 있다. 단, **원소 하나의 타입**을 쓴다는 점이 함정.

```python
from typing import Any

def f(*args: int, **kwargs: str) -> None:
    # args 는 tuple[int, ...] 로 해석됨 (원소가 int)
    # kwargs 는 dict[str, str] 로 해석됨 (값이 str)
    ...

f(1, 2, 3, name="a", city="b")   # OK
f(1, "bad")                       # mypy 가 잡아냄
```

타입이 제각각이면 `Any` 를 쓴다. 자세한 내용은 [[함수의 타입 힌트]].

---

## 주의할 함정 3가지

### 함정 1: 위치 인자 뒤에 `*args` 는 있어도 "필수 키워드 전용" 만들기 가능

```python
def connect(host, *, port, timeout=30):
    # `*` 뒤는 무조건 키워드로만 전달 가능
    ...

connect("localhost", 8080)              # ❌ TypeError
connect("localhost", port=8080)         # ✅
```

`*args` 대신 그냥 `*` 만 적으면 "여기 뒤부터는 키워드 전용" 표시.

### 함정 2: `**kwargs` 로 받은 건 원본 딕셔너리가 아니라 복사본

```python
def f(**kwargs):
    kwargs["new"] = 1   # 함수 내부에서만 변경됨

d = {"a": 1}
f(**d)
print(d)   # {'a': 1}  ← 원본은 그대로
```

`**d` 로 풀 때 Python 이 새 딕셔너리를 만들어서 넘긴다.

### 함정 3: 데코레이터에서 `**kwargs` 남발 → 시그니처 실종

```python
@decorator
def search(region, limit=10): ...

# IDE 에서 search 의 시그니처가 wrapper 의 (*args, **kwargs) 로 보임
# → 자동완성, 문서화 망가짐
# → 해결: functools.wraps + inspect 로 시그니처 보존
```

`@wraps(func)` 를 꼭 붙여야 `search.__name__`, `search.__doc__`, `search.__wrapped__` 가 원본을 유지한다. [[@데코레이터]] 에서 이어짐.

---

## 요약표

| 문법 | 데이터 형태 | 사용 방향 | 대표 상황 |
|---|---|---|---|
| `*args` | Tuple `(1, 2, ...)` | 정의: 위치 인자 묶기 | 데코레이터 wrapper, 합계 함수 |
| `**kwargs` | Dict `{"a": 1, ...}` | 정의: 키워드 인자 묶기 | 설정 dict 수신, LLM 메타데이터 |
| `*리스트` | 풀어서 여러 인자로 | 호출: unpacking | `f(*[1,2,3])` |
| `**딕셔너리` | 풀어서 키워드 인자로 | 호출: unpacking | `f(**config)` |

---

## 관련 문서

- [[@데코레이터]]  (`*args, **kwargs` 의 가장 큰 사용처)
- [[클로저(Closure)란]]  (wrapper 안에서 func, args 를 기억하는 원리)
- [[함수의 타입 힌트]]  (`*args: int` 같은 힌트)
- [[함수의 반환값을 파라미터로 넣기]]  (unpacking 과 대비되는 인자 전달 방식)

---

## 한마디 재요약

`*`/`**` 는 **"묶고 풀기"** 두 얼굴을 가진 문법이다. 정의부에서는 무제한 인자를 담는 바구니, 호출부에서는 컬렉션을 풀어 흩뿌리는 도구. Agent 코드에서 데코레이터·콜백·설정 전달의 공용어.
