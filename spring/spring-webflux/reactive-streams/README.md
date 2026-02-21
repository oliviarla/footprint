---
description: Spring WebFlux의 기반이 되는 Reactive Streams를 정리한다.
---

# Reactive Streams

## 리액티브 프로그래밍

*
  * 데이터 소스에 변경이 있을 때마다 데이터 전파
  * 선언형 프로그래밍 패러다임 → 실행할 동작을 명시하는 것(명령형)이 아니라 원하는 것을 정의
  * 함수형 프로그래밍 기법 사용

## 리액티브 스트림즈

* 리액티브 프로그래밍을 표준화한 명세
* 인터페이스 종류
  * Publisher
  * Subscriber
  * Subscription
  * Processor
* RxJava, Java 9 Flow API, Reactor 등이 이 명세를 구현한다.

