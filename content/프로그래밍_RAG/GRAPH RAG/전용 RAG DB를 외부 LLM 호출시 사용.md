---
생성날짜:
  - 2026-04-08 16:09
마지막수정날짜:
  - 2026-04-08-수요일 16:08
tags:
  - MCP
  - FunctionCalling
  - RAG
  - ToolUse
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## 방법 1: MCP 서버로 만들기 (Claude Code용)

내가 만든 검색 서버를 MCP(Model Context Protocol, 클로드가 외부 도구/데이터에 접근할 때 쓰는 표준 프로토콜) 서버로 래핑하면, Claude Code가 직접 검색 요청을 보낼 수 있다.

```python
# FastAPI로 MCP 서버 만들기
from fastapi import FastAPI

app = FastAPI()

@app.post("/search")
async def search(query: str):
    # 벡터 검색 실행: pgvector에서
    vector_results = await vector_db.search(query)
    # 그래프 탐색 실행: Neo4j에서
    graph_results = await neo4j.cypher(query)
    # 두 결과 합쳐서 반환
    return {"results": vector_results + graph_results}
```

Claude Code한테는 이렇게 지시하면 된다.

```
"http://my-server.com/search 에서 검색해서 답해라"
```

---

## 예외: Claude.ai MCP 연결

Claude.ai 한정으로, MCP 서버를 등록하면 웹에서도 쓸 수 있다.

```
Claude.ai 설정 → Integrations → MCP 서버 주소 등록
→ 이후 대화에서 Claude가 자동으로 서버 호출 가능
```

즉 Claude Code(로컬 CLI)뿐 아니라 Claude.ai(웹)에서도 같은 서버를 재활용할 수 있다.

---

## 방법 2: LLM API (GPT, Claude API 등)

### Tool Use / Function Calling 방식

LLM한테 툴을 등록해두면 알아서 검색 요청을 보낸다. OpenAI는 Function Calling, Anthropic은 Tool Use로 부른다. 이름만 다르고 동작은 같다.

```python
tools = [
    {
        "name": "search_knowledge_base",
        "description": "그래프DB와 벡터DB에서 검색한다",
        "parameters": {
            "query": {"type": "string"}
        },
        # 이 엔드포인트로 호출
        "url": "http://my-server.com/search"
    }
]
```

흐름: LLM이 질문을 받으면 → 툴 호출이 필요한지 판단 → 서버에 검색 요청 → 결과를 받아서 → 최종 답변 생성.

---

## 현실적인 구조

```
사용자 질문
    ↓
SaaS LLM (Claude/GPT)
    ↓ tool call
내 검색 서버 (FastAPI)
    ↓         ↓
Neo4j      pgvector
(그래프)    (벡터)
    ↓         ↓
    결과 합쳐서 LLM에 반환
    ↓
최종 답변
```

이게 사실상 **직접 만든 RAG 서버를 LLM의 외부 툴로 붙이는 패턴**이고, 제일 범용적인 방법이다.

| 환경 | 자연스러운 연결 방식 |
|---|---|
| Claude Code | MCP 서버 |
| Claude.ai 웹 | MCP Integration |
| GPT API / Claude API | Function Calling / Tool Use |

## 한마디 요약
RAG DB를 외부 LLM에 붙이는 방법은 두 갈래: Claude 계열은 MCP, 범용 API는 Function Calling. 둘 다 내부적으로는 "HTTP로 검색 엔드포인트 호출"이라는 같은 구조다.
