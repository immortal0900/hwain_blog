
> **SUPER DEVELOPER PROJECT 3부작 시리즈**
> - 1편 (현재): 서론 + THE FORGE: 정밀 대장간
> - 2편 (예정): MEMEX: 평생 축적되는 지식 그래프
> - 3편 (예정): ENGRAM: 기억을 새기는 촉매

# SUPER DEVELOPER PROJECT

주니어가 시니어를 따라잡으려면 어떻게 해야 할까?

천천히 걸어서는 방법이 없다. **무조건 뛰어야 한다.** 시간을 쏟아부어야 한다.

하지만 그것만으론 역부족이다.

시대가 변했다. **기존에 걷던 그들도 이제는 전동 킥보드를 타고 달리고 있다.** 무작정 뛴다고 따라잡히지 않는다. 
그렇기에 무작정 뛰어서는 방법이 없다. 그렇기에 개발을 보조해줄 수단들을 생각했다.

현대 개발에서 LLM은 빠질 수 없는 존재가 되었다. 개발 보조 수단은 그 존재의 단점을 보완하고 나 스스로의 견문을 넓히는 방법이다.

그래서 세 개의 프로젝트를 기획했다. 마치 마치 원피스에 나오는 3개의 고대병기 처럼 각각의 프로젝트에 이름을 붙였다.~~(사실 최신병기)~~

![[제목 없는 디자인 (3).png]]
### THE FORGE: 정밀 대장간

> Harness 패턴으로 **Evaluator가 Generator를 계속 때려** 더 나은 결과물로 수렴시키는, 무엇이든 찍어낼 수 있는 정밀 대장간.

LLM을 통제해 **실제로 쓸 수 있는 산출물**을 뽑아내는 시설.


### MEMEX: 축적되는 기록

> *"개인이 평생 본 모든 자료를 연관 경로(associative trails, 항목 간의 연결 궤적)로 엮어 언제든 꺼낼 수 있는 장치."* 
> — Vannevar Bush, *As We May Think* (1945)

Graph DB로 LLM과 인간이 **같은 시행착오를 두 번 겪지 않게 하는**, 평생 축적되는 자산.


### ENGRAM: 각인의 촉매

> 신경과학에서 **"기억이 뇌에 남긴 물리적 흔적"** 을 가리키는 용어.

MEMEX가 **기록**이라면, ENGRAM은 그 기록을 **몸에 새기는 주체**. Open WebUI 위에 얹어 **인간 자체의 성능을 끌어올리는 촉매**로 작동한다.


# 목차
1. [[#MEMEX 축적되는 기록_서론]] 
2. [[#ENGRAM 각인의 촉매_서론]] 
3. [[#THE FORGE 정밀 대장간_서론 + 본론]] 


---

# MEMEX: 축적되는 기록_서론
시니어들이 주니어와 다른게 무엇일까? 그것은 무수한 시행착오로 인해 생긴 경험일 것이다.
경험이 무엇인가 그 시행착오를 다시 겪지 않게 하는 혹은 같은 문제가 생겼을 경우 유연하게 넘어 갈 수 있게 하는 개인의 자산이다.

사람은 모든일을 다 기억하지 못한다 그렇기에 시행착오가 어느정도 누적이 되어야 즉, 같은일이 반복이 되어야 해당 문제에게 대해 온전하게 대응할 수 있고, 그걸 우리는 "경험"이라 부른다.

그렇다면 한번의 시행착오만 거쳐도 같은 문제가 발생했을때 대응이 가능하다면?
경험의 차이를 빠르게 따라잡을 수 있을 것이다.
그걸 가능하게 하는 것이 **개인화 DB**라고 생각한다.

---
### LLM 메모리의 한계

LLM은 세션이 바뀔 때마다 새롭게 시작한다. 이전의 기억이 없다는 말이다.
요즘은 Memory 기능이 꽤 잘 되어 있어 기억을 해주긴 한다. 하지만 딱 거기까지다.

- 개인의 데이터 메모리 용량은 한정되어 있고**용량이 한정되어 있어** 계속 요약된다. 즉 **단기기억** 수준이다. 
- 친구가 *"아 그때 말했던 그거?"* 하고 맞장구 쳐주는 정도의 기억이다. 
- 무엇보다 **저장 방식이 통제 불가능한 블랙박스**다.

### 어떻게 극복할 것인가?

**데이터를 조회할 때도, 저장할 때도 내 DB와 연결된 MCP를 경유하도록 한다.** 이렇게 하면 Claude Code든 SaaS LLM이든 상관없이 **같은 내 DB에 접근**할 수 있다.

### Karpathy의 LLM Wiki

Andrej Karpathy의 LLM Wiki 글을 읽었다. 그는 LLM을 코드 생성용이 아니라 **개인 지식 위키를 구축하는 데** 쓰고 있다고 하면서, 현재의 RAG 방식을 이렇게 꼬집었다. 
> "LLM이 매 질문마다 처음부터 지식을 재발견(rediscover)하는 것이다. 누적(accumulation)이 없다."
> **출처**: 
> https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

그의 제안은 3계층 구조다.

- **Layer 1 - RULE BOOK Schema**: LLM이 항상 읽고 있는 규칙 
- **Layer 2 - Wikilinks**: 원본으로 가는 링크 모음 
- **Layer 3 - 원본 자료 보관소**

이런식으로 평소에는 Layer 1 과 Layer 2 를 읽은 상태로 대화를 하다가, 뭔가 조사가 필요할 때 비로소 wikilinks를 보고 원본을 찾는다. 
100건 내외의 문서들은 LLM이 RAG를 사용할 필요가 없다는 것이다. 

### "이것도 결국 RAG다"

그런데 문득 그런 생각이 들었다. '이것도 RAG인데?'
작은 청크로 찾고, 큰 청크로 답하는 **Parent Document Retriever** 와 구조가 비슷하다는 생각이 들었다. 
하지만 그만큼 구현 복잡도가 줄어들기 때문에 화제가 되었던거 같다.

하지만 이 방식은 평생 지속할 수 없는 방식이다. 내가 원하는 것은 누적되는 개인지식이다.
Andrej Karpathy 의 방식은 결국 Layer 2 가 계속 누적될 수 밖에 없다.
그럼 결국 컨텍스트를 차지하게 되고 그럼 LLM은 엉뚱한 답을 내놓을 확률이 높아지게 될 것이다.

"Karpathy도 Bush의 Memex를 언급했지만, 그의 방식은 아직 Bush가 꿈꾼 '연상'까지 가지 못했다. Graph DB가 바로 그 연상 구조다."

때문에 나는 애초에 Graph DB 형태로 저장하는 방법을 선택했다.

### "굳이 Graph DB?"

이런 생각이 들 수 있다. 'VectorDB 에서 HNSW 인덱서도 그래프 형태로 묶이는데 굳이 Graph DB를 쓸 필요가 있나?'

VectorDB는 의미적으로 유사한 것 끼리 모여있는 것이다. 따라서 질문과 유사한 내용이 검색되는 것이다.
그렇기에 쿼리를 넣으면 해당 질문과 의미적으로 유사한 내용이 나온다.

차이는 **"무엇으로 연결되어 있느냐"**이다. 
- **VectorDB**: 의미적으로 유사한 것끼리 모여 있다 → 질문과 **비슷한 내용**이 나온다. 
- **Graph DB**: 명시적 관계로 연결되어 있다 → *"A 때문에 B 에러가 발생했고, C로 해결했다"* 같은 **인과관계를 논리적 맥락 그대로** LLM에 넘길 수 있다. 

물론 VectorDB도 전에 했던 [NPC 프로젝트](https://www.notion.so/MEMORIA_LABYRINTH-AI-2d9aa1b417488104bc73c233292a9c66?source=copy_link#30faa1b417488089b925f1bc85b1ebf5) 처럼 저장 시 LLM을 이용해 인과관계 형태로 넣을 수는 있다. 
하지만 *"이 에러와 관련된 결정사항 중, 기각된 대안은 무엇인가?"* 같은 **다단계 추론(multi-hop reasoning)**에서는 Graph DB가 본질적으로 유리하다. 

### Graph + Vector + Keyword 하이브리드 

결정적으로, **Neo4j는 HNSW 벡터 검색과 BM25 키워드 검색을 모두 지원한다.** 
즉 한 DB 안에서 세 가지를 동시에 쓸 수 있다. 
그래서 최종 선택은 **Graph + Vector + Keyword 하이브리드 검색**이다. 
- **벡터 검색이 필요한 이유**: 초반에는 형성된 엔티티가 적고, Graph 검색이 엔티티를 정확히 뽑아낸다는 보장이 없다. 
- **키워드 검색(BM25)이 필요한 이유**: 고유명사나 정확한 단어 매칭이 필요한 경우에 강하다. 
- **Graph 검색이 중심인 이유**: 누적된 관계망을 따라 **맥락과 인과를 보존한 채** LLM에 전달할 수 있다. 저장되는 내용은 개발 관련 질의응답, 개발 중 발생한 에러·근본원인·해결책, 결정사항과 기각된 대안 등이다.


---
**목차로 이동 → [[#목차]]**
# ENGRAM: 각인의 촉매_서론

LLM의 성능만 올리뭔 뭐하나 조종수가 명품이 되어야 한다.
앞서 **경험이란 같은 일을 여러 번 겪어본 것의 누적**이라고 말했다. 
그런데 사람은 자꾸 까먹는다. 그래서 시행착오가 어느 정도 누적되기 전까지는 경험이 제 역할을 못한다.

**ENGRAM(엔그램)은 MEMEX에 저장된 내용을 적절한 타이밍에 다시 꺼내 보여주는 장치다.** 
즉 **경험의 누적 속도를 인위적으로 끌어올리는 것**이 ENGRAM의 역할이다.

---
### 그럼 "적절한 타이밍"은 언제인가?

1,350명 이상의 참가자를 대상으로 한 대규모 연구는 단순한 법칙 하나를 제시한다. 
**기억하고자 하는 기간의 10~20% 시점에 다시 보면 된다.**

> **출처**: Cepeda, N. J., Vul, E., Rohrer, D., Wixted, J. T., & Pashler, H. (2008). 
> *Spacing effects in learning: A temporal ridgeline of optimal retention*. 
> **Psychological Science, 19(11), 1095–1102.**

이걸 실제 일정으로 풀어보면 이렇다.

- **1주일 뒤에 기억하고 싶으면** → 오늘 공부하고 내일 한 번 더 본다.
- **한 달 뒤 까지 기억하고 싶으면** → 오늘, 내일, 일주일 뒤에 본다.
- **1년까지 살려두고 싶은 지식이라면** → 오늘, 내일, 일주일 뒤, 3주 뒤, 두 달 뒤, 다섯 달 뒤에 본다.

핵심은 **처음에는 짧은 간격으로 시작해서 갈수록 간격을 늘리는 것**이다. 
매번 직전 간격의 약 2~2.5배씩. 
의사들이 시험공부에 널리 쓰는 학습 앱 ANKI도 바로 이 원리로 움직인다.

### 기존 지식과 연결될 때 더 잘 배운다

뇌는 정보를 있는 그대로 저장하지 않는다. **기존 스키마(schema, 특정 주제에 대해 이미 형성된 지식의 틀)에 맞춰 저장한다.** 
이미 아는 것과 연결되는 정보일수록 더 빠르게, 더 깊이 새겨진다는 의미다.

그래서 MEMEX의 Graph RAG(그래프 기반 검색 증강 생성)가 질문과 관련 있는 노드 하나만 던져주는 것이 아니라
해당 엔티티(entity, 지식 그래프의 노드 단위: 사람·개념·사건 등)와 **연결된 주변 엔티티와 관계까지 함께** 꺼내줘야 한다. 
그래야 새 지식이 기존 지식망에 접속되면서 장기기억으로 빠르게 자리 잡는다.

> **출처**:
> - Bartlett, F. C. (1932). *Remembering: A Study in Experimental and Social Psychology*. Cambridge University Press. (스키마 이론의 원전)
> - van Kesteren, M. T. R., Ruiter, D. J., Fernández, G., & Henson, R. N. (2012). *How schema and novelty augment memory formation*. **Trends in Neurosciences, 35(4), 211–219.** DOI: 10.1016/j.tins.2012.02.001

### 뇌는 리스트가 아니라 지도로 기억한다

"A → B → C → D"라는 순서를 외울 때 뇌는 일렬로 된 리스트로 저장하지 않는다. 
마치 지하철 노선도처럼 **"A는 출발점, D는 종점, B는 A 근처"**라는 공간 관계로 저장한다. 
**순서대로 훑는 것이 아니라 위에서 한 번에 내려다보는 형태**로 기억한다는 의미이다.

이 구조는 Graph  DB의 형태와 유사하다. 
따라서 **뇌가 이미 그래프로 기억하고 있으니, 그래프로 검색하는 것이 자연스러운 접근이다.**
>출처:**
>- Zhang, Y., Wang, L., Dehaene, S., et al. (2022). _Geometric shapes in the brain: Language and symbolic abilities shape how humans perceive geometric regularities_. **Proceedings of the National Academy of Sciences** 계열 연구.
>- Behrens, T. E. J., Muller, T. H., Whittington, J. C. R., et al. (2018). _What is a cognitive map? Organizing knowledge for flexible behavior_. **Neuron, 100(2), 490–509.** DOI: 10.1016/j.neuron.2018.10.002

### 꺼내보는 행위에서 기억이 강화된다

인간의 뇌는 **저장할 때가 아니라 꺼낼 때** 기억이 강화된다. 
수십 번 읽은 책보다 한 번 가르쳐본 책이 오래 남는 이유가 여기에 있다.

따라서 ENGRAM은 저장했던 내용을 **그대로 보여주지 않는다.** 
LLM이 질문 형태로 되묻도록 설계했다. 매 복습이 작은 시험이 되도록 만든 것이다.

> **출처**:
> - Roediger, H. L., & Karpicke, J. D. (2006). *Test-enhanced learning: Taking memory tests improves long-term retention*. **Psychological Science, 17(3), 249–255.** DOI: 10.1111/j.1467-9280.2006.01693.x
> - Karpicke, J. D., & Blunt, J. R. (2011). *Retrieval practice produces more learning than elaborative studying with concept mapping*. **Science, 331(6018), 772–775.** DOI: 10.1126/science.1199327

### 기획 동기

ANKI를 써보려고 한 적이 있다. 그런데 **질문 카드를 하나하나 직접 만들어야 하는 그 수작업**이 귀찮아서 자연스럽게 멀어지게 되었다.


그래서 이 프로젝트를 기획했다. 
이미 그래프 형태로 쌓여 있는 개인 DB를, **관련 엔티티와 관계까지 묶어서**, 
**다음 날부터 시작해 매번 직전 간격의 약 2~2.5배씩 늘어나는 지점**에, 
**질의응답 형태로** 자동 노출한다.

저장 따로, 복습 따로, 카드 작성 따로가 아니라 **한 줄기로 흐르게** 만드는 것이 ENGRAM의 목표다.

이 프로젝트는 Open Claw를 이용해 진행할 예정이다.


---
**목차로 이동 → [[#목차]]**
# THE FORGE: 정밀 대장간_서론 + 본론

이 글을 쓰는 시점에서 **THE FORGE는 이미 시범 구동까지 완료된 상태**다. 
https://github.com/immortal0900/THE_FORGE.git

THE FORGE는 **Planner → Generator → Evaluator 세 에이전트가 스프린트 단위로 협업해 코드를 생산하는 개인용 멀티에이전트 하네스(harness, 여러 LLM 호출을 조율하는 실행 골격)**다.

언뜻 보기엔 개인화 DB인 MEMEX가 더 우선일 거 같지만 MEMEX는 **데이터가 누적되어야 힘을 발휘**하는 반면, THE FORGE는 **만들자마자 바로 전력화**가 가능하다. 
또한 THE FORGE로 나머지 두 개(MEMEX, ENGRAM)의 개발 속도도 끌어올릴 수 있다. 
그래서 THE FORGE를 먼저 개발하게 되었다.

---

## 1. 왜 만들었나: LLM 통제의 한계

### LLM을 "잘 다룬다"는 것의 의미

현대 개발에서 LLM은 빠질 수 없는 존재가 되었다. 그런데 **"LLM을 잘 다룬다"**는 건 무엇을 의미할까?

**LLM을 잘 통제한다는 것이다.**

LLM의 성능은 꾸준히 좋아지고 있지만, 바로 그 **비결정론적 특성(non-deterministic, 같은 입력에도 매번 다른 출력이 나오는 성질)** 때문에 기업들은 제품화를 꺼려 왔다. 
현업자들 역시 LLM을 단순 반복 작업용으로만 활용했다. 
그렇기에 2025년 11월 [부동산 분양가 분석 멀티에이전트 프로젝트](https://root-waterlily-68c.notion.site/ALL_FOR_ONE-31daa1b4174880ebbfd5dc885ca69ee7?source=copy_link) 당시, 190개의 부동산 관련 회사가 가입된 협회 사무국장님께서 *"현재 부동산업계에서 이 주제로 에이전트를 활용하는 기업이 없다"*라고 말씀하신 것이다.

### 컨텍스트 포화: LLM이 브레인포그에 빠지는 순간

그럼 어떻게 통제할 것인가?

몇 달 전만 해도 LLM은 프롬프트를 정말 잘 작성해야 원하는 방향으로 생성을 해줬다. 지금은 적당히 말해도 찰떡같이 알아듣는다. **하지만 딱 거기까지다.**

컨텍스트가 어느 정도 포화 상태가 되면 LLM은 **브레인포그(brain fog, 머릿속이 안개 낀 듯 흐려지는 상태)에 걸린 사람처럼 날카로움을 잃어버린다.**

- 질문을 잘못 해석한다.
- 이전에 제공한 맥락을 잃어버린다.
- 코딩 에이전트는 지시에 없는 파일을 자기 멋대로 삭제하기도 한다.
- 많은 양의 컨텍스트를 처리하느라 레이턴시가 폭증한다. (Cursor로 한 달에 70만 원을 쓰고 깨달았다)

경험상 Claude Code는 **컨텍스트 윈도우 100만 토큰 중 60%만 넘어도** 위와 같은 증상이 나오기 시작한다.

### 오염된 답변의 자가복제

더 큰 문제는 이것이다. **한 번 이상한 답변을 하면, 그 이상한 답변이 메모리에 남아 계속 잘못된 방향으로 흘러간다.**

LLM이 Few-shot(몇 개의 예시를 프롬프트에 넣어 학습 효과를 내는 기법)을 주면 답변을 더 잘하는 것처럼, **프롬프트에 누적된 메모리를 학습 데이터처럼 참고해서 생성**하기 때문인 것으로 보인다. [NPC 프로젝트](https://www.notion.so/MEMORIA_LABYRINTH-AI-2d9aa1b417488104bc73c233292a9c66?source=copy_link#310aa1b4174880eba0a6edb278b3dcf0)에서도 *기억이 돌아왔다는 시점에서 이제 기억한다고 답해야 하는데, 이전에 "기억 안 난다"고 답했던 메모리를 더 참고해버려서 계속 기억이 안 난다고 우기던* 문제로 골머리를 썩인 적이 있다.

**결론은 하나다. 한 세션에서 컨텍스트를 최대한 적게 유지해야 하며, 낌새가 이상하다 싶으면 바로 새로운 세션을 여는 게 상책이다.**


## 2. Harness 패턴: 세션을 쪼개라

답은 **세션을 새로 여는 것**을 알았다.

Anthropic이 2025년 11월 공개한 **Harness 패턴**[^1]을 처음 봤을 때, [부동산 분양성 평가 프로젝트](https://www.notion.so/ALL_FOR_ONE-31daa1b4174880ebbfd5dc885ca69ee7?source=copy_link#328aa1b41748808ea5f5e8e4f9464c81)에서 구현한 **ReAct + Reflection 패턴**과 비슷한 것 아닌가 싶었다.

하지만 결정적으로 달랐다.

- **부동산 프로젝트의 ReAct + Reflection**: 하나의 세션에서 평가와 수정을 진행.
- **Harness 패턴**: **평가와 생성이 각각 다른 세션**에서 이루어짐.

세션을 분리한 이유는 **자기 선호 편향(Self-preference Bias, 자기 작품에 후한 점수를 주는 경향)**을 다른 세션을 통해 해결하기 위함이다. 

또한 주고받는 컨텍스트도 **메모리가 아닌 문서**로 전달된다. 
앞서 말했듯 컨텍스트가 누적되면 맥락을 잃기 때문에, Claude Code 프로젝트에서 프롬프트를 루트 폴더에 `.md` 파일로 넣어주는 것과 같은 원리다.  
→ LLM이 맥락을 잃어버렸을 경우 다시 참조가 가능하기 때문에 맥락소실을 방지할 수 있다.

그 전까지는 세션 창을 여러 개 켜놓고 *"이 답변이 맞는지 저 답변이 맞는지"* 내가 직접 오케스트레이터 역할을 하고 있었는데, **Harness의 Generator ↔ Evaluator 세션 분리가 이 수작업을 자동화**해준다.

### 최신 Anthropic V2는 다르게 가고 있다

2026년 3월에 나온 **Anthropic Harness V2**[^2]는 흐름이 다르다. 역할마다 세션은 분리하되, **각 에이전트가 스프린트를 나누지 않고 하나의 세션에서 연속 작업**을 진행하며, 컨텍스트가 길어지면 **SDK의 자동 컴팩션(compaction, 긴 컨텍스트를 요약으로 압축하는 기능)** 으로 축소시킨다.

V2 설계자들은 *"Opus 4.6이 4.5 대비 성능이 향상되어 이렇게 해도 된다"*는 입장이다. 하지만 **컨텍스트 60%만 채워도 헤매는 게 눈에 보이는 상황**에서, 컴팩션으로 요약해버리면 대략적인 맥락은 남아도 **정밀성은 떨어질 수밖에 없다.** ~~(이 또한 자기 선호 편향?!)~~

그래서 세션 문제와 비용 문제를 함께 고려해, 나는 **구버전 Harness(2025.11) 패턴**으로 THE FORGE를 구현하기로 결정했다.

[^1]: Anthropic Engineering. *"Effective harnesses for long-running agents"* (2025.11). 
[^2]: Anthropic Engineering. *"Harness design for long-running application development"* (2026.03).



---

## 3. 비용 문제: Harness 위에 얹은 Harness

하지만 문제는 비용이었다.

Anthropic 공식 페이지에서 Harness가 **API 비용으로 3시간 50분 동안 124달러를 지출**한 사례를 보고 눈을 의심했다.
 https://www.anthropic.com/engineering/harness-design-long-running-apps
> *"이걸 Claude SDK로 똑같이 구현하면, 나는 라면만 먹으며 생활해야 할 것이다."*

그래서 다른 방법을 찾았다. 내가 선택한 방식은 **Claude의 월정액 요금제를 활용하는 방식**이다. Anthropic은 제3자 도구(Zed, OpenCode, Cursor 등)의 월정액 구독 활용을 막고 있고, 심지어 자사 SDK에서도 막고 있다. **하지만 Claude Code CLI는 월정액 구독으로 이용이 가능하다.**

그래서 **`claude` CLI를 `subprocess`로 호출하는 방식**으로 Harness를 구성했다.

Python 스크립트가 Phase 순서만 제어하고, 실제 LLM 실행은 Claude Code의 완성된 Harness를 그대로 활용한다. **그야말로 Harness 위에 얹은 Harness라 할 수 있다.**

>**출처**
>Anthropic Engineering: "Effective harnesses for long-running agents" (2025.11) 
>Anthropic Engineering: "Harness design for long-running application development" (2026.03)



## 4. 원격 통제: 텔레그램에서 슬랙으로

### "적당히 그 방향으로"는 현업에서 쓸 수 없다

기존 Harness처럼 텍스트 몇 마디로 개발을 진행시키는 방식은 별로 선호하지 않았다. 
**정확히 내가 조준한 그 지점으로 가야지, 적당히 그 방향으로 가서는 현업에서 쓸 수가 없다.**

그래서 두 가지 장치를 넣었다.

1. **Planner 시작 시점에 내가 작성한 기획서를 제공**하는 경로
2. 각 공정마다 **Human-in-the-Loop(HITL, 자동화 중간중간 사람이 개입해 방향을 잡는 설계 패턴)** 지점

에이전트들끼리 주고받는 파일을 **Slack / Telegram으로 직접 확인**할 수 있게 했다.

![[Pasted image 20260420200852.png]]
`/resume` `/skip` `/exit` `/stop` `/revise` 로 진행 여부를 결정할 수 있다. 예를 들어 `/revise`를 누르면 원하는 방향성을 **자연어로 지시**할 수 있다.

초기엔 Generator를 대화형으로 구현했으나, 그러면 **휴대폰으로 의사결정할 수 있다는 장점이 사라지기 때문에** 비대화 반자동(의사결정 시점에만 개입)으로 바꾸게 되었다.


### Telegram에서 Slack으로 갈아탄 이유

처음엔 설정의 간편함에 Telegram으로 시작했다. BotFather(`@BotFather`, Telegram에서 봇을 발급해주는 공식 안내 봇)와 짧은 대화 한 번이면 봇 토큰이 발급되고, 환경 변수 두 개(`FORGE_TELEGRAM_BOT_TOKEN`, `FORGE_TELEGRAM_CHAT_ID`)만 넣으면 바로 동작한다.

그런데 문득 **하네스 여러 대를 동시에 돌려야 할 경우**가 염려됐다.

- Telegram은 **하나의 봇 토큰에 하나의 프로세스만** 붙일 수 있다.
- 하네스 2대면 봇 토큰 2개 + 채팅방 2개, 3대면 방 3개가 필요하다.
- 한 채팅방에 라우터를 두는 방법도 있지만, 프로젝트를 추가할 때마다 라우터 설정을 갱신해야 한다.

**그래서 Slack으로 전환했다.** 
Slack은 **같은 앱의 Socket Mode 연결을 동시에 최대 10개까지 허용**한다. 
이 10개 연결로 들어오는 이벤트를 Slack이 자동으로 **로드 밸런싱(load balancing, 부하를 여러 연결에 나눠 분배)**해준다. 

결과적으로 **한 Slack 앱 + 한 채널 + 한 세트의 토큰으로 최대 10대의 하네스를 동시에** 붙일 수 있다.


### 잘못 배달된 버튼 이벤트 문제

하네스들이 같은 채널을 공유하다 보니, **A 프로젝트 버튼을 누른 이벤트가 B 프로젝트 하네스로 배달될 수 있다.**

이를 해결하기 위해 알림 버튼에 **`프로젝트이름::동작이름` 형식의 숨겨진 값**을 심어뒀다. 버튼이 눌리면 `::` 기준으로 쪼개서, 앞쪽이 **지금 이 프로세스가 담당하는 프로젝트 이름과 일치할 때만** 반응하고 아니면 무시한다.

**무시된 이벤트는 Slack이 자동으로 다른 연결에 재전달**하므로, 결국 올바른 하네스가 이벤트를 받게 된다.


### 통신 프로토콜: 왜 WebSocket과 HTTP를 섞어 쓰는가

> **WebSocket**: Slack → (THE FORGE) = **실시간 푸시**(이벤트 수신) 
> **HTTP**: (THE FORGE) → Slack = **작업 시키기**(메시지 올리기, 파일 업로드, 모달 열기)

둘 다 Socket Mode를 쓸 수 없는 이유는 **Slack이 Socket Mode를 수신 전용으로 설계**했기 때문이다.

버튼 클릭이나 슬래시 커맨드를 실시간으로 받으려면 Slack이 제공하는 두 가지 이벤트 수신 방식 중 하나를 골라야 한다.

| 방식 | 공인 IP·HTTPS 엔드포인트 | 방화벽 뒤 동작 | 동시 연결 수 |
|---|---|---|---|
| Events API (HTTP 웹훅) | 필요 | 곤란 | 제한 없음, 단 서버마다 공인 주소 필요 |
| Socket Mode (웹소켓) | 불필요 | 가능 | **앱당 최대 10개** |

**하네스를 여러 대 돌리기 위해 Socket Mode**를 택했다. 
개인 노트북·집 Wi-Fi 환경에서도 인프라 준비 없이 바로 붙는 점, 한 번에 10개 연결을 허용해 멀티 프로젝트 운영이 가능한 점이 결정 요인이었다.

```
[사용자(Slack 앱에서 파일 드래그)]
        │ HTTP multipart (Slack Web API `files.upload_v2`)
        ↓
   [Slack 서버] ← 파일 저장 완료
        │ Socket Mode 웹소켓으로 "파일 올라왔어요" 이벤트 푸시
        ↓
   [THE FORGE 백그라운드 스레드]
        │ 파일 내용이 필요하면 다시 HTTP로 다운로드
        ↓
   artifacts/ 폴더에 저장
```

즉, 웹소켓은 **"파일이 올라왔다는 사실"만 통보**하고, 실제 파일 바이트는 **따로 HTTP로 다운로드**한다.

- **Slack 이벤트 수신에 Socket Mode를 쓰는 이유**: THE FORGE가 Slack에 먼저 접속해 연결을 유지하므로, 방화벽이 있어도 공인 IP·HTTPS 엔드포인트를 준비할 필요가 없다.
- **파일 바이트에 HTTP를 쓰는 이유**: HTTP는 대용량 바이너리 업로드·다운로드에 최적화되어 있고, 중단·재개·진행률 추적 도구가 성숙해 있다. 
  웹소켓으로 큰 파일을 쪼개 보내면 다른 실시간 이벤트가 밀린다. 
  
- **짧은 이벤트는 웹소켓, 무거운 바이트는 HTTP**로 역할을 분리한 것이다.


### 파일 시스템을 메시지 큐처럼 쓰다

THE FORGE는 **파일 시스템 자체를 메시지 큐처럼 활용**한다.

- `/resume` 수신 → `artifacts/.approval-signal` 빈 파일 생성
- `/skip` 수신 → `artifacts/.skip-signal` 생성
- `/stop` 수신 → `artifacts/.stop-signal` 생성
- `/eval` 수신 → `artifacts/.eval-signal` 생성
- `/revise` 수신(모달 Submit 후) → `artifacts/.revise-text`에 텍스트 저장

**오케스트레이터 메인 스레드는 이 파일들을 주기적으로 확인**하다가, 생겼으면 읽고 삭제한 뒤 다음 단계로 넘어간다.

**왜 파일 방식인가?**

- **크래시 복원력**: 메인 프로세스가 죽었다 살아나도 파일은 남아 있어 의사결정 내용을 복구할 수 있다. 메모리 변수로 주고받았다면 크래시 즉시 사라진다.
- **채널 독립성**: CLI·Telegram·Slack 세 채널 모두 **"같은 파일 약속"** 만 따르면 된다.

---

## 5. 안전장치

프로그램이 무한정 돌거나 고장 난 채 계속 실행되지 않도록 3가지 한계선을 뒀다.

- **전체 실행 시간**: 1440분(24시간) 초과 금지
- **연속 실패 제한**: Evaluator가 연속 3회 FAIL 시 사용자 승인 없이 진행 불가
- **총 반복 횟수**: 20회(`max_total_sprints = 20`)로 제한

---

## 6. Agent 간 주고받는 문서

![[Pasted image 20260421010934.png]]
### Planner Agent: 기획자

Planner는 기획자다. 세션을 **2개로 쪼개어** 운영한다.

- **Mode A/B**: 기획 생성(신규) 또는 리뷰(기존 spec 존재 시)
- **Mode C**: 현재 스프린트에서 할 작업 내용 정리

**왜 세션을 둘로 나눴나?**
1. Claude Code CLI(`claude -p`)는 **대기가 불가능**하다. 한 번 호출하면 결과를 받고 종료된다.
2. **기획이 잘못된 방향이면 현재 스프린트 정리 내용은 쓸모없다.** 한 세션에 다 넣으면 토큰만 더 쓰고 버리는 꼴. 차라리 기획을 다시 하게 하는 편이 낫다.
3. 어차피 **세션이 새로 열려도 주어져야 할 컨텍스트는 동일**하다.

**Mode D**: 사용자가 텍스트로 전달사항을 주는 경우에는 세션이 새로 열리고, 기존 작업물과 사용자 지시문을 함께 읽는다. 계획이 매번 바뀌지 않도록 `spec.md`는 한 번 생성되면 모든 에이전트가 수정 불가지만, **Mode D만은 예외적으로 수정을 허용**한다.

> 📎 [[#부록 Planner I/O 상세 스펙]]은 글 끝 부록 참조.

### Generator Agent: 편집자

Generator는 편집자다. **에이전트 중 유일하게 문서 이외의 작업(실제 소스 코드 작성)을 처리**한다.

- **세션 시작 시 읽음**: `progress-log.md` → `spec.md` → 관련 `specs/*.md` → `sprint-contract.md` → `qa-report.md`(FAIL 우선 수정) → `git log`
- **작업 중 생성·수정**: `src/`, `tests/`, `decisions/decision-NNN.md`(스펙 모호 시 의사결정 기록), `sprint-contract.md` 체크박스만 `[x]`로 (자체 수정 금지), 기능 단위 git commit
- **세션 종료 시 필수 생성**: `progress-log.md` 맨 위에 **append-top** 방식으로 이번 세션 기록 추가 (완료 작업, 진행률, 결정 이유, 미처리 이슈, 다음 세션 할 일)

### Evaluator Agent: 평가자

Evaluator는 평가자다. Sprint Contract의 각 항목이 실제로 구현됐는지 검증하고, **오직 `qa-report.md`만 생성한다.** 절대 코드를 수정하지 않는다.

- **읽음**: `spec.md` + `sprint-contract.md` + `specs/*.md` + `progress-log.md` + `src/`, `tests/` 실제 코드
- **생성**: `qa-report.md`에 종합 판정(PASS / FAIL), 점수(4개 기준 각 1–10점), 항목별 검증 결과, 이슈 등급(CRITICAL / HIGH / MEDIUM / LOW), **파일명:라인번호** 근거, 자동 검증 로그(pytest 결과 등)

---

## 7. 다른 프로젝트에서 활용하기

THE FORGE는 **프로젝트를 진행하는 "도구"**로 쓰도록 설계했다. 
`uv tool install .`을 터미널에 입력하면, uv가 파이썬 패키징 표준(Entry Points + Hatch force-include)으로 실행 파일과 템플릿을 **시스템 전역 위치에 한 번 심어둔다.** 
이후 어느 폴더에서든 아래 두 명령어만으로 바로 사용할 수 있다.

### 1단계: `forge init`으로 초기 구조 생성

프로젝트 폴더에서 `forge init`을 실행하면 다음이 생성·병합된다.

- **`.claude/agents/`** : 4개 에이전트 지시문(`planner.md`, `generator.md`, `evaluator.md`, `journal.md`). Claude Code CLI가 `--agent` 옵션으로 이 지시문을 읽어 각 역할을 수행한다.
- **`templates/`** : Planner가 `spec.md`(대략 기획서)와 `specs/*.md`(도메인 상세 기획서)를 작성할 때 참고할 골격 템플릿 + LangGraph·DB 셋업·Langfuse 같은 도메인 가이드.
- **`CLAUDE.md`** :  프로젝트 루트 지시문. 공통 커밋 규칙과 세션 시작 절차.
- **`artifacts/`** : 이후 `forge run` 실행 시점에 자동 생성되며, 스프린트를 진행하며 나오는 모든 중간·최종 산출물(`spec.md`, `sprint-contract.md`, `qa-report.md`, `progress-log.md`, 체크포인트, 백업 등)이 쌓인다.

### 2단계: `forge run`으로 스프린트 루프 시작

세 가지 방식 중 편한 것으로 시작한다.

- `forge run "원하는 내용"` : 짧은 요청을 인자로 넘기면 Planner가 한 문장을 확장해 `spec.md` 작성
- `forge run --plan ./기획서.md` : 이미 작성해둔 마크다운 기획서를 입력으로 사용
- `forge run` (인자 없음) : 이전에 중단된 체크포인트가 있으면 거기서부터 자동 재개

실행이 시작되면 Planner → Generator → Evaluator가 `artifacts/` 파일을 통해 순차 협업하고, 각 단계와 QA 결과는 **Slack/Telegram으로 실시간 알림**이 온다. 원격에서 승인·수정·중단 명령을 내릴 수 있다.

---

## 8. 평가와 추적

>If you can't measure it, you can't manage it -Peter Drucker-
>*"측정할 수 없으면 관리할 수 없다"*
### 토큰 추출: 2중 전략

- **1순위: Claude Code 세션 로그 파싱** 
	- Claude Code CLI는 `~/.claude/projects/<프로젝트ID>/*.jsonl`(JSON Lines, 한 줄에 JSON 객체 하나씩 기록하는 로그 포맷)에 모든 assistant 응답과 토큰 사용량(`input_tokens`, `output_tokens`, `cache_read_input_tokens`, `cache_creation_input_tokens`, 모델명)을 남긴다. 
	- SprintTracer가 span 시작~종료 시각 범위 안에 찍힌 레코드만 긁어 합산한다.
- **2순위: stdout 파싱 fallback** 
  JSONL이 없거나 비어 있으면 subprocess가 출력한 `Tokens: input X / output Y` 라인을 정규식으로 추출한다.

기본적으로 `harness-cost-log.txt`라는 이름으로 완료 시간, 에이전트명, 작업 시간, input·output 토큰을 로컬에 저장한다.

```
[2026-04-19T08:23:47] sprint-4 contract     |   104.8s | in        26 / out   7,932 | claude-p    | OK
[2026-04-19T08:38:20] sprint-4 generator    |   872.7s | in    10,963 / out  52,326 | claude-p    | OK
[2026-04-19T08:42:46] sprint-4 evaluator    |   263.5s | in    33,029 / out  21,471 | claude-p    | OK
[2026-04-19T08:57:12] sprint-5 journal      |   139.1s | in    19,946 / out   6,104 | claude-p    | OK
```

**Langfuse**(LLM 관측성 플랫폼)를 통해서는 input 프롬프트와 output stdout의 **실제 텍스트 내용까지** 원격 대시보드에 자동 전송된다. 
`sprint-N` 트레이스 아래에 각 에이전트 span이 **계층 구조**로 쌓인다.

### 자동 평가

평가는 `evaluator` 에이전트가 자동으로 수행한다. 
매 스프린트가 끝나면 Evaluator가 `sprint-contract.md`의 체크박스 목록과 실제 커밋된 코드·테스트 결과를 대조해 **PASS / FAIL 판정**과 구체적 실패 항목을 `qa-report.md`에 기록한다. Orchestrator는 그 결과에 따라 **다음 스프린트 진입 · 재구현 · 재평가 · 중단** 중 하나를 자동 선택하거나 사용자에게 원격 승인을 요청한다.

---

## 9. 체크포인트: 언제 끊어져도 이어간다

정전, 재부팅, 토큰 한도 초과, 실수, 네트워크 끊김. 중단 요인은 수도 없이 많다. 그래서 **어디서 끊어지더라도 그 지점부터 이어 시작**할 수 있도록 진행 상태를 파일 하나에 기록한다.

**저장 파일**: `artifacts/.harness-checkpoint` (JSON 형식)

```json
{
  "phase": 8,
  "phase_name": "EVALUATING_DONE",
  "detail": "qa passed",
  "timestamp": "2026-04-19T08:42:47.347666"
}
```

필드는 네 개뿐이다. **단계 번호, 단계 이름, 짧은 메모, 저장 시각.** 단계는 총 9개다.

```
0 NONE              → 시작 전
1 PLANNING          → Planner 작업 중
2 PLANNING_DONE     → spec.md 완료
3 CONTRACT          → Sprint Contract 작성 중
4 CONTRACT_DONE     → 계약서 승인 완료
5 GENERATING        → Generator 코딩 중
6 GENERATING_DONE   → 코딩 완료
7 EVALUATING        → Evaluator QA 중
8 EVALUATING_DONE   → QA 완료
```


**단계를 숫자로 둔 이유** 
재개할 때 *"지금 단계가 이미 끝난 지점보다 앞인가 뒤인가"*만 비교하면 되므로, **숫자 크기 비교 한 번으로 판정이 끝난다.**

**언제 저장하나?** 
각 단계에 **들어가기 직전과 끝난 직후** 각각 한 번씩 파일을 덮어쓴다. 그래서 어느 순간 크래시가 나도 **최악의 경우 지금 하던 단계만 처음부터 다시 하면 된다.** 
이미 끝난 단계를 다시 돌리는 낭비는 없다.

**재개 방법 3가지**

| 상황 | 방법 |
|---|---|
| 평소 (자동) | 같은 폴더에서 `forge run` 다시 실행. 파일 읽고 알아서 이어감 |
| 특정 단계부터 다시 | `forge run --from generating` 처럼 단계 이름으로 지정 |
| 수동 개입 | `.harness-checkpoint` 파일을 메모장으로 열어 `"phase"` 숫자만 변경 |

---


## 10. 토큰 먹는 중전차 

### 사례 1: Google Drive ↔ 로컬 동기화 프로젝트

Version Vector(분산 시스템에서 각 노드의 업데이트 순서를 벡터로 기록해 충돌을 감지하는 기법) 기반 동기화 프로젝트였다. 

**5일 분량의 작업이 3시간 만에 Core 구축까지 끝났다.**

세션 수는 **Planner → Generator → Evaluator** 한 세트가 1번이고, 세션별 기초 설계서 `sprint-contract.md`가 이미 있다면 **Generator → Evaluator**가 한 세션으로 진행된다.

| Sprint | 세션 수 | 누적 Latency | Input | Output | Σ Tokens | Total Cost |
|---|---|---|---|---|---|---|
| **sprint-1** | 8 | 1h 37m 42s | 32,694 | 273,482 | **31,952,167** | **$7.00052** |
| **sprint-2** | 1 | 24m 30s | 7,349 | 95,578 | **25,121,315** | **$2.426195** |
| **sprint-3** | 2 | 33m 02s | 32,553 | 134,601 | **15,593,319** | **$3.52779** |
| **sprint-4** | 1 | 20m 46s | 44,018 | 81,729 | **18,996,343** | **$2.263315** |
| **sprint-5** | 1 | 2m 19s | 19,946 | 6,104 | **312,670** | **$0.25233** |
| **합계** | **13** | **≈ 2h 58m 19s** | **136,560** | **591,494** | **91,975,814** | **$15.47015** |

### 사례 2: 옵시디언 61개 md 파일 정리

옵시디언 볼트에서 한 폴더 아래 md 파일들이 지저분하게 쌓여 있어서, 시험 삼아 **내용 보강 + 파일 트리 형태 정리**를 시켰다.

그 결과 허머(Hummer, 연비 나쁘기로 악명 높은 대형 SUV)가 *"기름 게이지가 실시간으로 내려가는 게 눈에 보인다"*는 소리처럼, 
**5시간 할당량 토큰이 쭉쭉 내려가는 게 실시간으로 보였다.** 
그래서 20%쯤 감소했을 때 중지했다. (나는 Claude Max $200 요금제를 쓰고 있다.) 
**앞으로 너무 광범위한 작업은 시키지 않는 게 좋겠다는 교훈**을 얻었다.

**더 큰 문제는 이 작업을 세션 1의 Planner가 진행했다는 사실이다.** 
Planner는 원래 작업을 하면 안 된다. 
설계만 하고 `spec.md`, `specs/*.md`, `plan-review.md`, `sprint-contract.md` 같은 설계 문서만 작성하는 역할이다. 
**할루시네이션(hallucination, LLM이 지시 범위를 벗어나 자기 멋대로 행동하는 현상)이 발생한 것이다.** 
그래서 이 운행을 계기로 **금지 조항을 추가**했다.

![[Pasted image 20260421190448.png]]

*"그냥 툴에서 Write와 Edit 기능을 빼버리면 되지 않나?"*라고 생각할 수 있다. 
하지만 적용해본 결과 Write·Edit 권한이 없으면 Planner가 설계 문서 자체를 작성할 수 없게 되므로 `spec.md`도 생성도 불가능하다.

그래서 **도구 제한이 아니라 프롬프트 단에서 "어떤 범위까지만 써라"**라고 제약을 걸어야 했다.

---

## 11. 그래서 개발자가 할 일은 무엇인가

혹자는 이렇게 물을 수 있다. *"그럼 개발자가 할 일이 무엇인가?"*

**이것 한번 돌린다고 개발이 다 끝나는 게 아니다.** 
중전차가 한번 쓸고 간 뒤, **잔당 처리(Dogfooding, 자신이 만든 제품을 직접 써보며 문제를 발견하는 것)는 개발자가 직접 해야 한다.** 
아무리 기획을 잘 했어도, 평가 점수가 아무리 높게 나왔어도, **실제로 적용하면 반드시 에로사항이 나온다.** 이 시간이 Harness를 돌리는 시간보다 오래 걸린다.

그리고 Mitchell Hashimoto는 이렇게 말했다.

> *"에이전트가 실수할 때마다, 그 에이전트가 다시는 같은 실수를 하지 않도록 솔루션을 엔지니어링하는 데 시간을 투자하라."*

구체적으로 이렇게 대응한다.

1. **Generator 반복 실수** → `CLAUDE.md` 규칙 추가
2. **Evaluator 누락 패턴** → `AGENT.md` 기준 추가
3. **Planner 빠뜨림** → `AGENT.md` 체크리스트 추가
4. **forge 자체의 오류** → Python 코드 수정 → `uv tool install --force .`로 즉시 반영

**진행하는 프로젝트에 맞는 방향으로의 수정이 필요하다.**

또한 같은 시행착오를 두 번 겪지 않도록, 프로젝트를 진행하면서 발생하는 에러들을 `templates/`에 추가한다. **이것 하나하나가 개발자의 자산이 된다.**

> *하지만 이것도 양이 많아지면 결국 context window를 차지한다. 현재 해결하고 싶은 문제와 같은 부분의 힌트만 얻으려면, 결국 **MEMEX가 필요하다.***

그래서 다음 글은 MEMEX다.


---

## 부록: Planner I/O 상세 스펙

| 구분 | Mode A (생성) | Mode B (리뷰) | Mode C (Sprint Contract) |
| --- | --- | --- | --- |
| **트리거** | `spec.md` 없음 + 사용자 요청 | `spec.md` 존재 (`--plan` 포함) | `sprint-contract.md` 없음 (매 스프린트 시작) |
| **호출 빈도** | 프로젝트당 1회 | 프로젝트당 0~1회 (A 경로면 호출 안 됨) | 프로젝트당 N회 (스프린트 수만큼) |
| **읽음** | • 사용자 요청 <br>• `templates/INDEX.md` <br>• 선정된 `templates/*.md` | • `artifacts/spec.md` <br>• `artifacts/specs/*.md` (있으면) <br>• `templates/INDEX.md` <br>• 선정된 `templates/*.md` | • `artifacts/spec.md` <br>• `artifacts/specs/*.md` <br>• `artifacts/progress-log.md` (2번째~) <br>• `artifacts/sprint-*-done.md` (이전 완료분) <br>• `templates/sprint-contract-template.md` <br>• `templates/INDEX.md` (specs/ 비었을 때만) |
| **생성** | • `artifacts/spec.md` <br>• `artifacts/specs/*.md` | • `artifacts/plan-review.md` (READY / NEEDS_REVISION) <br>• `artifacts/specs/*.md` (specs/ 비었을 때) | • `artifacts/sprint-contract.md` (frontmatter + 본문) <br>• `artifacts/specs/*.md` (specs/ 비었을 때 예외) |
| **Fallback 규칙** | 템플릿 없거나 매칭 안 되면 `spec.md` 내용만으로 specs 생성 | A와 동일 | A와 동일 (specs/ 비어있을 때만 예외 적용) |