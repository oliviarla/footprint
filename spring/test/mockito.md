# Mockito

## 어노테이션

* Mockito 라이브러리에서 제공하는 어노테이션에 대해 알아본다.
* 먼저 Mockito 어노테이션을 사용하려면 아래와 같이 테스트 클래스에 어노테이션을 달아주어야 한다.

```java
@ExtendWith(MockitoExtension.class)
class MyUnitTest {
    ...
}
```

### @Mock

* 가장 널리 쓰이는 어노테이션으로, Mock 객체를 생성하고 주입해준다.
* Mock 객체는 아무런 로직이 작성되지 않은 껍데기 객체이기 때문에 어떤 메서드에 어떤 값을 반환할지 직접 작성해주어야 한다.
* 보통 Service Layer에서 단위 테스트를 할 때 외부와 통신이 필요한 Repository 클래스는 Mocking하고 내부 로직이 잘 동작하는지 테스트하는 용도로 쓰인다.

```java
@Mock
private Calculator calc;

void add() {
  given(calc.add(anyLong(), anyLong()).willReturn(10);
  
  int result = calcService.add(5, 5);
  assertEquals(10, result);
}
```

### @InjectMock





### ArgumentCaptor





### @Spy

