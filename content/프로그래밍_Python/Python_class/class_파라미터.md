---
생성날짜:
- 2026-01-26 19:30
마지막수정날짜:
- 2026-01-26-월요일 19:30
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
```python
# Good - OCP 준수 (Strategy 패턴)

from abc import ABC, abstractmethod

  

class PaymentStrategy(ABC):

    """결제 전략 인터페이스"""

    @abstractmethod

    def process(self, amount):

        pass

  

class CreditCardPayment(PaymentStrategy):

    """신용카드 결제"""

    def process(self, amount):

        # 신용카드 처리 로직

        return f"Credit card payment: ${amount}"

  

class PayPalPayment(PaymentStrategy):

    """PayPal 결제"""

    def process(self, amount):

        # PayPal 처리 로직

        return f"PayPal payment: ${amount}"

  

class BankTransferPayment(PaymentStrategy):

    """계좌이체 결제"""

    def process(self, amount):

        # 계좌이체 처리 로직

        return f"Bank transfer: ${amount}"

  

class PaymentProcessor:

    """결제 처리기 - 기존 코드 수정 없이 확장 가능"""

    def __init__(self, strategy: PaymentStrategy):

        self.strategy = strategy

    def process(self, amount):

        return self.strategy.process(amount)

  

# 사용

processor = PaymentProcessor(CreditCardPayment())

processor.process(100)

  

# 새로운 결제 수단 추가 시 기존 코드 수정 불필요!

class CryptoPayment(PaymentStrategy):

    def process(self, amount):

        return f"Crypto payment: ${amount}"
```


## 질문1: 오른쪽 payment_method로 들어가는가?

**정답: YES, 정확히 맞음**
```python
class PaymentProcessor: 
def __init__(self, payment_method: PaymentStrategy): 
self.payment_method = payment_method
```

```python
# 사용 시나리오
credit_payment = CreditCardPayment()  # ← 여기서 객체 생성
processor = PaymentProcessor(credit_payment)  # ← 이게 아래로 전달됨
```

내부 흐름:
```python
def __init__(self, payment_method: PaymentStrategy):
    # 1. credit_payment 객체가 payment_method 파라미터로 들어옴 (오른쪽)
    # 2. 그걸 self.payment_method에 저장 (왼쪽)
    self.payment_method = payment_method
    #    ↑ 인스턴스 속성   ↑ 파라미터로 받은 객체

```

**실행 순서**:

1. `CreditCardPayment()` 객체 생성 → 메모리 어딘가에 존재
    
2. 그 객체를 `PaymentProcessor(여기)` 괄호 안에 넣음
    
3. `__init__` 파라미터 `payment_method`가 그 객체를 받음
    
4. `self.payment_method = payment_method`로 인스턴스 변수에 저장
    
5. 이제 `processor.payment_method`로 언제든 접근 가능