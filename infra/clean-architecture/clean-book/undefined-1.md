# 아키텍처 경계 강제하기

### 경계와 의존성

* 의존성이 올바른 방향을 향하도록 강제한다
* 도메인 계층 : 도메인 엔티티 존재
* 애플리케이션 계층: 애플리케이션 서비스 내부에 유스케이스를 구현하기 위해 도메인 엔티티에 접근
* 어댑터 계층: 아웃고잉 포트를 통해 어댑터에 접근하고 어댑터는 인커밍포트를 통해 서비스를 접근
* 설정 계층: 어댑터와 서비스 객체를 생성할 팩토리 포함, 의존성 주입 매커니즘 제공

### 접근 제한자

* package-private (default) 제한자
* 자바 패키지를 통해 클래스들을 응집적인 모듈로 만들어줌
* 모듈 내 클래스들은 서로 접근 가능하지만 패키지 외부에서는 접근 불가
* 모듈의 진입점으로 활용될 클래스들만 골라 public으로 만들면 의존성 규칙 위반할 위험이 줄어든다!
* 의존성 주입은 일반적으로 리플렉션 사용해 클래스를 인스턴스로 만드므로, package-private이어도 의존성 주입 가능
  * 단 클래스패스 스캐닝을 이용해야만 가능하다
* 패키지 내 클래스가 너무 많아지면 문제가 발생할 수 있다.
  * 일부 클래스를 하위 패키지로 분리하게 되면 상위 패키지에서 하위 패키지에 접근할 수 없어 public으로 만들어야 한다. 이렇게 되면 의존성 규칙이 깨질 수 있다.

### 컴파일 후 체크

* 런타임 체크
  * 코드가 컴파일된 후 런타임에 의존성 규칙을 위반했는지 확인하는 방식
  * 지속적인 통합 빌드환경에서 자동화된 테스트 과정에서 잘 동작
  * ArcuUnit
    * 의존성 방향이 잘 설정되어 있는지 확인하는 API 제공
    * 의존성 규칙이 위반되면 테스트 실패
* 오타가 나거나 패키지명을 리팩토링하면 테스트 전체가 무의미해질 수 있으므로 항상 코드와 함께 유지보수해야 한다.

### 빌드 아티팩트

* 빌드 아티팩트: 자동화된 빌드 프로세스의 결과물 (패키징된 JAR 파일)
* 어떤 코드베이스를 빌드 아티팩트로 변환하기 위해 가장 먼저 코드베이스가 의존하고 있는 모든 아티팩트가 사용가능한지 확인한다.
* 사용불가능한 것이 있다면 아티팩트 리포지토리로부터 가져오려고 시도한다. 만약 실패하면 빌드 에러 발생
* 모듈 혹은 계층에 대해 전용 코드베이스와 빌드 아티팩트로 분리된 JAR 파일 생성 가능
* 각 모듈의 빌드 스크립트에서 아키텍처에서 허용하는 의존성만 지정해 각각 빌드하는 방식

<figure><img src="../../../.gitbook/assets/Untitled (2) (1).png" alt=""><figcaption></figcaption></figure>

* 하나의 어댑터 모듈을 여러개로 쪼개 다른 서드파티 API와 관련된 세부사항이 서로 새어나가지 않도록 방지 가능
* 애플리케이션 모듈을 입력, 출력 포트별로 나눌 수 있다.
* 모듈을 세분화할수록 모듈 간 의존성을 잘 제어할 수 있다.
* 하지만 모듈 간 매핑을 더 많이 수행해야 한다.
* 패키지로 구분하는 방식보다 순환 의존성이 없음을 확인하기 좋다.
* 다른 모듈을 고려하지않고 특정 모듈의 코드를 격리한 채로 변경 가능
* 에러를 해결중인 일부 파일이 있더라도 다른 계층에 있는 독립된 빌드 모듈은 바로 컴파일해서 테스트 가능
* 모듈간 의존성이 빌드 스크립트에 선언되어 있어 새로 의존성을 추가하는 일은 의식적인 행동이 된다.
* **아키텍처가 충분히 안정적일 때 독립적인 빌드 모듈로 추출해야 한다.**
