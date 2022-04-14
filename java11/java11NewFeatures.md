# **Java 11 새로 추가된 기능**
## **1. `String`**
- ### `strip()`
  - 공백 문자를 제거해준다.
  - `trim()`과 차이점은 `strip()`은 유니코드 공백문자도 제거 가능
  - `strip()` : 앞뒤 공백제거
  - `stripLeading()` : 앞 공백제거
  - `stringTrailing()` : 뒤 공백제거
  ~~~java
  String str = "   test string   ";

  String striped = str.strip();  // "test string"
  striped = str.stripLeading();  // "test string   "
  striped = str.stripTrailing(); // "   test string"
  ~~~
- ### `isBlank()`
  - 빈 문자열 또는 공백문자 포함 여부
- ### `Stream<String> lines()`
  - 개행문자를 구분자로 문자열 순차 스트림 생성
  ~~~java
  String s = "one\ntwo\nthree";
  s.lines().forEach(System.out::println);
  // one
  // two
  // three
  ~~~
- ### `repeat(`*int*`)`
  - 인자값만큼 반복되는 문자열을 반환
  ~~~java
  String s1 = "-";
  String s2 = s1.repeat(5); // -----
  ~~~

## **2. `java.nio.file.Files`**
- 문자열을 파일에서 직접 읽거나 쓰는 메소드가 추가되었다.
  - `public static String readString(Path path) throws IOException`
  - `public static Path writeString(Path path, CharSequence csq, OpenOption... options) throws IOException`
  ~~~java
  Path path = Files.writeString(
      Files.createTempFile("test", ".txt"),
      "test file content");
  System.out.println(path); // test2314231...34.txt
  String content = Files.readString(path);
  System.out.println(content); // test file content
  ~~~
  - `public static String readString(Path path, Charset cs) throw IOException`
  - `public static String writeString(Path path, CharSequence csq, Charset cs, OpenOption... options) throws IOException`
  ~~~java
  Charset utf8Charset = StandardCharsets.UTF_8;
  Path path = Files.writeString(
      Files.createTempFile("test", ".txt"),
      "테스트 내용",
      utf8Charset);
  System.out.println(path); // test2314231...34.txt
  String content = Files.readString(path, utf8Charset);
  System.out.println(content); // 테스트 내용
  ~~~