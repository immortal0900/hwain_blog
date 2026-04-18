---
생성날짜:
- 2025-05-22 15:44
마지막수정날짜:
- 2025-05-22-목요일 15:44
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
![branches](https://www.yalco.kr/images/lectures/git-github/3-1/branches.png)

## **Branch**: 분기된 가지 (다른 차원)

- 프로젝트를 하나 이상의 모습으로 관리해야 할 때
    
    - 예) 실배포용, 테스트서버용, 새로운 시도용
- 여러 작업들이 각각 독립되어 진행될 때
    
    - 예) 신기능 1, 신기능 2, 코드개선, 긴급수정...
    - 각각의 차원에서 작업한 뒤 확정된 것을 메인 차원에 통합

  

### 이 모든 것을 **하나의 프로젝트 폴더**에서 진행할 수 있도록!

  

---

  

## 1. 브랜치 생성 / 이동 / 삭제하기

`add-coach`란 이름의 브랜치 생성

`   git branch add-coach   `

  

브랜치 목록 확인

`   git branch   `

  

`add-coach` 브랜치로 이동을 위해

`   git switch add-coach   ` 입력

- `checkout` 명령어가 Git 2.23 버전부터  `checkout`,`switch`, `restore`로 분리

  

### 💡 브랜치 생성과 동시에 이동하기
`   git switch -c (새 브랜치명)   `
`   git switch -c new-teams   `

- 기존의 `git checkout -b (새 브랜치명)`

  

### 🗑 브랜치 삭제하기

`   git branch -d (삭제할 브랜치명)   `

- `to-delete`란 브랜치 만들고 `to-erase` 로 이름 변경 후 삭제해보기

  

지워질 브랜치에만 있는 내용의 커밋이 있을 경우  
즉 다른 브랜치로 가져오지 않은 내용이 있는 브랜치를 지울 때는  
`-d` 대신 `-D`(대문자)로 강제 삭제해야 합니다.

`   git branch -D (강제삭제할 브랜치명)   `


| 옵션                    | 동작 (무엇을 하나요?) | 안전장치(검사 유무)                                 | 사용 상황(언제 쓰면 좋을까요?)                           |
| --------------------- | ------------- | ------------------------------------------- | -------------------------------------------- |
| `git branch -d <브랜치>` | 브랜치 **삭제**    | **메인(보통 `main`)에 병합됐는지** 확인 후, 안 됐으면 에러로 막음 | “작업 완료 → 메인에 병합 완료 → 가지 치워도 안전할 때”           |
| `git branch -D <브랜치>` | 브랜치 **강제 삭제** | **검사 없이 바로 삭제** (병합 여부 무시)                  | 임시·실험용 브랜치처럼 “기록 안 남아도 OK!” 할 때, 오타 브랜치 정리 등 |

### ✏️ 브랜치 이름 바꾸기

`   git branch -m (기존 브랜치명) (새 브랜치명)   `

  

---

  

## 2. 각각의 브랜치에서 서로 다른 작업해보기

### A. `main` 브랜치

1. Leopards의 `members`에 `Olivia` 추가
    
    - 커밋 메시지: `Add Olivia to Leopards`

  #### 모두저장:
#### `   git add .   `

- `git status`로 확인

  
#### 새로운 버전 추가
#### `   git commit -m "Replace Lions with Leopards"   `

  만약 소스트리에서 추가 되지 않았다면 탭을 추가했다가 바로 삭제 해서 새
  
  ### 💡 **TIP** `add`와 `commit` 한꺼번에

`   git commit -am "(메시지)"   `


2. Panthers의 `members`에 `Freddie` 추가
    
    - 커밋 메시지: `Add Freddie to Panthers`

  

#### ⭐️ `add-coach` 브랜치로 이동하여 해당 코드들 확인

  
  

### B. `add-coach` 브랜치

####  실습 전 주의사항!

실습시 **줄바꿈 등의 세부사항도 영상과 똑같이** 해주세요!  
_예) coach와 manager 사이에 한 줄 공백이 있는 것_  
이후의 실습에서 영상과의 사소한 차이로 충돌이 발생할 수 있습니다.  
  

1. Tigers의 매니저 정보 아래 `coach: Grace` 추가
    
    - 커밋 메시지: `Add Coach Grace to Tigers`

  

2. Leopards의 매니저 정보 아래 `coach: Oscar` 추가
    
    - 커밋 메시지: `Add Coach Oscar to Leopards`

  

3. Panthers의 매니저 정보 아래 `coach: Teddy` 추가
    
    - 커밋 메시지: `Add Coach Teddy to Panthers`

  
  

### C. `new-teams` 브랜치

1. `pumas.yaml` 추가
    
    - 커밋 메시지: `Add team Pumas`

`   team: Pumas  ​  
manager: Jude  ​  
members:  
- Ezra 
- Carter  
- Finn   `

  

2. `jaguars.yaml`
    
    - 커밋 메시지: `Add team Jaguars`

`   team: Jaguars  ​  manager: Stanley  ​  members:  - Caleb  - Harvey  - Myles   `

  

---

  

## 3. 결과 살펴보기

`git log`: 위치한 브랜치에서의 내역만 볼 수 있음

  

여러 브랜치의 내역 편리하게 보기

`   git log --all --decorate --oneline --graph   `

  
  

### 소스트리에서 확인

![3-branches](https://www.yalco.kr/images/lectures/git-github/3-1/3-branches.png)

### 서로 다른 브랜치를 합치는 두 방식

- **merge** : 두 브랜치를 한 커밋에 이어붙입니다.
    
    - 브랜치 사용내역을 남길 필요가 있을 때 적합한 방식입니다.
    - 다른 형태의 merge에 대해서도 이후 다루게 될 것입니다.
- **rebase** : 브랜치를 다른 브랜치에 이어붙입니다.
    
    - 한 줄로 깔끔히 정리된 내역을 유지하기 원할 때 적합합니다.
    - 이미 팀원과 공유된 커밋들에 대해서는 사용하지 않는 것이 좋습니다.

## 1. **merge**로 합치기

`add-coach` 브랜치를 `main` 브랜치로 **merge**

- `main` 브랜치로 이동
- 아래의 명령어로 병합
`   git merge add-coach(합칠대상)   `
`   git merge add-coach   `

- `:wq`로 자동입력된 커밋 메시지 저장하여 마무리
- 소스트리에서 확인

  

## 💡 `merge`는 `reset`으로 되돌리기 가능

- `merge`도 하나의 커밋
- `merge`하기 전 해당 브랜치의 마지막 시점으로

  

## 병합된 브랜치는 삭제

삭제 전 소스트리에서 `add-coach` 위치 확인

`   git branch -d add-coach   `

---

  

## 2. **rebase**로 합치기

> [!NOTE] Title
> merge에서는 중심 줄기로 이동해서 합칠 가지를
> git merge (합칠 가지명) 
> 이렇게 합치고
> 
> rebase는 합칠 가지로 이동해서
> git rebase (중심 줄기명)
> 이렇게 합친다
> 그 후에 중심줄기로 이동해서 다시 가지를 
> git merge (합칠 가지명)한다

`new-teams` 브랜치를 `main` 브랜치로 **rebase**

- `new-teams` 브랜치로 이동
    
    - 🛑 `merge`때와는 반대!
- 아래의 명령어로 병합

`   git rebase main   `

  

- 소스트리에서 상태 확인
    
    - `main` 브랜치는 뒤쳐져 있는 상황

  ![[{3AC273A4-7864-4D63-A93D-2D6E28677333}.png]]

- `main` 브랜치로 이동 후 아래 명령어로 `new-teams`의 시점으로 **fast-forward**

`   git merge new-teams   `

  ![[{3737BF8B-61C9-41D4-9B75-F44DA396E948}.png]]

- `new-teams` 브랜치 삭제

---

## 브랜치 간 충돌

- 파일의 같은 위치에 다른 내용이 입력된 상황

### 상황 만들기

1. `conflict-1`, `conflict-2` 브랜치 생성

  

2. `main` 브랜치
    
    - Tigers의 `manager`를 `Kenneth`로 변경
    - Leopards의 `coach`를 `Nicholas`로 변경
    - Panthers의 `coach`를 `Shirley`로 변경
    - 커밋 메시지: `Edit Tigers, Leopards, Panthers`

  

4. `conflict-1` 브랜치
    
    - Tigers의 `manager`를 `Deborah`로 변경
    - 커밋 메시지: `Edit Tigers`

  

5. `conflict-2` 브랜치 1차
    
    - Leopards의 `coach`를 `Melissa`로 변경
    - 커밋 메시지: `Edit Leopards`

  

5. `conflict-2` 브랜치 2차
    
    - Panthers의 `coach`를 `Raymond`로 변경
    - 커밋 메시지: `Edit Panthers`

  

---

  

## 1. `merge` 충돌 해결하기

`git merge conflict-1`로 병합을 시도하면 충돌 발생

- 오류 메시지와 `git status` 확인
- VS Code에서 해당 부분 확인

###  아래 기능 사라짐 _노랗게 표시한 부분_

![img](https://www.yalco.kr/images/lectures/git-github/3-4/vs-code.png)  
VS Code의 현재 버전에서는 충돌 난 부분 상단의 글자 버튼들 _(Accept Current...)_ 등이 나타나지 않습니다.  
충돌시 직접 해당 부분을 직접 타이핑해서 수정한 다음 `merge`를 계속 진행하시면 됩니다.
아니면 마우스 클릭으로 양자택일 가능

### 오류난 부분 찾는 방법 전체 검색에서 `<`  꺽쇠 여러개를 검색해서 찾는다.
![[Pasted image 20250525203020.png]]
  

당장 충돌 해결이 어려울 경우 아래 명령어로 `merge` 중단

`   git merge --abort   `

  

해결 가능 시 충돌 부분을 수정한 뒤 `git add .`, `git commit`으로 병합 완료 (그냥 git commit으로만 입력 해도 가능)

`: wq` 로 저장하고 나가기  ![[{921C81B3-AA87-47C2-9C91-E47819AC7FB1}.png]]

---

  

## 2. `rebase` 충돌 해결하기(여러개 충돌 될 경우)

`conflict-2`에서 `git rebase main`로 리베이스 시도하면 충돌 발생

- 오류 메시지와 `git status` 확인
- VS Code에서 해당 부분 확인

  

당장 충돌 해결이 어려울 경우 아래 명령어로 `rebase` 중단

`   git rebase --abort   `

  

### 해결 가능 시

- 충돌 부분을 수정한 뒤 `git add .`
- 아래 명령어로 계속

`   git rebase --continue   `  `:wq`

2번째 충돌 발생
![[Pasted image 20250525204741.png]]

- 충돌이 모두 해결될 때까지 반복
- rebase가 잘 된 모습
- ![[{B3EB881B-EA73-42A9-834B-CD7E86EC9E7F}.png]]
![[{84EF90D2-65FA-416B-A92D-B955AB9E184B}.png]]
  

`main`에서 `git merge conflict-2`로 마무리 (rebase로 하면 항상 이 과정을 해줘야 함)

  

`conflict-1`, `conflict-2` 삭제

git branch -d conflict-1
git branch -d conflict-2

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