---
생성날짜:
- 2025-05-21 18:08
마지막수정날짜:
- 2025-05-21-수요일 18:08
tags:
- 강의노트
type:
- 강의노트
과목: 
- 
관련인물:
source:
source url:
AreaReasource: 
Project: 
---

# 강의내용
---
- ## **윈도우에서 소스트리 문제**

윈도우의 소스트리에서 프로젝트의 상태가 바로 업데이트되어 보이지 않을 시  
영상에서 제가 했던 것처럼 새로 탭을 열었다 닫을 필요 없이 😅  
**F5**키를 눌러주시면 새로고침이 됩니다.

---

  

### 💡 무료 파트에서는 알기 쉬운 비유로 가볍게 설명합니다.

- 보다 기술적으로 정확한 설명은 5강부터 보실 수 있습니다.

  

## 1. 프로젝트의 변경사항들을 타임캡슐(버전)에 담기

```
No commits yet # 아직 버전이 없음을 의미함

Untracked files: # git이 관리하지 않는 파일을 의미함
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        lions.yaml
        tigers.yaml

nothing added to commit but untracked files present (use "git add" to track)

immor@desktopimmortal MINGW64 ~/Downloads/10. Project/git_practice (main)
```


변경사항 확인
### 현재 작업 폴더 상태 확인 — `git status`

`git status`는 **책상(작업 디렉터리)**·**바구니(스테이징)**·**앨범(저장소)** 세 칸을 비교해 “지금 바구니에 담을 것이 남았는지”를 알려 줍니다. 깔끔하면

pgsql

코드 복사

`nothing to commit, working tree clean`

이라고 나오죠. 이는 “할 일 없음, 책상이 앨범과 똑같다”는 뜻입니다.
`   git status   `

- 추적하지 않는(**untracked**) 파일: Git의 관리에 들어간 적 없는 파일

  

### 파일 하나 담기 ***git add + 파일명*** 

`   git add tigers.yaml   `

- `git status`로 확인

```
Changes to be committed: # 캡슐안에 들어옴
  (use "git rm --cached <file>..." to unstage)
        new file:   tigers.yaml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        lions.yaml
```
  

모든 파일 담기

`   git add .   `

- `git status`로 확인

  

---

  

## 2. 타임캡슐 묻기

아래 명령어로 **commit**

`   git commit   `

- Vi 입력 모드로 진입 - [Vim 강좌](https://www.yalco.kr/10_vim/)

`:` 가 붙은 입력 명령어 입력모드는 `ESC`를 눌러야 입력 가능

|작업|Vi 명령어|상세|
|:--|:--|:--|
|입력 시작|**i**|명령어 입력 모드에서 텍스트 입력 모드로 전환|
|입력 종료|`ESC`|텍스트 입력 모드에서 명령어 입력 모드로 전환|
|저장 없이 종료|**:q**||
|저장 없이 강제 종료|**:q!**|입력한 것이 있을 때 사용|
|저장하고 종료|**:wq**|입력한 것이 있을 때 사용|
|위로 스크롤|**k**|`git log`등에서 내역이 길 때 사용|
|아래로 스크롤|**j**|`git log`등에서 내역이 길 때 사용|

- `FIRST COMMIT` 입력한 뒤 저장하고 종료

  

커밋 메시지까지 함께 작성하기 (이렇게 하면 `" "`  안의 이름으로 바로 저장됨)

`   git commit -m "FIRST COMMIT"   `

  

아래 명령어와 소스트리로 확인
### 커밋(버전) 히스토리 확인 — `git log`

`git log`만 입력하면 **가장 최근 커밋부터 거꾸로** 자세한 목록을 보여 줍니다. 각 줄에는 커밋 ID(SHA-1), 작성자, 날짜, 메시지가 담겨요.
`   git log   `
- `j`로 내려가고 `k`로 올라감
- 종료는 `:q`

  

---

  

## 3. 다음 변경사항들을 만들고 타임캡슐에 묻기

###  실습 전 주의사항!

실습시 **줄바꿈 등의 세부사항도 영상과 똑같이** 해주세요!  
_예) team 줄과 manager 줄이 두 줄 간격인 것 등_  
이후의 실습에서 영상과의 사소한 차이로 충돌이 발생할 수 있습니다.

  

### 변경사항

- `lions.yaml` 파일 삭제
- `tigers.yaml`의 manager를 `Donald`로 변경 (변경하면 파일명 옆에  M이 떠있음)
- `leopards.yaml` 파일 추가 (새롭게 추가되면 파일명 옆에 U가 떠있음)

```
team: Leopards  ​  
manager: Luke  ​ 
members: 
- Linda 
- William 
- David  
```
 `

  

- `git status`로 확인
    
    - 파일의 **추가**, **변경**, **삭제** 모두 내역으로 저장할 대상
- `git diff`로 확인

```
On branch main
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    lions.yaml
        modified:   tigers.yaml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        leopards.yaml
```




### git diff상태에서

|작업|Vi 명령어|상세|
|:--|:--|:--|
|위로 스크롤|**k**|`git log`등에서 내역이 길 때 사용|
|아래로 스크롤|**j**|`git log`등에서 내역이 길 때 사용|
|끄기|**:q**|`:`가 입력되어 있으므로 `q`만 눌러도 됨|

  
  

### 캡슐에 담기
#### 모두저장:
#### `   git add .   `

- `git status`로 확인

  
#### 새로운 버전 추가
#### `   git commit -m "Replace Lions with Leopards"   `

  만약 소스트리에서 추가 되지 않았다면 탭을 추가했다가 바로 삭제 해서 새로고침
  

### 💡 **TIP** `add`와 `commit` 한꺼번에

`   git commit -am "(메시지)"   `

- 🛑 새로 추가된(untracked) 파일이 없을 때 한정

  

---

  

## 4. 다음 강을 위한 준비

다음의 세 커밋들을 추가하세요.

### 🎯 첫 번째 추가 커밋

- Tigers의 `members`에 `George` 추가
- 커밋 메시지: `Add George to Tigers`

  

### 🎯 두 번째 추가 커밋

- `cheetas.yaml` 추가

`   team: Cheetas  ​  manager: Laura  ​  members:  - Ryan  - Anna  - Justin   `

- 커밋 메시지: `Add team Cheetas`

  

### 🎯 세 번째 추가 커밋

- `cheetas.yaml` 삭제
- Leopards의 `manager`를 `Nora`로 수정
- `panthers.yaml` 추가

`   team: Panthers  ​  manager: Sebastian  ​  members:  - Violet  - Stella  - Anthony   `

- 커밋 메시지: `Replace Cheetas with Panthers`

  
  

### 소스트리 확인 결과

![logs](https://www.yalco.kr/images/lectures/git-github/2-1/logs.png)


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