---
생성날짜:
- 2026-01-28 02:42
마지막수정날짜:
- 2026-01-28-수요일 02:42
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## 다중 상속(Multiple Inheritance) 기본

**여러 부모 클래스를 동시에 상속**
```python
class Parent1:
    def method1(self):
        return "Parent1 메서드"

class Parent2:
    def method2(self):
        return "Parent2 메서드"

class Child(Parent1, Parent2):  # ← 2개 부모 상속
    def child_method(self):
        return "Child 메서드"

# 사용
obj = Child()
print(obj.method1())      # Parent1 메서드
print(obj.method2())      # Parent2 메서드
print(obj.child_method()) # Child 메서드
```

## 메서드 순서 (MRO: Method Resolution Order)

**왼쪽에서 오른쪽 순서로 메서드 찾음**
```python
class A:
    def greet(self):
        return "A"

class B:
    def greet(self):
        return "B"

class C(A, B):  # A가 먼저
    pass

obj = C()
print(obj.greet())  # "A" ← A가 우선!
print(C.mro())      # 순서 확인
# [<class 'C'>, <class 'A'>, <class 'B'>, <class 'object'>]
```
