# **Java 10 로컬 변수 타입 추론**
## **1. 로컬 변수 타입 추론이란?**
- 로컬 변수의 경우, 실제 타입 대신 특수 예약된 타입명인 **`var`** 를 사용할 수 있다.
- 이러한 새로운 타입 추론 기능을 통해 boiler-plate 코드를 줄일 수 있다.
- 하지만, 컴파일러는 우변을 통해 **`var`** 의 실제 타입을 추론해야 하므로 경우에 따라 제한이 있을 수 있다.
- 결론부터 말하자면, ***우변에 정확한 타입을 지정***해주어야 의도한대로 동작할 수 있다.
- 개인적으로는 코드 가독성을 위해 남용은 지양해야 하지 않나 싶다.

## **2. 단순 타입 추론**
- 다음 예에서 컴파일러는 우변이 문자열 리터럴임을 알 수 있다.
- 다만, Javascript의 **`var`** 키워드와는 다르게, 다른 타입을 재할당 할 수 없다.
- 또한 **`var`** 는 반드시 초기화가 되어야 한다. (`null` 도 안됨)
    ~~~java
    var str = "a test string";
    var substring = str.substring(2);
    System.out.println(substring);  // test string
    System.out.println(substring.getClass().getTypeName()); // java.lang.String

    str = 10;   // compile error!

    var x;          // compile error!
    var y = null;   // compile error!
    ~~~

## **3. 다형성**
- 실제 타입의 다형성과 동일하게 동작한다.
- 하위 타입의 **`var`** 는 상위 타입의 **`var`** 에 할당될 수 있다.
- 상위 타입의 **`var`** 는 하위 타입의 **`var`** 에 할당할 수 없다.
    ~~~java
    var formattedTextField = new JFormattedTextField("some text");
    var textField = new JTextField("other text");
    textField = formattedTextField;
    System.out.println(textField);  // some text

    textField = new JTextField();
    formattedTextField = textField; // compile error!
    ~~~

## **4. Collection 요소의 타입 추론**
- 이 부분이 가장 실수하기가 쉽다.
- 기존 다이아몬드 연산자를 생각하여 우변에 정확한 타입을 명시하지 않으면, 컴파일러가 의도와 다른 타입을 추론할 수 있다.
- 동일한 Collection 요소에 대하여 컴파일러는 타입 추론이 가능하다.
    ~~~java
    var list = Arrays.asList(10);
    System.out.println(list);   // [10]
    int i = list.get(0);    // 컴파일러는 int 타입임을 추론할 수 있다.
    System.out.println(i);  // 10
    ~~~
- 명시적으로 선언하지 않은 상이한 요소 *(설마 이렇게 코딩하지는 않겠지만..)* 에 대하여는 <u>타입들의 공통된 상위 타입들</u>로 추론한다.
    ~~~java
    var list = new ArrayList<>(Arrays.asList(10, "two"));
    // 컴파일러는 int와 String의 공통 타입인, Object & Serializable & Comparable로 추론한다.
    // 즉, List<Object>가 아니라, List<Object & Serializable & Comparable>
    System.out.println(list);   // [10, "two"]
    Object o = list.get(0);         // ok. 10
    Serializable s = list.get(0);   // ok. 10
    Comparable c = list.get(0);     // ok. 10
    Integer i = list.get(0);        // not ok. Integer는 하위 타입 (cast!)
    String s = list.get(1);         // not ok. String은 하위 타입 (cast!)
    list.add(new BigDecimal(10));   // ok. BigDecimal은 Serializable & Comparable 둘 다 구현 (Java 12부터는 안됨)
    list.add(new Date());           // ok. Date는 Serializable & Comparable 둘 다 구현 (Java 12부터는 안됨)
    list.add(Currency.getInstance("USD"));  // compile error! Currency는 Comparable이 구현되지 않음
    ~~~
- 요소가 없고 명시적으로 타입이 지정되지 않은 Collection은 `Object`로 추론한다.
    ~~~java
    var list1 = new ArrayList<>();   // ArrayList<Object>
    list1.add(10);
    int i1 = (int) list1.get(0);    // cast 필요
    ~~~
- 명시적으로 타입이 지정된 Collection은 해당 제네릭 타입으로 컴파일러가 추론한다.
    ~~~java
    var list2 = new ArrayList<Integer>();   // ArrayList<Integer>
    list2.add(10);
    int i2 = list2.get(0);
    ~~~

## **5. for loop**
~~~java
for (var i = 1; i < 4; i++) {
    var z = i * 3;          // int z = i * 3; 과 동일
    System.out.println(z);  // 3, 6, 9
}

var list = Arrays.asList(1, 2, 3);
for (var i : list) {
    var z = i * 3;          // int z = i * 3; 과 동일
    System.out.println(z);  // 3, 6, 9
}
~~~

## **6. Stream**
~~~java
var list = List.of(1, 2, 3, 4, 5, 6, 7);
var stream = list.stream();
stream.filter(i -> i % 2 == 0)
      .forEach(System.out::println);
// 2, 4, 6
~~~

## **7. 삼항 연산자**
- 삼항 연산자의 경우, 반환값이 동일 타입일 경우, 컴파일러는 해당 타입으로 추론한다.
    ~~~java
    public static void main(String[] args) {
        var x = args.length > 0 ? args.length : -1;
        int i = x;  // ok.
    }
    ~~~
- 반환값이 다른 타입일 경우, *(하아.. 진짜 이렇게 코딩하지 말자..)* 위의 Collection과 마찬가지로 해당 타입의 공통 상위 타입들로 추론한다.
    ~~~java
    public static void main(String[] args) {
        var x = args.length > 0 ? arg.length : "no args";
        System.out.println(x);  // 10 or no args
        System.out.println(x.getClass());   // java.lang.Integer or java.lang.String
        x = new Date();     // ok.
        x = Currency.getInstance("USD");    // compile error!
    }
    ~~~
