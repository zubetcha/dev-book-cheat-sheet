# Chapter 08 - HTTP/2의 시맨틱스: 새로운 활용 사례

- 검색 엔진과 소셜 미디어가 이해하는 프로토콜을 중심으로 소개하는 챕터

## 8.1 반응형 디자인

CSS 픽셀

- 브라우저는 `논리 해상도(논리적 스크린 크기)`의 디스플레이가 연결되어 있다고 인식하고 표시
  - 브라우저의 논리 해상도: `CSS 픽셀`
  - 논리 해상도와 물리 해상도의 비율: 기기 픽셀 비율
- 반응형 디자인은 브라우저가 아닌 사이트에서 콘텐츠 표시를 제어하므로, 브라우저에서 확대 및 축소를 하지 않도록 설정해야 함

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

> 논리 픽셀: n개의 픽셀이 1개의 픽셀처럼 보이게 함  
> 물리 픽셀: 실제 픽셀 개수

브라우저가 기기 픽셀 비율에 가까운 이미지를 선택하도록 하는 방법

- srcset 속성 사용
- css에서 image-set() 함수 사용

`srcset`

```html
<img src="photo.png" srcset="photo@2x.png 2x, photo@3x.png 3x" />
```

`image-set()`

- image-set 함수 인자: `[ <image> | <string> ] [ <resolution> || type( <string> ) ]`
- image: 아무 이미지나 가능. 단 image-set() 안에 또다른 image-set() 중첩은 불가능
- string: 이미지 주소 url
- resolution: x, dppx, dpi, dpcm
- type: 유효한 MIME 타입

```css
image-set(
  "image1.jpg" 1x,
  "image2.jpg" 2x
);

image-set(
  url("image1.jpg") 1x,
  url("image2.jpg") 2x
);

```

<br/>

## 8.2 시맨틱 웹

- 단순히 텍스트나 문서가 아니라 웹 안에서 의미를 다뤄 가능성 및 접근성을 제고하고자 하는..
- 의미를 다룬다는 건 페이지에 포함된 정보를 분석해서 정보를 집약하거나 검색하는 과정을 인간이 하지 않게 하는 것을 뜻함

### 8.2.1 RDF

- Resource Description Framework
- URI로 식별되는 엔티티 간의 관계성 기술
- XML 등을 이용해 의미를 기술하려는 접근에서 출발
- 공통 스키마 같은 것은 없으며 서비스별로 스키마를 생성함

시맨틱 웹 키워드

- 주어: 설명할 대상 요소
- 목적어: 주어의 속성
- 술어: 그 관계

### 8.2.2 더블린 코어

> 더블린 코어란,  
> RFC 5013과 ISO 등에서 정의된 메타데이터 집합

- 서적, 음악 등 저작물에 범용적으로 사용할 수 있는 메타데이터로 구성
- 대표적으로 .opf(OPF, Open eBook Package Format) 파일에 `dc:`로 시작하는 태그가 더블린 코어

### 8.2.3 RSS

- RDF의 응용 사례로, `RDF Site Summary`의 약자
- 웹사이트의 업데이트 기록 요약에 관한 배급(?) 형식
- 블로그나 웹사이트를 업데이트하면 블로그 엔진과 콘텐츠 관리 시스템이 자동으로 업데이트 함
- RSS 파일에는 새로운 순서대로 신규 컨텐츠의 요약이 저장됨
- RSS 리더는 설치해서 사용할 수도 있으며, 브라우저에 내장되어 있거나 익스텐션으로도 사용 가능

### 8.2.4 마이크로포맷

- HTML 태그와 클래스를 사용해 표현
- 클래스를 사용하기 때문에 CSS 클래스 이름과 충돌할 수 있음
- 공유 스키마 카탈로그화
- 검색 엔진 지원

```html
<div class="vcard">
  <a class="url fn" href="">정주혜</a>
  <span class="nickname">zu</span>
  <time class="bday">1994-07-04</time>
</div>
```

### 8.2.5 마이크로데이터

- HTML에 삽입 가능한 시맨틱 표현 형식
- 공유 스키마 카탈로그화
- 검색 엔진 지원
- 마이크로포맷과 달리 기존 HTML 속성과 충돌하지 않도 고유 속성 사용
  - itemscope, itemtype, itemprop
- 지원 어휘(스키마)
  - Event
  - Organization
  - Person
  - Product
  - Review
  - Review-aggregate
  - Breadcrumb
  - Offer
  - Offer-aggregate

```html
<div itemscope itemtype="http://">
  <span itemprop="name">정주혜</span>
</div>
```

### 8.2.6 RDF의 역습

시맨틱 웹 작성 포맷

| 이름    | 작성법    | 설명                                              | 변형                                  |
| :------ | :-------- | :------------------------------------------------ | :------------------------------------ |
| RDF/XML | XML 작성  | 기본 포맷                                         | RDF 1.0 XML, RDF 1.1 XML              |
| Turtle  | 고유 방식 | 작성하기 쉬운 일반 텍스트에 가까운 형식           | N-Triples, TriG, N-Quads              |
| JSON-LD | JSON      |                                                   | JSON으로 작성해, `script` 태그로 삽입 |
| RDFa    | HTML 속성 | HTML로서 인간도 바로 브라우저로 읽을 수 있는 표현 | RDFa, RDFa Lite, HTML/RDFa            |

<br/>

`JSON-LD`

- 향후 널리 사용될 것으로 예상되는 포맷
- HTML이 아니기 때문에 본문에 삽입할 수는 없지만 `script` 태그 안에 기술 가능
- 검색 엔진이 페이지와 관련된 메타데이터로 다루며, 검색 결과에도 표시됨

```html
<script>
  {
    "@context": "https://schema.org/",
    "@type": "Recipe",
    "name": "Party Coffee Cake",
    "author": {
      "@type": "Person",
      "name": "Mary Stone"
    },
    "datePublished": "2018-03-10",
    "description": "This coffee cake is awesome and perfect for parties.",
    "prepTime": "PT20M"
  }
</script>
```

<br/>

## 8.3 오픈 그래프 프로토콜

- 검색 엔진이 아닌 소셜 네트워크에서 사용되는 메타데이터
- 페이스북이 개발
- 비슷한 데이터 구조로는 트위터 카드가 있으며, 트위터 카드는 오픈 그래프 프로토콜을 바탕으로 개발됨
  - 트위터 카드에서 정보를 찾을 수 없을 때, 오픈 그래프 프로토콜 속성을 대신 참조하기 때문에 중복 정보를 두 가지 속성에 모두 작성할 필요는 없음
  - `twitter:` 가 붙으면 트위터 전용 속성

`link-rel canonical`

`자주 사용되는 기본 요소`

|      요소      |           설명            |
| :------------: | :-----------------------: |
|    og:title    |          타이틀           |
|    og:type     |           종류            |
|     og:url     |            URL            |
|    og:image    |          이미지           |
| og:description | 인용 페이지에 붙일 텍스트 |

`옵션 요소`

|        요소         |                 설명                 |
| :-----------------: | :----------------------------------: |
|      og:audio       |              음성 파일               |
|    og:determiner    |          a, the 등의 관심사          |
|      og:locale      | 페이지의 콘텐츠가 대상으로 하는 언어 |
| og:locale:alternate |    이 페이지가 제공하는 다른 언어    |
|    og:site_name     |             사이트 이름              |
|      og:video       |             동영상 파일              |

<br/>

## 8.4 AMP

- Accelerated Mobile Pages
- 휴대 기기에서 웹 페이지의 로딩 속도를 빠르게 하는 방법
- 사용자가 AMP 마크를 보고 로딩이 빠른 페이지를 선택할 수 있음
- 정적 웹 컨텐츠에 적합 (ex. 뉴스, 요리법, 영화 목록, 제품 페이지, 동영상, 블로그 등)
  - 인터랙션이 중요한 앱에서는 그닥 효과가 없음
- 정해져 있는 전용 태그만 사용할 수 있음
  - amp-img, amg-video
- AMP가 빠른 이유
  - 크롤러가 AMP이거나 link 태그에 AMP 페이지가 있는 컨텐츠를 발견하면, 모든 내용을 구글 전용 캐시 서버에 복사함

<br/>

AMP의 고속화 기법

- 페이지 구성을 고정시켜서 로딩 속도 제고
- CDN 지원 용이
- CDN 업로드 -> 자바스크립트 파일까지 캐싱
- 서버 업로드 -> 태그를 수정해 고속화
