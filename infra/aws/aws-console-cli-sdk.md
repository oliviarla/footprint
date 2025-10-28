# AWS Console / CLI / SDK

## Console

* 웹 사이트로 접근하여 AWS 리소스를 관리하는 방식
* 비밀번호와 MFA로 보호된다.
* CloudShell
  * 브라우저를 통해 AWS 콘솔에서 CLI를 사용할 수 있다.
  * 일부 region에서만 사용 가능하다.
  * AWS CLI와 동일하게 aws 명령을 사용할 수 있다.

## CLI

* CLI 환경에서 AWS 리소스를 관리하는 방식
* AWS Console을 통해 생성된 access key를 통해 보호된다.
* 다음 명령을 통해 access key id, access key, region name 등을 설정할 수 있다.

```
aws configure
```

* 이외에도 다양한 aws 명령을 통해 리소스를 관리할 수 있다.

## SDK

* 코드에서 사용할 수 있는 AWS 리소스 관리 라이브러리를 제공하는 방식
* 애플리케이션을 통해 AWS 리소스를 관리하게 된다.
* AWS CLI는 Python SDK를 이용해 구현되었다.
* AWS Console을 통해 생성된 access key를 통해 보호된다.

