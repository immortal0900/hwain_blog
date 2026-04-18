---
생성날짜:
- 2026-01-28 20:02
마지막수정날짜:
- 2026-01-28-수요일 20:01
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## 실무 구분법

## OCP 위반 신호
## O: Open-Closed Principle (개방-폐쇄 원칙)

**"확장에는 열려있고, 수정에는 닫혀있어야 한다"**
```python
# if-elif 체인이 계속 늘어남
if type == "A":
    ...
elif type == "B":
    ...
elif type == "C":  # ← 추가할 때마다 수정
    ...
```

## DIP 위반 신호
## D: Dependency Inversion Principle (의존성 역전 원칙)

**"고수준 모듈이 저수준 모듈에 의존하지 말고, 둘 다 추상화에 의존해야 한다"**
```python
# 클래스 내부에서 직접 생성
class Service:
    def __init__(self):
        self.db = MySQLDatabase()  # ← 구체 클래스 직접 생성
```

## 실행 예시
```python
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        pass

class CreditPayment(PaymentStrategy):
    def process(self, amount):
        return f"신용카드: {amount}원"

class PayPalPayment(PaymentStrategy):
    def process(self, amount):
        return f"PayPal: {amount}원"

class KakaoPayPayment(PaymentStrategy):
    def process(self, amount):
        return f"카카오페이: {amount}원"

# OCP + DIP 둘 다 준수
class PaymentProcessor:
    def __init__(self, strategy: PaymentStrategy):  # DIP: 추상화 의존
        self.strategy = strategy
    
    def process(self, amount):
        return self.strategy.process(amount)  # OCP: 확장 가능

# 실행
print("=== 신용카드 ===")
p1 = PaymentProcessor(CreditPayment())
print(p1.process(1000))  # 신용카드: 1000원

print("\n=== PayPal ===")
p2 = PaymentProcessor(PayPalPayment())
print(p2.process(2000))  # PayPal: 2000원

print("\n=== 카카오페이 (새로 추가) ===")
p3 = PaymentProcessor(KakaoPayPayment())
print(p3.process(3000))  # 카카오페이: 3000원
# PaymentProcessor 코드 수정 없이 추가!

```
## 정리


| |**OCP**|**DIP**|
|---|---|---|
|**질문**|새 기능 추가 시?|의존 대상은?|
|**나쁨**|기존 코드 수정 [](https://steady-coding.tistory.com/378)​|구체 클래스 의존|
|**좋음**|새 클래스 추가 [](https://blog.itcode.dev/posts/2021/08/14/open-closed-principle)​|추상화 의존 + 외부 주입|
|**키워드**|확장/수정 [](https://jonghoonpark.com/2023/10/10/solid-%EC%84%A4%EA%B3%84-%EC%9B%90%EC%B9%99)​|고수준/저수준, 추상 [](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%EC%84%A4%EA%B3%84%EC%9D%98-5%EA%B0%80%EC%A7%80-%EC%9B%90%EC%B9%99-SOLID)​|
|**예시**|if-elif 제거 [](https://blog.itcode.dev/posts/2021/08/14/open-closed-principle)​|내부 생성 제거|

**둘의 관계:**

- **OCP 달성하려면 DIP 필요함** (추상화 없이는 확장 어려움)
- 하지만 **목적이 다름**: OCP는 확장성, DIP는 의존 방향
    

**쉬운 구분법:**

- "새 기능 추가 시 코드 수정하나?" → **OCP**
- "구체 클래스 직접 쓰나?" → **DIP**