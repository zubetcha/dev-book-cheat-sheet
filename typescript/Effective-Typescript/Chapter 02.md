# Chpater 02 타입스크립트의 타입 시스템

## 1. 편집기를 사용하여 타입 시스템 탐색하기

- 편집기에서 타입 시스템이 어떻게 타입을 추론하는지 살펴볼 수 있음
  - 숫자, 함수, 객체의 프로퍼티, 조건문 분기 안에서의 값 등
- 에디터에서 Go to Definition 기능을 통해 라이브러리 또는 라이브러리 타입 선언 탐색 가능

  - 타입스크립트 선언 파일을 찾다보면 어떻게 동작을 모델링하는지 알 수 있음

<br/>

### 에디터상의 타입 오류

```tsx
function getElement(elOrId: string | HTMLElement | null): HTMLElement {
	if (typeof elOrId === 'object') {
		return elOrId; // 타입에러 1
	} else if (elOrId === null) {
		return document.body;
	} else {
		const el = document.getElementById(elOrId);
		return el // 타입에러 2
}
```

- 타입에러 1: 타입스크립트에서 null의 타입도 `object`이므로, null은 함수의 반환 타입인 HTMLElement에 할당이 안 되므로 타입 오류 발생
- 타입에러 2: getElementById 함수의 반환값의 타입은 `HTMLElement | null`이므로, 마찬가지로 null은 함수의 반환 타입인 HTMLElement에 할당이 안 되므로 타입 오류 발생

<br/>

## 2. 타입이 값들의 집합이라고 생각하기

- 타입은 할당 가능한 값들의 집합 혹은 범위
  - ex. `number` 타입은 모든 숫자값의 집합
- null과 undefined는 strictNullChecks 여부에 따라 number에 해당될 수도, 아닐 수도 있음
- 가장 작은 집합 → 아무 값도 포함되지 않는 공집합 → 타입스크립트에서는 `never` 타입
  - never 타입으로 선언된 변수의 범위는 공집합이기 때문에 아무런 값도 할당할 수 없음

```tsx
const x: never = 12;
// 타입에러: never 타입에 12 할당 불가
```

- 그 다음으로 작은 집합: `유닛(unit) 타입` 또는 `리터럴(literal) 타입`

```tsx
type A = 'A';
type Two = 2;
```

- 유닛 혹은 리터럴 타입을 2개 이상으로 묶은 타입 → `유니온(union) 타입`
  - 값 집합들의 **합집합**

```tsx
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;
```

### 할당 가능 여부 판단

- 할당하려는 값의 타입이 타입 집합의 `원소` 혹은 `부분 집합`인지 확인
- 원소 혹은 부분 집합이 아닌 경우 할당 불가

```tsx
type a: AB = 'A'; // 정상
type c: AB = 'C'; // 타입 에러
```
