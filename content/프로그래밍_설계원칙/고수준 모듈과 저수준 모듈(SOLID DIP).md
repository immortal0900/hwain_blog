---
생성날짜:
- 2026-01-28 04:35
마지막수정날짜:
- 2026-01-28-수요일 04:34
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
- **고수준 모듈**: 비즈니스 로직 (예: 결제 처리)
- **저수준 모듈**: 구체적 구현 (예: 신용카드 API, PayPal API)
- **추상화**: 인터페이스/추상 클래스

# DIP (Dependency Inversion Principle) - 의존성 역전 원칙
## 핵심 개념

**"고수준 모듈은 저수준 모듈에 의존하지 말고, 둘 다 추상화에 의존해야 한다"**[](https://pearlluck.tistory.com/760)

## 나쁜 예시 (DIP 위반)

## ❌ 클래스 내부에서 직접 객체 생성
**문제점**:[](https://velog.io/@chobe1/%ED%8C%8C%EC%9D%B4%EC%8D%AC-Dependency-Injection-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85)

1. PayPal로 바꾸려면? → `PaymentService` 코드 수정 필요
2. 테스트 어려움 (가짜 결제로 테스트 불가)
3. `PaymentService`가 `CreditCardPayment`에 강하게 결합
```python
class PaymentService:
    def __init__(self):
        # ❌ 내부에서 직접 생성 (나쁨!)
        self.payment = CreditCardPayment()
    
    def process(self, amount):
        return self.payment.process(amount)

# 실행
service = PaymentService()
result = service.process(100)
print(result)  # 신용카드 결제 100원
```

## ❌ 구체 클래스에 직접 의존
```python
class OrderProcessor:
    def __init__(self):
        # ❌ 구체 클래스에 의존
        self.db = MySQLDatabase()  # MySQL에 강하게 결합
        self.notifier = EmailNotifier()  # Email에 강하게 결합
    
    def process_order(self, order):
        self.db.save(order)
        self.notifier.send(f"주문 완료: {order}")

# PostgreSQL로 바꾸고 싶다면?
# → OrderProcessor 코드 수정해야 함!
```

## ❌ 하드코딩된 의존성
```python
class ReportGenerator:
    def generate(self):
        # ❌ 하드코딩
        db = MySQLDatabase("localhost", "root", "password")
        data = db.query("SELECT * FROM reports")
        return data

# 테스트 환경에서는? → 코드 수정해야 함
# 다른 DB 쓰고 싶다면? → 코드 수정해야 함
```

## 좋은 예시 (DIP + 의존성 주입)

## ✅ 추상화에 의존 + 외부 주입
```python
from abc import ABC, abstractmethod

# 1. 추상화 정의
class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        pass

# 2. 구체 클래스들 (저수준)
class CreditCardPayment(PaymentStrategy):
    def process(self, amount):
        return f"신용카드 결제: {amount}원"

class PayPalPayment(PaymentStrategy):
    def process(self, amount):
        return f"PayPal 결제: {amount}원"

class KakaoPayPayment(PaymentStrategy):
    def process(self, amount):
        return f"카카오페이 결제: {amount}원"

# 3. 고수준 모듈 (의존성 주입)
class PaymentService:
    def __init__(self, payment: PaymentStrategy):  # ← 외부에서 주입!
        self.payment = payment
    
    def process(self, amount):
        return self.payment.process(amount)

# 실행
print("=== 신용카드 ===")
service1 = PaymentService(CreditCardPayment())
print(service1.process(100))  # 신용카드 결제: 100원

print("\n=== PayPal ===")
service2 = PaymentService(PayPalPayment())
print(service2.process(200))  # PayPal 결제: 200원

print("\n=== 카카오페이 ===")
service3 = PaymentService(KakaoPayPayment())
print(service3.process(300))  # 카카오페이 결제: 300원

# PaymentService 코드 수정 없이 결제 수단 변경!

```

