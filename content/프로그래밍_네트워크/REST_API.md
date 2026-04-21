# REST API 구조

> Agent가 도구로 호출하는 외부 서비스 대부분이 REST API 형태로 열려 있다. "왜 POST인데 응답이 204야?" "PUT이랑 PATCH 차이가 뭔데?" 같은 질문이 매일 나와서, 설계 철학과 관례를 한번에 정리한다.

관련 문서: [[HTTP_HTTPS]], [[WebSocket]], [[포트]], [[프록시]]

---

## 1. REST가 뭔데

**REST(Representational State Transfer, 표현 상태 전이, 자원의 "현재 상태를 표현"한 데이터를 HTTP로 주고받는 아키텍처 스타일)** 는 엄밀히 말하면 프로토콜이 아니라 **설계 스타일**이다. 2000년 Roy Fielding의 박사 학위 논문에서 제안되었고, HTTP의 표준 기능을 최대한 그대로 쓰자는 철학을 담고 있다.

포함 관계로 보면:

```
네트워크 통신
 └─ TCP/IP
     └─ HTTP  ← 메시지 규격 (시작줄/헤더/본문)
         └─ REST API  ← "HTTP를 이렇게 쓰자"는 약속
             └─ 실제 서비스 API (OpenAI, GitHub, Stripe, ...)
```

즉 REST는 [[HTTP_HTTPS]]의 한 갈래 사용법이다.

---

## 2. 핵심 개념 4가지

### 2-1. 자원(Resource)

모든 것은 "자원"이다. 사용자, 주문, 파일, LLM 세션 모두 자원.

- 자원 하나 → `/users/42`
- 자원 집합 → `/users`
- 자원의 하위 자원 → `/users/42/orders/7`

### 2-2. URI로 자원 식별

**URI(Uniform Resource Identifier, 통합 자원 식별자, 자원 하나를 고유하게 가리키는 문자열)** 는 명사로 쓴다. 동사는 메서드가 맡는다.

| 나쁜 예 | 좋은 예 |
|---------|---------|
| `GET /getUser?id=42` | `GET /users/42` |
| `POST /createOrder` | `POST /orders` |
| `POST /user/42/delete` | `DELETE /users/42` |

### 2-3. 메서드(Method)로 행위 표현

| 메서드 | CRUD | 멱등성(Idempotent) | 안전성(Safe) |
|--------|------|--------------------|-------------|
| GET | Read | O | O |
| POST | Create | X | X |
| PUT | Update(전체) | O | X |
| PATCH | Update(부분) | X (관례상) | X |
| DELETE | Delete | O | X |

- **멱등성(Idempotent, 같은 요청을 여러 번 보내도 결과 상태가 같음)**: GET, PUT, DELETE. 네트워크 불안정하면 재시도해도 안전
- **안전성(Safe, 서버 상태를 바꾸지 않음)**: GET, HEAD, OPTIONS. 캐시 가능

### 2-4. 표현(Representation)

같은 자원을 JSON, XML, HTML로 다르게 표현할 수 있다. `Accept` 헤더로 클라이언트가 원하는 형식을 알려주고, 서버는 `Content-Type`으로 실제 형식을 붙여 응답.

요즘은 거의 `application/json`.

---

## 3. 요청/응답 예시

### 3-1. GET (조회)

```http
GET /v1/messages/msg_01abc HTTP/1.1
Host: api.anthropic.com
Accept: application/json
x-api-key: sk-ant-...
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "msg_01abc",
  "model": "claude-opus-4-7",
  "content": [{"type": "text", "text": "..."}]
}
```

### 3-2. POST (생성)

```http
POST /v1/messages HTTP/1.1
Host: api.anthropic.com
Content-Type: application/json
x-api-key: sk-ant-...

{
  "model": "claude-opus-4-7",
  "messages": [{"role": "user", "content": "hi"}]
}
```

```http
HTTP/1.1 201 Created
Location: /v1/messages/msg_01new
Content-Type: application/json

{"id": "msg_01new", ...}
```

생성 성공 시 상태 코드는 `201 Created`, 새 자원의 위치는 `Location` 헤더에 담는 게 관례.

---

## 4. 상태 코드 의미 맞추기

[[HTTP_HTTPS]] 문서에 전체 범위를 정리했고, 여기선 REST 설계 시 고민되는 것만.

| 상황 | 추천 코드 |
|------|----------|
| 조회 성공 | 200 OK |
| 생성 성공 | 201 Created |
| 요청은 성공, 응답 본문 없음 | 204 No Content (DELETE 후 자주) |
| 비동기 작업 접수 | 202 Accepted |
| 요청 스키마 틀림 | 400 Bad Request |
| 인증 토큰 없음/틀림 | 401 Unauthorized |
| 인증은 됐지만 권한 부족 | 403 Forbidden |
| 자원 없음 | 404 Not Found |
| 허용 안 된 메서드 | 405 Method Not Allowed |
| 리소스 충돌(버전 꼬임 등) | 409 Conflict |
| 요청 본문은 맞지만 의미상 처리 불가 | 422 Unprocessable Entity |
| 레이트 리밋 | 429 Too Many Requests |

---

## 5. 쿼리 파라미터 관례

자원 자체는 경로(path)로, 필터/페이지/정렬은 쿼리 스트링으로.

```
GET /v1/messages?model=claude-opus-4-7&limit=20&after=msg_01abc&order=desc
```

- **filter**: `status=active`, `role=user`
- **pagination**: `limit`, `offset` 또는 커서 방식의 `after` / `before`
- **sort**: `order=desc`, `sort=-created_at` (마이너스는 내림차순 관례)
- **projection**: `fields=id,name` (응답에 포함할 필드만 선택)

---

## 6. 버저닝(Versioning)

API 스키마는 언젠가 바뀐다. 클라이언트를 깨뜨리지 않으려면 버전을 명시해야 함.

| 방식 | 예시 | 장단점 |
|------|------|--------|
| URI 경로 | `/v1/users`, `/v2/users` | 가장 직관적, 많이 쓰임 |
| 헤더 | `Accept: application/vnd.myapi.v2+json` | URL 깔끔, 캐시/디버깅 불편 |
| 쿼리 | `/users?version=2` | 캐시 꼬임 |

대부분의 LLM 제공자는 경로 버저닝: `api.openai.com/v1/...`, `api.anthropic.com/v1/...`.

---

## 7. 인증(Authentication)

| 방식 | 설명 | 어디서 쓰나 |
|------|------|------------|
| API Key | 고정 문자열 키를 헤더에 삽입 | LLM API, 내부 서비스 간 호출 |
| Bearer Token | `Authorization: Bearer <JWT>` | OAuth2, JWT 기반 서비스 |
| OAuth 2.0 | 사용자 위임 인증 흐름 | GitHub, Google API |
| mTLS | 양방향 TLS 인증서 교환 | 금융권, 내부 서비스 메시 |

JWT(JSON Web Token, 서명된 JSON 형식 토큰)는 `header.payload.signature` 세 조각이 점으로 이어진 문자열. 디버깅 시 jwt.io에 붙이면 내용이 풀린다. (단, 운영 토큰은 절대 외부 사이트에 붙여 넣지 말 것)

---

## 8. RESTful한 설계 체크리스트

1. URI는 **명사**로. 동사는 HTTP 메서드에 맡긴다
2. **복수형 리소스 이름**: `/users`, `/orders` (관례)
3. **계층 관계는 경로로**: `/users/42/orders`
4. **필터는 쿼리로**, 자원 식별은 경로로
5. **적절한 상태 코드** (200 도배 금지)
6. **멱등성 보장**: PUT/DELETE는 재시도 안전하게
7. **일관된 에러 포맷**: `{"error": {"code": "...", "message": "..."}}`
8. **Pagination 지원**: 수천 건을 한 번에 주지 말 것
9. **HATEOAS(선택)**: 응답에 다음에 할 수 있는 링크 포함. 실제론 거의 안 씀

---

## 9. REST 아닌 동료들과 비교

| 스타일 | 특징 | 언제 쓰나 |
|--------|------|----------|
| **REST** | 자원 중심, HTTP 표준 활용 | 범용 CRUD, 외부 공개 API |
| **GraphQL** | 쿼리 언어로 원하는 필드만 조회 | 모바일처럼 대역폭 제한, 다양한 화면 |
| **gRPC** | Protocol Buffers + HTTP/2, 바이너리 RPC | 내부 마이크로서비스 고성능 통신 |
| **WebSocket** | 양방향 실시간 스트림 | 채팅, 라이브 업데이트 ([[WebSocket]]) |
| **JSON-RPC** | 메서드 호출 중심 JSON 프로토콜 | MCP(Model Context Protocol) 등 |

실제로 MCP, Ethereum JSON-RPC 등은 REST가 아닌 RPC 스타일이다. "REST가 항상 정답"은 아님.

---

## 10. 직접 확인

```bash
# GitHub API로 자원 조회 체험
curl -i https://api.github.com/users/torvalds

# 헤더만
curl -I https://api.github.com/users/torvalds

# 권한 필요한 엔드포인트
curl -H "Authorization: Bearer $GITHUB_TOKEN" https://api.github.com/user
```

응답 본문을 보면 `login`, `id`, `repos_url` 같은 필드가 JSON으로 나온다. `repos_url`을 다시 GET하면 그 사용자의 레포 목록이 나오는 자원 연결 구조가 REST답다.

---

## 한마디 요약

REST는 "URI는 명사로 자원을 가리키고, HTTP 메서드로 동사를 표현한다"는 약속이고, 나머지는 상태 코드와 에러 포맷 같은 관례의 묶음이다.
