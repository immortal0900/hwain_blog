---
생성날짜:
- 2026-04-21 12:40
마지막수정날짜:
- 2026-04-21-화요일 12:40
tags:
- 설계원칙
- SOLID
- SRP
- OOP
- AI에이전트
별칭:
- SRP
- Single Responsibility Principle
- 단일 책임 원칙
type:
- 자료수집
Area/Reasource:
Project:
---
# SRP (Single Responsibility Principle, 단일 책임 원칙)

> **"A module should be responsible to one, and only one, actor."**
> 하나의 모듈은 오직 하나의 액터(변경을 요구하는 사람/그룹)에 대해서만 책임을 져야 한다.
> Robert C. Martin, [The Single Responsibility Principle (2014)](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)

상위 개념: [[SOLID원칙]]
관련: [[OCP (개방 폐쇄 원칙)]], [[SOLID_ O와 D의 차이]]

---

## 1. 정의 다시 보기: "책임"의 오해

SRP를 "하나의 클래스는 하나의 일만 해야 한다"로 외우고 있는 사람이 많다. 이건 엄밀히 틀렸다. Uncle Bob이 2014년 블로그 글에서 직접 정정한 내용이 있다.

| 잘못된 이해 | 정정된 이해 |
| --- | --- |
| "한 가지 일(function)만 해라" | "한 가지 **변경 이유(reason to change)** 만 가져라" |
| "메서드를 잘게 쪼개라" | "변경을 요구하는 **사람(actor)** 기준으로 묶어라" |
| 기능 단위로 자르기 | 이해관계자(stakeholder) 단위로 자르기 |

> **한마디 요약**: SRP는 "기능"이 아니라 "누구 때문에 수정되는가"로 자른다.

### 왜 '사람' 기준인가?

Martin 원문: "This principle is about people. When you write a software module, you want to make sure that when changes are requested, those changes can only originate from a single person, or rather, a single tightly coupled group of people representing a single narrowly defined business function."

코드가 바뀌는 원인은 결국 누군가가 요구했기 때문이다. 그 요구자가 여러 명이면, 한 명의 요구를 반영하다가 다른 명의 요구사항이 깨진다. SRP는 이런 **"요구자 간 간섭"** 을 막는 원칙이다.

---

## 2. 일상 비유: 식당 홀 매니저

한 사람이 주방 요리 + 홀 서빙 + 계산대 + 식자재 발주까지 다 하는 식당을 상상해보자.

| 요구자(actor) | 요구 내용 | 충돌 |
| --- | --- | --- |
| 셰프 | "플레이팅 시간 줄여라" | 홀 서빙 루틴과 충돌 |
| 사장 | "원가 5% 줄여라" | 식자재 발주 루틴 수정 필요 |
| 단골 손님 | "카카오페이 결제 추가해달라" | 계산대 코드 수정 필요 |

한 사람이 네 영역을 겸하면, 셰프 요구를 반영하다가 손님 결제가 터진다. **사람마다 담당을 나누는 것**이 SRP의 본질이다.

---

## 3. 위반 신호 5가지

자기가 짠 클래스에 아래 증상이 있으면 SRP 위반을 의심해야 한다.

- [ ] 클래스 이름에 "and", "Manager", "Helper", "Util" 같은 애매한 단어가 들어간다
- [ ] 한 파일의 `import`가 DB, HTTP, 로깅, LLM SDK, 템플릿 엔진을 전부 포함한다
- [ ] `git log <파일>`을 찍었을 때 커밋 메시지가 "fix: DB migration", "refactor: LLM switch", "feat: Slack notify"처럼 도메인이 섞여 있다
- [ ] 단위 테스트를 짜려면 mock이 5개 이상 필요하다
- [ ] 한 메서드를 수정했는데 관련 없어 보이는 테스트가 깨진다

### 직접 확인하기

```bash
# 최근 20개 커밋의 수정 파일별 빈도 (Windows Git Bash 기준)
git log -n 20 --name-only --pretty=format: | sort | uniq -c | sort -rn | head
```

한 파일이 비정상적으로 자주 수정되고, 커밋 메시지가 제각각이면 그 파일은 SRP 위반 후보이다.

---

## 4. AI Agent 개발에서의 SRP

에이전트 코드는 특히 책임이 뭉치기 쉽다. "LLM 한 번 부르는" 흐름에 자연스럽게 여러 영역이 딸려오기 때문이다.

### 전형적 위반: 거대한 ChatAgent

```python
# ❌ 하나의 클래스, 다섯 명의 actor
class ChatAgent:
    def __init__(self):
        self.openai = openai.OpenAI(api_key=os.getenv("OPENAI_KEY"))
        self.db = psycopg2.connect(...)
        self.slack = SlackClient(...)

    def handle(self, user_msg: str, user_id: str) -> str:
        # 1) 프롬프트 조립
        system_prompt = "너는 친절한 상담원이다..."
        messages = [{"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_msg}]

        # 2) LLM 호출
        resp = self.openai.chat.completions.create(
            model="gpt-4o", messages=messages
        )
        answer = resp.choices[0].message.content

        # 3) 비용 계산
        cost = resp.usage.total_tokens * 0.00001

        # 4) DB 저장
        cur = self.db.cursor()
        cur.execute(
            "INSERT INTO conversations (user_id, msg, resp, cost) VALUES (%s,%s,%s,%s)",
            (user_id, user_msg, answer, cost),
        )
        self.db.commit()

        # 5) Slack 알림
        if cost > 0.1:
            self.slack.post(f"⚠️ 고비용 대화: ${cost}")

        # 6) 응답 포맷팅
        return f"[답변]\n{answer}\n\n(사용 토큰: {resp.usage.total_tokens})"
```

### 이 클래스의 actor 목록

| Actor (변경 요구자) | 바뀌면 건드려야 할 부분 |
| --- | --- |
| **Prompt 엔지니어** | system_prompt 문구 |
| **ML 엔지니어** | 모델명, temperature, 호출 방식 |
| **재무팀/FinOps** | 비용 단가, 알림 임계치 |
| **데이터 엔지니어** | DB 스키마, INSERT 쿼리 |
| **운영팀** | Slack 채널, 알림 포맷 |
| **프론트엔드** | 응답 포맷팅 문자열 |

**6개의 액터가 한 파일을 수정**한다. merge conflict가 쏟아지고, 테스트 하나 짜려고 DB/Slack/OpenAI를 전부 mock해야 한다.

### 리팩토링: 책임을 actor별로 자른다

```python
# ✅ 책임 분리
# prompt_builder.py  (Prompt 엔지니어 담당)
class PromptBuilder:
    def build(self, user_msg: str) -> list[dict]:
        return [
            {"role": "system", "content": "너는 친절한 상담원이다..."},
            {"role": "user", "content": user_msg},
        ]

# llm_client.py  (ML 엔지니어 담당)
class LLMClient:
    def __init__(self, client, model: str):
        self.client = client
        self.model = model

    def complete(self, messages: list[dict]) -> LLMResponse:
        resp = self.client.chat.completions.create(
            model=self.model, messages=messages
        )
        return LLMResponse(
            text=resp.choices[0].message.content,
            tokens=resp.usage.total_tokens,
        )

# cost_tracker.py  (FinOps 담당)
class CostTracker:
    RATE = 0.00001
    THRESHOLD = 0.1

    def calc(self, tokens: int) -> float:
        return tokens * self.RATE

    def is_expensive(self, cost: float) -> bool:
        return cost > self.THRESHOLD

# conversation_repo.py  (데이터 엔지니어 담당)
class ConversationRepository:
    def __init__(self, db):
        self.db = db

    def save(self, user_id: str, msg: str, resp: str, cost: float) -> None:
        cur = self.db.cursor()
        cur.execute(
            "INSERT INTO conversations (user_id, msg, resp, cost) VALUES (%s,%s,%s,%s)",
            (user_id, msg, resp, cost),
        )
        self.db.commit()

# alerter.py  (운영팀 담당)
class SlackAlerter:
    def __init__(self, client, channel: str):
        self.client = client
        self.channel = channel

    def alert_high_cost(self, cost: float) -> None:
        self.client.post(self.channel, f"⚠️ 고비용 대화: ${cost}")

# formatter.py  (프론트엔드 담당)
class ResponseFormatter:
    def format(self, answer: str, tokens: int) -> str:
        return f"[답변]\n{answer}\n\n(사용 토큰: {tokens})"

# chat_agent.py  (조립만 담당)
class ChatAgent:
    def __init__(self, prompt, llm, cost, repo, alerter, formatter):
        self.prompt = prompt
        self.llm = llm
        self.cost = cost
        self.repo = repo
        self.alerter = alerter
        self.formatter = formatter

    def handle(self, user_msg: str, user_id: str) -> str:
        messages = self.prompt.build(user_msg)
        resp = self.llm.complete(messages)
        cost = self.cost.calc(resp.tokens)
        self.repo.save(user_id, user_msg, resp.text, cost)
        if self.cost.is_expensive(cost):
            self.alerter.alert_high_cost(cost)
        return self.formatter.format(resp.text, resp.tokens)
```

### 리팩토링 후 얻은 것

| 변경 요구 | 수정할 파일 |
| --- | --- |
| "system prompt 문구 바꿔달라" | `prompt_builder.py` 한 곳 |
| "GPT-4o 대신 Claude 써보자" | `llm_client.py` 한 곳 (또는 새 LLMClient 구현체 추가) |
| "Slack 대신 Discord로 알림" | `alerter.py` 한 곳 |
| "비용 임계치 0.1 → 0.05" | `cost_tracker.py` 한 곳 |

**각 actor의 수정이 다른 actor의 코드를 건드리지 않는다.** merge conflict도 줄고, 테스트도 각자 고립된 상태로 짤 수 있다.

---

## 5. AI Agent 특화 가이드라인

### 5.1 LLM 호출 컴포넌트는 반드시 단독 분리

LLM 호출은 가장 자주 바뀌는 영역이다. 모델명 변경, SDK 버전업, 온도·토큰 파라미터 튜닝, 재시도·타임아웃 정책 등 actor가 많다. 반드시 단독 클래스(예: `LLMClient`, `ChatModel`)로 분리하고, 나머지 로직과 섞지 마라.

### 5.2 Prompt는 코드가 아니라 데이터로 취급

Prompt 문자열을 코드 중간에 박아두면 "프롬프트 엔지니어 actor"의 수정이 LLM 호출 로직과 엉킨다.

```python
# ❌ prompt가 코드에 박힘
def summarize(text):
    resp = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "너는 요약가이다. 3문장 이내로..."},
            {"role": "user", "content": text},
        ],
    )

# ✅ prompt를 별도 파일/클래스로
# prompts/summarize.md 또는 PromptRegistry.get("summarize")
```

### 5.3 Tool은 하나의 Tool = 하나의 클래스

Agent에 Tool(검색, 계산, 코드 실행 등)을 붙일 때, 하나의 Tool 클래스가 "검색 + 파싱 + 캐싱 + 로깅"을 다 하면 SRP 위반이다. `SearchTool`은 검색만 하고, 캐싱은 `CachedTool` 데코레이터로 감싸는 식이 깔끔하다.

### 5.4 Memory/Retrieval은 "읽기"와 "쓰기"가 다른 actor

벡터 DB에서 검색(읽기)과 저장(쓰기)은 다른 팀의 요구로 바뀐다.
- 읽기 쪽은 "검색 품질" actor(리콜/정확도 튜닝)
- 쓰기 쪽은 "파이프라인" actor(청킹 전략, 임베딩 모델 변경)

`VectorStore` 인터페이스는 유지하되, 내부 구현은 `VectorReader`와 `VectorWriter`로 나눠두면 CQRS(Command Query Responsibility Segregation) 패턴과 자연스럽게 맞물린다.

---

## 6. 다른 SOLID 원칙과의 포함 관계

- **SRP → OCP**: 책임이 분리돼 있어야 새 책임을 별도 클래스로 "확장"할 수 있다. 덩어리진 클래스에서는 확장이 곧 수정이 된다.
- **SRP → ISP**: 인터페이스를 작게 쪼개려면 먼저 구현 클래스 수준에서 책임이 나뉘어 있어야 한다.
- **SRP ← DIP**: 추상화에 의존하려면 구체 클래스가 단일 책임을 가져야 추상 인터페이스도 단순해진다.

```
    SRP (책임 분리)
      │
      ├─→ ISP (인터페이스 작게)
      │
      └─→ OCP (확장 가능)
```

SRP는 **모든 SOLID 원칙의 출발점**이다. SRP가 안 지켜지면 다른 네 가지도 실현 불가능하다.

---

## 7. 실전 체크리스트

AI Agent 코드를 짜면서 스스로 던질 질문.

- [ ] 이 클래스가 바뀔 이유를 3개 이상 말할 수 있는가? (있다면 쪼개라)
- [ ] 이 클래스 테스트하려고 mock이 몇 개 필요한가? (4개 이상이면 쪼개라)
- [ ] 클래스 이름이 "뭐 하는 놈"인지 한 단어로 말할 수 있는가? (못 하면 쪼개라)
- [ ] Prompt 엔지니어가 프롬프트만 바꾸고 싶을 때, 이 파일을 건드려야 하는가? (그렇다면 쪼개라)
- [ ] LLM Provider를 교체할 때, Agent 클래스를 수정해야 하는가? (그렇다면 쪼개라)

---

## 8. 흔한 오해와 함정

### 오해 1: "클래스를 최대한 잘게 쪼개는 게 SRP다"

아니다. 너무 쪼개면 오히려 "산탄총 수술(shotgun surgery)" 반대 방향의 안티패턴, **"기능 파편화(feature envy)"** 가 생긴다. 한 번의 논리적 변경이 10개 클래스를 순회하게 되면 그것도 문제.

**기준**: 같은 actor가 같은 이유로 바꿀 코드는 한 곳에 모으고, 다른 actor의 코드는 다른 곳에 둔다. actor 수만큼 나눠라.

### 오해 2: "한 클래스 = 한 메서드"

SRP는 메서드 수와 무관하다. `LLMClient`가 `complete()`, `stream()`, `embed()`를 다 갖고 있어도, 전부 "LLM 공급자 actor"가 바꾸는 영역이면 한 클래스에 두는 게 옳다.

### 오해 3: "SRP는 작은 프로젝트에선 오버엔지니어링"

소규모 스크립트에는 맞는 말이다. 하지만 AI Agent 프로젝트는 거의 항상 `LLM + DB + Vector Store + Tool + 외부 API` 조합이라 actor가 기본 5명 이상이다. 처음부터 SRP로 설계하는 편이 결국 빠르다.

---

## 9. 한마디 요약

> **"책임은 '기능'이 아니라 '누가 수정을 요구하느냐'로 나눈다. Actor가 겹치지 않으면 한 클래스, 겹치면 여러 클래스."**

관련 문서:
- [[SOLID원칙]]
- [[OCP (개방 폐쇄 원칙)]]
- [[SOLID_ O와 D의 차이]]
- [[고수준 모듈과 저수준 모듈(SOLID DIP)]]

참고 원문:
- Robert C. Martin, [The Single Responsibility Principle (cleancoder.com, 2014)](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)
- [Wikipedia: Single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)
