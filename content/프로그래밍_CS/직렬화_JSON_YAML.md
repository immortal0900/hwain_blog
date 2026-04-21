# 직렬화 (Serialization): JSON과 YAML

> **한마디 요약**: 메모리 안의 자료구조를 "문자열로 납작하게 펴서" 파일이나 네트워크로 주고받을 수 있게 바꾸는 작업. JSON은 기계 친화적·API용, YAML은 사람 친화적·설정 파일용. AI Agent 개발에서는 LLM Tool Call 응답부터 Agent 설정까지 전부 이 포맷 위에서 움직인다.

---

## 1. 직렬화가 뭔가

**Serialization(직렬화, 자료구조를 저장·전송 가능한 형태로 변환)**: Python dict, list 같은 메모리 안의 객체를 **바이트 열이나 문자열로 변환**하는 과정. 반대 방향(문자열 → 객체)은 **Deserialization(역직렬화)**.

**왜 필요한가**: 메모리 안의 `{"name": "철수"}` 는 실제로는 CPU 레지스터와 RAM의 특정 주소에 흩어진 구조. 이걸 그대로 네트워크로 보내거나 디스크에 저장할 수 없다. 네트워크·파일이 이해하는 **평평한 바이트 배열** 로 바꿔야 한다.

**일상 비유**: 이케아 가구. 조립된 책장(메모리 속 객체)을 그대로 택배에 못 실으니, 분해해서 납작한 박스(문자열)에 담아 보낸다. 받는 쪽이 설명서 보고 다시 조립(역직렬화)한다. 설명서가 곧 "포맷 규약"이고, JSON·YAML이 그 규약의 종류.

**포함 관계**:
- 직렬화 포맷에는 텍스트 기반(JSON, YAML, XML, TOML, CSV)과 바이너리 기반(Protocol Buffers, MessagePack, Pickle) 두 갈래가 있다.
- 이 문서는 텍스트 기반 두 대표인 JSON과 YAML을 다룬다.
- JSON은 YAML의 부분집합에 가깝다(YAML 1.2부터 공식적으로 JSON이 유효한 YAML). 즉 모든 JSON은 YAML이지만 역은 아니다.

---

## 2. JSON과 YAML 나란히 비교

| 관점 | JSON | YAML |
|------|------|------|
| **풀이** | JavaScript Object Notation | YAML Ain't Markup Language |
| **주 용도** | API 응답, 데이터 교환 | 설정 파일, CI 파이프라인 |
| **가독성** | 기계 친화적, 중괄호·대괄호 많음 | 사람 친화적, 들여쓰기 기반 |
| **문법 엄격도** | 쉼표·따옴표 엄격, 주석 없음 | 관대함, 주석(`#`) 가능 |
| **자료형** | string, number, boolean, null, array, object | 위 + 날짜, 다중 문서, 앵커/별칭 |
| **확장자** | .json | .yaml, .yml |
| **대표 예** | REST API, LLM Function Call 인자 | docker-compose.yml, GitHub Actions |
| **파싱 속도** | 빠름 | 느림(복잡한 규칙 때문) |
| **주석 지원** | 없음 | `# 주석` 가능 |
| **공백 민감** | 무시 | 들여쓰기가 의미를 결정 |

---

## 3. 같은 데이터를 양쪽 포맷으로

Python dict:
```python
config = {
    "model": "claude-opus-4-7",
    "temperature": 0.7,
    "max_tokens": 4096,
    "tools": ["web_search", "calculator"],
    "metadata": {
        "version": 2,
        "debug": True
    }
}
```

**JSON 표현**:
```json
{
  "model": "claude-opus-4-7",
  "temperature": 0.7,
  "max_tokens": 4096,
  "tools": ["web_search", "calculator"],
  "metadata": {
    "version": 2,
    "debug": true
  }
}
```

**YAML 표현**:
```yaml
# Claude Agent 설정
model: claude-opus-4-7
temperature: 0.7
max_tokens: 4096
tools:
  - web_search
  - calculator
metadata:
  version: 2
  debug: true
```

같은 데이터지만 YAML이 눈으로 읽기 편하다. 중괄호·따옴표가 사라지고 주석까지 달렸다. JSON은 기계가 파싱하기 쉬운 대신 사람 눈엔 답답하다.

---

## 4. JSON 문법 규칙 (단계 분해)

1단계: **객체는 `{}`**, 키·값 쌍을 `:`로 연결, 쌍 사이는 `,`로 구분.
2단계: **배열은 `[]`**, 원소 사이는 `,`.
3단계: **키는 반드시 큰따옴표 문자열**. 작은따옴표 금지. `'name'`은 에러.
4단계: **값 타입**: 문자열(`"..."`), 숫자(`3.14`), boolean(`true`/`false`, 소문자), `null`, 객체, 배열.
5단계: **주석 금지**. `//`도 `#`도 안 됨. (JSON5·JSONC 확장이 있지만 표준 아님.)
6단계: **마지막 원소 뒤 쉼표 금지**. `[1, 2, 3,]` 는 표준 JSON에서 에러.

이 엄격함이 파싱을 빠르고 안전하게 만든다. 대신 손으로 쓰기 피곤하다.

---

## 5. YAML 문법 규칙 (단계 분해)

1단계: **들여쓰기로 계층 표현**. 스페이스만 허용, 탭은 금지. 보통 2칸.
2단계: **키·값은 `key: value`**. 콜론 뒤 공백 필수.
3단계: **리스트는 `-` 접두**. 같은 들여쓰기 수준에 쭉 나열.
4단계: **문자열은 따옴표 선택**. `name: 철수` 처럼 그냥 써도 됨. 특수문자(`:`, `#`, `{}` 등) 포함되면 따옴표 필수.
5단계: **주석은 `#`** 부터 줄 끝까지.
6단계: **자료형 자동 추론**. `true`, `false`, `yes`, `no`, `on`, `off`는 모두 boolean. `123`은 숫자, `"123"`은 문자열. (YAML 1.1의 "Norway 문제(`NO`가 boolean false로 해석됨)"가 여기서 유래. YAML 1.2에선 `yes/no` 제거돼 완화.)

**주의할 함정**:
- 들여쓰기 실수하면 전혀 다른 트리가 됨 ([[트리]] 구조이므로)
- `version: 1.0` 은 숫자 1.0으로 해석 (따옴표 안 쓰면 버전 문자열 손실)
- 탭과 스페이스 섞으면 파서 에러

---

## 6. Python에서 직접 확인

### JSON 다루기 (표준 라이브러리)

```python
import json

data = {
    "model": "claude-opus-4-7",
    "temperature": 0.7,
    "tools": ["web_search", "calculator"]
}

# 직렬화 (Python → JSON 문자열)
json_str = json.dumps(data, indent=2, ensure_ascii=False)
print(json_str)

# 파일로 저장
with open("config.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2, ensure_ascii=False)

# 역직렬화 (JSON 문자열 → Python)
loaded = json.loads(json_str)
print(loaded["model"])  # claude-opus-4-7

# 파일에서 읽기
with open("config.json", "r", encoding="utf-8") as f:
    loaded = json.load(f)
```

`ensure_ascii=False`를 안 쓰면 한글이 `\uc804\uc2dc` 같은 이스케이프로 변환되어 가독성이 죽는다. 한글 데이터라면 필수 옵션.

### YAML 다루기 (PyYAML 필요)

```bash
pip install pyyaml
```

```python
import yaml

data = {
    "model": "claude-opus-4-7",
    "temperature": 0.7,
    "tools": ["web_search", "calculator"]
}

# 직렬화
yaml_str = yaml.dump(data, allow_unicode=True, sort_keys=False)
print(yaml_str)

# 역직렬화
loaded = yaml.safe_load(yaml_str)
print(loaded["model"])
```

**보안 경고**: `yaml.load()` (safe 없이) 는 **임의 코드 실행 취약점** 이 있다. 공격자가 YAML 안에 Python 객체 생성자를 심어두면 파싱만 해도 실행됨. 외부 입력에는 반드시 `yaml.safe_load()` 만 써라. JSON에는 이런 위험이 없다(코드 실행 경로가 없는 순수 데이터 포맷이라서).

---

## 7. AI Agent 개발에서 어디에 쓰이나

**LLM Tool Call(함수 호출) 인자**: Claude, OpenAI 모두 Tool의 입력·출력을 JSON으로 주고받는다. LLM이 `{"tool": "web_search", "query": "Claude 4.7"}` 같은 JSON을 뱉으면 Agent가 파싱해서 실행.

**API 통신**: HTTP 요청·응답 본문이 거의 다 JSON. `requests.post(..., json=payload)` 한 줄로 직렬화·전송·헤더 설정까지 끝남.

**Agent 설정 파일**: `config.yaml`로 모델, 온도, 프롬프트, 툴 목록을 관리. 사람이 자주 수정하는 곳이라 주석 달 수 있는 YAML이 유리.

**Prompt 템플릿 저장**: Jinja2 템플릿 메타데이터나 few-shot 예제를 YAML로 관리. `prompts/summarize.yaml` 안에 role, description, examples 를 계층적으로.

**LangChain·LangGraph 그래프 정의**: 노드·엣지 구조를 YAML로 선언해 코드 수정 없이 플로우 변경. 구조가 [[트리]]·그래프라 계층 표현이 자연스러운 YAML과 궁합이 좋음.

**Embedding 벡터 저장**: 벡터 자체는 용량 때문에 바이너리로 가되, 메타데이터({doc_id, source, timestamp})는 JSON.

**Chat History 로그**: 대화 기록을 JSON Lines(한 줄에 한 메시지) 형태로 저장. 뒤에 이어붙이기 쉽고, 스트리밍 처리에 적합.

**MCP(Model Context Protocol) 메시지**: Anthropic의 MCP 서버 간 통신이 JSON-RPC 기반. 전부 JSON 직렬화.

---

## 8. 왜 두 포맷이 공존하는가 (설계 의도)

**JSON의 철학**: "기계가 빠르게 파싱하고, 모호함 없이 주고받자." 그래서 엄격·단순·주석 없음.

**YAML의 철학**: "사람이 읽고 쓰기 편하게 하자. 파이썬처럼 들여쓰기로 계층 드러내자." 그래서 관대·표현력 높음·함정 많음.

**트레이드오프 정리**:
- **JSON 고를 때**: 네트워크 전송, 기계 간 통신, LLM 구조화 출력, 대량 데이터, 보안 민감
- **YAML 고를 때**: 사람이 자주 편집하는 설정, 복잡한 계층 구조, 주석 필요, CI/CD 파이프라인

**하이브리드 전략**: 많은 프로젝트가 "사람은 YAML로 작성 → 빌드 시 JSON으로 변환 → 런타임은 JSON 사용" 패턴을 쓴다. 양쪽 장점 모두 취함.

**다른 대안**:
- **TOML**: YAML의 함정을 줄이고 Cargo, pyproject.toml 에서 인기
- **XML**: 구버전. 태그가 장황. 여전히 SOAP·일부 레거시 시스템에 남음
- **Protocol Buffers**: 바이너리, 스키마 필수, 속도·크기 최강. gRPC 통신용
- **Pickle**: Python 전용, 매우 편리하지만 보안 위험, 다른 언어와 호환 안 됨

---

## 9. 성능 감각

JSON 파싱은 [[시간복잡도_기초]] 관점에서 입력 크기에 O(n). 100MB JSON 파싱은 Python 표준 `json`으로 수 초 걸림. 더 빠른 `orjson`, `ujson` 같은 C 구현 라이브러리가 있다. LLM 응답에서 큰 JSON을 자주 파싱한다면 고려할 가치 있음.

YAML 파싱은 규칙이 복잡해 같은 크기 JSON보다 3~10배 느리다. 설정 파일(작고, 앱 시작 시 한 번만 읽음)에 쓰이는 이유.

---

## 10. 관련 문서

- [[해시맵]]: JSON 객체와 1:1 대응되는 자료구조
- [[트리]]: 중첩된 JSON/YAML의 실체
- [[큐]]: 직렬화한 메시지를 주고받는 메시지 큐 패턴
- [[시간복잡도_기초]]: 파싱 비용이 왜 무시 못 할 크기까지 커지는가
