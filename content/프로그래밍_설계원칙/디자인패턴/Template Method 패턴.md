---
생성날짜:
- 2026-04-21 13:15
마지막수정날짜:
- 2026-04-21-화요일 13:15
tags:
- 설계원칙
- 디자인패턴
- 행동패턴
- AI에이전트
별칭:
- Template Method
- 템플릿 메서드 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# Template Method 패턴

**"알고리즘의 큰 흐름(뼈대)은 부모 클래스에 고정하고, 바뀌어야 할 세부 단계만 자식 클래스가 갈아끼우는 행동 패턴."**

Template Method 패턴(템플릿 메서드 패턴, 원형 절차 메서드)은 GoF 행동(Behavioral) 패턴 중 하나다. "공통 절차는 한 번만 쓰고, 변형 부분만 자식마다 다르게 구현"하는 고전적 OOP 재사용 기법이다. 함수형 언어에서는 "고차함수로 콜백을 받는 구조"와 본질이 같지만, OOP에서는 상속과 오버라이딩으로 푼다.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 행동(Behavioral) 패턴 |
| 목적 | 알고리즘 구조는 그대로, 단계별 구현만 서브클래스가 바꾸게 |
| 핵심 질문 | "여러 클래스에서 80% 똑같은 절차가 반복되는가?" |
| 대안 | Strategy(행동 자체를 외부 객체로 주입), 고차함수 |
| 트레이드오프 | 상속 기반이라 결합도가 높음. 런타임 교체 불가 |

> **한마디 요약**: "순서는 부모가, 디테일은 자식이."

---

## 일상 비유: 라면 레시피와 커피 드립

라면 끓이는 절차는 보편적이다.

1. 물 550ml 끓인다
2. 면과 스프를 넣는다
3. 4분 30초 기다린다
4. 추가 재료를 넣는다 (계란, 파, 치즈 등 취향)
5. 불을 끄고 그릇에 담는다

신라면, 진라면, 짜파게티 모두 위 5단계를 따른다. 단 4번 "추가 재료"와 2번 "스프의 종류"만 다르다. 뼈대는 공용, 세부만 변형 = 템플릿 메서드 그 자체다.

Java의 `HttpServlet` (서블릿 요청 처리 흐름), Python의 `unittest.TestCase` (setUp → test → tearDown), Spring의 `JdbcTemplate` 같은 클래스가 모두 이 패턴의 사례다.

---

## 구조

| 역할 | 설명 |
| --- | --- |
| Abstract Class | 템플릿 메서드(공통 절차) + 추상 메서드(단계별 구현 자리) 제공 |
| Concrete Class | 추상 메서드를 상황에 맞게 구현 |
| 템플릿 메서드 | 절차를 고정한 `final`/`@final` 메서드 (자식이 오버라이드 금지) |
| Hook Method | 선택적으로 오버라이드 가능한 기본 구현 메서드 (후크, 갈고리) |

단계 분해:
1. **1단계**: 공통 절차를 가진 메서드를 부모 클래스에 둔다
2. **2단계**: 변형이 생기는 구간만 추상 메서드 또는 Hook으로 노출
3. **3단계**: 자식 클래스가 필요한 단계만 오버라이드
4. **4단계**: Client는 `run()` 한 번 호출, 내부적으로 추상 메서드들이 자식 버전으로 분기 실행

---

## AI 에이전트 개발 예시 1: BaseAgent의 실행 루프

대부분의 AI 에이전트는 다음 고정 흐름을 따른다.

```
validate_input → build_prompt → call_llm → parse_output → run_tools → format_response
```

각 프로젝트마다 `build_prompt`, `parse_output`만 다르고 나머지는 공통이다.

### 나쁜 예: 복붙 에이전트

```python
class ResearchAgent:
    def run(self, q):
        if not q: raise ValueError("empty")
        prompt = f"[Research] {q}"
        resp = self.llm.chat([{"role": "user", "content": prompt}])
        tools_called = self._parse_tools(resp.text)
        results = [self.tools[t.name].run(t.args) for t in tools_called]
        return {"answer": resp.text, "tool_results": results}


class SupportAgent:
    def run(self, q):
        if not q: raise ValueError("empty")
        prompt = f"[Support] {q}"   # 여기만 다름
        resp = self.llm.chat([{"role": "user", "content": prompt}])
        tools_called = self._parse_tools(resp.text)  # 동일
        results = [self.tools[t.name].run(t.args) for t in tools_called]  # 동일
        return {"answer": resp.text, "tool_results": results}  # 동일
```

5~6줄 중 1줄만 다른데 전체가 복붙된다. 절차가 바뀌면 두 군데 수정.

### 좋은 예: Template Method 적용

```python
from abc import ABC, abstractmethod

class BaseAgent(ABC):
    def __init__(self, llm, tools: dict):
        self.llm = llm
        self.tools = tools

    # 템플릿 메서드: 자식이 오버라이드하면 안 됨
    def run(self, user_input: str) -> dict:
        self._validate(user_input)
        prompt = self.build_prompt(user_input)
        response = self._call_llm(prompt)
        tool_calls = self.parse_output(response)
        tool_results = self._run_tools(tool_calls)
        return self.format_response(response, tool_results)

    # 공통 구현
    def _validate(self, user_input: str) -> None:
        if not user_input or not user_input.strip():
            raise ValueError("user_input is empty")

    def _call_llm(self, prompt: str):
        return self.llm.chat([{"role": "user", "content": prompt}])

    def _run_tools(self, tool_calls: list) -> list:
        return [self.tools[tc.name].run(tc.args) for tc in tool_calls]

    # Hook Method: 기본 제공, 자식이 원하면 변경
    def format_response(self, response, tool_results) -> dict:
        return {"answer": response.text, "tool_results": tool_results}

    # 추상 메서드: 자식이 반드시 구현
    @abstractmethod
    def build_prompt(self, user_input: str) -> str: ...

    @abstractmethod
    def parse_output(self, response) -> list: ...


class ResearchAgent(BaseAgent):
    def build_prompt(self, q):
        return f"[Research Mode] Be rigorous.\nQuestion: {q}"

    def parse_output(self, response):
        return parse_json_tool_calls(response.text)


class SupportAgent(BaseAgent):
    def build_prompt(self, q):
        return f"[Support Mode] Be warm and concise.\nUser: {q}"

    def parse_output(self, response):
        return parse_xml_tool_calls(response.text)

    # Hook 오버라이드: 답변 포맷만 커스터마이즈
    def format_response(self, response, tool_results):
        base = super().format_response(response, tool_results)
        base["channel"] = "support"
        return base
```

Client 코드는 어느 에이전트든 똑같이 부른다.

```python
agent = SupportAgent(llm, tools={"search": SearchTool()})
print(agent.run("How do I reset my password?"))
```

---

## AI 에이전트 개발 예시 2: Document Processor

ETL(Extract-Transform-Load, 데이터 추출-변환-적재) 파이프라인도 전형적인 Template Method 자리다.

```python
class BaseIngestor(ABC):
    def ingest(self, source: str) -> int:
        raw = self.fetch(source)
        docs = self.parse(raw)
        cleaned = [self.clean(d) for d in docs]
        for d in cleaned:
            vec = self.embed(d.text)
            self.store.upsert([(d.id, vec, d.metadata)])
        return len(cleaned)

    @abstractmethod
    def fetch(self, source: str) -> bytes: ...

    @abstractmethod
    def parse(self, raw: bytes) -> list: ...

    def clean(self, doc):
        """Hook: 기본 클리너. 필요 시 자식이 재정의."""
        doc.text = doc.text.strip()
        return doc


class PDFIngestor(BaseIngestor):
    def fetch(self, path): return open(path, "rb").read()
    def parse(self, raw): return pdf_to_docs(raw)


class WebCrawlerIngestor(BaseIngestor):
    def fetch(self, url): return requests.get(url).content
    def parse(self, raw): return html_to_docs(raw)
    def clean(self, doc):
        doc = super().clean(doc)
        doc.text = remove_nav_boilerplate(doc.text)
        return doc
```

---

## 실제 라이브러리에서의 Template Method

| 라이브러리 | 템플릿 메서드 |
| --- | --- |
| Python `unittest.TestCase` | `setUp` → `test_xxx` → `tearDown` 흐름 고정, `test_xxx`만 오버라이드 |
| Django `View.dispatch()` | HTTP 메서드 라우팅 고정, `get/post/put`만 오버라이드 |
| LangChain `Chain._call()` | 입력 검증, 트레이싱, 메모리 저장 고정, 실제 로직만 자식이 |
| Java Servlet `HttpServlet.service()` | 요청 라우팅 고정, `doGet/doPost`만 구현 |
| Spring `JdbcTemplate.execute()` | 커넥션 획득/반환 고정, 쿼리만 콜백으로 |

---

## Template Method vs Strategy

Template Method를 공부하면 거의 반드시 Strategy가 따라온다. 둘은 같은 문제를 "상속이냐 합성이냐"로 다르게 푼다.

| 항목 | Template Method | Strategy |
| --- | --- | --- |
| 메커니즘 | 상속 + 오버라이딩 | 합성 + 객체 주입 |
| 교체 시점 | 컴파일 타임(서브클래스 선택) | 런타임(다른 Strategy 객체 주입) |
| 결합도 | 높음 (Abstract Class와 묶임) | 낮음 |
| 여러 축 조합 | 다중상속/믹스인 필요 | 그냥 여러 필드로 주입 |
| 언제 유리 | 절차가 거의 같고 변형이 고정적 | 변형축이 여러 개, 런타임에 바뀜 |

**경험칙**: 축이 1개면 Template Method, 축이 2개 이상이거나 런타임 교체가 필요하면 Strategy.

---

## 직접 확인해 보기

중복 코드 탐지 도구를 돌리면 된다.

```bash
pip install pylint
pylint --disable=all --enable=duplicate-code src/
```

80% 이상 겹치는 클래스 쌍이 나오면 공통 조상과 추상 메서드로 리팩터링 대상이다.

---

## 왜 이렇게 하는가

Template Method의 설계 의도와 장점:

- **코드 중복 제거**: 공통 절차 한 벌로 유지
- **변경 지점의 명시**: "어디만 바꾸면 되는가"가 추상 메서드 목록으로 드러남
- **프레임워크-애플리케이션 분리**: 프레임워크가 절차를, 앱이 구현을 담당 (Hollywood Principle, "너는 부르지 마. 내가 부를게")

트레이드오프:
- 상속 기반이라 결합도가 높음. 부모 절차가 바뀌면 모든 자식 영향
- 변형축이 2개 이상이면 클래스 폭발(Class Explosion) 위험 (`ResearchAgentForEnterprise`, `ResearchAgentForFreeTier`, `SupportAgentForEnterprise`, ...)
- 런타임에 절차 자체를 바꾸고 싶으면 부적합. Strategy를 쓰는 게 낫다

---

## 포함 관계

- **Template Method ⊂ 리스코프 치환 원칙(LSP)의 영향권**: 자식이 부모 절차의 사전/사후 조건을 깨면 호출자가 망가진다. [[SOLID원칙]] LSP 절 참고
- **Template Method + Strategy**: 템플릿 내부의 한 단계를 Strategy로 주입 받게 하면 두 패턴의 장점을 섞을 수 있음
- **Template Method + State**: 템플릿의 단계 중 하나에서 State에 따라 분기하는 조합도 자주 쓴다. [[State 패턴]] 참고

---

## 한마디 요약

> **"절차는 부모가 한 번만 쓰게 하고, 자식은 변형 지점에만 이름을 남긴다."**

관련 문서:
- [[State 패턴]]
- [[Facade 패턴]]
- [[SOLID원칙]]
