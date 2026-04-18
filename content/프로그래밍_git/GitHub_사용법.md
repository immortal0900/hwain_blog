# GitHub 사용법 가이드

## 1. GitHub 기본 개념
- **Repository (저장소)**: 프로젝트의 모든 파일과 각 파일의 수정 이력을 포함
- **Commit**: 파일 변경사항을 저장하는 작업
- **Branch**: 메인 프로젝트에서 분리된 작업 공간
- **Pull Request**: 변경사항을 메인 브랜치에 병합하기 위한 요청
- **Issue**: 버그 리포트, 기능 요청 등을 추적하는 기능

## 2. 기본 작업 흐름
1. **Input (입력)**:
   - 코드 파일 생성/수정
   - README.md 파일로 프로젝트 설명 작성
   - .gitignore 파일로 추적하지 않을 파일 지정

2. **Output (출력)**:
   - 웹 페이지에서 코드와 문서 확인
   - 이슈와 풀 리퀘스트 토론
   - 프로젝트 위키 및 문서

3. **버전 업데이트 작업**:
   - 로컬에서 변경사항 커밋 후 원격 저장소에 푸시
   - 브랜치 생성 및 관리
   - 버전 태그 지정 (Release)

## 3. GitHub 시작하기
1. **계정 생성 및 로그인**
2. **새 저장소 만들기**:
   - GitHub 홈페이지에서 "New repository" 클릭
   - 저장소 이름, 설명, 공개/비공개 설정, README 초기화 등 설정
3. **파일 추가 및 편집**:
   - Add file 버튼 → Create new file 또는 Upload files 선택
   - 웹 에디터에서 직접 편집 가능

## 4. 버전 관리를 위한 Git 명령어 (로컬 작업 시)
```bash
# 저장소 복제
git clone https://github.com/사용자명/저장소명.git

# 변경사항 추가
git add .

# 변경사항 커밋
git commit -m "변경 내용에 대한 설명"

# 변경사항 원격 저장소에 푸시
git push origin main

# 브랜치 생성
git branch 브랜치명

# 브랜치 전환
git checkout 브랜치명
```

#GitHub #버전관리 #입문가이드
