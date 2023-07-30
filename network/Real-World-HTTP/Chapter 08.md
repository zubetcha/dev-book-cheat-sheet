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
