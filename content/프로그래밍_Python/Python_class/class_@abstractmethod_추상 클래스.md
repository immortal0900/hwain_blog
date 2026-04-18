---
생성날짜:
- 2026-01-26 23:42
마지막수정날짜:
- 2026-01-26-월요일 23:42
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
파이썬은 추상 클래스(abstract class)라는 기능을 제공한다. 추상 클래스는 **메서드의 목록만 가진 클래스**이며 **상속받는 클래스에서 메서드 구현을 강제하기 위해 사용**한다.

## 질문1: abc가 뭐지?
먼저 추상 클래스를 만들려면 **import로 abc 모듈을 가져와야 한다**( abc는 **a**bstract **b**ase **c**lass의 약자). 그리고 클래스의 ( )(괄호) 안에 **metaclass=ABCMeta**를 지정하고, 메서드를 만들 때 위에 **@abstractmethod**를 붙여서 추상 메서드로 지정한다.

```python
from abc import *
 
class StudentBase(metaclass=ABCMeta):
    @abstractmethod
    def study(self):
        pass
 
    @abstractmethod
    def go_to_school(self):
        pass
 
class Student(StudentBase): -> 상속을 받아놓고 go_to_school()을 구현 안함
    def study(self):     
        print('공부하기')
 
james = Student()
james.study()
```

실행결과
```
Traceback (most recent call last): 
  File "C:\project\class_abc_error.py", line 16, in <module> 
    james = Student() 
TypeError: Can't instantiate abstract class Student with abstract methods go_to_school
```
실행을 해보면 에러가 발생한다. 왜냐하면 추상 클래스 **StudentBase**에서는 추상 메서드로 **study**와 **go_to_school**을 정의했다. 하지만 **StudentBase**를 상속받은 **Student**에서는 **study** 메서드만 구현하고, **go_to_school** 메서드는 구현하지 않았으므로 에러가 발생한다.

따라서 추상 클래스를 상속받았다면 **@abstractmethod**가 붙은 추상 메서드를 모두 구현해야 한다. 다음과 같이 **Student**에서 **go_to_school** 메서드도 구현해준다.

## 질문2: @abstractmethod는 뭐지?

**핵심**: "이 메서드는 **반드시 구현해야 합니다**"라고 강제하는 장치 
상속 받아놓고 구현 안하면 ==객체로 만들때 에러 발생==
```python
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def process(self, amount):
        pass  # 구현 안 함, 껍데기만

```

**@abstractmethod 효과**:
```python
# 1. PaymentStrategy를 직접 인스턴스화 시도
strategy = PaymentStrategy()  # ❌ TypeError 발생!
# "추상 클래스는 직접 객체 만들 수 없음"

# 2. process()를 구현 안 한 채 상속
class BadPayment(PaymentStrategy):
    pass  # process() 구현 안 함

bad = BadPayment()  # ❌ TypeError 발생!
# "추상 메서드 process()를 구현하지 않음"

# 3. process()를 제대로 구현
class GoodPayment(PaymentStrategy):
    def process(self, amount):  # ← 구현함!
        return f"결제: {amount}원"

good = GoodPayment()  # ✅ OK! 실행됨

```