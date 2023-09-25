# Spring REST Docs

## 🍽️ REST Docs를 사용하는 이유

* **테스트가 성공**해야만 문서가 생성되므로, 테스트 코드를 짜야 하는 의무가 생긴다.
* Swagger와 달리 테스트 코드에만 REST Docs 관련 코드가 생길 뿐, 프로덕션 코드에는 아무런 영향이 없다.

## 🍽️ 적용 방법

{% hint style="info" %}
build.gradle.kts 에서 REST Docs 적용하는 방법을 알아보자.
{% endhint %}

### 의존성 추가

* build.gradle.kts 파일은 아래와 같이 작성한다.

```kotlin
// 1. restdocs에 대한 dependencies를 추가한다.
val asciidoctorExt: Configuration by configurations.creating

dependencies {
    testImplementation("org.springframework.restdocs:spring-restdocs-mockmvc")
    testImplementation("org.springframework.restdocs:spring-restdocs-asciidoctor")
    asciidoctorExt("org.springframework.restdocs:spring-restdocs-asciidoctor")
}

// 2. test 시 restdocs에서 자동 생성할 snippets를 저장할 폴더를 명시한다.
val snippetsDir by extra { file("build/generated-snippets") }
tasks.test {
    outputs.dir(snippetsDir)
}

// 3. snippetsDir에서 얻은 snippets를 사용해 index.html 파일을 생성한다.
//     생성된 파일들은 build 디렉토리와 프로덕션 디렉토리 모두에 추가한다. 
tasks.asciidoctor {
    doFirst {
        delete("src/main/resources/static/docs")
    }
    inputs.dir(snippetsDir)
    configurations(asciidoctorExt.name)
    dependsOn(tasks.test)
    doLast {
        copy {
            from("build/docs/asciidoc")
            into("src/main/resources/static/docs")
        }
        copy {
            from("build/docs/asciidoc")
            into("build/resources/main/static/docs")
        }
    }
}

// 4. bootJar 사용 시 asciidoctor가 실행되어야 한다.
tasks.bootJar {
    dependsOn(tasks.asciidoctor)
}
```

> `src/main/resources/static` 폴더가 없으면 3번 과정에서 복사가 제대로 되지 않으므로 만들어주어야 한다.

### 테스트 코드 동작 검증

* controller test에 REST Docs를 위한 코드를 추가한다.
* 테스트 코드를 실행시키면 `build/generated/generated-snippets` 경로로 adoc파일들이 자동 생성되는지 확인한다.

### API 문서 작성

* 생성된 snippets를 하나의 HTML 문서로 모아보기 위해 AsciiDoc을 사용한다.
* REST Docs는 지정된 디렉토리에 `.adoc` 소스 파일이 있다면, Generated files 위치에 html 파일을 생성해준다.
* 따라서 `src/docs/asciidoc` 에 아래와 같이 `index.doc` 파일을 생성해준다.

```markdown
= Read A Perfume API
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:sectlinks:

== 샘플
=== 요청
include::{snippets}/index/http-request.adoc[]
include::{snippets}/index/request-body.adoc[]
=== 응답
include::{snippets}/index/http-response.adoc[]
include::{snippets}/index/response-body.adoc[]
```

* 첫째 단락은 목차를 비롯한 전체적인 형태를 지정한다.
* 둘째 단락은 generated-snippets의 자료를 include해준다. `==` 로 h2, `===` 로 h3 과 같이 제목을 나타낼 수 있고, 직접 해당하는 스니펫들을 지정해주어야 한다.

### REST Docs 보기

* gradlew 파일이 있는 경로에서 `./gradlew bootjar` 명령어를 사용해 jar 파일을 생성한다. 혹은 Intellij gradle 플러그인을 사용해 jar 파일을 생성할 수도 있다.
* `java -jar (프로젝트명).jar` 혹은 인텔리제이를 사용해 애플리케이션을 구동시킨 후, `http://localhost:8080/api/docs/index.html` 로 접근해 제대로 동작하는지 확인한다.
  * 만약 jar 파일 실행 시 제대로 동작하지 않는다면 `jar -tf build/libs/(프로젝트명).jar | grep index.html` 로 jar 파일 내부에 index.html 이 정상적으로 포함되었는지 확인한다.

#### 참고 자료

[공식문서](https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/)

[https://techblog.woowahan.com/2597/](https://techblog.woowahan.com/2597/)

[https://tecoble.techcourse.co.kr/post/2020-08-18-spring-rest-docs/](https://tecoble.techcourse.co.kr/post/2020-08-18-spring-rest-docs/)

[http://api.verby.co.kr/docs/api-docs.html#errors](http://api.verby.co.kr/docs/api-docs.html#errors)

[https://shanepark.tistory.com/424](https://shanepark.tistory.com/424)

#### 더 나아가기

[https://toss.tech/article/kotlin-dsl-restdocs](https://toss.tech/article/kotlin-dsl-restdocs)
