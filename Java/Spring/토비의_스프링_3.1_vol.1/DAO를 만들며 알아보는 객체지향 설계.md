# DAO를 만들며 알아보는 객체지향 설계

### 참조

- 토비의 스프링 3.1 vol.1 54p~87p
- [https://davidpark20.tistory.com/category/ Spring/토비의 스프링 3.1](https://davidpark20.tistory.com/category/%20Spring/%ED%86%A0%EB%B9%84%EC%9D%98%20%EC%8A%A4%ED%94%84%EB%A7%81%203.1)

### DAO

> DAO(Data Access Object)는 DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트를 말한다.
> 

### Java Bean

> 자바빈(Java Bean)은 아래 두가지 관례를 따르는 오브젝트를 가리키며, 간단히 빈이라고 부르기도 한다.
> 
> - 디폴트 생성자: 파라미터가 없는 디폴트 생성자
> - 프로퍼티: getter + setter

---

**사용자 정보를 JDBC API를 통해 DB에 저장하고 조회할 수 있는 간단한 DAO 만들기**

User

```java
public class User {
    String id;
    String name;
    String password;  

    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
}
```

User 테이블

UserDao

```java
public class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        // MySql
        // Class.forName("com.mysql.jdbc.Driver");
        // Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");

        // Oracle
        Class.forName("oracle.jdbc.driver.OracleDriver");
        Connection c = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:xe", "spring", "book");

        PreparedStatement ps = c.prepareStatement("INSERT INTO users(id, name, password) VALUES (?, ?, ?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("oracle.jdbc.driver.OracleDriver");
        Connection c = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:xe", "spring", "book");

        PreparedStatement ps = c.prepareStatement("SELECT * FROM users WHERE id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
}
```

DAO 테스트 코드

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException{
    UserDao dao = new UserDao();

    User user = new User();
    user.setId("whiteship");
    user.setName("백기선");
    user.setPassword("married");

    dao.add(user);

    System.out.println(user.getId() + "등록 성공");

    User user2 = dao.get(user.getId());
    System.out.println(user2.getName());
    System.out.println(user2.getPassword());

    System.out.println(user2.getId() + "조회 성공");
}
```

위의 코드들은 모든 기능이 잘 작동되지만, 이렇게 개발하면 바로 회사에서 쫓겨난다.

→ UserDao를 객체지향 기술의 원리에 충실한 스프링 스타일의 코드로 개선해보자

---

객체지향에서는 오브젝트에 대한 설계와 이를 구현한 코드 등 모든 것이 계속 변한다.

그러므로 객체를 설계할 때 개발자는 미래의 변화를 어떻게 대비할지를 염두해 둬야한다.

변화는 대체로 집중된 한 가지 관심에 대해 일어나지만 그에 따른 작업은 한곳에 집중되지 않는 경우가 많다.

→ 분리와 확장을 고려한 설계를 위해 관심사의 분리 필요

관심사의 분리

객체지향에서의 관심사의 분리는 관심사가 같은 것 끼리는 하나의 객체 or 가까운 객체로 모이게 하고, 관심사가 다른 것들은 최대한 떨어뜨려서 서로 영향을 주지 않도록 분리해야한다.

---

### UserDao의 관심사항

1. DB연결
2. DB에 보낼 SQL문
3. 작업 후 Statement와 Connection 오브젝트 close

가장 문제가 되는 부분은 1번 부분인 DB연결을 위한 Connection오브젝트를 가져오는 부분이다.

현재 DB 커넥션을 가져오는 코드는 다른 관심사와 섞여서 add() 메소드에 담겨있다. 또한 get() 메소드에도 add() 메소드에 있는 DB 커넥션을 가져오는 코드가 중복되어 있다

getConnection() 메소드를 추출해서 중복을 제거한 UserDao

```java
public class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();

        PreparedStatement ps = c.prepareStatement("INSERT INTO users(id, name, password) VALUES (?, ?, ?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();

        PreparedStatement ps = c.prepareStatement("SELECT * FROM users WHERE id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
		
		private Connection getConnection() throws ClassNotFoundException, SQLException {
				// MySql
        // Class.forName("com.mysql.jdbc.Driver");
        // Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
        Class.forName("oracle.jdbc.driver.OracleDriver");
				Connection c = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:xe", "spring", "book");
				return c;
		}
}
```

위와 같이 UserDao의 기능에는 아무런 영향을 주지 않으며 나중의 변화에 좀더 대응하기 좋도록 코드의 구조만 변경하는 작업을 리팩토링(refactoring)이라고 한다.

위의 예시에서는 중복된 코드를 메소드로 뽑아내는 작업을 했는데 이를 메소드 추출 기법이라 한다.

---

### 상속을 통한 확장

만약 getConnection을 특정 DB에 상관없이 각자 맞게 변형해서 쓸 수 있도록 하고싶다면 어떻게 해야할까?

→ getConnection을 추상 메소드로 만들어서 각자 원하는 형태로 구현할 수 있도록 한다

```java
public class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
				...
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
				...
    }
		
		private abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}

// ---------------------------------------

public class NUserDao extends UserDao {
		public Connection getConnection() throws ClassNotFoundException, SQLException {
				// N사의 DB Connection 생성 코드
		}
}
```

위와 같이 슈퍼클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 방법을 디자인 패턴에서 **템플릿 메소드 패턴**이라고 한다.

또한, 서브클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 **팩토리 메소드 패턴**이라고 부르기도 한다.

하지만, 상속을 사용했다는 단점이 있다. 자바는 다중상속을 허용하지 않으므로 이미 상속을 받은 상태면 적용하기 힘들다.

또한, 서브클래스는 슈퍼클래스의 기능을 사용할 수 있다. 그래서 슈퍼클래스 내부의 변경이 있을 때 모든 서브클래스를 함께 수정해야 할 수도 있다.

→ 클래스를 분리해보자

---

### 클래스의 분리

이번에는 관심사가 다른 두 가지 코드를 서로 독립적인 클래스로 만들어보겠다.

```java
public class UserDao {

    private SimpleConnectionMaker simpleConnectionMaker;

    public UserDao() {
        // 한 번만 만들어 인스턴스 변수에 저장해두고 메소드에서 사용하게 한다.
        simpleConnectionMaker = new SimpleConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = simpleConnectionMaker.makeNewConnection();
        // ...
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = simpleConnectionMaker.makeNewConnection();
        // ...
    }

    public static void main(String[] args) throws ClassNotFoundException, SQLException{
        // ...
    }
}
```

SimpleConnectionMaker

```java
public class SimpleConnectionMaker {
    public Connection makeNewConnection() throws ClassNotFoundException, SQLException{
        Class.forName("oracle.jdbc.driver.OracleDriver");
        Connection c = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:xe", "spring", "book");

        return c;
    }
}
```

이렇게 아예 다른 클래스로 분리해두고, UserDao에서는 SimpleConnectionMaker객체를 생성하여 SimpleConnectionMaker의 makeNewConnection 메서드를 사용하여 DB연결을 구현하였다.

하지만 이렇게되면 처음처럼 DB 커넥션을 가져오는 방법을 자유롭게 확장하기 힘들어진다.

---

### 인터페이스

위와 같은 문제를 해결하려면 두 개의 클래스가 서로 긴밀하게 연결되어 있지 않도록 인터페이스를 통해 추상적인 느슨한 연결고리를 만들어 줘야한다.

ConnectionMaker 인터페이스

```java
public interface ConnectionMaker {
	public Connection makeConnection() throws ClassNotFoundException, SQLException;
}
```

위의 인터페이스를 이용하여 각 회사는 자신에게 맞는 DB 커넥션 생성 코드를 구현할 수 있다.

ConnectionMaker 인터페이스를 상속받은 DConnectionMaker

```java
public class DConnectionMaker implements ConnectionMaker {
	public Connection makeConnection() throws ClassNotFoundException, SQLException {
		// D사의 DB 커넥션을 생성하는 코드
	}
}
```

ConnectionMaker 인터페이스를 사용하도록 개선한 UserDao

```java
public class UserDao {
    // 인터페이스를 통해 오브젝트에 접근하므로 구체적인 클래스 정보 알 필요 없음
    private ConnectionMaker connectionMaker;

    public UserDao() {
        // 그러나 여기에 클래스 이름이 나옴
        connectionMaker = new DConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        // 인터페이스에 정의된 메소드를 사용하므로 클래스가 바뀌어도 걱정없음
        Connection c = connectionMaker.makeConnection();
        // ...
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeConnection();
        // ...
    }

    public static void main(String[] args) throws ClassNotFoundException, SQLException{
        // ...
    }
}
```

하지만, 이 경우에도 DConnectionMaker라는 클래스 이름이 나온다..

---

### 관계설정 책임의 분리

위의 코드에서 다른 관심사가 완벽하게 분리되려면 UserDao가 어떤 ConnectionMaker 구현 클래스의 오브젝트를 사용할지를 결정하는 부분을 분리해야 한다.

→ UserDao를 사용하는 오브젝트에서 어떤 ConnectionMaker 구현 클래스의 오브젝트를 사용할지 정해주면 된다. 메소드 파라미터나 생성자 파라미터의 타입을 오브젝트의 인터페이스로 선언한다면 이 문제를 해결할 수 있다!

수정된 UserDao의 생성자

```java
public UserDao(ConnectionMaker = connectionMaker) {
	this.connectionMaker = connectionMaker;
}
```

관계설정 책임이 추가된 UserDao 클라이언트인 main() 메소드

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException{
    ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao dao = new UserDao(connectionMaker);
		// ...
}
```

비로소 UserDao에서 DB커넥션에 대한 관심사항을 분리하였다. 이리하여 UserDao의 수정 없이 DB연결 기능을 확장해서 사용할 수 있게 되었다!

![최종 구조](https://user-images.githubusercontent.com/108508730/195374533-b37b832a-eab7-4330-b0bd-b6be323b2952.png)

최종 구조

---

### 객체지향 설계 원칙(SOLID)

- SRP(The Single Responsibility Principle): 단일 책임 원칙
- OCP(Open Closed Principle): 개방 폐쇄 원칙
- LSP(The Liskov Substitution Principle): 리스코프 치환 원칙
- ISP(The Interface Segregation Principle): 인터페이스 분리 원칙
- DIP(The Dependency Inversion Principle): 의존관계 역전 원칙

### 개방 폐쇄 원칙(OCP)

개방 폐쇄 원칙을 간단히 정의하자면 ‘클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀있어야 한다.’라고 할 수 있다.

UserDao는 DB연결 방법이라는 기능을 확장할 수 있게 되어 있지만, UserDao 자신의 핵심기능을 구현한 코드는 변화에 영향을 받지 않고 유지할 수 있으므로 변경에는 닫혀 있다고 할 수 있다.

**높은 응집도와 낮은 결합도**

- 높은 응집도
    
    응집도가 높다는 것은 변화가 일어날 때 해당 모듈에서 변하는 부분이 크다는 것이다.
    
- 낮은 결합도
    
    결합도가 낮다는 것은 하나의 오브젝트가 변경이 일어날 때에 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도가 낮다는 것이다.
    

→ UserDao 클래스는 사용자의 데이터를 처리하는 기능이 DAO안에 깔끔하게 모여있다. 동시에 UserDao와 ConnectionMaker의 관계는 인터페이스를 통해 매우 느슨하게 연결되어 있다.

**전략 패턴**

> 전략 패턴은 자신의 기능에서 필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리시키고, 이를 구현한 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴이다.
>
