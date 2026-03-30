# Http 기본 지식 정리

---

## 목차

1. [인터넷 네트워크](#1-인터넷-네트워크)
2. [URI와 웹 브라우저 요청 흐름](#2-uri와-웹-브라우저-요청-흐름)
3. [HTTP 기본](#3-http-기본)
4. [HTTP 메서드](#4-http-메서드)
5. [HTTP 메서드 활용](#5-http-메서드-활용)
6. [HTTP 상태코드](#6-http-상태코드)
7. [HTTP 헤더 - 일반](#7-http-헤더---일반)
8. [HTTP 헤더 - 캐시와 조건부 요청](#8-http-헤더---캐시와-조건부-요청)

---

## 1. 인터넷 네트워크

### IP (Internet Protocol)

- 지정한 IP 주소(IP Address)에 데이터 전달
- **패킷(Packet)** 단위로 데이터 전달
- 패킷: 출발지 IP, 목적지 IP, 전송 데이터 포함

**IP 프로토콜의 한계**

| 한계 | 설명 |
|------|------|
| 비연결성 | 패킷을 받을 대상이 없거나 서비스 불능 상태여도 전송 |
| 비신뢰성 | 중간에 패킷 소실 가능, 패킷 순서 보장 불가 |
| 프로그램 구분 불가 | 같은 IP를 사용하는 서버에서 여러 애플리케이션 구분 불가 |

---

### TCP / UDP

**인터넷 프로토콜 4계층**

```
애플리케이션 계층 (HTTP, FTP)
전송 계층         (TCP, UDP)
인터넷 계층       (IP)
네트워크 인터페이스 계층
```

**TCP (Transmission Control Protocol) 특징**

- **연결지향** - TCP 3 way handshake (가상 연결)
- **데이터 전달 보증** - 패킷 누락 시 알 수 있음
- **순서 보장**
- 신뢰할 수 있는 프로토콜, 현재는 대부분 TCP 사용

**3 way handshake 과정**

```
클라이언트 → SYN       → 서버
클라이언트 ← SYN + ACK ← 서버
클라이언트 → ACK       → 서버
(데이터 전송)
```

**UDP (User Datagram Protocol) 특징**

- 연결지향 X, 데이터 전달 보증 X, 순서 보장 X
- 단순하고 **빠름**
- IP에 포트와 체크섬 정도만 추가
- 애플리케이션에서 추가 작업 필요 (HTTP/3에서 활용)

---

### PORT

- 같은 IP 내에서 **프로세스 구분**
- IP가 아파트라면, PORT는 동/호수
- 0 ~ 65535 할당 가능
- **잘 알려진 포트 (Well-known port)**
  - FTP: 20, 21
  - TELNET: 23
  - HTTP: 80
  - HTTPS: 443

---

### DNS (Domain Name System)

- IP는 기억하기 어렵고, 변경될 수 있음
- **도메인 명 → IP 주소** 변환 서비스
- ex) `google.com` → `216.58.220.142`

---

## 2. URI와 웹 브라우저 요청 흐름

### URI (Uniform Resource Identifier)

- **URI**: 자원을 식별하는 통합된 방법
  - **URL** (Locator): 리소스의 **위치** 지정 → `https://google.com/search`
  - **URN** (Name): 리소스의 **이름** 지정 → `urn:isbn:8960777331`
- 거의 URL을 URI와 동일하게 사용

**URL 문법**

```
scheme://[userinfo@]host[:port][/path][?query][#fragment]

https://www.google.com:443/search?q=hello&hl=ko
```

| 요소 | 설명 | 예시 |
|------|------|------|
| scheme | 프로토콜 | `https` |
| host | 도메인 또는 IP | `www.google.com` |
| port | 포트 (생략 시 scheme 기본값) | `443` |
| path | 리소스 경로 | `/search` |
| query | 쿼리 파라미터 | `q=hello&hl=ko` |
| fragment | html 내부 북마크, 서버 전송 X | `#getting-started` |

---

### 웹 브라우저 요청 흐름

```
1. DNS 조회 → IP/PORT 확인
2. HTTP 요청 메시지 생성
3. SOCKET 라이브러리를 통해 전달
4. TCP/IP 패킷 생성 (HTTP 메시지 포함)
5. 요청 패킷 전달
6. 서버에서 HTTP 응답 메시지 생성
7. 응답 패킷 전달
8. 브라우저 HTML 렌더링
```

---

## 3. HTTP 기본

### HTTP란?

- **HyperText Transfer Protocol**
- HTML, TEXT, IMAGE, 음성, 영상, JSON, XML 등 **거의 모든 형태의 데이터** 전송 가능
- 서버 간 데이터를 주고 받을 때도 대부분 HTTP 사용

**HTTP 역사**

| 버전 | 특징 |
|------|------|
| HTTP/0.9 (1991) | GET 메서드만 지원, 헤더 없음 |
| HTTP/1.0 (1996) | 메서드·헤더 추가 |
| **HTTP/1.1 (1997)** | 가장 많이 사용, 현재 주요 버전 (RFC 7230~7235) |
| HTTP/2 (2015) | 성능 개선 |
| HTTP/3 (진행중) | TCP 대신 UDP 사용, 성능 개선 |

---

### HTTP 특징

**1. 클라이언트 서버 구조**
- Request / Response 구조
- 클라이언트는 서버에 요청을 보내고 응답을 대기
- **비즈니스 로직, 데이터** → 서버 / **UI, 사용성** → 클라이언트 (독립적 진화)

**2. 무상태 프로토콜 (Stateless)**

| 구분 | 설명 |
|------|------|
| Stateful (상태 유지) | 서버가 클라이언트 상태 보존 → 서버 교체 불가 |
| **Stateless (무상태)** | 서버가 클라이언트 상태 보존 안 함 → 서버 교체 가능, 수평 확장 용이 |

- 한계: 로그인 같이 상태 유지가 필요한 경우 → **쿠키, 세션**으로 해결

**3. 비연결성 (Connectionless)**
- HTTP는 기본적으로 연결을 유지하지 않음
- 서버 자원을 효율적으로 사용
- 한계: TCP/IP 연결을 매번 새로 맺어야 함 (3 way handshake 시간 추가)
- 해결: **HTTP 지속 연결 (Persistent Connections)** 로 문제 해결

---

### HTTP 메시지 구조

```
start-line     (요청: request-line / 응답: status-line)
header
empty line (CRLF)
message body
```

**요청 메시지 예시**

```
GET /search?q=hello HTTP/1.1
Host: www.google.com
(공백)
```

**응답 메시지 예시**

```
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 3423
(공백)
<html>...</html>
```

---

## 4. HTTP 메서드

### 주요 메서드

| 메서드 | 설명 |
|--------|------|
| **GET** | 리소스 조회 |
| **POST** | 요청 데이터 처리 (주로 등록) |
| **PUT** | 리소스 대체, 없으면 생성 |
| **PATCH** | 리소스 부분 변경 |
| **DELETE** | 리소스 삭제 |
| HEAD | GET과 동일하나 body 제외 |
| OPTIONS | 대상 리소스의 통신 옵션 (CORS에서 사용) |

---

### GET

```http
GET /members/100 HTTP/1.1
Host: localhost:8080
```

- 리소스 **조회**
- 서버에 전달할 데이터는 **쿼리 파라미터** 사용
- 메시지 바디 사용 가능하나 권장하지 않음

---

### POST

```http
POST /members HTTP/1.1
Content-Type: application/json

{
  "username": "hello",
  "age": 20
}
```

- 요청 데이터 **처리**
- 주로 신규 리소스 **등록**, 프로세스 처리
- 다른 메서드로 처리하기 애매한 경우 사용
- **멱등 X** (같은 요청을 여러 번 보내면 여러 번 생성됨)

---

### PUT vs PATCH

| 메서드 | 설명 |
|--------|------|
| **PUT** | 리소스를 **완전히 대체** (없으면 생성) → 일부 필드만 보내면 나머지 필드 삭제됨 |
| **PATCH** | 리소스 **부분 변경** → 보낸 필드만 수정됨 |

---

### HTTP 메서드 속성

| 메서드 | 안전(Safe) | 멱등(Idempotent) | 캐시 가능 |
|--------|-----------|------------------|----------|
| GET | O | O | O |
| HEAD | O | O | O |
| POST | X | X | △ |
| PUT | X | O | X |
| PATCH | X | X | △ |
| DELETE | X | O | X |

- **안전(Safe)**: 호출해도 리소스 변경 없음
- **멱등(Idempotent)**: 몇 번을 호출해도 결과가 동일 (자동 복구 메커니즘 활용)
- **캐시 가능**: 응답 결과를 캐시해서 사용 가능

---

## 5. HTTP 메서드 활용

### 클라이언트에서 서버로 데이터 전송

**전달 방식 2가지**

1. **쿼리 파라미터** (GET) - 정렬 필터, 검색어
2. **메시지 바디** (POST, PUT, PATCH) - 회원 가입, 상품 주문, 리소스 등록/변경

**4가지 상황**

| 상황 | 방법 |
|------|------|
| 정적 데이터 조회 | GET + 이미지·정적 텍스트 경로 |
| 동적 데이터 조회 | GET + 쿼리 파라미터 |
| HTML Form 전송 | POST (multipart/form-data - 파일 업로드) |
| HTTP API 전송 | POST/GET/PUT/DELETE + JSON |

---

### HTTP API 설계 예시

**REST API 설계 원칙 - 리소스와 행위 분리**

```
# 나쁜 설계 (행위가 URI에 포함)
POST /createMember
GET  /readMember?id=1
POST /updateMember
POST /deleteMember

# 좋은 설계 (리소스 중심)
POST   /members       → 등록
GET    /members       → 목록 조회
GET    /members/{id}  → 단건 조회
PUT    /members/{id}  → 수정 (전체)
PATCH  /members/{id}  → 수정 (부분)
DELETE /members/{id}  → 삭제
```

**컨트롤 URI**: 도저히 리소스로 설계하기 어려운 경우 (동사 허용)

```
POST /orders/{orderId}/start-delivery
```

---

## 6. HTTP 상태코드

### 상태코드 분류

| 코드 | 분류 | 설명 |
|------|------|------|
| **1xx** | Informational | 요청 수신, 처리 중 (거의 사용 안 함) |
| **2xx** | Successful | 요청 정상 처리 |
| **3xx** | Redirection | 요청 완료를 위해 추가 행동 필요 |
| **4xx** | Client Error | 클라이언트 오류, 잘못된 문법 등 |
| **5xx** | Server Error | 서버 오류, 서버 처리 실패 |

---

### 2xx - 성공

| 코드 | 이름 | 설명 |
|------|------|------|
| 200 | OK | 요청 성공 |
| 201 | Created | 요청 성공 + 새로운 리소스 생성, Location 헤더로 위치 알림 |
| 202 | Accepted | 요청 접수, 처리 미완료 (배치 처리 등) |
| 204 | No Content | 성공, 응답 바디 없음 (save 버튼 등) |

---

### 3xx - 리다이렉션

> 응답 결과에 Location 헤더가 있으면, 해당 위치로 자동 이동

**영구 리다이렉션**: 원래 URL 사용 X (검색엔진도 인식)

| 코드 | 이름 | 설명 |
|------|------|------|
| 301 | Moved Permanently | 리다이렉트 시 GET으로 변경, 본문 제거 가능 |
| 308 | Permanent Redirect | 리다이렉트 시 메서드와 본문 유지 |

**일시 리다이렉션**: 현재 URL 사용 (검색엔진 인식 X)

| 코드 | 이름 | 설명 |
|------|------|------|
| 302 | Found | 리다이렉트 시 GET으로 변경 가능, 본문 제거 가능 |
| 303 | See Other | 리다이렉트 시 GET으로 변경 |
| 307 | Temporary Redirect | 리다이렉트 시 메서드와 본문 유지 |

**PRG 패턴 (Post/Redirect/Get)**

```
POST /order (주문) → 302 Found (Location: /order/result) → GET /order/result
→ 새로고침해도 GET 요청만 재전송 → 중복 주문 방지
```

**기타 리다이렉션**

| 코드 | 이름 | 설명 |
|------|------|------|
| 304 | Not Modified | 캐시 사용, 바디 없음 |

---

### 4xx - 클라이언트 오류

| 코드 | 이름 | 설명 |
|------|------|------|
| 400 | Bad Request | 요청 구문, 메시지 오류 |
| 401 | Unauthorized | 인증(Authentication) 필요 |
| 403 | Forbidden | 인가(Authorization) 거부 (권한 없음) |
| 404 | Not Found | 리소스 없음 |

> **401 vs 403**: 401은 로그인이 필요한 상태, 403은 로그인은 했으나 권한 없는 상태

---

### 5xx - 서버 오류

| 코드 | 이름 | 설명 |
|------|------|------|
| 500 | Internal Server Error | 서버 내부 오류 |
| 503 | Service Unavailable | 서버 일시적 과부하, 유지보수 중 |

> 4xx는 같은 요청을 반복해도 실패, 5xx는 서버 복구 시 성공 가능

---

## 7. HTTP 헤더 - 일반

### 헤더 분류

```
GET /search HTTP/1.1
Host: www.google.com          ← 요청 헤더
User-Agent: Mozilla/5.0 ...   ← 요청 헤더

HTTP/1.1 200 OK
Content-Type: text/html       ← 응답 헤더
Content-Length: 3423          ← 응답 헤더
```

---

### 표현 헤더 (Representation Headers)

> 메시지 본문(payload)에 포함된 데이터를 설명

| 헤더 | 설명 | 예시 |
|------|------|------|
| Content-Type | 표현 데이터 형식 | `text/html; charset=utf-8` |
| Content-Encoding | 표현 데이터 압축 방식 | `gzip`, `deflate`, `identity` |
| Content-Language | 표현 데이터 자연 언어 | `ko`, `en`, `en-US` |
| Content-Length | 표현 데이터 길이 (바이트) | `3423` |

---

### 협상 헤더 (Content Negotiation)

> 클라이언트가 선호하는 표현 요청 (요청 시에만 사용)

| 헤더 | 설명 |
|------|------|
| Accept | 클라이언트가 선호하는 미디어 타입 |
| Accept-Charset | 선호하는 문자 인코딩 |
| Accept-Encoding | 선호하는 압축 인코딩 |
| Accept-Language | 선호하는 자연 언어 |

**우선순위 (Quality Values, q)**

```
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
```

- q 값이 클수록 높은 우선순위 (생략 시 q=1)
- 구체적인 것이 우선

---

### 전송 방식

| 방식 | 헤더 | 설명 |
|------|------|------|
| 단순 전송 | Content-Length | 전체 크기를 알 때 |
| 압축 전송 | Content-Encoding: gzip | 압축해서 전송 |
| 분할 전송 | Transfer-Encoding: chunked | 크기를 모를 때 나눠서 전송, Content-Length 사용 불가 |
| 범위 전송 | Range, Content-Range | 특정 범위만 요청 |

---

### 일반 정보 헤더

| 헤더 | 방향 | 설명 |
|------|------|------|
| From | 요청 | 유저 에이전트의 이메일 (검색 엔진) |
| Referer | 요청 | 이전 웹 페이지 주소 (유입 경로 분석) |
| User-Agent | 요청 | 클라이언트 애플리케이션 정보 |
| Server | 응답 | 서버 소프트웨어 정보 |
| Date | 응답 | 메시지 생성 날짜 |

---

### 특별 정보 헤더

| 헤더 | 방향 | 설명 |
|------|------|------|
| Host | 요청 (필수) | 요청한 호스트 정보 (하나의 서버에 여러 도메인 처리 시 필수) |
| Location | 응답 | 3xx 리다이렉션 위치 |
| Allow | 응답 | 허용 가능한 HTTP 메서드 (405 응답 시 포함) |
| Retry-After | 응답 | 503 응답 시 서비스 재개 예상 시간 |

---

### 인증 헤더

| 헤더 | 방향 | 설명 |
|------|------|------|
| Authorization | 요청 | 클라이언트 인증 정보 (`Bearer <token>`) |
| WWW-Authenticate | 응답 | 401 응답 시 인증 방법 명시 |

---

### 쿠키

```http
Set-Cookie: sessionId=abc123; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=Lax
```

**쿠키 사용 이유**: HTTP는 무상태 프로토콜 → 로그인 상태 유지를 위해 쿠키 사용

**쿠키 속성**

| 속성 | 설명 |
|------|------|
| expires / max-age | 쿠키 만료 시간 (생략 시 브라우저 종료까지 유지 = 세션 쿠키) |
| domain | 도메인 지정 (생략 시 현재 도메인만, 명시 시 서브도메인 포함) |
| path | 경로 포함 하위 경로 페이지만 쿠키 접근 |
| Secure | HTTPS인 경우에만 전송 |
| HttpOnly | JavaScript에서 접근 불가 (XSS 공격 방지) |
| SameSite | XSRF 공격 방지, 요청 도메인과 쿠키 도메인 일치 시에만 전송 |

---

## 8. HTTP 헤더 - 캐시와 조건부 요청

### 캐시 기본 동작

**캐시가 없을 때**: 같은 리소스를 요청할 때마다 서버에서 재전송 → 네트워크 낭비

**캐시 적용**

```http
HTTP/1.1 200 OK
Cache-Control: max-age=60    ← 캐시 유효 시간 60초
Content-Type: image/jpeg
Content-Length: 1000
```

- 브라우저 캐시에 저장, 유효 기간 내 재요청 시 캐시에서 조회
- 캐시 유효 시간 초과 시 → **조건부 요청**으로 검증

---

### 검증 헤더와 조건부 요청

**Last-Modified / If-Modified-Since**

```http
(최초 응답)
HTTP/1.1 200 OK
Last-Modified: 2020-11-10 10:00:00

(재요청 - 캐시 만료 후)
GET /star.jpg
If-Modified-Since: 2020-11-10 10:00:00

(서버 응답 - 데이터 미변경)
HTTP/1.1 304 Not Modified   ← 바디 없음, 헤더만 전송
```

**ETag / If-None-Match** (더 권장)

```http
(최초 응답)
HTTP/1.1 200 OK
ETag: "aaabbbccc"

(재요청)
GET /star.jpg
If-None-Match: "aaabbbccc"

(서버 응답 - 데이터 미변경)
HTTP/1.1 304 Not Modified
```

- ETag: 파일 해시값 (날짜와 무관하게 내용이 같으면 동일한 ETag)
- 캐시 제어 로직을 서버에서 완전히 관리

---

### 캐시 제어 헤더

**Cache-Control 주요 지시어**

| 지시어 | 설명 |
|--------|------|
| `max-age=초` | 캐시 유효 시간 (초 단위), 권장 |
| `no-cache` | 데이터 캐시 가능, 항상 **원서버에서 검증** 후 사용 |
| `no-store` | 데이터에 민감한 정보 → 저장 X |
| `must-revalidate` | 캐시 만료 후 재검증 전까지 사용 X (원서버 장애 시 504 반환) |
| `public` | public 캐시에도 저장 가능 |
| `private` | 해당 사용자만을 위한 캐시 (기본값) |
| `s-maxage` | 프록시 캐시에만 적용되는 max-age |

---

### 프록시 캐시

```
클라이언트          프록시 캐시 서버           원서버 (Origin)
   ──────── 요청 ──────────→                 (미국)
            (한국 내 위치)
   ←──── 응답 (빠름) ─────←  ←── 원서버에서 받아옴 ───
```

- 첫 번째 사용자는 느리고, 이후 사용자는 프록시 캐시에서 빠르게 조회
- `Cache-Control: public` → 프록시 캐시 저장 가능
- `Cache-Control: private` → 프록시 캐시 저장 불가, 개인 브라우저 캐시만

---

### 캐시 무효화

```http
Cache-Control: no-cache, no-store, must-revalidate
Pragma: no-cache    ← HTTP 1.0 하위호환
```

> 통장 잔고, 재고처럼 절대 캐시해선 안 되는 데이터에 사용  
> 세 가지를 모두 넣어야 완벽한 캐시 무효화

---

## 핵심 요약

| 개념 | 핵심 포인트 |
|------|------------|
| TCP | 연결지향, 신뢰성 보장, 3 way handshake |
| UDP | 빠름, 신뢰성 없음, 개발자가 직접 구현 |
| HTTP | 무상태(Stateless), 비연결성, 텍스트 기반 |
| REST | 리소스 중심 URI + HTTP 메서드로 행위 표현 |
| 상태코드 | 2xx 성공 / 3xx 리다이렉션 / 4xx 클라이언트 오류 / 5xx 서버 오류 |
| 쿠키 | 무상태 HTTP에서 상태 유지, HttpOnly/Secure 보안 설정 필수 |
| 캐시 | Cache-Control로 제어, ETag로 변경 여부 검증 |

---

> 참고: 인프런 [모든 개발자를 위한 HTTP 웹 기본 지식] - 김영한
> 본 문서는 강의를 수강하며 개인 학습 목적으로 정리한 노트입니다.
