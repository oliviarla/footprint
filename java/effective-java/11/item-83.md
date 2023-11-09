# item 83) 지연 초기화는 신중히 사용하라

## 지연 초기화

* 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법
* 주로 최적화 용도로 사용
* 클래스와 인스턴스 초기화 시 발생하는 위험한 순환 문제를 해결하기 위해 사용

### 특징

* 대부분의 상황에선 일반적인 초기화가 나으므로, 필요한 상황이 아니라면 사용하지 말 것
* 인스턴스 생성 시 초기화 비용은 줄지만, 그 필드에 접근하는 비용은 커진다. 따라서 초기화가 이루어지는 비율, 실제 초기화에 드는 비용, 호출 빈도에 따라 오히려 성능이 느려질 수 있다.
* **인스턴스의 사용 빈도가 낮지만 초기화 비용이 클 때 효과적이다.** 하지만 이런 상황인지 아닌지 알기 위해 성능 측정이 필요하다.
* 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 반드시 동기화해야 하는데, 멀티 스레드 환경에서는 지연 초기화가 까다롭다.

### 지연 초기화 방법

#### 일반적인 초기화

* final 한정자를 사용한다.

```cpp
private final FieldType field = computeFieldValue();
```

#### 지연 초기화

* synchronized를 단 접근자를 사용해 초기화 순환성을 보장
* 정적 필드에도 똑같이 적용할 수 있다

```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
```

#### 지연 초기화 홀더 클래스

* 성능 때문에 **정적 필드를 지연 초기화할 경우** 사용
* 클래스가 처음 쓰일 때 초기화된다는 특성을 이용한 관용구이다.
* getField() 메서드가 호출되면 FieldHolder.field가 읽히면서 FieldHolder 클래스 초기화를 촉발한다.
* 동기화를 하지 않기 때문에 성능이 느려질 걱정이 전혀 없다는 장점이 있다.

```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```

#### 인스턴스 필드 지연 초기화 이중검사

* 성능 때문에 **인스턴스 필드를 지연 초기화할 경우** 사용
* 먼저 동기화 없이 검사해보고, 필드가 초기화되지 않았다면 동기화하여 검사해보고 초기화되지 않았다면 초기화한다.
* 초기화된 필드에 접근할 때의 동기화 비용을 없애준다.
* 필드가 초기화된 후로는 동기화하지 않으므로, 해당 필드는 volatile로 선언해야 한다.

```tsx
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null)// 첫 번째 검사 (락 사용 안 함)return result;

    synchronized(this) {
        if (field == null)// 두 번째 검사 (락 사용)
            field = computeFieldValue();
        return field;
    }
}
```

#### 단일검사

* 이중검사 시 반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화할 때 사용

```csharp
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null)
        field = result = computeFieldValue();
    return result;
}
```

#### racy single-check

* 단일검사해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 필드 선언에서 volatile 한정자를 제거해도 된다.
* 필드의 접근 속도를 높여주지만, 초기화가 스레드 당 최대 한 번 더 이뤄질 수 있다.
* 거의 사용되지 않는다.
