---
생성날짜:
- 2026-02-01 12:50
마지막수정날짜:
- 2026-02-01-일요일 12:49
tags:
- Python
- 함수
- 기본값
- pathlib
- Optional
- 설정관리
- AI_Agent
별칭:
- default None
- mutable default trap
- path resolution
type:
- 자료수집
Area/Reasource:
- 프로그래밍/Python/함수
Project:
---

## 한마디 요약

**`= None` 기본값 + `if x is None: x = ...` 패턴**은 Python 에서 "경로, 설정값, 리스트, 딕셔너리 같은 복잡한 기본값을 안전하게 주는 표준 관용구"다. 변경 가능한 기본값(mutable default)을 함수 시그니처에 직접 박으면 여러 호출 간에 상태가 섞이는 유명한 함정이 있기 때문. AI Agent 코드에서는 모델 경로, API 키, 세션 ID 같이 "없으면 알아서 채워줘야 하는" 값들에 자주 쓴다.

---

## 문제 상황 복습

```python
# 변경 전 (상대 경로 하드코딩)
def load_plm(plm_path: str = "./phishing_project/fine-tuned-phishing-model"):
    ...

def load_llm(adapter_path: str = "./phishing_project/gemma-phishing-adapter"):
    ...

# 변경 후 (None 기본값 + 내부 해석)
def load_plm(plm_path: str = None):
    if plm_path is None:
        plm_path = str(MODELS_DIR / "koelectra_finetuned")
    ...

def load_llm(base_model: str = "google/gemma-2b-it", adapter_path: str = None):
    if adapter_path is None:
        adapter_path = str(MODELS_DIR / "gemma_finetuned")
    ...
```

---

## `= None` 의 의미

**기본값(default value)**: 호출 시 인자를 생략하면 자동으로 `None` 이 들어간다는 뜻. 즉 "**필수가 아닌 선택 사항(Optional)**".

타입 힌트와 결합하면 본격 의미가 드러난다. [[함수의 타입 힌트]]

```python
from typing import Optional

def load_plm(plm_path: Optional[str] = None):
    #                   ↑                ↑
    #                   "str 이거나 None"  "생략 시 None"
    ...
```

---

## 왜 바로 `"./.../..."` 대신 `None` + 내부 해석으로 바꾸는가

### 이유 1: 실행 위치(cwd, current working directory) 에 영향 받는 상대 경로 제거

**상대 경로(relative path, 지금 실행 중인 폴더 기준)** 와 **절대 경로(absolute path, 루트부터의 고정 주소)** 의 차이.

| 구분 | 예시 | 기준 | 실행 위치 바꾸면 |
|---|---|---|---|
| 상대 경로 | `./phishing_project/...` | 현재 cwd | 경로 깨짐 |
| 절대 경로 | `C:/1.Project/.../models/...` | 파일시스템 루트 | 그대로 유효 |

```python
# 위험한 시나리오
# 터미널 A: cd C:/1.Project → python run.py       → OK
# 터미널 B: cd C:/         → python 1.Project/run.py → FileNotFoundError
```

해결: `MODELS_DIR` 같은 **절대 경로 상수**를 프로젝트 루트에서 한 번 계산해두고, 함수가 그걸 참조.

```python
# 프로젝트 상수 선언 (보통 config.py 나 paths.py)
from pathlib import Path

PROJECT_ROOT = Path(__file__).resolve().parent.parent   # 이 파일 기준 절대 경로
MODELS_DIR = PROJECT_ROOT / "phishing_project" / "models"
```

`Path(__file__).resolve()` 는 "지금 이 .py 파일의 실제 위치" 를 반환하므로, cwd 가 어디든 동일한 절대 경로가 나온다.

### 이유 2: OS 간 경로 구분자 호환

Windows 는 `\`, macOS/Linux 는 `/` 를 쓴다. 문자열 하드코딩은 한쪽에서 깨진다.

```python
# ❌ Windows 에서만 동작
path = "phishing_project\\models\\koelectra"

# ❌ macOS/Linux 에서만 동작  
path = "phishing_project/models/koelectra"

# ✅ 모든 OS 에서 동작 (pathlib 가 자동 변환)
from pathlib import Path
path = Path("phishing_project") / "models" / "koelectra"
```

**`pathlib.Path`** (path library, 객체 지향 경로 조작 모듈) 의 `/` 연산자는 내부적으로 OS 별 구분자를 결정한다. AI Agent 개발에서 모델 캐시 경로, 벡터 DB 파일 경로 등을 다룰 때 거의 필수.

### 이유 3: 설정 관리의 중앙화(single source of truth)

```python
# ❌ 흩뿌린 경로
def load_plm(plm_path: str = "./phishing_project/models/koelectra"): ...
def load_tokenizer(path: str = "./phishing_project/models/koelectra"): ...
def load_config(cfg: str = "./phishing_project/models/koelectra/config.json"): ...
# 모델 폴더 이름 바꾸면 3곳 다 고쳐야 함

# ✅ 한 곳에 모음
# config.py
MODELS_DIR = Path(__file__).resolve().parent / "models"
KOELECTRA_DIR = MODELS_DIR / "koelectra_finetuned"

# loaders.py  
def load_plm(plm_path: Optional[Path] = None):
    plm_path = plm_path or KOELECTRA_DIR
    ...
```

한 변수만 고치면 사용처 전부가 따라온다. **DRY(Don't Repeat Yourself) 원칙**의 실제 적용.

---

## 왜 하필 `None` 이어야 하는가: Mutable Default Argument 함정

단순히 "선택 사항" 표시가 목적이라면 `= "./some/path"` 를 써도 문자열은 불변이라 큰 문제가 없다. 그러나 기본값이 **리스트/딕셔너리/Path 객체** 같은 가변형이라면 Python 의 대표적 함정에 빠진다.

```python
# ❌ 함정 코드
def add_tag(name: str, tags: list = []):
    tags.append(name)
    return tags

print(add_tag("a"))   # ['a']
print(add_tag("b"))   # ['a', 'b']   ← 왜?!
print(add_tag("c"))   # ['a', 'b', 'c']
```

**원인**: 기본값 `[]` 는 **함수 정의 시점에 단 한 번** 생성되어 함수 객체에 붙는다. 매 호출마다 새 리스트가 만들어지지 않는다. 모든 호출이 동일한 리스트를 공유.

```python
print(add_tag.__defaults__)   # (['a', 'b', 'c'],)   ← 상태가 누적됨
```

### 정석 해결책: `None` + 내부 초기화

```python
# ✅ 안전한 패턴
def add_tag(name: str, tags: Optional[list] = None):
    if tags is None:
        tags = []        # 호출마다 새 리스트 생성
    tags.append(name)
    return tags

print(add_tag("a"))   # ['a']
print(add_tag("b"))   # ['b']    ← 독립
```

이 관용구가 있어서 "`None` 기본값 + `if x is None` 가드" 가 Python 커뮤니티의 표준 패턴이 됐다. 경로 같이 불변인 값에도 일관성 유지 차원에서 같은 패턴을 쓴다.

> `if tags is None` 은 `is` 로 비교한다. `==` 로는 빈 리스트·빈 딕셔너리와 섞일 수 있으니 `is None` 이 정석.

---

## AI Agent 개발에서의 흔한 `= None` 용례

### 용례 1: 모델/어댑터 경로

```python
def load_llm(
    base_model: str = "google/gemma-2b-it",
    adapter_path: Optional[Path] = None,
    device: Optional[str] = None,
):
    adapter_path = adapter_path or MODELS_DIR / "gemma_finetuned"
    device = device or ("cuda" if torch.cuda.is_available() else "cpu")
    ...
```

### 용례 2: LangChain CallbackHandler 선택 주입

```python
from typing import Optional
from langchain_core.callbacks import BaseCallbackHandler

def run_chain(
    query: str,
    callbacks: Optional[list[BaseCallbackHandler]] = None,
    metadata: Optional[dict] = None,
):
    callbacks = callbacks or []
    metadata = metadata or {}
    return chain.invoke(query, config={"callbacks": callbacks, "metadata": metadata})
```

### 용례 3: API 키 / 세션 ID (환경변수 폴백)

```python
import os

def get_client(api_key: Optional[str] = None):
    api_key = api_key or os.getenv("OPENAI_API_KEY")
    if api_key is None:
        raise ValueError("OPENAI_API_KEY 가 설정되지 않았습니다.")
    return OpenAI(api_key=api_key)
```

"호출자가 명시하면 그걸, 아니면 환경변수, 그것도 없으면 에러" 의 3단 폴백이 `= None` 기본값으로 깔끔해진다.

---

## `or` 단축 초기화 관용구

`if x is None: x = default` 를 한 줄로 줄이는 Python 숙어.

```python
def f(items: Optional[list] = None):
    items = items or []       # items 가 None 이면 [], 아니면 그대로
    ...
```

주의: `or` 는 "falsy 값" 전체를 걸러낸다. `0`, `""`, `[]`, `False` 가 들어오면 기본값으로 대체된다. 정말 `None` 일 때만 대체하고 싶다면 `items if items is not None else []` 를 써야 한다.

| 표현 | 대체 조건 | 사용 상황 |
|---|---|---|
| `x or default` | `x` 가 falsy 면 | 빈 컬렉션도 기본값으로 보고 싶을 때 |
| `x if x is not None else default` | `x` 가 `None` 일 때만 | `0`, `""`, `[]` 를 유효값으로 취급할 때 |

---

## 타입 힌트 정합성

```python
# Optional 을 명시하는 게 mypy/pyright 친화적
from typing import Optional

def load(path: Optional[str] = None):   # ✅
    ...

def load(path: str = None):             # △ 동작은 하지만 타입 검사기가 경고
    ...
```

Python 3.10+ 부터는 `str | None` 표기가 가능해서 `Optional[str]` 과 동일한 의미로 쓸 수 있다.

```python
def load(path: str | None = None):   # ✅ 3.10+
    ...
```

---

## 직접 확인: mutable default 함정 재현

```python
def broken(x=[]):
    x.append(1)
    return x

print(broken())   # [1]
print(broken())   # [1, 1]
print(broken())   # [1, 1, 1]

# 함수 객체에 매달려 있는 기본값 확인
print(broken.__defaults__)   # ([1, 1, 1],)
```

이 출력을 직접 보면 "아 정말로 공유되는구나" 가 손에 잡힌다.

---

## 요약 비교표

| 스타일 | 안전성 | 유지보수 | 권장도 |
|---|---|---|---|
| `def f(path="./hardcoded")` | cwd 의존 위험 | 산재 | △ |
| `def f(path=[])` (가변 기본값) | 상태 공유 버그 | 재현 어려움 | ❌ |
| `def f(path=None)` + 내부 해석 | cwd 무관, 버그 없음 | 중앙화 쉬움 | ✅ |
| `def f(path: Path = MODELS_DIR / "...")` | 모듈 로드 시점 고정 | OK | ○ (경로가 정말 변하지 않을 때만) |

---

## 관련 문서

- [[함수의 타입 힌트]]  (`Optional` 의 의미)
- [[반환 타입 힌트(화살표➡️)]]  (`-> Optional[X]` 와 대칭)
- [[함수(별args, 별별kwargs)]]  (`**kwargs` 로 넘어온 설정 해석)
- [[@데코레이터]]  (설정 주입 패턴)

---

## 한마디 재요약

`= None` 은 단순한 기본값이 아니라, **"가변 기본값 함정 회피 + 늦은(lazy) 초기화 + 환경 기반 폴백"** 을 한 문법으로 표현한 Python 숙어다. AI Agent 코드의 설정 관리에서는 거의 디폴트 선택지.
