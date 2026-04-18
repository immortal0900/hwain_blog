---
생성날짜:
- 2025-05-21 16:29
마지막수정날짜:
- 2025-05-21-수요일 16:21
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
- ## Git의 관리에서 특정 파일/폴더를 배제해야 할 경우

#### a. 포함할 **필요가 없을** 때

- 자동으로 생성 또는 다운로드되는 파일들 (빌드 결과물, 라이브러리)

#### b. 포함하지 **말아야 할** 때

- 보안상 민감한 정보를 담은 파일

  

### **.gitignore** 파일을 사용해서 배제할 요소들을 지정할 수 있습니다.

  

---

  

## .gitignore 사용해보기

폴더에 아래 파일 생성

_secrets.yaml_

`   id: admin  pw: 1234abcd   `

  

아래 명령어로 상태 확인

`   git status   `

  
  

`.gitignore` 파일 생성
: `.gitignore`  에 저장한 파일명을 숨김처리 하는 작업

 `.gitignore` 에 `   secrets.yaml   `파일명을 추가
![[{EB9264E2-E6F3-413E-84E3-D02CA8833EF5}.png]]
  

다시 상태 확인

`   git status   `

  

---

  

## .gitignore 형식
[git 명령어 참조](<[https://git-scm.com/docs/gitignore](https://git-scm.com/docs/gitignore)>)
[https://git-scm.com/docs/gitignore](https://git-scm.com/docs/gitignore) 참조

```   
# 이렇게 #를 사용해서 주석  ​  

# 모든 file.c  
file.c  ​  

# 최상위 폴더의 file.c  
/file.c  ​  

# 모든 .c 확장자 파일  
*.c  ​  

# .c 확장자지만 무시하지 않을 파일 (느낌표로)
!not_ignore_this.c  ​  

# logs란 이름의 파일 또는 폴더와 그 내용들 (파일일땐 파일, 폴더일때는 폴더와 폴더의 파일 전부)
logs  ​  

# logs란 이름의 폴더와 그 내용들  
logs/  ​  

# logs 폴더 바로 안의 debug.log와 .c 파일들  
logs/debug.log  
logs/*.c
​  
# logs 폴더 바로 안, 또는 그 안의 다른 폴더(들) 안의 debug.log   즉, **로 건너 뛸 수 있다
logs/**/debug.log   
```







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