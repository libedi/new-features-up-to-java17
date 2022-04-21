# **Java 12 새로 추가된 기능**
## **1. `String` 클래스 신규 메소드**
- ### `indent(int n)`
  - 문자열의 각각의 라인의 들여쓰기를 조정한다.
  - n > 0이면, n만큼 공백문자를 삽입한다.
    ~~~java
    String str = "Hello World!\nThis is Java 12 new features.";
    str = str.indent(4);
    System.out.println(str);
    System.out.println("--indented string--")
    //    Hello World!
    //    This is Java 12 new features.
    //
    //--indented string--
    ~~~
  - 주의할 점은, 공백 삽입 전에 개행문자로 라인을 구분하는데, 마지막에 개행문자가 없으면 공백 삽입 후 개행문자 `"\n"`(U+000A)이 접미사로 붙는다.
    ~~~java
    String str = "a test string";
    System.out.println(str);
    String indentedStr = str.indent(5);
    System.out.println(indentedStr.endsWith("\n"));
    System.out.printf("'%s'%n", indentedStr);
    System.out.println("-- indented string with quotes --");
    //a test string
    //true
    //'     a test string
    //'
    //-- indented string with quotes --
    ~~~
  - n < 0이면, n만큼 공백문자를 제거한다. 제거할 공백이 부족하더라도 공백만 영향을 받고, 다른 문자는 문제가 없다.
    ~~~java
    String str = "  a\n    test\n        string";
    System.out.println(str);
    String indentedStr = str.indent(-6);
    System.out.println("-- negatively indented string --");
    System.out.println(indentedStr);
    //  a
    //    test
    //        string
    //-- negatively indented string --
    //a
    //test
    //  string
    ~~~
  - n == 0 이면, 문자열은 그대로 유지한다. 다만 접미사에 개행문자는 여전히 추가된다.
    ~~~java
    String str = " a test string";
    System.out.println(str);
    String indentedStr = str.indent(0);
    System.out.println("-- negatively indented string --");
    System.out.printf("'%s'%n", indentedStr);
    // a test string
    //-- negatively indented string --
    //' a test string
    //'
    ~~~
- ### `transform(Function<? super String, ? extends R> function)`
  - 이 메소드를 사용하면 람다 함수를 문자열에 적용할 수 있다.
  ~~~java
  String str = "1000";
  Integer integer = str.transform(Integer::parseInt);
  ~~~
## **2. 간결한 숫자 서식**
## **3. Teeing Collector**
## **4. `File::mismatch` 메소드**