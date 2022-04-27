# **Java 14 Switch 문과 Switch 표현식**
- Java 12, Java 13에서 Preview 기능으로 제공하던 `switch` 개선사항이 Java 14에서 표준 기능으로 승격하였다.
- 
- 비교를 위하여 다음과 같은 `switch`문이 있다고 하자.
  ~~~java
  switch (month) {
      case JANUARY:
      case FEBRUARY:
      case MARCH:
        System.out.println("First Quarter");
        break;
      case APRIL:
      case MAY:
      case JUNE:
        System.out.println("Second Quarter");
        break;
      case JULY:
      case AUGUST:
      case SEPTEMBER:
        System.out.println("Third Quarter");
        break;
      case OCTOBER:
      case NOVEMBER:
      case DECEMBER:
        System.out.println("Forth Quarter");
        break;
      default:
        System.out.println("Unknown Quarter");
        break;
  }
  ~~~
- 동일 case에 대해 동일 로직을 실행할 경우, 가독성이 좋지 않고, 매번 `break`문을 강제하기에 실수를 범할 위험이 있다.
- 게다가 `switch`의 결과값을 반환할 변수에 담을 수도 없었다.
- ## **새로운 방식의 `switch` 문**
  - 새로운 방식의 `switch` 문은 좀 더 읽기 쉽고, 간결하며, `break`문도 강제하지 않는다.
    ~~~java
    switch (month) {
        case JANUARY, FEBRUARY, MARCH -> System.out.println("First Quarter");
        case APRIL, MAY, JUNE -> System.out.println("Second Quarter");
        case JULY, AUGUST, SEPTEMBER -> System.out.println("Third Quarter");
        case OCTOBER, NOVEMBER, DECEMBER -> System.out.println("Forth Quarter");
        default -> System.out.println("Unknown Quarter");
    }
    ~~~
  - 복잡한 로직이 필요할 경우, 중괄호로 감쌀 수 있다.
    ~~~java
    switch (month) {
        case JANUARY, FEBRUARY, MARCH -> {
            // ...
            System.out.println("First Quarter");
        }
        // other cases ...
    }
    ~~~
- ## **새로운 `switch` 표현식**
  - 이제 `switch` 표현식을 통해 값을 반환할 수 있다.
    ~~~java
    String quarter = switch (month) {
        case JANUARY, FEBRUARY, MARCH -> "First Quarter";
        case APRIL, MAY, JUNE -> "Second Quarter";
        case JULY, AUGUST, SEPTEMBER -> "Third Quarter";
        case OCTOBER, NOVEMBER, DECEMBER -> "Forth Quarter";
        default -> "Unknown Quarter";
    };
    System.out.println(quarter);
    ~~~
  - 복잡한 로직이 필요할 경우, 중괄호로 감싸고, `yield` 키워드를 사용하여 반환한다.
    ~~~java
    String quarter = switch (month) {
        case JANUARY, FEBRUARY, MARCH -> {
            // ...
            yield "First Quarter";
        }
        case APRIL, MAY, JUNE -> {
            // ...
            yield "Second Quarter";
        }
        case JULY, AUGUST, SEPTEMBER -> {
            // ...
            yield "Third Quarter";
        }
        case OCTOBER, NOVEMBER, DECEMBER -> {
            // ...
            yield "Forth Quarter";
        }
        default -> "Unknown Quarter";
    };
    System.out.println(quarter);
    ~~~
  - 주의할 점은, `switch` 표현식은 모든 case를 다루지 않는다면 컴파일 에러가 발생한다.
    ~~~java
    // 컴파일 에러
    // 모든 case를 다루던지, default를 지정하던지 해야한다.
    String quarter = switch (month) {
        case JANUARY, FEBRUARY, MARCH -> "First Quarter";
        case APRIL, MAY, JUNE -> "Second Quarter";
        case JULY, AUGUST, SEPTEMBER -> "Third Quarter";
    };
    System.out.println(quarter);
    ~~~