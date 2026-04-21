# WebSocket

> LLM 스트리밍은 대부분 SSE(Server-Sent Events)로 충분하지만, 사용자 입력과 LLM 출력이 동시에 왔다갔다해야 하는 음성 Agent, 실시간 협업 Agent, 브라우저 기반 채팅 UI에서는 WebSocket이 필요하다. HTTP와 뭐가 다르고 왜 따로 만들어졌는지 짚어두자.

관련 문서: [[HTTP_HTTPS]], [[REST_API]], [[포트]], [[프록시]]

---

## 1. 왜 필요한가

**HTTP(HyperText Transfer Protocol, 하이퍼텍스트 전송 규약)** 는 "클라이언트가 요청 → 서버가 응답"이라는 단방향 사이클이 기본이다. 서버가 먼저 "야, 새 알림 왔어"라고 말을 걸 수 없다.

과거 해결책:
- **Polling(폴링, 주기적으로 서버에 물어봄)**: 1초마다 GET 요청. 낭비 심함
- **Long Polling(긴 폴링)**: 서버가 데이터 생길 때까지 응답 미룸. 연결 낭비
- **SSE(Server-Sent Events, 서버 → 클라이언트 단방향 스트림)**: 서버 → 클라이언트 방향만 가능
- **WebSocket**: 양방향 실시간, 한 번 연결하면 계속 열려 있음

---

## 2. WebSocket 개념

**WebSocket(웹소켓, HTTP로 시작해서 TCP 연결을 그대로 양방향 메시지 채널로 전환하는 프로토콜, RFC 6455)** 은:

- **지속 연결(persistent connection)**: 한 번 연결하면 끊을 때까지 유지
- **양방향(full-duplex)**: 서버/클라이언트 누구든 먼저 말할 수 있음
- **프레임(frame) 단위**: HTTP처럼 텍스트 요청/응답이 아니라 작은 프레임을 주고받음
- **포트**: ws(80), wss(443). 기존 HTTP 포트 그대로 재사용 ([[포트]] 참고)
- **URI 스킴**: `ws://` 또는 `wss://`(TLS)

**비유:** HTTP가 편지라면 WebSocket은 전화 통화다. 편지는 매번 주소 쓰고 우표 붙여야 하지만, 전화는 한 번 연결되면 끊을 때까지 서로 자유롭게 말한다.

---

## 3. Handshake 단계 분해

WebSocket은 **HTTP 요청으로 시작해서 중간에 프로토콜을 바꾼다(Upgrade)**. 그래서 방화벽/프록시도 HTTP처럼 통과시킬 수 있다.

### 1단계 → 클라이언트 요청

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: https://example.com
```

핵심은 `Upgrade: websocket` 헤더. "이 TCP 연결을 HTTP 말고 WebSocket으로 써 줘"라는 요청.

### 2단계 → 서버 응답

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

`101 Switching Protocols` 상태 코드가 핵심. 이 순간 이후로 그 TCP 연결은 HTTP가 아니라 WebSocket 프레임 교환용이 된다.

### 3단계 → 메시지 교환

```
Client → Server: [프레임] "hello"
Server → Client: [프레임] "hi back"
Client → Server: [프레임] "still there?"
Server → Client: [프레임] "yes"
...
```

### 4단계 → 종료

한 쪽이 Close 프레임 전송 → 상대도 Close 프레임 전송 → TCP 연결 종료.

---

## 4. HTTP vs WebSocket 비교

| 항목 | HTTP/1.1 요청-응답 | SSE | WebSocket |
|------|-------------------|-----|-----------|
| 방향 | 클라 → 서버(단방향 왕복) | 서버 → 클라(단방향) | 양방향 |
| 연결 유지 | 요청마다 새로(keep-alive로 재사용은 가능) | 한 요청 동안 유지 | 명시적으로 닫을 때까지 |
| 메시지 구분 | Content-Length나 청크 | `\n\n` 구분 텍스트 | 바이너리/텍스트 프레임 |
| 오버헤드 | 요청마다 헤더 | 초기 1회 헤더 | 초기 핸드셰이크만 |
| 브라우저 지원 | 모두 | 대부분 | 모두 |
| 주요 용도 | REST API, 파일 다운로드 | LLM 스트리밍, 알림 | 채팅, 게임, 실시간 협업 |

---

## 5. 프레임(Frame) 구조

WebSocket 메시지는 프레임 하나 또는 여러 개로 나뉜다. 프레임 헤더는 2 ~ 14 바이트.

- **opcode**: 0x1(text), 0x2(binary), 0x8(close), 0x9(ping), 0xA(pong)
- **masking**: 클라이언트가 서버로 보낼 때는 4바이트 마스크로 페이로드를 XOR. 프록시 캐시 공격 방어 목적
- **payload length**: 7비트, 7+16비트, 7+64비트로 가변

이 덕에 HTTP 헤더처럼 매번 수백 바이트 붙이지 않고, 짧은 메시지 하나에 오버헤드가 6바이트 정도로 끝난다.

---

## 6. Ping / Pong 과 연결 유지

중간 장비(NAT, [[프록시]], 로드밸런서)는 "한동안 아무 패킷 없는 TCP 연결"을 끊어버린다. 그래서 WebSocket은 주기적으로:

- 서버 → 클라이언트로 **Ping 프레임** 전송
- 클라이언트는 **Pong 프레임**으로 응답

대략 30초 간격이 관례. 응답 없으면 죽은 연결로 판단하고 닫는다. 라이브러리들은 보통 자동 처리.

---

## 7. Agent 개발에서 쓰는 경우

1. **실시간 음성 Agent**: OpenAI Realtime API, Deepgram 같은 STT/TTS 스트림이 WebSocket. 오디오 청크를 프레임으로 계속 밀어넣고, 서버는 인식 결과를 실시간으로 쏜다
2. **브라우저 기반 채팅 UI**: 사용자 입력과 LLM 토큰 스트리밍을 한 채널에서 처리
3. **MCP(Model Context Protocol) 서버**: stdio 또는 WebSocket으로 Agent와 도구 서버가 양방향 JSON-RPC
4. **멀티에이전트 협업**: 여러 Agent가 서로 이벤트를 주고받는 메시지 버스

### Python 예시 (websockets 라이브러리)

```python
import asyncio, websockets

async def echo():
    async with websockets.connect("wss://echo.websocket.org") as ws:
        await ws.send("hello")
        print(await ws.recv())

asyncio.run(echo())
```

### JS 예시 (브라우저)

```javascript
const ws = new WebSocket("wss://example.com/chat");
ws.onopen = () => ws.send("hi");
ws.onmessage = (e) => console.log(e.data);
ws.onclose = () => console.log("bye");
```

---

## 8. 직접 확인

브라우저 개발자 도구 → Network → WS 탭에서 실제 프레임 내용을 볼 수 있다. 텍스트든 바이너리든 보낸/받은 순서대로 펼쳐진다.

CLI에서는:

```bash
# websocat으로 간단 테스트
websocat wss://echo.websocket.org
```

타이핑한 문자열이 그대로 에코되는 걸 확인할 수 있음.

---

## 9. 왜 HTTP 대신 쓰는가 (트레이드오프)

### 장점

- 진짜 실시간 양방향
- 한 번 연결된 후엔 오버헤드 작음
- 기존 HTTP(80/443) 포트 사용 → 방화벽 통과 쉬움

### 단점

- **상태 있음(stateful)**: 서버가 연결별 상태를 기억해야 해서 수평 확장 어려움. sticky session 필요
- **프록시/로드밸런서 설정 까다로움**: 긴 연결을 끊지 않도록 타임아웃 늘려야 함 ([[프록시]] 참고)
- **재연결 로직 필요**: 모바일 네트워크는 자주 끊김. exponential backoff로 다시 붙이는 코드 필수
- **인증이 HTTP보다 까다로움**: 핸드셰이크 이후엔 헤더가 없어서 첫 프레임에 토큰 넣거나 쿼리스트링으로 토큰 전달

### 언제 쓰면 안 되나

- 단순 요청-응답: [[REST_API]]로 충분
- 서버 → 클라만 필요: SSE가 더 간단
- 파일 다운로드: 그냥 HTTP

---

## 한마디 요약

WebSocket은 HTTP로 악수(handshake)만 하고 그 TCP 연결을 양방향 실시간 채널로 바꿔 쓰는 프로토콜이다.
