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
## **2. 간결한 숫자 형식**
- Java 12에는 기존 숫자 형식 API를 확장하여 locale에 해당하는 간결한 숫자 형식을 지원한다.
- ### `CompactNumberFormat` 클래스
  - 10진수에 대한 숫자 형식을 제공하는 `NumberFormat`의 하위 클래스
  - `NumberFormat` 클래스는 `CompactNumberFormat` 인스턴스를 생성할 수 있는 2개의 정적 팩토리 메소드를 제공한다.
    ~~~java
    // default locale with NumberFormat.Style#SHoRT
    public static NumberFormat getCompactNumberInstance();

    // specfied locale and NumberFormat.Style
    public static NumberFormat getCompactNumberInstance(Locale locale, NumberFormat.Style formatStyle);
    ~~~
- ### `NumberFormat.Style` enum 클래스
  - 주어진 `NumberFormat` 인스턴스의 숫자 형식을 지정하기 위한 스타일을 나타낸다.
  - `SHORT`, `LONG` 두 개의 형식을 지원한다.
- 예제
  ~~~java
  NumberFormat short = NumberFormat.getCompactNumberInstance(
    Locale.US, NumberFormat.Style.SHORT);
  String shortFormat = short.format(1000);  // 1K

  NumberFormat long = NumberFormat.getCompactNumberInstance(
    Locale.US, NumberFormat.Style.LONG);
  String longFormat = long.format(1000);    // 1 thousand

  // 대한민국 단위도 가능하다. 다만 SHORT/LONG이 상관없이 동일하다.
  NumberFormat korean = NumberFormat.getCompactNumberInstance(
    Locale.KOREAN, NumberFormat.Style.SHORT);
  String koreanFormat = korean.format(1000);  // 1천
  ~~~
## **3. Teeing Collector**
- Java Stream API의 강력함 중 하나는 다양한 Collector를 지원하는데 있다.
- Java 12에서는 `teeing`이라는 새로운 Collector가 도입되었다.
- ### `Collectors.teeing(`*Collector*, *Collector*, *BiFunction*`)`
  - 두 개의 Collector를 병합하는 기능을 제공한다.
  - 모든 요소는 두 개의 Collector에 의해 처리되고, 그 결과가 병합되어 최종 결과를 반환한다.
    ~~~java
    double mean = Stream.of(1, 2, 3, 4, 5)
				.collect(Collectors.teeing(Collectors.summingDouble(i -> i), 
						Collectors.counting(), (sum, count) -> sum / count));
    // 3.0
    ~~~
## **4. `Files::mismatch` 메소드**
- `nio.file.Files` 클래스에 두 개의 파일을 비교하는 새로운 메소드가 추가되었다.
- ### `public static long mismatch(`*Path*, *Path*`) throws IOException`
  - 두 파일을 비교하고 내용이 일치하지 않는 첫번째 바이트의 인덱스 위치를 반환한다.
  - 파일이 동일한 경우, -1L을 반환한다.
    ~~~java
    Path path1 = Files.createTempFile("file1", "txt");
    Path path2 = Files.createTempFile("file2", "txt");
    Files.writeString(path1, "Java 12 Article");
    Files.writeString(path2, "Java 12 Article");

    long mismatch1 = Files.mismatch(path1, path2);
    // -1L

    Path path3 = Files.createTempFile("file3", "txt");
    Path path4 = Files.createTempFile("file4", "txt");
    Files.writeString(path3, "Java 12 Article");
    Files.writeString(path4, "Java 12 Tutorial");

    long mismatch2 = Files.mismatch(path3, path4);
    // 8L
    ~~~