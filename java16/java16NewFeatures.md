# **Java 16 새로 추가된 기능**
## **1. Pattern Matching for `instanceof`**
  - Java 14에서 Preview로 소개된 기능이 Java 16에 정식으로 포함되었다.
  - `instanceof`는 이전에는 다음과 같이 작성하였다.
    ~~~java
    Object obj = "TEST";

    if (obj instanceof String) {
        String s = (String) obj;
        // ...
    }
    ~~~
  - 이 코드는 먼저 `obj` 인스턴스가 `String`인지 확인한 다음, `String`으로 캐스팅하고, 새 변수 `s`에 할당해야 한다.
  - 이를 패턴 매칭을 도입하여 다음과 같이 작성할 수 있다.
    ~~~java
    Object obj = "TEST";

    if (obj instanceof String s) {
        // ...
    }
    ~~~
  - 이제 `instanceof` 검사의 일부로 변수 선언을 할 수 있다.
  - 이는 다음과 같이 조건도 적용이 가능하다.
    ~~~java
    if (obj instanceof String s && s.length() < 5) {
        // ...
    }
    ~~~
## **2. `Stream::toList` 메소드 추가**
  - `Stream`에서 정말 많이 사용하게 되는 `.collect(Collectors.toList())`의 boiler-plate 코드를 이제 `.toList()`로 줄일 수 있다.
    ~~~java
    List<String> stringList = Arrays.asList("1", "2", "3");

    List<Integer> intList1 = stringList.stream()
                                       .map(Integer::parseInt)
                                       .collect(Collectors.toList());
    List<Integer> intList2 = stringList.stream()
                                       .map(Integer::parseInt)
                                       .toList();
    ~~~
  - `intList1`과 `intList2`는 동일한 결과를 갖는다.