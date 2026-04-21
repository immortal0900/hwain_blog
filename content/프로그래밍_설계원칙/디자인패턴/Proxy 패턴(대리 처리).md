---
생성날짜:
- 2026-04-21 15:30
마지막수정날짜:
- 2026-04-21-화요일 15:30
tags:
- 설계원칙
- 디자인패턴
- Proxy
- GoF
- AI에이전트
별칭:
- Proxy
- 대리자 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# Proxy 패턴 (대리 처리)

Proxy(프록시, 대리인)는 실제 객체(Real Subject, 본체) 앞에 **대리인 객체**를 세워서 클라이언트 요청이 본체에 곧장 닿지 않도록 가로채는 구조 패턴(Structural Pattern)이다. 가로채는 김에 접근 제어, 지연 로딩, 캐시, 로깅, 과금 같은 부가 처리를 끼워 넣는다.

> GoF(Gang of Four, 1994년 디자인 패턴 책 저자 4인방)의 23개 패턴 중 구조 패턴에 속함.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 구조 패턴(Structural Pattern) |
| 핵심 아이디어 | 본체와 같은 인터페이스를 가진 "대리인"을 앞에 세움 |
| 풀려는 문제 | 본체 호출 전/후로 뭔가(권한, 캐시, 로깅)를 끼워 넣고 싶다 |
| 클라이언트 체감 | 본체를 쓰는지 대리인을 쓰는지 모르게 투명함 |
| 비슷한 패턴 | Decorator(기능 추가), Adapter(인터페이스 변환) |

## 일상 비유

사장 비서가 대표적 프록시다. 외부에서 사장에게 전화가 오면 비서가 먼저 받아서 스팸인지 거르고(접근 제어), 같은 용건이면 "어제 답 드린 그 건입니다" 하고 바로 답한다(캐시). 중요한 건만 사장에게 연결(본체 호출)하고, 통화 기록을 남긴다(로깅). 전화 건 사람 입장에서는 사장과 통화한 것처럼 느껴진다.

## 구조

```
Client ──▶ Subject(인터페이스)
              ▲           ▲
              │           │
          RealSubject   Proxy ──▶ RealSubject
            (본체)      (대리인)     (내부에서 본체 참조)
```

Proxy와 RealSubject는 **같은 Subject 인터페이스**를 구현한다. 그래서 Client는 자기가 쥔 참조가 본체인지 대리인지 신경 쓰지 않아도 된다.

## Proxy 4종 분류

| 종류 | 하는 일 | AI 에이전트 쓰임새 |
| --- | --- | --- |
| Virtual Proxy | 진짜 객체 생성을 미룸(지연 로딩) | 무거운 LLM 클라이언트를 첫 호출 때 초기화 |
| Protection Proxy | 권한 검사 후 통과시킴 | 유저 role에 따라 Tool 호출 허용/차단 |
| Remote Proxy | 원격에 있는 객체를 로컬처럼 쓰게 함 | 외부 API를 로컬 객체처럼 래핑 |
| Caching/Smart Proxy | 결과 캐시, 참조 카운트, 로깅 | LLM 응답 캐시, 토큰 비용 집계 |

## AI 에이전트 개발 예시: LLM 호출 프록시

LLM(Large Language Model, 대규모 언어 모델)을 직접 호출하면 요금, 속도, 중복 호출이 문제가 된다. 프록시로 캐시와 비용 로깅을 끼워 넣어 본다.

```python
from abc import ABC, abstractmethod

# 1. 공통 인터페이스 (Subject)
class LLMClient(ABC):
    @abstractmethod
    def chat(self, prompt: str) -> str: ...

# 2. 본체 (RealSubject)
class OpenAIClient(LLMClient):
    def chat(self, prompt: str) -> str:
        # 실제로 API 호출해서 돈 쓰는 친구
        return openai_call(prompt)

# 3. 대리인 (Proxy)
class CachedLLMClient(LLMClient):
    def __init__(self, real: LLMClient):
        self._real = real
        self._cache: dict[str, str] = {}
        self._cost = 0

    def chat(self, prompt: str) -> str:
        if prompt in self._cache:
            return self._cache[prompt]          # 캐시 hit, 본체 안 부름
        result = self._real.chat(prompt)         # 캐시 miss일 때만 본체 호출
        self._cache[prompt] = result
        self._cost += estimate_cost(prompt, result)
        return result

# 클라이언트는 본체 쓰는지 대리인 쓰는지 몰라도 됨
client: LLMClient = CachedLLMClient(OpenAIClient())
answer = client.chat("파이썬에서 리스트 뒤집는 법?")
```

같은 프롬프트가 두 번째로 들어오면 API 호출이 일어나지 않음. 본체(`OpenAIClient`) 코드는 손대지 않았는데 캐시 기능이 얹어짐.

## 또 다른 예시: Tool 권한 프록시 (Protection Proxy)

에이전트가 쓰는 Tool 중 위험한 것(파일 삭제, 결제 API)은 사용자 권한을 체크한 뒤에만 실행되게 해야 한다.

```python
class Tool(ABC):
    @abstractmethod
    def run(self, args: dict, user: "User") -> str: ...

class DeleteFileTool(Tool):
    def run(self, args, user):
        os.remove(args["path"])
        return "deleted"

class ProtectedTool(Tool):
    def __init__(self, real: Tool, required_role: str):
        self._real = real
        self._required = required_role

    def run(self, args, user):
        if user.role != self._required:
            raise PermissionError(f"{user.name}은 {self._required} 권한 없음")
        return self._real.run(args, user)

# 사용
tool = ProtectedTool(DeleteFileTool(), required_role="admin")
tool.run({"path": "/tmp/a"}, user=some_user)
```

`DeleteFileTool`은 권한 검사 로직을 몰라도 된다. 권한 정책이 바뀌면 `ProtectedTool`만 고친다.

## 왜 이렇게 하는가

1. **본체 수정 없이 기능 추가**: 캐시/권한/로깅은 본체의 책임이 아님. 분리해 두면 본체 코드가 깨끗해지고 테스트도 쉬워진다.
2. **투명성**: 클라이언트 코드는 인터페이스만 보고 쓰기 때문에, 프록시를 끼우거나 빼도 클라이언트 코드는 불변.
3. **SOLID 원칙의 OCP, SRP 적용**: 본체는 자기 책임(실제 일 처리)에만 집중(SRP), 부가 기능은 프록시로 확장(OCP). [[SOLID원칙]] 참고.

### 대안 대비 트레이드오프

| 접근 | 장점 | 단점 |
| --- | --- | --- |
| 본체에 직접 캐시 코드 박음 | 간단 | 책임 섞임, 캐시 정책 바꿀 때 본체 수정 |
| Proxy 패턴 | 책임 분리, 교체 쉬움 | 클래스 수 증가, 한 단계 우회 |
| Decorator 패턴 | 기능 "추가" 목적에 맞음 | 접근 제어 같은 "본체 대체" 뉘앙스는 약함 |

## Proxy vs Decorator vs Adapter 헷갈림 정리

이 셋은 "다른 객체를 감싸는 구조"라 자주 섞인다.

| 패턴 | 감싸는 목적 | 인터페이스 변화 | 비유 |
| --- | --- | --- | --- |
| Proxy | 본체 접근을 제어/대리 | 같음 | 비서 |
| Decorator | 본체 기능을 확장 | 같음 | 케이크 위 토핑 |
| Adapter | 다른 인터페이스로 변환 | 달라짐 | 110V to 220V 변환 플러그 |

## 직접 확인

파이썬이면 `functools.lru_cache`가 바로 Caching Proxy의 축약판이다. 데코레이터 하나 붙이면 함수 앞에 캐시 프록시가 생기는 셈.

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_embed(text: str):
    return openai_embed(text)

expensive_embed("hello")  # 실제 호출
expensive_embed("hello")  # 캐시 hit, API 안 부름
expensive_embed.cache_info()  # 히트/미스 카운트 확인
```

## 포함 관계 및 관련 패턴

- Proxy는 [[SOLID원칙]]의 **OCP(개방 폐쇄)** 와 **SRP(단일 책임)** 를 구조적으로 실현하는 장치 중 하나임.
- [[Iterator 패턴(순회)]], [[Command 패턴(요청의 객체화)]] 등 다른 패턴과 결합하면 "요청 큐에 쌓기 전 권한 체크" 같은 조합이 가능함.
- Decorator와의 차이는 "의도"다. 기능 추가가 목적이면 Decorator, 접근 제어/대리 호출이 목적이면 Proxy로 분류.

## 한마디 요약

> **"본체 앞에 비서를 하나 세워라. 비서가 거를 건 거르고, 기록할 건 기록하고, 진짜 중요한 것만 사장에게 연결해 준다."**

관련 문서:
- [[SOLID원칙]]
- [[Command 패턴(요청의 객체화)]]
- [[Mediator 패턴(중재자)]]
