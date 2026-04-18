---
생성날짜:
- 2026-01-27 22:45
마지막수정날짜:
- 2026-01-27-화요일 22:45
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## 클로저(Closure)란?

**"안쪽 함수가 바깥 함수의 변수를 기억하는 현상"**
- `return` 없어도 변수 사용만 하면 기억됨
- Python이 메모리에 변수 환경을 저장해둠
- 이게 데코레이터가 작동하는 핵심 원리!
```python
def outer(x):
    # x = 10이 여기 있음
    
    def inner():
        print(x)  # ← outer의 x를 기억해서 접근 가능!
    
    return inner  # inner 함수 자체를 반환

func = outer(10)
# outer는 실행 끝났지만...
func()  # 10 출력! ← x=10을 아직도 기억!

```

## 어떻게 기억하나?

**Python이 자동으로 "환경"을 저장**
```python
def retry(max_attempts=3):
    # max_attempts = 3 (여기서 선언)
    
    def decorator(func):
        # func = call_api (여기서 받음)
        
        def wrapper(*args, **kwargs):# ← 여기서 *args, **kwargs 받음
            # ← 여기서 max_attempts와 func 사용 가능!
            for attempt in range(max_attempts):  # max_attempts 기억
                return func(*args, **kwargs)  # func 기억, *args, **kwargs 전달
        
        return wrapper
    return decorator

```

**내부 메모리 구조**:
```python
# wrapper 함수가 반환될 때, 함께 저장되는 것들:
wrapper.__closure__  # 클로저 정보
# → max_attempts = 3
# → func = call_api 함수 객체

# 이게 "기억"의 정체!
```

## 왜 이렇게 작동하나?
**Python의 스코프 규칙 (LEGB)**:[](https://realpython.com/inner-functions-what-are-they-good-for/)

1. **L**ocal: 현재 함수 안
2. **E**nclosing: 바깥 함수 (← 클로저가 이거 참조!)
3. **G**lobal: 전역
4. **B**uilt-in: Python 내장

---
---
## *args, **kwargs란?

**"몇 개가 들어올지 모르는 인자들을 다 받는 방법"**[](https://velog.io/@clueless_coder/%ED%8C%8C%EC%9D%B4%EC%8D%AC-args-%EC%99%80-kwargs-%EA%B0%80-%EB%AD%90%EC%98%88%EC%9A%94)

- `*args` = 위치 인자들을 **튜플**로 받음[](https://pybo.kr/pybo/question/detail/30/)
- `**kwargs` = 키워드 인자들을 **딕셔너리**로 받음

**실행 흐름**:
```python
# 1. call_api("강남", limit=20) 호출
#    ↓
# 2. 실제로는 wrapper("강남", limit=20) 실행 (데코레이터 때문에)
#    ↓
# 3. wrapper 안에서:
#    args = ("강남",)       # 위치 인자 → 튜플
#    kwargs = {"limit": 20} # 키워드 인자 → 딕셔너리
#    ↓
# 4. func(*args, **kwargs) 실행
#    = call_api("강남", limit=20)  # 원본 함수에 그대로 전달
```

## 왜 *args, **kwargs를 쓰나?

**원본 함수가 어떤 인자를 받는지 모르니까!**[](https://ddanggle.gitbooks.io/interpy-kr/content/ch1-args-kwargs.html)

## 문제 상황: 인자를 고정하면?
```python
def decorator(func):
    def wrapper():  # ← 인자 없음!
        return func()
    return wrapper

@decorator
def call_api(region):  # region 인자 필요
    return f"{region} 데이터"

call_api("강남")  # ❌ TypeError: wrapper() 인자 안 받음!
```

## 해결: *args, **kwargs 사용
```python
def decorator(func):
    def wrapper(*args, **kwargs):  # ← 모든 인자 받음!
        return func(*args, **kwargs)  # 그대로 전달
    return wrapper

@decorator
def call_api(region, limit=10):
    return f"{region} 데이터 {limit}개"

call_api("강남", limit=20)  # ✅ 작동!
```

## *args 상세 설명

**위치 인자를 튜플로 받음**
```python
def print_all(*args):
    print(f"args 타입: {type(args)}")
    print(f"args 내용: {args}")
    for i, value in enumerate(args):
        print(f"  {i}번째: {value}")

print_all(1, 2, 3)
# args 타입: <class 'tuple'>
# args 내용: (1, 2, 3)
#   0번째: 1
#   1번째: 2
#   2번째: 3

print_all("A", "B")
# args 타입: <class 'tuple'>
# args 내용: ('A', 'B')
#   0번째: A
#   1번째: B
```

## **kwargs 상세 설명

**키워드 인자를 딕셔너리로 받음**
```python
def print_info(**kwargs):
    print(f"kwargs 타입: {type(kwargs)}")
    print(f"kwargs 내용: {kwargs}")
    for key, value in kwargs.items():
        print(f"  {key} = {value}")

print_info(name="김철수", age=30, city="서울")
# kwargs 타입: <class 'dict'>
# kwargs 내용: {'name': '김철수', 'age': 30, 'city': '서울'}
#   name = 김철수
#   age = 30
#   city = 서울
```

## 둘 다 사용하기
```python
def flexible_function(*args, **kwargs):
    print("위치 인자들:", args)
    print("키워드 인자들:", kwargs)

flexible_function(1, 2, 3, name="test", value=100)
# 위치 인자들: (1, 2, 3)
# 키워드 인자들: {'name': 'test', 'value': 100}
```

**결론**:[](https://junghogit.github.io/python/args-kwargs/)

- `*args, **kwargs`는 **wrapper 호출 시** 전달된 인자를 받음
- `func(*args, **kwargs)`로 **원본 함수에 그대로 전달**
- 데코레이터는 원본 함수의 인자를 몰라도 됨 (범용성)
- `*args` = 튜플 (위치 인자)[](https://velog.io/@clueless_coder/%ED%8C%8C%EC%9D%B4%EC%8D%AC-args-%EC%99%80-kwargs-%EA%B0%80-%EB%AD%90%EC%98%88%EC%9A%94)
- `**kwargs` = 딕셔너리 (키워드 인자)

---
---
함수를 사용할 시 일어나는 것들

```python
def outer(x):          # 1. x=10 받음
    def inner():       # 2. inner 함수 정의 (실행 X)
        print(x)       # 3. "이 함수는 x를 쓸 거야" Python이 감지
    return inner       # 4. inner + x 정보를 함께 반환
```
- ❌ "inner가 x를 사용했으니, 그 이후부터 기억"
- ✅ "inner가 x를 **사용할 예정**이니, 미리 기억해둠"

**Python이 하는 일**:
```python
# 2~3단계에서 Python 내부 분석:
# "inner 함수가 x를 참조하네?"
# "x는 outer의 지역 변수인데..."
# "outer 끝나면 x 사라지겠네?"
# "→ x를 inner에 묶어서(bind) 기억시키자!"

# 4단계: 반환 시
return inner  # inner 함수 + x=10 정보 묶음 반환
```

## 일반 변수 vs 클로저 변수

## 일반 변수 (클로저 아님)
```python
def outer():
    x = 10  # 지역 변수
    print(x)
    # outer 끝나면 x 사라짐

outer()  # 10 출력
# x는 이제 메모리에서 삭제됨
```

## 클로저 변수
```python
def outer():
    x = 10
    
    def inner():
        print(x)  # ← x 참조! 클로저 형성
    
    return inner
    # outer 끝나도 x는 inner에 묶여서 유지됨!

func = outer()
# x는 아직 살아있음 (inner에 묶임)

func()  # 10 출력 (x 사용 가능)
```

## 코드 작성 vs 코드 실행 -> 클로저가 발동되는 시점은 코드 실행시

```python
# 이건 그냥 "코드 작성" (파일에 적기만 함)
def outer(x):
    def inner():
        print(x)
    return inner

# 아직 아무 일도 안 일어남!
# outer도 실행 안 함, inner도 정의 안 됨
```
**inner가 실제로 정의되는 시점:**
```python
func = outer(5)  # ← 바로 이 순간!
```

## 단계별 실행
```python
def outer(x):          # 1. outer 함수 "작성"만 함 (실행 X)
    def inner():       # 2. 아직 안 읽음
        print(x)       
    return inner

# ===== 여기까지는 아무것도 실행 안 됨 =====

func = outer(5)        # 3. outer(5) 실행!
                       #    → x=5로 outer 내부 실행
                       #    → inner 정의됨 (여기서!)
                       #    → 클로저 생성 (x=5 저장)
                       #    → inner 반환

func()                 # 4. inner 실행
```




---
---
# 복잡한 개념_함수에 인자를 넣고 실행해서 함수 내부에 있는 함수를 반환 
```python
def make_multiplier(factor):  # factor = 2
    def multiply(value):      # 이 함수를 만들기만 함
        return factor * value
    return multiply           # ← 괄호 없음! 실행 X, 함수 자체 반환

step1 = make_multiplier(2)
# step1 = multiply 함수 (아직 실행 안 함!)
# "factor=2를 기억하는 multiply 함수"를 담음
step2 = step1(10)
#
```
- Step 1에서는 multiply **실행 안 됨** (함수만 반환)
- Step 2에서 **처음으로** multiply 실행됨

**step1은 이제 이런 함수를 담고 있음**:
```python
# step1이 가리키는 함수 (개념적으로)
def step1(value):  # 원래 이름은 multiply
    return 2 * value  # factor=2로 고정됨
```

```python
step1 = make_multiplier(2)
# step1 = "2를 곱하는 기능을 가진 함수"
# 계산 안 함, value도 필요 없음
```

## 타입 확인으로 증명(변수가 함수 내부의 함수를 받았다는 것)
```python
def make_multiplier(factor):
    def multiply(value):
        return factor * value
    return multiply

step1 = make_multiplier(2)

print(type(step1))        # <class 'function'> ← 함수!
print(step1)              # <function multiply at 0x...>
print(step1.__name__)     # 'multiply'

# step1은 숫자가 아니라 "함수 객체"
```

## 네 오해 정정

**❌ 잘못된 이해:**
```python
step1 = make_multiplier(2)
# "2 * value를 계산했다" → 틀림!
# value가 없으니 2가 나온다? → 틀림!

step2 = step1(10)
# "factor를 10으로 바꾼다" → 틀림!
```

**✅ 올바른 이해:**
```python
step1 = make_multiplier(2)
# "factor=2를 기억하는 multiply 함수를 받았다"
# 아직 아무것도 계산 안 함!

step2 = step1(10)
# "step1 함수를 실행 (value=10 전달)"
# multiply(10) 실행 → 2 * 10 → 20


step2 = step1(10)
# step1 함수 실행 (factor=2 고정, value=10 새로 받음)
# multiply(value=10) 실행
# factor(2) * value(10) = 20

```

## 계산기 비유
```python
# 계산기 공장
def make_multiplier(factor):
    def multiply(value):
        return factor * value
    return multiply

# "2를 곱하는 계산기" 주문
calculator = make_multiplier(2)
# 계산기를 받았지만 아직 버튼 안 누름!

# 계산기에 10 입력
result = calculator(10)
# 계산기: "내 곱셈 값은 2니까... 10 × 2 = 20"
# 결과: 20
```

## 메모리 관점으로 보기
```python
# 메모리 상태 추적
def make_multiplier(factor):
    # factor=2가 메모리 어딘가에 저장됨
    
    def multiply(value):
        # 이 함수는 factor가 있는 메모리 위치를 기억
        return factor * value
    
    return multiply  # multiply + factor 위치 정보 반환

step1 = make_multiplier(2)
# step1 = {
#   함수 코드: multiply,
#   기억하는 변수: factor=2 (메모리 0x1234 주소)
# }

step2 = step1(10)
# step1 실행:
#   1. value=10 받음
#   2. factor 주소(0x1234) 가서 2 가져옴
#   3. 2 * 10 = 20 반환
```

## 최종 정리
```python
def make_multiplier(factor):
    def multiply(value):
        return factor * value
    return multiply

# 1단계
step1 = make_multiplier(2)
# step1 = multiply 함수 (factor=2 기억)
# 계산 0%, 준비 100%

# 2단계  
step2 = step1(10)
# step1(10) = multiply(10)
# factor=2, value=10
# 2 * 10 = 20
# 계산 100%, 결과: 20
```
**핵심 3가지:**

1. `step1`은 **값이 아니라 함수**
2. factor는 **절대 안 바뀜** (클로저로 고정)
3. value는 **step1(10) 호출 시 전달**됨


## 실무 활용

## 1. 설정 값 기억
```python
def create_logger(log_level):
    def log(message):
        if log_level == "DEBUG":
            print(f"[DEBUG] {message}")
        elif log_level == "INFO":
            print(f"[INFO] {message}")
    return log

debug_log = create_logger("DEBUG") # -> 아직 실행된 상태 아님 def create_logger()는 log()를 반환함 
# 근데 log_level 인자가 "DEBUG" 인 상태의 log()를 반환함
info_log = create_logger("INFO")

debug_log("테스트")  # [DEBUG] 테스트
info_log("테스트")   # [INFO] 테스트

```

## 2. 카운터 만들기
```python
def make_counter():
    count = 0  # ← 이 변수를 기억!
    
    def counter():
        nonlocal count  # 바깥 변수 수정 가능
        count += 1
        return count
    
    return counter

counter1 = make_counter()
counter2 = make_counter()

print(counter1())  # 1
print(counter1())  # 2
print(counter2())  # 1 (독립적!)
```

## 3. LangGraph 코드_인스턴스 변수 기억하는 것도 클로저임
```python
class RealEstateAgent:
    def __init__(self, region, api_key):
        self.region = region  # ← 인스턴스 변수로 저장
        self.api_key = api_key
        
        # 내부 함수가 self.region, self.api_key 기억
        def fetch_data():
            return f"{self.region}의 데이터 (키: {self.api_key})"
        
        self.fetcher = fetch_data
    
    def get_data(self):
        return self.fetcher()  # region, api_key 기억!

agent = RealEstateAgent("강남", "secret123")
print(agent.get_data())  # 강남의 데이터 (키: secret123)

```


