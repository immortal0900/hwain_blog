---
생성날짜:
- 2025-05-21 11:52
마지막수정날짜:
- 2025-05-21 11:52
tags:
- 강의노트
- GitHub
type:
- 강의노트
과목: 
- 기타
관련인물:
source:
source url:
AreaReasource: git
Project: git
---

# 강의내용
---
- ##  머티리얼 테마

VS Code에서 강의 영상의 화면과 같이 파일 탐색기(파일 아이콘들) 부분이 보이게 하려면  
왼쪽의 `확장` 탭(블럭 모양 아이콘)에서 **Material Icon Theme** 테마를 검색하여 설치하세요.

![git-bash](https://www.yalco.kr/images/lectures/git-github/1-2/material.png)

  

---

  

## 1. Git 최초 설정

### Git 전역으로 사용자 이름과 이메일 주소를 설정

- GitHub 계정과는 별개

  

터미널 프로그램 (Git Bash, iTerm2)에서 아래 명령어 실행

`   git config --global user.name "(본인 이름)"   `

 "hwain"

`   git config --global user.email "(본인 이메일)"   `

"immortal0900@gmail.com"
  

아래의 명령어들로 확인

`   git config --global user.name   `

 

`   git config --global user.email   `

  

### 기본 브랜치명 변경

`   git config --global init.defaultBranch main   `

  

---

  

## 2. 프로젝트 생성 & Git 관리 시작

적당한 위치에 원하는 이름으로 폴더를 생성하고 **VS Code**로 열람

⭐️ 이후 강에서도 계속 사용

  

해당 폴더에서(VS Code 터미널 기본) 아래 명령어 입력

`   git init   `

  : 입력하면 현재 폴더에 숨은 `.git` 디렉터리가 생겨 “여기부터는 버전 관리하겠다!”고 선언하는 과정이 진행됩니다.

폴더에 숨김모드로 **.git** 폴더 생성 확인

- 🛑 이 폴더를 지우면 Git 관리내역이 삭제됩니다. (현 파일들은 유지)
- 맥에서 숨김 파일 보기: `command` + `shift` + `.`

  
  

아래의 파일들 생성

_tigers.yaml_

`   team: Tigers  ​  manager: John  ​  members:  - Linda  - William  - David   `

  

_lions.yaml_

`   team: Lions  ​  manager: Mary  ​  members:  - Thomas  - Karen  - Margaret   `

  

### ❗️ 모든 작업(파일 생성, 수정)마다 파일을 꼭 **저장**하세요!

  
  

터미널에 아래 명령어 입력

`   git status   `

  : 현재 폴더의 상황을 git의 관점으로 보여주는것

---

  

## 3. 소스트리로 해보기

### 현존하는 저장소 추가

- 소스트리에 폴더를 드래그하거나, `로컬 저장소 추가`

  

### Git이 관리하는 저장소 새로 만들기

- .git 폴더 삭제 후 진행
- 소스트리에 폴더를 드래그하거나, `로컬 저장소 생성`

### Git에서 파일 옆에 알파벳 의미

|색·글자|뜻|해야 할 일|
|---|---|---|
|**초록색 + U**|**Untracked** : Git이 아직 감시하지 않음|`git add <파일·폴더>`로 추적 시작|
|**파란색 + M**|**Modified** : 추적 중인데 수정됨|스테이징하려면 `git add`|
|**A**|**Added** : 새로 추가돼 스테이징 완료|곧바로 `git commit` 가능|

[[강의노트_Git에서 보이지 않게 설정]]
[[강의노트_Git에 버전 추가하기]]
[[강의노트_Git 복구하기]]
[[강의노트_Git소스코드]]
[[강의노트_Git_branch]]



## 목적
---
- 


## 궁금한 부분(애매한 부분)
---
- 


## 답변 혹은 자료 수집
---
- 


## 해야 할 일
---
- 


## 관련 노트
- 