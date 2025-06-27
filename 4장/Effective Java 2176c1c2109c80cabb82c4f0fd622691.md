# Effective Java

**태그 달린 클래스보다는 클래스 계층구조를 활용하라**

객체지향의 다형성에 가까운 얘기다.

"태그달린 클래스(tagged class)"는 보통 **Java나 객체지향 프로그래밍에서 잘못된 설계 패턴** 중 하나로 언급되는 개념이에요. 간단히 말해서, **여러 종류의 객체를 하나의 클래스에 넣고, `tag` 필드를 통해 구분**하는 방식이에요.

```jsx
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 – 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드는 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                // 예외 처리를 하지 않으면 컴파일 에러 발생할 수 있음
                throw new AssertionError("Unknown shape: " + shape);
        }
    }
}

```

로버트 마틴 만든 [SOLID 규칙](https://mangkyu.tistory.com/194) 을 생각해보자.

### **단일 책임 원칙 (Single responsibility principle)**

SRP는 클래스의 책임이 한가지만 있어야 한다는 것이다. 여기서 FIgure 클래스는 두 가지의 책임이 있다. 한 가지는 Circle의 관련된 책임이고, 다른 한 가지는 Rectangle에 관련된 책임이다.

### **개방-폐쇄 원칙 (Open/closed principle)**

OCP는 확장에는 열려있으나 변경에는 닫혀 있어야한다. **확장에 대해 열려 있다는 말은**,애플리케이션의 요구사항이 변경될 때 이 변경에 맞게 새로운 동작을 추가해서 애플린케이션의 기능을 확장할 수 있다는 말이다. **수정에 대해 닫혀 있다는 말은,** 기존의 코드를 수정하지 않고도 애플리케이션의 동작을 추가하거나 변경할 수 있다는 말이다.

- 정사각형을 추가해본다.

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE, SQUARE };

    // 태그 필드 – 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드는 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;
    
    // 정사각형
    int side;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    Figure(int side){
	    shape = Shape.SQUARE;
	    this.side = side;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            case SQUARE:
		            return side * side;
            default:
                // 예외 처리를 하지 않으면 컴파일 에러 발생할 수 있음
                throw new AssertionError("Unknown shape: " + shape);
        }
    }
}
```

정사각형을 추가한다면 위와 같이 기존의 코드를 수정을 해야한다. 이런 태그 달린 클래스에 단점은, 우선 열거 타입 선언, 태그 필드, switch등 쓸데없는 코드가 많다. 여러 구현이 한 클래스에 혼합돼 있어서 가독성이 나쁘기 때문에 추후에 수정을해야 할 상황이 생기면 어디를 수정해야할지 감이 안온다. 또한 새로운 타입을 추가할 때마다 모든 switch 문을 찾아 새 의미를 처리하는 코드를 추가해야 하는데, 하나라도 빠지면 역시 런타임에 문제가 일어난다.

```jsx
Figure figure = new Figure(0.5,0.3); //어떤 타입인지 알수가 없다.
```

따라서 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.

클래스 계층구조로 바꾸어 보면,

```java
abstract class Figure {
  abstract dobule area(); 
}

class Circle extends Figure {
  final double radius;
  
  Circle(double radius) {this.radius = radius;}
  
  @Override double area() {return Math.PI * (radius * radius);}
}

class Rectangle extends Figure {
  final double length;
  final double width;
  
  Rectangle(double length, double width) {
    this.length = length;
    this.width = width;
  }
  
  @Override double area() {return length * width;}
}

//정사각형
class Square extends Rectangle {
		Square(double side) {
				super(side, side);
		}
}
```

일반적인 웹어플리케이션에서 우리가 만드는 조회, 저장, 삭제 등의 컨트롤러나 서비스 로직은 대부분 다형성을 고려하지 않는 형태로 작성한다.
하지만 그게 **잘못된 것은 아니고**, **적절한 추상화나 다형성이 필요한 경우에만 사용하는 것이 원칙**이다.

### **보통의 CRUD 구조**

```java
@RestController
@RequestMapping("/users")
public class UserController {
    private final UserService userService;

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }

    @PostMapping
    public void createUser(@RequestBody User user) {
        userService.save(user);
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}

```

```java
@Service
public class UserService {
    public User findById(Long id) { /* ... */ }
    public void save(User user) { /* ... */ }
    public void delete(Long id) { /* ... */ }
}

```

### 다형성이 필요한 경우는?

```java
// Product 인터페이스
interface Notification {
    void notifyUser();
}

// ConcreteProduct 클래스
class EmailNotification implements Notification {
    @Override
    public void notifyUser() {
        System.out.println("이메일 알림을 보냅니다.");
    }
}

class SmsNotification implements Notification {
    @Override
    public void notifyUser() {
        System.out.println("SMS 알림을 보냅니다.");
    }
}

// Factory 클래스
class NotificationFactory {
    public static Notification createNotification(String type) {
        if (type.equalsIgnoreCase("EMAIL")) {
            return new EmailNotification();
        } else if (type.equalsIgnoreCase("SMS")) {
            return new SmsNotification();
        }
        throw new IllegalArgumentException("알 수 없는 알림 타입: " + type);
    }
}

// Client 코드
public class FactoryPatternExample {
    public static void main(String[] args) {
        Notification emailNotification = NotificationFactory.createNotification("EMAIL");
        emailNotification.notifyUser();

        Notification smsNotification = NotificationFactory.createNotification("SMS");
        smsNotification.notifyUser();
    }
}
```

```jsx
// Strategy 인터페이스
interface PaymentStrategy {
    void pay(int amount);
}

// ConcreteStrategy 클래스
class CardPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("카드로 " + amount + "원을 결제합니다.");
    }
}

class KakaoPayPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("카카오페이로 " + amount + "원을 결제합니다.");
    }
}

// Context 클래스
class PaymentContext {
    private PaymentStrategy paymentStrategy;

    // 전략을 설정하는 메서드
    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    // 결제 수행
    public void executePayment(int amount) {
        if (paymentStrategy == null) {
            throw new IllegalStateException("결제 전략이 설정되지 않았습니다.");
        }
        paymentStrategy.pay(amount);
    }
}

// Client 코드
public class StrategyPatternExample {
    public static void main(String[] args) {
        PaymentContext paymentContext = new PaymentContext();

        // 카드 결제 전략 설정
        paymentContext.setPaymentStrategy(new CardPayment());
        paymentContext.executePayment(50000);

        // 카카오페이 결제 전략 설정
        paymentContext.setPaymentStrategy(new KakaoPayPayment());
        paymentContext.executePayment(30000);
    }
}
```