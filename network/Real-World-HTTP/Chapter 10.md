# Chapter 10 - 보안: 브라우저를 보호하는 HTTP의 기능

- 인터넷과 브라우저가 발달하면서 공격 방식이 다양해짐
- 일반적인 보안 사안이 어떤 매커니즘에서 발생하는지, 어떻게 방지할 수 있는지 등 소개

<br/>

## 10.1 기존의 공격

- 브라우저가 아닌 외부 OS에 접근하는 공격

> 멀웨어(malware)  
> 컴퓨터에 위협을 주는 소프트웨어로, 증식 방법이나 목적에 따라 여러 가지로 분류

<br/>

**증식 방법**

- 컴퓨터 바이러스: 다른 실행 파일 등을 감염시켜 감염된 파일이 실행되면 다른 프로그램에도 자기를 복제해서 증가시킴
- 웜: 네크워크 장비나 OS의 보안 취약점을 공격해 감염

<br/>

**공격 방법**

- OS 부팅을 불가하게 만들거나 속도가 느려지게 하여 파괴
- 프록시 서버를 설정해서 통신 내용 훔쳐보기
- 키 입력을 기록해 암호 훔치기 (키로거)
- 외부에서 원격 조작하기 위해 백도어 설치 (트로이 목마)

<br/>

## 10.2 브라우저를 노리는 공격의 특징

> 세션 토큰 또는 쿠키  
> 쿠키는 브라우저 세션을 유지하는 방법으로 널리 사용되는 기술.  
> 서버와 브라우저의 관계를 고유하게 식별하는 것이 토큰.  
> 그 외 브라우저가 인증되고 사용자 고유의 콘텐츠에 대한 통행 증표로 사용되는 역할로 세션 토큰, 세션 쿠키, 세션 키, 세션 ID, 액세스 토큰 등 다양한 이름이 있지만 대체로 비슷한 뜻으로 사용됨.

<br/>

## 10.3 크로스 사이트 스크립팅

- `XSS`라고도 부름
- 유저가 입력한 값을 검증하지 않고 그대로 출력하는 경우, 입력한 값이 아닌 악의적인 스크립트가 삽입해 브라우저에서 그대로 실행되도록 하는 공격 수단
- 쿠키 정보에 접근하는 스크립트이거나, 유저가 입력한 정보를 가로채는 스크립트이면 특히 위험
- 서버 측 방어로는 1) 입력 시점 검증, 2) 출력 시점 검증 등이 있음

### 10.3.1 유출 방지 쿠키의 설정

- `httpOnly` 속성 설정
- 자바스크립트에서 쿠키에 접근할 수 없게됨

### 10.3.2 X-XSS-Protection 헤더

- HTML 인라인에서 스크립트 태그를 사용하는 경우 등 수상한 패턴 감지
- `X-`가 붙어있기 때문에 비공식 헤더이지만 IE, 크롬, 사파리 등의 브라우저가 지원하며 파이어폭스는 미지원

```
X-XSS-Protection: 1; mode=block
```

### 10.3.3 Content-Security-Policy 헤더

- 웹사이트에서 사용할 수 있는 기능을 세밀하게 on/off 할 수 있는 기술로, W3C에서 정의
- 웹사이트에 필요한 기능을 서버에서 설정하여 XSS처럼 자바스크립트가 예상치 못하게 동작하는 것을 제한

#### HTML에서 로드할 수 있는 리소스 관련

**리소스 파일 사용 권한 설정 지시문**

- base-uri: 도큐먼트의 base URI (상대 경로의 시작점)
- child-src: `Web Worker`, `frame`, `iframe`으로 이용할 수 있는 URL
- connect-src: `XMLHttpRequest`, `WebSocket`, `EventSource`와 같은 자바스크립트로 연결할 출처
- font-src: CSS의 `@font-face`에서 로드할 웹 폰트
- img-src: `이미지`와 `파비콘`을 로드할 출처
- manifest-src: `매니페스트`를 로드할 출처
- media-src: `audio`와 `video`를 제공하는 출처
- object-src: 플래시나 자바 애플릿 등 기타 `플러그인`에 대한 제어
- script-src: `자바스크립트`를 로드할 출처
- style-src: 읽기 가능한 `스타일시트`를 로드할 출처

**지시문에 대해 설정할 수 있는 속성**

- none: 리소스 로드 금지
- self: 같은 출처 지정
- unsafe-inline: 인라인 스크립트 태그, 인라인 스타일 태그 등을 허가하 - `XSS 위험`
- unsafe-eval: 문자열을 자바스크립트로서 실행하는 eval(), new Function(), setTimeout() 등의 실행 허가 - `XSS 위험`
- data:: data:URI 허가
- mmediastream:: mediastream:URI 허가
- blob:: blob:URL 허가
- filesystem: filesystem:URI 허가

#### 리소스 이외의 설정 지시문

**일괄 보안 설정**

- default-src: 리소스의 접근 범위 일괄 설정 (개별 설정이 우선함)
- sandbox: 팝업, 폼 등의 허용 설정
- upgrade-insecure-requests: HTTP 통신을 모두 HTTPS로 변경

**기타**

- referer: 리퍼러의 동작 변경
- report-uri: 브라우저에서 위반 사항을 탐지하면 지정된 서버로 JSON 포맷으로 전송
- reflected-xss: 반사형 크로스 사이트 스크립팅으로 불리는 공격에 대한 필터 활성화

#### Content-Security-Policy-Report-Only

- `Content-Security-Policy` 헤더는 XSS의 위협을 막아주지만, 종종 웹사이트의 동작을 멈추게 할 수도 있음
- `Content-Security-Policy-Report-Only` 헤더를 사용하면 검사는 하지만 동작은 멈추지 않음

### 10.3.4 Content-Security-Policy와 자바스크립트 템플릿 엔진

- 일부 자바스크립트 템플릿 엔진은 `new Function`을 사용하여 동적으로 함수를 생성함
- 동적 함수 생성은 CSP의 `unsafe-eval`에 해당하므로 제대로 동작하지 않을 수도 있음
- 사용하는 템플릿 엔진이 CSP를 지원하는지 확인해 볼 필요 O

### 10.3.5 Mixed Content

> `Mixed Content`  
> 광고나 외부 서비스가 제공하는 컨텐츠 등으로 인해 HTTPS와 HTTP가 뒤섞이는 경우를 뜻하며, 이런 경우 브라우저는 경고나 오류를 표시함

- 근본적으로 해결하는 방법은 모두 HTTPS로 대체하는 것
- 그러나 `Content-Security-Policy` 헤더에 `ungrade-insecure-request` 지시문을 설정하여 해결 가능
  - 해당 지시문을 설정하면 `http://` 로 시작하더라도 `https://` 로 적혀 있는 것처럼 취득함

```
// 헤더 설정
Content-Security-Policy: upgrade-insecure-request

// meta 태그
<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-request"/>

// mixed-content를 오류로 처리
Content-Security-Policy: block-all-mixed-content
```

### 10.3.6 교차 출처 리소스 공유

> 교차 출처 리소스 공유 (cross-origin resource sharing, CORS)  
> 오리진(출처, 도메인) 사이에 리소스를 공유하는 방법으로, W3C에서 규격화

- 클라이언트에서 서버로 접근하기 직전까지의 권한 확인 프로토콜로, 보안을 위한 기능
- 이 때, 보호 대상은 클라이언트가 아닌 API를 제공하는 서버
- 리소스 공유는 `XMLHttpRequest`나 `Fetch API`에 의한 것을 뜻함
- `Content-Security-Policy` 헤더에 `conntect-src:` 지시문을 설정할 수도 있으나 더 엄격하게 제한하기 위함


