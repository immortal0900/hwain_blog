# 동시성과 async/await

> LLM API를 100개 동시에 때리려다가 `requests`로는 안 된다는 걸 깨닫고 `asyncio`로 넘어오면서 정리한 내용. 원리를 모르면 `await` 잘못 써서 오히려 더 느려지는 경우가 생김.

## 한 줄 요약
**async/await는 "대기하는 동안 CPU를 놀리지 말고 다른 일 시켜줘"라고 OS가 아니라 내가 직접 스케줄링하는 기법이다.** 스레드가 "전화 여러 대를 여러 직원이 동시에 받는 것"이라면, async는 "전화 여러 대를 한 직원이 돌아가면서 받는 것"이다.

## 용어부터 정리

- **동시성(Concurrency, 여러 작업을 번갈아 진행해서 마치 동시에 되는 것처럼 보이게 하는 것)**: 작업들이 "겹쳐서" 진행됨. 반드시 같은 순간에 실행되진 않음.
- **병렬성(Parallelism, 여러 작업이 실제로 같은 순간에 다른 CPU 코어에서 실행되는 것)**: 물리적으로 동시 실행.
- **블로킹(Blocking, 호출한 함수가 끝날 때까지 호출자가 멈춰서 기다리는 것)**: `time.sleep(5)` 하면 5초 동안 그 줄에서 못 움직임.
- **논블로킹(Non-blocking, 함수가 즉시 반환하고 작업은 뒤에서 처리되는 것)**: 요청만 걸어두고 결과는 나중에 받음.
- **코루틴(Coroutine, 중간에 멈췄다 재개할 수 있는 함수)**: 파이썬의 `async def` 함수가 만들어 내는 객체.
- **이벤트 루프(Event Loop, 대기 중인 작업들을 돌아가며 깨우는 무한 반복기)**: `asyncio.run()`이 내부적으로 돌리는 핵심 엔진.

## 동시성 vs 병렬성 비교표

| 관점 | 동시성 (Concurrency) | 병렬성 (Parallelism) |
|---|---|---|
| **무엇** | 번갈아 실행 | 진짜 동시 실행 |
| **CPU 코어** | 1개로도 가능 | 2개 이상 필요 |
| **적합한 작업** | I/O 바운드 (네트워크, 디스크) | CPU 바운드 (연산, 학습) |
| **파이썬 도구** | `asyncio`, `threading` | `multiprocessing` |
| **비유** | 한 요리사가 국 끓이며 양파 썰기 | 요리사 3명이 각자 요리 |

**포함 관계**: 병렬성은 동시성의 한 종류다. 병렬이면 무조건 동시지만, 동시라고 병렬은 아님.

## 블로킹/논블로킹 × 동기/비동기 2×2

| | 동기(Sync) | 비동기(Async) |
|---|---|---|
| **블로킹** | `requests.get()` (기본) | 거의 없음 |
| **논블로킹** | 폴링 루프 (옛 방식) | `await httpx.get()` (현대 표준) |

## async/await 이벤트 루프 단계 분해

LLM 응답 3개를 동시에 받는다고 치자. 각 호출은 3초씩 걸림.

### 동기 방식 (`requests`)
```
0초  → 호출1 시작 (대기 중, CPU 놀고 있음)
3초  → 호출1 완료, 호출2 시작
6초  → 호출2 완료, 호출3 시작
9초  → 호출3 완료
총 9초
```

### async 방식 (`asyncio.gather`)
```
0초  → 호출1 요청 송신 → await로 멈춤, 이벤트 루프가 호출2로 점프
     → 호출2 요청 송신 → await, 호출3으로 점프
     → 호출3 요청 송신 → await, 모두 네트워크 대기 중
3초  → OS가 "호출1 응답 도착!" 알림 → 이벤트 루프가 호출1 재개
     → 이어서 호출2, 호출3 응답도 처리
총 약 3초
```

**포인트**: 이벤트 루프가 `await`를 만나면 **"이 작업은 지금 네트워크 기다리니까 다른 것부터 처리하자"**고 점프함. CPU가 노는 시간이 사라짐.

## 파이썬 코드로 체감하기

### 1단계: 동기 버전 (느림)
```python
import time, requests

def call_llm(i):
    r = requests.get(f"https://httpbin.org/delay/2")  # 2초 대기
    return i

start = time.time()
results = [call_llm(i) for i in range(5)]
print(f"동기: {time.time()-start:.1f}초")  # 약 10초
```

### 2단계: async 버전 (빠름)
```python
import asyncio, httpx, time

async def call_llm(i, client):
    r = await client.get("https://httpbin.org/delay/2")
    return i

async def main():
    async with httpx.AsyncClient() as client:
        tasks = [call_llm(i, client) for i in range(5)]
        return await asyncio.gather(*tasks)

start = time.time()
asyncio.run(main())
print(f"async: {time.time()-start:.1f}초")  # 약 2초
```

약 5배 차이. 호출 수가 늘수록 격차는 더 벌어짐.

## 직접 확인해 볼 수 있는 것

```python
import asyncio

async def task(name, delay):
    print(f"[{name}] 시작")
    await asyncio.sleep(delay)
    print(f"[{name}] 끝")

async def main():
    await asyncio.gather(
        task("A", 2),
        task("B", 1),
        task("C", 3),
    )

asyncio.run(main())
```

출력 순서:
```
[A] 시작
[B] 시작
[C] 시작
[B] 끝   ← 1초 후
[A] 끝   ← 2초 후
[C] 끝   ← 3초 후
```

시작은 거의 동시, 끝은 짧은 것부터. 이게 이벤트 루프가 돌아가는 증거.

## async가 절대 안 먹히는 함수

`await` 앞에 쓸 수 있는 함수는 **async로 작성된 라이브러리**여야 함. 그냥 `requests.get()`에 `await` 붙이면 에러. 대체 라이브러리를 찾아야 함.

| 동기 | async 버전 |
|---|---|
| `requests` | `httpx`, `aiohttp` |
| `time.sleep()` | `asyncio.sleep()` |
| `open()` (파일 I/O) | `aiofiles` |
| `psycopg2` | `asyncpg` |
| OpenAI SDK `OpenAI()` | `AsyncOpenAI()` |
| Anthropic SDK `Anthropic()` | `AsyncAnthropic()` |

## async에서 CPU 무거운 작업을 돌리면 안 되는 이유

```python
async def bad():
    # CPU를 5초간 잡아먹는 순수 계산
    total = sum(i*i for i in range(10**8))
    return total
```

`await`가 없으니 이벤트 루프가 **5초간 점프할 기회를 못 얻음**. 다른 async 작업 전부 정지. 이럴 때는 `run_in_executor`로 [[프로세스 vs 스레드|스레드/프로세스 풀]]에 넘겨야 함.

```python
loop = asyncio.get_running_loop()
result = await loop.run_in_executor(None, heavy_cpu_work)
```

## 일상 비유

- **동기 블로킹 = 은행 창구 1개**: 앞 사람 일 끝날 때까지 줄 서서 대기. 다른 일 못함.
- **멀티스레딩 = 창구 N개**: 창구 여러 개 열어서 동시 처리. 직원 인건비(메모리) 비쌈.
- **async/await = 창구 1개 + 똑똑한 직원**: 손님 A가 "잠시 서류 가져올게요" 하면 직원이 손님 B 받음. A 돌아오면 다시 A 처리. 직원 한 명으로 여러 손님을 효율적으로.

## 왜 이렇게 설계했나

- **[[프로세스 vs 스레드|스레드]]의 대안**: 스레드는 OS가 강제로 끊어가며 스케줄링하는데(선점형), 이게 컨텍스트 스위칭 비용이 있고 race condition 위험도 있음.
- **async는 협력적 스케줄링(cooperative scheduling)**: 프로그래머가 `await`로 "여기서 양보할게"라고 명시. 덕분에 락 없이 안전하고, 수만 개 코루틴도 메모리 부담 적음.
- **AI Agent에 최적인 이유**: LLM 호출, RAG 검색, DB 조회, 툴 실행 다 **네트워크 대기** 위주. CPU는 거의 안 쓰고 기다리는 시간이 대부분이라 async가 가장 효율적.

## 자주 하는 실수

1. **`async def` 안에서 `requests.get()` 호출**: 동기 함수라 이벤트 루프 멈춤. 반드시 async 버전 쓰기.
2. **`await` 안 붙이고 코루틴 호출**: `call_llm()`만 쓰면 코루틴 객체만 생기고 실행 안 됨. `await call_llm()` 또는 `asyncio.create_task()` 써야 함.
3. **CPU 바운드를 async로**: 위에 설명한 이벤트 루프 정지 문제.
4. **`asyncio.run()` 중첩**: 이미 이벤트 루프 안에서 또 `run()` 호출하면 에러. Jupyter에서는 `nest_asyncio` 필요할 때 있음.

## 한마디 요약
**async/await는 I/O 대기 시간에 다른 일 시키는 협력적 동시성. AI Agent처럼 외부 API 많이 때리는 시스템엔 사실상 필수.**

## 관련 문서
- [[프로세스 vs 스레드]]
- [[파일 I-O]]
- [[메모리 구조]]
