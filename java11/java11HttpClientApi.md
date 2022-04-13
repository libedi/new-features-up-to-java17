# **Java 11 HTTP Client API**
- 기존 `java.net.URLConnection` API의 대안으로 권장되는 HTTP Client API가 추가되었다.
- HTTP 1.1과 HTTP 2 모두 지원
- `CompletableFuture`를 통해 non-blocking 요청과 응답의 방식 제공
- 패키지 경로 `java.net.http`
## **`HttpRequest`**
- HTTP 요청의 URI, body와 headers 정보를 갖는 클래스
## **`HttpRequest.Builder`**
- `HttpRequest` 객체 생성을 도와주는 빌더 클래스
- `HttpRequest.newBuilder()`로 생성
## **`HttpClient`**
- HTTP 요청을 보내고 응답을 받는데 사용되는 클래스
- `HttpClient.newHttpClient()` 정적 메소드를 통해 기본 생성
- 또는 `HttpClient.newBuilder()`를 통해 custom값을 설정할 수 있다.
- `HttpClient.send()` 메소드를 통해 주어진 요청을 전송
- 반환값은 `HttpResponse<T>`
- `HttpClient.sendAsync()` 메소드를 통해 주어진 요청을 비동기 전송 가능
- 반환값은 `CompletableFuture<HttpResponse<T>>`
## **`HttpResponse`**
- 이 클래스의 인스턴스는 `HttpRequest` 전송의 결과로서 반환된다.
- 다양하게 유용한 핸들러를 내부 정적 인터페이스로 제공한다. ex) `HttpResponse.BodyHandler<T>`
~~~java
// 요청 생성
HttpRequest request = HttpRequest.newBuilder()
                                 .url(URI.create("http://www.test.com"))
                                 .GET() // 기본값
                                 .build();
// 응답 body handler 생성
HttpResponse.BodyHandler<String> bodyHandler = HttpResponse.BodyHandlers.ofString();

// HttpClient를 사용해 비동기 요청 전송
HttpClient client = HttpClient.newHttpClient();
CompletableFuture<HttpResponse<String>> responseFuture = client.sendAsync(request, bodyHandler);
responseFuture.thenApply(HttpResponse::body)
              .thenAccept(System.out::println)
              .join();
~~~
- 개인적으로 Spring의 `RestTemplate`보다 사용법이 훨씬 나아보인다.
- 더 자세한 내용은 Baeldung의 [Exploring the New HTTP Client in Java](https://www.baeldung.com/java-9-http-client) 참고