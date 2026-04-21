---
생성날짜:
- 2026-01-27 22:45
마지막수정날짜:
- 2026-01-27-화요일 22:45
tags:
- Python
- 함수
- 클로저
- 스코프
- 데코레이터
- LangGraph
- AI_Agent
별칭:
- closure
- LEGB
- enclosing scope
- nonlocal
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python/함수
Project:
---

## 한마디 요약

**클로저(closure, 폐쇄)** 는 "안쪽 함수가 자신을 감싼 바깥 함수의 변수를 기억하는 현상". 바깥 함수가 끝나도 그 변수들이 살아남아 안쪽 함수에 묶여 있다. [[@데코레이터]] 의 작동 원리이자, 팩토리 함수·콜백·LangGraph 노드 주입의 밑바닥 메커니즘. AI Agent 코드에서 "설정값을 미리 기억시킨 함수" 를 만드는 거의 모든 패턴이 클로저로 돌아간다.

---

## 1. 클로저란?

**"안쪽 함수가 바깥 함수의 변수를 기억하는 현상"**

- `return` 없어도 변수를 사용만 하면 Python 이 자동으로 기억
- 메모리에 "변수 환경(cell)" 을 저장
- 이게 데코레이터·팩토리·콜백이 돌아가는 핵심 원리

```python
def outer(x):
    # x = 10 이 여기 있음
    
    def inner():
        print(x)    # ← outer 의 x 를 기억해서 접근 가능
    
    return inner    # inner 함수 자체를 반환

func = outer(10)
# outer 는 실행 끝났지만...
func()   # 10 출력! ← x=10 을 아직도 기억
```

### 포함 관계

```
함수 스코프 시스템 (LEGB)
├─ L (Local)        현재 함수 안
├─ E (Enclosing)    바깥 함수                 ← 클로저가 이걸 참조
├─ G (Global)       모듈 전역
└─ B (Built-in)     Python 내장
```

클로저는 LEGB 중 **E(Enclosing)** 스코프를 "끌고 다니는" 현상이라고 보면 된다.

---

## 2. 어떻게 기억하나

### Python 이 자동으로 "환경" 을 저장

```python
def retry(max_attempts=3):
    # max_attempts = 3 (여기서 선언)
    
    def decorator(func):
        # func = call_api (여기서 받음)
        
        def wrapper(*args, **kwargs):
            # 여기서 max_attempts 와 func 사용 가능
            for attempt in range(max_attempts):
                return func(*args, **kwargs)
        
        return wrapper
    return decorator
```

### 내부 메모리 구조 확인

```python
# wrapper 함수가 반환될 때 함께 저장되는 것들
print(wrapper.__closure__)
# → (<cell at 0x...: int object at ...>, <cell at 0x...: function ...>)
#   max_attempts=3, func=call_api 가 cell 로 박혀 있음

# cell 에 실제로 들어있는 값 꺼내기
print([cell.cell_contents for cell in wrapper.__closure__])
# → [3, <function call_api at 0x...>]
```

`__closure__` 속성이 "기억" 의 정체. 각 cell 하나가 바깥 변수 하나에 대응.

---

## 3. 코드 작성 vs 코드 실행 (클로저가 발동되는 시점)

```python
# 이건 그냥 "코드 작성" (파일에 적기만 함)
def outer(x):
    def inner():
        print(x)
    return inner

# 아직 아무 일도 안 일어남!
# outer 도 실행 안 함, inner 도 정의 안 됨
```

### inner 가 실제로 정의되는 시점

```python
func = outer(5)    # ← 바로 이 순간
```

### 단계 분해

```
1단계: def outer(x): ...           # 함수 "작성" 만 함 (실행 X)
2단계: def inner(): print(x)       # 아직 안 읽음
3단계: return inner

--- 여기까지는 아무것도 실행 안 됨 ---

4단계: func = outer(5)
       → x=5 로 outer 내부 실행
       → inner 정의됨 (여기서!)
       → 클로저 생성 (x=5 저장)
       → inner 반환

5단계: func()                      # 마침내 inner 실행 → 5 출력
```

---

## 4. 일반 변수 vs 클로저 변수

### 일반 변수 (클로저 아님)

```python
def outer():
    x = 10   # 지역 변수
    print(x)
    # outer 끝나면 x 사라짐

outer()   # 10 출력
# x 는 이제 메모리에서 삭제됨
```

### 클로저 변수

```python
def outer():
    x = 10
    
    def inner():
        print(x)    # ← x 참조! 클로저 형성
    
    return inner
    # outer 끝나도 x 는 inner 에 묶여서 유지됨

func = outer()
# x 는 아직 살아있음 (inner 에 묶임)

func()   # 10 출력 (x 사용 가능)
```

---

## 5. `nonlocal`: 바깥 변수를 "수정" 하려면

클로저는 바깥 변수를 **읽기**는 자유롭지만, 대입(`=`) 으로 바꾸려면 `nonlocal` 선언이 필요하다.

```python
def make_counter():
    count = 0
    
    def counter():
        nonlocal count       # "바깥 count 를 건드릴게"
        count += 1           # 선언 없으면 새 지역 변수로 오해함
        return count
    
    return counter

c1 = make_counter()
c2 = make_counter()
print(c1(), c1(), c1())   # 1 2 3
print(c2())               # 1   ← 독립적 상태
```

### 비교표

| 키워드 | 범위 | 사용 목적 |
|---|---|---|
| (없음) | 지역 | 읽기는 LEGB 탐색, 쓰기는 Local 변수 새로 만듦 |
| `nonlocal` | Enclosing | 바깥 함수 변수를 수정 |
| `global` | Global | 모듈 전역 변수를 수정 |

---

## 6. 핵심 오해 정정 (클로저는 "정의 시점" 이 아니라 "실행 결과")

```python
def outer(x):          # 1. x=10 받음
    def inner():       # 2. inner 함수 정의 (실행 X)
        print(x)       # 3. "이 함수는 x 를 쓸 거야" Python 이 감지
    return inner       # 4. inner + x 정보를 함께 반환
```

- ❌ "inner 가 x 를 사용했으니, 그 이후부터 기억"
- ✅ "inner 가 x 를 **사용할 예정** 이니, 미리 묶어둠"

Python 내부의 판단 흐름:
```
"inner 함수가 x 를 참조하네?"
"x 는 outer 의 지역 변수인데..."
"outer 끝나면 x 사라지겠네?"
→ "x 를 inner 에 묶어서(bind) 기억시키자!"
```

---

## 7. 복잡한 개념 복습: 설정값 고정 팩토리

```python
def make_multiplier(factor):   # factor = 2
    def multiply(value):       # 이 함수를 만들기만 함
        return factor * value
    return multiply            # ← 괄호 없음! 실행 X, 함수 자체 반환

step1 = make_multiplier(2)
# step1 = multiply 함수 (factor=2 를 기억)
# 계산 0%, 준비 100%

step2 = step1(10)
# step1(10) = multiply(10) → factor(2) * value(10) = 20
```

### 타입 확인으로 증명

```python
step1 = make_multiplier(2)

print(type(step1))        # <class 'function'> ← 함수!
print(step1.__name__)     # 'multiply'
print(step1.__closure__)  # (<cell at 0x...: int object at 0x...>,)
print(step1.__closure__[0].cell_contents)   # 2   ← factor
```

`step1` 은 숫자가 아니라 **"factor=2 를 기억하는 함수 객체"**.

### 올바른 이해

```python
step1 = make_multiplier(2)
# "factor=2 를 기억하는 multiply 함수를 받았다"
# 아직 아무것도 계산 안 함!

step2 = step1(10)
# "step1 함수 실행 (value=10 전달)"
# multiply(10) → 2 * 10 = 20
```

**핵심 3가지**:
1. `step1` 은 **값이 아니라 함수**
2. `factor` 는 **절대 안 바뀜** (클로저로 고정)
3. `value` 는 **step1(10) 호출 시 전달**됨

---

## 8. `*args`, `**kwargs` 와의 결합 (데코레이터 자연사)

클로저로 바깥 변수를 기억하고, `*args/**kwargs` 로 호출 시 전달된 인자를 받는다. 이 조합이 데코레이터의 표준형. [[함수(별args, 별별kwargs)]]

```python
def decorator(func):
    def wrapper(*args, **kwargs):      # 호출 인자 유연 수신
        # func 는 클로저로 기억 (바깥에서 묶임)
        print(f"[call] {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

**실행 흐름**:
```
1. call_api("강남", limit=20) 호출
    ↓
2. 실제로는 wrapper("강남", limit=20) 실행 (데코레이터 때문에)
    ↓
3. wrapper 안에서:
    func         = call_api         (클로저로 기억)
    args         = ("강남",)         (위치 인자 → 튜플)
    kwargs       = {"limit": 20}    (키워드 인자 → 딕셔너리)
    ↓
4. func(*args, **kwargs) 실행
    = call_api("강남", limit=20)    (원본 함수에 그대로 전달)
```

---

## 9. 실무 활용

### 9-1. 설정 값 기억 (로거 팩토리)

```python
def create_logger(log_level):
    def log(message):
        if log_level == "DEBUG":
            print(f"[DEBUG] {message}")
        elif log_level == "INFO":
            print(f"[INFO] {message}")
    return log

debug_log = create_logger("DEBUG")   # 아직 실행 안 됨
info_log  = create_logger("INFO")

debug_log("테스트")   # [DEBUG] 테스트
info_log("테스트")    # [INFO] 테스트
```

### 9-2. 독립 카운터

```python
def make_counter():
    count = 0
    
    def counter():
        nonlocal count
        count += 1
        return count
    
    return counter

c1, c2 = make_counter(), make_counter()
print(c1())   # 1
print(c1())   # 2
print(c2())   # 1   ← 독립 상태
```

### 9-3. LangGraph 노드 팩토리 (AI Agent 실전)

```python
from langchain_openai import ChatOpenAI

def make_planner_node(model_name: str, system_prompt: str):
    llm = ChatOpenAI(model=model_name)    # 클로저가 기억
    
    def planner_node(state):              # 그래프가 매번 호출
        messages = [("system", system_prompt)] + state["messages"]
        response = llm.invoke(messages)
        return {"messages": [response]}
    
    return planner_node

# 그래프 구성 시 주입
graph.add_node("planner",
    make_planner_node("gpt-4o", "당신은 계획 수립 전문가..."))
graph.add_node("critic",
    make_planner_node("gpt-4o-mini", "당신은 비판적 검토자..."))
```

**포인트**: 각 노드가 자기만의 `llm`, `system_prompt` 를 클로저로 들고 다닌다. 클래스 만들 필요 없이 함수 팩토리로 해결.

### 9-4. 인스턴스 메서드가 self 를 기억하는 것도 같은 원리

```python
class RealEstateAgent:
    def __init__(self, region, api_key):
        self.region = region
        self.api_key = api_key
        
        # 내부 함수가 self 를 기억 (클로저)
        def fetch_data():
            return f"{self.region} 데이터 (키: {self.api_key})"
        
        self.fetcher = fetch_data
    
    def get_data(self):
        return self.fetcher()   # region, api_key 기억

agent = RealEstateAgent("강남", "secret123")
print(agent.get_data())   # 강남 데이터 (키: secret123)
```

메서드가 `self` 를 참조하는 것도 결국 클로저의 일반화된 형태. Agent 객체의 상태(설정, 도구 리스트, 메모리) 가 메서드에 "기억" 되는 것.

---

## 10. 클로저 vs 클래스 (언제 뭘 쓸까)

| 상황 | 클로저 | 클래스 |
|---|---|---|
| 기억할 상태가 1~2개, 동작도 1개 | ✅ | 과함 |
| 기억할 상태가 여러 개, 동작도 여러 개 | 복잡해짐 | ✅ |
| 타입 힌트/자동완성 중요 | 약함 | 강함 |
| 데코레이터 | ✅ 표준 | `__call__` 사용 시 가능 |
| LangGraph 노드 | ✅ 가볍게 | 더 큰 에이전트면 클래스 |
| 테스트 격리 | 호출마다 독립 | 인스턴스마다 독립 |

경험칙: "함수 한 개에 설정만 꽂으면 되는 상황" 이면 클로저, "여러 메서드가 같은 상태를 공유해야 한다" 면 클래스.

---

## 11. 함정: 반복문 안에서 lambda 로 클로저 생성하기

유명한 "late binding" 함정.

```python
# ❌ 기대와 다르게 동작
funcs = []
for i in range(3):
    funcs.append(lambda: i)

print([f() for f in funcs])   # [2, 2, 2]   ← 왜?!

# 이유: 각 lambda 가 "i 라는 변수" 를 참조할 뿐,
#       반복이 끝났을 때 i 는 2 로 고정
```

### 해결법 2가지

```python
# 방법 1: 기본 인자로 "지금 값" 을 묶어두기
funcs = [lambda i=i: i for i in range(3)]
print([f() for f in funcs])   # [0, 1, 2] ✅

# 방법 2: 팩토리 함수로 새 스코프 만들기
def make(i):
    return lambda: i

funcs = [make(i) for i in range(3)]
print([f() for f in funcs])   # [0, 1, 2] ✅
```

클로저는 "변수 이름" 을 묶는 것이지 "변수 값" 을 복사하지 않는다는 걸 기억.

---

## 관련 문서

- [[@데코레이터]]  (클로저가 가장 자주 드러나는 형태)
- [[함수(별args, 별별kwargs)]]  (wrapper 안에서 쓰이는 가변 인자)
- [[함수의 반환값을 파라미터로 넣기]]  (함수 객체 전달 = 클로저 반환)
- [[lambda와 순회함수]]  (짧은 클로저 생성 수단)
- [[함수의 타입 힌트]]  (`Callable` 힌트로 반환된 함수 타입 명시)

---

## 한마디 재요약

클로저는 **"함수가 자기 태어난 동네의 변수를 들고 다니는 현상"**. Python 이 알아서 묶어주니 개발자는 "바깥 값을 미리 꽂아둔 함수" 를 공짜로 얻는다. 데코레이터·팩토리·Agent 노드 주입의 배후에서 늘 돌아가는 메커니즘.
