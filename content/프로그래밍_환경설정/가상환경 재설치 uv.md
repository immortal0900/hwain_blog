---
생성날짜:
- 2026-02-28 17:52
마지막수정날짜:
- 2026-02-28-토요일 17:52
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
3.12 버전으로
```
rmdir /s /q .venv 
uv venv --python 3.12 .venv

```

### sync 시  아래와 같은 에러가뜨는 이유
에러 발생의 핵심 원인 분석
- **`uv`의 작동 원리:** `uv`가 다른 도구들보다 압도적으로 빠른 이유는, 패키지 파일을 매번 새로 다운로드하지 않고 캐시 폴더(`AppData`)에 있는 원본 파일과 가상환경(`.venv`)을 **'하드 링크(Hard Link)'** 라는 기술로 거울처럼 연결하기 때문입니다.
    
- **충돌 원인:** 현재 작업 중이신 `C:\1.Project\ALL_FOR_ONE` 폴더 혹은 C드라이브 전체가 **구글 드라이브나 원드라이브 같은 클라우드 동기화 서비스**와 연동되어 백업 중인 것으로 보입니다. 윈도우 운영체제는 클라우드로 동기화되는 폴더 안에서는 파일의 물리적 연결인 '하드 링크' 생성을 엄격하게 차단합니다.
    
- 결론적으로 `uv`는 하드 링크를 만들려 하고, 윈도우(클라우드 동기화)는 이를 막아버리면서 `os error 396`이 터진 것입니다.
```
warning: Failed to hardlink files; falling back to full copy. This may lead to degraded performance.
         If the cache and target directories are on different filesystems, hardlinking may not be supported.
         If this is intentional, set `export UV_LINK_MODE=copy` or use `--link-mode=copy` to suppress this warning.
error: Failed to install: jsonpointer-3.0.0-py2.py3-none-any.whl (jsonpointer==3.0.0)
  Caused by: failed to hardlink file from C:\Users\immor\AppData\Local\uv\cache\archive-v0\6EkhOVQnX0N4Es66NlKEm\jsonpointer-3.0.0.dist-info\AUTHORS to C:\1.Project\ALL_FOR_ONE\.venv\Lib\site-packages\jsonpointer-3.0.0.dist-info\AUTHORS: 클라우드 작업은 호환되지 않는 하드 링크가 있는 파일에서 수 행할 수 없습니다. (os error 396)
```


:: [명령어 1] uv 링크 모드 환경 변수 설정 (복사 모드 강제)
:: 존재 이유: 현재 작업 폴더가 구글 드라이브나 원드라이브 같은 클라우드 동기화 폴더로 설정되어 있어 윈도우가 하드 링크 생성을 차단(os error 396)하고 있기 때문에, uv가 에러를 일으키는 하드 링크를 시도하지 않고 파일 데이터를 통째로 옮기는 '복사' 방식을 사용하도록 강제 지시하기 위해 존재합니다. (에러의 핵심 원인을 우회하기 위한 가장 중요한 필수 설정입니다.)
:: 변수 (set): 윈도우 cmd 터미널에서 환경 변수(시스템 설정값)를 임시로 생성하거나 값을 할당할 때 사용하는 기본 명령어 변수입니다.
:: 변수 (UV_LINK_MODE): uv 도구가 패키지를 설치할 때 어떤 방식(하드 링크, 심볼릭 링크, 또는 복사)을 사용할지 결정하는 uv 내부 전용 환경 변수(설정 키)입니다.
:: 변수 (copy): 하드 링크 대신 실제 파일 데이터를 안전하게 그대로 복사(copy)해서 가상환경에 붙여넣으라는 값(Value) 변수입니다.
:: 나중에 쓰이는 곳: 이 명령어를 치고 나면, 현재 열려있는 cmd 터미널 창이 완전히 닫힐 때까지 uv가 패키지를 설치하거나 동기화(sync)할 때 무조건 '복사' 방식을 사용하게 만드는 전역 스위치 역할을 합니다.
```

set UV_LINK_MODE=copy

```

:: [명령어 2] 패키지 동기화 및 설치 재개 (uv sync)
:: 존재 이유: 하드 링크 에러 때문에 중간에 멈춰버렸던 프로젝트의 필수 파이썬 패키지들(jsonpointer 등 약 400개)을, 앞서 설정한 '복사(copy)' 방식을 통해 오류 없이 안전하게 .venv 가상환경 안으로 다운로드하고 설치하기 위해 존재합니다.
:: 변수 (uv): Astral.sh에서 만든 초고속 파이썬 패키지 관리자 실행 명령어 변수입니다. (앞서 설명했지만 다시 명시합니다.)
:: 변수 (sync): 프로젝트 폴더 내의 환경 설정 파일(pyproject.toml 또는 uv.lock)에 기록된 패키지 목록을 읽어와서, 현재 활성화된 가상환경(.venv)의 상태를 해당 목록과 똑같이 일치(동기화)시키라는 명령어 변수입니다.
:: 나중에 쓰이는 곳: 이 명령어가 완료되면 .venv 폴더 안에 deepeval을 포함한 모든 필수 패키지 파일들이 물리적으로 복사되어 저장되며, 이후 사용자님이 코드를 실행하거나 Ctrl+Click으로 소스 코드를 탐색할 때 뼈대가 되는 실제 라이브러리 파일들로 쓰이게 됩니다.
```
uv sync
```

