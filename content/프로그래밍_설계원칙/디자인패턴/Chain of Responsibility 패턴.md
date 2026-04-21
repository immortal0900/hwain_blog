---
생성날짜:
- 2026-04-21 12:14
마지막수정날짜:
- 2026-04-21-화요일 12:14
tags:
- 설계원칙
- 디자인패턴
- GoF
- 행위패턴
- AI에이전트
- 미들웨어
별칭:
- Chain of Responsibility
- CoR
- 책임 연쇄 패턴
- 책임 사슬 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# Chain of Responsibility 패턴이란?

Chain of Responsibility(체인 오브 리스판서빌리티, CoR, 책임 연쇄 패턴, 요청을 여러 처리기가 줄지어 순차 처리하되 자기가 처리할 수 없으면 다음 처리기로 넘기는 행위 디자인 패턴)는 **"요청을 여러 핸들러가 줄 서서 하나씩 검사하고, 해당되는 핸들러만 처리하거나 전부를 거쳐가는"** 구조의 패턴이다. GoF 분류상 **행위(Behavioral) 패턴**.

웹 프레임워크에서 자주 보는 **미들웨어(middleware, 요청과 응답 사이에 끼어들어 전처리/후처리를 담당하는 함수 사슬)** 구조가 바로 이 패턴의 대표 사례임.

## 한눈에 보기

| 항목     | 내용                                            |
| ------ | --------------------------------------------- |
| 분류     | 행위 패턴(Behavioral)                             |
| 한 줄 정의 | 요청을 줄 선 핸들러들에 순차로 넘기며 처리 책임을 할당              |
| 핵심 구성  | Handler(추상), ConcreteHandler들, next 포인터       |
| 해결 문제  | `if/elif` 폭탄, 전처리/검증/권한 체크 등 파이프라인 조립 |
| 관련 원칙  | SRP, OCP                                      |

> **한마디 요약**: "거쳐 가는 창구들. 각 창구는 자기 일만 하고 다음으로 넘긴다."

---

## 일상 비유

출입국 심사를 떠올리면 쉽다.

**여권 심사대 → 세관 X-ray → 검역**

각 창구는 자기 영역만 보고 통과시키거나 막는다. 아무도 전체를 혼자 처리하지 않고, 책임이 체인을 따라 이어짐.

다른 비유:
- 회사 결재 라인(팀장 → 부장 → 이사, 해당 금액까지만 결재 권한 있음)
- 택배 물류센터(지역 허브 → 간선 → 배송 기사, 각 단계가 자기 역할)
- 콘센트 멀티탭(가전 → 멀티탭 → 벽 콘센트 → 차단기, 문제 생기면 차단기까지 올라감)

---

## 두 가지 변형

CoR은 실무에서 크게 두 가지 변형으로 쓰인다.

| 변형                      | 동작                   | 예시                         |
| ----------------------- | -------------------- | -------------------------- |
| **Pure CoR** (찾으면 멈춤)   | 처리할 수 있는 핸들러 하나가 처리하면 체인 종료 | 예외 핸들러, 로그 레벨 필터, 승인 레벨    |
| **Pipeline** (모두 통과)    | 모든 핸들러가 순서대로 모두 실행    | HTTP 미들웨어, 데이터 전처리 파이프라인 |

두 변형 모두 "다음 핸들러로 넘긴다"는 뼈대는 같다.

---

## 구조

```
Request ──▶ [Handler1] ──▶ [Handler2] ──▶ [Handler3] ──▶ Response
               │next            │next            │
               ▼                ▼                ▼
           처리 or 패스        처리 or 패스     처리 or 끝
```

```python
class Handler(ABC):
    def __init__(self):
        self._next: Handler | None = None

    def set_next(self, h: "Handler") -> "Handler":
        self._next = h
        return h  # 체이닝 문법 지원

    @abstractmethod
    def handle(self, request): ...
```

---

## 나쁜 예 vs 좋은 예 (Python)

### 나쁜 예: if문 체인

```python
def handle_request(req):
    # 인증
    if not req.token:
        raise AuthError()

    # 레이트 리밋
    if too_many_requests(req.user):
        raise RateLimitError()

    # 로깅
    log(req)

    # 페이로드 검증
    if not is_valid(req.body):
        raise ValidationError()

    # 비즈니스 로직
    return process(req)
```

문제:
- 새 단계(예: CORS 체크) 추가하려면 함수 본문을 수정해야 함
- 각 단계를 단독으로 테스트하기 어려움
- 순서를 바꾸려면 줄 위치를 옮겨야 함

### 좋은 예: CoR 패이프라인

```python
from abc import ABC, abstractmethod

class Middleware(ABC):
    def __init__(self):
        self._next: "Middleware | None" = None

    def set_next(self, h):
        self._next = h
        return h

    def handle(self, req):
        result = self.process(req)
        if self._next and result is not None:
            return self._next.handle(result)
        return result

    @abstractmethod
    def process(self, req): ...

class AuthMiddleware(Middleware):
    def process(self, req):
        if not req.token:
            raise AuthError()
        return req

class RateLimitMiddleware(Middleware):
    def process(self, req):
        if too_many_requests(req.user):
            raise RateLimitError()
        return req

class LoggingMiddleware(Middleware):
    def process(self, req):
        log(req)
        return req

# 조립
chain = AuthMiddleware()
chain.set_next(RateLimitMiddleware()).set_next(LoggingMiddleware())

chain.handle(request)
```

순서 변경, 단계 추가/제거가 체인 조립부만 건드리면 끝난다.

---

## AI 에이전트 개발 예시: LLM 입력/출력 파이프라인

LLM 에이전트에서는 사용자 입력이 들어와 응답이 나가기까지 거쳐야 할 단계가 많다. 프롬프트 인젝션(prompt injection, 악성 사용자가 프롬프트에 지시어를 심어 원래 의도를 뒤집으려는 공격) 탐지, 개인정보 마스킹, 토픽 필터, 캐시 확인 같은 것들.

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class AgentRequest:
    user_input: str
    user_id: str
    context: dict

class AgentMiddleware(ABC):
    def __init__(self):
        self._next = None

    def set_next(self, h):
        self._next = h
        return h

    def handle(self, req: AgentRequest):
        out = self.process(req)
        if self._next:
            return self._next.handle(out)
        return out

    @abstractmethod
    def process(self, req: AgentRequest) -> AgentRequest: ...

class PromptInjectionGuard(AgentMiddleware):
    def process(self, req):
        if contains_injection_pattern(req.user_input):
            raise SecurityError("prompt injection suspected")
        return req

class PIIMasker(AgentMiddleware):
    """개인정보(전화번호, 주민번호) 마스킹"""
    def process(self, req):
        req.user_input = mask_pii(req.user_input)
        return req

class SemanticCache(AgentMiddleware):
    def process(self, req):
        hit = cache.lookup(req.user_input)
        if hit:
            req.context["cached_response"] = hit
        return req

class TopicFilter(AgentMiddleware):
    def process(self, req):
        if is_off_topic(req.user_input):
            raise ValueError("범위 밖 질문")
        return req

# 조립
pipeline = PromptInjectionGuard()
(pipeline
    .set_next(PIIMasker())
    .set_next(TopicFilter())
    .set_next(SemanticCache()))

safe_req = pipeline.handle(request)
# 이제 Agent 본체로 넘김
response = agent.run(safe_req)
```

LangChain의 Runnable 체인, Guardrails AI, NeMo Guardrails, LlamaIndex의 QueryPipeline 같은 프레임워크가 내부적으로 이 패턴을 쓴다.

### Pure CoR 변형: Tool 라우팅

여러 Tool 중 하나가 처리하는 구조라면 "찾으면 멈춤" 형태가 적합함.

```python
class ToolHandler(ABC):
    def __init__(self):
        self._next = None

    def set_next(self, h):
        self._next = h
        return h

    def handle(self, query):
        if self.can_handle(query):
            return self.execute(query)
        if self._next:
            return self._next.handle(query)
        return None  # 아무도 처리 못함

    @abstractmethod
    def can_handle(self, query): ...
    @abstractmethod
    def execute(self, query): ...

class CalculatorHandler(ToolHandler):
    def can_handle(self, query):
        return looks_like_math(query)
    def execute(self, query):
        return eval_math(query)

class WebSearchHandler(ToolHandler):
    def can_handle(self, query):
        return needs_fresh_info(query)
    def execute(self, query):
        return search_web(query)

class LLMFallbackHandler(ToolHandler):
    def can_handle(self, query):
        return True  # 최후의 보루
    def execute(self, query):
        return llm.chat(query)

router = CalculatorHandler()
router.set_next(WebSearchHandler()).set_next(LLMFallbackHandler())
router.handle(user_query)
```

---

## 함수형 변형: 미들웨어 합성

클래스 없이 함수를 체인으로 묶는 방식도 흔하다. Express.js, Koa, FastAPI의 dependency injection 등이 이 스타일임.

```python
from functools import reduce

def auth(req, next):
    if not req.get("token"):
        raise AuthError()
    return next(req)

def log_middleware(req, next):
    print(f"incoming: {req}")
    result = next(req)
    print(f"outgoing: {result}")
    return result

def compose(middlewares, final):
    def chain(req, i=0):
        if i == len(middlewares):
            return final(req)
        return middlewares[i](req, lambda r: chain(r, i + 1))
    return chain

app = compose([log_middleware, auth], handler)
app(request)
```

---

## 언제 쓰면 좋은가

1. **요청이 여러 단계를 거쳐 처리**되어야 할 때 (전처리, 검증, 변환)
2. **단계 순서와 구성을 런타임/설정으로 바꾸고 싶을** 때
3. **각 단계 책임을 독립적으로 테스트**하고 싶을 때
4. **"해당되는 것 하나만 처리"** 같은 디스패처 역할이 필요할 때

## 트레이드오프

| 장점                     | 단점                                   |
| ---------------------- | ------------------------------------ |
| 단계별 SRP 깔끔             | 요청이 어디까지 갔는지 추적 어려움                  |
| 런타임 조립/순서 변경 용이         | 체인이 길어지면 성능 오버헤드                     |
| 새 단계 추가가 기존 단계 수정 없이 가능 | 아무 핸들러도 처리 안 하면 조용히 지나갈 위험 (fallback 필수) |
| 테스트가 단계별로 쉬움           | 너무 잘게 쪼개면 역추적이 괴로워짐                  |

---

## 실전 주의점

### 1. 종료 처리
Pure CoR 변형에서는 "아무도 처리 못 했을 때" 기본 응답 또는 명시적 에러를 반환할 것. 조용한 실패는 디버깅 지옥임.

### 2. 예외 정책
어떤 단계에서 예외가 나면 체인 전체가 멈춘다. 복구 가능한 오류인지, 전체 실패인지 분류해 핸들링할 것.

### 3. 비동기 체인
`async` 환경에서는 체인 조립도 `await` 기반으로 가야 함.

```python
class AsyncMiddleware(ABC):
    async def handle(self, req):
        out = await self.process(req)
        if self._next:
            return await self._next.handle(out)
        return out
```

---

## 다른 패턴과의 관계 (포함 관계)

- **[[Decorator 패턴|Decorator]]와 구조적 유사**: 둘 다 "다음"을 가리키며 호출을 위임. 차이점은 Decorator는 원래 객체의 인터페이스를 유지한 채 "기능을 추가", CoR은 "처리할지 말지 결정하고 넘김"
- **[[Observer 패턴|Observer]]와 구분**: Observer는 브로드캐스트(모두에게), CoR은 순차 처리(하나씩)
- **[[Strategy 패턴|Strategy]]와 구분**: Strategy는 "알고리즘 하나를 고름", CoR은 "여러 단계를 거침"
- **Command 패턴과 자주 조합**: 각 Handler의 처리 단위를 Command 객체로 표현
- **미들웨어/파이프라인 아키텍처의 객체지향 표현**: Express, Django, FastAPI, Rack 등 모든 웹 프레임워크의 근간

---

## 직접 확인하기

자기 코드에서 "요청 → 여러 if 단계 통과 → 최종 처리" 형태를 찾아봐라. 단계가 3개 이상이고 앞으로 더 늘 가능성이 있다면 CoR 후보임. 특히 **순서가 중요하면서** 단계가 독립적이면 딱 맞는다.

Python 표준 라이브러리 `logging` 모듈의 Filter 체인, Django의 MIDDLEWARE 설정, Starlette/FastAPI의 `app.add_middleware()` 등이 모두 CoR 구현체이다.

---

## 요약

> **"요청을 줄 세운 핸들러들에 차례로 넘겨라. 각자 자기 책임만 지고 다음으로 패스한다."**

관련 문서:
- [[SOLID원칙]]
- [[Observer 패턴]]
- [[Decorator 패턴]]
