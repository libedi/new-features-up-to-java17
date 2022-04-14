# **Java 10 새로 추가된 기능**
## **1. `Collection`**
- 수정할 수 없는 `Collection`을 만드는 정적 메소드가 추가되었다.
- **`List.copyOf(`*Collection*`)`**
    ~~~java
    List<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    List<Integer> integers = List.copyOf(list); // [1, 2]
    ~~~
- **`Set.copyOf(`*Collection*`)`**
    ~~~java
    List<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    Set<Integer> integers = Set.copyOf(list);   // [2, 1]
    ~~~
- **`Map.copyOf(`*Map*`)`**
    ~~~java
    Map<Integer, String> map = new HashMap<>();
    map.put(1, "one");
    map.put(2, "two");
    Map<Integer, String> map2 = Map.copyOf(map); // {2=two, 1=one}
    ~~~
- **`List.copyOf()` vs `Collections.unmodifiableList()`**
  - `Collections.unmodifiableList()`가 원본 `List`의 참조를 유지하는데 반해, `List.copyOf()`는 새로운 `List`를 반환한다.
  ~~~java
  List<Integer> list = new ArrayList<>();
  list.add(1);
  list.add(2);
  List<Integer> unmodifiableList = Collections.unmodifiableList(list);
  List<Integer> copyOfList = List.copyOf(list);
  list.add(3);
  System.out.println("unmodifiableList: " + unmodifiableList);
  System.out.println("copyOfList: " + copyOfList);
  // unmodifiableList: [1, 2, 3]
  // copyOfList: [1, 2]
  ~~~

  ## **2. Stream**
  - `Collectors`에 수정할 수 없는 `Collection`을 반환하는 메소드가 추가되었다.
  - **`Collectors.toUnmodifiableList()`**
  - **`Collectors.toUnmodifiableSet()`**
  - **`Collectors.toUnmodifiableMap(Function keyMapper, Function valueMapper)`**
  - **`Collectors.toUnmodifiableMap(Function keyMapper, Function valueMapper, BinaryOperator mergeFunction)`**
  