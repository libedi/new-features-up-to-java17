# **Java 15 새로 추가된 기능**
## **1. 텍스트 블록**
- +로 연결할 필요없이 여러 줄 문자열 리터럴을 작성할 수 있는 방법
- 이스케이프 문자 없이 줄바꿈과 따옴표를 자유롭게 사용할 수 있다.
- Java 13 및 14에서 Preview 기능으로 제공하던 텍스트 블록이 Java 15부터 표준 기능이 되었다.
- 큰 따옴표 3개(""")로 열고, 다음 줄부터 문자열의 내용이 시작한다.
- 여는 방법과 동일하게 큰 따옴표 3개로 닫는다.
- 각 줄의 마지막 공백들은 무시된다.
  ~~~java
  String str = """
      <html>
        <body>
          <div class=""></div>
        </body>
      </html>
      """;
  System.out.println(str);
  //<html>
  //  <body>
  //    <div class=""></div>
  //  </body>
  //</html>
  ~~~
- 들여쓰기는 여는 큰 따옴표 이후의 다음 라인들 중 *(닫는 큰 따옴표도 포함)*, 가장 좌측 공백이 적은 라인을 기준으로 한다.
  ~~~java
  String str1 = """
        <html>
            <body>
              <div class=""></div>
            </body>
        </html>
      """;  // 닫는 큰 따옴표가 가장 좌측임
  System.out.println(str1);
  //  <html>
  //    <body>
  //      <div class=""></div>
  //    </body>
  //  </html>
  String str2 = """
      <html>
    <body>
          <div class=""></div>
        </body>
      </html>""";   // <body>가 가장 좌측임
  System.out.println(str2);
  //  <html>
  //<body>
  //      <div class=""></div>
  //    </body>
  //  </html>
  ~~~
- 개행을 무시하려면 줄 마지막에 이스케이프 문자를 추가해준다.
  ~~~java
  String str = """
      This is \
      Text Blocks Example.
      """;
  System.out.println(str);
  //This is Text Blocks Example.
  ~~~
- 개인적으로 이 기능은 Spring-jdbc의 JdbcTemplate에서 SQL작성시 강력한 효과를 낼 것으로 보인다.
## **2. 유용한 `NullPointerException`**
- Java 14에서 도입된 기능으로 Java 14에서는 기본적으로 disable 상태였지만, Java 15부터 기본 지원되었다.
- 기존의 `NullPointerException`은 (이후 NPE) 예외가 발생한 위치는 표시하지만, 정확히 어떤 변수나 표현식이 `null`인지 확인하기 어려웠다.
- 유용한 NPE는 이를 해결하기 위한 목적으로, 어떤 변수나 표현식이 `null`인지 설명하여 *JVM에 의해 생성된* ***NPE***의 가독성을 향상시킨다.
  ~~~shell
  Exception in thread "main" java.lang.NullPointerException: Cannot invoke "String.toUpperCase()" because "str" is null
	at HelpfulNPETest.printUpperCase(HelpfulNPETest.java:8)
	at HelpfulNPETest.main(HelpfulNPETest.java:4)
  ~~~
- ### 특징
  - 이러한 메시지 계산은 JVM이 NPE를 throw할 경우에만 수행된다. Java 코드에서 명시적으로 예외를 throw하면 충분히 의미있는 메시지를 전달할 수 있으므로, 위 작업이 수행되지 않는다.
  - 메시지는 lazy하게 계산된다. 즉, 메시지가 예외가 발생할 때가 아닌, print될 때만 계산되므로 성능에 영향이 없도록 개발자가 다루어야 한다.
## **3. 새로운 Garbage Collectors**
- ZGC, Shenandoah GC를 정식으로 사용할 수 있다.
- 기본 설정은 G1이다.