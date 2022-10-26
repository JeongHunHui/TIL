### 참조링크

- 트랜잭션 이란?
    
    [[Spring] Spring의 트랜잭션 관리 (feat: @Transactional)](https://yeonyeon.tistory.com/223)
    

### 트랜잭션이란?

> DB상태를 변환시키는 하나의 논리적 기능을 수행하기 위한 작업의 단위 또는 한꺼번에 모두 수행되어야 할 일련의 연산들
> 

### 트랜잭션의 예시

온라인 쇼핑물에서 물건을 사면 아래의 작업들이 이루어진다.

1. 판매처에 돈 보내기
2. 판매처에 돈 받기

하지만, 둘중 하나만 성공하는 경우가 발생해서는 안된다.

→ 모든 작업이 성공적으로 완료되어야 작업 결과를 적용(commit)하고, 작업 중에 어느 한 곳에서라도 오류가 발생하면 작업 실행 전의 상태로 돌아가야하는 것(rollback)이 트랜잭션의 개념이다.

### 트랜잭션 관리의 종류

**비즈니스 로직과 트랜잭션 코드의 분리**

비즈니스 로직과 트랜잭션 처리 로직이 동시에 존재하면 코드의 중복이 생길 수 있고, 비즈니스 로직에만 집중하기 어렵다.

이를 해결하기위한 2가지 트랜잭션 기술

1. Transaction Template
    
    데이터 접근 기술에는 JdbcTemplate, JPA등 여러가지가 존재한다. 이 기술이 바뀌면 트랜잭션을 사용하는 방법도 달라진다. Spring에서는 이러한 상황을 고려해 PlatformTransactionManager 인터페이스를 만들었다. 해당 인터페이스는 3가지 메서드를 제공한다.
    
    - getTransaction(): 현재 TransactionStatus를 return
    - commit(): 변경 내역 커밋
    - rollback(): 변경 내역 롤백
2. @Transactional 어노테이션
    
    DB와 관련된, 트랜잭션이 필요한 클래스 혹은 메서드에 @Transactional 어노테이션을 달면 된다. (클래스에 달면 해당 클래스 및 하위 클래스까지 적용됨)
    

### @Transactional 예제

```java
@Transactional
public void buyProduct(Long money) { 
    userWallet.loseMoney(money);
    sellerWallet.gainMoney(money);
}
```

위의 코드는 loseMoney 또는 gainMoney 둘 중 하나라도 실패하면 전체 작업을 취소한다.

모든 작업이 성공할 경우 DB에 해당 변경 내역이 반영된다.

readOnly 속성(기본 false)

- 트랜잭션을 읽기 전용으로 설정
- 성능 최적화, 특정 트랜잭션에서 쓰기 작업이 일어나는 것을 방지

### 테스트와 @Transactional

테스트 환경에서는 @Transactional이 있으면 결과에 상관없이 테스트 메서드 종료후 롤백된다.
