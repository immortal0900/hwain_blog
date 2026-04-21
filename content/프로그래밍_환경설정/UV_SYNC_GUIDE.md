---
생성날짜:
- 2025-10-13 10:52
마지막수정날짜:
- 2025-10-13-월요일 10:51
tags:
- uv
- 가상환경
- 패키지관리
- pyproject-toml
- requirements-txt
별칭:
- uv 동기화 가이드
- uv sync 치트시트
type:
- 자료수집
Area/Reasource:
- 프로그래밍/환경설정
Project:
---
# UV 가상환경 동기화 가이드

## 한마디 요약
**`uv`(Astral.sh가 만든 Rust 기반 초고속 파이썬 패키지 매니저, pip+venv+pip-tools를 하나로 묶은 도구)를 써서 `.venv` 내용물을 선언 파일(`pyproject.toml` 또는 `requirements.txt`)과 정확히 일치시키는 절차 모음임.**

## `install` vs `sync` 핵심 차이

| 구분 | `uv pip install -r requirements.txt` | `uv pip sync requirements.txt` | `uv sync` |
|------|-----|-----|-----|
| 기준 파일 | requirements.txt | requirements.txt | pyproject.toml + uv.lock |
| 목록에 없는 패키지 | 그대로 남김 | **제거함** | **제거함** |
| 가상환경 생성 | 생성 안 함 | 생성 안 함 | **없으면 자동 생성** |
| 잠금 파일 | 사용 안 함 | 사용 안 함 | **uv.lock 자동 생성/갱신** |
| 권장 상황 | 기존 워크플로 호환 | pip-tools 대체 | uv 네이티브 프로젝트 |

일상 비유: **`install`은 냉장고에 장을 봐온 것만 "추가"로 집어넣기**, **`sync`는 장보기 목록과 냉장고 내용물을 "완전히 일치"시키기** (목록에 없는 건 버림).

## 1. 기본 명령어

### requirements.txt에서 설치만 (추가 방식)

```cmd
# 1) 가상환경 활성화 (Windows CMD)
.venv\Scripts\activate.bat

# 2) requirements.txt의 패키지를 추가
uv pip install -r requirements.txt
```

### requirements.txt와 정확히 동기화 (권장)

```cmd
.venv\Scripts\activate.bat
uv pip sync requirements.txt
```

> [!warning] sync는 파괴적임
> `uv pip sync`는 requirements.txt에 없는 패키지를 **가차없이 제거**함. 잘못된 requirements.txt로 돌리면 멀쩡한 패키지가 다 날아감. 실행 전에 `uv pip list`로 현재 상태를 한 번 찍어두는 습관 권장.

## 2. pyproject.toml과 동기화 (uv 네이티브 방식)

`pyproject.toml`(PEP 518에 정의된 파이썬 프로젝트 표준 설정 파일, 빌드 시스템/의존성/도구 설정을 담음)을 쓰는 프로젝트라면 이 방식이 공식 권장임.

```cmd
# 가상환경 자동 생성 + 의존성 설치 + uv.lock 갱신, 한 번에
uv sync
```

`uv sync`의 내부 동작 (공식 문서 기준):
1. `pyproject.toml` 에서 의존성 제약 조건을 읽음
2. `uv.lock`(정확한 해석 결과가 기록된 잠금 파일)이 최신인지 검사, 없거나 뒤처졌으면 새로 해결함
3. 잠금 파일을 근거로 `.venv`에 패키지를 설치하되, 목록에 없는 건 제거함 ("exact" 동기화)

제거를 원치 않으면 `--inexact` 플래그를 붙임:

```cmd
uv sync --inexact
```

## 3. requirements.txt ↔ pyproject.toml 변환

### pyproject.toml → requirements.txt (배포/호환성용)

```cmd
uv pip install -e .
uv pip freeze > requirements.txt
```

### requirements.txt → pyproject.toml (마이그레이션)

```cmd
uv pip install -r requirements.txt
uv pip freeze > requirements.txt
# 이후 pyproject.toml [project.dependencies]에 수동으로 옮겨 적음
```

## 4. 완전한 동기화 워크플로우

### 방법 1: requirements.txt 기반

```cmd
# 1) 가상환경 생성 (없는 경우)
uv venv

# 2) 활성화
.venv\Scripts\activate.bat

# 3) 동기화
uv pip sync requirements.txt

# 4) 확인
uv pip list
```

### 방법 2: pyproject.toml 기반 (권장)

```cmd
# 1) 생성 + 동기화 (한 번에)
uv sync

# 2) 활성화
.venv\Scripts\activate.bat

# 3) 확인
uv pip list
```

## 5. 업데이트 및 업그레이드

### 전체 업그레이드

```cmd
# requirements.txt 방식
uv pip install --upgrade -r requirements.txt

# pyproject.toml 방식 (모든 패키지 최신으로)
uv lock --upgrade
uv sync
```

### 특정 패키지만 업그레이드

```cmd
# pip 계열
uv pip install --upgrade 패키지명

# uv 네이티브
uv lock --upgrade-package 패키지명
uv sync
```

### 특정 버전으로 고정 업그레이드

```cmd
uv lock --upgrade-package 패키지명==버전
uv sync
```

## 6. 현재 설치된 패키지 확인

```cmd
.venv\Scripts\activate.bat

# 설치된 패키지 목록
uv pip list

# requirements.txt 형식으로 출력
uv pip freeze

# 파일로 저장
uv pip freeze > requirements.txt
```

## 7. 문제 해결

### 가상환경이 없는 경우

```cmd
uv venv
.venv\Scripts\activate.bat
```

### 패키지 충돌 해결 (완전 재설치)

```cmd
rmdir /s /q .venv
uv venv
.venv\Scripts\activate.bat
uv pip sync requirements.txt
```

자세한 재설치 시나리오: [[가상환경 재설치 uv]]

### uv가 설치되지 않은 경우

```cmd
# Windows 공식 설치 스크립트
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# 또는 pip로
pip install uv
```

### 클라우드 동기화 폴더에서 hardlink 에러가 날 때 (os error 396)

uv 기본 링크 모드(Windows에서는 `hardlink`)가 OneDrive/Google Drive 폴더에서 차단됨. 해결법:

```cmd
set UV_LINK_MODE=copy
uv sync
```

자세한 원리와 `UV_LINK_MODE` 설명: [[가상환경 재설치 uv]]

## 8. 자주 쓰는 명령 요약

```cmd
uv venv                                   # 가상환경 생성
.venv\Scripts\activate.bat                # 활성화 (Windows CMD)
uv pip sync requirements.txt              # requirements.txt와 정확히 동기화
uv sync                                   # pyproject.toml + uv.lock 기반 동기화
uv pip install 패키지명                   # 패키지 설치
uv pip uninstall 패키지명                 # 패키지 제거
uv pip list                               # 설치 목록 확인
uv pip freeze > requirements.txt          # 현재 상태 스냅샷
```

## 9. 현재 프로젝트 권장 방법

이 프로젝트는 `pyproject.toml` 기반이므로 기본 경로는 다음과 같음.

```cmd
# 1) 가상환경 생성 + 동기화
uv sync

# 2) 활성화
.venv\Scripts\activate.bat

# 3) 실행
python src/streamlit/web.py
```

requirements.txt 경로가 필요한 경우(배포 이미지, 레거시 CI 등):

```cmd
uv venv
.venv\Scripts\activate.bat
uv pip sync requirements.txt
```

## 10. 부록: docker 컨테이너 실행
```cmd
docker-compose up -d
```

## 포함 관계

- **`uv`** (Astral.sh의 파이썬 툴체인)
  - **프로젝트 네이티브 명령** (pyproject.toml + uv.lock 기반)
    - `uv venv` (환경 생성)
    - `uv sync` (환경을 lockfile 기준으로 맞춤)
    - `uv lock` (pyproject.toml 해석 결과를 uv.lock 으로 고정)
    - `uv add / uv remove` (의존성 수정)
  - **pip 호환 명령** (`uv pip ...`)
    - `uv pip install` (추가)
    - `uv pip sync` (일치 + 불필요 제거)
    - `uv pip freeze` (스냅샷 출력)

## 관련 노트

- [[가상환경 세팅 방법]]
- [[가상환경 재설치 uv]]
- [[IDE 가 가상환경의 파이썬 인식 못할때]]
