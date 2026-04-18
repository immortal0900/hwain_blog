---
생성날짜:
- 2026-01-28 03:19
마지막수정날짜:
- 2026-01-28-수요일 03:19
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## super() 사용 (추천)
**작동 원리**:[](https://stackoverflow.com/questions/21639788/difference-between-super-and-calling-superclass-directly)​​
- Python이 **MRO(Method Resolution Order)** 따라 순서대로 호출
- 중복 호출 방지 (다이아몬드 상속 문제 해결)
```python
class RealEstateTool(BaseTool, LoggableMixin, CacheableMixin):
    def __init__(self):
        super().__init__()  # ← MRO 순서대로 자동 호출
```

## ClassName.**init**(self) 직접 호출
**차이점**:[](https://www.designgurus.io/answers/detail/what-does-super-do-in-python-what-is-the-difference-between-super__init__-and-explicit-superclass-__init__)
- 특정 클래스만 골라서 호출 가능
- 하지만 MRO(실행순서) 무시 → 중복 호출 위험
```python
class RealEstateTool(BaseTool, LoggableMixin, CacheableMixin):
    def __init__(self):
        BaseTool.__init__(self)       # 명시적 호출
        CacheableMixin.__init__(self) # 명시적 호출
```

## 실제 실행 예시
```python
class A:
    def __init__(self):
        print("A 초기화")

class B(A):
    def __init__(self):
        print("B 초기화")
        super().__init__()

class C(A):
    def __init__(self):
        print("C 초기화")
        super().__init__()

# super() 사용
class D1(B, C):
    def __init__(self):
        print("D1 초기화")
        super().__init__()

print("=== super() 사용 ===")
d1 = D1()
# 출력:
# D1 초기화
# B 초기화
# C 초기화
# A 초기화  ← A는 1번만! <- 부모들인 B와 C 가 상속받은 부모

print("\n=== MRO 확인 ===")
print(D1.mro())
# [D1, B, C, A, object]

# 직접 호출
class D2(B, C):
    def __init__(self):
        print("D2 초기화")
        B.__init__(self)
        C.__init__(self)

print("\n=== 직접 호출 ===")
d2 = D2()
# 출력:
# D2 초기화
# B 초기화
# A 초기화  ← A 호출 1번 D2의 부모인 B의 부모
# C 초기화
# A 초기화  ← A 호출 2번! (중복) D2의 부모인 C의 부모

```

## 언제 뭘 쓰나?

**super() 써야 할 때**:[](https://stackoverflow.com/questions/21639788/difference-between-super-and-calling-superclass-directly)
- 다중 상속 (대부분의 경우)
- 유연한 구조 원할 때
    

**직접 호출 써야 할 때**:
- 특정 부모만 골라서 호출
- Mixin이 `__init__` 없을 때 필요한 것만

```python
class LoggableMixin:
    def log(self, message):
        print(f"[LOG] {message}") # <- 인자가 message로 들어옴

class CacheableMixin:
    def __init__(self):
        print("CacheableMixin 초기화")
        self.cache = {}
    
    def get_cached(self, key):
        return self.cache.get(key) # 딕셔너리 key 값을 넣어서 value를 가져오게 하는 것

class BaseTool:
    def __init__(self):
        print("BaseTool 초기화")
        self.name = ""

# ✅ 올바른 방식
class RealEstateTool(BaseTool, LoggableMixin, CacheableMixin):
    name = "real_estate_search"
    description = "부동산 검색"
    
    def __init__(self):
        print("RealEstateTool 초기화")
        super().__init__()  # ← MRO 순서대로 모두 호출
    
    def _run(self, query: str):
        self.log(f"검색 시작: {query}") # f"[LOG] {message}" 의 message로 f"검색 시작: {query}" 이 들어옴
        cached = self.get_cached(query) 
        if cached:
            return cached # 맨처음엔 저장된 캐시가 없으나, 그 후로 같은 질문이 들어오면(캐시가 있으면) `query` 를 key로 해서 value를 반환함
        result = f"{query} 검색 결과"
        self.cache[query] = result # f"{query} 검색 결과" 가 value , query 가 key로  
        return result

print("=== 객체 생성 ===")
tool = RealEstateTool()
# 출력:
# RealEstateTool 초기화
# BaseTool 초기화
# CacheableMixin 초기화

print("\n=== 실행 ===")
result1 = tool._run("강남 아파트")
# 출력: [LOG] 검색 시작: 강남 아파트
print(result1)  # 강남 아파트 검색 결과

result2 = tool._run("강남 아파트")  # 캐시 사용
print(result2)  # 강남 아파트 검색 결과 (캐시에서)

```

## .get()이 뭐하는 용도?

**딕셔너리에서 안전하게 값 가져오기** (키 없어도 에러 안 남)
```python
# 일반 접근 vs .get() 비교
cache = {"강남": "강남 데이터"}

# 1. 대괄호 [] 접근
print(cache["강남"])  # "강남 데이터"
print(cache["서초"])  # ❌ KeyError 발생!

# 2. .get() 메서드
print(cache.get("강남"))  # "강남 데이터"
print(cache.get("서초"))  # None (에러 안 남!)
print(cache.get("서초", "기본값"))  # "기본값"
```