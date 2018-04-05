# Web on Reactive Stack
https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html

Web on Reactive Stack에서는 아래와 같이 4가지로 구성이 되어있다.
1. Spring Webflux
2. Webclient
3. Testing
4. Reactive Libraries

## Spring Webflux

### Introduction
기존의 Spring Framework MVC
- Servlet API와 Servlet Container 기반
- spring-webmvc

Spring Reactive Stack, Spring Webflux
- Reactive Stream 기반 (Netty, Undertow, Servlet +3.1 Container)
- spring-webflux

### Why a new web Framework?
- 적은 자원(less hardware resource)에서 사용되는 작은 수의 쓰레드 및 논블로킹(non-blocking) concurrent를 처리하기 위해서는 non-blocking web stack이 필요하다.
- 기존 Se rvlet의 api의 자체가 synchronous 하기 때문에 새로운 non-blocking runtime(netty 등)을 사용하게 되었다.

## What is reactive?
- 변환에 반응하는것 (reacting to change)
  - 마우스 이벤트 클릭에 반응하는 것
  - I/O 이벤트에 반응하는것
- 반응하는 것(Reactring)은 non-blocking 메카니즘의 연관이 있다.
- java 9에서 작은 부분(small spec) 도입이 되었다.
  - Publihser
  - Subscriber

## Reactive API
- Spring Webflux는 Reactive Library로 Reactor를 사용한다.
  - Mono - 하나의 데이터 0..1
  - Flux - N개의 데이터 0..N
- Reactive Libraries를 자세하게 보려면 아래의 사이트를 참고
https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-reactive-libraries

### Programming Model
- Annotation 방식
  - 기존 Spring MVC와 동일한 방식
- Functional Endpoint 방식
  - lambda based, lightweight, Functional Programming Model

### Applicability
- Spring Web MVC를 사용하고 있다면?
  - 바꿀필요는 없다.
- non-blocking web stack를 찾고 있다면?
  - sever (netty, tomcat, jetty), Programming model(Annotation, Functional0), Reactive Library(Reactor, RxJava)
- java8의 람다나 kotlin을 이용하여 가벼운 application을 제작할때?
  - 좋은 선택이 될수가 있다.
- Microserivce Architecture
- Spring MVC를 사용하고 있다면 Reactive Webclient

## Servers
- tomcat
- jetty
- servlet+ 3.1 containers
- Spring boot는 기본적으로 netty를 사용하고 있다. starter module를 추가해서 다른 server (tomcat, jetty 등)으로 변경이 가능하다.

## Performance vs Scale
- Reative가 application을 빠르게하지는 않는다.
- 고정된 자원(Resource)를 이용하여 scale이 가능하는 것을 목표를 한다.

## Concurrency Mdoel
Spring MVC
- block (current thread)

Spring Webflux
- non-blocking
- 작은 단위의 thread-pool를 사용할 수 있다.

Blocking API
- Blocking의 특징을 가진 API를 사용할수가 있다.
- Reactor, Rxjava는 둘다 Operatetor publihserOn을 사용할수가 있다.

## Spring boot 2.0 Webflux
- spring boot 2.0를 이용하여 spring webflux를 적용을 할수 있다.

https://projects.spring.io/spring-boot/
https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#boot-features-webflux
