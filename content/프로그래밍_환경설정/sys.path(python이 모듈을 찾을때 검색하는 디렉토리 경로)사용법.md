---
생성날짜:
- 2026-01-30 04:05
마지막수정날짜:
- 2026-01-30-금요일 04:04
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
# sys.path.insert() 사용법

## 기본 개념

sys.path: Python이 모듈을 찾을 때 검색하는 디렉토리 경로 리스트

## 문법

```python
import sys
sys.path.insert(index, path)
```

- index: 삽입할 위치 (0 = 맨 앞, 최우선 검색)
- path: 추가할 디렉토리 경로 (문자열)

## 주요 사용 패턴

### 1. 최우선 검색 (가장 흔한 사용)

```python
sys.path.insert(0, '/my/custom/path')
```

- 다른 모든 경로보다 먼저 검색
- 같은 이름의 모듈이 여러 곳에 있을 때 우선순위 지정

### 2. 상대 경로로 프로젝트 루트 추가

```python
from pathlib import Path
sys.path.insert(0, str(Path(__file__).parent.parent))
```

- 현재 파일 기준으로 상위 디렉토리를 경로에 추가
- .parent 개수 = 올라갈 디렉토리 단계

### 3. 맨 뒤에 추가 (낮은 우선순위)

```python
sys.path.append('/fallback/path')  # insert 대신 append
```

## 검색 순서

```python
sys.path = ['A', 'B', 'C']
sys.path.insert(0, 'X')
# 결과: ['X', 'A', 'B', 'C']
# 검색 순서: X → A → B → C
```

## 주의사항

- 임포트 전에 실행: 모듈을 import하기 전에 경로를 추가해야 함
- 문자열로 변환: Path 객체는 str()로 변환 필요
- 절대 경로 권장: 상대 경로는 실행 위치에 따라 달라질 수 있음