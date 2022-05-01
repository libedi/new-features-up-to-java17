# **Java 16 Record**
- ## **도입배경**
  - 우리는 일반적으로 데이터의 전달 목적만을 갖고 있는 DTO라는 이름의 클래스들을 많이 사용한다.
  - 많은 경우, 이러한 데이터는 여러 장점으로 인해 [불변 객체](https://www.baeldung.com/java-immutable-object)로 만들어 사용한다.
  - 이러한 불변 객체는 다음과 같은 방법으로 만들 수 있다.
    1. 데이터는 **`private final` 필드**로 선언
    2. 각 필드의 값을 반환하는 **`getter` 메소드**
    3. **모든 필드를 인수로 갖는 `public` 생성자**
    4. **모든 필드가 일치할 때**, 동일한 객체에 대해 `true`를 반환하는 **`equals` 메소드**
    5. **모든 필드가 일치할 때**, 동일한 값을 반환하는 **`hashCode` 메소드**
    6. 클래스의 이름, 각 필드의 이름과 해당하는 값을 포함하는 **`toString` 메소드**
  - 위 방법으로 다음과 같은 불변 클래스를 작성할 수 있다.
    ~~~java
    public final class Person {
        private final String name;
        private final String gender;
        private final int age;

        public Person(String name, String gender, int age) {
            this.name = name;
            this.gender = gender;
            this.age = age;
        }

        @Override
        public boolean equals(Object obj) {
            if (this == obj) {
                return true;
            } else if (!(obj instanceof Person)) {
                return false;
            }
            Person person = (Person) obj;
            return age == person.age &&
                Objects.equals(name, person.name) &&
                Objects.equals(gender, person.gender);
            
        }

        @Override
        public int hashCode() {
            return Objects.hash(name, gender, age);
        }

        @Override
        public void toString() {
            return "Person[name=" + name + 
                    ", gender=" + gender +
                    ", age=" + age + "]";
        }

        // getters...
    }
    ~~~
  - 위 방식은 두가지 문제점을 갖고 있다.
    1. **너무 많은 boiler-plate 코드** : 각 데이터 클래스에 대해 동일하고 지루한 프로세스를 반복해야 한다. 또한 필드가 추가/삭제될 경우, 자동으로 업데이트를 하지 못할 위험성을 갖고 있다.
    2. **명시적이지 않은 데이터 클래스** : 작성한 클래스가 `name`, `gender`, `age` 필드가 있는 단순한 데이터 클래스인지 명시적이지 않다.
  - 더 나은 접근 방법은 데이터 클래스임을 명시적으로 선언함과 동시에 반복되는 boiler-plate 코드를 줄이는 것이다.
- ## **Record 사용하기**
  - Record는 필드의 타입과 이름만 필요로 하는 불변 데이터 클래스이다.
  - `equals`, `hashCode`, `toString` 메소드와 `private final` 필드, `public` 생성자는 컴파일러에 의해 생성된다.
  - 우리는 **`record`** 키워드를 사용하여 위의 코드를 다음과 같이 다시 작성할 수 있다.
    ~~~java
    public record Person(String name, String gender, int age) {
    }
    ~~~
  - 단 두 줄로 줄였다. 이렇게 하면 훨씬 더 쉽고, 오류 발생을 줄여준다.
  - 주의할 점은, `getter` 메소드가 `getXXX`가 아닌, 필드명과 동일한 메소드명을 갖고 있다는 점이다.
    ~~~java
    public static void main(String[] args) {
        Person person = new Person("libedi", "male", 40);
        System.out.println("name is " + person.name());
        System.out.println("gender is " + person.gender());
        System.out.println("age is " + person.age());
    }
    ~~~
- ## **Record의 생성자**
  - 생성자가 생성되는 동안, 생성자 구현을 커스터마이징할 수 있다.
    ~~~java
    public record Person(String name, String address) {
        public Person {
            Objects.requireNonNull(name);
            Objects.requireNonNull(address);
        }
    }
    ~~~
  - 이러한 커스터마이징은 유효성 검사에 사용하기 위한 것이며, 가능한한 간단하게 유지해야 한다.
  - 다른 인수 갯수를 갖는 생성자를 만들 수도 있다.
    ~~~java
    public record Person(String name, String address) {
        public Person(String name) {
            this(name, "Unknown");
        }
    }
    ~~~
  - 자동으로 생성되는 `public` 생성자와 동일한 생성자를 생성할 수 있지만, 각 필드를 수동으로 초기화해주어야 한다.
    ~~~java
    public record Person(String name, String address) {
        public Person(String name, String address) {
            this.name = name;
            this.address = address;
        }
    }
    ~~~
  - 생성자 구현과 인수 갯수가 동일한 생성자를 선언하면 컴파일 오류가 발생한다.
    ~~~java
    public record Person(String name, String address) {
        public Person {
            Objects.requireNonNull(name);
            Objects.requireNonNull(address);
        }
        
        // 컴파일 오류!
        public Person(String name, String address) {
            this.name = name;
            this.address = address;
        }
    }
    ~~~
- ## **Record의 특징**
  - ### **Record는 다른 클래스를 상속받을 수 없다.**
    - 위 코드를 컴파일하면 이미 `java.lang.Record` 클래스를 상속받은 것을 알 수 있다.
      ~~~shell
      $ javap Person.class
      Compiled from "Person.java"
      public final class io.github.libedi.Person extends java.lang.Record {
          public final java.lang.String toString();
          public final int hashCode();
          public final boolean equals(java.lang.Object);
          public java.lang.String name();
          public java.lang.String gender();
          public int age();
      }
      ~~~
  - ### **Record는 final이다.**
    - Record는 `final` 클래스이기 때문에 다른 클래스에 상속할 수 없다.
  - ### **Record는 추상적일 수 없다.**
    ~~~java
    public abstract record Person(String name) {
        // error: modifier abstract not allowed here
    }
    ~~~
  - ### **Record는 추가적인 인스턴스 필드를 가질 수 없다.**
  - ### **Record는 추가적인 메소드는 허용된다.**
  - ### **Record는 인터페이스를 구현할 수 있다.**
    ~~~java
    public interface Address {
        String address();
    }
    public record Person(String name) implements Address {
        // 인터페이스 구현을 위해 추가적인 메소드가 허용된다.
        @Override
        public String address() {
            return "address";
        }
    }
    ~~~
  - ### **Record의 구성요소들은 암묵적으로 `final`이다.**
    ~~~java
    public record Person(String name) {
        public void changeName(String newName) {
            this.name = newName;    // 컴파일 에러! name은 final.
        }
    }
    ~~~
  - ### **Record는 `static` 필드를 가질 수 있다.**
    ~~~java
    public record Person(String name) {
        private static int count = 10;

        public static void main(String[] args) {
            System.out.println(Person.count);   // 10
        }
    }
    ~~~
  - ### **`static` 메소드, `static` 필드, `static` 초기화 모두 허용된다.**
    ~~~java
    public record Person() {
        static int count;
        static {
            count = 10;
        }
        public static void showCount() {
            System.out.println(count);
        }
        public static void main(String[] args) {
            Person.showCount();
        }
    }
    ~~~
  - ### **자동 생성된 메소드들을 명시적으로 생성할 수 있다.**
    ~~~java
    public record Person(String name) {
        public String name() {
            return "name is " + name;
        }
    }
    ~~~
  - ### **중첩된 Record가 허용된다.**
    - 중첩된 Record는 암묵적으로 `static`이 된다.
    ~~~java
    public class MyClass {
        public record Person(String name) {

        }

        public static void main(String[] args) {
            String modifier = Modifier.toString(Person.class.getModifiers());
            System.out.println(modifier);
            // public static final
        }
    }
    ~~~
  - ### **Generic Record도 허용된다.**
    ~~~java
    public record Line<N extends Number>(N length) {
        public static void main(String[] args) {
            Line<Double> line = new Line<>(5.3);
            System.out.println(line);
        }
    }
    ~~~
  - ### **Record에는 annotation을 달 수 있다.**
    ~~~java
    @JavaBean
    public record Person(@Nonnull String name) {

    }
    ~~~
- ## 결론
  - 우리가 자주 사용하는 DTO나 JPA의 VO를 생성할 때 매우 유용할 것 같다.
  - 다양한 사용법이 있지만, 대부분 기본 사용법을 사용할 것 같고, 그것으로도 충분히 매력적이다.