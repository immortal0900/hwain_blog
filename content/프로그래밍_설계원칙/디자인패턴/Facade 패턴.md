---
생성날짜:
- 2026-04-21 13:10
마지막수정날짜:
- 2026-04-21-화요일 13:10
tags:
- 설계원칙
- 디자인패턴
- 구조패턴
- AI에이전트
별칭:
- Facade
- 퍼사드 패턴
- 파사드
type:
- 자료수집
Area/Reasource:
Project:
---
# Facade 패턴

**"복잡한 서브시스템(Subsystem, 여러 클래스가 얽혀 있는 내부 구역) 앞에 단 하나의 간단한 창구를 세우는 구조 패턴."**

Facade 패턴(퍼사드 패턴, 정면 파사드라는 건축 용어에서 유래)은 GoF 구조(Structural) 패턴 중 하나다. 건물 내부의 복잡한 구조는 감추고, 바깥에서 보이는 건 잘 다듬어진 전면(facade)뿐이다. 코드에서는 내부의 여러 클래스/모듈을 조정하는 한 개의 고수준 클래스를 둬서, 클라이언트가 그것만 호출하면 되도록 한다.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 구조(Structural) 패턴 |
| 목적 | 복잡한 서브시스템을 단순한 인터페이스로 노출 |
| 핵심 질문 | "클라이언트가 내부 5~6개 클래스를 직접 조합해서 쓰고 있나?" |
| 대안 | Mediator(중재자), Gateway(게이트웨이) |
| 트레이드오프 | Facade가 "갓 오브젝트(God Object, 모든 걸 다 아는 비대한 클래스)"로 비대해질 위험 |

> **한마디 요약**: "내부는 복잡해도 바깥에는 버튼 하나만 보이게 하라."

---

## 일상 비유: 호텔 컨시어지

호텔 투숙객이 "내일 공연 보고 싶은데"라고 말하면 컨시어지(Concierge, 호텔 안내 담당자)가 티켓 예약처, 택시 회사, 저녁 식사 레스토랑, 일정표까지 알아서 조율한다. 투숙객은 개별 업체 전화번호를 몰라도 된다. 컨시어지가 곧 Facade다.

반대로 Facade 없이 살면, 투숙객이 티켓 예매 앱, 택시 앱, 식당 예약 앱을 각각 열어서 시간 맞춰 맞물려 놓아야 한다. 실수도 많고 귀찮다.

---

## 구조

| 역할 | 설명 |
| --- | --- |
| Facade | 클라이언트가 쓸 수 있는 고수준 메서드를 노출 (`ask`, `ingest` 등) |
| Subsystem Classes | 실제 일을 분담하는 내부 클래스들 (검색기, LLM, 포매터 등) |
| Client | Facade만 호출, 서브시스템의 존재를 몰라도 됨 |

단계 분해:
1. **1단계**: Client가 `facade.do_something(input)` 호출
2. **2단계**: Facade 내부에서 여러 서브시스템 객체의 메서드를 올바른 순서로 호출
3. **3단계**: 중간 결과를 조립하거나 변환해 최종 결과만 Client에 반환

---

## AI 에이전트 개발 예시 1: RAG Facade

RAG(Retrieval-Augmented Generation, 검색 기반 생성, 외부 지식을 찾아 LLM에게 붙여주는 기법) 파이프라인은 최소 5개 컴포넌트가 순서대로 돈다.

1. Embedder: 질문을 벡터로 바꿈
2. VectorStore: 유사 문서 검색
3. Reranker(선택): 검색 결과 재정렬
4. PromptBuilder: 컨텍스트 + 질문을 프롬프트로 조립
5. LLM: 최종 답변 생성

### 나쁜 예: 클라이언트가 내부를 다 안다

```python
def handle_question(q: str) -> str:
    vec = embedder.embed(q)
    hits = vector_store.search(vec, k=10)
    docs = [doc_store.get(h.id) for h in hits]
    reranked = reranker.rank(q, docs)[:4]
    prompt = prompt_builder.build(q, reranked)
    answer = llm.chat([{"role": "user", "content": prompt}])
    return answer.text
```

이 함수가 비즈니스 로직 여기저기에 있으면, k 값 조정, Reranker 껐다 켰다, LLM 교체 같은 변경이 생길 때 여러 파일을 동시에 건드리게 된다.

### 좋은 예: Facade 도입

```python
class RAGFacade:
    def __init__(
        self,
        embedder,
        vector_store,
        doc_store,
        reranker,
        prompt_builder,
        llm,
        *,
        top_k: int = 10,
        rerank_top_k: int = 4,
    ):
        self._embedder = embedder
        self._vector_store = vector_store
        self._doc_store = doc_store
        self._reranker = reranker
        self._prompt_builder = prompt_builder
        self._llm = llm
        self._top_k = top_k
        self._rerank_top_k = rerank_top_k

    def ask(self, question: str) -> str:
        vec = self._embedder.embed(question)
        hits = self._vector_store.search(vec, k=self._top_k)
        docs = [self._doc_store.get(h.id) for h in hits]
        reranked = self._reranker.rank(question, docs)[: self._rerank_top_k]
        prompt = self._prompt_builder.build(question, reranked)
        answer = self._llm.chat([{"role": "user", "content": prompt}])
        return answer.text

    def ingest(self, documents: list[str]) -> None:
        """문서 인덱싱도 같이 노출. 같은 서브시스템을 공유하니까 자연스럽다."""
        for d in documents:
            vec = self._embedder.embed(d)
            doc_id = self._doc_store.put(d)
            self._vector_store.upsert([(doc_id, vec, {"src": "ingest"})])


# 클라이언트 코드
rag = RAGFacade(
    embedder=OpenAIEmbedder(),
    vector_store=PineconeAdapter(index),
    doc_store=PostgresDocStore(conn),
    reranker=CohereReranker(),
    prompt_builder=DefaultPromptBuilder(),
    llm=OpenAIAdapter(openai_client, "gpt-4o"),
)

print(rag.ask("What is SOLID?"))
```

`rag.ask("...")` 한 줄로 끝난다. 서브시스템 구성 변경은 Facade 내부에서만 일어난다.

---

## AI 에이전트 개발 예시 2: Agent Runtime Facade

에이전트 실행 루프(관측 → 계획 → 도구 사용 → 답변)는 내부적으로 꽤 복잡하다. Facade로 감싸면 호출측이 간결해진다.

```python
class AgentFacade:
    def __init__(self, planner, tool_registry, memory, llm, tracer):
        self._planner = planner
        self._tools = tool_registry
        self._memory = memory
        self._llm = llm
        self._tracer = tracer

    def run(self, user_input: str, max_steps: int = 10) -> str:
        with self._tracer.span("agent.run") as span:
            span.set_attr("user_input", user_input[:200])
            self._memory.append_user(user_input)

            for step in range(max_steps):
                plan = self._planner.next_action(self._memory.snapshot())
                if plan.action == "final":
                    self._memory.append_assistant(plan.text)
                    return plan.text

                tool = self._tools.get(plan.tool_name)
                result = tool.run(plan.tool_args)
                self._memory.append_tool(plan.tool_name, result)

            return "Max steps exceeded"
```

계획, 도구 실행, 메모리 저장, 트레이싱이 한 메서드에 응집돼 있다. API 서버 핸들러는 `agent.run(user_input)`만 호출하면 된다.

---

## 실제 라이브러리에서의 Facade

| 라이브러리 | 적용 |
| --- | --- |
| LangChain | `RetrievalQA.from_chain_type(...).invoke({"query": q})`가 사실상 RAG Facade |
| Requests | `requests.get(url)` 한 줄로 소켓 연결, TLS 핸드셰이크, 응답 파싱을 다 처리 |
| FastAPI | `@app.get("/")` 데코레이터 뒤에 ASGI, 의존성 주입, 직렬화가 모두 숨겨짐 |
| Django ORM | `User.objects.filter(...).first()`가 SQL 생성, 커넥션, 역직렬화를 감춤 |

---

## 직접 확인해 보기

프로젝트 루트에서 다음을 확인한다.

```bash
# 서비스 계층이 라이브러리 내부 타입을 직접 들고 다니는가?
grep -rn "from openai" src/api/
grep -rn "pinecone.Index" src/api/
```

API 핸들러 쪽에서 외부 SDK 타입이 튀어나오면 Facade 계층이 부족하다는 신호다. `src/services/` 또는 `src/facades/` 레이어를 두고 거기서만 SDK를 쓰게 한다.

---

## 왜 이렇게 하는가

Facade의 설계 의도와 장점:

- **복잡도 격리**: 내부 5~6개 클래스를 바꿔도 Facade 바깥은 영향 없음
- **온보딩 속도**: 신규 개발자가 `AgentFacade.run(...)`만 봐도 시스템 구동이 가능
- **테스트 단순화**: 통합 테스트를 Facade 레벨에서 돌리면 End-to-End 시나리오가 간결해짐
- **Layered Architecture의 경계**: Facade가 도메인-인프라 경계에서 안정적인 API 역할

트레이드오프:
- Facade가 너무 많은 기능을 모으면 갓 오브젝트가 됨. `ask`, `ingest`, `rerank_only`, `embed_only`, `admin_reindex`... 메서드가 20개 넘으면 분할 신호
- 서브시스템의 세밀한 기능이 필요한 고급 사용자는 Facade를 우회하고 싶을 수 있음. "Facade는 필수가 아니라 선택"이라는 원칙을 지키고, 서브시스템도 외부에서 접근 가능하게 둔다

---

## 포함 관계와 다른 패턴

| 패턴 | 관점 | 차이 |
| --- | --- | --- |
| Facade | 내부 복잡도를 단순 API로 숨김 | 인터페이스 "수"가 줄어듦 (N → 1) |
| Adapter | 인터페이스 모양을 변환 | 인터페이스 "모양"이 바뀜 (A 규격 → B 규격) |
| Mediator | 내부 객체들끼리 서로 모르게 하고 중앙에서 조율 | 서브시스템 내부에서 작동, 외부용 창구는 아님 |
| Proxy | 원본과 같은 인터페이스로 접근 제어/캐싱 | Facade와 달리 인터페이스는 동일 |

- **Facade는 종종 Adapter 위에 쌓인다**: 내부 Adaptee들이 Adapter로 표준화된 다음, 그 Adapter들을 Facade가 조율하는 2층 구조가 많다. [[Adapter 패턴]] 참고

---

## 한마디 요약

> **"내부가 복잡하면 정문을 하나 내라. 손님은 정문만 봐도 충분하게 만들어라."**

관련 문서:
- [[Adapter 패턴]]
- [[Builder 패턴]]
- [[SOLID원칙]]
