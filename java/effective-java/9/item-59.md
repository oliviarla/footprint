# item 59) 라이브러리를 익히고 사용하라

## 직접 구현한 메서드

* 아래는 직접 random integer를 구하는 메서드를 구현한 것이다. 하지만 여러가지 결함(값의 쏠림, 범위 외의 수 반환 등)이 있으니, 직접 구현하지 말고 오랜 기간동안 검증되어 온 `Random.randomInt(int n)` 메서드를 사용하도록 하자.

```cpp
static Random rnd = new Random();

static int random(int n) {
	return Math.abs(rnd.nextInt()) % n;
}
```

## 표준 라이브러리 사용의 장점

* 핵심 로직과 크게 관련없는 문제 해결에 시간을 쓰지 않아도 된다.
* 따로 노력하지 않아도 성능이 지속해서 개선된다.
* 릴리즈마다 기능이 점점 많아진다.
* 다른 개발자들이 읽기 좋고 유지보수하기 좋고 재활용하기 쉬운 코드가 된다.
* 적어도 java.lang, java.util, java.io와 그 하위 패키지들에는 익숙해져야 한다.
* 표준 라이브러리에서 원하는 기능을 찾지 못하면, 서드파티 라이브러리를 찾아보고, 정 없으면 직접 구현해야 한다.