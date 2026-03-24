# 5장: 스프링 데이터 JPA를 이용한 조회 기능

## 스펙

### 기본 개념

* 다양한 검색 조건을 조합해야 할 때 필요한 조합마다 find 메서드를 정의하는 대신 스펙을 사용해 애그리거트가 특정 조건을 충족하는지 검사할 수 있다.
* 스펙을 리포지터리 인터페이스에 사용하면 입력 인자는 애그리거트 루트가 되고, 스펙을 DAO에 사용하면 입력 인자는 검색 결과로 리턴할 데이터 객체가 된다.
* 스펙을 이용하면 검색 대상을 걸러낼 수 있다. 아래와 같이 검사 대상 객체가 조건을 충족하면 true를 리턴하고, 그렇지 않으면 false를 리턴하는 메서드를 구현하면 된다.

```java
public interface Speficiation<T> {
	public boolean isSatisfiedBy(T agg);
}
```

* 하지만 이러한 방식은 메모리에 모든 애그리거트 데이터가 올라와 있는 경우에 한해 적용할 수 있기 때문에, 실제 환경에서 사용하려면 Spring Data JPA에서 제공해주는 스펙 인터페이스를 활용할 수밖에 없다.

### Spring Data JPA의 스펙

* Spring Data JPA에서는 검색 조건을 표현하기 위해 Specification 인터페이스를 제공한다. toPredicate 메서드는 JPA Criteria API에서 조건을 표현하는 Predicate을 생성한다.

```java
public interface Specification<T> extends Serializable {
	// ...
	@Nullable
	Predicate toPredicate(Root<T> root,
												CriteriaQuery<?> query,
												CriteriaBuilder cb);
}
```

* 아래는 스펙 인터페이스를 구현한 클래스 예시로, OrderSummary에 대해 ordererId가 동일한지에 대한 검색 조건을 표현한다.

```java
public class OrdererIdSpec implements Specification<OrderSummary> {
	private String ordererId;
	public OrdererIdSpec(String ordererId) {
	this.ordererId = ordererId;
	}
	@Override
	public Predicate toPredicate(Root<OrderSummary> root,
														 	 CriteriaQuery<?> query,
													 		 CriteriaBuilder cb) {
		return cb.equal(root.get(OrderSummary_.ordererId), ordererId);
	}
}
```

> **JPA 정적 메타 모델**
>
> * 정적 메타 모델은 @StaticMetamodel 애너테이션을 이용해서 관련 모델을 지정한다.
> * 메타 모델 클래스는 모델 클래스의 이름 뒤에 ‘\_’을 붙인 이름을 갖는다.
> * 대상 모델의 각 프로퍼티와 동일한 이름을 갖는 정적 필드를 정의한다.
> * Criteria를 사용할 때 정적 메타 모델 클래스를 사용하면 컴파일 시점에 오류를 검사하기 때문에 문자열로 프로퍼티를 지정하는 방식보다 코드 안정성이나 생산성 측면에서 유리하다.
>
> ```java
> // 정적 메타 모델 사용
> cb.equal(root.get(OrderSummary_.ordererId), ordererId)
> // 문자열 프로퍼티 활용 (컴파일 타임에 오타가 나도 오류 잡기 불가)
> cb.equal(root.<String>get("ordererId"), ordererId)
> ```

* 스펙 생성 기능을 별도의 유틸 클래스에 모아두고 사용하면 편리하다. 스펙 인터페이스는 함수형 인터페이스이기 때문에 람다식을 사용해 쉽게 생성할 수 있다.

```java
public class OrderSummarySpecs {
	public static Specification<OrderSummary> ordererId(String ordererId) {
		return (Root<OrderSummary> root, CriteriaQuery<?> query,
		CriteriaBuilder cb) ->
			cb.equal(root.<String>get("ordererId"), ordererId);
	}
	
	public static Specification<OrderSummary> orderDateBetween(
		LocalDateTime from, LocalDateTime to) {
		return (Root<OrderSummary> root, CriteriaQuery<?> query,
		CriteriaBuilder cb) ->
			cb.between(root.get(OrderSummary_.orderDate), from, to);
	}
}
```

### 리포지토리에서 스펙 사용하기

* findAll 메서드의 인자로 스펙 인터페이스를 입력받기 때문에, 스펙 구현 객체를 findAll 메서드에 인자로 넣어 호출하면 원하는 조건에 맞는 엔티티를 조회할 수 있다.
* 아래는 user1이라는 OrdererId를 가진 OrderSummary의 모든 엔티티를 조회하기 위한 코드이다.

```java
Specification<OrderSummary> spec = new OrdererIdSpec("user1");

List<OrderSummary> results = orderSummaryRepository.findAll(spec);
```

### 스펙 조합

* Spring Data JPA의 Specification 인터페이스는 스펙을 조합할 수 있도록 and, or 디폴트 메서드를 제공한다.

```java
Specification<OrderSummary> spec =
	OrderSummarySpecs.ordererId("user1").and(OrderSummarySpecs.orderDateBetween(from, to));
```

* not 정적 메서드를 제공하여 조건을 반대로 적용할 수 있다.

```java
Specification<OrderSummary> spec =
	Specification.not(OrderSummarySpecs.ordererId("user1"));
```

* where 메서드를 사용해 nullable한 Specification이 생성되더라도 예외를 발생시키지 않고 아무 조건도 생성하지 않는 스펙 객체를 반환한다.

```java
Specification<OrderSummary> spec = 
	Specification.where(createNullableSpec()).and(createOtherSpec());
```

## 정렬과 페이징

### 정렬 기준 지정

* 메서드 이름에 원하는 기준을 정의하여 정렬 조건을 설정할 수 있다.
* 아래는 ordererId 프로퍼티 값을 만족하는 엔티티들을 number 프로퍼티 값 역순으로 정렬해 조회할 수 있는 메서드이다.

```java
public interface OrderSummaryRepository extends Repository<OrderSummary, String> {
	List<OrderSummary> findByOrdererIdOrderByNumberDesc(String ordererId);
}
```

* 두 개 이상의 프로퍼티에 대한 정렬 순서도 지정할 수 있다.

```java
public interface OrderSummaryRepository extends Repository<OrderSummary, String> {
	List<OrderSummary> findByOrdererIdOrderByOrderDateDescNumberAsc(String ordererId);
}
```

* 정렬 기준 프로퍼티가 많아지면 메서드명이 너무 길어지기 때문에 Sort 타입을 사용해 정렬 순서를 지정할 수 있다.

```java
Sort sort = Sort.by("number").ascending().and(Sort.by("orderDate").descending());

List<OrderSummary> results = orderSummaryRepository.findByOrdererId("user1", sort);
```

### 페이징 처리하기

* Spring Data JPA는 페이징 처리를 위해 Pageable 타입을 이용한다. Pageable 타입은 인터페이스로 실제 Pageable 타입 객체는 PageRequest 클래스를 이용해서 생성한다.
* PageRequest에 페이지 번호와 한 페이지에 몇 개를 보여줄 지 지정한 후 조회 메서드를 호출하면 된다.

```java
PageRequest pageReq = PageRequest.of(1, 10);
List<MemberData> user = memberDataRepository.findByNameLike("사용자%", pageReq);
```

* PageRequest에 Sort 구현체를 입력하면 정렬 순서를 지정할 수 있다.

```java
PageRequest pageReq = PageRequest.of(1, 2, Sort.by("name").descending());
List<MemberData> user = memberDataDao.findByNameLike("user%", pageReq);
```

* Page 타입을 반환받으면 데이터 목록뿐만 아니라 조건에 해당하는 전체 개수도 구할 수 있다.

```java
public interface MemberDataDao extends Repository<MemberData, String> {
	Page<MemberData> findByBlocked(boolean blocked, Pageable pageable);
}
```

```java
Page<MemberData> page = memberDataDao.findByBlocked(false, pageReq);
List<MemberData> content = page.getContent(); // 조회 결과 목록
long totalElements = page.getTotalElements(); // 조건에 해당하는 전체 개수
int totalPages = page.getTotalPages(); // 전체 페이지 번호
int number = page.getNumber(); // 현재 페이지 번호
int numberOfElements = page.getNumberOfElements(); // 조회 결과 개수
int size = page.getSize(); // 페이지 크기
```

* 프로퍼티를 비교하는 findBy프로퍼티 형식의 메서드는 Pageable 타입을 사용하더라도 리턴 타입이 List면 COUNT 쿼리를 실행하지 않는다.
* 스펙을 사용하는 메서드에 Pageable 타입을 함께 쓰면 리턴 타입이 Page가 아니어도 COUNT 쿼리를 실행한다. 스펙을 사용하고 페이징 처리를 하면서 COUNT 쿼리는 실행하고 싶지 않다면 스프링 데이터 JPA가 제공하는 커스텀 리포지터리 기능을 이용해서 직접 구현해야 한다.

### 스펙 빌더

* 조건에 따라 특정 스펙을 조합할 지 말 지 다루기 위해 스펙 빌더 만들어 사용하면 코드가 간결해진다.

```java
Specification<MemberData> spec = SpecBuilder.builder(MemberData.class)
	.ifTrue(searchRequest.isOnlyNotBlocked(),
					() -> MemberDataSpecs.nonBlocked())
	.ifHasText(searchRequest.getName(),
						 name -> MemberDataSpecs.nameLike(searchRequest.getName()))
	.toSpec();
List<MemberData> result = memberDataDao.findAll(spec, PageRequest.of(0, 5));
```

```java
public class SpecBuilder {
	public static <T> Builder<T> builder(Class<T> type) {
		return new Builder<T>();
	}
	public static class Builder<T> {
		private List<Specification<T>> specs = new ArrayList<>();

		public Builder<T> and(Specification<T> spec) {
			specs.add(spec);
			return this;
		}

		public Builder<T> ifHasText(String str,
			Function<String, Specification<T>> specSupplier) {
			if (StringUtils.hasText(str)) {
			specs.add(specSupplier.apply(str));
			}
			return this;
		}

		public Builder<T> ifTrue(Boolean cond,
			Supplier<Specification<T>> specSupplier) {
			if (cond != null && cond.booleanValue()) {
			specs.add(specSupplier.get());
			}
			return this;
		}

		public Specification<T> toSpec() {
			Specification<T> spec = Specification.where(null);
			for (Specification<T> s : specs) {
			spec = spec.and(s);
			}
			return spec;
		}
	}
}
```

## 동적 인스턴스와 @Subselect

### 동적 인스턴스

* JPA는 쿼리 결과에서 임의의 객체를 동적으로 생성하는 기능을 제공한다. 아래와 같이 select 절에서 조회 결과를 사용해 바로 새로운 객체를 생성할 수 있다.

```java
public interface OrderSummaryRepository extends Repository<OrderSummary, String> {
@Query("""
	select new com.myshop.order.query.dto.OrderView(
	o.number, o.state, m.name, m.id, p.name
	)
	from Order o join o.orderLines ol, Member m, Product p
	where o.orderer.memberId.id = :ordererId
	and o.orderer.memberId.id = m.id
	and index(ol) = 0
	and ol.productId.id = p.id
	order by o.number.number desc
	""")
	List<OrderView> findOrderView(String ordererId);
}
```

* OrderView와 같이 조회 전용 모델을 사용하여 표현 영역에서 사용자에게 꼭 필요한 데이터만을 보여주도록 할 수 있다.

### @Subselect

* 쿼리 결과를 @Entity로 매핑할 수 있는 기능이다.
* 조회 쿼리를 어노테이션의 값으로 입력하여, 쿼리 실행 결과를 매핑할 테이블처럼 사용한다.
* @Subselect의 값으로 지정한 쿼리는 from 절의 서브 쿼리로 사용한다. 즉 findById 등의 메서드를 사용해 조회하고자 할 때 from 절 내부에 @Subselect에 정의한 서브 쿼리가 들어가게 되는 것이다.

```java
@Entity
@Immutable
@Subselect(
	"""
	select o.order_number as number,
	o.version, o.orderer_id, o.orderer_name,
	o.total_amounts, o.receiver_name, o.state, o.order_date,
	p.product_id, p.name as product_name
	from purchase_order o inner join order_line ol
		on o.order_number = ol.order_number
		cross join product p
	where	ol.line_idx = 0
	and ol.product_id = p.product_id"""
)
@Synchronize({"purchase_order", "order_line", "product"})
public class OrderSummary {
	@Id
	private String number;
	
	@Column(name = "orderer_id")
	private String ordererId;

	private String productId;
	// ...
}
```

* @Subselect를 이용해 조회한 엔티티의 필드를 수정하면 하이버네이트는 변경 내역을 반영하는 update 쿼리를 실행하지만 매핑 한 테이블이 없으므로 에러가 발생한다. 이러한 문제 발생을 방지하기 위해 @Immutable을 사용하여 해당 엔티티의 매핑 필드/프로퍼티가 변경되더라도 DB에 반영하지 않고 무시하도록 한다.
* @Synchronize를 이용해 엔티티와 관련된 테이블에 변경이 발생하도록 하는 메서드가 호출되었을 경우 이를 먼저 플러시하고 조회하도록 한다.
