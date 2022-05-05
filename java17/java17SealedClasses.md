# **Java 17 Sealed 클래스**
- Java 15, 16의 Preview 기능이던 Sealed 클래스가 Java 17에서 표준 기능으로 포함되었다.
- 클래스(또는 인터페이스)는 이제 이를 상속하거나 구현할 수 있는 하위 클래스 또는 인터페이스를 제한할 수 있다.
- 이는 도메인 모델링 및 라이브러리 보안 강화에 유용한 기능이 될 수 있다.
- 생성시점에 모든 타입을 알 수 있는 enum의 장점을 클래스 계층구조에 적용한 것으로 보인다.
- ## **도입배경**
  - Java의 클래스 계층구조(Class Hierarchy)는 상속을 통해 코드를 재사용할 수 있다.
  - 그러나 클래스 계층구조의 목적이 항상 코드의 재사용에 있지는 않다.
  - 클래스 계층구조의 다른 목적은 도메인에 존재하는 다양한 가능성을 모델링하는데 있다.
  - 이는 다시 말하면, ***추상화된*** 비즈니스 도메인은 특정 비즈니스를 갖는 다양한 도메인 모델로 확장할 수 있다는 의미이다. *(라이브러리의 API 모델링에도 마찬가지다.)*
  - 이 경우, 개발자의 목표는 알 수 있는 하위 클래스를 처리하는 코드의 명확성에 있다.
  - 문제는 Java가 이 계층구조를 코드 재사용의 목적으로 하기에 무조건 확장이 가능하다는데 있다.
  - 이러한 이유로, 개발자의 주된 관심의 목표가 아닌, 알 수 없는 하위 클래스의 확장을 방어하는 코드를 만들어내야 한다.
  - 이를 위한 방법은 수퍼 클래스를 package-private으로 만들고, 서브 클래스는 `final`로 선언하여 확장을 막는 것이다.
    ~~~java
    abstract class Shape {
      // ...
    }
    public final class Triangle extends Shape {
      // ...
    }
    public final class Rectangle extends Shape {
      // ...
    }
    ~~~
  - 위의 코드의 경우, `Shape`의 코드를 공유하는 코드 재사용의 관점에서는 유용하다.
  - 하지만, 클라이언트 코드가 수퍼 클래스인 `Shape`에 접근할 수 없기 때문에, 모델링의 관점에서는 쓸모가 없다.
  - 다음과 같은 코드들을 작성할 수 없기 때문이다.
    ~~~java
    Shape shape = ...

    public void method(Shape shape) {
      //...
    }
    ~~~
  - 또한, 수퍼 클래스는 그 의도한 사용법을 문서화하여 공개할 수도 없다.
  - 여기서 우리에게 필요한 요구사항이 도출된다.
    1. ***수퍼 클래스는 클라이언트를 위해 광범위하게 접근 가능해야 한다.***
    2. ***수퍼 클래스는 서브 클래스 제한을 위해 광범위하게 확장될 수 없어야 한다.***
    3. ***더불어 수퍼 클래스는 서브 클래스가 `final` 등으로 상태를 정의하도록 과도하게 제한해서는 안된다.***
- ## **Sealed 클래스/인터페이스 생성**
  - 주요 키워드는 **`sealed`** 와 **`permits`** 이다.
  - ### **Sealed 인터페이스**
    ~~~java
    public sealed interface Vehicle permits Truck, Car, Bike {
      void go();
      void stop();
    }
    ~~~
  - ### **Sealed 클래스**
    ~~~java
    public abstract sealed class EngineVehicle permits Truck, Car {
      public void start() {
        System.out.println("Start the " + engineType() + " engine.");
      }
      public void turnOff() {
        System.out.println("Turn off the " + engineType() + " engine.");
      }
      protected abstract String engineType();
    }
    ~~~