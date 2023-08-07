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
