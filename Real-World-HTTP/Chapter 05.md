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
