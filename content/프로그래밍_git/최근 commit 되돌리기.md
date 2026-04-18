---
생성날짜:
- 2026-02-03 11:06
마지막수정날짜:
- 2026-02-03-화요일 11:04
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
```
# 최근 커밋 1개를 취소하고 스테이징 영역(Index/Cache)도 비웁니다.
# 작업 디렉토리의 파일 내용(코드)은 그대로 보존됩니다.
git reset HEAD~1
```

```
# 최근 커밋 2개를 취소하고 스테이징 영역(Index/Cache)도 비웁니다.
# 작업 디렉토리의 파일 내용(코드)은 그대로 보존됩니다.
git reset HEAD~2
```

### git revert
**`git revert`는 merge를 "해제"하는 게 아닙니다.** 해당 커밋의 변경사항을 **정확히 반대로 적용하는 새 커밋**을 만드는 겁니다.
- `git reset --hard fb3d5e0` → 히스토리를 **삭제**해서 되돌림 (force push 필요, 위험)
- `git revert HEAD` → 히스토리를 **보존**하면서 반대 커밋을 추가 (안전)
```
fb3d5e0  원래 main (think_tool 있음)
   ↓
57a61af  merge: think_tool 제거 (-423줄)
   ↓
새 커밋   revert: think_tool 복원 (+423줄)  ← git revert가 만드는 것

```

### git restore 파일 하나만 되돌리기

`git restore e2e_result.json`은 **현재 HEAD 커밋(`e353c5d`)에 저장된 버전으로 워킹 디렉토리의 파일을 되돌리는** 명령어입니다.

commit id를 명시하지 않으면 자동으로 HEAD(마지막 커밋)를 기준으로 합니다. 명시하고 싶다면:

```bash
git restore --source=e353c5d e2e_result.json   # HEAD와 동일
git restore --source=fb3d5e0 e2e_result.json   # 특정 커밋 지정
```

현재 HEAD(`e353c5d`)는 revert 커밋이라 코드 내용이 `fb3d5e0`(원본)과 동일하므로, commit id 없이 `git restore e2e_result.json`만 해도 원래 캐시로 복원됩니다.
 