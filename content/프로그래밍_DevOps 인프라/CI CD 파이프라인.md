---
생성날짜:
- 2026-01-19 19:10
마지막수정날짜:
- 2026-01-19-월요일 19:09
tags:
- DevOps
- CI_CD
- 자동화
- 파이프라인
별칭:
- 지속적 통합
- 지속적 배포
- Continuous Integration
- Continuous Delivery
type:
- 자료수집
Area/Reasource:
Project:
---

# CI/CD 파이프라인

**CI/CD 파이프라인**(Continuous Integration/Continuous Delivery Pipeline, 코드 커밋부터 운영 배포까지 자동화된 단계 흐름)은 한마디로 **"코드를 짜서 실제 사용자에게 도달하기까지의 과정을 자동화한 컨베이어 벨트"**임.

관련 노트: [[GitHub Actions]]

**포함 관계**:
```
DevOps 문화
└─ CI/CD (자동화 실천)
    ├─ CI (Continuous Integration)    : 합치고 테스트
    ├─ CD (Continuous Delivery)        : 언제든 배포 가능 상태 유지 (수동 승인)
    └─ CD (Continuous Deployment)      : 자동 배포까지 포함
```

---

## 1. 왜 필요한가 (설계 의도)

개발자가 코드를 작성한 뒤 실제 서비스에 반영되기까지 수많은 반복 작업(빌드, 테스트, 배포 등)이 필요함. 이걸 사람이 수동으로 하면:

| 문제 | 결과 |
|---|---|
| 테스트 수동 실행 → 까먹거나 생략 | 버그가 운영까지 흘러감 |
| 배포 매뉴얼 30단계 | 실수 발생 확률 매우 높음 |
| "내 PC에선 되는데" 증후군 | 환경 차이로 운영 장애 |
| 릴리스 주기 월 1회 | 피드백 늦어짐 |

CI/CD는 이를 **"사람이 수동으로 하지 않고 기계가 자동으로 처리"**하도록 만든 시스템임.

**일상 비유**: 택배 물류센터의 컨베이어 벨트. 상품(코드)이 올라오면 → 검수(테스트) → 포장(빌드) → 배송(배포)까지 라인을 따라 자동으로 흘러감.

---

## 2. CI (Continuous Integration, 지속적 통합)

**"개발자들의 코드를 자주 합치고 잘 돌아가는지 자동 확인하는 단계"**

### 상황
여러 개발자가 각자 브랜치에서 기능을 개발함. 각자의 변경사항이 합쳐졌을 때 충돌이나 버그가 없는지 검증 필요.

### 자동으로 하는 일 (단계 분해)

1. **코드 push 감지**: 개발자가 Git 원격 저장소에 커밋 올림
2. **체크아웃(Checkout)**: CI 서버가 최신 코드를 가져옴
3. **의존성 설치**: `pip install -r requirements.txt`, `npm install` 등
4. **정적 분석(Lint)**: 코드 스타일/문법 오류 탐지 (`ruff`, `eslint` 등)
5. **빌드(Build)**: 실행 가능한 형태로 변환 (컴파일, 번들링 등)
6. **유닛 테스트**: `pytest`, `jest` 등으로 함수/모듈 단위 검증
7. **통합 테스트**: 여러 모듈이 함께 동작하는지 확인
8. **결과 리포트**: 성공/실패 표시, 실패 시 개발자에게 알림

### 목적
- **버그를 미리미리 잡자**
- **코드 통합을 작게 자주** (큰 덩어리로 합치면 충돌 지옥)

### 핵심 지표
- **빌드 성공률**: 90% 이상 목표
- **파이프라인 실행 시간**: 10분 이내 권장 (너무 느리면 개발자가 안 기다림)
- **테스트 커버리지**: 프로젝트에 따라 60~80%

---

## 3. CD (Continuous Delivery vs Continuous Deployment)

**CD는 두 가지 다른 개념을 한 약자로 부름**. 실무에서 혼동 많음.

| 구분 | Continuous **Delivery** | Continuous **Deployment** |
|---|---|---|
| **번역** | 지속적 제공 | 지속적 배포 |
| **테스트 통과 시** | 배포 가능 상태로 대기 | **자동으로 운영 배포** |
| **배포 트리거** | 사람 승인 버튼 | 자동 |
| **위험도** | 낮음 (통제 가능) | 높음 (견고한 테스트 필요) |
| **적합한 상황** | 금융/의료 등 보수적 영역 | SaaS, 웹 서비스 |

실무에서 "CD"라고만 쓰면 둘 중 어느 쪽인지 맥락으로 구분해야 함.

### CD가 하는 일 (공통)

1. **아티팩트 패키징**: 빌드 결과물을 Docker 이미지 등으로 묶음
2. **이미지 레지스트리 푸시**: Docker Hub, ECR, GHCR 등에 업로드
3. **스테이징 배포**: 운영과 같은 구성의 테스트 서버에 먼저 배포
4. **스모크 테스트(Smoke Test)**: "불 나는지만" 체크하는 간단 검증
5. **운영 배포**: 실제 사용자 서버에 반영
6. **헬스체크**: 서비스 정상 동작 확인
7. **(실패 시) 롤백**: 이전 버전으로 자동 복구

### 주요 배포 전략

| 전략 | 설명 | 트레이드오프 |
|---|---|---|
| **Rolling** | 인스턴스를 순차 교체 | 가장 간단, 교체 중 버전 혼재 |
| **Blue-Green** | 새 환경(Green) 준비 후 스위치 전환 | 즉시 롤백 가능, 자원 2배 |
| **Canary** | 트래픽 5% → 25% → 100% 점진 | 위험 최소화, 구성 복잡 |

---

## 4. 파이프라인 전체 흐름 예시

간단한 Python 웹 서비스 기준 예시.

```
[1] 개발자: git push origin feature/new-api
        ↓
[2] GitHub Actions 감지 → workflow 실행
        ↓
[3] CI 단계
    ├─ actions/checkout@v4 (코드 가져오기)
    ├─ setup-python@v5 (파이썬 3.11 설치)
    ├─ pip install -r requirements.txt
    ├─ ruff check . (Lint)
    ├─ pytest tests/ (테스트)
    └─ [실패] → 슬랙 알림 + 파이프라인 중단
        ↓
[4] CD 단계 (main 브랜치만)
    ├─ docker build -t my-app:$SHA .
    ├─ docker push ghcr.io/me/my-app:$SHA
    ├─ kubectl set image deployment/my-app app=my-app:$SHA
    └─ curl /health (헬스체크)
        ↓
[5] 사용자: 새 기능 사용 가능
```

---

## 5. 주요 CI/CD 도구 비교

| 도구 | 방식 | 장점 | 단점 |
|---|---|---|---|
| **GitHub Actions** | GitHub 내장, YAML | GitHub과 완전 통합, 공개 저장소 무료 | GitHub 의존 |
| **GitLab CI** | GitLab 내장, YAML | 올인원 (이슈~레지스트리 포함) | GitLab 의존 |
| **Jenkins** | 자체 호스팅, Groovy | 무한 커스터마이징, 플러그인 풍부 | 서버 운영 부담 |
| **CircleCI** | SaaS, YAML | 빠른 실행, 병렬 처리 우수 | 유료 한도 |
| **ArgoCD** | K8s 전용 GitOps | 쿠버네티스 선언적 배포 | CI는 따로 필요 |

실무 2026년 기준 가장 많이 쓰는 조합: **GitHub Actions + ArgoCD** 또는 **GitHub Actions 단독**.

---

## 6. 직접 확인

간단한 `.github/workflows/ci.yml`로 최소 CI 파이프라인 구성 가능. (세부 문법은 [[GitHub Actions]] 참고)

```yaml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: pytest
```

`main` 브랜치 push 또는 PR 생성 시 자동 실행됨. GitHub 저장소의 **Actions** 탭에서 결과 확인 가능.

---

## 7. 핵심 원칙 (왜 이렇게 설계하나)

1. **Fail Fast**: 빠른 단계부터 실행 (Lint → 유닛 → 통합 → E2E). 초반에 실패하면 뒤는 스킵해 시간 절약
2. **Idempotent**: 몇 번 돌려도 같은 결과. 랜덤 테스트 금지
3. **Reproducible**: 로컬/CI 환경 일치. Docker/Lock 파일로 보장
4. **Fast Feedback**: 10분 이내 목표. 느리면 개발자가 푸시 후 다른 일 하다 까먹음
5. **Pipeline as Code**: YAML로 버전 관리. "어제 Jenkins UI에서 설정 바꿨는데"는 안 됨

---

## 8. 한마디 요약

**CI/CD = "코드 push → 자동 테스트 → 자동 배포"로 이어지는 공장 라인**. CI는 품질 검증(버그 조기 발견), CD는 전달(신속한 운영 반영). 이 둘을 합친 자동화 흐름 전체가 파이프라인임.

---

## 참고 자료
- GitHub Actions 공식 문서: https://docs.github.com/ko/actions
- Martin Fowler, Continuous Integration: https://martinfowler.com/articles/continuousIntegration.html
- The Twelve-Factor App (배포 원칙): https://12factor.net/
- GitLab CI/CD 가이드: https://docs.gitlab.com/ee/ci/
