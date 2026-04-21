---
생성날짜:
- 2026-04-21 15:31
마지막수정날짜:
- 2026-04-21-화요일 15:31
tags:
- 설계원칙
- 디자인패턴
- Iterator
- GoF
- AI에이전트
별칭:
- Iterator
- 반복자 패턴
type:
- 자료수집
Area/Reasource:
Project:
---
# Iterator 패턴 (순회)

Iterator(이터레이터, 반복자)는 **컬렉션 내부 구조를 노출하지 않고** 원소를 하나씩 꺼내 쓰는 방법을 제공하는 행위 패턴(Behavioral Pattern)이다. 리스트든, 트리든, 그래프든, 파일 스트림이든 클라이언트는 `hasNext()` + `next()`(또는 파이썬 `__iter__` + `__next__`) 두 메서드만 알면 끝이다.

> GoF 23패턴 중 행위 패턴. "컬렉션 순회 책임을 컬렉션에서 분리하라"가 핵심.

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 분류 | 행위 패턴(Behavioral Pattern) |
| 핵심 아이디어 | 순회 책임을 Iterator 객체로 분리 |
| 풀려는 문제 | 컬렉션마다 순회 코드가 중복되고, 내부 구조에 의존함 |
| 대표 언어 지원 | Python `__iter__`, Java `Iterator`, JS Iterator Protocol |
| 연관 개념 | Generator(제너레이터, 지연 생성), Stream(스트림) |

## 일상 비유

음악 앱 재생목록 "다음 곡" 버튼이 딱 Iterator다. 사용자는 곡 목록이 배열인지 셔플 알고리즘인지 스마트 큐인지 몰라도 된다. 다음 버튼 한 번 누르면 다음 곡이 나오고, 목록 끝에 닿으면 멈춘다. 재생목록 내부 자료구조가 바뀌어도 버튼 UX는 그대로다.

## 구조

```
Client ──▶ Iterator
              │ hasNext() / next()
              ▼
           Aggregate(컬렉션) ──▶ createIterator()
```

컬렉션(Aggregate)은 "나를 순회할 도구"를 만들어 내놓고, 클라이언트는 그 도구의 `next()`만 반복해서 부른다.

## Python에서의 Iterator 프로토콜

파이썬은 Iterator 패턴을 언어 차원에서 흡수함. 다음 두 메서드만 있으면 `for` 루프가 돌아간다.

| 메서드 | 역할 |
| --- | --- |
| `__iter__(self)` | 자기 자신(또는 iterator 객체) 반환 |
| `__next__(self)` | 다음 원소 반환, 끝이면 `StopIteration` 예외 |

## AI 에이전트 개발 예시 1: LLM 스트리밍 응답 순회

LLM 응답은 토큰(token, 모델이 한 번에 만들어내는 글자 조각 단위)을 하나씩 흘려보냄. 이걸 Iterator로 감싸면 소비자 코드가 단순해진다.

```python
class LLMStreamIterator:
    """LLM 응답 스트림을 토큰 단위로 순회"""
    def __init__(self, stream):
        self._stream = stream      # 서버에서 오는 이벤트 제너레이터

    def __iter__(self):
        return self

    def __next__(self) -> str:
        event = next(self._stream)        # 이벤트 하나 받기
        if event.type == "done":
            raise StopIteration           # 끝났다고 파이썬에 알림
        return event.delta                # 이번에 새로 들어온 텍스트 조각

# 소비자 코드
for token in LLMStreamIterator(openai_stream):
    print(token, end="", flush=True)     # 타이핑 효과로 실시간 출력
```

소비자 코드는 "스트림 이벤트 구조"를 몰라도 된다. 이벤트 포맷이 OpenAI에서 Anthropic으로 바뀌어도 Iterator 내부만 바꾸면 끝.

## AI 에이전트 개발 예시 2: 문서 청크 순회 (RAG)

RAG(Retrieval-Augmented Generation, 검색 기반 생성)에서 긴 문서를 청크(chunk, 조각)로 쪼개 임베딩(embedding, 의미 벡터)을 만든다. 청크 생성 전략이 여러 개일 때 Iterator로 전략을 숨긴다.

```python
class ChunkIterator(ABC):
    @abstractmethod
    def __iter__(self): ...

class FixedSizeChunkIterator(ChunkIterator):
    def __init__(self, text: str, size: int = 500):
        self._text, self._size = text, size

    def __iter__(self):
        for i in range(0, len(self._text), self._size):
            yield self._text[i:i+self._size]   # 제너레이터로 이터레이터 자동 구현

class SentenceChunkIterator(ChunkIterator):
    def __iter__(self):
        for sentence in re.split(r'[.!?]\s', self._text):
            if sentence.strip():
                yield sentence

# 소비자 입장에서는 둘 다 똑같이 씀
def embed_all(chunks: ChunkIterator):
    return [model.embed(c) for c in chunks]
```

`embed_all`은 청크 분할 방식을 신경 쓰지 않음. 새 분할 전략(예: 시맨틱 청킹)이 생기면 Iterator 구현체만 하나 더 만들면 됨. [[SOLID원칙]]의 OCP 실현 예시.

## `yield`와 Generator 관계

파이썬 `yield`(이일드, 값을 내보내고 잠깐 멈춤)가 들어간 함수는 자동으로 Iterator를 만들어 주는 Generator(제너레이터) 문법 설탕. 아래 둘은 동등함.

```python
# 1. 제너레이터 함수
def counter(n):
    for i in range(n):
        yield i

# 2. Iterator 클래스로 직접 구현
class Counter:
    def __init__(self, n):
        self._n, self._i = n, 0
    def __iter__(self):
        return self
    def __next__(self):
        if self._i >= self._n:
            raise StopIteration
        self._i += 1
        return self._i - 1
```

단기적으로 쓰는 순회에는 제너레이터가 편함. 상태를 오래 들고 있어야 하거나 여러 Iterator가 동시에 돌아야 하면 클래스 버전이 유리함.

## 왜 이렇게 하는가

1. **컬렉션과 순회 로직 분리**: 컬렉션은 "데이터 보관"(SRP), Iterator는 "순회 책임"(SRP). 컬렉션이 트리에서 그래프로 바뀌어도 소비자 코드가 안 깨짐.
2. **지연 평가 가능**: 전체 리스트를 메모리에 올리지 않아도 됨. 1GB 로그 파일을 한 줄씩 읽는 게 가능.
3. **다중 순회 지원**: 같은 컬렉션에서 서로 독립된 Iterator 여러 개를 돌릴 수 있음(DFS Iterator, BFS Iterator 각각).

### 대안 대비 트레이드오프

| 접근 | 장점 | 단점 |
| --- | --- | --- |
| 컬렉션이 직접 `for_each` 제공 | 단순 | 순회 전략 바꾸기 어려움, 중간 중단 못함 |
| Iterator 분리 | 전략 교체, 지연 평가, 중단 | 객체 수 증가 |
| Callback 콜백 방식 | 간단 | 제어 역전, 디버깅 까다로움 |

## 직접 확인

파이썬 `iter()`와 `next()`를 수동으로 불러 보면 프로토콜이 드러남.

```python
it = iter([1, 2, 3])
print(next(it))   # 1
print(next(it))   # 2
print(next(it))   # 3
print(next(it))   # StopIteration 예외
```

`for x in [1,2,3]`은 내부적으로 이 과정을 그대로 돌린다.

## 포함 관계 및 관련 패턴

- Iterator는 [[Command 패턴(요청의 객체화)]]의 명령 큐를 "순회"하는 도구로도 쓰임.
- [[Proxy 패턴(대리 처리)]]를 Iterator에 씌우면 "페이지 단위로 API를 호출하며 투명하게 순회" 같은 구성이 가능(원격 페이징 반복자).
- 컬렉션 내부를 숨긴다는 점에서 [[SOLID원칙]]의 캡슐화 및 SRP와 맞닿아 있음.

## 한마디 요약

> **"컬렉션에게 '전부 다 줘'라 하지 말고, 옆에 세운 Iterator에게 '다음 거 하나'만 물어봐라. 내부 구조가 뭐든 너는 알 필요 없다."**

관련 문서:
- [[SOLID원칙]]
- [[Proxy 패턴(대리 처리)]]
- [[Command 패턴(요청의 객체화)]]
