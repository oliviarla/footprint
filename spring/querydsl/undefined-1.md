# 기본 문법

{% hint style="info" %}
런타임오류가 아닌 컴파일 오류를 발생시키기 때문에 JPQL보다 예외상황을 찾기 쉽다.
{% endhint %}

## 들어가기

```java
// 1)
JPAQueryFactory queryFactory = new JPAQueryFactory(em);

//2)
QMember m = new QMember("m");

//3)
Member findMember = queryFactory
    .select(m)
    .from(m)
    .where(m.username.eq("member1")) //파라미터 바인딩 처리
    .fetchOne();

assertThat(findMember.getUsername()).isEqualTo("member1");
```

1. em(entity manager)을 사용해 JPAQueryFactory 객체를 만든다.
2. compile해 Q타입을 가져오고 변수명에 별칭을 단다.
3. 자바 메서드 형태로 원하는 쿼리를 날린다.

## 기본 Q-Type 활용

*   JPQL의 별칭을 직접 지정할 수 있다.

    `QMember qMember = new QMember("m");`
*   혹은 기본 인스턴스를 사용할 수 있다.

    `QMember qMember = QMember.member;`
*   기본 인스턴스를 static import해 사용하는 것이 가장 권장된다.

    ```java
    import static study.querydsl.entity.QMember.*;

    ...

    Member findMember = queryFactory
                  .select(member)
                  .from(member)
                  .where(member.username.eq("member1"))
                  .fetchOne();
    ```

## 검색 조건 쿼리

* selectFrom → select 와 from을 합친 메서드
*   검색 조건을 `.and()`, `.or()` 메서드로 체이닝할 수 있다.

    ```java
    Member findMember = queryFactory
                    .selectFrom(member)
                    .where(member.username.eq("member1")
                            .and(member.age.eq(10)))
                    .fetchOne();
    ```
*   `.and()`가 여러개일 경우 `,` 를 사용해 여러 조건을 넘겨도 된다.

    ```java
    List<Member> result1 = queryFactory
                  .selectFrom(member)
                  .where(member.username.eq("member1"),
                          member.age.eq(10))
                  .fetch();
    ```
* JPQL이 제공하는 모든 검색 조건을 제공한다.

```java
// 비교 조건
member.username.eq("member1") // username = 'member1'
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'

// null 조건
member.username.isNotNull() //이름이 is not null

// 범위 조건
member.age.in(10, 20) // age in (10,20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) //between 10, 30member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30

// 문자열 조건
member.username.like("member%") //like 검색 
member.username.contains("member") // like ‘%member%’ 검색 
member.username.startsWith("member") //like ‘member%’ 검색
```

## 결과 조회

* fetch() : 리스트 조회, 데이터 없으면 빈 리스트 반환
* fetchOne() : 단 건 조회
  * 결과가 없으면 : null
  * 결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException
* fetchFirst() : limit(1).fetchOne()과 동일
*   fetchResults() : 페이징 정보 포함, total count 쿼리 추가 실행

    ```java
    QueryResults<Member> results = queryFactory
              .selectFrom(member)
              .fetchResults();
    ```
* fetchCount(): count 쿼리로 변경해서 count 수 조회

## 정렬

* orderBy() 메서드 내부에 정렬 조건을 넣을 수 있다.
* desc() , asc() : 일반 정렬 (내림차순, 오름차순)
* nullsLast() , nullsFirst() : null 데이터 순서 부여

```java
List<Member> result = queryFactory
  .selectFrom(member)
  .where(member.age.eq(100))
  .orderBy(member.age.desc(), member.username.asc().nullsLast())
  .fetch();
```

## 페이징

* 조회 건수 제한
* orderBy가 필요하다.
* offset: 0부터 시작, 몇 번째 row에서 데이터 조회를 시작할지 정함

```java
List<Member> result = queryFactory
    .selectFrom(member)
    .orderBy(member.username.desc()) 
    .offset(1)
    .limit(2) //최대 2건 조회
    .fetch();
```

## 조인

* 첫 번째 파라미터에 조인 대상을 지정하고, 두 번째 파라미터에 별칭(alias)으로 사용할 Q 타입을 지정

```java
QMember member = QMember.member;
QTeam team = QTeam.team;

List<Member> result = queryFactory
    .selectFrom(member)
    .join(member.team, team)
    .where(team.name.eq("teamA"))
    .fetch();
```

* join() , innerJoin() : 내부 조인(inner join)
* leftJoin() : left 외부 조인(left outer join)
* rightJoin() : rigth 외부 조인(rigth outer join)
* JPQL의 on과 성능 최적화를 위한 fetch 조인 제공
*   세타 조인

    * rom 절에 여러 엔티티를 선택하면 세타 조인이다.

    ```java
    List<Member> result = queryFactory
        .select(member)
        .from(member, team)
        .where(member.username.eq(team.name))
        .fetch();
    ```

### on 절 사용한 조인

*   조인 대상 필터링

    ```java
    List<Tuple> result = queryFactory
        .select(member, team)
        .from(member)
        .leftJoin(member.team, team).on(team.name.eq("teamA"))
        .fetch();
    ```

    * on 절을 활용해 조인 대상을 필터링 할 때, 내부조인(inner join)을 사용하면, where 절에서 필터링 하는 것과 기능이 동일하므로 익숙한 where 절로 해결할 것
    * 외부조인이 필요한 경우에만 on 절을 사용하자
*   연관관계 없는 엔티티 외부조인

    ```java
    // 회원의 이름과 팀의 이름이 같은 대상 외부 조인
    List<Tuple> result = queryFactory
        .select(member, team)
        .from(member)
        .leftJoin(team).on(member.username.eq(team.name))
        .fetch();
    ```

    * username과 team 이름이 같은 것이 없다면? member와 null(~~team이었던 것..~~)이 결과로 반환됨

### 페치 조인

* SQL조인을 활용해서 연관된 엔티티를 SQL 한번에 조회하는 기능
* join(), leftJoin() 등의 뒤에 fetchJoin()을 추가하면 된다.

```java
Member findMember = queryFactory
              .selectFrom(member)
              .join(member.team, team).fetchJoin()
              .where(member.username.eq("member1"))
              .fetchOne();
```

## 서브쿼리

* `com.querydsl.jpa.JPAExpressions` 사용

```java
// 나이가 가장 많은 회원 조회

// 서브쿼리용 alias 추가
QMember memberSub = new QMember("memberSub");

List<Member> result = queryFactory
      .selectFrom(member)
      .where(member.age.eq(
            JPAExpressions
                    .select(memberSub.age.max())
                    .from(memberSub)))
      .fetch();
```

```java
List<Member> result = queryFactory
              .selectFrom(member)
              .where(member.age.in(
                      JPAExpressions
                              .select(memberSub.age)
                              .from(memberSub)))
              .fetch();
```

```java
List<Tuple> fetch = queryFactory
          .select(member.username,
                  JPAExpressions
                          .select(memberSub.age.avg())
                          .from(memberSub)
          ).from(member)
          .fetch();
```

**from 절의 서브쿼리 해결방안**

1. 서브쿼리를 join으로 변경한다. (가능한 상황도 있고, 불가능한 상황도 있다.)
2. 애플리케이션에서 쿼리를 2번 분리해서 실행한다.
3. nativeSQL을 사용한다.

> DB에서 데이터를 가져올 때는 **데이터를 최소화해서 가져오는 역할만 처리**하는 것이 좋다.

## case문

* select, 조건절(where), order by에서 사용 가능

***

```java
List<String> result = queryFactory
    .select(member.age
    .when(10).then("열살") 
    .when(20).then("스무살") 
    .otherwise("기타"))
    .from(member)
    .fetch();
```

* 복잡한 조건을 CaseBuilder 변수로 선언

```java
NumberExpression<Integer> rankPath = new CaseBuilder()
             .when(member.age.between(0, 20)).then(2)
             .when(member.age.between(21, 30)).then(1)
             .otherwise(3);

List<Tuple> result = queryFactory
       .select(member.username, member.age, rankPath)
       .from(member)
       .orderBy(rankPath.desc())
       .fetch();
```

## 상수, 문자 더하기

*   상수를 결과값에 같이 반환하기

    * 사실 사용할 일이 있긴 할까 싶다

    ```java
    Tuple result = queryFactory
          .select(member.username, Expressions.constant("A"))
          .from(member)
          .fetchFirst();
    ```
*   문자를 더해서 반환하기

    ```java
    String result = queryFactory
    	  .select(member.username.concat("_").concat(member.age.stringValue()))
    	  .from(member)
    	  .where(member.username.eq("member1"))
    	  .fetchOne();
    ```

> `stringValue()` : Enum 타입과 같이 문자가 아닌 다른 타입일 경우 이 함수를 사용해 가져와야 한다.
