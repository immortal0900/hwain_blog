---
생성날짜:
- 2026-01-26 19:18
마지막수정날짜:
- 2026-01-26-월요일 19:18
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---


## 질문2-2: `self.`는 왜 붙이나? 언제 붙이나?

**핵심 개념**: `self` = "이 인스턴스 객체 자신"[](https://www.toptal.com/python/python-class-attributes-an-overly-thorough-guide)

## 왜 필요한가?

```python
class PaymentProcessor:
    def __init__(self, payment_method):
        # self.를 안 붙이면?
        payment_method = payment_method  # ← 로컬 변수, 함수 끝나면 사라짐
        
        # self.를 붙이면?
        self.payment_method = payment_method  # ← 인스턴스 변수, 객체에 저장됨
```

**실제 차이**:
```python
class Bad:
    def __init__(self, value):
        value = value  # 로컬 변수
    
    def get_value(self):
        return self.value  # AttributeError! (저장 안 했으니)

class Good:
    def __init__(self, value):
        self.value = value  # 인스턴스 변수
    
    def get_value(self):
        return self.value  # OK!

```


## 언제 `self.`를 붙이나?

**규칙**: "다른 메서드에서도 접근해야 하는 데이터"면 `self.` 붙임[](https://www.pythonmorsels.com/customizing-what-happens-when-you-assign-attribute/)

```python
class PaymentProcessor:
    def __init__(self, payment_method):
        # 1. 인스턴스 변수 (self. 필수)
        self.payment_method = payment_method  # ← 다른 메서드에서 사용
        
        # 2. 로컬 변수 (self. 안 붙임)
        temp_value = "초기화 중..."  # ← __init__에서만 사용
        print(temp_value)
    
    def process(self, amount):
        # 3. 인스턴스 변수 접근 (self. 필수)
        return self.payment_method.process(amount)
        
        # 4. 로컬 변수 (self. 안 붙임)
        result = "처리 완료"  # ← 이 메서드에서만 사용
        return result

```

## 실무 패턴
```python
class RealEstateAgent:
    def __init__(self, api_key: str, region: str):
        # 인스턴스 변수 (self. 붙임) - 여러 메서드에서 재사용
        self.api_key = api_key
        self.region = region
        self.client = APIClient(api_key)  # ← 다른 메서드에서 사용
        
        # 로컬 변수 (self. 안 붙임) - 초기화 시에만 사용
        welcome_msg = f"{region} 분석 시작"
        print(welcome_msg)
    
    def fetch_data(self):
        # self.client 사용 (위에서 저장했으니 접근 가능)
        data = self.client.get(f"/region/{self.region}")
        return data
```

```python
processor1 = PaymentProcessor(CreditCardPayment())
processor2 = PaymentProcessor(PayPalPayment())

# 각 인스턴스는 독립적인 self.payment_method를 가짐
processor1.payment_method  # → CreditCardPayment 객체
processor2.payment_method  # → PayPalPayment 객체

```


**결론**:

- `: PaymentStrategy` = 타입 문서화 (IDE 도움용, 실행엔 무관)[](https://realpython.com/python-property/)​
    
- `self.` = "이 객체의 속성으로 저장" (다른 메서드에서 접근 가능)[](https://www.toptal.com/python/python-class-attributes-an-overly-thorough-guide)
    
- `self.` 안 붙이면 = 로컬 변수 (함수 끝나면 사라짐)

## 메서드/속성 접근시 `self.`

```python
class LoggableMixin:
    def log(self, message):  # ← 인스턴스 메서드
        print(f"[LOG] {message}")

class Tool(LoggableMixin):
    def run(self):
        self.log("실행")  # ← self. 필수! (인스턴스 메서드 호출)
        # log("실행")  ← 에러! log는 함수가 아님
        
tool = Tool()
result=tool.run()
print(result) # 출력 `[LOG] 실행`

```