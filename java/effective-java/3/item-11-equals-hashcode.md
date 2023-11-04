# item 11) equals 재정의 시 hashCode도 재정의하라

## hashCode 메서드의 일반 규약

* equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값 반환해야 한다.
* equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
* equals(Object)가 두 객체를 다르다고 판단했을 때, 두 객체의 hashCode가 반드시 서로 다른 값을 반환할 필요는 없다.
* 서로 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

## hashCode 메서드 재정의

* equals 메서드를 재정의했는데, hashCode를 재정의 하지 않으면 논리적 동치인 두 객체가 서로 다른 해시코드를 반환한다.
* 동치인 모든 객체에서 같은 해시코드를 반환할 경우, 해시 테이블의 버킷 하나에 모든 객체가 담긴다. 따라서 linked list처럼 동작하게 되어 객체가 많아지면 O(n)으로 느려져 사용할 수 없다.
* 핵심 필드를 사용한 전형적인 hashCode 구현은 다음과 같다.

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

* 핵심 필드를 생략하면 해시 품질이 나빠져 해시 테이블의 성능을 떨어뜨릴 수 있다.
* 클래스가 불변이고 해시코드 계산 비용이 크다면 초기화 값을 0 등으로 지정해두고 호출될 때 계산해두는 방법을 사용할 수 있다.
* hashCode가 반환하는 값의 생성 규칙은 API 사용자에게 알리지 않아야 클라이언트가 이 값에 의지하지 않고, 추후 계산 방식을 변경할 수 있다.
