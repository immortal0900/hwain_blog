---
생성날짜:
- 2026-01-20 01:58
마지막수정날짜:
- 2026-01-20-화요일 01:58
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
## 1. GitHub UI에서 빌드 결과 및 로그 확인하기 (완전 통합형)

GitHub Actions의 가장 큰 장점은 별도의 사이트에 들어갈 필요 없이 GitHub 대시보드 내에서 모든 실행 과정을 실시간으로 볼 수 있다는 점입니다.

### 확인 방법

1. **Actions 탭 클릭:** GitHub 저장소 상단의 메뉴 중 **[Actions]** 탭을 클릭합니다.
    
2. **워크플로우 선택:** 왼쪽 사이드바에서 확인하려는 워크플로우 이름을 선택합니다.
    
3. **실행 기록(Run) 선택:** 중앙 리스트에서 특정 시점에 실행된 로그(주로 커밋 메시지 제목)를 클릭합니다.
    
4. **로그 확인:** 왼쪽의 `Jobs` 리스트 중 하나를 클릭하면, 각 `Step`별로 어떤 명령어가 실행되었고 어디서 오류가 났는지 상세 로그를 볼 수 있습니다.
    

- **공식 문서:** [작업 실행 기록 보기 및 이해](https://docs.github.com/ko/actions/monitoring-and-troubleshooting-workflows/using-workflow-run-logs)
    

---

## 2. GitHub Marketplace에서 액션 찾아 쓰기

마켓플레이스는 전 세계 개발자들이 미리 만들어 놓은 자동화 '부품'들이 모여 있는 곳입니다. 예를 들어 "파일을 서버에 전송"하거나 "코드 스타일을 체크"하는 코드를 직접 짤 필요 없이 가져다 쓰기만 하면 됩니다.

### 사용 방법

1. **마켓플레이스 접속:** [GitHub Marketplace](https://github.com/marketplace?type=actions)에 접속합니다.
    
2. **필요한 기능 검색:** 검색창에 `Docker`, `Slack notification`, `AWS S3` 등 필요한 키워드를 입력합니다.
    
3. **코드 복사:** 마음에 드는 액션을 클릭하면 우측에 `Use latest version` 버튼이나 `Installation` 예시 코드가 있습니다.
    
4. **YAML 파일에 적용:** 내 프로젝트의 `.github/workflows/*.yml` 파일에 해당 내용을 붙여넣습니다.
    

### 대표적인 필수 액션 예시

- **actions/checkout:** 코드를 실행 환경으로 가져오는 필수 액션 ([링크](https://github.com/marketplace/actions/checkout))
    
- **actions/setup-python:** 파이썬 환경 설정 ([링크](https://github.com/marketplace/actions/setup-python))
    
- **aws-actions/configure-aws-credentials:** AWS 인증 설정 ([링크](https://www.google.com/search?q=https://github.com/marketplace/actions/configure-aws-credentials-for-github-actions))
    

---

### 실전 팁: 워크플로우 편집기 활용

GitHub 웹사이트 내에서 워크플로우 YAML 파일을 직접 편집하면, 오른쪽 사이드바에 **Marketplace**가 바로 뜹니다. 여기서 검색해서 버튼 하나로 코드를 바로 삽입할 수 있어 매우 편리합니다.

현재 진행 중인 프로젝트에서 **어떤 작업을 자동화(예: 배포, 테스트, 알림 등)**하고 싶으신가요? 구체적인 상황을 알려주시면 마켓플레이스에서 가장 적합한 액션을 조합한 코드를 짜드릴 수 있습니다.