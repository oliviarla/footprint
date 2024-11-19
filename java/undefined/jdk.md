# JDK

## 자바와 JDK

* Java 자체는 **언어**이며, JDK는 이를 **개발하고 실행시킬 수 있도록 해주는 프로그램**이다.
* Java와 JDK는 독립적이다. 특정 기업에서 제공하는 JDK가 유료화된다 하더라도 자바 자체가 유료화되는 것이 아니다.
* JDK는 Oracle, Amazon, Zul 등 다양한 기업에서 다양한 플랜으로 제공하고 있다.
* JVM으로 Java 외에 Closure, Groovy, Kotlin, Scala, Jython 등의 언어도 실행할 수 있다.

```sh
# Java 실행하기
javac Hello.java
java Hello

# Kotlin 실행하기
kotlinc Hello.kt -include-runtime -d hello.jar
java -jar hello.jar
```

## JDK 구성

<figure><img src="../../.gitbook/assets/image (19) (1).png" alt="" width="347"><figcaption><p>대략적 구성도</p></figcaption></figure>

* Java 플랫폼
  * JDK보다 넓은 의미로 **Java 개발 및 실행 환경**을 의미
  * SE, EE, ME 등 JDK를 구현한 제품
* JDK (Java Development Kit)
  * Java 프로그램 실행 및 **개발 환경**을 제공한다.
  * JVM과 자바클래스 라이브러리 외 자바 개발에 필요한 프로그램들이 설치된다.
  * JRE + 개발에 필요한 실행파일 (javac.exe, java.exe, javap.exe 등) 로 구성된다.
* JRE (Java Runtime Environment)
  * 자바 실행 환경
  * JVM과 자바 핵심 라이브러리, 자바 런타임 환경에서 사용하는 프로퍼티 세팅이나 리소스 파일으로 구성된다.
  * 자바로 작성된 응용프로그램이 실행되기 위한 최소 환경이다.
  * JVM + 클래스 라이브러리(Java API)로 구성된다.
  * 오라클은 자바 11부터 JRE를 따로 제공하지 않는다.
* JVM
  * 프로그램을 실행하는 자바 가상 머신
  * 인터프리터와 JIT 컴파일러를 이용해 **Java 바이트 코드를 기계어(OS에 특화된 코드)로 변환**하고 실행한다.
  * 다양한 플랫폼 위에서 자바를 동작시킬 수 있게 하는 중요한 역할을 한다.
  * 대표적인 역할로는 클래스 로딩, GC 등 메모리 관리, 스레드 관리, 예외 처리 등이 있다.
  * 자바 컴파일러와는 다른 모듈이며 논리적인 개념, 여러 모듈의 결합체이다.
  * JVM에는 표준이 존재하여 JDK를 제공하는 기업들은 이 표준에 맞추어 개발해야 한다.
    * [https://docs.oracle.com/javase/specs/jvms/se22/html/index.html](https://docs.oracle.com/javase/specs/jvms/se22/html/index.html)

> 모듈: 코드,데이터를 그룹화하여 재사용이 가능한 정적인 단위
>
> 컴포넌트: 독립적으로 실행될 수 있는 소프트웨어 단위

* 아래의 그림을 보면 컴파일러인 `javac`, 바이트 코드를 JVM 상에서 실행하는 `java` 등은 JDK에 속하고 `java.lang` 과 같은 라이브러리는 JRE에 속한다. JVM은 JDK, JRE에 모두 속하고 있다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>자세한 구성도</p></figcaption></figure>



### JVM 구성 요소 <a href="#jvm" id="jvm"></a>

1\. 자바 인터프리터(interpreter)

&#x20;   자바 컴파일러에 의해 변환된 자바 바이트 코드를 읽고 해석함

2\. 클래스 로더(class loader)

&#x20;   프로그램이 실행 중인 런타임에서 모든 코드가 JVM과 연결될 때, 동적으로 클래스를 로딩함

3\. JIT 컴파일러(Just-In-Time compiler)

&#x20;   자바 컴파일러가 생성한 자바 바이트 코드를 런타임에 바로 기계어로 변환

4\. 가비지 컬렉터(garbage collector)

&#x20;   더이상 사용하지 않는 메모리 해제
