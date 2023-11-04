# item 12) 항상 toString을 재정의할 것

## toString 메서드

* Object의 기본 제공 메서드이다.
* 간결하면서 사람이 읽기 쉬운 형태의 유익한 주요 정보를 모두 반환해야 한다.
* toString 메서드는 객체를 println, printf, 문자열 연결 연산자(+), assert 구문에 넘길 때, 혹은 디버거가 객체를 출력할 때 자동으로 불린다.

## 포맷의 문서화

* 반환값의 포맷을 문서화 할 지 정해야 한다. 포맷을 명시하면 객체는 표준적이고, 명확하고 가독성이 좋아진다.
* 단, 포맷을 한번 명시하면, 평생 해당 포맷을 지원해야 하므로 유연성이 줄어든다.
* 명시한 포맷에 맞는 문자열 <-> 객체 간 상호 전환할 수 있도록 정적 팩토리나 생성자를 제공해주면 좋다.
* 아래 예시는 long 값을 BigInteger 객체로 전환해주는 메서드이다.

```java
public static BigInteger valueOf(long val) {
// If -MAX_CONSTANT < val < MAX_CONSTANT, return stashed constantif (val == 0)
        return ZERO;
    if (val > 0 && val <= MAX_CONSTANT)
        return posConst[(int) val];
    else if (val < 0 && val >= -MAX_CONSTANT)
        return negConst[(int) -val];

    return new BigInteger(val);
}
```

## toString 파싱하는 경우가 없도록 하기

* toString으로밖에 정보를 얻지 못한다면 클라이언트가 직접 파싱하는 불상사가 발생한다.
* toString의 반환값에 포함된 정보를 얻어올 수 있는 **별도의 API를 반드시 제공**해야 한다.

## 주의 사항

* 정적 유틸리티 클래스나 Enum 타입은 toString을 따로 재정의할 필요 없다.
* 그러나, 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스일 경우 재정의가 필요하다.
* 자동 생성에 적합하지는 않더라도 객체의 값에 관해 아무것도 알려주지 않는 Object의 toString보다는 **AutoValue로 생성**된 toString이 훨씬 유용하다. (item 10)
