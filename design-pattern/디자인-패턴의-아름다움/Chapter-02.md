# Chapter 02 객체지향 프로그래밍 패러다임

## 2.2 캡슐화, 추상화, 상속, 다형성이 등장한 이유

### 2.2.1 캡슐화

> **캡슐화**  
> 정보 은닉 또는 데이터 액세스 보호  
> 접근 가능한 인터페이스를 제한하여 클래스가 제공하는 메서드를 통해서만 내부 정보나 데이터에 대한 외부 접근을 허용하는 것

<br/>

```typescript
class Wallet {
  id: number;
  createTime: string;
  balance: number;
  balanceLastModifiedTime: string;

  getId() {
    return this.id;
  }

  getCreateTime() {
    return this.createTime;
  }

  getBalance() {
    return this.balance;
  }

  getBalanceLastModifiedTime() {
    return this.balanceLastModifiedTime;
  }

  increaseBalacne(increaseAmount) {
    this.balance = this.balance + increaseAmount;
  }

  decreaseBalance(decreaseAmount) {
    this.balance = this.balance - decreaseAmount;
  }
}
```

- 데이터에 접근하거나 데이터를 변경하는 건 클래서의 메서드를 통해서만 가능
