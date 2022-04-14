# **Java 9 모듈 시스템**
## **1. Java 모듈이란?**
- Java 패키지가 클래스들의 그룹이라면, Java 모듈은 패키지들의 그룹.
- 기존 패키지 구조에서는 클래스/메소드 레벨에서 공개수준을 가질 수 있었지만, 패키지 레벨의 공개수준은 가질 수 없었다.
- 모듈은 이러한 패키지 구조와는 다른 수준의 Java 아티팩트 그룹화를 위해 도입.
- 이를 통해, `private`한 패키지를 가질 수 있다.
- 모듈은 소스 코드 루트경로에 `module-info.java` 파일을 추가하여 선언.
    ~~~java
    module abc.xyz {
        ...
    }
    ~~~
## **2. `exports`문**
- 외부에 보여지기 위한 패키지를 `module-info.java`파일에 `exports`문을 통해 지정할 수 있다.
    ~~~java
    module abc.xyz {
        exports com.foo.bar; // package
    }
    ~~~
- 주어진 패키지의 클래스가 `public`이더라도, 해당 패키지가 `exports`문을 통해 명시적으로 내보내지지 않으면, 외부에서 (컴파일/런타임 모두) 볼 수 없다.
- 이러한 특성들로 모듈은 공개 기능을 숨길 수 있으며, Java의 캡슐화 시스템을 향상시킨다.

## **3. `requires`문**
- 내보내진 패키지를 사용하려는 다른 모듈은 `module-info.java` 파일에 `requires` 키워드를 사용하여 대상 모듈을 가져올 수 있다.
    ~~~java
    module def.stu {
        requires abc.xyz;
    }
    ~~~

## **4. `exports ... to`문**
- 내보내진 패키지를 `requires`할 수 있는 모듈을 지정할 수 있다. 해당 모듈 외에는 이 패키지를 가져올 수 없다.
    ~~~java
    module abc.xyz {
        exports com.foo.bar to def.stu;
    }
    module def.stu {
        requires abc.xyz;   // O
    }
    module ghi.opq {
        requires abc.xyz;   // X
    }
    ~~~
- 더 자세한 내용은 Baeldung의 [A Guide to Java 9 Modularity](https://www.baeldung.com/java-9-modularity) 참고