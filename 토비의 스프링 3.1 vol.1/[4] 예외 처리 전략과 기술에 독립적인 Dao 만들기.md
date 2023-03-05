### 참조

- 토비의 스프링 3.1 vol.1 291p~316p

### 예외처리 전략 - 런타임 예외

체크 예외는 복구할 가능성이 조금이라도 있는 예외적인 상황에 발생하기 때문에 자바는 이를 처리하는 catch 블록이나 throws 선언을 강제한다.

자바의 환경이 서버로 이동하며 이러한 체크 예외의 활용도와 가치는 점점 떨어지고 있다. 잘못하면 의미없는 throws Exception이 붙어있는 메소드들만 만들어낼 뿐이다. 그러므로 **대응이 불가능한 체크 예외라면 빨리 런타임 예외로 전환**해서 던지는 게 낫다.

예시로 이전에 살펴보았던 add() 메소드를 수정해보자.

우선 유저아이디 중복시 반환할 에러인 `DuplicateUserldException` 을 정의해보자.

```java
public class DuplicateUserldException extends RuntimeException {
	public DuplicateUserldException(Throwable cause) {
		super(cause);
	}
}
```

`RuntimeException` 을 상속 받아서 런타임 에러로 선언하였고, 중첩 예외를 만들 수 있도록 생성자를 추가하였다.

그리고 이제 add() 메소드를 `SQLException` 을 직접 던지게 하지 않고 런타임 예외로 전환해서 던지도록 만들면 된다.

```java
public void add() throws DuplicateUserldException {
	try {
		// JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
		// 그런 기능이 있는 다른 SQLException을 던지는 메소드를 호출하는 코드
	}
	catch (SQLException e) {
		if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
			throw new DuplicateUserldException(e); // 예외 전환
		else
			throw new RuntimeException(e); // SQLException을 런타임 예외로 포장
	}
}
```

이로써 add를 호출해서 예외가 발생하면 런타임 에러로 포장, 전환 했으므로 add를 호출하는 곳에서 에러처리를 강요 받지 않고 불필요한 throws 선언을 할 필요는 없으면서, 필요한 경우 아이디 중복 상황을 처리하기 위해 `DuplicateUserldException` 을 이용할 수 있다.

---

### 예외처리 전략 - 애플리케이션 예외

애플리케이션 자체의 로직에 의해 의도적으로 발생시키고, 반드시 catch 해서 무엇인가 조치를 취하도록 요구하는 예외를 애플리케이션 예외라고 한다.

예를 들어 사용자가 요청한 금액을 은행계좌에서 출금하는 기능을 가진 메소드가 있다고 할 때, 당연히 현재 잔고를 확인하고, 허용하는 범위를 넘어서 출금을 요청하면 작업을 중단시키고 사용자에게 경고를 보내야 한다.

이런 기능을 담은 메소드를 설계하는 방법중 하나는 정상적인 흐름을 따르는 코드는 그대로 두고, 잔고 부족과 같은 예외 상황에서는 비즈니스적인 의미를 띤 예외를 던지도록 만드는 것이다.

정상적인 흐름을 따르지만 예외가 발생할 수 있는 코드를 try블록 안에 정리하고, 예외상황에 대한 처리는 catch 블록에 모아둘 수 있어서 코드를 이해하기도 편하다.

이때 사용하는 예외는 체크 예외로 만들어서 잊지 않고 예외상황에 대한 로직을 구현하도록 강제하는게 좋다.

```java
try {
	BigDecimal balance = account.withdraw(amount);
	...
	// 정상적인 처리 결과를 출력하도록 진행
}
catch(InsufficientBalanceException e) { // 체크 예외
	// InsufficientBalanceException에 담긴 인출 가능한 잔고금액 정보를 가져옴
	BigDecimal availFunds = e.getAvailFunds();
	...
	// 잔고 부족 안내 메시지를 준비하고 이를 출력하도록 진행
}
```

출금 메소드를 코드로 나타낸 모습이다.

---

### 예외 전환

예외 전환의 목적은 두 가지가 있었다. 하나는 런타임 예외로 포장해서 굳이 필요하지 않은 catch/throws를 줄여주는 것이고, 다른 하나는 로우레벨의 예외를 좀 더 의미 있고 추상화된 예외로 바꿔서 던져주는 것이다.

### JDBC의 한계

1. 비표준 SQL
    
    대부분의 DB는 각 DB별로 비표준 문법과 기능을 제공한다. 비표준 SQL은 결국 DAO 코드에 들어가고 해당 DAO를 특정 DB에 종속적인 코드로 만든다.
    
2. 호환성 없는 SQLException의 DB 에러정보
    
    DB마다 SQL만 다른 것이 아니라 에러의 종류와 원인도 제각각이다. 그래서 JDBC는 다양한 예외를 SQLException 하나로 던져버리고, 원인은 예외 안의 정보나 에러코드로 확인 해야하는데, 에러 코드는 DB별로 모두 다르다.
    

### 에러코드 매핑을 통한 전환

위에서 나온 JDBC의 한계의 2번에 해당하는 문제를 해결하기 위해서는 DB별 에러 코드를 참고해서 발생한 예외의 원인이 무엇인지 해석해주는 기능을 만드는 것이다.

스프링은 DataAccessException이라는 SQLException을 대체할 수 있는 런타임 예외를 정의하고 있을 뿐 아니라 SQL 문법 때문에 발생히는 에러라면 BadSqlGrammarException을, DB 커넥션을 가져오지 못했을 때는 DataAccessResourceFailureException 을 반환하는 등 다양한 예외 클래스를 제공한다.

문제는 DB별로 에러 코드가 제각각이라는 것인데, 스프링은 DB별 에러코드를 분류해서 스프링이 정의한 예외 클래스와 매핑해놓은 테이블을 만들어 두고 이를 이용한다.

JdbcTemplate은 SQLException을 단지 런타임 예외인 DataAccessException으로 포장하는 것이 아니라 DB의 에러코드를 DataAccessException 계층구조의 클래스중 하나로 매핑해준다.

---

### DAO 인터페이스와 구현의 분리

DAO를 따로 만들어서 사용하는 이유는 데이터 엑세스 로직을 담은 코드를 성격이 다른 코드에서 분리하기 위해서이다. DAO를 사용하는 쪽에서는 DAO가 내부에서 어떤 데이터 엑세스 기술을 사용하는지 신경 쓰지 않아도 된다.

그런데 DAO의 사용 기술과 구현 코드는 감출 수 있지만, 메소드 선언에 나타나는 예외정보가 문제가 된다. UserDao의 인터페이스를 분리해서 기술에 독립적인 인터페이스로 만들려면 아래와 같이 정의해야 한다.

```java
public interface UserDao {
	public void add(User user);
	...
```

하지만 이와 같은 메소드 선언은 불가능하다. DAO에서 사용하는 데이터 엑세스 기술의 API가 예외를 던지기 때문이다. 하지만 JDBC API를 사용하는 UserDao 구현클래스의 add 메소드라면 SQLException을 던질 것이고, JPA라면 PersistentException 를 던지는 등  각 데이터 엑세스 기술의 API는 자신만의 독자적인 예외를 던진다.

결국 **DAO 인터페이스를 기술에 완전히 독립적으로 만들려면 예외가 일치하지 않는 문제도 해결**해야 한다.

스프링은 이를 해결하기 위해 자바의 다양한 데이터 엑세스 기술을 사용할 때 발생하는 예외들을 추상화해서 DataAccessException 계층구조 안에 정리해놓았다.

결국 인터페이스 사용, 런타임 예외 전환과 함께 DataAccessException 예외 추상화를 적용하면 데이터 액세스 기술과 구현 방법에 독립적인 이상적인 DAO를 만들 수가 있다.

---

### 기술에 독립적인 UserDao 만들기

**인터페이스 적용**

인터페이스와 인터페이스 구현 클래스의 이름을 정하는 방법은 여러가지가 있는데, 인터페이스임을 구분하기 위해 이름 앞에 I라는 접두어를 붙히는 방법도 있고, 인터페이스의 이름은 가장 단순하게 하고 구현 클래스는 각각의 특징을 따르는 이름을 붙이는 방법도 있다.

후자의 방법을 사용하여 UserDao 인터페이스의 이름을 UserDao, JDBC를 이용해 구현한 클래스의 이름을 UserDaoJdbc라고 하자.

```java
public interface UserDao {
	void add(User user);
	User get(String id);
	List<User> getAll();
	void deleteAll();
	int getCount();
}
```

위와 같이 인터페이스를 만들었고, 이에 따라 기존 UserDao 클래스는 UserDaoJdbc로 바꿔주고, UserDao 인터페이스를 구현하도록 implements로 선언해줘야 한다.

또한 스프링 설정파일의 userDao 빈 클래스를 UserDaoJdbc로 바꿔줘야 한다.

**테스트 보완**

```java
public class UserDaoTest {
	@Autowired
	private UserDao dao; // UserDaoJdbc로 변경해야 하나?
```

기존 UserDao의 테스트 코드의 UserDao 인스턴스 변수 선언도 변경해야 할까? 굳이 그럴 필요는 없다. @Autowired는 정의된 빈 중에 주입 가능한 빈을 찾아주고, UserDao는 UserDaoJdbc가 구현한 인터페이스이므로 상관없다.

그림 4-4는 UserDao의 인터페이스와 구현을 분리함으로써 데이터 액세스의 구체적 인 기술과 UserDao의 클라이언트 사이에 DI가 적용된 모습을 보여준다.

![image](https://user-images.githubusercontent.com/108508730/199265382-998243a2-cf52-4a5f-b650-3db4c4211230.png)

---

### DataAccessException 활용 시 주의사항

이렇게 스프링을 활용하면 DB 종류나 데이터 액세스 기술에 상관없이 동일항 예외 상황에서 동일한 예외가 발생할 것 같지만 다른 예외가 던져진다. 그 이유는 SQLException에 담긴 DB의 에러 코드를 바로 해석히는 JDBC의 경우와 달리 JPA나 하이버네이트, JDO 등에서는 각 기술 이 재정의한 예외를 가져와 스프링이 최종적으로 DataAccessException으로 변환하는데, DB의 에러 코드와 달리 이런 예외들은 세분화되어 있지 않기 때문이다.

DataAccessException이 기술에 상관없이 어느 정도 추상화된 공통 예외로 변환해주 긴 하지만 근본적인 한계 때문에 완벽하다고 기대할 수는 없다. 따라서 사용에 주의를 기울여야 한다.
