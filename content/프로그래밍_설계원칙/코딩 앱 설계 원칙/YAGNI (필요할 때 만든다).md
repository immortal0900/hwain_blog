---
생성날짜:
- 2026-04-21 13:00
마지막수정날짜:
- 2026-04-21-화요일 13:00
tags:
- 설계원칙
- 코딩원칙
- YAGNI
- XP
- 익스트림프로그래밍
- AI에이전트
- 과설계방지
별칭:
- YAGNI
- You Aren't Gonna Need It
- 지금필요없음
- 필요할때만든다
type:
- 자료수집
Area/Reasource:
Project:
---
# YAGNI (You Aren't Gonna Need It, 지금 필요 없는 기능은 미리 만들지 마라)

> **"Always implement things when you actually need them, never when you just foresee that you need them."**
> 실제로 필요해졌을 때 구현해라. "나중에 필요할 것 같다"는 예측만으로는 만들지 마라.
> Ron Jeffries, Extreme Programming 공동 창시자
> Martin Fowler, [Yagni (martinfowler.com, 2015)](https://martinfowler.com/bliki/Yagni.html)

상위 개념: [[코딩 앱 설계 원칙_index]]
관련: [[KISS]], [[DRY]], [[SRP (단일 책임 원칙)]], [[OCP (개방 폐쇄 원칙)]]

---

## 한눈에 보기

| 항목      | 내용                                                  |
| ------- | --------------------------------------------------- |
| 분류      | 코딩 원칙 (Extreme Programming 출신)                      |
| 한 줄 정의  | 지금 안 쓰는 기능은 미리 만들지 않는다                              |
| 핵심 대상   | presumptive feature(추측성 기능, 아직 실사용되지 않는 선제 코드)      |
| 해결 문제   | 과설계(over-engineering), 추측 실패로 인한 코드 유지 부담, 기회비용 낭비  |
| 관련 원칙   | KISS, SRP, OCP, Lean Startup의 "Build-Measure-Learn" |
| AI Agent 적용 포인트 | LLM 추상화 레이어, Tool registry, 멀티 Provider, 캐싱, 스트리밍 이중 모드 |

> **한마디 요약**: "예측이 아니라 실제 요구가 발생했을 때 코드를 추가한다. 예측은 거의 틀린다."

---

## 1. YAGNI의 정확한 의미

### 1.1 원문이 말하는 범위

YAGNI는 1990년대 후반 Extreme Programming(XP, 익스트림 프로그래밍, Kent Beck이 주도한 애자일 소프트웨어 개발 방법론) 커뮤니티에서 나온 구호다. Ron Jeffries의 원문은 단순하다. "실제 필요가 있을 때만 구현한다."

Martin Fowler는 2015년 [Yagni](https://martinfowler.com/bliki/Yagni.html) 글에서 범위를 명확히 했다.

| 흔한 오해 | 실제 의미 |
| --- | --- |
| "테스트도 쓰지 마라" | 아니다. 테스트는 코드 수정 가능성을 유지하므로 YAGNI 대상이 아님 |
| "리팩토링 하지 마라" | 아니다. 리팩토링은 오히려 YAGNI를 가능하게 하는 전제 조건 |
| "지금 당장 쓰는 코드만 써라" | 아니다. 현재 기능을 구현하기 위한 내부 구조는 자유롭게 설계 가능 |
| "설계 하지 마라" | 아니다. "아직 요구되지 않은 future feature"만 미루라는 뜻 |

> Fowler 원문: "Yagni only applies to capabilities built into the software to support a presumptive feature, it does not apply to effort to make the software easier to modify."

**핵심 대상은 "presumptive feature"** 다. 아직 쓰이지 않는 기능을 위해 선제적으로 박아둔 코드. 이게 YAGNI가 겨냥하는 정확한 타깃임.

### 1.2 presumptive feature란?

presumptive feature(추측성 기능, 아직 실사용되지 않는데 "곧 필요할 것"이라는 예측만으로 미리 심어둔 코드 경로)는 세 가지 스펙트럼으로 나뉜다.

1. **아예 안 쓰이는 경우**: 예측이 완전히 틀려서 코드가 영원히 죽어있음
2. **부분만 맞는 경우**: 일부는 쓰이지만 대부분 덜어내야 함
3. **"맞는 기능을 틀리게 만든" 경우**: 미래 요구사항을 미리 구현했으나, 실제로 기능이 필요해질 때쯤엔 이해가 바뀌어서 전체를 다시 써야 함

세 경우 모두 손해다. 가장 흔한 건 3번이다. 6개월 전의 본인이 예측한 요구와 지금 실제 요구가 일치하는 경우가 거의 없기 때문.

---

## 2. 일상 비유: 이사 올 때 "언젠가 쓸 것 같은" 물건

새 집으로 이사 갈 때 두 가지 선택지가 있다.

| 선택 | 이사 당일 | 3개월 후 | 1년 후 |
| --- | --- | --- | --- |
| A: 현재 쓰는 물건만 가져감 | 이삿짐 적음, 짐 정리 빠름 | 필요한 물건 새로 삼 (싸고 취향 맞음) | 공간 여유 있음 |
| B: "언젠가 쓸 것 같은" 물건 전부 가져감 | 이삿짐 2배, 비용 2배 | 그 물건 여전히 안 씀 | 창고 차지, 청소 때 매번 걸리적거림 |

B의 물건은 대부분 3년 뒤 버린다. 버릴 때 "왜 저걸 이고 왔지" 하는 후회가 남는 것이 **cost of carry**(보유 비용)다.

코드도 똑같다. 안 쓰는 인터페이스, 안 쓰는 설정 옵션, 안 쓰는 추상화 레이어는 전부 "이삿짐에 끼어온 미사용 물건"이다.

---

## 3. Fowler가 말한 4가지 비용

Martin Fowler는 presumptive feature가 발생시키는 비용을 4가지로 분해했다. AI Agent 개발에서 이 4가지가 어떻게 나타나는지 같이 본다.

### 3.1 cost of build (구축 비용)

**정의**: 실제로 안 쓸지도 모르는 기능을 분석하고, 코딩하고, 테스트하는 데 들어가는 즉시 비용.

**Agent 예시**: "나중에 Claude, Gemini, Llama도 써야 할 수 있으니까" 하고 `LLMProvider` 추상 베이스 클래스 + `OpenAIProvider` + `AnthropicProvider` + `GeminiProvider` 껍데기를 미리 만드는 경우. 당장은 OpenAI만 쓰는데 4개 클래스를 유지해야 함.

### 3.2 cost of delay (지연 비용)

**정의**: 추측성 기능에 시간을 써서 "실제 필요한 기능"의 출시가 늦어지는 기회비용.

**Agent 예시**: 상담 Agent MVP(Minimum Viable Product, 최소 기능 제품)를 금요일에 출시해야 하는데 "토큰 비용 트래킹 대시보드"를 미리 만드느라 출시가 다음 주로 밀림. 그 한 주 동안 사용자 피드백을 못 얻는 것이 지연 비용.

### 3.3 cost of carry (보유 비용)

**정의**: 추측성 코드가 거기 있음으로 인해, 이후 모든 기능 추가가 느려지는 지속 비용.

**Agent 예시**: 안 쓰는 `MultiTenantContext` 객체가 함수 시그니처 곳곳에 끼어있으면, 이후 모든 Tool 추가 시 이 파라미터를 전달·무시하는 코드를 매번 써야 함. CI 빌드 시간, 타입 체크 시간, 코드 리딩 시간이 전부 늘어남.

### 3.4 cost of repair (수리 비용)

**정의**: 미리 만든 코드가 실제 요구와 어긋날 때, 그걸 버리고 다시 쓰거나 고치는 비용. 보통 4가지 중 가장 큼.

**Agent 예시**: 6개월 전 "RAG(Retrieval-Augmented Generation, 검색 증강 생성)를 써야 할 것"이라 예상하고 `VectorStore` 인터페이스와 청킹 파이프라인을 다 깔아뒀다. 그런데 실제로 필요해졌을 때는 GraphRAG가 표준이 되어있고, 미리 만든 구조는 전부 재작성해야 함.

### 3.5 4가지 비용 요약표

| 비용 | 언제 발생 | AI Agent 전형 사례 |
| --- | --- | --- |
| **Build** | 추측성 코드 작성 시점 | 쓰지도 않을 LLMProvider 추상화 4개 |
| **Delay** | MVP/출시 시점 | "토큰 대시보드"에 시간 써서 MVP 한 주 지연 |
| **Carry** | 코드 수명 내내 | 안 쓰는 MultiTenantContext가 모든 함수 시그니처에 끼어있음 |
| **Repair** | 요구가 실제로 생겼을 때 | 6개월 전 만든 RAG 구조를 GraphRAG에 맞춰 전면 재작성 |

---

## 4. AI Agent 개발에서의 YAGNI

### 4.1 왜 Agent 코드가 특히 취약한가

AI Agent는 다음 특성 때문에 과설계(over-engineering) 함정에 유독 잘 빠진다.

1. **LLM 생태계 변화가 빠름**: 3개월 전 베스트 프랙티스가 지금은 구식. 선제 추상화 수명이 짧음
2. **"확장성" 환상**: 프레임워크(LangChain, LangGraph, LlamaIndex) 예제가 전부 "나중에 교체 가능한" 구조로 되어있어서 따라하기 쉬움
3. **Tool, Memory, Retrieval 등 조합 가짓수가 많음**: "다 지원해야 할 것 같다"는 착각
4. **프롬프트·모델 실험이 중심**: 실제 가치는 프롬프트·평가 루프에 있는데, 인프라 추상화에 시간을 뺏김

### 4.2 전형적 위반: 한 번도 안 쓸 LLM Provider 추상화

```python
# ❌ OpenAI 하나만 쓰는데 4개 Provider 껍데기
from abc import ABC, abstractmethod

class LLMProvider(ABC):
    @abstractmethod
    def complete(self, messages: list[dict], **kwargs) -> str: ...

    @abstractmethod
    def stream(self, messages: list[dict], **kwargs): ...

    @abstractmethod
    def embed(self, texts: list[str]) -> list[list[float]]: ...

class OpenAIProvider(LLMProvider):
    def __init__(self, api_key: str, model: str = "gpt-4o"):
        self.client = openai.OpenAI(api_key=api_key)
        self.model = model
    # ... 구현 ...

class AnthropicProvider(LLMProvider):   # 아직 안 씀
    def complete(self, messages, **kwargs):
        raise NotImplementedError

class GeminiProvider(LLMProvider):      # 아직 안 씀
    def complete(self, messages, **kwargs):
        raise NotImplementedError

class LlamaProvider(LLMProvider):       # 아직 안 씀
    def complete(self, messages, **kwargs):
        raise NotImplementedError

# 호출 쪽
provider: LLMProvider = OpenAIProvider(api_key=os.getenv("OPENAI_KEY"))
answer = provider.complete(messages)
```

**이 코드의 문제**:
- `stream()`, `embed()`는 당장 안 쓰는데 추상 메서드로 강제됨
- Anthropic/Gemini/Llama 클래스가 `NotImplementedError`만 뱉는 껍데기로 존재
- 새 메서드 하나 추가하려면 4개 클래스를 전부 고쳐야 함 (cost of carry)
- 6개월 뒤 실제로 Claude를 붙일 때는, 그 사이에 Anthropic SDK API가 바뀌어서 지금 만든 껍데기와 안 맞음 (cost of repair)

### 4.3 YAGNI 적용: 지금 쓰는 것만 쓴다

```python
# ✅ 지금 쓰는 OpenAI만 직접 호출
import os
import openai

client = openai.OpenAI(api_key=os.getenv("OPENAI_KEY"))

def complete(messages: list[dict], model: str = "gpt-4o") -> str:
    resp = client.chat.completions.create(model=model, messages=messages)
    return resp.choices[0].message.content

# 호출 쪽
answer = complete(messages)
```

**나중에 Claude를 실제로 붙이게 되면** 그때 추상화한다. 그 시점엔 이미 Anthropic SDK의 실제 shape을 알고 있고, 두 Provider의 공통점/차이점을 "경험"으로 알기 때문에 추상화가 정확해진다.

```python
# 🔄 실제 필요가 생긴 시점에 리팩토링
def complete_openai(messages: list[dict]) -> str: ...
def complete_anthropic(messages: list[dict]) -> str: ...

def complete(messages: list[dict], provider: str = "openai") -> str:
    if provider == "openai":
        return complete_openai(messages)
    if provider == "anthropic":
        return complete_anthropic(messages)
    raise ValueError(provider)
```

작은 분기로 시작해서, 3~4개 Provider가 될 때 비로소 인터페이스로 추상화한다. 이게 YAGNI가 말하는 "필요할 때 리팩토링"의 실제 흐름이다.

---

## 5. AI Agent 특화 가이드라인

### 5.1 "멀티 모델 지원" 선제 설계 금지

**함정**: 첫 프로토타입부터 모델 스위칭 인터페이스를 갖추려 함.

**원칙**: 1개 모델로 시작. 실제로 두 번째 모델을 붙일 때 추상화. 그 전엔 `gpt-4o` 문자열 직접 박아두는 게 낫다.

**예외**: 이미 두 모델을 "동시에" 써야 한다는 비즈니스 요구가 확정됐다면 그건 presumptive가 아니라 genuine need다. 이땐 추상화해도 됨.

### 5.2 Tool registry 선제 구축 금지

**함정**: Tool 2~3개 쓰면서 `ToolRegistry`, `ToolMetadata`, `ToolVersioning` 같은 구조를 먼저 만듦.

**원칙**: Tool 2~3개일 땐 그냥 리스트면 충분하다.

```python
# ❌ Tool 2개인데 registry 패턴
class ToolRegistry:
    def __init__(self): self._tools = {}
    def register(self, tool: Tool): self._tools[tool.name] = tool
    def get(self, name: str) -> Tool: return self._tools[name]
    def list_by_category(self, cat: str): ...
    def list_by_version(self, v: str): ...

# ✅ Tool 2개면 리스트로 충분
tools = [search_tool, calc_tool]
```

Tool이 10개를 넘기기 시작하면, 그때 registry·카테고리·버전 관리가 실제로 가치가 있다. 2~3개일 땐 딕셔너리 하나가 registry보다 낫다.

### 5.3 멀티 유저/멀티 테넌트 선제 대응 금지

**함정**: 혼자 쓰는 MVP 단계에서 `tenant_id`, `user_context`, `organization_id`를 모든 함수 시그니처에 집어넣음.

**원칙**: 실제 두 번째 사용자가 생길 때까지는 single-user로 둔다. DB 스키마에 `user_id` 컬럼만 있으면 충분. 그 위의 격리 레이어는 필요해지면 만든다.

### 5.4 Streaming/Batch 이중 모드 선제 지원 금지

**함정**: 동기 호출만 쓰는데 "나중에 스트리밍 필요할지 모르니까" `AsyncIterator` 기반으로 전부 감쌈.

**원칙**: UI가 스트리밍을 "요구"할 때 추가한다. 요구 전까지는 동기 호출이 디버깅·테스트·로깅 전부 쉽다.

### 5.5 선제 캐싱 레이어 금지

**함정**: LLM 호출 비용이 걱정돼서 Redis 기반 semantic cache를 첫날부터 깖.

**원칙**: 먼저 실제 호출 패턴·중복률을 측정한다. 중복률 5% 미만이면 캐싱 구조는 cost of carry만 늘린다. 측정 도구(tiktoken, LangSmith, Langfuse) 붙이는 게 선제 캐시보다 먼저다.

### 5.6 Eval/테스트 선제 설계는 YAGNI 대상이 아님

**반례**: "테스트도 YAGNI니까 나중에 쓰자"는 오해.

**원칙**: Eval 셋, 회귀 테스트, 로깅, 리팩토링은 Fowler가 명시적으로 "YAGNI 예외"라 짚은 영역이다. 이들은 presumptive feature가 아니라 **"코드를 바꿀 수 있는 상태로 유지하는 인프라"** 다. 오히려 YAGNI를 가능하게 해주는 전제다.

LLM Agent에서 특히 중요한 건:
- **Prompt regression eval**: 프롬프트 수정 시 golden set으로 성능 회귀 감지
- **Structured logging**: 호출 입출력·토큰·latency 기록
- **Deterministic test mode**: `temperature=0` + 고정 seed로 재현 가능한 테스트

이 셋은 첫날 갖춘다. YAGNI 적용 대상 아님.

---

## 6. YAGNI vs KISS vs DRY: 세 원칙의 포함 관계

```
        [과설계 / 복잡성 문제]
              │
    ┌─────────┼─────────┐
    │         │         │
  YAGNI     KISS       DRY
(시간축)   (복잡도축) (중복축)
```

세 원칙은 "뭘 만들지 말라"를 각기 다른 축으로 말한다.

| 원칙 | 축 | 질문 | Agent 예시 |
| --- | --- | --- | --- |
| **YAGNI** | 시간 | "지금 필요한가?" | "나중에 필요할 것 같은" LLM 추상화 지금 만들지 않음 |
| **KISS** | 복잡도 | "이게 가장 단순한 구현인가?" | Agent 3개 역할을 StateMachine + EventBus + Orchestrator로 쪼개지 않음 |
| **DRY** | 중복 | "같은 로직이 여러 곳에 있는가?" | OpenAI API 키 로딩 로직이 5개 파일에 복붙되어 있으면 한 곳으로 |

**셋이 충돌하기도 한다**: "나중에 쓸지 모를 중복 제거를 위해 지금 추상화"는 DRY와 YAGNI의 충돌이다. 이땐 YAGNI가 이긴다. 중복은 3번째 등장할 때 합치는 게 보통 정답(Rule of Three).

**세 원칙의 공통 상위**: 세 가지 모두 **"복잡도는 자산이 아니라 부채"** 라는 공통 전제에서 출발한다.

---

## 7. YAGNI가 통하지 않는 상황

YAGNI는 만능이 아니다. 무조건 "나중에 만들자"로 가면 오히려 더 큰 비용이 든다. 다음 두 가지는 **선제 설계가 정당화**된다.

### 7.1 되돌리기 어려운 결정(irreversible decision)

**예시**: 공개 API의 URL 스킴, DB의 primary key 타입, 라이선스 선택, 외부에 뿌려진 SDK의 퍼블릭 시그니처.

이들은 한번 릴리즈되면 수천 명의 클라이언트가 의존하게 돼서, "나중에 바꾸면 되지"가 불가능하다. YAGNI가 전제로 하는 "코드를 쉽게 바꿀 수 있다"가 깨진다. 처음부터 신중히 설계해야 한다.

### 7.2 비용이 확정된 요구

**예시**: 법적·컴플라이언스 요건. "GDPR 삭제 요청 처리"를 "나중에 필요할 때" 만들면 이미 법적 위험에 노출됨.

이때는 요구가 presumptive가 아니라 **genuine**하다. YAGNI 대상 아님.

### 7.3 판별 기준

| 질문 | YES → YAGNI 적용 | NO → 선제 설계 |
| --- | --- | --- |
| 실제 요구가 확정됐는가? | NO | YES |
| 나중에 리팩토링 가능한가? | YES | NO |
| 테스트로 회귀 잡을 수 있는가? | YES | NO |
| 외부에 노출된 인터페이스인가? | NO | YES |

---

## 8. 실전 체크리스트

Agent 코드 PR을 올리기 전 스스로 던질 질문.

- [ ] 이 추상 클래스의 구현체가 지금 **2개 이상** 있는가? (1개면 추상화 치워라)
- [ ] 이 설정 옵션이 실제로 바뀐 적 있는가? (없으면 상수로 박아라)
- [ ] 이 파라미터를 지금 호출부에서 **쓰고 있는가**? (안 쓰면 지워라)
- [ ] 이 인터페이스가 외부에 공개되는가? (아니면 내부 추상화 더 미뤄라)
- [ ] "나중에 ~할 수도 있으니까"라는 이유로 추가한 코드가 있는가? (있으면 삭제 후보)
- [ ] Eval/테스트/로깅은 갖췄는가? (이건 YAGNI 예외. 없으면 지금 추가)

---

## 9. 직접 확인하기

자기 레포에서 "지금 안 쓰는 코드"를 찾는 간단한 방법.

```bash
# 사용처 없는 심볼 검출 (Python 기준, vulture 사용)
pip install vulture
vulture src/ --min-confidence 80

# import는 되어 있지만 실제 사용이 없는 경우
ruff check --select F401 src/

# 최근 6개월간 한 번도 수정되지 않은 파일 (죽은 코드 후보)
git log --since="6 months ago" --name-only --pretty=format: \
    | sort -u > recently_touched.txt
git ls-files | sort -u > all_files.txt
comm -23 all_files.txt recently_touched.txt | head -20
```

결과 목록에 **Provider/Adapter/Factory/Strategy** 같은 이름의 클래스가 잡힌다면 YAGNI 위반 후보이다. 한 번도 안 쓰이는 추상화일 가능성이 높음.

---

## 10. 흔한 오해와 함정

### 오해 1: "YAGNI = 설계하지 마라"

아니다. 현재 기능의 내부 구조는 여전히 잘 설계해야 한다. YAGNI가 겨냥하는 건 "아직 요구되지 않은 future feature"뿐이다. 지금 기능을 위한 SRP, OCP는 정상 적용해야 함.

### 오해 2: "나중에 바꾸면 더 비쌀 것"

보통 반대다. 지금 모르는 요구에 맞춰 추측 설계하면 평균적으로 틀린다. 틀린 설계를 깨고 다시 만드는 비용이, 처음부터 단순하게 만들어두고 실제 요구 때 리팩토링하는 비용보다 크다. **"리팩토링 가능한 상태"** 를 유지하는 것이 핵심.

### 오해 3: "XP에서만 통하는 이야기"

아니다. Lean Startup의 "Build-Measure-Learn", Amazon의 "two-way door vs one-way door decision", Shape Up의 "appetite" 같은 다른 방법론도 본질적으로 같은 주장을 다른 각도에서 하고 있다. 추측은 틀린다는 경험칙은 방법론 불문 공통.

### 오해 4: "AI Agent는 복잡하니까 처음부터 큰 구조가 필요하다"

Agent의 **본질적 복잡성**(프롬프트 설계, 평가, 관찰성)과 **우발적 복잡성**(선제 추상화, 미사용 인터페이스)을 구분해야 한다. YAGNI가 줄이는 건 후자다. 전자는 처음부터 투자해야 함.

---

## 11. 한마디 요약

> **"예측하지 말고 반응해라. 실제 요구가 생겼을 때, 그 요구의 실제 모습에 맞춰 코드를 추가한다. 6개월 전의 내 예측은 거의 항상 틀린다."**

AI Agent에서의 적용 한 줄: **"일단 동작하게 → Eval·로깅 갖춘 채로 방치 → 실제 요구 발생 → 그때 리팩토링."**

---

## 관련 문서

- [[코딩 앱 설계 원칙_index]]
- [[KISS]]
- [[DRY]]
- [[SRP (단일 책임 원칙)]]
- [[OCP (개방 폐쇄 원칙)]]

## 참고 원문

- Martin Fowler, [Yagni (martinfowler.com, 2015)](https://martinfowler.com/bliki/Yagni.html)
- Ron Jeffries, [You're NOT gonna need it! (ronjeffries.com)](https://ronjeffries.com/xprog/articles/practices/pracnotneed/)
- [Wikipedia: You aren't gonna need it](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)
- Kent Beck, "Extreme Programming Explained" (2nd ed., 2004)
