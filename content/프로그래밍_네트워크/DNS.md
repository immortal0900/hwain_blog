# DNS

> `api.openai.com`을 쳤을 때 IP 주소 `104.18.7.192`로 바뀌는 과정 전체가 DNS 일이다. Agent 개발에서 "연결이 안 돼요"의 절반은 사실 DNS 문제, 나머지 절반은 방화벽이나 포트 문제다. 단계별로 풀어둔다.

관련 문서: [[HTTP_HTTPS]], [[포트]], [[프록시]]

---

## 1. DNS란

**DNS(Domain Name System, 도메인 이름 시스템, 사람이 기억하기 쉬운 이름을 컴퓨터가 쓰는 IP 주소로 번역해주는 분산 데이터베이스)** 는 인터넷의 전화번호부다.

- 사람이 쓰는 이름: `api.anthropic.com`
- 컴퓨터가 쓰는 주소: `160.79.104.10` (IPv4) 또는 `2606:4700::6812:7c0` (IPv6)

이 둘을 맞바꿔주는 게 DNS.

**비유:** 전화번호부다. "홍길동 집"이라고 적힌 이름을 찾아 실제 전화번호를 보여주는 것. 한 번 본 번호는 수첩(캐시)에 적어두고 다음엔 바로 건다.

---

## 2. 도메인 계층 구조

도메인은 오른쪽부터 왼쪽으로 점점 구체화된다. 포함 관계로 보면:

```
.                          ← root (점, 보통 생략)
 └─ com                    ← TLD (Top-Level Domain, 최상위 도메인)
     └─ anthropic          ← SLD (Second-Level Domain, 2차 도메인, 등록한 이름)
         └─ api            ← 서브도메인 (Subdomain)
```

즉 `api.anthropic.com.` (끝의 점이 root, 보통 생략)

### TLD 종류

| 유형 | 예시 |
|------|------|
| gTLD(일반) | `.com`, `.org`, `.net`, `.io`, `.ai` |
| ccTLD(국가) | `.kr`, `.jp`, `.us`, `.uk` |
| new gTLD | `.dev`, `.app`, `.cloud` |

---

## 3. 이름 해석(Resolution) 단계 분해

`api.anthropic.com`을 처음 방문하는 상황을 예로 들자.

### 1단계 → 로컬 캐시 확인

OS와 브라우저는 최근에 해석한 이름을 저장해둠. 있으면 바로 반환.

```bash
# Windows: 현재 DNS 캐시 보기
ipconfig /displaydns

# macOS
sudo killall -INFO mDNSResponder   # 로그로 덤프
```

### 2단계 → hosts 파일 확인

OS는 DNS 서버 묻기 전에 로컬 hosts 파일을 먼저 참조.

- Windows: `C:\Windows\System32\drivers\etc\hosts`
- Linux/macOS: `/etc/hosts`

개발 중에 특정 도메인을 강제로 내 서버로 보내고 싶을 때 자주 건드린다.

```
127.0.0.1   api.local.test
```

### 3단계 → Recursive Resolver 질의

**Recursive Resolver(재귀 해석기, 대신 여기저기 물어 다녀서 답을 가져다주는 DNS 서버)** 에 질문한다. 보통 공유기가 ISP 해석기를 쓰거나, 사용자가 직접 Cloudflare(`1.1.1.1`), Google(`8.8.8.8`) 같은 공용 DNS를 설정.

### 4단계 → Root → TLD → Authoritative 순회

Recursive Resolver 입장에서의 여정:

1. **Root 서버**에 묻기: ".com 담당은 누구?" → `.com` TLD 서버 주소 반환
2. **.com TLD 서버**에 묻기: "anthropic.com 담당은 누구?" → `anthropic.com` 권한 서버(Authoritative NS) 주소 반환
3. **anthropic.com 권한 서버**에 묻기: "api.anthropic.com IP?" → `160.79.104.10` 반환

### 5단계 → 캐싱 후 반환

Recursive Resolver는 TTL(Time To Live, 생존 시간, 초 단위로 캐시 유효 기간) 동안 결과를 저장하고 클라이언트에게 전달. 클라이언트 OS와 브라우저도 각자 캐시.

다음 방문부터는 1단계 또는 3단계에서 바로 응답이 오기 때문에 빠르다.

---

## 4. 레코드 타입(Record Type)

DNS가 저장하는 값은 여러 종류다.

| 레코드 | 뜻 | 예시 |
|--------|-----|------|
| **A** | 도메인 → IPv4 | `api.anthropic.com → 160.79.104.10` |
| **AAAA** | 도메인 → IPv6 | `... → 2606:4700::...` |
| **CNAME** | 도메인 → 다른 도메인(별칭) | `www.example.com → example.com` |
| **MX** | 메일 서버 지정 | `example.com → mx1.example.com (priority 10)` |
| **TXT** | 임의 텍스트(SPF, DKIM, 도메인 소유권 증명에 자주) | `v=spf1 include:_spf.google.com ~all` |
| **NS** | 권한 서버 지정 | `example.com → ns1.example.com` |
| **SOA** | 존(zone) 관리 정보 | 관리자 메일, 시리얼, TTL 기본값 |
| **SRV** | 서비스 위치 + 포트 | MS Teams, XMPP 같은 서비스 |
| **CAA** | 이 도메인에 인증서 발급 가능한 CA 지정 | 보안 강화 |

### CNAME vs A 차이점

| 항목 | A 레코드 | CNAME 레코드 |
|------|----------|--------------|
| 가리키는 대상 | IP 주소 | 다른 도메인 이름 |
| 서버 IP 바뀔 때 | 여기저기 고쳐야 함 | 원본만 고치면 자동 반영 |
| 루트 도메인(`example.com` 자체)에 사용 | 가능 | 불가 (대부분의 DNS에서) |

클라우드(CloudFront, Vercel 등)에 연결할 때 대부분 CNAME을 쓴다.

---

## 5. 직접 확인

```bash
# dig: DNS 질의 상세 결과
dig api.anthropic.com

# 짧게
dig api.anthropic.com +short

# 특정 레코드 타입
dig anthropic.com MX
dig anthropic.com NS
dig anthropic.com TXT

# 특정 DNS 서버로 질의
dig @1.1.1.1 api.anthropic.com

# Windows는 nslookup
nslookup api.anthropic.com
```

`dig`의 ANSWER SECTION을 보면 TTL 값도 확인 가능. 예를 들어 `300`이면 앞으로 5분 동안 이 결과가 캐시된다는 뜻.

---

## 6. DNS over HTTPS / DNS over TLS

**DoH(DNS over HTTPS, HTTPS 위로 DNS 질의를 감쌈)** , **DoT(DNS over TLS, TLS 위로 DNS 질의)** 는 DNS 내용을 암호화한다.

- 기존 DNS: 평문 UDP 53번 포트. ISP나 공공 와이파이가 내용을 볼 수 있음
- DoH: 포트 443으로 HTTPS 요청처럼 보여서 차단 어려움
- DoT: 포트 853, TLS로 암호화

최근 OS와 브라우저는 기본 DoH를 지원하는 쪽으로 이동 중.

---

## 7. 캐시와 TTL의 함정

### 바꿨는데 안 바뀌어요

도메인의 IP를 바꿨는데 접속이 여전히 옛날 서버로 간다면:

1. OS 캐시 → 남아있음
2. 브라우저 캐시 → 남아있음
3. Recursive Resolver 캐시 → TTL 만료까지 유지
4. 하위 DNS 서버들도 TTL만큼 유지

해결:
```bash
# Windows
ipconfig /flushdns

# macOS
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder

# Linux (systemd-resolved)
sudo systemd-resolve --flush-caches
```

중요 변경 며칠 전에는 TTL을 60초 정도로 낮춰두고 변경 후 다시 올리는 관례가 있음.

---

## 8. DNS와 TLS, HTTPS 연결의 관계

실제 API 호출 한 번의 흐름을 꿰어보면:

1. **DNS 해석**: `api.openai.com` → `104.18.7.192` (이 문서가 다루는 부분)
2. **TCP 연결**: `104.18.7.192:443`으로 3-way handshake ([[포트]] 참고)
3. **TLS 악수**: 인증서 검증 + 키 교환 ([[HTTP_HTTPS]] 참고)
4. **HTTP 요청 전송**: 실제 API 호출

1단계가 막히면 2~4단계는 시작도 못 한다. "DNS 해결 실패" 에러(`ENOTFOUND`, `getaddrinfo ENOTFOUND`)를 만나면 제일 먼저 `dig`와 hosts 파일 확인.

---

## 9. Agent 개발에서 자주 만나는 상황

- **회사 VPN 들어가니 API 호출 안 됨**: 내부 DNS 서버가 특정 외부 도메인만 막고 있을 수 있음. `dig @8.8.8.8 api.openai.com`으로 우회 테스트
- **Docker 컨테이너에서 호스트 이름 해석 실패**: 컨테이너의 `/etc/resolv.conf`가 기본 DNS를 못 찾음. `--dns 8.8.8.8` 옵션 추가
- **로컬 개발에서 도메인 변경 테스트**: `/etc/hosts`에 한 줄 추가해서 내 서버로 돌리기
- **Cloudflare 뒤의 API**: A 레코드가 여러 개에 TTL이 짧음. 매번 다른 IP가 나올 수 있음
- **`.local` 이름 안 됨**: mDNS(multicast DNS) 영역이라 일반 DNS가 해석 못 함

---

## 10. 왜 이렇게 복잡한가

- **분산 구조**: 세계 어디서나 빠르게 답하려면 중앙 집중형은 불가. 각 도메인 소유자가 자기 권한 서버를 운영
- **캐싱 필수**: 매 요청마다 root부터 물으면 root 서버가 터짐. TTL 기반 캐시로 부하 분산
- **계층화**: TLD마다 다른 조직이 운영(`.com`은 Verisign, `.kr`은 KISA). 위임 체계 덕에 확장 가능

---

## 한마디 요약

DNS는 "도메인 이름 → IP 주소" 번역을 root → TLD → 권한 서버 순으로 분산 조회하고 TTL만큼 캐시하는 시스템이다.
