# item 33) 타입 안전 이종 컨테이너를 고려하라

## 타입 안전 이종 컨테이너

* 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다.
* 컨테이너 대신 **키를 매개변수화 한 다음 컨테이너에 값을 넣거나 뺄 때 매개변수화 한 키를 같이 제공**하는 설계 방식이다.

### **예제**

```java
public class Favorites {

    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);//해당 객체가 Null이면, NullPointerException 발생
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }

    //타입 안전 이종 컨테이너 패턴
    public static void main(String[] args) {
        Favorites f = new Favorites();

        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);

        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);

        System.out.println(favoriteString);
        System.out.println(favoriteInteger);

        Class<?> favoriteClass = f.getFavorite(Class.class);

        System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
    }

}
```

* getFavorite 메서드에서는 cast메서드를 사용해 Object 객체를 Class 객체가 가리키는 T 타입으로 동적 형변환한다.
* 제약 사항 1)
  * 악의적인 클라이언트가 Class 객체를 로 타입으로 넘기면 타입 안전성이 깨진다. 하지만 컴파일 단에서 경고가 뜨긴 할 것이다.
  * 타입 불변식을 어기는 일이 없도록 보장하려면, putFavorite 메서드에서 인수로 주어진 instance의 타입이 type으로 명시한 타입과 같은지 type.cast메서드를 사용해 확인하면 된다.
* 제약 사항 2)
  * Class 객체를 얻을 수 없는 **실체화 불가 타입(List 등과 같은 Collections..)에는 사용 불가**

## 한정적 타입 토큰 활용

* 단순히 한정적 타입 매개변수나 한정적 와일드카드를 사용해 표현가능한 타입을 제한하는 타입 토큰
* 메서드들이 허용하는 타입을 제한하고 싶을 때 사용한다.
* asSubClass 메서드를 통해 클래스는 형변환을 안전하게 수행하도록 하고, 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다.
