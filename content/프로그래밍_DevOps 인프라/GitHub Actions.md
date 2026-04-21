---
생성날짜:
- 2026-01-20 01:58
마지막수정날짜:
- 2026-01-20-화요일 01:58
tags:
- DevOps
- CI_CD
- GitHub
- 자동화
- YAML
별칭:
- GHA
- GitHub Workflows
type:
- 자료수집
Area/Reasource:
Project:
---

# GitHub Actions

**GitHub Actions**(GitHub에 내장된 CI/CD 및 자동화 플랫폼, 이벤트 기반 워크플로 실행기)는 별도 서버 없이 GitHub 저장소 안에서 바로 CI/CD를 돌릴 수 있는 도구임.

관련 노트: [[CI CD 파이프라인]]

**포함 관계**:
```
GitHub Actions (플랫폼)
├─ Workflow (.github/workflows/*.yml)
│   └─ Job (실행 단위, VM 1대)
│       └─ Step (순차 명령)
│           └─ Action (재사용 가능한 부품) or run (셸 명령)
├─ Runner (실제 실행 머신)
│   ├─ GitHub-hosted (Ubuntu/Windows/macOS 무료 제공)
│   └─ Self-hosted (내 서버)
└─ Marketplace (남이 만든 Action 공유소)
```

---

## 1. 기본 개념

### 핵심 용어 한 번에 정리

| 용어 | 풀이 | 예시 |
|---|---|---|
| **Workflow** | 자동화 시나리오 전체. YAML 파일 1개 = 워크플로 1개 | `.github/workflows/ci.yml` |
| **Event** | 워크플로를 트리거하는 사건 | `push`, `pull_request`, `schedule` |
| **Job** | 하나의 Runner에서 실행되는 작업 묶음 | `test`, `build`, `deploy` |
| **Step** | Job 안의 순차 실행 단위 | 체크아웃 → 설치 → 테스트 |
| **Action** | 재사용 가능한 Step 부품 | `actions/checkout@v4` |
| **Runner** | Job을 실제로 돌리는 가상 머신 | `ubuntu-latest` |

### 파일 위치와 구조
워크플로 파일은 **반드시** 이 경로에 있어야 GitHub이 인식함.
```
저장소 루트/
└─ .github/
    └─ workflows/
        ├─ ci.yml        ← 각각 하나의 워크플로
        ├─ deploy.yml
        └─ nightly.yml
```

**일상 비유**: Action은 "부품", Step은 "조립 단계", Job은 "라인", Workflow는 "공장 한 동", Runner는 "공장 직원"임.

---

## 2. 최소 예제 (직접 돌려보기)

가장 단순한 Python CI 워크플로.

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      - name: Python 3.11 설치
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: 의존성 설치
        run: pip install -r requirements.txt

      - name: 테스트 실행
        run: pytest
```

이 파일을 커밋/푸시하면 즉시 Actions 탭에서 실행 로그가 뜸.

---

## 3. 이벤트 트리거 종류

언제 워크플로를 발동시킬지 정함. 주요 이벤트만 추림.

| 이벤트 | 의미 | 사용 예 |
|---|---|---|
| `push` | 브랜치 푸시 | CI, 배포 |
| `pull_request` | PR 생성/업데이트 | 리뷰 전 자동 테스트 |
| `schedule` | cron 일정 | 야간 배치, 주기 점검 |
| `workflow_dispatch` | 수동 실행 버튼 | 릴리스, 긴급 배포 |
| `release` | 릴리스 생성 | 패키지 배포 |
| `issue_comment` | 이슈/PR 댓글 | ChatOps |
| `repository_dispatch` | 외부 API 트리거 | 타 시스템 연동 |

### 조건 필터 (세밀한 제어)

```yaml
on:
  push:
    branches: [main, develop]
    paths:
      - "src/**"           # src 폴더 변경 시만
      - "!**/*.md"         # 마크다운 변경은 제외
  pull_request:
    types: [opened, synchronize]
  schedule:
    - cron: "0 3 * * *"    # 매일 03:00 UTC
```

### 직접 확인
`workflow_dispatch`를 추가하면 Actions 탭에 "Run workflow" 버튼이 생김.
```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        description: "배포할 버전"
        required: true
        default: "latest"
```

---

## 4. Job과 Step 설계

### Job 간 의존성 (`needs`)
기본적으로 Job은 **병렬 실행**됨. 순서 필요하면 `needs` 지정.

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]

  test:
    runs-on: ubuntu-latest
    needs: lint            # lint 성공 후에만 실행
    steps: [...]

  deploy:
    runs-on: ubuntu-latest
    needs: [lint, test]    # 둘 다 성공해야 실행
    if: github.ref == 'refs/heads/main'  # main 브랜치일 때만
    steps: [...]
```

### 매트릭스 빌드 (여러 환경 동시 테스트)

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python: ["3.10", "3.11", "3.12"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python }}
      - run: pytest
```

→ OS 3개 × Python 3개 = **9개 Job 병렬 실행**.

---

## 5. 자주 쓰는 Action

GitHub Marketplace(https://github.com/marketplace?type=actions)에 전 세계 개발자가 올린 "자동화 부품"이 모여있음.

| Action | 역할 | 비고 |
|---|---|---|
| `actions/checkout@v4` | 저장소 코드 가져오기 | **거의 필수** |
| `actions/setup-python@v5` | Python 설치 | 언어별 `setup-node`, `setup-java` 등 존재 |
| `actions/cache@v4` | 의존성 캐싱 | pip/npm 빌드 시간 단축 |
| `actions/upload-artifact@v4` | 빌드 결과물 저장 | Job 간 파일 공유 |
| `docker/build-push-action@v6` | Docker 이미지 빌드/푸시 | 이미지 배포 표준 |
| `aws-actions/configure-aws-credentials@v4` | AWS 인증 | OIDC 지원 |
| `peaceiris/actions-gh-pages@v4` | GitHub Pages 배포 | 정적 사이트 |
| `slackapi/slack-github-action@v1` | 슬랙 알림 | 성공/실패 통보 |

### Action 버전 고정 권장
- `@v4` : 메이저 버전 고정 (권장)
- `@main` : 항상 최신 (위험, 작성자가 바꾸면 깨짐)
- `@abc1234` : 커밋 SHA 고정 (보안 최강)

**직접 확인 팁**: 워크플로 YAML 웹 편집기에서 우측 사이드바의 **Marketplace**를 쓰면 검색 후 코드 자동 삽입 가능.

---

## 6. 환경 변수와 시크릿

### 일반 변수 (env)
```yaml
env:
  GLOBAL_VAR: "전역"      # 전체 워크플로에 적용

jobs:
  test:
    env:
      JOB_VAR: "잡 한정"   # 해당 Job만
    steps:
      - run: echo $GLOBAL_VAR $JOB_VAR
        env:
          STEP_VAR: "스텝 한정"
```

### 시크릿 (Secrets)
비밀번호, API 키 등은 **절대 YAML에 평문으로 넣으면 안 됨**.

1. GitHub 저장소 → Settings → Secrets and variables → Actions
2. `New repository secret`으로 이름/값 등록 (예: `AWS_ACCESS_KEY_ID`)
3. 워크플로에서 `${{ secrets.AWS_ACCESS_KEY_ID }}`로 참조

```yaml
- name: AWS 인증
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-northeast-2
```

**보안 원칙**:
- Pull Request는 외부인이 열 수 있음 → PR 이벤트에서 시크릿 노출 주의
- 로그에 시크릿이 찍혀도 GitHub이 자동으로 `***`로 마스킹해줌 (변수명이 표준적일 때 한정)
- 가능하면 OIDC(OpenID Connect, 키 없이 클라우드 임시 자격 발급 방식) 사용

---

## 7. Runner 선택

### GitHub-hosted Runner (기본)

| 종류 | 사양 | 무료 한도 (퍼블릭 저장소) |
|---|---|---|
| `ubuntu-latest` | 4 vCPU, 16GB RAM | **무제한** |
| `windows-latest` | 4 vCPU, 16GB RAM | 무제한 |
| `macos-latest` | 3 vCPU, 14GB RAM | 무제한 |

프라이빗 저장소는 월 2,000분 무료 (Free 플랜 기준).

### Self-hosted Runner
- **언제**: 대용량 빌드, 특수 하드웨어(GPU), 사내망 접근 필요
- **설정**: 저장소 Settings → Actions → Runners → New self-hosted runner 명령 복사해 서버에 설치
- **사용**: `runs-on: self-hosted` 또는 라벨로 지정

---

## 8. 빌드 결과 확인 (완전 통합형)

GitHub UI 안에서 모든 실행 과정을 실시간으로 볼 수 있음. 외부 사이트 이동 불필요.

### 단계별 확인 경로

1. **Actions 탭 클릭**: 저장소 상단 메뉴에서 `Actions` 진입
2. **워크플로 선택**: 왼쪽 사이드바에서 원하는 워크플로 이름 클릭
3. **실행 기록(Run) 선택**: 중앙 목록에서 특정 커밋의 실행 결과 클릭
4. **Job 클릭 → Step 로그 펼치기**: 실패한 Step이 빨간색으로 표시됨
5. **Re-run jobs**: 실패 Job만 재실행 가능 (비용 절감)

### 활용 팁

- **Status Badge 삽입**: `README.md`에 실시간 빌드 상태 뱃지 추가
  ```markdown
  ![CI](https://github.com/USER/REPO/actions/workflows/ci.yml/badge.svg)
  ```
- **Annotations**: 실패한 파일의 특정 줄이 PR의 "Files changed" 탭에 표시됨
- **Summary**: `$GITHUB_STEP_SUMMARY`에 Markdown 쓰면 실행 결과 페이지 상단에 요약 표시

---

## 9. 캐싱으로 속도 최적화

의존성 설치 시간이 워크플로 시간의 대부분 차지함. 캐시로 극적 단축 가능.

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

- `requirements.txt`가 바뀌지 않으면 **같은 캐시 재사용** → 1분 → 5초 단축
- `actions/setup-python@v5`는 `cache: 'pip'` 옵션으로 내장 캐싱 지원 (더 간단)

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: "3.11"
    cache: "pip"
```

---

## 10. 실전 예: 파이썬 앱을 Docker로 배포

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write       # GHCR 푸시 권한

    steps:
      - uses: actions/checkout@v4

      - name: Docker Buildx 설정
        uses: docker/setup-buildx-action@v3

      - name: GHCR 로그인
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: 이미지 빌드 & 푸시
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: 슬랙 알림
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {"text": "배포 결과: ${{ job.status }}"}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## 11. 자주 겪는 문제와 해결

| 문제 | 원인 | 해결 |
|---|---|---|
| 워크플로가 실행 안 됨 | 파일 경로 오타 | `.github/workflows/` 안에 있는지 확인 |
| `Permission denied` | GITHUB_TOKEN 권한 부족 | `permissions:` 블록 명시 |
| 시크릿이 `***`로 안 가려짐 | 커스텀 환경변수 평문 출력 | Secrets에 등록하고 `${{ secrets.X }}` 사용 |
| 매트릭스 한 개만 실패해도 전체 멈춤 | 기본 동작 | `strategy.fail-fast: false` 지정 |
| PR에서 시크릿 접근 안 됨 | 외부 포크 PR 보안 제한 | `pull_request_target` 신중 사용 |

---

## 12. 한마디 요약

**GitHub Actions = "YAML 파일 하나로 push/PR/일정에 맞춰 VM을 띄워 빌드/테스트/배포를 자동 수행"하는 GitHub 내장 CI/CD**. Marketplace의 기성 Action 조합 + 시크릿 + 매트릭스만 익히면 대부분의 자동화 가능.

---

## 참고 자료
- 공식 문서(한국어): https://docs.github.com/ko/actions
- 워크플로 문법 레퍼런스: https://docs.github.com/ko/actions/reference/workflow-syntax-for-github-actions
- 작업 실행 기록 보기: https://docs.github.com/ko/actions/monitoring-and-troubleshooting-workflows/using-workflow-run-logs
- GitHub Marketplace: https://github.com/marketplace?type=actions
- actions/checkout: https://github.com/actions/checkout
- actions/setup-python: https://github.com/actions/setup-python
- Docker build-push-action: https://github.com/docker/build-push-action
- OIDC로 AWS/GCP 인증: https://docs.github.com/ko/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect
