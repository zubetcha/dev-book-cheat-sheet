# Chapter 05 - HTTP/1.1의 시맨틱스: 확장되는 HTTP의 용도

- 파일 다운로드 (파일명 지정)
- 다운로드 중단과 재개 (범위 액세스)
- XMLHttpRequest
- 지오로케이션
- X-Powered-By
- 원격 프로시저 호출
- WebDAV
- 웹사이트 간 공통 인증, 허가 플랫폼

<br/>

### 5.1 파일 다운로드 후 로컬에 저장하기

- 브라우저가 파일을 어떻게 처리할 지 결정하는 것
  - 파일 확장자 X
  - 서버가 보낸 MIME 타입
- 서버 응답에 `Content-Disposition` 헤더가 있는 경우
  ```
  Content-Disposition: attachment; filename=filename.xlsx
  ```
  - 브라우저는 다운로드 대화상자를 표시하고 파일 저장
  - `filename`으로 지정된 파일명이 다운로드 대화상자에 기본값으로 표시됨
- 한글 파일명도 가능
  ```
  Content-Disposition: attachment; filename*=utf-8'' 파일명.xlsx; filename=filename.xlsx
  ```
  - UTF-8 인코딩 지정
- `Content-Disposition` 헤더를 이용한 다운로드 기능은 HTTP를 위해 만들어진 것은 아님
  - **이메일의 첨부파일**을 위해 정의된 규격

<br/>

### 5.2 다운로드 중단과 재시작

- 파일 크기가 크면 다운로드에 걸리는 시간도 길어지고, 통신이 불안정해지거나 다운로드가 도중에 실패할 확률 증가
- HTTP는 다운로드가 중단되면 중단 지점부터 다시 시작하는 방법 제공
  - 커다란 파일에서 지정한 범위를 잘라내 다운로드
  - 서버가 범위 지정 다운로드를 지원하는 경우 `Accept-Ranges` 헤더를 응답에 설정함
  ```
  Accept-Ranges: bytes
  ```

**범위 지정 다운로드**

- Accept-Ranges 헤더는 두 가지 값을 가질 수 있음
  - Accept-Ranges: bytes - 범위 지정 다운로드를 지원하며 단위는 바이트이다.
  - Accept-Ranges: none - 범위 지정 다운로드 지원 불가
- 다른 단위도 받을 수는 있으나 IANA에 단위를 등록해야 하며, 현재 등록되어 있는 단위는 `bytes`와 `none`뿐임
- 클라이언트는 요청에 `Range` 헤더를 추가해 원하는 범위 지정 가능
  ```
  Range: bytes=1000-1999
  ```
- 서버는 범위가 지정된 컨텐츠를 반환하며 다음과 같이 응답함
  - Status Code `206`
  ```
  HTTP/1.1 206 Partial Content
  :
  Content-Length: 1000 // 실제로 보낸 바이트 수
  Content-Ranges: 1000-1999/5000 // 범위/전체 바이트 수
  ```
- 클라이언트가 지정한 범위가 무효인 경우
  - Status Code `416`
  ```
  HTTP/1.1 416 Range Not Satisfiable
  :
  Content-Ranges: */5000
  ```

#### 복수 범위 다운로드

- `Ranges` 헤더를 사용해 복수의 범위도 지정 가능
  ```
  Range: bytes=500-999,7000-7999
  ```
- 이 경우 multipart 폼과 비슷한 `mutlipart/byteranges`라는 Content-Type 헤더로 응답

#### 병렬 다운로드

- 영역을 나눠 세션마다 Range 헤더를 이용해 HTTP 접속하여 세션별로 다운로드한 조각을 나중에 결합하는 방식
- 병렬 다운로드는 서버에 부하를 줄 수 있기 때문에 권장하지는 않음

### 5.3 XMLttpRequest
- curl 커맨드 기능을 자바스크립트로 사용할 수 있게 해주는 기능
- 처음에는 마이크로소프트의 IE5 용으로 설계되었지만 현재는 `WHATWG(Web Hypertext Application Working Group)`에서 사양이 정해져, 각종 브라우저에서 사용할 수 있게 됨 
- HTTP처럼 **클라이언트가 서버로** 요청을 보내는 것에는 변함이 없으나 **서버가 클라이언트로** 요청을 보내는 것은 불가능

```javascript
var xhr = new XMLHttpRequest();
xhr.open("GET", "/json", true);
xhr.onload = function () {
  // 응답이 돌아왔을 때 호출되는 메서드
  if (xhr.status === 200) {
    // JSON 파싱해서 표시
      console.log(JSON.parse(xhr.responseText));
  }
};
xhr.setRequestHeader("MyHeader", "HeaderValue")
xhr.send(JSON.stringify({"message":  "hello world"}));
```

- open() 메서드로 도착지와 메서드를 설정하며, 세 번째 인자를 `true`로 설정하면 `비동기`로 실행됨
- send() 메서드로 실제 전송 시작

#### XMLRequestHTTP와 브라우저의 HTTP(폼) 요청 차이

- 송수신할 때 HTML 화면이 새로고침되지 않음
- GET, POST 이외의 메서드도 전송 가능
- 폼의 경우 `key-value` 쌍의 데이터만 전송할 수 있지만, XMLHttpRequest는 `플레인 텍스트`, `JSON`, `바이너리 텍스트` 등 다양한 형식의 데이터 송수신 가능
- 몇 가지 보안상 제약이 있음

**Ajax**
- asynchronous Javascript + XML의 약자
- 화면을 지우지 않고 웹페이지를 읽어 오거나 시간이나 타이밍에 따라 몇 번이고 갱신할 수 있는 아키텍쳐
- 브라우저가 송수신하면 파일 다운로드를 제외한 나머지 서버 응답을 받을 때 화면이 지워지고 새로운 페이지가 렌더링됨
- XMLHttpRequest는 `자바스크립트`가 송수신을 담당하므로 화면이 지워지지 않음

#### 코멧

- `XMLHttpRequest`를 이용해 거의 실시간 `양방향 통신`할 수 있는 기능
- 단방향 통신을 이용해 양방향 통신을 하기 위한 두 가지 방법
  - `폴링`
  - `롱 폴링`
  
[long polling](pubnub.com/blog/http-long-polling/)

**폴링**
- 클라이언트가 서버 쪽으로 빈번하게 요청하는 방식
- 불필요한 요청과 응답 발생
  - 대역과 CPU 낭비
  - 모바일 환경이라면 소비 전력 낭비

<p align="center">
  <img src="https://images.ctfassets.net/3prze68gbwl1/20KLOkp9uPZUiZtjp5VpWn/e4e6f4e48f9f0129766611ead3da8fa6/HTTP_Polling.png" alt="폴링" />
</p>


**롱 폴링**
- 클라이언트가 서버로 요청을 보내면 서버는 바로 응답하지 않고 응답을 보류한 채 대기
- 서버가 통신을 종료하거나 요청이 타임아웃 될 때까지 응답이 돌아오지 않고 자유로운 타이밍에 서버 응답
- HTTP는 서버에서 클라이언트로 요청을 보내는 전용 API가 아님
- 또한 HTTP는 쿠키를 포함한 대량의 헤더를 송수신에 포함하는 구조
- 서버에서 클라이언트로 연속으로 응답을 해야 하는 케이스에는 약한 편

<p align="center">
  <img src="https://images.ctfassets.net/3prze68gbwl1/2FEB0j6VRCXSe28P5ruiSZ/a149b8fdbbdb77e5f3bb81aab9902dac/HTTP_Long_Polling.png" alt="롱 폴링" />
</p>



#### XMLHttpRequest의 보안

- `접근 가능한 정보 제한`과 `전송 제한`이라는 두 가지 제한으로 구성

**접근 가능한 정보 제한**

- 쿠키
  - 스크립트로 `document.cookie` 를 통해 브라우저의 쿠키 접근 가능
  - 이를 방지하기 위해 쿠키 속성 중 `httpOnly`를 설정하면 스크립트로 쿠키 접근 불가

**전송 제한**
- 도메인
  - `동일 출처 정책(same origin policy)`으로 요청을 보낼 수 있는 도메인 제한 설정
  - `교차 출처 리소스 공유(CORS, cross-origin resource sharing)` 액세스 제한 시스템
- 메서드
  - `CONNECT`, `TRACE`, `TRACK`을 메서드로 지정하면 open() 메서드 호출 시 `SecurityError` 예외 응답
- 헤더
  - 현재 금지되어 있는 것
    - 현재의 프로토콜 규약이나 환경에 영향을 미치는 것
    - 쿠키처럼 보안에 영향을 주는 것
    - 브라우저의 능력을 넘을 수 없을 것
      - 브라우저 자신이 지원하지 않는 압축 형식을 Accept-Encoding으로 지정하는 행위 등
  - 앞으로 사용될 것에 대비해 `Sec-`, `Proxy-`로 시작하는 키 이름도 금지


### 5.4 지오로케이션

#### 클라이언트 자신이 위치를 구하는 방법
- 모던 브라우저는 지오로케이션 API 제공
  - 모바일: 내장된 GPS나 기지국 정보를 활용
  - 컴퓨터: 와이파이 등을 이용해 대략적인 위치 축측


**와이파이에서 위치 정보를 알아내는 방식**
- 클라이언트는 OS의 API를 이용해 현재 자신이 접근 가능한 액세스 포인트의 BSSID 획득
- BSSID로 서버에 문의해 위도 및 경도 조회


> BSSID는 SSID와는 다른 정보로, 와이파이 기기의 식별자의 48비트 수치   
> 기기마다 독특한 수치로 되어 있으며, 맥 주소와 같은 것


#### 서버가 클라이언트 위치를 추측하는 방법
- 지오 IP라고 불리는 IP 주소로 추측하는 방법
- IP 주소는 지역마다 등록 관리 기관이 있어, 기업이나 프로바이더 등에 IP 주소 할당
- 등록 기관이 정확한 장소까지 관리하는 것은 아님

**지오로케이션 API 장단점**
- 장점: 서비스 제공자 입장에서 사용자의 양해를 구하지 않아도 정보를 구할 수 있음
- 단점: 
  - GPS보다 정확도 떨어짐
  - 클라이언트 입장에선 프록시 등을 이용하지 않는 한 숨길 수 없음

### 5.5 X-Powered-By 헤더

- 서버가 브라우저에 응답할 때 부여하는 헤더 중 하나
- 서버에서 시스템 이름을 반환하는 목적으로 사용
- `X-`가 붙어있기 때문에 RFC 규격에는 없지만 사실상 많은 서버에서 사용하고 있어 표준에 가까움

**보안**
- OS 버전 등 서버 이름 이외의 불필요한 정보가 들어가서는 안됨
- 서버 개발자는 헤더의 표시와 숨김을 전환할 수 있게 구현하는 것이 좋음

### 5.6 원격 프로시저 호출

> 프로시저(prodcedure)란?   
> 각 프로그래밍 언어가 제공하는 함수, 클래스 메서드(정적 메서드)와 같은 것을 뜻함

- 원격 프로시저 호출(RPC, Remote Procedure Call)은 다른 컴퓨터에 있는 기능을 마치 자신의 컴퓨터에 있는 것처럼 호출하고, 필요에 따라 반환 값을 받는 구조


