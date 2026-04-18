---
생성날짜:
- 2026-01-31 16:25
마지막수정날짜:
- 2026-01-31-토요일 16:23
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---


## [프로세스 개요]

GitHub 공식 문서(https://docs.github.com/en/get-started/quickstart/fork-a-repo)에 따르면, 다음과 같은 전략을 사용할 수 있습니다:

### **전략 1: Fork 후 Remote URL 변경 (추천)**
- 원본 저장소를 upstream으로 유지하면서 fork를 origin으로 설정
- 향후 원본 저장소의 업데이트를 받을 수 있음

### **전략 2: Remote URL 완전 교체**
- 원본 연결을 끊고 fork만 사용
- 단순하지만 원본 업데이트 추적 불가

---

## [단계별 가이드]

### **Step 1: GitHub에서 Fork 생성**
1. 브라우저에서 https://github.com/LonerStayle/ProjectML 접속
2. 우측 상단의 **Fork** 버튼 클릭
3. 본인 계정으로 fork 생성 확인

### **Step 2-A: Remote 재설정 (전략 1 - 추천)**

```bash
# 현재 origin을 upstream으로 이름 변경 (원본 추적용)
git remote rename origin upstream

# 본인의 fork된 저장소를 origin으로 추가
# YOUR_USERNAME을 본인 GitHub ID로 변경
git remote add origin https://github.com/YOUR_USERNAME/ProjectML.git

# 설정 확인
git remote -v
```

**예상 결과:**
```
origin    https://github.com/YOUR_USERNAME/ProjectML.git (fetch)
origin    https://github.com/YOUR_USERNAME/ProjectML.git (push)
upstream  https://github.com/LonerStayle/ProjectML.git (fetch)
upstream  https://github.com/LonerStayle/ProjectML.git (push)
```

### **Step 2-B: Remote URL 교체 (전략 2 - 단순)**

```bash
# origin URL을 fork된 저장소로 변경
git remote set-url origin https://github.com/YOUR_USERNAME/ProjectML.git

# 설정 확인
git remote -v
```

---

### **Step 3: Fork로 Push**

```bash
# 현재 수정사항 커밋 (필요시)
git add .
git commit -m "Update LLM.py"

# fork된 저장소로 push
git push -u origin main
```

---

## [중요 개념 설명]

### **1. Origin vs Upstream의 역할**
- **origin**: 본인이 직접 push할 수 있는 저장소 (fork)
- **upstream**: 원본 저장소 (읽기 전용, Pull Request 대상)

```
REM === 현재 작업 커밋 ===
git add .
git commit -m "refactor: 작업 내용"

REM === 원본(upstream)에 직접 push ===
git push upstream main

REM === 본인 fork도 동기화 (선택사항) ===
git push origin main

REM === 한 번에 양쪽에 push ===
git push origin main && git push upstream main


REM upstream의 최신 변경사항 가져오기 (다운로드만)
git fetch upstream

REM 현재 브랜치에 upstream/main 병합
git merge upstream/main

REM 또는 한 번에: fetch + merge
git pull upstream main


REM === 브랜치 작업 ===
git checkout -b feature/이름  # 새 브랜치 생성 및 이동
git checkout main             # main 브랜치로 복귀
git branch -d feature/이름    # 브랜치 삭제
```



**참고 문서:**
- GitHub Docs - Fork a repo: https://docs.github.com/en/get-started/quickstart/fork-a-repo
- GitHub Docs - Configuring a remote: https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories