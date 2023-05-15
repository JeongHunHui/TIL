### 자료 구조

자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.

엔티티, DTO도 자료구조에 해당한다.

### 객체

추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.

### **객체 + 객체 지향 기법**

만약 여러가지 도형과 해당 도형의 넓이를 구해야 한다면 어떻게 할까?

객체 + 객체 지향 기법 으로 구현한다면, Shape라는 인터페이스와, 이를 상속받는 Square, Ractangle, Circle 등의 클래스를 만든다.

아래의 코드와 같다.

```java
public class Square implements Shape {
    private Point topLeft;
    private double side;
 
    public double area() {
        return side * side;
    }
}
 
public class Rectangle implements Shape {
    private Point topLeft;
    private double height;
    private double width;
 
    public double area() {
        return height * width;
    }
}
 
public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.141592653589793;
 
    public double area() {
        return PI * radius * radius;
    }
}
```

이렇게 했을 때의 장/단점은

- 장점: 새로운 도형을 추가하기 쉽다.
- 단점: 새로운 함수를 추가하기 어렵다.

### **자료 구조 + 절차적인 코드**

반면 똑같은 코드를 자료구조와 절차적인 코드로 구현했을 때는, 각 도형을 자료구조로 구현하고, 도형을 받아서 넓이를 계산하는 메소드를 가진 클래스를 만드는 방식으로 구현하게 된다.

아래의 코드와 같다.

```java
public class Square {
    private Point topLeft;
    private double side;
}
 
public class Rectangle {
    private Point topLeft;
    private double height;
    private double width;
}
 
public class Circle {
    private Point center;
    private double radius;
}
 
public class Geometry {
    public final double PI = 3.141592653589793;
    
    public double area(Object shape) throw NoSuchShapeException {
        if (shape instanceof Square) {
            Square s = (Square)shape;
            return s.side * s.side;
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangel)shape;
            return r.height * r.width;
        } else if (shape instanceof Circle) {
            Circle c = (Circle)shape;
            return PI * c.radius * c.radius;
        }
    }
}
```

이렇게 했을 때의 장/단점은 

- 장점: 함수 추가가 쉽다.
- 단점: 새로운 도형을 추가하기 어렵다.

⇒ 둘의 장단점을 잘 생각하여 무조건 객체지향적으로 짠다는 생각만 하지 말고 상황에 맞게 잘 사용하자.

### 디미터 법칙

자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙을 말한다.

예를 들어, 클래스 c의 메서드 f는 아래와 같은 객체의 메서드만 호출해야 한다.

- c
- f가 생성한 객체
- f의 파라미터로 넘어온 객체
- c 인스턴스 변수에 저장된 객체
