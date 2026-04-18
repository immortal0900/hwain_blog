---
생성날짜:
- 2026-01-26 19:36
마지막수정날짜:
- 2026-01-26-월요일 19:35
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## 질문2-1: `: PaymentStrategy`는 왜 쓰나?

**정답: 타입 힌트(Type Hint)**[](https://docs.python.org/3/tutorial/classes.html)​

```python
def __init__(self, payment_method: PaymentStrategy):
                                  ↑ 이 부분

```

**역할**:

- "이 파라미터는 `PaymentStrategy` 타입이어야 합니다" 라고 **문서화**하는 용도[](https://realpython.com/python-property/)​
- **런타임에는 아무 영향 없음** (Python은 실행 시 타입 체크 안 함)
- IDE(VS Code, Cursor)가 자동완성/에러 검출에 활용

**비교**:
```python
# 타입 힌트 없음 (Python 3.4 이하 스타일) 
def __init__(self, payment_method): 
    self.payment_method = payment_method 
    
# 타입 힌트 있음 (Python 3.5+ 권장) 
def __init__(self, payment_method: PaymentStrategy): 
    self.payment_method = payment_method
```

```python
# 타입 힌트 없으면
processor = PaymentProcessor("잘못된 문자열")  # 실행됨, 나중에 에러
processor.payment_method.process(100)  # AttributeError!

# 타입 힌트 있으면
processor = PaymentProcessor("잘못된 문자열")  # IDE가 경고 표시
                                              # 하지만 실행은 됨
```