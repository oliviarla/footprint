# 브리지 패턴

## 접근

* 사용자에 의해 매번 요구사항이 바뀌므로 구체적인 구현 부분과 추상화 부분을 모두 바꿀 수 있어야 한다.

## 개념

* 큰 클래스 또는 밀접하게 관련된 클래스들의 집합을 두 개의 개별 계층구조​(추상화 및 구현)​로 나눈 후 각각 독립적으로 개발할 수 있도록 하는 디자인 패턴이다.
* 예를 들어 모양과 색깔을 나타내야 할 때 Shape라는 공통 클래스를 두고 Rectangle, Triangle이 이를 구현할 수 있다. 만약 빨강, 파랑색을 나타내려면 RedRectangle, BlueRectangle, RedTriangle, BlueTriangle 처럼 클래스를 증식시킬 수도 있지만, Shape을 추상 클래스로 만들고 Color 인터페이스를 컴포지트하는 형태로 만들면 더 유연한 설계가 된다.
* 아래 다이어그램에서 Abstraction이 Implementation이라는 인터페이스를 필드로 갖는 것을 **브리지**라고 한다.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## 장단점

* 장점
  * 구현과 인터페이스를 완전히 결합하지 않아 분리가 쉽고 독립적으로 확장 가능하다.
  * 구현 클래스가 바뀌어도 클라이언트에 영향이 가지 않는다.
* 단점
  * 코드가 복잡해진다.

## 사용 방법

* 하나의 클래스에 여러 독립적인 개념이 섞여있는지 확인한다.
* 클라이언트가 필요로 하는 작업들을 추상 클래스에 정의한다.
* 추상 클래스에서 필요한 개념들을 독립적인 인터페이스로 선언하고, 이를 추상 클래스에서 참조하도록 한다.
* 인터페이스에 대한 구현 클래스들을 만든다.