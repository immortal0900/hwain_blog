---
생성날짜:
  - 2026-04-08 16:09
마지막수정날짜:
  - 2026-04-08-수요일 16:08
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## 방법 1: MCP 서버로 만들기 (Claude Code용)

화인씨 서버를 MCP 서버로 래핑하면, Claude Code가 직접 검색 요청을 보낼 수 있다.

python

```python
# FastAPI로 MCP 서버 만들기
from fastapi import FastAPI

app = FastAPI()

@app.post("/search")
async def search(query: str):
    # 벡터 검색 실행해라: pgvector에서
    vector_results = await vector_db.search(query)
    # 그래프 탐색 실행해라: Neo4j에서
    graph_results = await neo4j.cypher(query)
    # 반환해라: 두 결과 합쳐서
    return {"results": vector_results + graph_results}
```

그러면 Claude Code한테:

```
"http://my-server.com/search 에서 검색해서 답해라"
```

이렇게 지시 가능하다.

---
## 예외: Claude.ai MCP 연결

Claude.ai 한정으로, MCP 서버를 등록하면 웹에서도 쓸 수 있다.

```
Claude.ai 설정 → Integrations → MCP 서버 주소 등록
→ 이후 대화에서 Claude가 자동으로 서버 호출 가능
```

---

## 방법 2: LLM API(GPT, Claude API 등)

### Tool Use / Function Calling 방식

LLM한테 툴을 등록해두면 알아서 검색 요청 보낸다.

python

```python
tools = [
    {
        "name": "search_knowledge_base",
        "description": "그래프DB와 벡터DB에서 검색한다",
        "parameters": {
            "query": {"type": "string"}
        },
        # 호출해라: 이 엔드포인트를
        "url": "http://my-server.com/search"
    }
]
```

LLM이 질문 받으면 → 툴 호출 필요하다 판단 → 서버에 검색 요청 → 결과 받아서 → 최종 답변 생성.

---

## 현실적인 구조

```
사용자 질문
    ↓
SaaS LLM (Claude/GPT)
    ↓ tool call
화인씨 검색 서버 (FastAPI)
    ↓         ↓
Neo4j      pgvector
(그래프)    (벡터)
    ↓         ↓
    결과 합쳐서 LLM에 반환
    ↓
최종 답변
```

이게 사실상 **직접 만든 RAG 서버를 LLM의 외부 툴로 붙이는 패턴**이고, 제일 범용적인 방법이다.

Claude Code는 MCP가 더 자연스럽고, GPT나 Claude API는 Function Calling이 더 자연스럽다.