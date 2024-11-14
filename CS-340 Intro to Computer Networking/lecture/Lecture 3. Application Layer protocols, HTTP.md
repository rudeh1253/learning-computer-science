# Last lecture
- 패킷이 목적지에 도달하기까지 여러 hop을 거친다.
  - 라우터는 고정된 사이즈의 Queue를 가지고 있으며, Queue가 꽉 차면 이후에 들어오는 패킷은 버려진다.
  - 패킷 bit에 에러가 감지되면 그 패킷은 버려진다.
- 각 hop마다 4가지의 packet delay가 발생한다.
  - Nodal processing, queueing (라우터 관련)
  - Transmission, propagation (Link 관련)
- 인터넷은 network의 network이다.
  - Tier 1 ISP와 Content provider는 속도가 빠른 backbone link로 연결된다.
  - Peering은 네트워크끼리 연결되는 것을 의미한다. (ISP 간에 요금이 발생하지 않음)
- 네트워크는 layered protocol을 사용한다. e.g. Ethernet, IP, TCP, TLS, HTTP
- Socket은 네트워크 연결의 소프트웨어 추상화이다. (TCP, UDP)
  
# Separation of concerns
네트워크는 여러 Layer로 구성되어 있으며, Layer는 다른 Layer와 decoupling 관계를 유지한다.

# Application-layer protocols
- Application layer 프로토콜의 목적은 서로 다른 컴퓨터에서 돌아가고 있는 애플리케이션끼리 통신하는 것이다.
  - 통신을 컨트롤하는 곳이 어딘지에 따라 client-server 흑은 peer-to-peer 아키텍처를 이룬다.
- 애플리케이션은 low-level 디테일에 대해 알지 못한다 (Separation of concerns)

# Client-server architecture
- Server: 요청 처리
  - 항상 켜져 있음
  - 영구적 IP 주소
  - DNS hostname을 보통 가지고 있음
  - 보통 데이터 센터에 위치함
  - 클라이언트로부터의 요청을 **듣고** 있음
- Clients: 요청 생성
  - **요청하지 않은 message는 듣지 않는다**
    - 오직 자신의 요청에 대한 응답만 듣는다
  - Client끼리는 직접 통신하지 않는다. Server가 Client끼리의 통신을 중개한다.

# Peer-to-peer architecture (P2P)
- 모든 참여자들이 동등한 책임을 지닌다. (peer)
  - 중앙 서버에 의존하지 않는다.
- 매우 scalable한 구조이다.
  - 새로운 참여자가 나타날 때마다 peer-to-peer 아키텍처의 capacity가 늘어난다.
- Client-server architecture에 비해 구성하기가 어렵다.
  - Host들은 네트워크에 자유로이 참여하거나 떠날 수 있다.
  - IP 주소가 바뀐다.
  - 방화벽이 peer의 접근을 막을 수 있다.
  - Edge network의 업로드 속도는 제한되어 있다.
- Examples: BitTorrent, Skype
  - SMTP 또한 P2P로 생각할 수 있다.

# Hyper Text Transfer Protocol (HTTP)
- HTTP는 TCP 위에서 작동하며 client-server architecture에 입각한 데이터 교환을 위한 프로토콜이다.
  - TCP는 두 노드 간에 신뢰할 수 있는 (reliable) 양방향 (bi-directional) 데이터 스트림을 제공한다.
- HTTP는 기본적으로 **Hyper text**를 교환하기 위해 설계되었다. (프라우저가 웹 서버로부터 웹 페이지를 가져오기 위해 설계되었다)
- Request/Response
- Request는 다음 항목을 가지고 있다.
  - Human-readable 헤더. 다음을 포함하고 있다: URL, method (+ optional headers)
  - Optional body (raw 데이터를 포함하고 있음 - bytes)
- Response는 다음 항목을 포함하고 있다.
  - Human-readable headers. 다음을 포함하고 있다: Response code (+ optional headers)
  - Optional body
- HTTP는 **stateless** 프로토콜이다.
  - 모든 요청은 서로 간에 독립적이다. - 각 요청은 필요한 모든 데이터를 가지고 있다.
  - 서버는 클라이언트의 요청 각각을 독립적으로 처리한다. - 각 요청을 처리할 때마다 이전 요청은 고려하지 않는다.

# Request/Response 구조

## Request
```
GET /doc/test.html HTTP/1.1
HOST: www.test101.com
Accept: image/gif, image/jpeg, */*
Accept-Language: en-us
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/4.0
Content-Length: 35

bookId=12345&author=Tan+Ah+Teck
```
- 첫 번째 라인은 *Request Line*이다.
- 두 번째 라인부터 빈 라인 (Content-Length 헤더 바로 아래에 있는 빈 줄)까지는 *Request Header*이다.
- Request Line과 Request Headers를 합쳐서 *Request Message Header*라고 한다.
- Request Message Header는 사람이 읽을 수 있는 String 타입 데이터이다.
- 빈 줄 아래는 Request Body이다. (Optional for GET)
- Content-Length 헤더의 값을 통해 Request 메시지가 끝나는 시점을 알 수 있다.

## Response
```
HTTP/1.1 200 Ok
Date: Sun, 08 Feb xxxx 01:11:12 GMT
Server: Apache/1.3.29 (Win32)
Last-Modified: Sat, 07 Feb xxxx
ETag: "0-23-4024c3a5"
Accept-Ranges: bytes
Content-Length: 35
Connect: close
Content-Type: text/html

<h1>My Home page</h1>
```
-  첫 번째 라인은 *Status Line*이다.
-  두 번째 라인부터 빈 라인까지는 *Response Header*이다.
-  Status Line과 Response Headers를 합쳐서 *Response Message Header*라고 한다.
-  빈 줄 아래는 Response Body이다.

# HTTP transaction 단계
1. Client에서 TCP 소켓을 생성하고 서버에서 Socket을 받아들인다.

TCP socket: bi-directional pipe/stream of bytes

2. Client에서 HTTP 요청을 Socket에 보낸다. 그리고 응답을 기다린다. (**Listen**)
3. Server는 Socket을 통해 새로운 데이터가 왔다는 것을 알아채고 Request data를 읽어들이기 시작한다.
4. Server가 모든 HTTP 요청 데이터를 받았다는 걸 알게 되면 웹 서버에서 해당 요청을 처리한다.
5. 요청 처리가 완료되면 요청에 대한 응답을 Socket을 통해 전송한다.
6. Client는 응답을 읽어들이고 파싱한다. 응답이 완료되면 읽어들이기를 멈춘다.

# HTTP method와 response
## Method
- GET: 데이터 요청
- POST: 데이터를 서버에 전송하고 처리 요청
- PUT: 새로운 document를 서버에 생성
- DELETE: document 삭제
- HEAD: GET과 비슷하지만 오직 헤더만 반환

## Response codes
- 200 OK: 성공
- 301 Moved Permanently: 다른 URI로 리다이렉팅
- 403 Forbidden: 인가 오류
- 404 Not Found: 해당 리소스가 존재하지 않음 (URL에 문제가 있음)
- 500 Internal Server Error: 포괄적인 서버 에러 코드

# Cookie
HTTP는 stateless protocol이다. 그런데 클라이언트가 로그인을 했으면 클라이언트 로그인 정보를 유지해야 한다. Stateless한 HTTP 프로토콜에서 State를 유지하기 위해 Cookie를 사용할 수 있다.

- Cookie는 웹 애플리케이션이 클라이언트의 **상태**를 추적하는 수단이다.
- 클라이언트의 로그인 상태를 Cookie로 저장한다고 한다면
  - Cookie는 브라우저에 저장된다.
  - 클라이언트의 요청마다 Cookie 정보를 포함하여 클라이언트의 로그인 여부를 나타낸다.
- Cookie를 사용한다고 해서 HTTP가 Stateful protocol이 되는 것은 아니다. 서버는 여전히 클라이언트의 이전 요청을 기억하지 않는다. 단지 클라이언트에서 Cookie에 추가적인 데이터를 집어넣어 요청을 보내는 것이다.

# HTTP와 Web의 진화
## 1990년대 초반
HTTP는 단지 document를 가져오는 서비스였을 뿐이다.

- 웹 서버는 정적 HTML과 이미지 파일을 제공했을 뿐이다.
- ```GET /index.html```은 서버에 저장된 HTML 파일을 나타낸다.
  
## 1990년대 후반
웹 서버에서 스크립트를 실행하여 Content를 생성한다.

- ```GET /product/1234```는 "product 1234"에 해당하는 데이터를 데이터베이스에서 찾아 해당 데이터를 가지고 웹 페이지를 동적으로 생성한다.

## 2005년 이후
JavaScript를 활용해 페이지와 동적으로 상호작용할 수 있게 되었다.

- AJAX: 페이지를 리로딩하지 않고 HTTP 요청을 보낼 수 있음

## 2010년대
온세상이 HTTP다. 다만 여전히 HTTP는 request/response framework로 사용된다.

- HTTP 인프라가 방대해졌다.
  - libraries, software, caches, proxies, encryption, compression
- 거의 모든 Client-server, request-response architecture에서 HTTP가 유용하게 사용된다.
- e.g., 스마트폰 앱-to-서버, server-to-server

# 날씨 정보 서비스 예제 (REST API)
## HTTP Request
```
GET http://api.whr.com/[key]/forecase?location=San+Francisco HTTP/1.1
Accept-Encoding: gzip
Cache-Control: no-cache
Connection: keep-alive
```
- **REST Endpoint**
- HTTP 스타일의 요청 메시지이지만 웹 페이지를 요청하지 않는다.
- REST API를 구현하기 위해 HTTP를 사용하는 예제이다.

## HTTP Response
```
HTTP/1.1 200 OK
Content-Length: 2102
**Content-Type: application/json**

{
    "wind_dir": "NNW",
    "wind_degrees": 346,
    ...
}
```

# 왜 HTTP를 사용하는가?
- HTTP는 이미 많은 문제를 해결했다.
  - Encryption
  - Compression
  - 수많은 프로그래밍 언어가 이미 HTTP client 라이브러리를 가지고 있다.
  - 수많은 서버 프레임워크가 HTTP를 사용하고 있으며, 이 서버 프레임워크들은 encryption, queueing, database connection pooling과 같은 기능을 제공하고 있다.
    - e.g., Apache httpd, Tomcat, Node.js, Django, Flask
  - Web proxy와 cache가 재사용될 수 있다. (Squid, Nginx, Akamai, etc.)
  - HTTP response code는 다른 서비스에 적용될 수 있을 만큼 충분히 generic하다.
- 단점:
  - 불필요한 복잡성과 unexpected behavior를 가질 수 있다.
  - Human-readable header가 오버헤드를 야기할 수 있다. (압축이 여기서 도움이 된다)
  - API를 URL/resource 모델에 맞춰야 한다.

# Simple Mail Transfer Protocol (SMTP)
- TCP 위에서 동작하는 Protocol
- RFC 2821에 정의되어 있다.
- 기본적으로 25번 포트 사용
- HTTP보다 먼저 개발되었다. (1982)
- Mail server는 P2P protocol로 이루어져 있다.
  - Mail server는 메일을 전송할 때 client 역할을 하고 메일을 받을 때 서버 역할을 한다.
- User agent는 email을 가져올 때 다른 protocol을 사용한다. (IMAP, POP3, webmail)

## Example
S: Server
C: Client
```
S: 220 smtp.example.com ESMTP Postfix
C: HELO relay.example.com
S: 250 smtp.example.com, I am glad to meet you
C: MAIL FROM:<bob@example.com>
S: 250 OK
C: RCPT TO:<alice@example.com>
S: 250 OK
C: RCPT TO:<theboss@example.com>
S: 250 OK
C: DATA
S: 354 Emd data with <CR><LF>.<CR><LF>
C: From: "Bob Example" <bob@example.com>
C: To: "Alice Example" <alice@example.com>
C: Cc: theboss@example.com
C: Date: Tue, 15 January 2008 16:02:43 -0500
C: Subject: Test message
C:
C: Hello Alice.
C: This is a test message with 5 header fields and 4 lines in the emssage body
C: Your firend,
C: Bob
C: .
S: 250 OK: queued as 12345
C: QUIT
S: 221 Bye
```
SMTP는 ***stateful*** protocol이다. SMTP는 1982년에 stateful protocol로 설계되었다. 1982년에는 실제로 저렇게 수동으로 mail을 전송했다.