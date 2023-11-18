# 순수 JPA와 QueryDSL

{% hint style="info" %}
spring data jpa를 사용하지 않고 QueryDSL을 사용하는 경우를 다룬다.
{% endhint %}

## QueryDSL 적용하기

#### 순수 JPA를 사용하는 경우

* 쿼리 작성을 문자열 형태로 하기 때문에 휴먼 에러가 발생할 가능성이 크다.

```java
@Repository
public class MemberJpaRepository {
		private final EntityManager em;
		
		public MemberJpaRepository(EntityManager em) {
				this.em = em;
		}
		public List<Member> findAll() {
				return em.createQuery("select m from Member m", Member.class)
				.getResultList();
		}
		public List<Member> findByUsername(String username) {
				return em.createQuery("select m from Member m where m.username = :username", Member.class)
				.setParameter("username", username)
				.getResultList();
		}
}
```

#### QueryDSL을 적용한 경우

* 자바 컴파일에서 오류가 날 경우 잡아준다.

```java
@Repository
public class MemberJpaRepository {

		private final EntityManager em;
		private final JPAQueryFactory queryFactory;
		
		public MemberJpaRepository(EntityManager em) {
				this.em = em;
				this.queryFactory = new JPAQueryFactory(em);
		}
		
		public List<Member> findAll() {
				return queryFactory.selectFrom(member).fetch();
		}
		public List<Member> findByUsername(String username) {
				return queryFactory
					.selectFrom(member)
					.where(member.username.eq(username))
					.fetch();
		}
}
```

## JpaQueryFactory 스프링 빈 등록

* `JPAQueryFactory` 는 Configuration에 Bean으로 등록해두면 테스트 작성에 용이하다.
* 동시성 문제는 걱정하지 않아도 된다. 여기서 스프링이 주입해주는 엔티티 매니저는 실제 동작 시점에 진짜 엔티티 매니저를 찾아주는 프록시용 가짜 엔티티 매니저이다.
* 가짜 엔티티 매니저는 실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저(영속성 컨텍스트)를 할당해준다.

```java
@Configuration
public class QueryDSLConfig {

	@Bean
	JPAQueryFactory jpaQueryFactory(EntityManager em) {
			return new JPAQueryFactory(em);
	}
}
```

## Builder 사용하기

* 원하는 MemberTeamDto를 조회하기 위한 파라미터(조건)을 MemberSearchParams 클래스에 담아 넘긴다.

```java
@Data
public class MemberSearchParams {
	//회원명, 팀명, 나이(ageGoe, ageLoe)
	private String username;
	private String teamName;
	private Integer ageGoe;
	private Integer ageLoe;
}
```

* 동적 쿼리를 작성하기 위해 BooleanBuilder를 선언한다.
* 빌더에 조건을 추가하고 where 절에 빌더를 넘기면, 동적 쿼리를 날려 결과를 조회할 수 있다.

```java
public List<MemberTeamDto> searchByBuilder(MemberSearchParams params) {
      BooleanBuilder builder = new BooleanBuilder();
      if (StringUtils.hasText(params.getUsername())) {
          builder.and(member.username.eq(params.getUsername()));
      }
      if (StringUtils.hasText(params.getTeamName())) {
          builder.and(team.name.eq(params.getTeamName()));
      }
      if (condition.getAgeGoe() != null) {
          builder.and(member.age.goe(params.getAgeGoe()));
      }
      if (condition.getAgeLoe() != null) {
          builder.and(member.age.loe(params.getAgeLoe()));
      }

      return queryFactory
          .select(new QMemberTeamDto(
                  member.id,
                  member.username,
                  member.age,
                  team.id,
                  team.name))
          .from(member)
          .leftJoin(member.team, team)
          .where(builder) // 빌더를 통해 조건절 설정
          .fetch();
 }
```

* 조건을 따로 정해주지 않으면 모든 데이터를 가져오기 때문에, 기본 조건이나 limit 정도는 꼭 존재해야 장애가 나지 않는다.

## Where 절 파라미터 사용하기

* where 절에 BooleanExpression을 파라미터로 넣는 방식
* 메서드명으로 한눈에 무슨 의미인지 알아보기 쉬우며 재사용이 가능하다.

```java
public List<MemberTeamDto> search(MemberSearchCondition condition) {
    return queryFactory
        .select(new QMemberTeamDto(
                member.id,
                member.username,
                member.age,
                team.id,
                team.name))
        .from(member)
        .leftJoin(member.team, team)
        .where(usernameEq(condition.getUsername()),
                teamNameEq(condition.getTeamName()),
                ageGoe(condition.getAgeGoe()),
                ageLoe(condition.getAgeLoe()))
        .fetch();
							
private BooleanExpression usernameEq(String username) {
    return isEmpty(username) ? null : member.username.eq(username);
}
private BooleanExpression teamNameEq(String teamName) {
    return isEmpty(teamName) ? null : team.name.eq(teamName);
}
private BooleanExpression ageGoe(Integer ageGoe) {
    return ageGoe == null ? null : member.age.goe(ageGoe);
}
private BooleanExpression ageLoe(Integer ageLoe) {
    return ageLoe == null ? null : member.age.loe(ageLoe);
}
```
