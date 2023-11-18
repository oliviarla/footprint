# 중급 문법

## 프로젝션 결과 반환

### 기본

*   프로젝션 대상이 하나면 결과의 타입을 명확하게 지정할 수 있음

    ```java
    List<String> result = queryFactory
        .select(member.username)
        .from(member)
        .fetch();
    ```
*   프로젝션 대상이 둘 이상이면 튜플이나 DTO로 조회

    ```java
    List<Tuple> result = queryFactory
        .select(member.username, member.age)
        .from(member)
        .fetch();

    String username = result.get(member.username);
    ```

    * 튜플은 respository 계층 안에서만 사용되어야 한다.

### DTO

* **Querydsl 빈 생성(Bean population)**
  *   프로퍼티 접근

      * DTO는 기본 생성자와 setter를 가지고 있어야 한다.

      ```java
      List<MemberDto> result = queryFactory
            .select(Projections.bean(MemberDto.class,
                  member.username,
                  member.age))
            .from(member)
            .fetch();
      ```
  *   필드 직접 접근

      * getter, setter 필요 없음

      ```java
      List<MemberDto> result = queryFactory
          .select(Projections.fields(MemberDto.class,
              member.username,
              member.age))
          .from(member)
          .fetch();
      ```
  *   생성자 사용

      * dto 필드의 타입과 column 타입이 맞아야 한다.

      ```java
      List<MemberDto> result = queryFactory
          .select(Projections.constructor(MemberDto.class,
              member.username,
              member.age))
          .from(member)
          .fetch();
      ```
*   별칭이 다를 때

    * DTO의 필드 명과 DB의 컬럼 명이 다를 경우
    * 필드 직접 접근 방식은 `as`를 사용해 직접 필드명을 지정해주어야 한다.

    ```java
    List<UserDto> fetch = queryFactory
            .select(Projections.fields(UserDto.class,
                    member.username.as("name"),
                    ExpressionUtils.as( //서브쿼리
                            JPAExpressions
                            .select(memberSub.age.max())
                            .from(memberSub), "age"))
            ).from(member)
            .fetch();
    ```

    * 생성자 사용 방식은 필드 타입으로 생성되기 때문에 따로 지정해줄 필요가 없다.

### @QueryProjection

* DTO의 생성자에 `@QueryProjection` 어노테이션을 붙인다.
* 아래와 같이 DTO도 Q타입을 사용할 수 있다
* 컴파일러로 타입 체크가 가능해 안전하다
* DTO에 QueryDSL 어노테이션을 유지해야 하므로 의존성을 가지게 된다.
* DTO까지 Q 파일을 생성해야 하는 단점 존재

```java
List<MemberDto> result = queryFactory
    .select(new QMemberDto(member.username, member.age))
    .from(member)
    .fetch();
```

## 동적 쿼리

### BooleanBuilder

* where 절에 BooleanBuilder 객체가 들어간다
* null일 때 등 조건에 따라 검색 조건이 변경되는 경우를 처리한다

```java
BooleanBuilder builder = new BooleanBuilder();
if (usernameCond != null) {
    builder.and(member.username.eq(usernameCond)); //조건 추가
}
if (ageCond != null) {
    builder.and(member.age.eq(ageCond)); //조건 추가
}

List<Member> members = queryFactory
				.selectFrom(member)
				.where(builder)
				.fetch();
```

### Where 다중 파라미터

* BooleanBuilder 보다 훨씬 깔끔한 방법!
* where 조건에 null 값은 무시된다.
* 메서드를 다른 쿼리에서도 재활용 할 수 있다.
* 쿼리 자체의 가독성이 높아진다.

```java
private List<Member> searchMember2(String usernameCond, Integer ageCond) {
  return queryFactory
          .selectFrom(member)
          .where(usernameEq(usernameCond), ageEq(ageCond))
          .fetch();
}

private BooleanExpression usernameEq(String usernameCond) {
    return usernameCond != null ? member.username.eq(usernameCond) : null;
}

private BooleanExpression ageEq(Integer ageCond) {
    return ageCond != null ? member.age.eq(ageCond) : null;
}

// 두 조건 조합도 가능
private BooleanExpression allEq(String usernameCond, Integer ageCond) {
    return usernameEq(usernameCond).and(ageEq(ageCond));
 }
```

## 수정, 삭제 벌크 연산

* 한번에 여러 데이터를 갱신하는 경우 변경 감지 대신 벌크 연산을 하는 것이 유용
* 영속성 컨텍스트에 있는 엔티티를 무시하고 DB 쿼리를 실행한다.
* **배치 쿼리를 실행하고 나면 영속성 컨텍스트를 초기화 하는 것이 안전**하다.

```java
long count = queryFactory
          .update(member)
          .set(member.username, "비회원") 
          .where(member.age.lt(28)) 
          .execute();
```

```java
long count = queryFactory
       .delete(member)
       .where(member.age.gt(18))
       .execute();
```

## SQL function

* JPA와 같이 Dialect에 등록된 내용만 호출할 수 있다.

```java
queryFactory.select(member.username)
        .from(member)
        .where(member.username.eq(
                Expressions.stringTemplate("function('lower', {0})", member.username)))
```
