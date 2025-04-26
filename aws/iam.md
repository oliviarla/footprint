# IAM

## IAM 권한

### 유저와 그룹

* 유저
  * AWS 계정 설정하는 경우 외에는 루트 계정 대신 IAM 계정을 만들어 사용해야 한다.
  * 사용자를 그룹에 할당하여 그룹 별로 권한을 할당할 수 있다.
  * 다양한 권한 종류가 존재한다.
* 그룹
* 정책
  * JSON 기반으로 설정할 수도 있다.

## 방어 매커니즘

* 비밀번호 정책 정의
  * IAM 콘솔에서 최소 비밀번호 길이, 포함해야 할 문자 타입, 만료 시간, 이전 비밀번호 재사용 금지 등의 정책을 설정할 수 있다.
* MFA (Multi Factor Authentication)
  * 다중 인증 (n단계 인증)
  * 관리자인 경우 많은 작업을 사용할 수 있다.
  * 비밀번호를 도난당하여도 등록된 물리적 장비가 있어야 로그인 가능하게 된다.
  * 다음 장치들을 사용할 수 있다.
    * Virtual MFA device (Duo Mobile, Google Authenticator, Authy 등 애플리케이션)
    * YubiKey (단일 보안키를 사용해 여러 루트 및 IAM 사용자를 지원한다.)
    * Hardware Key Fob MFA Device (장치로부터 TOTP 토큰을 생성해 사용한다.)

## IAM Roles

* AWS 서비스에 할당하는 권한을 IAM Roles으로 관리한다.
* 실제 사용자가 사용하지 않는다.
* AWS에서 정보를 조회하기 위해 IAM Roles으로 접근 권한을 부여해야 한다.

## IAM Security Tools

* IAM Credentials Report (자격 증명 보고서)
  * 계정에 있는 사용자와 다양한 자격 증명 상태를 포함한다.
* Access Advisor (액세스 관리자)
  * 사용자에게 부여된 서비스 권한과 서비스에 마지막으로 접근한 시간을 알려준다.
  * 어떤 권한이 사용되지 않는지 확인하여 최소 권한 원칙을 지킬 수 있다.
