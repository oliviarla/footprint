# 바이트 코드 조작

## 바이트 코드 조작

### 라이브러리

* ASM
  * visitor 패턴, adapter 패턴을 이해해야 사용하기 쉽다.
* Javaassist
* ByteBuddy
  * 가장 간단하게 바이트 코드를 조작할 수 있다.

### 조작 예제

* 아래와 같이 빈 문자열을 반환하는 메서드를 가진 Moja 클래스의 바이트코드를 조작하여 Rabbit을 반환하도록 만들 수 있다.

```java
public class Moja {

    public String pullOut() {
        return "";
    }
}

```

```java
try {
    new ByteBuddy.redefine(Moja.class)
        .method(named("pullOut")).intercept(FixedValue.value("Rabbit"))
        .make().saveIn(new File("/User/workspace/example/target/classes/"));
} catch (IOException e) {
    e.printStackTrace();
}
```

## 자바 에이전트

### 소개

* 클래스 로딩 시 자바 에이전트를 거쳐 변경된 바이트 코드가 메모리에 저장되어 사용된다.
* 모델
  * premain 모델
    * 애플리케이션 구동 시 옵션을 주어 에이전트를 연동하는 방식
  * agent 모델
    * 구동중인 프로세스에 에이전트를 연동하는 방식
* 속성
  * Can-Redefine-Classes
    * 에이전트에 의해 클래스를 재정의할 수 있는지 여부 설정
  * can-Retransform-Classes
    * 에이전트에 의해 클래스를 변형할 수 있는지 여부 설정

### 실습

* bytebuddy에 대한 의존성을 등록해주고 아래와 같이 AgentBuilder를 통해 에이전트 클래스를 생성한다.

```java
public class MasulsaAgent {
    public static void premain(String agentArgs, Instrumentation inst) { 
        new AgentBuilder.Default()
        .type(ElementMatchers.any())
        .transform((builder, typeDescription, classLoader, javaModule) 
            -> builder.method(named("pullOut")).intercept(FixedValue.value("Rabbit!")))
        .installOn(inst);
    }
}
```

* 아래와 같이 premain 클래스를 등록하고 redifine, retransform 속성을 활성화한다.

```groovy
tasks.named('jar') {
	manifest {
		attributes(
			'Premain-Class' : "com.java.agent.MasulsaAgent",
			'Can-Redefine-Classes' : true,
			'Can-Retransform-Classes' : true)
	}
}
```

* `./gradlew build` 를 통해 생성된 agent jar 파일을 사용해 애플리케이션을 실행하면 클래스로딩 시 에이전트에 의해 변경된 바이트 코드가 애플리케이션에서 사용된다. 이 때 클래스 파일이 변경되지는 않는다.

```
java -javaagent:agent.jar -jar application.jar
```



## 바이트 코드 조작 활용

* 프로그램 분석
  * 코드에서 버그를 찾거나 복잡도를 계산하는 툴에서 사용할 수 있다.
* 클래스 파일 생성
  * 프록시 클래스 생성, 특정 API 호출 접근 제한, 스칼라 등 JVM 언어의 컴파일러에서 사용할 수 있다.
* Mockito, Pinpoint 등 프로파일러, 최적화, 로깅 등 다양한 경우에 사용된다.
* 스프링의 컴포넌트 스캔에서도 후보 클래스 정보를 찾는 데에 ASM 라이브러리가 사용된다.
  * ClassPathScanningCandidateComponentProvider 클래스를 통해 컴포넌트 스캔이 진행되며, SimpleMetadataReader 클래스에서 ClassReader와 Visitor를 사용해 클래스에 있는 메타 정보를 읽어온다.
