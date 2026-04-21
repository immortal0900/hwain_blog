---
생성날짜:
- 2026-04-21 00:00
마지막수정날짜:
- 2026-04-21-화요일 00:00
tags:
- 설계원칙
- SOLID
- LSP
- 상속
- 다형성
- 계약에의한설계
- AI에이전트
별칭:
- LSP
- 리스코프 치환 원칙
- Liskov Substitution Principle
type:
- 자료수집
Area/Reasource:
Project:
---
# 리스코프 치환 원칙 (SOLID - LSP)

SOLID 중 세 번째 글자 L에 해당하는 LSP(Liskov Substitution Principle, 리스코프 치환 원칙)를 정리함. 전체 지도는 [[SOLID원칙]], 이웃 원칙 비교는 [[SOLID_ O와 D의 차이]], [[고수준 모듈과 저수준 모듈(SOLID DIP)]] 참고.

---

## 한 줄 정의

> **"자식 클래스는 언제 어디서 부모 클래스 자리에 대신 들어가도 시스템이 깨지면 안 된다."**

상속(inheritance, 부모 클래스의 속성/메서드를 자식이 물려받는 관계)은 코드만 물려받는 게 아니라 **행동의 약속**까지 물려받는다. LSP는 그 약속을 어기지 말라는 규칙이다.

---

## 1. 어디서 나왔는가 (공식 정의)

### 1.1 Barbara Liskov (1987)

MIT 교수 Barbara Liskov가 1987년 OOPSLA 기조연설 "Data Abstraction and Hierarchy"에서 처음 제안함. 1994년 Jeannette Wing과 공동 논문 "A Behavioral Notion of Subtyping"에서 형식적으로 정리.

### 1.2 원문 그대로

> "Let φ(x) be a property provable about objects x of type T. Then φ(y) should be true for objects y of type S where S is a subtype of T."

풀어 쓰면 이렇다.

- `T`: 부모 타입 (예: `Embedder`)
- `S`: `T`의 자식 타입 (예: `OpenAIEmbedder`)
- `φ`: 부모 타입 객체에 대해 "증명 가능한 모든 성질" (예: "embed()는 list[float]을 리턴한다")

→ 부모에 대해 성립하는 성질은 **자식에도 그대로 성립해야 한다**는 뜻.

### 1.3 일상 비유

AA 건전지(부모)가 들어가는 리모컨이 있다고 하자. 시중에 나오는 모든 AA 건전지(자식)는 그 리모컨에 끼워 써도 동작해야 정상이다. "이 브랜드 AA는 방향이 반대" "저 브랜드 AA는 전압이 3V" 같은 제품이 있으면 AA 규격을 위반한 것이고, 리모컨(호출자)이 망가진다. 이것이 LSP 위반 상황.

---

## 2. 무엇을 지켜야 하는가 (Design by Contract 4계명)

Bertrand Meyer의 Design by Contract(DbC, 계약에 의한 설계)에서 나온 네 가지 규칙을 LSP가 그대로 받아들임.

| 계약 항목 | 자식은 어떻게 해야 하나 | 짧은 설명 |
| --- | --- | --- |
| **Precondition (사전 조건)** | 더 **엄격하게** 만들면 안 됨 (동등하거나 약화만 허용) | 부모가 "양수 받는다" 했는데 자식이 "1000 이하 양수만 받는다"로 좁히면 위반 |
| **Postcondition (사후 조건)** | 더 **약하게** 만들면 안 됨 (동등하거나 강화만 허용) | 부모가 "정렬된 리스트 리턴" 했는데 자식이 "그냥 리스트" 리턴하면 위반 |
| **Invariant (불변 조건)** | 부모의 불변 조건을 **깨면 안 됨** | 부모가 "잔액은 항상 0 이상"이면, 자식도 그 규칙 유지 |
| **History constraint (이력 조건)** | 부모가 허용 안 한 상태 변화를 자식이 **새로 허용해선 안 됨** | 부모가 "한 번 정해지면 불변"이라 했으면 자식이 setter 추가 금지 |

추가로 예외(exception)에 대해서는 이런 규칙이 따라붙는다.

- **예외**: 자식은 부모가 던지지 않는 새 예외를 던져선 안 됨. 던지더라도 부모가 던지는 예외의 **하위 타입**이어야 함.

### 2.1 시그니처 공변/반공변 (Covariance / Contravariance)

타입 시스템 레벨에서도 규칙이 있음.

| 위치 | 자식은 | 이유 |
| --- | --- | --- |
| **파라미터 타입** | 부모보다 더 **넓은(상위)** 타입까지 받을 수 있음 (반공변, contravariant) | 호출자가 부모 기준으로 좁게 넘겨도 자식이 수용 가능해야 함 |
| **리턴 타입** | 부모보다 더 **좁은(하위)** 타입을 리턴 가능 (공변, covariant) | 호출자가 부모 타입 변수로 받아도 문제 없음 |

Python은 런타임 덕 타이핑이라 강제력이 약하지만, `mypy`나 TypeScript, Java 같은 정적 타입 시스템에서는 이 규칙을 컴파일러가 잡아낸다.

---

## 3. 고전적인 위반 예시

### 3.1 정사각형-직사각형 문제 (Square-Rectangle Problem)

LSP 교과서에 항상 등장하는 예시임.

```python
class Rectangle:
    def __init__(self, w, h):
        self.w = w
        self.h = h
    def set_width(self, w): self.w = w
    def set_height(self, h): self.h = h
    def area(self): return self.w * self.h

# "정사각형은 직사각형이다(is-a)" 라고 생각해서 상속
class Square(Rectangle):
    def set_width(self, w):
        self.w = w
        self.h = w  # 정사각형은 변이 같아야 하니까
    def set_height(self, h):
        self.w = h
        self.h = h
```

얼핏 수학적으로 맞아 보이지만 호출자 관점에서는 함정이다.

```python
def resize(r: Rectangle):
    r.set_width(5)
    r.set_height(4)
    assert r.area() == 20  # Rectangle 사전제: 20이 나와야 함

resize(Rectangle(1, 1))  # OK: 20
resize(Square(1, 1))     # 실패: 16 (set_height가 w까지 4로 덮어씀)
```

`Rectangle` 자리에 `Square`를 끼웠더니 호출자 코드가 깨졌다. "수학적으로 is-a"여도 "행동적으로 is-a"가 아니면 상속 금지.

### 3.2 펭귄-새 문제 (Penguin-Bird Problem)

```python
class Bird:
    def fly(self): ...

class Penguin(Bird):
    def fly(self):
        raise NotImplementedError("펭귄은 못 난다")  # ❌ LSP 위반
```

부모가 "fly()는 성공한다"는 약속을 했는데 자식이 예외를 던지면 약속 파기. 해결은 계층을 다시 짜는 것이다.

```python
class Bird: ...
class FlyingBird(Bird):
    def fly(self): ...
class Penguin(Bird): ...  # FlyingBird는 상속 안 함
```

**포인트**: "현실 세계의 is-a"가 아니라 "코드 레벨의 행동 교환 가능성"이 상속의 기준.

---

## 4. AI Agent 개발에서 LSP 적용

에이전트 코드는 "인터페이스 + 여러 구현체" 구조가 많아서(LLM 프로바이더, Vector Store, Tool, Memory) LSP가 매우 자주 걸림.

### 4.1 Embedder 교체 가능성

```python
from abc import ABC, abstractmethod

class Embedder(ABC):
    """
    계약:
      - 입력: 비어있지 않은 str
      - 출력: 고정 차원 list[float]
      - 실패 시: EmbeddingError 예외
    """
    @abstractmethod
    def embed(self, text: str) -> list[float]: ...
    @abstractmethod
    def dim(self) -> int: ...
```

#### 위반 자식들

```python
# ❌ 위반 1: 사후 조건 약화 (None 리턴 추가)
class FlakyEmbedder(Embedder):
    def embed(self, text):
        if len(text) > 1000:
            return None  # 부모 약속: list[float] 보장
        return openai_embed(text)
    def dim(self): return 1536

# ❌ 위반 2: 사전 조건 강화 (더 좁게 제한)
class StrictEmbedder(Embedder):
    def embed(self, text):
        if len(text) > 500:
            raise ValueError("500자 초과 금지")  # 부모는 그런 제약 없었음
        return openai_embed(text)
    def dim(self): return 1536

# ❌ 위반 3: 새 예외 추가 (예고 없는 예외)
class NetworkEmbedder(Embedder):
    def embed(self, text):
        try:
            return api_call(text)
        except Exception:
            raise RuntimeError("망함")  # 부모는 EmbeddingError만 명시

# ❌ 위반 4: 불변식 파기 (차원이 매번 다름)
class DynamicEmbedder(Embedder):
    def embed(self, text):
        return [random() for _ in range(random.choice([768, 1536]))]
    def dim(self): return 1536  # 실제 리턴과 불일치
```

#### 합격 자식

```python
# ✅ 긴 입력은 내부에서 청크로 나눠 평균 냄, 계약 유지
class ChunkedEmbedder(Embedder):
    def __init__(self, base: Embedder, chunk_size: int = 500):
        self.base = base
        self.chunk_size = chunk_size
    def embed(self, text):
        if len(text) <= self.chunk_size:
            return self.base.embed(text)
        chunks = [text[i:i+self.chunk_size]
                  for i in range(0, len(text), self.chunk_size)]
        vecs = [self.base.embed(c) for c in chunks]
        return [sum(x)/len(x) for x in zip(*vecs)]  # 차원/타입 유지
    def dim(self): return self.base.dim()
```

### 4.2 Vector Store 교체 가능성

```python
class VectorStore(ABC):
    """
    계약:
      - upsert(vecs): 성공하면 None, 실패 시 StoreError
      - search(q, k): 유사도 내림차순 상위 k개 (id, score) 리스트
    """
    @abstractmethod
    def upsert(self, vecs: list[tuple[str, list[float]]]) -> None: ...
    @abstractmethod
    def search(self, q: list[float], k: int) -> list[tuple[str, float]]: ...

# ❌ LSP 위반: "정렬 안 된 리스트" 리턴 (사후 조건 약화)
class LazyStore(VectorStore):
    def search(self, q, k):
        return self._raw_scan(q)[:k]  # 정렬하지 않음

# ✅ LSP 합격: 내부 구현은 달라도 계약은 동일
class InMemoryStore(VectorStore):
    def __init__(self): self.data = {}
    def upsert(self, vecs):
        for id_, v in vecs: self.data[id_] = v
    def search(self, q, k):
        scored = [(id_, cosine(q, v)) for id_, v in self.data.items()]
        return sorted(scored, key=lambda x: -x[1])[:k]
```

LSP를 지켰기 때문에 `agent = RAGAgent(InMemoryStore())`로 테스트하고, 프로덕션은 `RAGAgent(PineconeStore())`로 바꿔도 Agent 코드를 한 줄도 건드릴 필요가 없음. 이것이 [[고수준 모듈과 저수준 모듈(SOLID DIP)|DIP]]와 LSP가 결합할 때 나오는 파괴력.

### 4.3 Agent 계층에서의 LSP

```python
class Agent(ABC):
    @abstractmethod
    def run(self, query: str) -> str:
        """항상 문자열 응답을 리턴한다"""

class OpenAIAgent(Agent):
    def run(self, query): return openai_chat(query)

class ClaudeAgent(Agent):
    def run(self, query): return claude_chat(query)

# 호출자 입장
def process(agent: Agent, queries: list[str]):
    return [agent.run(q) for q in queries]

process(OpenAIAgent(), [...])  # 동작
process(ClaudeAgent(), [...])  # 동작해야 한다
```

`ClaudeAgent.run()`이 어떤 입력에서 `None`을 돌려주거나 stream generator를 리턴해버리면 `process()` 함수가 깨진다. 이게 실무에서 제일 자주 밟는 지뢰.

---

## 5. 위반 신호 빠른 체크리스트

아래 중 하나라도 있으면 LSP를 의심해볼 것.

- [ ] 자식 메서드에 `raise NotImplementedError`가 있다
- [ ] `isinstance(obj, SpecificChild)` 분기가 호출부에 생겼다 (= 다형성이 안 통한다는 뜻)
- [ ] 자식에서 파라미터 타입을 더 좁혔다 (`str` → `ShortString`)
- [ ] 자식에서 리턴 타입을 더 넓혔다 (`list[float]` → `Optional[list[float]]`)
- [ ] 자식이 부모가 약속 안 한 예외를 던진다
- [ ] 자식에서 어떤 입력에 대해 `None`, 빈 리스트, 기본값을 몰래 리턴한다
- [ ] 부모의 불변식(잔액 ≥ 0, 리스트 정렬됨)을 자식이 깬다

---

## 6. 직접 확인해보기

자기 프로젝트에서 이렇게 테스트해보면 LSP 준수 여부가 즉시 드러난다.

### 6.1 계약 기반 테스트 (contract test)

부모 인터페이스에 대한 테스트를 작성한 뒤, **모든 자식 구현체를 같은 테스트에 돌려본다**.

```python
import pytest

@pytest.fixture(params=[
    OpenAIEmbedder(),
    CohereEmbedder(),
    ChunkedEmbedder(OpenAIEmbedder()),
    InMemoryEmbedder(),
])
def embedder(request):
    return request.param

def test_returns_list_of_floats(embedder):
    v = embedder.embed("hello")
    assert isinstance(v, list)
    assert all(isinstance(x, float) for x in v)
    assert len(v) == embedder.dim()

def test_rejects_empty_input(embedder):
    with pytest.raises(ValueError):
        embedder.embed("")
```

테스트가 하나의 구현체에서만 깨지면 그 구현체가 LSP 위반.

### 6.2 mypy / pyright 돌려보기

```bash
mypy --strict agents/
```

정적 타입 체커는 파라미터 반공변/리턴 공변 규칙을 자동으로 잡아준다. `error: Return type "Optional[list[float]]" of "embed" incompatible with return type "list[float]" in supertype "Embedder"` 같은 경고가 대표적인 LSP 위반 알림.

---

## 7. 왜 이 원칙이 필요한가 (설계 의도)

### 7.1 다형성의 안전장치

OOP에서 다형성(polymorphism, 같은 인터페이스로 여러 구현을 다루는 능력)이 작동하려면 "부모 자리에 자식을 꽂아도 깨지지 않는다"는 보장이 필수. LSP가 깨지면 다형성이 거짓말이 되고, 호출자 쪽에 `isinstance` 분기가 번져나감.

### 7.2 [[SOLID원칙#2. O: Open-Closed Principle (개방 폐쇄 원칙)|OCP]]의 전제 조건

OCP("새 구현체를 추가해도 기존 코드는 수정 안 한다")는 **새 구현체가 기존 자리에 안전하게 들어갈 수 있을 때만** 성립함. LSP 위반 구현체가 있으면 OCP를 지키려다 런타임 폭발이 발생.

### 7.3 테스트 가능성

계약만 통과하면 아무 구현체나 주입 가능 → 테스트용 Fake/Mock/InMemory 구현체를 만들기 쉬워짐. LSP가 깨지면 "실제 구현에서만 동작, Mock에서는 다르게 동작" 현상이 발생해서 테스트 신뢰도가 망가짐.

---

## 8. 다른 SOLID 원칙과의 포함 관계

| 관계 | 의미 |
| --- | --- |
| **ISP → LSP** | 인터페이스가 작을수록 약속이 단순해지고, 자식이 약속을 깨기 어려워짐. ISP는 LSP의 전제 |
| **LSP → OCP** | 자식이 부모 자리에 안전하게 들어갈 수 있어야 "새 구현체 추가로 확장"이 가능. LSP는 OCP의 전제 |
| **LSP + DIP** | 추상에 의존(DIP)하는 모듈은 어떤 구현체든 같은 계약을 지킨다고 믿음. LSP가 그 믿음을 실제로 보증 |
| **LSP vs 상속 남용** | is-a 관계라도 행동이 다르면 상속 대신 컴포지션(composition, 필드로 갖기) 사용 |

```
ISP (인터페이스 쪼개기)
   │
   ▼
LSP (행동 약속 지키기)  ◄── 이 글의 주제
   │
   ├──► DIP (추상에 의존하되 계약이 보증되어야 함)
   │
   └──► OCP (교체/확장이 안전해야 함)
```

---

## 9. 실무 체크리스트 (AI Agent Developer용)

- [ ] LLM Provider 클래스들이 똑같은 계약(입출력 타입, 예외, 스트리밍 여부)을 지키는가?
- [ ] Vector Store 구현체 전부에 같은 pytest를 돌리면 모두 통과하는가?
- [ ] Tool 구현체들이 `run(args) -> str` 계약을 안 깨고 따르는가? (예외 타입, None 리턴 여부 포함)
- [ ] Memory(Short/Long/Episodic) 구현체를 서로 바꿔 끼워도 Agent 동작이 동일한가?
- [ ] 호출 코드에 `isinstance(llm, OpenAIClient)` 같은 분기가 생기진 않았는가?
- [ ] 자식 클래스 어딘가에 `NotImplementedError`가 숨어있진 않은가?

---

## 10. 한마디 요약

> **"상속은 '코드 재사용' 장치가 아니라 '행동 계약' 장치다. 부모가 맺은 약속을 자식이 어기면 호출자가 피를 본다."**

"is-a"만 보지 말고 "대체 가능한가(substitutable)?"를 물어볼 것. 대체 불가능하면 상속 대신 컴포지션으로 간다.

---

## 관련 문서

- [[SOLID원칙]]
- [[SOLID_ O와 D의 차이]]
- [[고수준 모듈과 저수준 모듈(SOLID DIP)]]

## 참고 자료

- Liskov, B. (1987). "Data Abstraction and Hierarchy". OOPSLA '87 Keynote
- Liskov, B. & Wing, J. (1994). "A Behavioral Notion of Subtyping". ACM TOPLAS, 16(6)
- Meyer, B. (1997). "Object-Oriented Software Construction" (Design by Contract 원전)
- Martin, R. C. (2000). "Design Principles and Design Patterns" (SOLID 용어 정착)
