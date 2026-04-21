---
생성날짜:
- 2026-02-28 17:28
마지막수정날짜:
- 2026-02-28-토요일 17:28
tags:
- vscode
- ide
- 가상환경
- 파이썬인터프리터
- 트러블슈팅
별칭:
- VS Code 인터프리터 인식 실패
- .vscode settings.json 수동 주입
type:
- 자료수집
Area/Reasource:
- 프로그래밍/환경설정
Project:
---
# IDE가 가상환경의 파이썬을 인식하지 못할 때

## 증상 요약

VS Code 계열 에디터(VS Code, Cursor, Antigravity 등)에서 프로젝트의 `.venv` 폴더가 있음에도 불구하고 인터프리터 선택 UI에서 잡히지 않거나, "uv 설치 팝업"이 반복적으로 뜨는 상황임. 이런 경우 UI를 통한 인터프리터 선택이 오작동한 것이므로, 워크스페이스 설정 파일(`.vscode/settings.json`)에 **인터프리터 경로를 직접 기록**해 IDE의 탐색 로직을 우회함.

## 한마디 요약
**`.vscode/settings.json`에 `python.defaultInterpreterPath`로 `.venv` 속 `python.exe` 경로를 박아 넣으면, IDE가 UI 대신 설정 파일을 최종 근거로 쓰게 됨.**

## 해결 절차

### 1단계: `.vscode` 폴더 생성

**존재 이유**: VS Code 계열 IDE(예: Cursor, Antigravity)가 현재 프로젝트에 대한 **워크스페이스(workspace, 하나의 프로젝트 폴더 단위)** 고유 설정 파일을 보관할 전용 공간을 만들기 위함임. 이 숨김 폴더 안에 설정 파일을 넣어야 IDE가 "사용자 지정 설정"으로 인식함.

**변수 풀이**:
- `.vscode`: 에디터가 약속된 경로로 인식하는 **고정 폴더명**. 이름을 바꾸면 IDE가 못 찾음.

**나중에 쓰이는 곳**: 설정 파일(`settings.json`)이 저장되는 물리적 위치임. IDE가 기동되거나 프로젝트를 다시 열 때 이 폴더를 가장 먼저 탐색함.

```cmd
mkdir .vscode
```

**직접 확인**: 명령 실행 후 프로젝트 루트에서 `dir /a` 를 치면 `.vscode` 폴더가 보임 (Windows 탐색기에서는 "숨김 파일 표시"를 켜야 보임).

### 2단계: `settings.json` 파일 생성 및 인터프리터 경로 주입

**존재 이유**: IDE의 인터프리터 선택 UI가 오작동해 uv 설치 팝업을 띄우는 현상을 우회하기 위해, cmd의 `echo` 명령으로 `.vscode\settings.json` 파일을 직접 만들고 그 안에 올바른 파이썬 경로를 박아 넣음. 이렇게 하면 IDE는 UI 탐색 결과를 무시하고 설정 파일 값을 사용함.

**변수 풀이**:

| 키 / 값 | 역할 |
|---------|------|
| `python.defaultInterpreterPath` | 프로젝트 기본 인터프리터 경로를 지정하는 설정 키(String 타입) |
| `${workspaceFolder}` | VS Code 내장 변수. 현재 열린 프로젝트 폴더의 절대 경로로 런타임에 치환됨 |
| `\\.venv\\Scripts\\python.exe` | 가상환경 내부의 실제 파이썬 실행 파일 위치 (Windows 기준) |

일상 비유: **리모컨의 채널 기본값을 강제로 5번에 고정**하는 것과 같음. UI 버튼으로는 다른 채널이 자꾸 눌려도, 리모컨 내부 설정 파일이 "기본 5번"이라고 박혀 있어 재부팅할 때마다 5번으로 맞춰짐.

**나중에 쓰이는 곳**:
1. IDE 재시작 시, 코드 분석 엔진(Pylance 등)이 이 경로를 읽고 작업 환경을 구성함
2. 내장 터미널을 새로 열 때, 이 경로의 가상환경을 자동 활성화(Activate)함
3. `Ctrl+Click`(정의로 이동) 시, 외부 패키지(예: deepeval)의 소스 코드를 이 경로 기준으로 역추적함

```cmd
echo {"python.defaultInterpreterPath": "${workspaceFolder}\\.venv\\Scripts\\python.exe"} > .vscode\settings.json
```

**직접 확인**: `type .vscode\settings.json` 명령으로 내용이 잘 박혔는지 확인 가능함.

## VS Code 인터프리터 선택 우선순위 (공식 문서 기준)

IDE가 어떤 파이썬을 쓸지 결정하는 순서임. 아래일수록 우선순위가 낮음.

| 순위 | 출처 | 적용 조건 |
|------|------|----------|
| 1 | `python-envs.pythonProjects[]` | 신규 Python Environments 확장이 구성된 경우 |
| 2 | `python-envs.defaultEnvManager` | 기본 환경 관리자가 명시 설정된 경우 |
| 3 | **`python.defaultInterpreterPath`** | **레거시 설정, fallback 역할** |
| 4 | 자동 검색 | 워크스페이스 내 `.venv` 혹은 글로벌 인터프리터 |

> [!note] 본 절차에서 3순위 키를 사용하는 이유
> 1~2순위는 확장 업데이트 등에 따라 동작이 달라질 수 있음. **3순위 `python.defaultInterpreterPath`는 오래 유지된 공식 fallback**이므로, UI가 막혔을 때 가장 안정적인 주입 지점임.

## 왜 이 방식인가 (트레이드오프)

- **대안 1: UI에서 Python: Select Interpreter 재클릭**. 근본 원인이 UI 오작동이므로 재시도해도 같은 팝업이 반복됨.
- **대안 2: 가상환경 재생성**. 유효한 해결책이지만 기존 `.venv`가 정상인 상황에서 불필요한 재설치 비용 발생.
- **본 방식(settings.json 직접 주입)**: 몇 줄로 끝나고, 프로젝트에 설정이 남으므로 팀원이나 다른 기기에서도 같은 효과를 재현 가능함. 단, `.vscode/settings.json` 이 git에 포함되는지 여부를 확인해야 함(보통 팀 공통 설정은 포함, 개인 설정은 제외).

## 포함 관계

- **워크스페이스(프로젝트 폴더 단위 설정 컨테이너)**
  - **`.vscode/` 폴더** (IDE가 탐색하는 고정 경로)
    - **`settings.json`** (JSON 키-값으로 IDE 동작 제어)
      - `python.defaultInterpreterPath` (이 노트의 주제 키)
      - 기타 키: `editor.formatOnSave`, `python.analysis.typeCheckingMode` 등

## 관련 노트

- [[가상환경 세팅 방법]] (`.venv` 생성 방법)
- [[가상환경 재설치 uv]] (`.venv` 손상 시 완전 재설치 절차)
- [[UV_SYNC_GUIDE]] (가상환경에 패키지를 넣는 법)
