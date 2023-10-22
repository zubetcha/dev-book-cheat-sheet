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

- 타입스크립트가 자바스크립트로 컴파일되는 과정에서 모든 **인터페이스, 타입, 타입 구문은 제거**됨
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

#### 런타임에도 타입을 유지하는 방법

1. 속성 확인

- 타입 체커가 shape의 타입을 Rectangle로 보정해줌

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

2. 태그 기법

- 런타임에 접근 가능한 타입 정보를 명시적으로 저장

```typescript
interface Square {
  kind: 'square';
  width: number;
}

interface Rectangle {
  kind: 'rectangle';
  height: number;
  width: number;
}

// tagged union
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape.kind === 'rectangle') {
    shape; // Regtangle로 타입 추론
    return shape.width * shape.height;
  } else {
    shape; // Square로 타입 추론
    return shape.width * shape.width;
  }
}
```

3. 타입을 클래스로

- 타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법

```typescript
class Square {
  constructor(public width: number) {}
}

class Rectangle extends Square {
  constructor(public width: number, public height: number) {
    super(width);
  }
}

// 클래스를 타입으로 참조
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  // 클래스를 값으로 참조
  if (shape instanceof Rectangle) {
    shape; // Regtangle로 타입 추론
    return shape.width * shape.height;
  } else {
    shape; // Square로 타입 추론
    return shape.width * shape.width;
  }
}
```

### 타입 연산은 런타임에 영향을 주지 않는다.

- ex. string 또는 number 타입의 값을 number 타입으로 정제하여 반환하는 함수

#### 잘못된 함수

```typescript
function asNumber(val: number | string): number {
  return val as number;
}
```

#### 실제 동작

- 타입 구문과 타입 연산은 런타임에 아무런 영향을 주지 않음

```typescript
function asNumber(val) {
  return val;
}
```

#### 수정

- 런타임 타입 체크 및 자바스크립트 연산 필요

```typescript
function asNumber(val: number | string): number {
  return typeof val === 'string' ? Number(val) : val;
}
```

### 런타임 타입은 선언된 타입과 다를 수 있다.

- 런타임에서는 value의 boolean 타입이 제거됨
- API 등 외부에서 받는 값으로 함수를 실행시키는 경우, 받은 값의 타입이 선언된 타입과 다를 수 있음

```typescript
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnLightOn();
      break;
    case false:
      turnLightOff();
      break;
    default:
      console.log('실행되지 않을까봐 걱정입니다.');
  }
}
```

### 타입스크립트 타입으로는 함수를 오버로드할 수 없다.

> 함수 오버로딩  
> 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용하는 것

- 타입스크립트는 타입 오버로딩 기능을 지원하기는 하지만 온전히 타입 수준에서만 동작함
  - 하나의 함수에 대해 복수의 선언문 작성 가능
  - 구현체는 오직 하나뿐

```typescript
function add(a: number, b: number) {
  // error: 중복된 함수 구현
  return a + b;
}

function add(a: string, b: string) {
  // error: 중복된 함수 구현
  return a + b;
}

// 타입 함수 선언문은 여러 개 허용
// 단, 런타임에는 제거됨
function add(a: number, b: number): number; // OK
function add(a: string, b: string): string; // OK

// 런타임
function add(a, b) {
  return a + b;
}
```

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.

- 타입과 타입 연산자는 자바스크립트 컴파일 시점에 제거되기 때문에 런타임의 성능에는 아무런 영향을 주지 않음
- 다만 `런타임` 오버헤드 대신 `빌드타임` 오버헤드는 존재
  - 빌드타임 오버헤드가 큰 경우 빌드 툴에서 트랜스파일만 하도록 선택해 타입 체크는 스킵할 수 있음
- 타입스크립트가 컴파일 하는 코드는 호환성 vs 성능 중 선택해야 하는 문제에 직면할 수 있음
  - 다만 호환성과 성능 사이의 선택은 `컴파일 타깃`과 `언어` 레벨 문제
  - 타입과는 무관

## 4. 구조적 타이핑에 익숙해지기

- 자바스크립트는 본질적으로 덕 타이핑(duck typing) 기반
  - 덕 타이핑: 객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주하는 방식

#### 구조적 타이핑 예시

```typescript
interface Vertor2D {
  x: number;
  y: number;
}

function calclulateLength(v: Vertor2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface NamedVertor {
  name: string;
  x: number;
  y: number;
}

const v: NamedVector = { x: 2, y: 4, name: 'Z33' };
calculateLength(v); // 정상
```

- Vector2D와 NamedVector의 관계를 선언하지 않아도 NamedVertor는 number타입의 x와 y 프로퍼티를 가지고 있기 때문에 caculateLength 함수로 호출 가능
- NamedVector를 위한 별도의 calculateLength를 구현할 필요 또한 없음
- **Vector2D와 NamedVector의 구조가 호환되기 때문** (`구조적 타이핑`)

#### 구조적 타이핑 문제 예시

```typescript
interface Vector3D {
  x: number;
  y: number;
  z: number;
}

// 벡터의 길이를 1로 만드는 정규화 함수
function normalize(v: Vector3D) {
  const length = calculateLength(v);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}

normalize({ x: 3, y: 4, z: 5 }); // { x: 0.6, y: 0.8, z: 1 }
```

- 정규화 함수는 기대값 1보다 큰 1.41 길이를 반환
- calculateLength 함수는 2D 벡터를 기반으로 연산하지만 정규화 함수에서는 3D 벡터 기바으로 연산한 것이 버그
- 그러나 구조적 타이핑의 문제로 타입 체커는 문제를 인식하지 못함
  - 구조적 타이핑 관점에서는 Vector3D에 x와 y가 존재하기 때문에 Vector2D와 호환 가능하다고 인식
- 즉, 함수를 호출할 때 매개변수에 선언되어 있는 속성을 가지고 있고 그 외 다른 속성을 추가적으로 가지고 있어도 호출이 가능함

#### 클래스 할당과 관련된 문제 예시

```typescript
class C {
  foo: string;
  constructor(foo: string) {
    this.foo = foo;
  }
}

const c = new C('instance of C');
const d: C = { foo: 'object literal' }; // 정상
```

- 변수 d는 string 타입의 foo 프로퍼티와 하나의 매개변수로 호출되는 생성자 프로퍼티를 가짐
- 구조적으로 봤을 때에는 인스턴스 C에 필요한 속성과 생성자가 존재하기 때문에 문제가 없는 것으로 인식
- 만약, 생성자 함수에 단순 할당이 아닌 연산 로직이 존재했다면 d는 생성자를 실행시키지 않으므로 문제가 발생

#### 요약

- 자바스크립트는 duck typing 기반이며, 타입스크립트는 이를 모델링하기 위해 구조적 타이핑을 사용함
- 클래스 또한 구조적 타이핑 규칙을 따름

<br/>

## 5. any 타입 지양하기

```typescript
let age: number;
age = '12'; // Error: '12'는 number 타입에 할당할 수 없음
age = '12' as any; // OK
```

#### any 타입에는 타입 안정성이 없다.

- as any 타입 단언을 사용하면 선언한 타입 외의 타입을 할당할 수 있게 됨
- 그러나 타입 체커는 선언에 따라 number 타입으로 판단

```typescript
age += 1; // 런타임에서는 정상이나 값이 "121"이 됨
```

#### any는 함수 시그니처를 무시한다.

- 함수 시그니처: 약속된 타입의 입력을 제공, 약속된 타입의 출력을 반환
- any를 사용하면 위와 같은 약속을 어길 수 있게 됨
- 자바스크립트에서는 암시적으로 타입을 변환할 수도 있어 런타임에서 오류 없이 실행될 수도 있기 때문에 원치 않는 결과를 일으킬 수도 있음
  - ex. string 타입과 number 타입의 연산

#### any 타입에는 언어 서비스가 적용되지 않는다.

- any를 사용하면 자동완성 기능과 도움말 제공을 사용할 수 없음
- 한 번에 이름을 변경해주는 Rename Symbol 기능을 사용할 수 없음

#### any 타입은 코드 리팩토링 때 버그를 감춘다.
