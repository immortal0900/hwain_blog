---
생성날짜:
- 2026-01-30 04:05
마지막수정날짜:
- 2026-01-30-금요일 04:04
tags:
- python
- sys-path
- 모듈검색경로
- 가상환경
별칭:
- sys.path 사용법
- 파이썬 모듈 탐색 경로
type:
- 자료수집
Area/Reasource:
- 프로그래밍/환경설정
Project:
---
# sys.path 사용법

## 한마디 요약
**`sys.path`는 파이썬이 `import` 문을 만났을 때 모듈을 찾아 뒤지는 폴더 목록(리스트)임.** 이 리스트의 순서와 내용을 내가 직접 고칠 수 있음.

## 기본 개념

`sys.path`(전역 모듈 탐색 경로, 파이썬이 `import X` 문장을 만났을 때 `X` 라는 모듈 파일을 어디서 찾을지 결정하는 문자열 리스트)은 파이썬 인터프리터 기동 시점에 자동으로 채워짐.

일상 비유: **택배 기사가 배달 주소를 찾을 때 뒤지는 "주소 장부"** 와 같음. 장부의 맨 위 주소부터 차례로 뒤지다가 맞는 집이 나오면 거기서 멈춤.

## `sys.path` 구성 요소 (초기화 순서)

파이썬이 시작될 때 다음 순서대로 `sys.path`에 경로가 쌓임.

| 순번 | 들어오는 경로 | 언제 생기는지 |
|------|--------------|--------------|
| 1 | 스크립트가 있는 폴더 (`python script.py` 실행 시) | 인터프리터 기동 직후 최우선 삽입 |
| 2 | `PYTHONPATH`(환경변수, 사용자가 직접 지정하는 추가 탐색 경로) 값 | 시스템 환경변수 읽어와서 확장 |
| 3 | 설치 종속 기본 경로 (표준 라이브러리 폴더, `site-packages` 등) | 파이썬 빌드 시 결정된 값 |

> [!note] 가상환경 활성화 시
> 가상환경을 켜면 `sys.prefix`(현재 파이썬이 사용하는 인스톨 루트)가 `.venv` 쪽으로 바뀌면서, 설치 종속 기본 경로의 `site-packages`가 `.venv/Lib/site-packages`로 교체됨.

## 실제 값 직접 확인

현재 파이썬이 어떤 경로를 뒤지고 있는지 눈으로 보고 싶을 때:

```python
import sys
for i, p in enumerate(sys.path):
    print(f"{i}: {p}")
```

출력 예시 (가상환경 활성화 상태):
```
0: C:\1.Project\ALL_FOR_ONE
1: C:\Users\immor\AppData\Local\Programs\Python\Python312\python312.zip
2: C:\Users\immor\AppData\Local\Programs\Python\Python312\DLLs
3: C:\Users\immor\AppData\Local\Programs\Python\Python312\Lib
4: C:\Users\immor\AppData\Local\Programs\Python\Python312
5: C:\1.Project\ALL_FOR_ONE\.venv
6: C:\1.Project\ALL_FOR_ONE\.venv\Lib\site-packages
```

## 문법

```python
import sys
sys.path.insert(index, path)
sys.path.append(path)
```

- `index`: 삽입할 위치 (0이면 맨 앞 = 최우선 검색)
- `path`: 추가할 디렉토리 경로 (**반드시 문자열**, `Path` 객체는 무시됨)

## 주요 사용 패턴

### 패턴 1. 최우선 검색 (가장 흔한 경우)

```python
sys.path.insert(0, '/my/custom/path')
```

- 다른 모든 경로보다 먼저 검색함
- 같은 이름의 모듈이 여러 곳에 있을 때 "내가 지정한 쪽"을 먼저 쓰게 만드는 용도

### 패턴 2. 상대 경로로 프로젝트 루트 추가

```python
from pathlib import Path
import sys
sys.path.insert(0, str(Path(__file__).parent.parent))
```

- `__file__`: 현재 파일의 경로
- `.parent`: 현재 파일이 들어있는 폴더
- `.parent.parent`: 한 단계 더 위 (프로젝트 루트)
- `str(...)` 로 감싸는 이유: `sys.path`는 문자열만 받고, `Path` 객체는 조용히 무시되기 때문

### 패턴 3. 맨 뒤에 추가 (낮은 우선순위)

```python
sys.path.append('/fallback/path')
```

- 기존 경로를 모두 뒤진 다음, 마지막 수단으로만 참고할 폴더를 붙일 때 사용

## 검색 순서 이해하기

```python
sys.path = ['A', 'B', 'C']
sys.path.insert(0, 'X')
# 결과: ['X', 'A', 'B', 'C']
# import foo 실행 시 X → A → B → C 순으로 foo 를 찾음
```

파이썬은 `sys.path[0]` → `sys.path[1]` → ... 순서로 폴더를 뒤지다가, 원하는 모듈을 만나는 순간 **즉시 탐색을 중단**함. 따라서 같은 이름의 모듈이 `A`와 `C` 양쪽에 있다면 `A`에 있는 것이 import 됨.

## 주의사항

- **import 전에 실행해야 함**: `sys.path.insert()` 다음 줄에 `import`를 써야 효과가 있음
- **문자열로 변환 필수**: `Path` 객체나 숫자 등 다른 타입은 import 과정에서 무시됨 (공식 문서 명시)
- **절대 경로 권장**: 상대 경로는 실행 시점의 현재 작업 디렉토리(cwd)에 따라 달라질 수 있음
- **반복 중 수정 금지**: `for p in sys.path: sys.path.remove(p)` 같은 구문은 인덱스 꼬임 발생. `sys.path.copy()` 후 제거할 것

## 왜 이렇게 설계되었는가

`sys.path`를 리스트로 노출한 이유:
1. **유연성**: 프로젝트마다 모듈 구조가 다르므로, 런타임에 경로를 조정할 수 있어야 함
2. **순서 보장**: 리스트는 순서가 있으므로 "어느 경로를 먼저 뒤질 것인가"를 명확히 제어 가능
3. **대안 대비 장점**: 환경변수(`PYTHONPATH`)로만 관리하면 스크립트 외부 설정을 건드려야 하지만, `sys.path` 조작은 코드 내부에서 자기 완결적으로 해결됨

트레이드오프: 코드 어디서든 `sys.path`를 건드릴 수 있어 **경로 조작이 산재되면 디버깅이 힘들어짐**. 되도록 프로젝트 진입점(`main.py` 최상단, `conftest.py` 등)에서만 수정하는 편이 안전함.

## 포함 관계

- **`sys` 모듈** (파이썬 런타임 상태를 다루는 표준 라이브러리)
  - **`sys.path`** (모듈 탐색 경로 리스트)
    - `sys.path.insert(idx, path)` (특정 위치 삽입)
    - `sys.path.append(path)` (끝에 붙이기)
    - `sys.path.remove(path)` (제거)
    - `sys.path.copy()` (반복 중 수정을 위한 복사본)

## 관련 노트

- [[운영체제와 상관없는 경로설정]] (`Path(__file__).parent` 활용법)
- [[가상환경 세팅 방법]] (가상환경 활성화 시 `sys.path`가 어떻게 바뀌는지)
