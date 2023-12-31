# item 76) 가능한 한 실패 원자적으로 만들라

## 실패 원자적이란?

* 호출된 메서드가 실패하더라도 해당 객체는 가능한 메서드 호출 전 상태를 유지해야 한다는 의미이다.
* 실패 원자성은 일반적으로 권장되지만, 보장하기 위한 비용이나 복잡도가 아주 큰 연산일 경우도 존재한다.
* 메서드 명세에 기술한 예외라면 예외가 발생하더라도 객체의 상태는 메서드 호출 전과 똑같이 유지돼야 한다는 것이 기본 규칙이다.
* 이 규칙을 지키지 못한다면 실패 시의 객체 상태를 API 설명에 명시해야 한다.

## 실패 원자적인 메서드 만드는 방법

* **불변 객체로 설계**한다. 생성 시점에 상태가 고정되므로, 메서드가 실패해도 기존 객체가 불안정한 상태에 빠질 일이 없다.
* 작업 수행하기 전 **매개변수의 유효성을 검사**한다.(item 49)
  * 객체 내부 상태를 변경하기 전에 잠재적 예외의 가능성 대부분을 걸러낼 수 있다.
  * 예를 들어 key를 기준으로 하여 원소들을 정렬하는 TreeMap은 엉뚱한 타입의 원소를 추가하려 들면 해당 원소가 들어갈 위치를 찾는 과정에서 ClassCastException을 던져, 변경 작업을 수행하지 않는다.
* 객체의 **임시 복사본에서 작업을 수행**한 다음, 작업이 성공적으로 완료되면 원래 객체와 교체한다.
* 작업 도중 발생하는 실패를 가로채는 **복구 코드를 작성**하여 작업 전 상태로 되돌린다. 주로 디스크 기반의 내구성을 보장하는 자료구조에 쓰이는데, 자주 쓰이는 방법은 아니다.
