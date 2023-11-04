# item 28) 배열보다 리스트를 사용하라

## 배**열과 제네릭 타입의 차이**

* 배열은 공변(함께 변함)이다. Sub가 Super의 하위 타입이라면 배열 Sub\[]는 배열 Super\[]의 하위 타입이다.
* 제네릭은 불공변이다. List\<Type1>와 List\<Type2>는 서로의 하위 타입도 상위 타입도 아니다. 문제가 있는건 배열이다.
* 배열의 문제점은 아래와 같이 Object 배열에 String 타입도 넣고 Integer 타입도 넣게 되면 컴파일에는 잘 동작하지만 런타임에서 ArrayStoreException이 발생한다는 것이다.

```java
Object[] stringArray = new String[3];
stringArray[1]= 251;
```

* 배열은 실체화된다. 즉 런타임에 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 런타임에 type safe하지만 컴파일 타임에는 safe하지 않다.
* 제네릭은 컴파일타임에만 원소 타입을 검사하고, 타입 정보가 런타임에는 소거된다. (소거를 통해 제네릭 지원 전의 하위 호환성을 유지) 따라서 컴파일 타임에는 safe하고 런타임에는 type safe하지 않다. (배열과 정반대)
* 따라서 배열과 제네릭을 섞어 사용하다가 컴파일 오류나 경고가 나오면, **배열을 리스트로 대체하자.**

## **리스트와 제네릭 사용**

* 리스트는 아래와 같이 \<Long> 타입의 리스트 인스턴스를 \<Object> 타입에 넣을 수 없고, 컴파일이 되지 않으므로 오류 확인이 쉽다.
* 아래의 경우 다음 컴파일 오류가 발생한다.
  * `incompatible types: java.util.ArrayList<java.lang.Long> cannot be converted to java.util.List<java.lang.Object>`

```java
List<Object> stringList = new ArrayList<Long>();
```

* E, List\<E>, List\<String> 같은 타입은 런타임에 컴파일 타임보다 적은 타입 정보를 가져 실체화 불가 타입이다.
* 매개변수화 타입 가운데 실체화될 수 있는 타입은 List\<?>와 Map\<?,?> 같은 비한정적 와일드카드 타입뿐이다.
* 제네릭 타입과 가변 인수 메서드(item53)을 함께 쓸 때 실체화 불가 타입이므로 가변 인수 매개변수를 저장할 배열을 만들 수 없어 경고 메시지가 발생한다. @SafeVarargs 어노테이션을 사용하여 대처할 수 있다. (item32)
* 배열로 형변환 시 **제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 배열대신 리스트를 사용**하면 해결된다.
* 성능은 저하되어도 타입 안전성과 상호운용성이 좋아진다.

## **제네릭을 사용해야 하는 예시**

```tsx
public class Chooser {
	private final Object[] choiceArray;
	public Chooser(Collection choices) {
		choiceArray = choices.toArray();
	}
	public Object choose() {
		Random rnd = ThreadLocalRandom.current();
		return choiceArray[rnd.nextInt(choiceArray.length)];
	}
}
```

* 사용자는 choose 메서드에서 반환된 Object(생성자에서 입력받은 choices 중 하나)를 알아서 형변환해 사용해야 한다. 이 때 타입이 다른 원소가 들어있으면 형변환 오류가 런타임에 발생할 것이다.
* 위 문제를 방지하기 위해 배열은 리스트로 변경하고, 클래스를 제네릭으로 만들어야 한다.

```tsx
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }

    public static void main(String[] args) {
        List<Integer> intList = List.of(1, 2, 3, 4, 5, 6);

        Chooser<Integer> chooser = new Chooser<>(intList);

        for (int i = 0; i < 10; i++) {
            Number choice = chooser.choose();
            System.out.println(choice);
        }
    }
}
```

* choices에 입력되는 형식을 T 배열로 만드려고 하니 제네릭을 사용 시 원소의 타입 정보가 런타임에 소거되고, 배열은 런타임에 원소의 타입 정보를 확인하기 때문에, **런타임 safe하지 않다**. 따라서 T 리스트를 사용하여 ClassCastException을 차단한다.
* choose메서드의 반환 타입도 Object에서 T 로 변경하여 사용자가 형변환하는 수고로움을 덜어줄 수 있다.
