### 참조

- 토비의 스프링 3.1 vol.1 111p~128p

### 제어의 역전(IoC)과 의존관계 주입(DI)

IoC는 너무 폭넓은 개념이라 정의하기 힘들다. 그래서 스프링이 제공하는 IoC 방식을 의존관계 주입(Dependency Injection)이라는 좀 더 의도가 명확히 드러나는 이름을 사용하기 시작했다.

### 의존관계 주입이란?

> Dependency Injection이란, 객체를 직접 생성하는 것이 아닌 외부에서 생성 후 주입시키는 방식을 말한다. DI를 통해 모듈간의 결합도가 낮아지고, 유연성이 상승한다.
> 

### 의존관계와 의존관계 주입

일단 두 개의 클래스 또는 모듈이 의존관계에 있다고 말할 때는 항상 방향성을 부여해줘야 한다. 만약 A가 B에 의존하고 있다면, B가 변하면 그 영향이 A로 전달된다. 하지만 A가 변해도 B에는 아무런 영향이 없다.

이전에 만든 UserDao클래스를 예로 들면 UserDao 클래스는 ConnectionMaker 인터페이스에게만 직접 의존하고, DConnectionMaker라는 클래스의 존재도 알지 못한다. 이처럼 모델이나 코드에서 클래스와 인터페이스를 통해 드러나는 의존관계 말고, 런타임 시에 오브젝트 사이에서 만들어지는 의존관계도 있다.

의존관계 주입은 구체적인 의존 오브젝트와 그것을 사용할 주체, 보통 클라이언트라고 부르는 오브젝트를 런타임 시에 연결해주는 작업을 말한다.

정리하면 의존관계 주입이란 다음과 같은 세 가지 조건을 충족히는 작업을 밀한다.

- 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야 한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
- 의존관계는 시용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.

UserDao의 의존관계 주입을 다시 보면

```java
public class UserDao (
	private ConnectionMaker connectionMaker;

	public UserDao(ConnectionMaker connectionMaker) {
		this.connectionMaker =connectionMaker;
	}
	// ...
}
```

DaoFactory는 DI 컨테이너로써 UserDao를 만드는 시점에서 생성자의 파라미터로 이미 만들어진 DConnectionMaker의 오브젝트의 레퍼런스를 전달한다.

UserDao는 DaoFactory에게 전달받은 런타임 의존관계를 갖는 오브젝트를 connectionMaker에 저장해둔다.

![image](https://user-images.githubusercontent.com/108508730/195784169-4fdeff89-027d-425d-b8b2-684572ec96a0.png)

위의 사진은 이런 객체에 대한 런타임 의존관계 주입과 그것으로 발생하는 런타임 사용 의존관계의 모습을 보여 준다.

DI는 자신이 사용할 오브젝트에 대한 선택과 생성 제어권을 외부로 넘기고 자신은 수동적으로 주입받은 오브젝트를 사용한다는 점에서 IoC의 개념에 잘 맞는다. 스프링 컨테이너의 IoC는 주로 의존관계 주입에 초점이 맞춰져 있다. 그래서 스프링을 IoC 컨테이너 또는 DI 컨테이너, DI 프레임워크 라고 부른다.

---

### 의존관계 검색

의존관계 검색은 런타임 시 의존관계를 맺을 오브젝트를 결정히는 것과 오브젝트의 생성 작업은 외부 컨테이너에게 IoC로 맡기지만 이를 가져올 때는 메소드나 생성자를 통한 주입 대신 스스로 컨테이너에게 요청하는 방법을 사용한다.

```java
public UserDao() {
	AnnotationConfigApplicationContext context =
		new AnnotationConfigApplicationContext(DaoFactory.class);
	this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```

UserDao에 의존관계 검색을 적용해보면 위와 같이 스프링의 IoC 컨테이너인 애플리케이션 컨텍스트가 제공하는 getBean()이라는 매소드를 통해 의존관계를 검색한다.

하지만, 의존관계 검색 방법은 코드안에 오브젝트 팩토리 클래스나 스프링 API가 나타난다. 애플리케이션 컴포넌트가 컨테이너와 같이 성격이 다른 오브젝트에 의존하게 되는 것이므로 그다지 바람직하지 않다.

→ 그래서 대개는 의존관계 주입 방식을 사용하는 편이 낫다.

(의존관계 검색 부분 잘 이해 안됨.. 118p~119p 다시보기)

---

### 의존관계 주입의 응용

- **기능 구현의 교환**
    
    개발중에는 개발자 PC에 설치된 로컬 DB를 사용하다가 지금까지 개발한 것을 운영서버에 배치하려고 한다. 이때 각 DAO에서 로컬 DB를 연결해주는 클래스를 사용하고 있었다면, 이 클래스들을 전부 운영서버 DB를 연결해주는 클래스로 전부 바꿔야한다.
    
    하지만, DI방식을 적용했다고 하면 각 DAO는 DB연결 클래스를 ConnectionMaker라는 타입의 오브젝트로 받고 있었고, 우리는 DaoFactory에서 어떤 ConnectionMaker 구현 클래스를 사용할 것인지만 바꿔주면 된다.
    
    ```java
    @Bean
    public ConnectionMaker connectionMaker() (
    	// return new LocalDBConnectionMaker(); 로컬 DB 연결 클래스
    	return new ProductionDBConnectionMaker(); // 운영서버 DB 연결 클래스
    }
    ```
    
    또한, 별도의 테스트용 DB를 만들어서 적용시킨다 해도 DAO코드에 전혀 손댈 필요없이 테스트 DB접속 클래스를 만들고, 위의 코드만 수정해주면 된다.
    
- **부가기능 추가**
    
    만약 DAO가 DB를 얼마나 많이 연결하는지 파악하고 싶다고 해보자. 일반적으로는 모든 DAO의 makeConnection() 메소드를 호출하는 부분에 DB호출 카운터를 증가시키는 코드를 넣어야 한다.
    
    하지만 DI컨테이너에서는 DAO와 DB커넥션을 만드는 오브젝트 사이에 연결횟수를 카운팅하는 오브젝트를 하나 더 추가하는 식으로 해결할 수 있다.
    
    ```java
    public class CountingConnectionMaker implements ConnectionMaker {
    	int counter = 0;
    	private ConnectionMaker realConnectionMaker;
    	public CountingConnectionMaker(ConnectionMaker realConnectionMaker) {
    		this.realConnectionMaker =realConnectionMaker;
    	}
    	public Connection makeConnection() throws ClassNotFoundException, SQLException {
    		this.counter++;
    		return realConnectionlaker.makeConnection();
    	}
    	public int getCounter() {
    		return this.counter;
    	}
    }
    ```
    
    위와 같이 ConnectionMaker 인터페이스를 상속받는 CountingConnectionMaker를 만들고,
    
    ```java
    @Configuration
    public class CountingDaoFactory {
    	@Bean
    	public UserDao userDao() {
    		return new UserDao(connectionMaker());
    	}
    	@Bean
    	public ConnectionMaker connectionMaker() (
    		return new CountingConnectionMaker(realConnectionMaker());
    	}
    	@Bean
    	public ConnectionMaker realConnectionMaker() (
    		return new DConnectionMaker();
    	}
    }
    ```
    
    이렇게 설정용 클래스를 만든 뒤 위 두개의 클래스를 이용한 새로운 Test 클래스를 만들면
    
    ```java
    public class UserDaoConnectionCountingTest {
    	public static void main(String[) args) throws ClassNotFoundException,
    			SQLException (
    		AnnotationConfigApplicationContext context =
    			new AnnotationConfigApplicationContext(CountingDaoFactory.class);
    		UserDao dao = context.getBean("userDao" , UserDao .class);
    		CountingConnectionMaker ccm = context.getBean("connectionMaker",
    			CountingConnectionMaker.class);
    		System.out.println("Connection counter "+ ccm.getCounterO);
    	}
    }
    ```
    
    애플리케이션 컨텍스트 설정파일로 위에 만들어둔 CountingDaoFactory를 지정하고, getBean()로  DConnectionMaker를 주입받은 CountingConnectionMaker와 CountingConnectionMaker를 주입받은 UserDao를 가져와서 테스트를 진행한다.
    
    아래는 CountingConnectionMaker를 적용한 뒤의 런타임 오브젝트 의존관계 그림이다.
    
    ![image](https://user-images.githubusercontent.com/108508730/195784094-ce4006d6-711b-4617-8935-e9fc1c96cd69.png)
    

---

### 메소드를 이용한 의존관계 주입

의존관계 주입을 할 때 항상 생성자를 사용해야 하는 것은 아니다. 의존관계를 주입하는 다른 방법은 크게 두 가지가 있다.

1. 수정자(setter) 메소드를 이용한 주입
    
    setter 메소드는 외부에서 오브젝트 내부의 속성값을 변경할 때 사용된다. setter는 외부로부터 제공받은 오브젝트 레퍼런스를 저장해뒀다가 내부의 메소드에서 사용하게 하는 DI방식에서 활용하기에 좋다.
    
2. 일반 메소드를 이용한 주입
    
    setter는 set으로 시작해야하고, 한 번에 한 개의 파라미터만 가질 수 있다는 제약이 있다. 이러한 제약이 싫다면 일반 메소드를 통해서도 DI를 할 수 있다.
    

스프링은 전통적으로 setter 주입을 많이 사용해왔다. 보통 이름은 set + 오브젝트의 타입이름(ex: ConnectionMaker) 와 같이 짓는다.
