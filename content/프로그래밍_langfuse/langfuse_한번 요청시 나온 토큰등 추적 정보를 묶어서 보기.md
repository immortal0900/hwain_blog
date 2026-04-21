---
생성날짜:
  - 2026-03-08 22:27
마지막수정날짜:
  - 2026-03-08-일요일 22:26
tags:
  - langfuse
  - dashboard
  - sessions
  - trace
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
파이프라인 1회 실행(예: 40분짜리 E2E 작업) 동안 발생한 토큰/비용을 Langfuse에서 한 덩어리로 보려면 2가지 방법이 있다.

## 방법 1: Dashboards의 시간 필터 (가장 빠름)

좌측 메뉴 최상단 **[Dashboards]** 진입 → 우측 상단 **Date Range(기간 필터)** 조정.

- 파이프라인이 돌았던 구간(예: 오늘 20:00 ~ 20:50)으로 정확히 좁힌다
- 화면 상단에 해당 시간 동안의 **Total Cost(총 비용)**, **Total Tokens(총 토큰 수)**가 합산되어 큰 글씨로 뜬다

장점: 코드 수정 필요 없음. 단점: 같은 시간대에 다른 호출이 섞여 있으면 구분 불가.

## 방법 2: Sessions 탭 활용

참고 문서: https://langfuse.com/docs/observability/features/sessions

좌측 메뉴 **[Sessions]** 진입.

- 코드에서 호출 시 **Session ID**(예: E2E 작업의 `job_id`)를 같이 넘기도록 설정해 두면, **"파이프라인 1회 실행 = 1개의 세션"**으로 묶여서 표시됨
- 특정 세션 클릭 → 그 안에서 발생한 모든 Trace의 토큰 량과 총 비용이 자동 합산

주의: 현재 Sessions 탭이 비어 있다면 코드단에서 Session ID를 지정하지 않은 상태라는 뜻.

## 현재 상태 진단

스크린샷 기준으로 Trace 구조를 보면 LangGraph 전체가 하나의 부모 Trace 아래로 깔끔히 묶이지 않고 낱개로 흩뿌려지는 중.

파이프라인 1회 실행(40분) 전체를 한 줄의 Trace 또는 하나의 Session으로 묶고 싶다면 둘 중 하나가 필요하다.

1. 최상단에서 Langfuse 설정을 한 번 주입해서 하위 노드가 모두 상속받게 한다
2. `Session ID`를 명시하는 짧은 코드 보강을 넣는다

## 포함 관계

- **Session**(1회 실행 단위) ⊃ **Trace**(개별 요청 단위, 여러 개) ⊃ **Span**(각 요청 내부 작업 단위)
- Sessions 탭은 Session 단위로 합산, Dashboards는 시간 범위 기반 합산
- Session ID를 붙이면 Dashboards에서도 필터링 가능해짐 → 두 방법이 결국 연결됨

## 한마디 요약

빠르게 보려면 Dashboards 시간 필터, 재현 가능하게 묶으려면 Session ID를 코드에 박는다.

## 연관노트
- [[langfuse_한번 호출한 분량 분리해서 호출]]
- [[langfuse_propagate_attributes()의 원래 역할]]
