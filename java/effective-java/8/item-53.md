# item 53) 가변인수는 신중히 사용하라

## 가변인수 메서드

* 매개변수를 가변적으로 조정가능하며 명시한 타입의 인수를 0개 이상 받을 수 있다.
* 인수의 개수와 길이가 같은 배열 생성 후 인수들은 배열에 저장하여 가변인수 메서드에 전달한다.
* `...`을 사용하여 가변인수임을 나타낸다.
* 인수 개수가 정해지지 않았을 때 유용하다.
* 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화하므로 성능에 민감할때는 조심해야 한다.
* 아래 예제는 여러 숫자를 받아 더하는 메서드이고, sum()을 하면 0이 반환된다.

```java
static int sum(int... args){
    int sum = 0;
    for (int arg : args)
        sum += arg;
    return sum;
}
```

## 가변인수 활용법

* 인수가 1개 이상이어야 할 경우, 매개변수를 2개 받도록 설계하면 된다.
* 첫번째 매개변수에는 평범한 매개변수 받고, 두번째 매개변수에는 가변인수를 받는다.

```java
static int min (int firstArg, int... args){
		int min = firstArg;
		for(int arg : args)
			if(arg < min) {
				min = arg;
			}
	return min;
}

```

## 가변인수와 다중 정의를 함께 사용하기

* 성능 이점을 보면서 가변인수 유연성이 필요할 때 사용할만한 패턴이다.
* 예를 들어 만약 메서드 호출의 95%가 인수를 3개 이하로 사용하고 5%는 4개 이상 사용하는 경우, 아래와 같이 인수가 0\~3개인 메서드를 다중정의한다.

```java
public void test(){}
public void test(int a1){}
public void test(int a1, int a2){}
public void test(int a1, int a2, int a3){}
public void test(int a1, int a2, int a3, int... rest){}// 5%의 호출 담당
```

* EnumSet의 정적팩터리의 열거 타입 집합 생성 비용도 이 패턴을 사용하여 최소화한다.

```java
public static <E extends Enum<E>> EnumSet<E> of(E e)
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2)
...
public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest)
```
