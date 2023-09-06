# Chapter 01 타입스크립트 알아보기

## 1. 타입스크립트와 자바스크립트의 관계 이해하기

- 타입스크립트는 문법적으로도 자바스크립트의 상위집합
  - 자바스크립트에 문법 오류가 없다면 유효한 타입스크립트 프로그램이라고 할 수 있음
- 타입스크립트 컴파일러는 타입 구문이 없어도 `타입 추론`을 통해 문제를 발견할 수 있음

```javascript
let city = 'seoul';
console.log(city.toUppercase());
//               ~~~~~~~~~~~
```

- 오류를 발생시키지는 않지만 의도대로 동작하지 않는 코드도 있기 때문에 명시적으로 타입을 선언해 의도를 분명하게 할 수 있음

```javascript
const profiles = [
  { name: '정주혜', nickname: 'zubetcha' },
  { name: '임찬수', nickname: 'batt' },
  // ...
];

for (const profile of profiles) {
  console.log(profile.nicknane); // undefined
}
```

- 타입스크립트는 자바스크립트의 런타임 동작을 모델링하는 타입 시스템을 가지고 있음
  - 따라서 런타임 에러를 발생시키는 코드를 찾아내려고 함
  - 그러나 작성한 타입이 타입 체크를 통과하더라도 런타임 에러가 발생할 수 있음

```javascript
const names = ['Alice', 'Bob'];
console.log(names[2].toUpperCase());
//                   ~~~~~~~~~~~
```

- 즉, 타입 시스템은 정적 타입의 정확성을 완전히 보장하지는 않음

<br/>

## 2. 타입스크립트 설정 이해하기

`noImplicitAny`

- 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어
- 아래 코드는 noImplicitAny가 설정되어 있지 않으면 매개변수와 반환값은 모두 암시적으로 `any`로 추론됨
- 그러나 noImplicitAny를 true로 설정하면 오류가 발생함

```javascript
function sum(a, b) {
  return a + b;
}
```

`strictNullChecks`

- `null`과 `undefined`가 모든 타입에서 허용되는지 확인
- 런타임 에러를 방지하기 위해 설정하는 것이 좋음
- 아래 코드는 strictNullChecks를 해제하면 유효하며, true로 설정하면 오류가 발생함

```javascript
const x: number = null;
```

- strictNullChecks를 true로 설정했을 때에는 명시적으로 타입을 설정해야 함

```javascript
const x: number | null = null;
```

<br/>

## 3. 코드 생성과 타입이 관계없음을 이해하기

> 타입스크립트 컴파일러의 역할
>
> - 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일
> - 코드의 타입 오류 체크

### 타입 오류가 있는 코드도 컴파일 가능하다.

- 컴파일은 타입 체크와 독립적으로 동작 > 타입 오류가 있는 코드도 컴파일 가능
- 문제가 될 만한 부분을 알려주긴 하지만 빌드를 멈추지는 않음
- 오류가 있을 때 컴파일하지 않으려면 tsconfig에 `noEmitOnError` 설정

### 런타임에는 타입 체크가 불가능하다.

- 타입스크립트가 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 제거됨
- 타입을 명확하기 하게 위해서는 런타임에도 타입 정보를 유지하는 방법이 필요

```typescript
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  // ts error: Rectangle은 타입인데 값으로 참조되고 있음
  if (shape instanceof Rectangle) {
    // ts error: Shape 타입에는 height 속성이 없음
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

```typescript
function calculateArea(shape: Shape) {
  if ('height' in shape) {
    shape; // Regtangle로 타입 추론
    return shape.width * shape.height;
  } else {
    shape; // Square로 타입 추론
    return shape.width * shape.width;
  }
}
```
