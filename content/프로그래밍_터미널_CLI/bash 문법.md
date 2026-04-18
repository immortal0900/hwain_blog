---
생성날짜:
  - 2026-04-15 00:34
마지막수정날짜:
  - 2026-04-15-수요일 00:34
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## 셔뱅(Shebang)

bash

```bash
#!/bin/bash
```

`#!` — 셔뱅. OS에게 "이 파일을 어떤 인터프리터로 실행할지" 알려주는 지시자. `/bin/bash` — bash 셸의 절대 경로. 즉 "이 스크립트를 /bin/bash로 해석하라."

---

## set 내장 명령어

bash

```bash
set -e
```

`set` — 셸의 동작 옵션을 설정하는 내장 명령어(builtin command). `-e` — "exit on error" 옵션. 이후 실행되는 명령어 중 하나라도 종료 코드(exit code)가 0이 아니면(= 실패하면), 스크립트 전체를 즉시 중단하라.

---

## 변수 대입과 명령어 치환(Command Substitution)


```bash
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
```


**바깥 구조**: `변수명="값"` — 변수 대입.  `=` 양쪽에 공백이 있으면 안 됨.

**`$( ... )`** — 명령어 치환(command substitution). 괄호 안의 명령어를 실행하고, 그 표준출력(stdout) 결과를 문자열로 반환.


**안쪽부터 해석**:

- `"$0"` — 현재 실행 중인 스크립트 자신의 경로. `$`는 변수 참조 연산자. `0`은 특수 변수로, 스크립트 자체의 이름/경로를 담고 있음.
- `dirname "$0"` — `dirname` 명령어는 경로에서 파일명을 제거하고 디렉토리 부분만 반환. 예: `dirname "/a/b/script.sh"` → `/a/b`
- `cd "$(dirname "$0")"` — 그 디렉토리로 이동.
- `&&` — 논리 AND 연산자. 왼쪽 명령이 성공(exit code 0)해야만 오른쪽 명령을 실행.
- `pwd` — 현재 작업 디렉토리의 절대 경로를 출력.
- 즉, `cd`로 이동한 뒤 `pwd`로 절대 경로를 얻는 패턴. 상대 경로를 절대 경로로 변환하는 관용구(idiom).


```bash
PROJECT_ROOT="$(cd "$SCRIPT_DIR/.." && pwd)"
```

`..` — 상위 디렉토리를 의미하는 특수 경로. `$SCRIPT_DIR/..`는 스크립트 디렉토리의 부모 디렉토리.

---

## 조건문(if문)


```bash
if [ -f "$SCRIPT_DIR/.env" ]; then
  source "$SCRIPT_DIR/.env"
fi
```

**`if ... ; then ... fi`** — bash 조건문의 기본 구조.

- `if` — 조건 시작
- `then` — 조건이 참일 때 실행할 블록 시작
- `fi` — if문 종료 (`if`를 거꾸로 쓴 것)

**`[ ... ]`** — `test` 명령어의 약어. 조건식을 평가. 사실 `[`는 명령어이고 `]`는 닫는 인자.

**`-f`** — 파일 존재 여부 + 일반 파일(regular file)인지 검사하는 테스트 연산자(file test operator). 참이면 exit code 0 반환.

**`;`** — 명령어 구분자(command separator). 한 줄에 여러 명령을 쓸 때 사용. `] ; then`은 `]`와 `then`을 한 줄에 쓰기 위한 것.

**`source`** — 내장 명령어. 지정된 파일을 현재 셸 프로세스 안에서 실행(= 변수/함수가 현재 셸에 반영됨). `.`(dot)과 동일.

---

## 함수 정의



```bash
notify() {
  "$SCRIPT_DIR/notify.sh" "$@"
}
```

**`함수명() { ... }`** — bash 함수 정의 문법.

- `notify` — 함수 이름
- `()` — 함수 선언 표시 (인자를 여기에 쓰지 않음)
- `{ ... }` — 함수 본문(body)

**`"$@"`** — 특수 변수. 함수(또는 스크립트)에 전달된 모든 인자를 각각 개별 문자열로 전개(expand). `"$*"`와의 차이: `"$@"`는 `"인자1" "인자2" "인자3"`, `"$*"`는 `"인자1 인자2 인자3"` (하나의 문자열).

---

## mkdir

bash

```bash
mkdir -p artifacts/specs artifacts/decisions
```

`mkdir` — 디렉토리 생성 명령어. `-p` — "parents" 옵션. 중간 경로가 없으면 함께 생성하고, 이미 존재해도 에러를 내지 않음.

---

## 파이프라인과 명령어 체이닝

bash

```bash
SPRINT_NUM=$(ls artifacts/sprint-*-done.md 2>/dev/null | wc -l | tr -d ' ')
```

**`ls artifacts/sprint-*-done.md`** — `*`은 글로브 패턴(glob pattern, 와일드카드). 0개 이상의 임의 문자와 매칭. `sprint-`로 시작하고 `-done.md`로 끝나는 모든 파일을 나열.

**`2>/dev/null`** — 리다이렉션(redirection).

- `2` — 파일 디스크립터 2번 = 표준에러(stderr)
- `>` — 출력 리다이렉션 연산자
- `/dev/null` — "블랙홀" 특수 파일. 여기로 보낸 데이터는 버려짐.
- 즉, 에러 메시지(파일 없음 등)를 화면에 표시하지 않겠다는 뜻.

**`|`** — 파이프(pipe). 왼쪽 명령의 stdout을 오른쪽 명령의 stdin으로 연결.

**`wc -l`** — `wc`는 word count 명령어. `-l`은 줄 수(line count)만 출력.

**`tr -d ' '`** — `tr`은 문자 변환/삭제 명령어. `-d`는 delete 옵션. `' '`(공백 문자)를 삭제. `wc`가 출력에 공백 패딩을 넣는 경우가 있어서 이를 제거.

bash

```bash
SPRINT_NUM=$((SPRINT_NUM + 1))
```

**`$(( ... ))`** — 산술 확장(arithmetic expansion). 괄호 안의 수식을 정수 연산으로 계산. 변수 참조 시 `$` 생략 가능.

---

## echo와 문자열 보간(String Interpolation)

bash

```bash
echo "  프로젝트: $(basename "$PROJECT_ROOT")"
echo "  스프린트: #${SPRINT_NUM}"
```

**`echo`** — 문자열을 stdout으로 출력.

**큰따옴표(`"`) 안에서**:

- `$변수명` 또는 `${변수명}` — 변수 값으로 치환(expansion). `{}`는 변수명의 경계를 명확히 할 때 사용. 예: `${SPRINT_NUM}` vs `$SPRINT_NUMabc`(후자는 `SPRINT_NUMabc`라는 변수로 해석됨).
- `$(명령어)` — 명령어 치환도 큰따옴표 안에서 작동.

**`basename`** — 경로에서 디렉토리를 제거하고 파일/폴더명만 반환. `dirname`의 반대. 예: `basename "/a/b/project"` → `project`

---

## 복합 조건문

bash

```bash
if [ ! -f artifacts/spec.md ] && [ -n "$1" ]; then
```

**`!`** — 부정 연산자(NOT). 뒤의 테스트 결과를 반전.

- `[ ! -f artifacts/spec.md ]` — "spec.md가 일반 파일로 존재하지 않으면 참"

**`&&`** — 두 `[ ]` 테스트를 논리 AND로 연결. 둘 다 참이어야 전체가 참.

**`-n`** — 문자열 테스트 연산자. 문자열의 길이가 0이 아니면(= 비어있지 않으면) 참.

- `[ -n "$1" ]` — "첫 번째 인자가 비어있지 않으면 참"

**`$1`** — 위치 매개변수(positional parameter). 스크립트에 전달된 첫 번째 인자.

---

## elif / else

bash

```bash
elif [ -f artifacts/spec.md ]; then
  ...
else
  ...
fi
```

**`elif`** — "else if"의 축약. 이전 `if` 조건이 거짓이고, 이 조건이 참이면 실행. **`else`** — 위의 모든 조건이 거짓일 때 실행.

---

## cp / mv

bash

```bash
cp "$1" artifacts/spec.md
mv artifacts/sprint-contract.md "artifacts/sprint-${SPRINT_NUM}-done.md"
```

**`cp`** — copy. 첫 번째 인자를 두 번째 경로로 복사. **`mv`** — move. 파일 이동 또는 이름 변경.

---

## for 반복문

bash

```bash
for SPEC_FILE in artifacts/specs/*.md; do
  if [ -f "$SPEC_FILE" ]; then
    SPEC_NAME=$(basename "$SPEC_FILE")
    notify "spec_detail" "도메인 상세 스펙: ${SPEC_NAME}" "$SPEC_FILE"
  fi
done
```

**`for 변수 in 목록; do ... done`** — for 루프.

- `SPEC_FILE` — 반복 변수. 매 반복마다 목록의 다음 항목이 대입됨.
- `in artifacts/specs/*.md` — 글로브 패턴이 전개되어 매칭되는 파일 경로 목록이 됨.
- `do` — 루프 본문 시작.
- `done` — 루프 종료.

매칭 파일이 없으면 글로브가 리터럴 문자열 `artifacts/specs/*.md`로 남으므로, `[ -f "$SPEC_FILE" ]`로 실제 파일인지 한 번 더 확인.

---

## read

bash

```bash
read -p "수정 후 Enter, 무시하고 진행하려면 s 입력: " CHOICE
```

**`read`** — 표준입력(stdin)에서 한 줄을 읽어 변수에 저장하는 내장 명령어. **`-p "프롬프트"`** — prompt 옵션. 입력 전에 이 문자열을 표시. **`CHOICE`** — 입력값을 저장할 변수명.

---

## 문자열 비교

bash

```bash
if [ "$CHOICE" != "s" ]; then
```

**`!=`** — 문자열 부등 비교 연산자. 두 문자열이 같지 않으면 참.

- 반대: `=` (같으면 참). bash에서 `==`도 `[ ]` 안에서 동작하지만, POSIX 표준은 `=`.

---

## grep

bash

```bash
grep -q "종합 판정: PASS" artifacts/qa-report.md 2>/dev/null
```

**`grep`** — 텍스트에서 패턴을 검색하는 명령어. **`-q`** — quiet/silent 모드. 매칭 결과를 출력하지 않고, 찾았으면 exit code 0, 못 찾았으면 1만 반환. `if`문의 조건으로 쓸 때 유용.

---

## 히어 독(Here Document) 대신 사용된 멀티라인 문자열

bash

```bash
claude -p --max-turns 8 \
    "다음을 수행하라:
     1. artifacts/spec.md를 읽어라
     ..." \
    2>&1 | tee /tmp/harness-contract-log.txt
```

**`\`(백슬래시)** — 줄 이음(line continuation). 행 끝에 놓으면 다음 줄과 이어진 하나의 명령어로 취급.

**`2>&1`** — 리다이렉션.

- `2` — stderr(파일 디스크립터 2)
- `>&` — 파일 디스크립터를 다른 파일 디스크립터로 복제(duplicate)하는 연산자
- `1` — stdout(파일 디스크립터 1)
- 즉, "stderr를 stdout과 같은 곳으로 보내라" = 에러 출력과 일반 출력을 합쳐라.

**`tee`** — 입력을 받아서 stdout으로 출력하면서 동시에 파일에도 기록하는 명령어. T자 파이프 모양에서 이름 유래.

---

## ls 옵션

bash

```bash
LATEST_DECISION=$(ls -t artifacts/decisions/*.md 2>/dev/null | head -1)
```

**`ls -t`** — 수정 시간(modification time) 기준 내림차순 정렬. 가장 최근 파일이 맨 위. **`head -1`** — 입력의 첫 번째 줄만 출력. 즉 가장 최근 파일 하나만 가져옴.

---

## exit

bash

```bash
exit 1
```

**`exit`** — 스크립트를 종료하는 내장 명령어. **`1`** — 종료 코드(exit code). `0`은 성공, `1` 이상은 실패/에러를 의미. 호출한 쪽(부모 프로세스)에서 이 값으로 성공/실패를 판단할 수 있음.

---

## 따옴표 정리

|표기|이름|동작|
|---|---|---|
|`"..."`|큰따옴표(double quote)|변수 확장(`$`), 명령어 치환(`$()`) 수행. 공백 포함 문자열을 하나의 인자로 묶음|
|`'...'`|작은따옴표(single quote)|모든 것을 리터럴(문자 그대로) 취급. 변수 확장 없음|
|따옴표 없음||변수 확장 수행 + 공백 기준 단어 분리(word splitting) + 글로브 전개 발생|

---