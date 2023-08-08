# Chapter 01 타입스크립트 알아보기

## 아이템 1 타입스크립트와 자바스크립트의 관계 이해하기

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

## 아이템 2 타입스크립트 설정 이해하기

`noImplicitAny`

- 변수들이 미리 정의된 타입으 가져야 하는지 여부를 제어
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

## 아이템 3 코드 생성과 타입이 관계없음을 이해하기

> 타입스크립트 컴파일러의 역할
>
> - 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일
> - 코드의 타입 오류 체크

### 타입 오류가 있는 코드도 컴파일 가능
