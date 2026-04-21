---
생성날짜:
- 2026-01-27 00:09
마지막수정날짜:
- 2026-01-27-화요일 00:08
tags:
- python
- class
- 객체지향
- OOP
- AI_Agent
별칭:
- Instance
- 객체
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python
Project:
---

# 인스턴스(Instance)

## 포함 관계 한눈에 보기

**클래스(Class) ⊃ 인스턴스(Instance) ⊃ 속성/메서드**

| 개념 | 무엇 | 비유 | 예시 |
|---|---|---|---|
| 클래스(Class, 설계도 코드) | 타입 정의 문서 | 붕어빵 틀 | `class Agent:` |
| 인스턴스(Instance, 메모리에 찍힌 실물) | 클래스로 만든 실제 객체 | 붕어빵 한 개 한 개 | `agent = Agent()` |
| 속성/메서드(Attribute/Method) | 인스턴스가 들고 있는 데이터/동작 | 팥/슈크림 같은 붕어빵 속 | `agent.name`, `agent.run()` |

**한마디 요약**: 클래스는 붕어빵 틀, 인스턴스는 찍어낸 붕어빵, 속성은 붕어빵 속이다.

## 질문: 인스턴스가 뭐야?

**핵심**: 클래스로 만든 **실제 객체**(메모리에 존재하는 실물)

### 일상 비유: 붕어빵 틀과 붕어빵

- **클래스** = 붕어빵 틀(설계도, 코드)
- **인스턴스** = 틀로 찍어낸 붕어빵 1개, 2개, 3개(각각 독립된 실물)
- `ClassName()` 호출 = 틀에 반죽 붓고 찍어내는 행위(인스턴스화, instantiation)

## 메모리 관점으로 보기

```python
# 클래스는 코드(설계도)
class Person:
    def __init__(self, name):
        self.name = name

# 인스턴스는 메모리에 실제 존재
person1 = Person("김철수")  # 메모리 주소: 0x1A2B3C
person2 = Person("이영희")  # 메모리 주소: 0x4D5E6F

print(person1.name)  # "김철수"
print(person2.name)  # "이영희"
# 같은 클래스지만, 다른 데이터를 가진 독립적인 객체들
```

### 실제 메모리 주소 직접 확인

```python
print(id(person1))  # 예: 140234567891232 (10진수)
print(hex(id(person1)))  # 예: '0x7f8b1c2d4e20' (16진수)
print(hex(id(person2)))  # person1과 다른 주소
```

**왜 이렇게 설계됐나**: 같은 타입의 데이터를 반복해서 만들되, 각각 독립된 상태(state)를 갖게 하려고. 만약 인스턴스가 없으면 전역 변수만으로 상태를 관리해야 하는데, 그러면 상태 충돌 지옥이 펼쳐진다.

## AI Agent 개발 예시

**여러 Agent를 동시에 돌리는 경우**: 각 Agent가 독립된 상태(메모리, 툴, 대화 기록)를 가져야 서로 간섭하지 않는다. 이게 바로 인스턴스화가 필요한 이유.

```python
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph

class ResearchAgent:
    def __init__(self, llm: ChatOpenAI, region: str):
        self.llm = llm
        self.region = region
        self.history = []  # 각 인스턴스가 자신의 대화 기록 보유
        self.graph = self._build_graph()

    def _build_graph(self):
        builder = StateGraph(dict)
        # 노드/엣지 구성
        return builder.compile()

    def invoke(self, query: str):
        self.history.append(query)
        return self.graph.invoke({"query": query})

# 지역별 독립 Agent 인스턴스
llm = ChatOpenAI(model="gpt-4o-mini")
seoul_agent = ResearchAgent(llm, "서울")    # 인스턴스 1
busan_agent = ResearchAgent(llm, "부산")    # 인스턴스 2

# 각 Agent의 history는 완전히 분리됨
seoul_agent.invoke("강남 아파트 시세?")
busan_agent.invoke("해운대 오피스텔 시세?")

print(seoul_agent.history)  # ["강남 아파트 시세?"]
print(busan_agent.history)  # ["해운대 오피스텔 시세?"]
```

**포인트**: `seoul_agent`와 `busan_agent`는 같은 `ResearchAgent` 클래스에서 나왔지만, `self.region`과 `self.history`가 완전히 분리돼 있다. 이게 [[class_상속시`super()` 사용]]에서 다루는 상태 격리의 핵심이다.

## 단계 분해: 인스턴스가 만들어지는 과정

**1단계 → 2단계 → 3단계**

1. **1단계**: 파이썬이 `Agent(...)` 호출을 만나면 먼저 `Agent.__new__(cls)`를 호출해 **메모리에 빈 객체 껍데기**를 할당한다.
2. **2단계**: 그 껍데기를 `Agent.__init__(self, ...)`의 `self`로 전달. [[class_`__init__`]]에서 속성을 채운다.
3. **3단계**: `__init__`이 끝나면 완성된 객체를 호출자에게 돌려준다. 이제 변수 이름(`agent`)에 바인딩돼 있고, 외부에서 `agent.method()`로 접근 가능.

**직접 확인**: `__new__`가 실제로 먼저 호출되는지 보고 싶다면

```python
class Debug:
    def __new__(cls, *args, **kwargs):
        print(f"1단계: __new__ 호출, 메모리 할당")
        obj = super().__new__(cls)
        return obj

    def __init__(self, name):
        print(f"2단계: __init__ 호출, 속성 채우기")
        self.name = name

d = Debug("테스트")
# 출력:
# 1단계: __new__ 호출, 메모리 할당
# 2단계: __init__ 호출, 속성 채우기
```

## 용어 정리

```python
class PaymentProcessor:  # ← 클래스 (Class, 설계도)
    pass

processor1 = PaymentProcessor()  # ← 인스턴스 (Instance, 실제 객체)
processor2 = PaymentProcessor()  # ← 또 다른 인스턴스

# "인스턴스화(Instantiation)" = 클래스로 객체 만드는 행위
# processor1, processor2 각각이 "인스턴스"
```

| 용어 | 영문 | 의미 | 예시 코드 |
|---|---|---|---|
| 클래스 | Class | 타입 정의(코드) | `class Agent:` |
| 인스턴스 | Instance | 클래스의 실체화된 객체 | `agent = Agent()` |
| 인스턴스화 | Instantiation | 인스턴스 생성 행위 | `Agent()` |
| 객체 | Object | 파이썬에서는 거의 모든 것(숫자, 함수도 객체) | 문자열 `"hello"`도 객체 |

**주의**: 파이썬에서 "객체"는 인스턴스보다 넓은 개념이다. 함수도 객체, 클래스 자체도 객체. 다만 실무에서는 "인스턴스"와 "객체"를 거의 같은 뜻으로 쓴다.

## self의 의미 (다시 정리)

```python
class PaymentProcessor:
    def __init__(self, payment_method):
        self.payment_method = payment_method
        # self = "이 특정 인스턴스"를 의미

# 인스턴스 2개 생성
p1 = PaymentProcessor(CreditCardPayment())
p2 = PaymentProcessor(PayPalPayment())

# p1의 self.payment_method = CreditCardPayment 객체
# p2의 self.payment_method = PayPalPayment 객체
# 각각 독립적!
```

`self`가 왜 필요한지는 [[class에서 self. 필요성]] 참고.

## isinstance()로 인스턴스 확인

**실무에서 타입 체크할 때 자주 쓴다**. AI Agent에서 LLM 응답이 특정 클래스의 인스턴스인지 검증할 때 유용.

```python
from langchain_core.messages import AIMessage, HumanMessage

response = agent.invoke({"query": "hello"})
message = response["messages"][-1]

if isinstance(message, AIMessage):
    print("AI 응답입니다")
elif isinstance(message, HumanMessage):
    print("사용자 입력입니다")
```

## 실무 주의점

**1. 클래스 변수 vs 인스턴스 변수 혼동**

```python
class BadAgent:
    history = []  # ⚠️ 클래스 변수! 모든 인스턴스가 공유

class GoodAgent:
    def __init__(self):
        self.history = []  # ✅ 인스턴스 변수! 각자 독립

bad1 = BadAgent()
bad2 = BadAgent()
bad1.history.append("질문1")
print(bad2.history)  # ["질문1"] ← 공유돼서 오염됨!

good1 = GoodAgent()
good2 = GoodAgent()
good1.history.append("질문1")
print(good2.history)  # [] ← 독립
```

**왜 이렇게 되나**: 클래스 변수는 클래스 자체에 묶이고, 인스턴스 변수는 각 인스턴스에 묶인다. 가변 객체(list, dict)를 클래스 변수로 두면 모든 인스턴스가 같은 메모리 주소를 참조하므로 서로 영향을 준다.

**2. 인스턴스 없이 메서드 호출 시도**

```python
class Tool:
    def run(self):
        return "실행"

Tool.run()  # ❌ TypeError: missing 'self'
tool = Tool()
tool.run()  # ✅ "실행"
```

## 정리

- **클래스** = 설계도(코드), **인스턴스** = 설계도로 만든 실제 객체(메모리에 존재)
- **인스턴스화** = `ClassName()` 호출해서 객체 만드는 행위
- `self` = 현재 인스턴스 자기 자신을 가리킴
- 각 인스턴스는 **독립된 상태**를 가진다(AI Agent 여러 개 돌릴 때 핵심)
- 클래스 변수(공유) vs 인스턴스 변수(독립) 구분 필수

## 관련 문서

- [[class_`__init__`]]: 인스턴스 생성 시 자동 실행되는 초기화 메서드
- [[class에서 self. 필요성]]: self가 왜 필요한지, 언제 붙이는지
- [[class_파라미터]]: 인스턴스 생성 시 파라미터 전달
- [[class_타입힌트]]: 인스턴스 타입 힌트 표기법
