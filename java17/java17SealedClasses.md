# **Java 17 Sealed 클래스**
- Java 15, 16의 Preview 기능이던 Sealed 클래스가 Java 17에서 표준 기능으로 포함되었다.
- 클래스(또는 인터페이스)는 이제 이를 상속하거나 구현할 수 있는 하위 클래스 또는 인터페이스를 제한할 수 있다.
- 이는 도메인 모델링 및 라이브러리 보안 강화에 유용한 기능이 될 수 있다.
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
  - 클래스/인터페이스 수정자 위치에 `sealed` 키워드를 사용하여 해당 클래스/인터페이스가 Sealed 클래스 인터페이스 임을 선언한다.
  - `permits` 키워드로 해당 Sealed 클래스/인터페이스를 확장할 수 있는 서브 클래스/인터페이스를 정의한다.
  - ### **Sealed 인터페이스**
    ~~~java
    public sealed interface Vehicle permits Truck, Car, Bike {
      void go();
      void stop();
    }
    ~~~
  - ### **Sealed 클래스**
    - 반드시 추상 클래스로만 선언할 수 있다.
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
- ## **서브 클래스**
  - Sealed 클래스/인터페이스를 확장/구현하는 서브 클래스는 반드시 다음 3가지 수정자 키워드 중 하나를 선택해야 한다.
    1. ***`final`***
    2. ***`non-sealed`***
    3. ***`sealed`***
  - ### **`final` 클래스**
    - 더이상 확장이 불가능하도록 `final`클래스로 선언한다.
    ~~~java
    public final class Bike implements Vehicle {
      @Override
      public void go() {
        System.out.println("Go bike.");
      }
      @Override
      public void stop() {
        System.out.println("Stop bike.");
      }
    }
    ~~~
  - ### **`non-sealed` 클래스**
    - 해당 키워드를 선언하면 알려지지 않은 서브 클래스에 의해 확장이 가능하다.
    ~~~java
    // Car.java
    // non-sealed 선언으로 확장이 가능하다.
    public non-sealed class Car extends EngineVehicle implements Vehicle {
      @Override
      public void go() {
        System.out.println("Go car.");
      }
      @Override
      public void stop() {
        System.out.println("Stop car.");
      }
      @Override
      protected String engineType() {
        return "Gasoline";
      }
      public void getIn() {
        System.out.println("Get it passengers.");
      }
      public void getOut() {
        System.out.println("Get out passengers.");
      }
    }

    // Avante.java
    // Car 클래스를 확장한 클래스
    public class Avante extends Car {    
    }

    // Tesla.java
    // Car 클래스를 확장한 클래스
    public class Tesla extends Car {
      @Override
      protected String engineType() {
        return "Electric";
      }
    }
    ~~~
  - ### **`Sealed` 클래스/인터페이스**
    - 하위 계층의 Sealed 구조를 생성한다.
    - `sealed` 와 `permits` 키워드를 사용한다.
    - 반드시 추상 클래스 또는 인터페이스로만 선언이 가능하다.
    ~~~java
    // Truck.java
    // Sealed 클래스
    public abstract sealed class Truck extends EngineVehicle implements Vehicle permits DumpTruck {
      public abstract void load();
      public abstract void upload();
    }

    // DumpTruck.java
    // Truck Sealed 클래스를 확장한 final 클래스
    public final class DumpTruck extends Truck {
      @Override
      public void go() {
        System.out.println("Go dump truck.");
      }
      @Override
      public void stop() {
        System.out.println("Stop dump truck.");
      }
      @Override
      protected String engineType() {
        return "Diesel";
      }
      @Override
      public void load() {
        System.out.println("Load cargo.");
      }
      @Override
      public void upload() {
        System.out.println("Unload cargo.");
      }
    }
    ~~~
- ## **제약사항**
  - Sealed 클래스는 허용되는 서브 클래스에 다음 3가지 중요한 제약조건을 부과한다.
    1. 허용된 모든 서브 클래스는 Sealed 클래스와 동일한 모듈에 속해야 한다.
    2. 허용된 모든 서브 클래스는 Sealed 클래스를 명시적으로 확장해야 한다.
    3. 허용된 모든 서브 클래스는 수정자(*`final`*, *`non-sealed`*, *`sealed`*)를 정의해야 한다.
- ## **클라이언트에서의 사용**
  - Sealed 클래스는 도메인/API 개발자 뿐만 아니라, 해당 클래스를 사용하는 클라이언트에게도 이점이 있다.
  - 클라이언트 개발자도 이제 서브 클래스가 무엇인지 명확하게 알 수 있어, 불필요한 코드가 줄어들게 된다.
  - 이전에는 다음과 같이 알려지지 않은 서브 클래스도 고려한 코드를 작성해야 했다.
    ~~~java
    if (vehicle instanceof Truck truck) {
      // ...
    } else if (vehicle instanceof Car car) {
      // ...
    } else if (vehicle instanceof Bike bike) {
      // ...
    } else {
      throw new RuntimeException("Unknown instance of Vehicle");
    }
    ~~~
  - 더이상 `else`는 필요하지 않고, [JEP406: Pattern Matching for `switch`](https://openjdk.java.net/jeps/406)가 적용되면, `switch` 문에서도 `enum` 처럼 사용이 가능하다.
  - 생성시점에 모든 타입을 알 수 있는 `enum` 의 장점을 클래스 계층구조에 적용한 것으로 보인다.