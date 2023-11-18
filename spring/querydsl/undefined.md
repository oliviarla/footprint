# 프로젝트 환경설정

## 설정파일

* build.gradle에 다음과 같이 설정해주어야 한다.

```gradle
plugins {
  id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

dependencies {
  implementation 'com.querydsl:querydsl-jpa'
}

def querydslDir = "$buildDir/generated/querydsl"

querydsl {
  jpa = true
  querydslSourcesDir = querydslDir
}
sourceSets {
  main.java.srcDir querydslDir
}
configurations {
  querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
  options.annotationProcessorPath = configurations.querydsl
}
```

## Q 타입

* 빌드 시 compileQuerydsl로 인해 QXXX.java 파일이 생성된다.
* QXXX.java 파일은 git에서 관리되면 안된다.
  * 일반적으로 `.gitignore`에 build 폴더가 들어가므로, 이 build 폴더에 Q 타입 파일이 생성되도록 하면 된다.

> `@PersistenceContext` vs `@Autowired` \
> \
> `@PersistenceContext`: 자바 표준 스펙이다. 스프링 말고 다른 컨테이너에도 사용 가능\
> `@Autowired` : 스프링만 사용 가능

## 라이브러리

* querydsl-apt: Querydsl 관련 코드 생성 기능 제공
* querydsl-jpa: querydsl 라이브러리

## application.yml

*   실행되는 JPQL 보기

    `spring.jpa.properties.hibernate.use_sql_comments: true`
