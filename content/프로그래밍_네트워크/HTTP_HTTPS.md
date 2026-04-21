# HTTP / HTTPS 동작 원리

> AI Agent가 외부 API(OpenAI, Anthropic, Tavily, 내부 RAG 서버 등)와 대화할 때 대부분 HTTP/HTTPS 위에서 주고받는다. 여기서 한 줄이라도 삐끗하면 "왜 401만 돌아오지?"가 하루를 잡아먹는다. 그래서 요청-응답의 뼈대부터 TLS 악수까지 한 번에 훑어두자.

관련 문서: [[REST_API]], [[포트]], [[프록시]], [[DNS]], [[WebSocket]]

---

## 1. HTTP라는 약속

**HTTP(HyperText Transfer Protocol, 하이퍼텍스트 전송 규약, 클라이언트와 서버가 문자열 메시지를 주고받기 위한 표준 형식)** 는 한마디로 "이런 형식으로 말 걸면 이런 형식으로 답할게"라는 합의다.

- **전송 계층**: 기본적으로 TCP(Transmission Control Protocol, 전송 제어 규약, 패킷이 순서대로 빠짐없이 도착하도록 보장)  위에서 동작. HTTP/3부터는 QUIC(UDP 기반)로 바뀜
- **기본 포트**: HTTP는 80, HTTPS는 443 ([[포트]] 문서 참고)
- **stateless(무상태)**: 서버가 이전 요청을 기억하지 않음. 로그인 상태 유지는 쿠키나 토큰으로 별도로 전달

**비유:** 택배 송장 양식과 같다. 송장에 "받는 사람, 주소, 물품"을 정해진 칸에 쓰기로 약속했기 때문에, 어떤 택배사든 그 양식만 지키면 물건이 간다.

---

## 2. 요청(Request) 구조

HTTP 요청은 딱 세 덩어리로 나뉜다.

```http
POST /v1/messages HTTP/1.1               ← 1) 시작줄 (start-line)
Host: api.anthropic.com                  ← 2) 헤더 (headers)
Content-Type: application/json
x-api-key: sk-ant-...
Content-Length: 87
                                         ← 빈 줄
{"model":"claude-opus-4-7","messages":[...]}   ← 3) 본문 (body)
```

### 2-1. 메서드(Method)

| 메서드 | 뜻 | 본문 유무 | Agent에서 주로 쓰는 곳 |
|--------|-----|-----------|----------------------|
| GET | 조회 | 없음 | 벡터 DB 검색, 뉴스 크롤링 |
| POST | 생성/전송 | 있음 | LLM API 호출, 메시지 전송 |
| PUT | 전체 교체 | 있음 | 파일 전체 업로드 |
| PATCH | 부분 수정 | 있음 | 세션 상태 일부 갱신 |
| DELETE | 삭제 | 선택 | 임시 리소스 정리 |
| HEAD | 헤더만 조회 | 없음 | 파일 존재/크기 확인 |
| OPTIONS | 허용 메서드 질의 | 없음 | CORS preflight |

자세한 메서드 설계 원칙은 [[REST_API]] 참고.

### 2-2. 주요 헤더

- **Host**: 목적지 도메인. 한 서버에 여러 도메인이 올라가 있을 수 있어서 필수
- **Content-Type**: 본문 형식. `application/json`, `multipart/form-data`, `text/event-stream` 등
- **Authorization / x-api-key**: 인증 토큰. LLM API는 보통 `Bearer sk-...` 또는 전용 헤더
- **Accept**: 내가 받을 수 있는 형식. `text/event-stream`이면 서버가 SSE 스트리밍으로 응답
- **User-Agent**: 요청 보낸 클라이언트 식별
- **Cookie**: 세션 상태 전달

---

## 3. 응답(Response) 구조

```http
HTTP/1.1 200 OK                          ← 1) 상태줄 (status-line)
Content-Type: application/json           ← 2) 헤더
anthropic-ratelimit-requests-remaining: 49
Content-Length: 512

{"id":"msg_01...","content":[...]}       ← 3) 본문
```

### 3-1. 상태 코드(Status Code) 한눈에

| 범위 | 의미 | 대표 코드 | Agent 디버깅 시 보게 되는 상황 |
|------|------|-----------|------------------------------|
| 1xx | 정보성 | 100 Continue | 거의 안 만남 |
| 2xx | 성공 | 200 OK, 201 Created, 204 No Content | 정상 |
| 3xx | 리다이렉트 | 301 영구 이동, 302 임시, 304 캐시 적중 | 크롤링할 때 Location 헤더 따라가야 함 |
| 4xx | 클라이언트 잘못 | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests | API 키 오타, JSON 스키마 틀림, 레이트 리밋 |
| 5xx | 서버 잘못 | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout | 지수 백오프(Exponential Backoff, 재시도 간격을 점점 늘리는 전략)로 재시도 |

**직접 확인:**

```bash
curl -i https://api.anthropic.com/v1/messages
# -i 옵션: 응답 헤더까지 출력
```

첫 줄에 `HTTP/2 401` 이런 식으로 상태 코드가 찍힌다.

---

## 4. HTTPS : HTTP + TLS

**HTTPS(HTTP Secure, 보안 HTTP, 암호화된 터널 위에서 HTTP를 주고받는 방식)** 는 HTTP 자체를 바꾼 게 아니라, 그 밑에 **TLS(Transport Layer Security, 전송 계층 보안, 데이터를 암호화하고 상대가 진짜인지 인증하는 프로토콜)** 를 끼운 것이다.

### 4-1. TLS가 해주는 세 가지

1. **기밀성(Confidentiality)**: 중간에서 엿봐도 암호문만 보임
2. **무결성(Integrity)**: 중간에서 바꾸면 검증 실패
3. **인증(Authentication)**: 접속한 서버가 진짜 그 도메인의 주인인지 인증서로 확인

### 4-2. TLS Handshake 단계 분해

1단계 → **ClientHello**: 클라이언트가 "난 이런 암호 알고리즘들 쓸 수 있어" 목록 전송
2단계 → **ServerHello + Certificate**: 서버가 암호 알고리즘 하나 선택 + 자기 인증서(Certificate, 공개키와 도메인 소유권을 CA가 보증한 문서) 전송
3단계 → **인증서 검증**: 클라이언트가 인증서 발급자(CA, Certificate Authority, 인증 기관)를 OS나 브라우저의 루트 저장소와 대조
4단계 → **키 교환**: ECDHE(타원곡선 디피-헬먼) 같은 알고리즘으로 대칭키 공유. 서로 같은 비밀키를 만들지만 네트워크에는 흘리지 않음
5단계 → **Finished**: 이후 통신은 대칭키로 암호화된 애플리케이션 데이터

TLS 1.3부터는 1-RTT(왕복 1회)로 줄었고, 0-RTT 재접속도 가능.

**비유:** 낯선 사람과 비밀 이야기하러 만났을 때, 서로 신분증 확인(인증서) → 종이에 쓸 암호표 교환(키 교환) → 이후엔 암호표대로 쓴 쪽지(암호문)만 주고받는 흐름과 같다.

### 4-3. 직접 확인

```bash
# 인증서 체인 보기
openssl s_client -connect api.openai.com:443 -showcerts

# 어떤 TLS 버전, 어떤 암호 사용 중인지
curl -v https://api.openai.com/ 2>&1 | grep -E "TLS|SSL|cipher"
```

---

## 5. HTTP 버전별 차이

| 버전 | 연결 모델 | 멀티플렉싱 | 헤더 압축 | 전송 계층 |
|------|-----------|-----------|----------|----------|
| HTTP/1.1 | 요청당 TCP 1개(또는 keep-alive 재사용) | 없음, 파이프라이닝은 사실상 사장 | 없음 | TCP |
| HTTP/2 | 한 TCP 위에 여러 스트림 병렬 | 있음 | HPACK | TCP + TLS |
| HTTP/3 | QUIC 위에 스트림, 패킷 손실이 한 스트림만 막음 | 있음 | QPACK | UDP(QUIC) |

LLM 스트리밍 응답은 보통 HTTP/1.1 또는 HTTP/2 위의 **SSE(Server-Sent Events, 서버 보내기 이벤트, 서버가 단방향으로 청크를 연속 push하는 방식)** 로 온다. 실시간 양방향이 필요하면 [[WebSocket]]으로 넘어가야 함.

---

## 6. Agent 개발에서 자주 걸리는 지점

- **401/403**: API 키 오타, 키 만료, 권한 스코프 부족. Authorization 헤더 실제 값 로깅해서 확인
- **429**: 레이트 리밋. 응답 헤더 `Retry-After`나 `x-ratelimit-reset` 보고 지수 백오프
- **502/504**: 업스트림(프록시 뒤 실제 서버) 문제. [[프록시]] 쪽 로그 확인
- **CORS 에러**: 브라우저 전용. Python/Node 서버 호출은 CORS 무관. OPTIONS preflight가 실패했는지 네트워크 탭 확인
- **타임아웃**: LLM 응답은 수십 초 걸릴 수 있음. `httpx.Timeout(connect=10, read=300)`처럼 read 타임아웃 넉넉히

---

## 7. 왜 이렇게 설계됐나

- **Stateless**: 서버가 상태 기억 안 하니 수평 확장이 쉬움. 로드밸런서 뒤에 서버 100대를 붙여도 세션 고정이 필요 없음
- **텍스트 기반**: 디버깅이 쉬움. `curl -v`로 사람이 읽을 수 있는 요청/응답을 그대로 확인
- **TLS는 옵션이 아니라 기본**: 2020년 이후 주요 브라우저는 HTTP를 "안전하지 않음"으로 표시. API 제공자도 HTTPS만 허용
- **HTTP/2,3의 멀티플렉싱**: HTTP/1.1은 "한 연결에 한 번에 한 요청"이라 여러 리소스 받을 때 대기열이 쌓임(HOL blocking, Head-Of-Line 차단). HTTP/2부터 한 연결에서 여러 스트림을 병렬 처리

---

## 한마디 요약

HTTP는 "시작줄 + 헤더 + 본문" 세 덩어리 택배 송장, HTTPS는 그 송장을 TLS라는 봉투에 넣어 암호화해 보내는 것이다.
