# **Java 11 Lambda 식 파라미터에 `var` 사용**
- **lambda 식의 파라미터에 `var`를 사용할 수 있다.**
    ~~~java
    IntStream.of(1, 2, 3, 4, 5, 6, 7)
             .filter((var i) -> i % 3 == 0)
             .forEach(System.out::println);
             // 3, 6
    ~~~
- lambda 식에 `var`를 사용하는 장점
  - [Type Annotaion](https://www.logicbig.com/tutorials/core-java-tutorial/java-8-enhancements/type-annotations.html)을 사용할 수 있다.
    ~~~java
    (@Nonnull var x, @Nullable var y) -> x.process(y)
    ~~~
  - `final`을 사용할 수 있다.
    ~~~java
    (final var x) -> Math.pow(x, 4)
    ~~~
- Type Annotaion을 사용할 경우가 아니라면, 개인적으로 큰 장점은 안 느껴진다.
- 게다가 어차피 람다식의 파라미터는 자유변수로서 암묵적으로 `final`로 선언된다.