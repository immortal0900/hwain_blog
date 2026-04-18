---
생성날짜:
  - 2026-03-20 03:07
마지막수정날짜:
  - 2026-03-20-금요일 03:07
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---

아, `매매아파트`, `주소와단지명`, `세대수` 같은 **필드 이름(키)을 미리 정의해서 LLM이 그 키로만 출력하게 만드는 것** 자체를 말씀하시는 거군요.

그건 **Schema (스키마)** 입니다.

```python
# structured_schemas.py — 이 Pydantic 클래스 자체가 "스키마"
class NearbyMarketGeminiSchema(BaseModel):
    매매아파트: List[SaleApartment]   # ← 이 키 이름이 곧 스키마
    분양아파트: List[NewApartment]    # ← LLM은 이 키로만 응답 가능
```

좀 더 정확하게 구분하면:

|용어|의미|이 프로젝트 예시|
|---|---|---|
|**Schema (스키마)**|키 이름 + 타입 + 구조의 **정의(blueprint)**|`SaleApartment` 클래스의 필드 정의|
|**Structured Output**|스키마를 LLM API에 **적용하는 기법**|`.with_structured_output(스키마)`|

비유하면:

- **스키마** = 빈 엑셀 시트의 **헤더 행** (`주소와단지명 | 세대수 | 타입 | 평당매매가격 | ...`)
- **Structured Output** = 그 헤더 행을 LLM에게 건네면서 "이 칸에만 채워라"라고 **강제하는 행위**

"키 형태를 제공한다" → **스키마를 정의한다 (Schema Definition)**  
"그 형태로 출력을 고정한다" → **Structured Output으로 스키마를 강제한다 (Schema Enforcement)**

### 1. **Structured Output (구조화된 출력)**

`nearby_market_agent`와 `location_insight_agent`에서 `.with_structured_output(PydanticModel)`로 LLM 출력의 **JSON 형태(Shape)를 API 레벨에서 강제**하는 것을 **Structured Output**이라고 부릅니다.

이 프로젝트의 [structured_schemas.py](vscode-webview://1cu38o7ihevnaaolg8f21b16m0v1ti8eubk93s2sjjehnrhtrtri/src/agents/state/structured_schemas.py)가 바로 그 중앙 스키마 파일이고, [docs/error.md:1535-1738](vscode-webview://1cu38o7ihevnaaolg8f21b16m0v1ti8eubk93s2sjjehnrhtrtri/docs/error.md#L1535-L1738) 19번 섹션에 상세히 기록되어 있습니다.

**작동 원리 — 멘탈 모델:**

```
프롬프트만으로 요청       →  LLM이 자유롭게 응답  →  파싱 실패 가능
                              (마크다운, 설명 텍스트 섞임)

Structured Output 적용   →  API가 JSON Schema를 LLM에 주입  →  스키마에 맞는 JSON만 반환 가능
                              (Pydantic 모델이 곧 계약서)
```

- **Gemini**: `response_schema` 파라미터로 JSON Schema를 전달 → API 레벨에서 출력 형태 강제
- **OpenAI 계열 (GPT, Perplexity)**: `response_format` + JSON Schema → 동일 효과
- **LangChain 래퍼**: `.with_structured_output(PydanticModel)` → 위 두 방식을 통합 추상화

### 2. **Schema Enforcement / Output Schema Binding (스키마 강제 바인딩)**

"어딘가에서 원본 열 형태로 출력을 고정"하는 것은 `MovePopulationQuery` 스키마로 SQL의 **SELECT 컬럼을 고정**하는 패턴입니다. 이것도 Structured Output의 일종이지만, 더 구체적으로는 **Schema Enforcement**(스키마 강제) 또는 **Output Constraining**(출력 제약)이라고 부릅니다.

```python
# kostat_api.py — 컬럼 형태를 스키마로 고정
class MovePopulationQuery(BaseModel):
    sql: str = Field(
        description="SELECT year, origin, destination, total FROM age_population WHERE ..."
    )

llm = LLMProfile.dev_llm().with_structured_output(MovePopulationQuery)
response = llm.invoke(prompt)
query = response.sql  # 반드시 4개 컬럼만 SELECT
```

---

## 정리: 용어 계층 구조

|범위|용어|이 프로젝트에서의 예시|
|---|---|---|
|**포괄적 개념**|**Structured Output**|LLM 출력을 Pydantic 스키마로 형태 고정|
|**JSON 형태 고정**|**JSON Schema Enforcement**|`NearbyMarketGeminiSchema`, `LocationInsightGeminiSchema`|
|**컬럼/필드 고정**|**Output Constraining**|`MovePopulationQuery`로 SQL SELECT 컬럼 고정|
|**중앙 관리**|**Schema Registry** (스키마 레지스트리)|`structured_schemas.py`에 6개 스키마 집중 관리|

공식 문서에서 가장 널리 쓰이는 용어는 **"Structured Output"**이며, Google(Gemini), OpenAI, LangChain 모두 이 용어를 공식적으로 사용합니다. 이 프로젝트의 `error.md` 19번 섹션 제목도 정확히 이 용어를 사용하고 있습니다.