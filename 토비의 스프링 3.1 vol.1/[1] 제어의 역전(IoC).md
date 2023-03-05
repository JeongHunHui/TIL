# 제어의 역전(IoC)

### 참조

- 토비의 스프링 3.1 vol.1 88p~103p

지금까지 UserDao를 리팩토링하는 작업을 했지만, UserDaoTest는 어떤 ConnectionMaker 구현 클래스를 사용할지를 결정하는 기능까지 맡게 되었다.

UserDaoTest는 원래 관심사였던 UserDao의 기능 테스트외에 다른 책임을 맡게 되었으니 UserDao와 ConnectionMaker 구현 클래스의 오브젝트를 만드는 기능과, 그렇게 만들어진 두 개의 오브젝트가 연결되어서 사용될 수 있도록 관계를 맺어주는 기능을 분리해보자.

**팩토리**

팩토리는 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 돌려주는 오브젝트를 말한다.

DaoFactory

```java
public class DaoFactory {
	public UserDao userDao() {
		ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao userDao = new UserDao(connectionMaker);
		return userDao
	}
}
```

팩토리를 사용하도록 UserDaoTest를 수정해보면

```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		UserDao dao = new DaoFactory().userDao();
		// ...
	}
}
```

이렇게 되고, 이리하여 UserDaoTest에서 어떤 ConnectionMaker 구현 클래스를 사용할지를 결정하는 기능을 DaoFactory를 통해 분리하였다.

```java
public class DaoFactory {
	public UserDao userDao() {
		return new UserDao(new DConnectionMaker());
	}
	public AccountDao accountDao() {
		return new AccountDao(new DConnectionMaker());
	}
	public MessageDao messageDao() {
		return new MessageDao(new DConnectionMaker());
	}
}
```

하지만 만약 위와 같이 DaoFactory에서 UserDao만이 아니라 AccountDao, MessageDao등 여러 DAO를 생성한다면 어떤 ConnectionMaker 구현 클래스를 사용할지를 결정하는 기능이 중복돼서 나타난다.

```java
public class DaoFactory {
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}
	public AccountDao accountDao() {
		return new AccountDao(connectionMaker());
	}
	public MessageDao messageDao() {
		return new MessageDao(connectionMaker());
	}
	public ConnectionMaker connectionMaker() {
		return new DConnectionMaker();
	}
}
```

위와 같이 어떤 ConnectionMaker 구현 클래스를 사용할지를 결정하는 기능을 분리하면 connectionMaker만 바꾸면 일일히 수정할 필요가 없다.

---

### 제어의 역전(IoC)

UserDao의 초기 버전을 보면, 자신이 사용할 클래스를 결정하고, 언제 어떻게 그 오브젝트를 만들지를 스스로 결정한다. 즉 모든 종류의 작업을 사용하는 쪽에서 제어하는 구조이다.

제어의 역전이란 이런 제어 흐름의 개념을 뒤집는 것이다.

제어의 역전에서는 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지도, 생성하지도 않는다. 모든 제어 권한을 다른 대상에게 위임하기 때문이다.

프레임워크도 제어의 역전 개념이 적용된 대표적인 기술이다.

라이브러리를 사용하는 애플리케이션 코드는 애플리케이션 흐름을 직접 제어한다. 반면에 프레임워크는 애플리케이션 코드가 프레임워크에 의해 사용된다.

보통 프레임워크 위에 개발한 클래스를 등록해두고, 프레임워크가 흐름을 주도하는 중에 개발자가 만든 애플리케이션 코드를 사용하도록 만드는 방식이다.

---

### 스프링의 IoC

스프링에서는 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트를 빈(bean)이라고 부른다. 동시에 스프링 빈은 스프링 컨테이너가 생성과 관계설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트를 가리키는 말이다.

스프링에서는 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트를 빈 팩토리(Bean Factory)라고 부른다. 보통은 빈 팩토리보다는 이를 좀더 확장한 애플리케이션 컨텍스트를 주로 사용한다. (그냥 빈 팩토리 = 애플리케이션 컨텍스트라고 생각해도된다.)

애플리케이션 컨텍스트는 별도의 설정정보를 참고해서 빈의 생성, 관계설정 등의 제어 작업을 총괄한다. 즉 애플리케이션 컨텍스트는 어떤 클래스의 오브젝트를 생성하고 어디서 사용하도록 연결해줄 것인가 등의 설정정보를 담고있는 무언가를 가져와서 활용하는 범용적인 IoC엔진 같은 것이라고 볼 수 있다.

→ 애플리케이션은 애플리케이션 컨텍스트와 그 설정정보를 따라서 만들어지고 구성된다.

스프링이 빈 팩토리를 위한 오브젝트 설정을 담당하는 클래스라고 인식할 수 있도록 @Configuration이라는 어노테이션을 추가하고, 오브젝트를 만들어주는 메소드에는 @Bean이라는 어노테이션을 붙여준다.

```java
@Configuration // 애플리케이션 컨텍스트 or 빈 팩토리가 사용할 설정정보라는 표시
public class DaoFactory {
	@Bean // 오브젝트 생성을 담당하는 IoC용 메소드라는 표시
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}

	@Bean
	public ConnectionMaker connectionMaker() {
		return new DConnectionMaker();
	}
}
```

이 두가지 어노테이션을 통해 DaoFactory를 빈 팩토리가 사용할 설정정보를 담은 모습니다.

아래는 새로 만든 DaoFactory 어플리케이션 컨텍스트를 적용한 UserDaoTest이다.

```java
public class UserDaoTest (
	public static void main(String[) args) throws ClassNotFoundException,SQLException (
		ApplicationContext context = 
			new AnnotationConfigApplicationContext(DaoFactory.class);
		UserDao dao = context.getBean("userDao", UserDao.class);
		// ...
}
```

getBean() 메소드는 ApplicationContext가 관리하는 오브젝트를 요청히는 메소드 다. getBean()의 파라미터인 "userDao"는 ApplicationContext에 등록된 빈의 이름이다. userDao라는 이름의 빈을 가져온다는 것 은 DaoFactory의 userDao() 메소드를 호출해서 그 결과를 가져온다고 생각하면 된다.

---

### 애플리케이션 컨텍스트의 동작방식

스프링에서는 애플리케이션 컨텍스트를 IoC 컨테이너 또는 스프링 컨테이너라고 부른다. 

애플리케이션 컨택스트는 애플리케이션에서 IoC를 적용해서 관리할 모든 오브젝트에 대한 생성과 관계설정을 담당한다. 대신 ApplicationContext에는 직접 오브젝트를 생성하고 관계를 맺어주는 코드가 없고, 그런 생성정보와 연관관계 정보를 별도의 설정정보를 통해 얻는다.

![Untitled](https://user-images.githubusercontent.com/108508730/195782835-ec67c060-7a37-4131-9b39-69196abadd68.png)

@Configuration이 붙은 DaoFactory는 이 애플리케이션 컨텍스트가 활용하는 IoC 설정정보다. 내부적으로는 애플리케이션 컨텍스트가 DaoFactory의 userDao( ) 메소드 를 호출해서 오브젝트를 가져온 것을 클라이언트가 getBean()으로 요청할 때 전달해준다.

이와 같은 애플리케이션 컨텍스트를 사용했을 때 얻을 수 있는 장점은 다음과 같다.

- **클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다**
    
    애플리케이션이 커지면 DaoFactory처럼 IoC를 적용한 오브젝트도 계속 추가될 것이다. 그러면 매번 어떤 팩토리 클래스를 사용할지 알아야하고, 필요할 때마다 팩토리 오브젝트를 생성해야하는 번거로움이 있다. 하지만 애플리케이션 컨텍스트를 사용하면 일관된 방식으로 원하는 오브젝트를 가져올 수 있다.
    
- **애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다**
- **애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다**

---

### 스프링 IoC의 용어 정리

- **빈**
    
    빈 또는 빈 오브젝트는 스프링이 IoC방식으로 관리하는 오브젝트라는 뜻이다.
    
- **빈 팩토리**
    
    스프링의 IoC를 담당하는 핵심 컨테이너를 가리킨다. 빈의 등록, 생성, 조회 등 빈의 관리하는 기능을 담당한다. 보통은 빈 팩토리를 바로 사용하기 보다 이를 확장한 애플리케이션 컨텍스트를 이용한다.
    
- **애플리케이션 컨텍스트**
    
    빈을 등록, 관리하는 기능은 빈 팩토리와 동일하지만, 스프링이 제공하는 각종 부가 서비스를 추가로 제공한다. 빈 팩토리라고 부를 때는 주로 빈의 생성과 제어의 관점에서 이야기하는 것이고, 애플리케이션 컨텍스트라고 할 때는 스프링이 제공히는 애플리케이션 지원 기능을 모두 포함해서 이야기하는 것이라고 보면 된다. ApplicationContext라고 적으면 애플리케이션 컨텍스트가 구현해야 하는 기본 인터페이스를 가리키는 것이기도 하다. ApplicationContext는 BeanFactory를 상속한다.
    
- **설정정보**
    
    스프링의 설정정보란 애플리케이션 컨텍스트 또는 빈 팩토리가 IoC를 적용하기 위해 사용하는 메타정보를 말한다. 스프링의 설정정보는 IoC 컨테이너에 의해 관리 되는 애플리케이션 오브젝트를 생성하고 구성할 때 사용된다.
    
- **컨테이너 또는 IoC 컨테이너**
    
    IoC 방식으로 빈을 관리한다는 의미에서 애플리케이션 컨텍스트나 빈 팩토리를 컨테이너 또는 IoC 컨테이너라고도 한다.
    
- **스프링 프레임워크**
    
    스프링 프레임워크는 IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 말할 때 주로 시용한다.
