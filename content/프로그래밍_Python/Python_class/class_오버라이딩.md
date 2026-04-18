---
생성날짜:
- 2026-01-28 02:06
마지막수정날짜:
- 2026-01-28-수요일 02:06
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## 오버라이딩(Overriding) 기본 컨셉

**내가 클래스가 부모 클래스의 방법을 재정의하는 것**

```python
class Parent:
    def greet(self):
        return "안녕하세요"

class Child(Parent):
    def greet(self):  # ← 오버라이딩 (재정의)
        return "안녕"

p = Parent()
c = Child()
print(p.greet())  # "안녕하세요"
print(c.greet())  # "안녕" ← 부모 메서드 무시, 자식 메서드 사용

```

## 나쁜 구현

## 1. 빈 구현으로 오버라이드 (❌ 나쁨)
**부모 기능을 아예 없애버림**
**왜 나쁜가:**
- 부모 클래스의 처리(결제 처리)를 무시합니다.
- 사용자는 결제가 될 것이라고 예상했지만 실제로는 마감되지 않을 것입니다.
- 버그발생!
```python
class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        """결제를 처리합니다"""
        pass

class CreditCardPayment(PaymentStrategy):
    def process(self, amount):
        pass  # ← 빈 구현! 아무것도 안 함 (나쁨!)

# 사용
payment = CreditCardPayment()
result = payment.process(100)  # None 반환, 아무 일도 안 일어남!
```

**올바른 방식:**
```python
class CreditCardPayment(PaymentStrategy):
    def process(self, amount):
        # 실제 로직 구현
        return f"신용카드 {amount}원 결제 완료"
```

## ## 2. 약한 전제조건 / 강한 후속조건 (❌ 나쁨)

## 전제조건(Precondition): 함수 실행 전 요구사항
- **원칙 위반:** 자식은 부모보다 **더 관대**해야 함 (덜 까다로워야 함)
```python
class FileProcessor:
    def read_file(self, filepath):
        """파일을 읽습니다. filepath는 문자열이어야 함"""
        if not isinstance(filepath, str):
            raise TypeError("filepath는 문자열이어야 함")
        with open(filepath) as f:
            return f.read()

# ❌ 나쁜 자식: 전제조건을 더 강하게 만듦
class StrictFileProcessor(FileProcessor):
    def read_file(self, filepath):
        # 부모: str만 요구
        # 자식: str + 확장자까지 요구 (더 까다로움!)
        if not filepath.endswith('.txt'):
            raise ValueError("txt 파일만 가능")  # ← 추가 제약!
        return super().read_file(filepath)

# 문제 발생
processor = FileProcessor()
processor.read_file("data.json")  # ✅ 작동

strict = StrictFileProcessor()
strict.read_file("data.json")  # ❌ 에러! (자식이 더 까다로움)

```

## 후속조건(Postcondition): 함수 실행 후 보장 사항
- **원칙 위반:** 자식은 부모보다 **더 강한 보장**을 해야 함
```python
class DataFetcher:
    def fetch(self):
        """데이터를 반환합니다. 반환값: list"""
        return [1, 2, 3]

# ❌ 나쁜 자식: 후속조건을 약하게 만듦
class WeakDataFetcher(DataFetcher):
    def fetch(self):
        # 부모: list 반환 보장
        # 자식: None 반환 가능 (보장 약화!)
        if random.random() < 0.5:
            return None  # ← 부모 약속 위반!
        return [1, 2, 3]

# 문제 발생
fetcher = DataFetcher()
data = fetcher.fetch()
print(len(data))  # ✅ 3 (list 보장)

weak = WeakDataFetcher()
data = weak.fetch()
print(len(data))  # ❌ TypeError! (None일 수 있음)
```

## 3. 예외를 던지는 오버라이드 (❌ 나쁨)
**왜 나쁜가:**
- 부모를 사용하던 코드에 자식을 넣으면 갑자기 에러 발생
- **리스코프 치환 원칙(LSP) 위반**
```python
class Calculator:
    def add(self, a, b):
        """두 수를 더합니다. 예외 없음"""
        return a + b

# ❌ 나쁜 자식: 갑자기 예외 던짐
class StrictCalculator(Calculator):
    def add(self, a, b):
        if a < 0 or b < 0:
            raise ValueError("음수는 안 됨!")  # ← 부모는 안 던지는 예외!
        return a + b

# 문제 발생
calc = Calculator()
result = calc.add(-5, 10)  # ✅ 5 (작동)

strict = StrictCalculator()
result = strict.add(-5, 10)  # ❌ ValueError! (예상 못 한 에러)
```

## 올바른 오버라이딩 패턴

## 1. 기능 확장 (✅ 좋음)
```python
class Logger:
    def log(self, message):
        print(message)

class FileLogger(Logger):
    def log(self, message):
        super().log(message)  # 부모 기능 유지
        with open("log.txt", "a") as f:
            f.write(message + "\n")  # 추가 기능
```

## 2. 구체적 구현 (✅ 좋음)
```python
class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        pass

class CreditCardPayment(PaymentStrategy):
    def process(self, amount):
        # 구체적 구현 (빈 구현 아님!)
        return f"신용카드 {amount}원 결제"
```

## 3. 더 관대한 입력 허용 (✅ 좋음)
```python
class Greeter:
    def greet(self, name: str):
        return f"안녕, {name}"

class FlexibleGreeter(Greeter):
    def greet(self, name):  # str뿐만 아니라 모든 타입 허용
        if name is None:
            name = "손님"  # None도 처리
        return f"안녕, {name}"
```

## 실무 예시

## ❌ 나쁜 예: FastAPI
```python
class BaseAPI:
    def get_data(self):
        return {"status": "ok", "data": [1, 2, 3]}

class BrokenAPI(BaseAPI):
    def get_data(self):
        pass  # ← 빈 구현! None 반환
        
@app.get("/data")
def endpoint():
    api = BrokenAPI()
    return api.get_data()  # ❌ None 반환, 클라이언트 에러!
```


## 요약

|나쁜 패턴|설명|예시|
|---|---|---|
|**빈 구현**|`pass`만 쓰고 아무것도 안 함|`def process(self): pass` [](https://rebro.kr/134)​|
|**약한 전제조건**|자식이 더 까다로운 입력 요구|부모: str / 자식: `.txt`만|
|**강한 후속조건**|자식이 더 약한 결과 반환|부모: list / 자식: None 가능|
|**예외 추가**|부모는 안 던지는 예외 던짐|부모: 정상 / 자식: ValueError [](https://blog.naver.com/star7sss/222290939578)​|

**정리**:

- 오버라이딩 = 자식이 부모 메서드 재정의[](https://wikidocs.net/234363)
    
- 빈 구현(`pass`) = 기능 없앰 (나쁨)[](https://rebro.kr/134)​
    
- 자식은 부모보다 **더 관대하게 입력**, **더 강하게 보장** 해야 함
    
- 예외 추가 = 사용자 예상 깸 (나쁨)