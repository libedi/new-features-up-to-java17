# **Java 9 New Features**
## **1. Try-With-Resources 개선**
- Java 7/8에서는 다음과 같이 `try`문 안에서만 리소스 선언이 가능했다.
    ~~~java
    try (InputStream stream1 = new InputStream(...);
         InputStream stream2 = new InputStream(...)) {
        ....
    }
    ~~~
- Java 9에서는 더이상 `try`문 안에 선언할 필요없이 리소스의 ***참조만 사용하면 된다.***
    ~~~java
    InputStream stream1 = new InputStream(...);
    InputStream stream2 = new InputStream(...);

    try (stream1; stream2) {
        ....
    }
    ~~~
## **2. Collection 변경사항**
### 1. `Set`
- 불변객체를 간편하게 생성하는 팩토리 메소드 제공
    ~~~java
    Set<Integer> integers = Set.of(1, 2, 3, 4);
    ~~~
- 가능하면 10개 안에서 생성하자. 10개를 넘어가면 가변인자를 사용한다.
    ~~~java
    of() // empty set
    of(E)
    of(E, E)
    ...
    of(E, E, E, E, E, E, E, E, E, E) // 10개 요소까지
    of(E...) // 가변인자
    ~~~
### 2. `List`
- 불변객체를 간편하게 생성하는 팩토리 메소드를 제공
    ~~~java
    List<Integer> integers = List.of(1, 2, 3, 4);
    ~~~
- 가능하면 10개 안에서 생성하자. 10개를 넘어가면 가변인자를 사용한다.
### 3. `Map`
- 불변객체를 간편하게 생성하는 팩토리 메소드를 제공
    ~~~java
    Map<Integer, String> map1 = Map.of(1, "one", 2, "two");
    Map<Integer, String> map2 = Map.ofEntries(
                                    Map.entry(1, "one"),
                                    Map.entry(2, "two")
                                    );  // 가변인자
    ~~~
- of() 메소드는 최대 10개의 Key-Value Pair를 제공한다.
    ~~~java
    of(K, V, K, V, K, V, K, V, K, V, K, V, K, V, K, V, K, V, K, V) // 10 pairs
    // 내부적으로 2 pairs부터 가변인자를 사용하여 생성한다.
    ~~~
### 4. `Arrays`
- `Arrays.mismatch()`
  - 두 배열 요소 중 일치하지 않는 첫번째 인덱스를 반환. (인덱스 기준은 첫번째 배열)
  - 없으면 -1 반환.
    ~~~java
    int[] arr1 = {1, 2, 3, 4, 5};
    int[] arr2 = {1, 2, 3, 5, 6};
    int i = Arrays.mismatch(arr1, arr2); // i == 3
    ~~~
  - 두 배열의 크기가 다를 경우, 오버라이드 메소드를 사용하여 검색 인덱스 지정
  - `int mismatch(int[] a, int aFromIndex, int aToIndex, int[] b, int bFromIndex, int bToIndex)`
    ~~~java
    int[] arr1 = {-2, 1, 2, 3, 4, 5};
    int[] arr2 = {-1, 0, 1, 2, 3, 5, 6, 7};
    int i = Arrays.mismatch(arr1, 1, arr1.length, arr2, 2, 6);  // i == 4
    ~~~
  
- `Arrays.equals()`
  - 두 배열의 요소가 동일한지 비교.
    ~~~java
    int[] arr1 = {1, 2, 3};
    int[] arr2 = {1, 2, 3};
    boolean b = Arrays.equals(arr1, arr2); // b == true
    ~~~
  - 배열의 요소 중, 비교할 검색 인덱스 지정 가능
  - `boolean equals(int[] a, int aFromIndex, int aToIndex, int[] b, int bFromIndex, int bToIndex)`
    ~~~java
    int[] arr1 = {0, 1, 2, 3};
    int[] arr2 = {-2, -1, 1, 2, 3, 4};
    boolean b = Arrays.equals(arr1, 1, arr1.length, arr2, 2, 4); // b == true
    ~~~

## **3. Stream 신규 메소드**
### 1. `Stream.takeWhile(`*Predicate*`)`
- `Predicate`의 조건이 `false`가 되기 이전의 스트림을 반환.
    ~~~java
    String[] fruits = {"apple", "banana", "orange", "mango", "peach"};
    Stream<String> stream = Arrays.stream(fruits).takeWhile(s -> !"orange".equals(s));
    // ["apple", "banana"]
    ~~~

### 2. `Stream.dropWhile(`*Predicate*`)`
- `Predicate`의 조건이 `false`를 포함한 이후의 스트림을 반환.
- `Stream.takeWhile()`가 반대되는 메소드.
    ~~~java
    String[] fruits = {"apple", "banana", "orange", "mango", "peach"};
    Stream<String> stream = Arrays.stream(fruits).dropWhile(s -> !"orange".equals(s));
    // ["orange", "mango", "peach"]
    ~~~

### 3. `Stream<T> Stream.iterate(`*T, Predicate, UnaryOperator*`)`
- `Predicate`의 조건을 만족하는 순차 스트림을 생성하여 반환.
    ~~~java
    Stream<String> iterate = Stream.iterate("-", s -> s.length() < 5, s -> s+ "-");
    // [-, --, ---, ----]
    
    // Predicate가 없으면 무한 스트림 생성
    Stream<String> infinite = Stream.iterate("-", s -> s + "-");
    ~~~

### 4. `Stream<T> Stream.ofNullable(`*T*`)`
- `Optional.ofNullable()`처럼 단일 요소의 순차 스트림을 반환.
- 요소가 `null`인 경우, 빈 스트림을 반환.
- 이 메소드는 단일 요소의 스트림 추가시 유용할 수 있다.
    ~~~java
    Stream<String> stream1 = Stream.of("apple", "banana");

    String nullableElement1 = "orange";
    Stream<String> stream2 = Stream.concat(stream1, Stream.ofNullable(nullableElement1));
    // ["apple", "banana", "orange"]

    String nullableElement2 = null;
    Stream<String> stream3 = Stream.concat(stream1, Stream.ofNullable(nullableElement2));
    // ["apple", "banana"]
    ~~~

## **4. `java.util.Optional` 개선**
- **java 8에서 `Optional`을 사용할 때 다소 아쉬운 부분이 있었는데, java 9에서는 그러한 부분의 많은 기능 추가가 있었다.**

### 1. `Optional.ifPresentOrElse(`*Consumer, Runnable*`)`
- 값이 존재하면 `Consumer`를 실행하고, 없으면 `Runnable`을 실행한다.
- java 8에서 가장 아쉬웠던 부분이었는데, 가장 유용한 기능 중의 하나가 아닐까 한다.
    ~~~java
    IntStream.of(1, 2, 4)
             .filter(i -> i % 3 == 0)
             .findFirst()
             .ifPresendOrElse(System.out::println, () -> {
                 System.out.println("No multiple of 3 found");
             });
             // No multiple of 3 found
    ~~~

### 2. `Optional.stream()`
- 값이 존재하면 순차 스트림을 반환하고, 없으면 빈 스트림을 반환한다.
- 이 메소드는 `Optional` 스트림을 스트림으로 변환할 때 사용될 수 있다.
    ~~~java
    Stream<Optional<T>> os = ...
    Stream<T> s = os.flatMap(Optional::stream);
    ~~~

### 3. `Optional.or(`*Supplier*`)`
- 값이 없을 때, `Supplier`를 통해 새로운 `Optional`을 지연 생성할 수 있다.
- `orElseGet()`과 유사하지만, `orElseGet()`이 값을 반환하는 반면, `or()`는 `Optional`의 인스턴스를 반환하는 차이가 있다.
    ~~~java
    char result = Stream.of('a', 'b', 'c')
                        .filter(Character::isDigit)
                        .findFirst()
                        .or(() -> Optional.of('0'))
                        .get();
    // '0'
    ~~~

## **5. `java.util.Objects` 신규 메소드**
