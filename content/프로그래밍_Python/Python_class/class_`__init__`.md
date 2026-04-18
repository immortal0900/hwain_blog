---
생성날짜:
- 2026-01-27 00:24
마지막수정날짜:
- 2026-01-27-화요일 00:23
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
# **init** 있는 클래스 vs 없는 클래스

**핵심**: `__init__`은 **객체 생성 시 자동 실행되는 초기화 메서드**. 없어도 클래스는 작동하지만, 속성 설정을 수동으로 해야 함.[](https://www.geeksforgeeks.org/python/__init__-in-python/)

## **init** 없는 클래스
```python
class Person:
    pass  # __init__ 없음

# 사용
person1 = Person()  # ✅ 객체 생성은 됨
person1.name = "김철수"  # 수동으로 속성 추가
person1.age = 30       # 수동으로 속성 추가

person2 = Person()
person2.name = "이영희"
person2.age = 25
```

**문제점**:[](http://www.w3schools.com/PYTHON/python_class_init.asp)​
- 객체마다 속성 일일이 설정해야 함 → 반복 작업
- 실수로 속성 안 넣으면 → `AttributeError`
- 필수 데이터 강제 불가 (나중에 추가하면 되니까)


## **init** 있는 클래스
```python
class Person:
    def __init__(self, name, age):  # 초기화 메서드
        self.name = name
        self.age = age

# 사용
person1 = Person("김철수", 30)  # ✅ 생성과 동시에 초기화
person2 = Person("이영희", 25)  # 간결함!

# name, age 안 넣으면?
person3 = Person()  # ❌ TypeError: 필수 인자 누락!

```
**장점**:[](https://www.stratascratch.com/blog/what-is-the-purpose-of-__init__-in-python/)
- 객체 생성 시 필수 데이터 강제
- 코드 중복 제거
- 일관된 초기화 보장

## 실제 차이 비교

## 시나리오: 결제 클래스
```python
# Good: __init__ 있음
class PaymentProcessor:
    def __init__(self, payment_method, api_key):
        self.payment_method = payment_method
        self.api_key = api_key

# 생성 시 강제로 필수 데이터 받음
processor1 = PaymentProcessor(CreditCardPayment(), "secret123")
processor1.process(100)  # ✅ 안전!



# Bad: __init__ 없음
class PaymentProcessor:
    pass

# 사용할 때마다 수동 설정
processor1 = PaymentProcessor()
processor1.payment_method = CreditCardPayment()  # 깜빡하면 끝
processor1.api_key = "secret123"                 # 이것도 깜빡하면 끝

processor1.process(100)  # AttributeError 위험!
```

---

## __init__이 하는 일
```python
class Car:
    def __init__(self, brand, color):
        # 1. 객체 생성되자마자 자동 실행 [web:46][web:48]
        self.brand = brand
        self.color = color
        
        # 2. 초기 계산/설정도 가능
        self.mileage = 0
        self.is_running = False
        
        # 3. 초기화 시 로직 실행
        print(f"{brand} {color} 차량 등록 완료")

# 호출
car = Car("Tesla", "Red")  # ← __init__ 자동 실행됨!
# 출력: "Tesla Red 차량 등록 완료"
```

## **init** 없으면 무슨 일이?

**Python은 자동으로 기본 `__init__` 제공**
**단, 기본 __init__은 아무것도 안 함** → 속성 초기화 X
```python
class Empty:
    pass

# 위 코드는 실제로는 이렇게 작동:
class Empty:
    def __init__(self):  # 자동 생성됨 (보이지 않음)
        pass

empty = Empty()  # ✅ 작동함! 기본 __init__ 호출됨

```


## 실무 패턴

## 간단한 클래스 (데이터만)
```python
class Point:
    pass

# 이 정도면 __init__ 없어도 OK
point = Point()
point.x = 10
point.y = 20
```

## 복잡한 클래스 (로직 + 데이터)
```python
class DatabaseConnection:
    def __init__(self, host, port, username, password):
        # 초기화 시 연결 생성
        self.host = host
        self.port = port
        self.connection = self._create_connection(username, password)
        
    def _create_connection(self, username, password):
        # 복잡한 초기화 로직
        return f"Connected to {self.host}:{self.port}"

# __init__ 필수! 연결 설정 자동화
db = DatabaseConnection("localhost", 5432, "admin", "pw123")

```

예시:
```python
class ResearchAgent:
    def __init__(self, llm: ChatOpenAI, tools: list):
        self.graph = self._build_graph()
    
    def _build_graph(self):
        builder = StateGraph(...)
        # 노드 추가, 엣지 연결 등
        return builder.compile()  # ← compile()이 핵심!
    
    def invoke(self, input_data):
        # self.graph가 이미 invoke()를 가지고 있으니 위임
        return self.graph.invoke(input_data)

# 사용
agent = ResearchAgent(llm, tools)
agent.invoke({"query": "부동산 분석"})  

```

