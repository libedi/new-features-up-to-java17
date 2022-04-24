# **Java 9 인터페이스 private 메소드 추가**
- ## **도입배경**
  - Java 8부터 인터페이스에 `static` 메소드와 `default` 메소드가 도입되었다.
  - 하지만, `private` 메소드가 없어서 `static`과 `default` 메소드 구현시 중복된 코드가 발생하거나, 중복 코드를 없애기 위해 구현에 해당하는 부분을 불필요하게 노출시켜야 하는 경우가 발생한다.
  - 이는 재사용성을 떨어뜨리고 캡슐화를 오염시킨다.
    ~~~java
    public interface MyInterface {
        // 중복 코드 발생
        static void method1() {
            System.out.println("This is static method in Interface"); //
            System.out.println("method number 1");
        }
        static void method2() {
            System.out.println("This is static method in Interface"); //
            System.out.println("method number 2");
        }
        // 구현부분을 불필요하게 노출
        default void method3() {
            printDefaultMethod();
            System.out.println("method number 3");
        }
        default void method4() {
            printDefaultMethod();
            System.out.println("method number 4");
        }
        default void printDefaultMethod() {
            System.out.println("This is default method in Interface");
        }
    }
    ~~~
  - Java 9부터는 인터페이스에 `private` 메소드가 도입되어 재사용성을 높였고, 불필요한 메소드 노출을 방지해 캡슐화를 지켜준다.
- ## **특징**
  1. 메소드 body가 있고, 추상 메소드가 될 수 없다.
  2. static 이거나 non-static 일 수 있다.
  3. 인터페이스 내에서 사용할 수 있다.
  4. 인터페이스에서 다른 메소드를 호출할 수 있다.
- ## **`private static` method**
  - `static` 메소드답게 `static` 또는 `private static` 메소드만 호출할 수 있다.
    ~~~java
    public interface MyInterface {

        static void method1() {
            method2();
        }

        private static void method2() {
            method3();  // static 메소드 호출
            method4();  // private static 메소드 호출
        }

        static void method3() {}
        private static void method4() {}
    }
    ~~~
- ## **`private` method**
  - `private` 메소드는 `private`, `default`, `abstract`, `static` 메소드를 호출할 수 있다.
    ~~~java
    public interface MyInterface {

        void method1();

        default void method2() {
            method3();
        }

        private void method3() {
            method1();  // abstract 메소드 호출
            method4();  // private 메소드 호출
            method5();  // default 메소드 호출
            method6();  // static 메소드 호출
            mehotd7();  // private static 메소드 호출
        }

        private void method4() {}
        default void method5() {}
        static void method6() {}
        private static void method7() {}
    }
    ~~~