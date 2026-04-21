---
생성날짜:
- 2026-04-21 12:18
마지막수정날짜:
- 2026-04-21-화요일 12:18
tags:
- 설계원칙
- 디자인패턴
- GoF
- 생성패턴
- AI에이전트
- 안티패턴논쟁
별칭:
- Singleton
- 싱글턴
- 단일 인스턴스
type:
- 자료수집
Area/Reasource:
Project:
---
# Singleton 패턴이란?

Singleton(싱글턴, 단일 인스턴스 패턴, 한 클래스의 인스턴스가 프로세스 전체에 단 하나만 존재하도록 강제하는 생성 디자인 패턴)은 **"어디서 불러도 같은 객체를 돌려주는"** 패턴이다. GoF 분류상 **생성(Creational) 패턴**에 속함.

현대에는 "쓰지 마라"는 의견이 많은 **논쟁적인 패턴(controversial pattern)** 임. 필요한 상황과 피해야 할 상황을 구분해 이해하는 게 핵심.

## 한눈에 보기

| 항목     | 내용                                          |
| ------ | ------------------------------------------- |
| 분류     | 생성 패턴(Creational)                           |
| 한 줄 정의 | 한 클래스의 인스턴스가 단 하나만 존재하도록 보장                 |
| 핵심 구성  | private 생성자, 클래스 변수에 유일 인스턴스 보관, get_instance() |
| 해결 문제  | 전역 리소스(설정, 커넥션 풀, 캐시, 로거)의 중복 생성 방지         |
| 안티패턴 지적 | 숨은 전역 상태, 테스트 어려움, 멀티스레드 이슈                 |

> **한마디 요약**: "이 타입은 지구에 딱 하나. 새로 만들지 말고 가져다 써라."

---

## 일상 비유

국가는 하나, 대통령도 한 명, 현재 시각도 하나다. 아무리 많이 물어봐도 "대한민국의 대통령을 주세요" 하면 같은 사람이 나온다.

다른 비유:
- 게임의 저장 파일(sav 파일 하나, 여러 창에서 동시 편집 금지)
- 프린터 스풀러(운영체제에 하나, 모든 앱이 공유)
- 건물의 중앙 보일러(방마다 따로 두지 않고 하나로 전체 공급)

---

## 구조

```
┌──────────────────────┐
│     Singleton        │
├──────────────────────┤
│ - _instance (static) │
│ - __init__()         │ ← 외부 호출 제한
├──────────────────────┤
│ + get_instance() ──→ │ 있으면 기존 것, 없으면 새로 생성
└──────────────────────┘
```

---

## Python 구현 6가지

Python은 언어 특성상 순수 Singleton을 막기 어려운 면이 있어 여러 관용구가 쓰인다.

### 1. 모듈 레벨 전역 (가장 Pythonic)

```python
# config.py
class _Config:
    def __init__(self):
        self.api_key = os.getenv("API_KEY")

config = _Config()  # 모듈 로드 시 한 번 생성
```

```python
# 다른 파일
from config import config
print(config.api_key)
```

Python 모듈 자체가 import 캐시되므로 자연스럽게 싱글턴처럼 동작함. **Python에서 가장 권장되는 방식**.

### 2. `__new__` 오버라이드

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

s1 = Singleton()
s2 = Singleton()
assert s1 is s2  # True
```

주의: `__init__`은 매번 호출되어 상태가 덮어쓰일 수 있으니 초기화 플래그 필요.

### 3. 메타클래스 활용

```python
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Config(metaclass=SingletonMeta):
    def __init__(self):
        self.loaded_at = time.time()
```

### 4. 데코레이터 방식

```python
def singleton(cls):
    instances = {}
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class Logger:
    ...
```

### 5. `functools.lru_cache` 활용

```python
from functools import lru_cache

@lru_cache(maxsize=1)
def get_config() -> Config:
    return Config()
```

함수 결과를 캐시해 사실상 싱글턴. 테스트 시 `get_config.cache_clear()` 가능해 유연함.

### 6. 의존성 주입 컨테이너

대형 앱에서는 직접 구현보다 `dependency-injector` 같은 DI 컨테이너로 `Singleton` 스코프를 선언하는 게 깔끔함.

---

## 나쁜 예 vs 좋은 예

### 나쁜 예: 매번 새 커넥션 풀 생성

```python
def query_user(user_id):
    engine = create_engine("postgresql://...")  # 호출마다 풀 생성
    with engine.connect() as conn:
        return conn.execute(...)
```

요청이 쏟아지면 DB 커넥션 풀이 수백 개 생성되어 DB가 터짐.

### 좋은 예: 프로세스에 하나

```python
# db.py
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql://...",
    pool_size=10,
    max_overflow=20,
)
```

```python
from db import engine

def query_user(user_id):
    with engine.connect() as conn:
        return conn.execute(...)
```

모듈 import 캐시 덕에 `engine`은 프로세스에 단 하나만 존재.

---

## AI 에이전트 개발 예시: LLM 클라이언트 / 벡터스토어 연결

AI 에이전트에서 싱글턴이 어울리는 자원들:

| 자원                 | 이유                               |
| ------------------ | -------------------------------- |
| LLM API 클라이언트      | HTTP 커넥션 풀, 재시도 정책, 토큰 카운터 공유    |
| 벡터스토어 연결           | 연결 풀, 인덱스 캐시 유지                  |
| 임베딩 모델 (로컬)        | GPU/RAM 로드 비용이 큼. 여러 번 로드하면 OOM  |
| 캐시 (LRU, Redis 풀)  | 캐시는 공유되어야 의미가 있음                 |
| Langfuse/OTEL 트레이서 | 전역 이벤트 수집기                       |
| 설정(Config)         | 환경 변수/시크릿의 단일 출처                 |

```python
# llm_client.py
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("OPENAI_API_KEY"),
    timeout=30.0,
    max_retries=3,
)
```

```python
# 임베딩 모델 (로컬 허깅페이스)
from functools import lru_cache
from sentence_transformers import SentenceTransformer

@lru_cache(maxsize=1)
def get_embedder():
    return SentenceTransformer("BAAI/bge-m3")  # 2GB+ 메모리
```

`get_embedder()`를 몇 번 호출해도 모델은 한 번만 GPU에 올라감.

### 안 맞는 경우: 대화 세션

"사용자별 대화 컨텍스트"는 절대 싱글턴으로 두면 안 됨. 사용자 A와 B의 대화가 섞여버림. 이 경우 요청별 인스턴스 또는 유저 ID를 키로 한 딕셔너리가 맞다.

---

## 싱글턴이 욕먹는 이유 (안티패턴 논쟁)

### 1. 숨은 전역 상태
호출부 시그니처만 봐서는 의존성을 알 수 없다. 어느 함수가 `Config.get_instance()`를 쓰는지 추적하려면 전체 코드를 뒤져야 함.

### 2. 테스트가 어려움
```python
def test_something():
    Config.get_instance().api_key = "test-key"  # 다른 테스트로 누수
    ...
```
이전 테스트가 남긴 상태가 다음 테스트에 섞임. `setUp/tearDown`에서 리셋 로직이 필요해짐.

### 3. 멀티스레드/멀티프로세스 문제
순진한 구현은 동시 생성 경쟁(race condition)이 생김. Double-checked locking 등의 대책이 필요.

```python
import threading

class SingletonMeta(type):
    _instances = {}
    _lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]
```

### 4. DIP 위반 소지
`Config.get_instance()`는 구체 클래스 직접 호출 = 추상 의존이 아님. 테스트 시 Mock 주입이 어려움. **의존성 주입** 방식이 현대적 대안.

---

## 언제 써도 괜찮은가

| 쓸만함                          | 쓰지 말 것                          |
| ---------------------------- | -------------------------------- |
| 설정, 로거, 커넥션 풀, 캐시 같은 **진짜 전역 리소스** | 비즈니스 객체, 도메인 엔티티                 |
| **읽기 전용(immutable)** 설정      | 읽고 쓰는 상태를 여기저기서 공유하려는 목적         |
| 생성 비용이 매우 큰 리소스(ML 모델 로드)    | "코드 편하려고" 단순 편의성 목적             |
| 프로세스 수명 동안 **상태가 바뀌지 않음**    | 테스트마다 리셋이 필요한 객체                 |

---

## 대안들

현대 설계에서는 직접 구현보다 다음을 우선 검토함.

1. **의존성 주입(DI)**: 생성자 파라미터로 전달, 호출부에서 명시적 의존성 노출
2. **DI 컨테이너의 Singleton 스코프**: `dependency-injector`, FastAPI `Depends`
3. **모듈 레벨 전역** (Pythonic)
4. **`functools.lru_cache(maxsize=1)`**: 함수 결과 캐싱

DI 쪽이 테스트/유지보수 측면에서 우월하므로, 신규 코드에서는 이쪽을 먼저 고려할 것.

---

## 멀티프로세스 주의

Singleton은 **프로세스 단위**로 유일함. 파이썬 `multiprocessing`, Gunicorn 워커, Celery 워커마다 각각 새 인스턴스가 생성됨. "전역"이라고 착각하면 안 됨.

진짜 모든 워커가 공유해야 하는 데이터(예: 실시간 사용자 세션)는 Redis, Memcached 같은 외부 저장소로 가야 함.

---

## 실전 주의점

### 1. 초기화 순서
여러 싱글턴이 서로 참조하면 import 순서에 따라 문제가 생김. 지연 초기화(lazy init)로 해결.

### 2. 직렬화
싱글턴을 `pickle` 했다가 다른 프로세스에서 복원하면 유일성이 깨진다. `__reduce__` 정의 필요.

### 3. 테스트 격리
전역 상태는 테스트 사이 리셋되지 않는다. pytest fixture로 매번 fresh instance를 강제할 것.

```python
@pytest.fixture(autouse=True)
def reset_config():
    yield
    _Config._instance = None
```

---

## 다른 패턴과의 관계 (포함 관계)

- **[[Factory 패턴|Factory]]와 자주 결합**: Factory 자체를 싱글턴으로 두는 경우 많음
- **[[SOLID원칙|DIP]]와 충돌 가능**: 직접 `get_instance()` 호출은 구체 의존. DI가 대안
- **Multiton 패턴**: 키별로 하나씩 존재하는 변형 (예: `UserSession.get(user_id)`)
- **Borg 패턴**: 인스턴스는 여러 개지만 상태를 공유하는 파이썬 관용구
- **모노스테이트(Monostate)**: Borg와 유사, 상태 공유로 같은 효과

---

## 직접 확인하기

자기 코드에서 `get_instance()` 또는 모듈 전역으로 쓰는 객체를 목록화해봐라. 각 항목마다 다음을 질문해볼 것.

- [ ] 상태가 **진짜 전역**이어야 하는가, 아니면 사용자/요청별로 달라야 하는가?
- [ ] 테스트에서 이 상태를 쉽게 격리/리셋할 수 있나?
- [ ] 의존성 주입으로 바꿔도 되지 않나?
- [ ] 멀티프로세스에서 "공유되지 않음"을 알고 있나?

대답이 애매하면 싱글턴 대신 DI로 옮겨볼 것.

---

## 요약

> **"정말 전역 자원일 때만 싱글턴. 편하다는 이유로 쓰면 나중에 테스트와 멀티스레드에서 복수당한다. Python이면 보통 모듈 전역으로 충분."**

관련 문서:
- [[SOLID원칙]]
- [[Factory 패턴]]
