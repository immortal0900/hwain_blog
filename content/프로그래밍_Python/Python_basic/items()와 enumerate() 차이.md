---
생성날짜:
  - 2026-04-24 11:38
마지막수정날짜:
  - 2026-04-24-금요일 11:38
tags:
별칭:
type:
  - 자료수집
Area/Reasource:
Project:
---
## items()와 enumerate() 차이

### 1. items(): "사전(Dictionary)에서 쌍으로 꺼낸다"

`items()`는 반드시 **사전(dict)** 데이터에서 사용한다. '이름(Key)'과 '값(Value)'을 한꺼번에 묶어서 꺼내준다.

Python

```
scores = {"A": 0.5, "B": 0.9} # 사전 데이터

for name, score in scores.items():
# [for]: 반복을 시작한다
# [name, score]: 꺼낸 '이름'과 '점수'를 각각 이 변수들에 담는다
# [in scores.items()]: scores 사전에서 (이름, 점수) 쌍을 하나씩 통째로 꺼낸다
    print(name, score)
```

### 2. enumerate(): "목록(List)에서 순서와 함께 꺼낸다"

`enumerate()`는 주로 **리스트(list)**에서 사용한다. 데이터와 함께 **"얘가 몇 번째인지(번호)"**를 덤으로 붙여준다.

Python

```
ranking = ["doc1", "doc2", "doc3"] # 리스트 데이터

for rank, doc_id in enumerate(ranking, start=1):
# [for]: 반복을 시작한다
# [rank]: 덤으로 받은 '순서 번호'를 담는다
# [doc_id]: 리스트 안에 들어있던 '실제 데이터'를 담는다
# [enumerate(ranking, start=1)]: 리스트를 순회하되, 1번부터 번호를 매겨서 같이 가져온다
    print(rank, doc_id)
```

---

## 핵심 비교 요약

|**구분**|**items()**|**enumerate()**|
|---|---|---|
|**대상**|**사전(Dictionary)** 전용|**리스트(List)**, 튜플 등|
|**꺼내는 것**|이름(Key) + 값(Value)|**번호(Index)** + 실제 데이터|
|**특징**|원래 짝지어 있던 데이터를 꺼낸다|원래 없던 **번호표를 즉석에서 만든다**|
|**비유**|서랍장에서 (라벨, 내용물)을 꺼낸다|줄 선 사람에게 (번호표, 사람)을 부여한다|

---

## RRF 코드에서의 역할

- **`enumerate(ranking, start=1)`**: 리스트의 문서들을 순서대로 꺼내며, 계산에 필요한 **순위 번호(1, 2, 3...)**를 생성한다.
    
- **`scores.items()`**: 점수 계산이 끝난 후, 정렬을 위해 사전에 저장된 **문서 식별자(ID)와 점수**를 한 쌍으로 묶어 모두 가져온다.
    
