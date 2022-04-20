# **Reactive Streams and Java 9 Flow API**
## **1. Reactive Streams**
- ***스트림***은 시간이 지남에 따라 생성되는 일련의 데이터/이벤트/신호 이다.
  - Java 8의 `Stream` API와 혼동하지 말자.
  - Reactor에서는 *시퀀스*라고 표현하기도 한다.
- Reactive Streams는 이러한 스트림의 비동기 처리를 위한 표준적인 방식을 제안한다.
- 그러니까 일종의 API 규격, 스펙, 명세이다.
- 이것은 [Reactive 선언문](https://www.reactivemanifesto.org/ko)에서 소개한 Reactive System 구축을 위해 제안된 표준 API로서,
- 구현체로는 RxJava, Reactor, 그리고 ***Java 9의 Flow*** 등이 있다.
- 해당 스펙을 다이어그램으로 표현하면 아래와 같다.
  ![Reactive Streams API Diagram](https://github.com/libedi/new-features-up-to-java17/blob/main/img/reactive-pub-sub-1024x463.png?raw=true)
  ##### *출처 : https://www.getoutsidedoor.com/2020/11/23/reactive-streams-%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C/*
  - 여기서 Publisher는 데이터 소스(Producer)라고 볼 수 있고, Subscriber는 해당 데이터를 사용하는 클라이언트(Consumer)라고 볼 수 있다.
  - Subscription은 Publisher가 생성하고, 구독을 위해 Subscriber에게 전달된다.
  - Subscription은 Subscriber의 요청만큼 Publisher의 데이터를 Subscriber에게 전달한다. 구독의 행위에 해당.
  - Publisher는 구독하고자 하는 Subscriber를 `subscribe(Subscriber)` 메소드를 통해 구독을 시작한다.
  - Publisher는 자신이 생성하고 관리하는 Subscription을 Subscriber에게 `onSubscribe(Subscription)`을 통해 전달한다.
  - Subscription은 `request(long)` 메소드를 통해 Subscriber의 요청만큼 Publisher의 데이터를 Subscriber의 `onNext(Object)`를 통해 전달한다.
    - 도중에 오류가 발생하면 Subscriber의 `onError(Throwable)` 메소드를 호출하여 오류를 전달한다.
    - 데이터 전달이 완료되면 Subscriber의 `onComplete()` 메소드를 호출하여 완료되었음을 전달한다.
  - 여기서 `request(long)`가 Back Pressure에 해당한다.
  - Subscriber는 Subscription의 `request(long)` 메소드를 호출하여 Publisher의 데이터를 요청한다.
  - Subscriber는 수신한 데이터를 처리하기 위한 `onNext(Object)`, `onError(Throwable)`, `onComplete()` 메소드를 구현한다.
- Reactive System, Observer 패턴 등 이론적인 부분은 추후 정리하고, 여기서는 **`Flow`** 의 사용법 위주로 알아본다.
## **2. Java 9 `Flow` API**
  - Java 9에서는 Reactive Streams를 정의하는 인터페이스들을 제공한다.
  - 해당 인터페이스들은 `java.util.concurrent.Flow` 인터페이스의 정적 중첩 클래스로 정의되어 있다.
  - ### `Flow.Subscriber` 인터페이스
    ~~~java
    public static interface Subscriber<T> {
      public void onSubscribe(Subscription subscription);
      public void onNext(T item);
      public void onError(Throwable throwable);
      public void onComplete();
    }
    ~~~
  - ### `Flow.Publisher` 인터페이스
    ~~~java
    @FunctionalInterface
    public static interface Publisher<T> {
      public void subscribe(Subscriber<? super T> subscriber);
    }
    ~~~
  - ### `Flow.Subscription` 인터페이스
    ~~~java
    public static interface Subscription {
      public void request(long n);
      // 구독자가 구독을 취소. 호출 후에는 더이상 item이 수신되지 않음.
      public void cancel();
    }
    ~~~
  - ### `Flow.Processor` 인터페이스
    - Subscriber이자 Publisher 역할을 하는 컴포넌트
    ~~~java
    public static interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
    }
    ~~~
## **3. 샘플코드**
  - ### `IntegerListPublisher`
    - `Integer` 타입의 리스트 데이터를 갖고 있는 Publisher
    ~~~java
    public class IntegerListPublisher implements Publisher<Integer> {
      private final List<Integer> data;
      
      public IntegerListPublisher(List<Integer> data) {
        this.data = data;
      }

      @Override
      public void subscribe(Subscriber<? super Integer> subscriber) {
        subscriber.onSubscribe(new Subscription() {

          @Override
          public void request(long n) {
            try {
              for (int i=0,length=Integer.min(data.size(), (int) n); i<length; i++) {
                // item 전달
                subscriber.onNext(data.get(i));
              }
            } catch (Exception e) {
              // 오류 전달
              subscriber.onError(e);
            }
            // 완료 전달
            subscriber.onComplete();
          }

          @Override
          public void cancel() {
            // 메시지 수신 취소
          }
        });
      }
    }
    ~~~
  - ### `IntegerSubscriber`
    - `Integer` 타입의 데이터를 수신하는 Subscriber
    ~~~java
    public class IntegerSubscriber implements Subscriber<Integer> {
      private final long requestCount;
      private Subscription subscription;

      public IntegerSubscriber(long requestCount) {
        this.requestCount = requestCount;
      }

      @Override
      public void onSubscribe(Subscription subscription) {
        this.subscription = subscription;
        this.subscription.request(requestCount);
      }

      @Override
      public void onNext(Integer item) {
        System.out.println("Received data: " + item);
      }

      @Override
      public void onError(Throwable throwable) {
        System.out.println("Error: " + throwable.getMessage());
      }

      @Override
      public void onComplete() {
        System.out.println("Completed!");
      }
    }
    ~~~
  - ### **main**
    ~~~java
    public static void main(String[] args) {
      Publisher<Integer> pub = new IntegerListPublisher(
        List.of(1, 2, 3, 4, 5));
      pub.subscribe(new IntegerSubscriber(3));
    }
    // Received data: 1
    // Received data: 2
    // Received data: 3
    // Completed!
    ~~~
## **4. 용어정리**
- 동기 : 작업을 하나의 프로세스(쓰레드)에서 순차적으로 실행하는 것
  - *집안일을 남편 혼자서 순차적으로 하는 것*
- 비동기 : 작업을 다른 프로세스(쓰레드)에서 실행하는 것
  - *집안일을 가족 모두가 다같이 나눠서 하는 것*
- 블로킹 : 작업 실행이 끝날 때까지 쓰레드가 대기하는 것
  - *설거지와 빨래, 청소를 손으로 직접 하는 것*
- 논블로킹 : 쓰레드가 작업 완료를 기다리지 않고 진행하는 것
  - *식기세척기와 세탁기, 로봇청소기를 작동시키는 것*