---
생성날짜:
- 2026-02-28 17:52
마지막수정날짜:
- 2026-02-28-토요일 17:52
tags:
- uv
- 가상환경
- 재설치
- 트러블슈팅
- hardlink
- UV_LINK_MODE
별칭:
- uv sync 하드링크 에러
- os error 396 해결
- 클라우드 동기화 폴더 uv 문제
type:
- 자료수집
Area/Reasource:
- 프로그래밍/환경설정
Project:
---
# 가상환경 재설치 (uv)

## 한마디 요약
**`.venv`를 통째로 날리고 `uv venv`로 다시 짓는 절차임. 클라우드 동기화 폴더(Google Drive/OneDrive)에서 `uv sync` 시 발생하는 `os error 396` 해결법까지 포함.**

## 기본 재설치 (파이썬 3.12 기준)

```cmd
rmdir /s /q .venv
uv venv --python 3.12 .venv
```

- `rmdir /s /q`: 하위 파일까지 조용히 모두 지움 (`/s` 재귀, `/q` 확인 질문 생략)
- `uv venv --python 3.12 .venv`: 파이썬 3.12 인터프리터로 `.venv` 재생성

**직접 확인**:
```cmd
.venv\Scripts\python.exe --version
```
→ `Python 3.12.x` 가 찍혀야 함.

## `uv sync` 시 "hardlink" 에러가 뜨는 이유

### 증상

```
warning: Failed to hardlink files; falling back to full copy.
error: Failed to install: jsonpointer-3.0.0-py2.py3-none-any.whl
  Caused by: failed to hardlink file from C:\Users\immor\AppData\Local\uv\cache\...
  to C:\1.Project\ALL_FOR_ONE\.venv\Lib\site-packages\...:
  클라우드 작업은 호환되지 않는 하드 링크가 있는 파일에서 수행할 수 없습니다. (os error 396)
```

### 원인 분석

**1. `uv`의 작동 원리**

uv가 다른 도구들보다 압도적으로 빠른 이유는, 패키지 파일을 매번 새로 다운로드/복사하지 않고 **캐시 폴더(`%USERPROFILE%\AppData\Local\uv\cache`)** 의 원본과 가상환경(`.venv`)을 **하드 링크(hard link, 동일한 파일 데이터를 가리키는 두 개의 파일시스템 엔트리)** 로 연결하기 때문임.

일상 비유: **중앙 창고(캐시)에 있는 상자를 지점 창고(`.venv`)로 "진짜 복사"하지 않고, "같은 상자에 라벨만 하나 더 붙이는" 방식**. 디스크 용량도 아끼고 속도도 빠름.

**2. 공식 문서 기준 기본 link-mode**

| OS | 기본값 |
|------|------|
| macOS/Linux | `clone` (copy-on-write) |
| **Windows** | **`hardlink`** |

**3. 충돌 원인**

작업 폴더가 Google Drive 또는 OneDrive 같은 **클라우드 동기화 폴더** 안에 있으면, Windows는 해당 경로에서 하드 링크 생성을 엄격하게 차단함. 이유: 클라우드 동기화 서비스가 파일을 업로드 단위로 추적하는데, 하드 링크는 하나의 파일이 여러 경로에서 보이는 구조여서 동기화 상태를 계산할 수 없음.

결론: **uv는 하드 링크를 만들려 하고, Windows(클라우드 동기화)는 이를 막음 → `os error 396` 발생.**

## 해결: UV_LINK_MODE 환경 변수

### 1단계: uv 링크 모드를 `copy`로 강제 지정

**존재 이유**: 하드 링크가 막힌 환경에서 uv가 에러 없이 동작하도록, **파일을 실제로 복사(copy)** 하는 방식으로 전환함.

**변수 풀이**:

| 요소 | 역할 |
|------|------|
| `set` | Windows CMD에서 환경 변수를 현재 세션에 임시 생성/지정하는 내장 명령 |
| `UV_LINK_MODE` | uv가 패키지 설치 시 사용할 링크 방식을 결정하는 uv 전용 환경 변수 |
| `copy` | 하드 링크 대신 실제 파일 데이터를 그대로 복사하라는 값 |

**가능한 값** (공식 문서):

| 값 | 동작 | 특징 |
|------|------|------|
| `clone` | copy-on-write (APFS, Btrfs 등 지원 시) | macOS/Linux 기본 |
| `copy` | 파일 데이터를 그대로 복사 | 가장 안전, 클라우드 폴더 호환 |
| `hardlink` | 동일 inode 공유 | **Windows 기본**, 가장 빠름, 클라우드 폴더에서 차단됨 |
| `symlink` | 심볼릭 링크 | 캐시 삭제 시 환경 파괴 위험, 비권장 |

**나중에 쓰이는 곳**: 현재 CMD 창이 살아 있는 동안, uv가 `sync`/`install` 등으로 패키지를 설치할 때마다 이 값을 읽어 복사 방식을 적용함. **창을 닫으면 값이 사라짐**.

```cmd
set UV_LINK_MODE=copy
```

**영구 적용**이 필요하면 환경 변수를 시스템에 등록하거나, 프로젝트 `pyproject.toml`에 명시:

```toml
[tool.uv]
link-mode = "copy"
```

또는 `uv.toml`:

```toml
link-mode = "copy"
```

**직접 확인**:
```cmd
echo %UV_LINK_MODE%
```
→ `copy` 가 찍혀야 함.

### 2단계: 패키지 동기화 재시도

**존재 이유**: 하드 링크 에러로 중단됐던 설치를, 방금 설정한 복사 방식으로 다시 돌림.

**변수 풀이**:
- `uv`: Astral.sh의 파이썬 패키지 매니저 실행 명령
- `sync`: `pyproject.toml` + `uv.lock` 을 기준으로 `.venv`를 일치시키는 하위 명령

**나중에 쓰이는 곳**: 이 명령이 완료되면 `.venv\Lib\site-packages` 에 deepeval을 포함한 모든 의존 패키지가 **물리적으로 복사되어** 저장됨. 이후 코드 실행이나 IDE의 `Ctrl+Click`(정의로 이동)의 근거 파일로 쓰임.

```cmd
uv sync
```

## 전체 복구 스크립트 (원클릭)

클라우드 폴더에서 깨진 `.venv`를 전부 재구축:

```cmd
rmdir /s /q .venv
set UV_LINK_MODE=copy
uv venv --python 3.12 .venv
.venv\Scripts\activate.bat
uv sync
```

## 왜 `copy` 모드가 트레이드오프인가

| 관점 | `hardlink` (기본) | `copy` |
|------|------|------|
| 설치 속도 | 매우 빠름 | 상대적으로 느림 (실제 I/O 발생) |
| 디스크 사용량 | 적음 (캐시와 공유) | 많음 (파일 전체 복제) |
| 클라우드 동기화 폴더 호환 | 불가 | 가능 |
| 캐시 삭제 시 영향 | 없음 (하드 링크는 독립 inode) | 없음 |

결론: **클라우드 폴더에서 개발한다면 `copy`가 유일한 실사용 선택지**. 속도가 아까우면 프로젝트 폴더를 `C:\Dev` 같은 비동기화 경로로 옮기는 것도 방법.

## 자주 마주치는 변형 에러

| 에러 메시지 | 원인 | 해결 |
|------|------|------|
| `os error 396` | 클라우드 폴더 하드 링크 차단 | `UV_LINK_MODE=copy` |
| `The system cannot find the path specified` | `.venv` 경로 깨짐 | 재설치 |
| `ModuleNotFoundError` after sync | IDE가 다른 인터프리터 물고 있음 | [[IDE 가 가상환경의 파이썬 인식 못할때]] |

## 포함 관계

- **환경 변수 (Environment Variable)** (프로세스에 주입되는 키-값 설정)
  - **`UV_LINK_MODE`** (uv 전용 링크 방식 결정)
    - `copy` / `hardlink` / `clone` / `symlink`
  - **기타 uv 관련 환경 변수**: `UV_CACHE_DIR`, `UV_PYTHON`, `UV_INDEX_URL` 등

## 관련 노트

- [[가상환경 세팅 방법]] (처음부터 만드는 절차)
- [[UV_SYNC_GUIDE]] (`uv sync`의 자세한 동작)
- [[IDE 가 가상환경의 파이썬 인식 못할때]] (재설치 후 IDE가 인식 못할 때)
