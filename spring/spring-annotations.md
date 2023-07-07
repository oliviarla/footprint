# Spring Annotations

### @Value

* property 설정 파일의 값을 쉽게 주입시켜주는 어노테이션

### @JsonProperty

* 스네이크 케이스로 받은 JSON 데이터를 객체 내 케멀케이스 변수에 매핑시킬 때 사용

### @RequestBody

* @NoArgsConstructor를 생성해주어야 한다.
* JSON 형태로 들어온 Http Message Body를 읽기 위해 사용
* &#x20;생략이 불가능하다. @RequestBody는 생략할 경우 우선 순위인 @ModelAttribute가 적용되어 문자열로 변환된다. 따라서 객체로 바인딩하려면 json 파서를 한 번 더 거쳐야 한다.

### @ModelAttribute

* HTTP 요청에서 form 형태로 들어온 데이터를 읽기 위해 사용
* 생략 가능하다.

### @RequestParam

* 생략 가능하다. String이나 int 같은 단순 타입을 사용할 때는 @RequestParam이 붙고, 객체 등을 사용할 때는 @ModelAttribute가 붙는다.
* @RequestParam은 query parameter를 받을 때 사용하는 어노테이션이다.
* Boxing Type을 사용하는지 아닌지에 따라 발생하는 예외의 종류가 다르다.
* 아래는 Boolean과 boolean 타입을 사용하는 예시로, 값이 없다면 NULL을 저장할지 false를 저장할지 달라질 수 있다. 따라서 가급적 Boxing Type으로 입력받는 것이 의도치 않은 로직 수행을 방지할 수 있을 것 같다.

<table><thead><tr><th width="173.33333333333331">RequestParam 옵션</th><th>query parameter null로 선언  (?param1=)</th><th>query parameter 선언 X</th></tr></thead><tbody><tr><td>required = true</td><td><p>Boolean - 예외 발생하지 않고 NULL 저장</p><p>boolean - MethodArgumentTypeMismatchException</p></td><td>Boolean - Required boolean parameter 'replication' is not present<br>boolean - Required boolean parameter 'replication' is not present</td></tr><tr><td>required = false</td><td>Boolean - 예외 발생하지 않고 NULL 저장<br>boolean - MethodArgumentTypeMismatchException</td><td>Boolean - 예외 발생하지 않고 NULL 저장<br>boolean - 예외 발생하지 않고 false 저장</td></tr></tbody></table>
