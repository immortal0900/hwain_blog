---
생성날짜:
- 2026-02-03 15:57
마지막수정날짜:
- 2026-02-03-화요일 15:57
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
`git add .`로 스테이징한 파일을 내리는 방법은 크게 세 가지가 있습니다.

## 방법 1: `git restore --staged` (최신 권장 방법)

```
# 특정 파일만 unstage
git restore --staged 파일명.txt

# 특정 폴더 unstage
git restore --staged 폴더명/

# 모든 스테이징된 파일 unstage
git restore --staged .

```

이 명령은 스테이징된 파일을 내리지만 작업 디렉토리의 변경 사항은 유지합니다.[theserverside+1](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/git-unstage-file-all-index-commit-folder-add-delete)

## 방법 2: `git reset` (전통적 방법)

```
# 특정 파일만 unstage
git reset 파일명.txt

# 특정 폴더 unstage
git reset 폴더명/

# 모든 스테이징된 파일 unstage
git reset

```

## 방법 3: `git rm --cached` (파일 제거 시)

```
# 특정 파일을 Git 추적에서 완전히 제거 (로컬에는 유지)
git rm --cached 파일명.txt

# 폴더 전체를 추적에서 제거
git rm -r --cached 폴더명/

# 모든 파일을 추적에서 제거
git rm -r --cached .
```

`--cached` 옵션은 인덱스(스테이징 영역)에서만 제거하고 작업 디렉토리의 파일은 유지합니다.[github+1](https://docs.github.com/articles/ignoring-files)

## 차이점 정리

|명령어|용도|작업 디렉토리 파일|
|---|---|---|
|`git restore --staged`|스테이징 취소|유지됨 [[theserverside](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/git-unstage-file-all-index-commit-folder-add-delete)]​|
|`git reset`|스테이징 취소|유지됨 [[git-scm](https://git-scm.com/docs/gitignore)]​|
|`git rm --cached`|Git 추적 제거|유지됨 [[docs.github](https://docs.github.com/articles/ignoring-files)]​|

## 확인 방법
```
# 현재 스테이징 상태 확인 
git status 

# 스테이징된 변경사항 확인 
git diff --staged

```
bash


일반적인 경우에는 `git restore --staged .` 또는 `git reset`을 사용하시면 됩니다. `.gitignore`나 `.git/info/exclude`에 추가했는데도 이미 추적 중이라면 `git rm --cached`를 사용하세요.[git-scm+2](https://git-scm.com/docs/gitignore)