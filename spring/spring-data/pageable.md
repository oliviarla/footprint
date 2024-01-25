# Pageable

<figure><img src="../../.gitbook/assets/Untitled (2) (1).png" alt=""><figcaption></figcaption></figure>

## Pageable

* query string으로 page, size 정보가 들어왔을 때, controller에서 입력 파라미터로 사용 가능
* sort 객체를 입력 파라미터로 받거나 Direction 객체와 함께 properties를 입력받으면 정렬 기능도 수행해준다.
* 구성 요소
  * page : 요청할 페이지 번호
  * size : 한 페이지 당 조회 할 갯수 (default : 20)
  * sort : Sorting에 대한 값 설정하는 파라미터로, 기본적으로 오름차순이다. 표기는 정렬한 필드명,정렬기준 ex) createdDate,desc
* PageRequest는 Pageable 인터페이스의 구현체이며, 직접 Pageable 객체를 생성해줄 때 사용할 수 있다.
  * `PageRequst.of(page, size)`
  * `PageRequst.of(page, size, sort)`
  * `PageRequest.of(page,vsize, direction, "property1", "property2", ..)`

## Slice

```java
public interface Slice<T> extends Streamable<T> {

	int getNumber(); // 현재 페이지

	int getSize(); //페이지 크기

	int getNumberOfElements(); // 현재 페이지에 조회한 데이터 개수

	List<T> getContent(); // 현재 페이지에 조회한 데이터 목록 

	boolean hasContent(); // 현재 페이지에 데이터가 있는지 여부 

	Sort getSort(); // 정렬 정보

	boolean isFirst(); // 첫 번째 페이지인지 여부

	boolean isLast(); // 마지막 페이지인지 여부

	boolean hasNext(); // 다음 페이지가 있는지 여부

	boolean hasPrevious();  // 이전 페이지가 있는지 여부 

  	// 페이지 요청 정보
	default Pageable getPageable() {
		return PageRequest.of(getNumber(), getSize(), getSort());
	}

	// 다음 페이지 정보
	Pageable nextPageable();

	// 이전 페이지 정보
	Pageable previousPageable();

		
	<U> Slice<U> map(Function<? super T, ? extends U> converter);
	default Pageable nextOrLastPageable() {
		return hasNext() ? nextPageable() : getPageable();
	}
	
	default Pageable previousOrFirstPageable() {
		return hasPrevious() ? previousPageable() : getPageable();
	}
}
```

## Page

* Slice 인터페이스를 상속받은 인터페이스
* 전체 데이터 개수를 조회해 가지고 있기 때문에 전체 데이터 개수, 페이지 개수를 getTotalElements(), getTotalPages() 메서드로 얻을 수 있다.

## Page vs Slice

* 페이징 처리에 다양한 리턴타입을 제공하는 만큼 **요구사항에 따라서 Page와 Slice를 사용**하면 될 것 같다.
* **전체 페이지 개수와 전체 데이터 수를 이용하는 실제 게시판 형식의 페이징 처리를 구현해야한다면 Page를 사용**
* **무한 스크롤 방식을 사용한다면 전체 페이지와 데이터 개수가 필요없기 때문에 Slice 방식**을 이용하는 것이 효율적
