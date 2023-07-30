# Chapter 06 - Go 언어를 이용한 HTTP1.1 클라이언트 구현

- Chapter 04, 05 내용을 직접 코드로 구현함으로써 이해를 돕기 위한 챕터

<br/>

## 6.1 Keep-Alive

- Go 언어에서는 HTTP API에서 따로 설정하지 않더라도 기본적으로 `Keep-Alive` 유효
- 단, 세션을 유지하려면 에러가 발생한 경우를 제외하고 클라이언트에서 response.Body를 끝까지 읽은 후 명시적으로 close 해야 함

```go
resp, err := http.Get("http://...")
if err != nil {
  // 오류 발생
  panic(err)
}
// 이 스코프를 벗어난 곳에서 반드시 닫음
defer resp.Body.Close()
// ioutil.ReadAll로 서버 응답을 끝까지 일괄적으로 읽음
body, err := ioutil.ReadAll(resp.Body)
```

<br/>

## 6.2 TLS

### 6.2.1 인증서 만들기

하나의 증명서를 만드는 기본 흐름은 다음과 같음

- OpenSSL 커맨드로 비밀 키 파일 생성
- 인증서 요청 파일 생성
- 인증서 요청 파일에 서명하여 인증서 생성

OpenSSL 권장 설정 항목

```
[req_distinguised_name]
// 기본 국가 코드
countryName_default =

// 기본 도/주
stateOrProvinceName_default =

// 기본 도시명
localityName_default =

// 기본 조직명
0.organizationName_default =

// 기본 관리자 메일 주소
emailAddress_default
```

### 6.2.2 HTTPS 서버와 인증서 등록

- curl 버전 확인 시 libcurl 다음에 있는 글자가 TLS 구현할 때 사용되는 라이브러리 이름

```bash
$ curl --version
curl 7.43.0 (x86_64-apple-drawin15.0) libcurl/ 7.43.0 SecureTransport zlib/1.2.5
```

- OpenSSL: OpenSSL(포터블 라이브러리)
- Schannel: 윈도우의 보안 관리 시스템
- SecureTransport: 맥 OS의 보안 관리 시스템
- NSS: 몇몇 리눅스 배포판에서 이용되는 보안 관리 시스템

### 6.2.3 Go 언어를 이용한 클라이언트 구현

- x509: ISO에서 규정한 인증서 형식
- PEM: BASE64로 부호화된 바이너리에 헤더와 푸터를 붙인 데이터 구조

### 6.2.4 클라이언트 인증서

- TLSConfig의 ClientAuth에 아래 ENUM 값 중 하나를 설정하면 클라이언트 인증서에 관한 움직임이 변화
  - NoClientCert: 클라이언트의 증명서를 요구하지 않음 (default)
  - RequestClientCert: 클라이언트 증명서 요구
  - RequireAnyClientCert: 어떤한 클라이언트 증명서라도 필요
  - VerifyCleintCertIfGiven: 만약 클라이언트 증명서가 주어진다면 검증
  - RequireAndVerifyClientCert: 클라이언트 증명서를 요구하고 검증

<br/>

## 6.3 프로토콜 업그레이드

- 프로토콜 업그레이드는 통신 도중에 HTTP 이외의 통신을 하는 방법이었음
- 업그레이드하면 HTTP의 컨텍스트에서 벗어나 통신하므로 직접 소켓을 송수신하게 됨

### 6.3.1 서버 코드

- http.ResponseWriter를 http.Hijacker로 캐스팅해 하이재킹
  - 하이재킹하면 http.ResponseWriter는 아무런 전송 메시지를 보내지 않게 됨
- bufio.ReadWriter

### 6.3.2 클라이언트 코드

- 클라이언트 쪽도 소켓을 직접 다룸
- 서버와 같은 하이재킹 구조가 없으므로, 통신을 시작할 때부터 소켓을 다룸

<br/>

## 6.4 청크

- Go 언어의 청크 지원은 이미 net/http의 각 기능에 포함되어 있었음
- 단, 통신 패키지 내부에 은폐되어 있으므로 통신에서 청크 형식이 사용되었는지는 외부에서 알 수 없음
- http.Post로 `2048Byte` 이상의 파일을 전송할 때, Request.ContentLength를 설정하지 않으면 자동으로 청크 형식으로 전송됨

### 6.4.1 서버에서 송신하기

```go
fucn handlerCunkedResponse(w http.ResponseWriter, r *http.Request) {
  flusher, ok := w.(http.Flusher)
  if (!ok) {
    panic("expected http.ResponseWriter to be an http.Flusher")
  }

  for i := 1; i <= 10; i++ {
    fmt.Fprintf(w, "Chunk #%d\n", i)
    flusher.Flush()
    time.Sleep(500 * time.Millisecond)
  }
  flusher.Flush()
}
```

### 6.4.2 클라이언트에서 순차적으로 수신하기(간단판)

```go
package main

import (
  "bufio"
  "bytes"
  "io"
  "log"
  "net/http"
)

func main() {
  resp, err := http.Get("http://...")
  if err != nil {
    log.Fatal(err)
  }
  defer resp.Body.Close()
  reader := bufio.NewReader(resp.Body)
  for {
    line, err := reader.ReadBytes('\n')
    if err == io.EOF {
      break
    }
    log.PrintIn(string(bytes.TrimSpace(line)))
  }
}
```

### 6.4.3 클라이언트에서 순차적으로 수신하기(완전판)

```go

```

<br/>

## 6.5 원격 프로시저 호출

- Go 언어는 net/rpc 패키지에서 RPC를 실현하는 프레임워크 제공
  - 오브젝트를 생성해 등록하면 외부에서 접근 가능
- Go 언어용 직렬화 포맷으로 서버와 클라이언트가 통신하지만, 코덱을 지정하면 다른 형식으로도 전환 가능

net/rpc 공개 메서드(대문자로 시작하는 이름) 충족 조건

- 메서드가 속한 구조체의 `형`이 공개되어 있다.
- `메서드`가 공개되어 있다.
- 메서드는 `두 개 인수`를 가지며, 양쪽 다 공개되어 있거나 내장형이다.
- 메서드의 두 번째 인수는 `포인터`이다.
- 메서드는 error 형의 반환값을 가진다.

<br/>

## 6.6 마치며
