---
생성날짜:
- 2025-05-22 04:25
마지막수정날짜:
- 2025-05-22-목요일 04:25
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
## 1. 변경사항 만들고 커밋하기

- `leopards.yaml` 삭제
- `.gitignore`에 `*.config` 추가
- `hello.txt` 추가 (내용 자유)
- 커밋 메시지: `Commit with SourceTree`

  

맥에서는 체크표시로 add

![logs](https://www.yalco.kr/images/lectures/git-github/2-4/mac-st-add.png)

### 스테이지에 commit 가능 ![[Pasted image 20250522042918.png]]
### 커밋 누르고 아래에 커밋이름을 입력
![[Pasted image 20250522043449.png]]

![[Pasted image 20250522043850.png]]



---

  

## 2. **revert**

- `Add George to Tigers`의 수정사항 되돌려보기
- 해당 커밋에 마우스 우클릭 - `커밋 되돌리기`

  

---

  

## 3. **reset**

- `Replace Cheetas with Panthers` 시점으로 되돌려보기
- 해당 커밋에 마우스 우클릭 - `... 이 커밋으로 초기화`
- 선택지에서 `Hard` 선택

  ![[Pasted image 20250522044058.png]]

---

  

##  Revert시 충돌 해결

지난 강에서 revert시 일어났던 충돌을 소스트리에서 해결하는 방법입니다.

### A. **윈도우**의 경우

1. `replace lions with leopards` 커밋을 되돌리기합니다.

![01](https://www.yalco.kr/images/lectures/git-github/2-4/01.png)

  

2. 충돌이 일어남을 알려주는 경고창이 뜹니다.

![02](https://www.yalco.kr/images/lectures/git-github/2-4/02.png)

  

3. 스테이지되지 않은 `leopards.yaml`을 우클릭하고 `충돌 해결`에서 저장소 것을 선택합니다.

![03](https://www.yalco.kr/images/lectures/git-github/2-4/03.png)

  

4. 메시지를 입력하고 커밋을 하면 완료됩니다.

![04](https://www.yalco.kr/images/lectures/git-github/2-4/04.png)

  

### B. **맥**의 경우

![04](https://www.yalco.kr/images/lectures/git-github/2-4/05.png)

맥의 소스트리는 해당 상황에 대한 기능이 미비하여  
위와 같은 오류 팝업이 나타납니다.

때문에 팝업에 나와 있듯, CLI에서 `git rm (파일명)` 명령어로 해당 파일을 지운 뒤  
`git commit`을 입력하여 수동으로 해결해주어야 합니다.


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