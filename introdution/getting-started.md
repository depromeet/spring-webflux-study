> 해당 내용은 [DZone - Java Zone / Spring Webflux:Getting started](https://dzone.com/articles/spring-webflux-getting-started)를 번역한 글입니다.

이 포스트는 Spring Webflux에서 시작하는 방법에 관한 것 입니다. 몇 주 전, 나는 Spring Webflux, Reactive Streams 등에 대해 자세하게 읽기 시작했습니다. 나를 되돌아보며 잘못된 순서로 포스트를 했습니다. 과거에는 Spring Webflux를 시작하기 위한 계획을 제공하기 위해 내용를 구성하려고 노력 했었습니다. 이 예제는 [GitHub](https://github.com/mydeveloperplanet/myspringwebfluxplanet)에서 찾을 수 있습니다.

## Reactive Manifesto

Reactive Streams에 대한 기초는 [Reactive Manifesto](https://www.reactivemanifesto.org/ko)에서 찾을 수 있습니다. 길이가 2페이지에 불과하기 때문에 시간이 오래 걸리지 않으므로 읽어보는 것이 좋습니다.

## Reactive Streams

[Reactive Streams](http://www.reactive-streams.org/)은 웹사이트에서 읽을 수 있듯이, 논블로킹 백 프래셔와 함께 비동기 스트림 프로세싱를 위한 표준을 제공하는 발의(?)입니다. 이것은 매우 많은 데이터로 대상을 압도하지 않고 보낼 수 있는 것을 의미합니다. 그 대상은 얼마나 많은 데이터를 이용할 수 있는지 알려줍니다. 이 방법은 리소스를 사용하는데 매우 효과적입니다. Reactive Streams 사양이 표준으로 정해지길 원합니다.

JVM의 표준은 [GitHub](https://github.com/reactive-streams/reactive-streams-jvm)에서 볼 수 있습니다. 요약하자면, 다음 항목으로 구성됩니다.

- **Publisher**: Publisher는 하나 이상의 Subscriber들에게 데이터를 보낼 수 있는 출처입니다.

- **Subscriber**: Subscriber는 Publisher에게 자신을 구독하도록 하고 Publisher가 보낼 수있는 데이터의 양을 표시하며 데이터를 처리합니다.

- **Subscription**: Publisher측에서 Subscription이 생성되면, Subscriber와 공유됩니다.

- **Processor**: Publisher와 Subscriber 사이에 Processor를 사용하면 데이터 변환이 가능합니다.

Reactive Streams 사양은 표준이고 Java 9 이후에는 [Flow API](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html)와 함께 Java에 포함되었습니다. Reactive Streams 사양과 Java 9 Flow API를 자세히 살펴보세요. Publisher와 Subscriber, Subscription, Processor의 인터페이스가 동일한 것을 알 수 있습니다.

## Project Reactor

다음 단계는 [Project Reactor](https://projectreactor.io/) 입니다. 이 프로젝트는 Reactive Streams 사양을 기반으로 하는 reactive 라이브러리를 제공합니다. 즉, Reactive Streams 사양의 구현체입니다. 기본적으로 Reactor는 두 가지 타입을 제공합니다.

- **Mono**: Publisher를 구현하고 0 또는 1개의 요소를 반환합니다.
- **Flux**: Publisher를 구현하고 N개의 요소를 반환합니다.

하지만 Spring Webflux과 관계 있나요? Spring Webflux는 Project Reactor를 기반으로 하고 있으며, 이 포스트에서 주된 목표를 이룰 수 있습니다. Spring Webflux에 대해 자세히 알아보세요.

## Spring Webflux

Spring Webflux는 Spring 5와 함께 소개 되었습니다. 공식 Spring 문서는 [여기](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux)에서 찾아볼 수 있습니다. 전체 문서를 읽을 수 있겠지만 상당히 많은 정보입니다. 먼저 이 예제(포스트)를 보고나서 문서를 살펴보세요. 중요한건 Spring Webflux를 사용하는 방법이 두 가지라는 것 입니다. 하나는 Spring MVC와 아주 비슷한 애노테이션을 사용하는 것이고, 다른 하나는 functional한 방법을 사용하는 것 입니다. 나는 functional한 방법을 사용할 것 입니다. 게다가, Spring MVC와 Spring Webflux가 서로 나란히 있다는 것을 알아야 합니다. Spring MVC는 동기 처리를 위해 사용되고, Spring MVC는 비동기 처리를 위해 사용됩니다.

~~ (필요없는 내용인 것 같아..) 생략 ~~

콘솔 로그를 보면, Netty 웹 서버가 사용되고 있음을 알 수 있습니다. 이는 Spring Webflux 애플리케이션의 디폴트입니다.

## Spring Webflux Basic Example

이제 기본 예제를 만들고 여기서 정확히 무엇이 일어나는지 이해 해보세요.

http://localhost:8080/example URL에 액세스 할 수 있도록 하기위해 최소한의 작업을 한 [Greeting Example](https://spring.io/guides/gs/reactive-rest-service/)과 비슷하게 com.mydeveloperplanet.myspringwebfluxplanet.examples 패키지를 만들고 ExampleRouter와 ExampleHandler를 추가합니다.

ExampleRouter는 다음과 같이 됩니다:

```java
@Configuration
public class ExampleRouter {
  @Bean
  public RouterFunction<ServerResponse> route(ExampleHandler exampleHandler) {
    return RouterFunctions
        .route(RequestPredicates.GET("/example").and(RequestPredicates.accept(MediaType.TEXT_PLAIN)), exampleHandler::hello);
  }
}
```

ExampleHandler는 다음과 같이 됩니다:

```java
@Component
public class ExampleHandler {
  public Mono<ServerResponse> hello(ServerRequest request) {
    return ServerResponse.ok().contentType(MediaType.TEXT_PLAIN)
         .body(BodyInserters.fromObject("Hello, Spring Webflux Example!"));
  }
}
```

메이븐 타겟을 spring-boot:run로 실행하세요. 브라우저의 URL http://localhost:8080/example 으로 이동하세요. 다음 오류가 표시됩니다:

![Alt 스플링 웹플럭스 라우딩 에러 페이지](https://mydeveloperplanet.files.wordpress.com/2018/02/spring-webflux-routing-error.png)

콘솔 로그에서 다음과 같은 내용을 볼 수 있습니다:

```text
Overriding bean definition for bean 'route' with a different definition: replacing [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=exampleRouter; factoryMethodName=route; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [com/mydeveloperplanet/myspringwebfluxplanet/examples/ExampleRouter.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=greetingRouter; factoryMethodName=route; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [com/mydeveloperplanet/myspringwebfluxplanet/greeting/GreetingRouter.class]]
```

이제 ExampleRouter 클래스의 route 메소드를 routeExample으로 변경하고 응용 프로그램을 다시 실행합니다. 이제 브라우저는 예상했던 내용을 보여줍니다:

```text
Hello, Spring Webflux Example!
```

최소한 Router와 Handler만 필요합니다. 우리는 Router Bean이 유니크한 이름을 가지고 있는지 확인해야 합니다. 그렇지 않으면 같은 이름을 가진 다른 Bean에 의해 덮어 쓰여질 수 있습니다. RouterFunction은 일치하는 route를 결정하고 결국 Handler 메소드로 해석됩니다.

## Spring Webflux 'one step further' Example

이제 기본 설정 방법을 알았으므로, 한 단계 더 나아가 추가 기능을 살펴 보겠습니다.

우리는 ExampleRouter 클래스에서 routeExampleOneStepFurther 메소드를 만들고 andRoute 메소드를 사용하여 두 개의 라우트를 서로 연결합니다. 이렇게하면 여러 route를 서로 연결할 수 있고 서로 다른 Handler 메소드를 해결할 수 있습니다. route는 하나씩 평가되므로 일반적인 route보다 먼저 특정 route를 정의해야 합니다. routeExampleOneStepFurther 메소드는 다음과 같습니다:

```java
public RouterFunction<ServerResponse> routeExampleOneStepFurther(ExampleHandler exampleHandler) {
  return RouterFunctions
    .route(RequestPredicates.GET("/exampleFurther1").and(RequestPredicates.accept(MediaType.TEXT_PLAIN)), exampleHandler::helloFurther1)
    .andRoute(RequestPredicates.GET("/exampleFurther2").and(RequestPredicates.accept(MediaType.TEXT_PLAIN)), exampleHandler::helloFurther2);
 }
```

Handler 메소드들은 ExampleHandler 클래스에서 생성되고 단지 문자열을 리턴합니다:

```java
public Mono<ServerResponse> helloFurther1(ServerRequest request) {
  return ServerResponse.ok().contentType(MediaType.TEXT_PLAIN)
    .body(BodyInserters.fromObject("Hello, Spring Webflux Example 1!"));
}
public Mono<ServerResponse> helloFurther2(ServerRequest request) {
  return ServerResponse.ok().contentType(MediaType.TEXT_PLAIN)
    .body(BodyInserters.fromObject("Hello, Spring Webflux Example 2!"));
}
```

이제 애플리케이션을 build하고 실행해서 http://localhost:8080/exampleFurther1 URL을 호출하세요. 결과는 다음과 같습니다:

```java
Hello, Spring Webflux Example 1!
```

같은 방법으로, http://localhost:8080/exampleFurther2 URL은 다음 결과를 보여줍니다:

```java
Hello, Spring Webflux Example 2!
```

우리는 수동으로 테스트 했으니 이제 단위 테스트를 만들어 보겠습니다. 단위 테스트는 다음과 같습니다:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ExampleRouterTest {
  @Autowired
  private WebTestClient webTestClient;
  @Test
  public void testExampleOneStepFurther() {
    webTestClient
      .get().uri("/exampleFurther1")
      .accept(MediaType.TEXT_PLAIN)
      .exchange()
      .expectStatus().isOk()
      .expectBody(String.class).isEqualTo("Hello, Spring Webflux Example 1!");
    webTestClient
      .get().uri("/exampleFurther2")
      .accept(MediaType.TEXT_PLAIN)
      .exchange()
      .expectStatus().isOk()
      .expectBody(String.class).isEqualTo("Hello, Spring Webflux Example 2!");
  }
}
```

이 테스트는 Spring Webflux 예제를 기반으로 합니다. 테스트하는 동안, 임의의 포트에서 서버가 시작되고 서로 하나씩 URL을 호출합니다. exchange 메소드를 통해 응답에 액세스 할 수 있습니다. 먼저 응답 상태를 확인한 다음 응답이 올바른지 확인합니다. Spring Webflux 예제에서 equals 메서드는 응답을 확인하는데 사용되지만 응답이 결과와 동일하지 않은 경우에도 항상 성공합니다. 이것은 올바르지 않습니다. 올바른 테스트를 수행하려면 isEqualTo 메소드를 사용해야 합니다.

## Spring Webflux 'last steps' Example

마지막으로 하나의 파라미터를 갖는 route에 대한 테스트를 추가하고 실패하는 API 호출에 대한 테스트를 만듭니다.

routeExampleOneStepFurther 메소드에서 다른 route를 추가하고 ExampleHandler 클래스의 helloFurther3 메소드로 해석하도록 합니다. route는 다음과 같습니다:

```java
.andRoute(RequestPredicates.GET("/exampleFurther3").and(RequestPredicates.accept(MediaType.TEXT_PLAIN)), exampleHandler::helloFurther3)
```

helloFurther2 메소드는 name 파라미터를 추출하고 주어진 이름에 대한 hello를 말합니다:

```java
public Mono<ServerResponse> helloFurther3(ServerRequest request) {
  String name = request.queryParam("name").get();
  return ServerResponse.ok().contentType(MediaType.TEXT_PLAIN)
    .body(BodyInserters.fromObject("Hello, " + name + "!"));
}
```

애플리케이션을 build하고 실행하고 http://localhost:8080/exampleFurther3?name=My%20Developer%20Planet URL로 가보세요. 우리가 기대한 결과를 줍니다:

```text
Hello, My Developer Planet!
```

우리가 추가한 테스트는 앞서 작성한 테스트와 비슷합니다:

```java
@Test
public void testExampleWithParameter() {
  webTestClient
    .get().uri("/exampleFurther3?name=My Developer Planet")
    .accept(MediaType.TEXT_PLAIN)
    .exchange()
    .expectStatus().isOk()
    .expectBody(String.class).isEqualTo("Hello, My Developer Planet!");
}
```

결론을 내리기 위해, 404 http 응답을 반환하는 테스트를 추가합니다. 해결되지 않은 URL 경로에 대한 예상 상태는 isNotFound 메소드를 사용하여 테스트합니다.

```java
@Test
public void testExampleWithError() {
  webTestClient
    .get().uri("/example/something")
    .accept(MediaType.TEXT_PLAIN)
    .exchange()
    .expectStatus().isNotFound();
}
```

## Summary

이 포스트에서는 Spring Webflux를 시작하는 방법에 대한 정보를 주려고 노력했습니다:

- Reactive 프로그래밍을 시작하기 전에 보는 것을 제안
- Spring에서 제공하는 기본 Spring Webflux 예제
- 몇가지 첫 번째 단계를 수행하기 위한 약간의 Spring Webflux 예제

이것은 매우 기본적인 정보일 뿐이며, 이 시점부터 Spring Webflux와 관련하여 살펴볼 다른 많은 것들이 있습니다.

Actor 모델은 단순하지만 힘을 어떻게 제공하는지 배우세요.
