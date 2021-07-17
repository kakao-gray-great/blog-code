
Transaction에 대해 누군가 물어보면 **"데이터베이스의 상태를 변화시키는 하나의 논리적 기능을 수행하기 위한 작업의 단위"** 라고 기계처럼 설명했다. 이 기계적인 답변에서 벗어나기 위해 transaction에 대해 공부해보겠다.

## 1. Transaction 맛보기

옷을 구매한 후, 판매자에 3만원을 계좌이체를 해야 한다고 생각해보자.

우리는 ATM기에 가서 판매자에게 돈을 송금할 것이다.

1. 나의 계좌에서 3만원이 출금된다.

UPDATE account SET balance = balance - 30000 WHERE user="park";

2. 판매자의 계좌에 3만원이 입금된다.

UPDATE account SET balance = balance + 30000 WHERE user="판매자";

이러한 상황에 어떠한 오류가 발생할 수 있을까?

* 나의 계좌에서 돈이 출금된 뒤, DB가 다운된다.
* 구매자의 계좌에서 돈이 출금되지 않았는데, 판매자에게 돈이 입금된다.
* 출금도 입금도 되지 않는다.

오류가 발생하면 올바른 계좌이체가 됐다고 볼 수 없기 때문에 전부 원래 상태로 돌려주어야 하는데, 이것을 transaction이 해준다.

이처럼 하나의 기능을 수행하기 위한 단위를 transaction이라고 말한다.


## 2. Transaction이란?

**여러 쿼리를 논리적인 하나의 작업으로 묶어주는 것**으로 생각하자.

즉, 쿼리들이 한 번에 모두 실행되거나 아무 쿼리도 실행되지 않도록 하여 사용자 혹은 시스템상의 문제가 있더라도 데이터베이스가 데이터를 안정적으로 보장할 수 있도록 한다.


### 3. Transaction의 성질

### 3-1. ACID

Transaction이 안전하게 수행된다는 것을 보장하기 위한 성질이다.

### 3-2. Atomicity(원자성)

Transaction은 DB에 모두 반영되거나, 전혀 반영되지 않아야 한다.
커밋되지 않은 transaction의 중간 상태 DB에 반영해서는 안 된다.

### 3-3. Consistency(일관성)

Transaction 처리 결과는 항상 일관성 있어야 한다.
즉, DB는 항상 일관된 상태로 유지되어야 한다.

### 3-4. Isolation(독립성)

둘 이상의 transaction이 동시 실행되고 있을 때, 각각의 transaction은 독립적으로 이루어져 어떤 트랜잭션도 다른 transaction 연산에 끼어들 수 없다.

### 3-5. Durability(지속성)

Transaction이 성공적으로 완료되었으면 결과는 영구히 반영되어야 한다.

### 3-6. 하지만... 

ACID를 전부 보장하려고 하면 문제가 생길 수 있다. ACID는 transaction이 이론적으로 보장해야 하는 성질이며 실제로 성능 보장이 완화되기도 한다.

우리는 Spring에서 다양한 옵션을 통해 transaction의 문제들을 방지할 수 있다.


## 4. Transaction Problem 

다수의 transaction이 접근할 때, 발생할 수 있는 문제들을 살펴보자.

### 4-1. Dirty Read

![](https://i.imgur.com/wWiKFPL.png)

transaction A가 transaction을 끝마치지 못하고 rollback 한다면 transaction B는 무효가 된 데이터 값을 조회하고 처리하기 때문에 문제가 발생한다.

> 우리 개발 환경에서는 해당 Transaction 수준을 허용하지 않는다.

### 4-2. Non-Repeatable Read

동일한 transaction 내에서 select 문을 두 번 조회했는데 데이터가 다르게 조회되는 데이터 불일치 문제이다.

### 4-3. Phantom Read

Non-repeatable-read의 한 종류이다.

하나의 transaction 내에서 동일한 쿼리를 조회했을 때, 데이터가 생기거나 없어져 있는 현상이다.


## 5. Transaction Isolation (격리 수준)

Isolation은 동시에 DB에 접근할 때 그 접근을 어떻게 제어할지에 대한 설정이다.

![](https://i.imgur.com/3Kjjz9Z.png)

READ-UNCOMMITTED에서 SERIALIZABLE로 내려갈수록 isolation 수준이 높아지지만, 성능이 떨어진다.

### 5-1. READ-UNCOMMITTED

Isolation 수준이 가장 낮은 속성이다.
Commit 전의 transaction의 데이터 변경 내용을 다른 transaction이 읽는 것 허용한다.

![](https://i.imgur.com/AajehI4.png)

Transaction A가 아직 commit 되지 않은 상태에서 transaction B가 DB를 조회했는데, update를 통해 변경되고 commit 되지 않은 데이터를 조회하는 것을 볼 수 있다.

READ-UNCOMMITTED에서는 **dirty read**가 발생할 수 있다.

### 5-2. READ-COMMITTED

Dirty Read를 방지한 속성으로, commit이 완료된 transaction의 데이터만 다른 transaction에서 조회할 수 있다.

![](https://i.imgur.com/XFZ6jDY.png)

READ-COMMITTED에서는 **NON-REPEATABLE READ**가 발생할 수 있다.

### 5-3. REPEATABLE-READ

하나의 transaction 내에서 조회한 내용이 항상 동일함을 보장한다.

![](https://i.imgur.com/WoQHtng.png)

> READ-COMMITTED와 다른 점은 한 transaction이 조회한 데이터는 transaction이 종료될 때까지 다른 transaction이 변경하거나 삭제하는 것을 막는다. 그렇기에 한번 조회한 데이터는 반복적으로 조회해도 같은 값을 반환한다.

하지만 여기서도 **Phantom Read** 문제가 발생한다.

### 5-4. SERIALIZABLE

하나의 transaction에서 사용하는 데이터를 다른 transaction에서 접근 불가하게 한다.

Transaction의 ACID 성질이 잘 지켜지나 성능은 가장 떨어진다.

### 5-5. 사용 예

```java=
@Transactional(isolation=Isolation.DEFAULT)
public void service() {
    // ...
}
```


## 6. Transaction Propagation (전파)

Transaction 동작 도중 다른 Transaction을 호출하는 상황에 대한 설정이다.

![](https://i.imgur.com/tRG4S7N.png)

### 6-1. 사용 예
```java=
@Transactional(propagation = Propagation.REQUIRED)
public void example() {
    // ...
}
```


## 7. Transaction readOnly 

Transaction을 읽기 전용으로 설정할 수 있다.

성능을 최적화하기 위해서 사용할 수 있으며 특정 transaction 작업 안에서 write 되는 것을 막아줄 수 있다. **'readOnly=true'** 옵션이 적용되면 INSERT, UPDATE, DELETE 같은 작업 시 예외가 발생한다.

### 7-1. 사용 예

```java=
@Transactional(readOnly=true)
public void findById(Long id) {
    // ...
}
```


### 참고

https://www.youtube.com/watch?v=ImvYNlF_saE
https://www.youtube.com/watch?v=e9PC0sroCzc
https://jeong-pro.tistory.com/228
https://akasai.space/what-is-isolation-level
https://nesoy.github.io/articles/2019-05/Database-Transaction-
https://goddaehee.tistory.com/167
