# VPC

## CIDR

* 클래스 없는 도메인 간 라우팅
* IP 주소를 할당하는 방법이다.
* 서브넷을 통해 IP 주소의 범위를 표현할 수 있다.
* `<BaseIP>/<Subnet Mask>` 과 같이 정의된다.
  * BaseIP
    * 범위에 포함되는 IP
    * 32 bit로 표현되며, 8 bit로 쪼개 옥텟이라는 단위로 부른다.
  * Subnet Mask
    * IP 주소 중 변경이 가능한 비트 수를 정의한다.
    * `/0`, `/24`, `/32`  처럼 표현할 수 있으며, 이는 32 bit 중 몇 개의 비트가 고정되어 있는지를 표현한다.
    * `/0` : 변경 가능한 비트 수가 32개이다. 따라서 모든 IP를 의미한다.
    * `/24` : 변경 가능한 비트 수가 (32 - 24 = 8)개이므로 (2^8 = 256)개씩 IP를 쪼갰을 때 BaseIP가 존재하는 범위를 의미한다. 예를 들어 192.168.0.0/24 는 192.168.0.0 \~ 192.168.0.255의 범위를 나타내게 된다.
    * `/26` : 변경 가능한 비트 수가 (32 - 26 = 6)개이므로 (2^6 = 64)개씩 IP를 쪼갰을 때 BaseIP가 존재하는 범위를 의미한다. 예를 들어 192.168.0.0/26 는 192.168.0.0 \~ 192.168.0.63의 범위를 나타내게 된다.
    * `/16` : 변경 가능한 비트 수가 (32 - 16 = 16)개이므로 (2^12 = 65536)개씩 IP를 쪼갰을 때 BaseIP가 존재하는 범위를 의미한다. 예를 들어 192.168.0.0/16 는 192.168.0.0 \~ 192.168.255.255의 범위를 나타내게 된다.
    * `/32` : 변경 가능한 비트 수가 없다. 따라서 하나의 BaseIP를 의미하게 된다.
* Public vs Private
  * 사설 IP 대역
    * `10.0.0.0/8`
      * 10.0.0.0 \~ 10.255.255.255
    * `172.16.0.0/12`
      * 172.16.0.0 \~ 172.31.255.255
      * AWS 기본 VPC 대역
    * `192.168.0.0/16`
      * 192.168.0.0 \~ 192.168.255.255
  * 이외의 대부분 IP는 공인 IP 대역에 해당한다.&#x20;

## VPC

* Virtual Private Cloud
* AWS 클라우드에 있는 사설 네트워크
* AWS 리소스를 배포하는 곳이다.
* 리전마다 하나씩 존재하며 최대 5개까지 만들 수 있으나 더 늘릴 수도 있다.
* 새로운 AWS 계정을 생성하면 기본 VPC가 배정된다. 새로운 EC2 인스턴스 생성 시 서브넷을 따로 지정하지 않으면 기본 VPC에 생성되고 인터넷에 연결 가능하며 공인 IP를 갖게 된다. 또한 public, private IPv4 DNS 이름을 갖게 된다.

### CIDRs

* VPC에서 허용하는 IP 범위(CIDR 범위)를 가질 수 있다. 최대 5개의 범위를 지정할 수 있다.
* CIDR의 최소 크기는 `/28`이고, 최대 크기는 `/16`이다.
* VPC 자체가 private이므로, 사설 IP 대역이 할당된다.
* **다른 VPC나 네트워크와 겹치지 않게 IP 범위를 설정**해야 한다.

### 서브넷

> public subnet: 공용 서브넷
>
> private subnet: 사설 서브넷

* VPC 내부에는 네트워크를 분할하기 위한 서브넷이 있다. 서브넷은 AZ 리소스이다.
* 공용 서브넷은 인터넷으로부터 접근 가능하다.
* 사설 서브넷은 인터넷에서 접근할 수 없다. 사설 서브넷에 있는 EC2 인스턴스 역시 인터넷에 접근할 수 없다.
* VPC 내에서 여러 라우팅 테이블을 정의하여 인터넷으로의 접근과 서브넷 사이의 접근을 정의한다.

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

## 게이트웨이

### 인터넷 게이트웨이

* VPC의 리소스를 인터넷에 연결하도록 허용하는 EC2 인스턴스나 람다 함수 등을 의미한다.
* 공용 서브넷은 인터넷 게이트웨이로 라우팅된다.
* 수평으로 확장되고 고가용성을 보장하는 관리형 리소스이다.
* VPC와는 별개로 생성해야 하고 VPC는 인터넷 Gateway 하나에만 연결된다.
* 라우팅 테이블을 수정해 리소스가 라우터를 통해 인터넷 게이트웨이와 연결되도록 해주어야 비로소 인터넷 연결이 가능해진다.

### NAT 게이트웨이

* 사설 서브넷의 리소스로부터 인터넷에 접근하게 하고, 인터넷에서 사설 서브넷에는 접근하지 못하게 하기 위해 사용된다.
* AWS에서 관리하기 때문에 프로비저닝과 스케일링 관련해서 신경 쓸 필요가 없다.
* NAT 게이트웨이를 공용 서브넷에 배포하고, 사설 서브넷에서 NAT 게이트웨이로 라우팅한다. 공용 서브넷에 있던 NAT 게이트웨이는 인터넷 게이트웨이를 통해 라우팅되어 인터넷에 접근 가능하게 된다.
* 특정 AZ에 Elastic IP를 통해 생성된다.
* 같은 서브넷 상의 EC2 인스턴스를 사용할 수 없다. 따라서 다른 서브넷에서 접근할 때에만 NAT 게이트웨이가 유용하다.
* 초당 5Gbps 대역폭을 제공하며 자동으로 45Gbps까지 확장 가능하다.
* **보안 그룹을 관리할 필요가 없다.**
* 다중 AZ에서 각각 NAT 게이트웨이를 배포해두면 fault tolerance를 보장할 수 있다.
*   한 AZ가 중지되면 해당 AZ의 NAT 게이트웨이와 EC2 인스턴스 모두 정상적인 상태가 아니게 되므로, 라우팅 테이블로 AZ를 서로 연결하는 cross-AZ failover는 필요 없다.

    <figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>
* 서브넷 대역 중 5개의 IP (첫 4개, 마지막 1개)는 예약된 IP 주소이다.
  * 예를 들어 10.0.0.0/24 CIDR 블록을 지정했다면, 다음 주소들은 사용할 수 없다.
    * 10.0.0.0: 네트워크 주소
    * 10.0.0.1: AWS VPC router에 사용되는 주소
    * 10.0.0.2: Amazon-provided DNS에 매핑되는 주소
    * 10.0.0.3: 나중에 AWS에서 사용하기 위한 예비용 주소
    * 10.0.0.255: 네트워크 브로드캐스트 주소, AWS는 VPC 내에서 브로드캐스팅을 허용하지 않으므로 실제로 사용할 수는 없다.
* 29개의 IP 주소를 필요로 하는 경우 서브넷을 `/27` 로 두면 실제로 사용 가능한 주소는 (2^5 - 5 = 27)개 이므로 부족하다. 따라서 서브넷을 `/26`으로 두어야 한다.

### NAT Instance

* NAT 게이트웨이와 달리 자체적으로 관리할 수 있는 NAT 인스턴스도 있다. 하지만 점차 지원하지 않을 예정이다.

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

* 사설 서브넷에 있는 EC2 인스턴스가 인터넷으로 연결할 수 있도록 해준다.
* 공용 서브넷에 고유의 보안 그룹이 있는 NAT 인스턴스를 실행하고, 탄력적 IP를 연결한다.
* 라우팅 테이블을 수정하여 사설 서브넷과 공용 서브넷의 두 서브넷에 있는 EC2 인스턴스로부터 NAT 인스턴스로 트래픽을 전송하도록 한다.
* 사설 서브넷에 위치한 인스턴스의 IP는 NAT 인스턴스를 통과하면서 감춰진다. 즉, 실제 외부로 트래픽이 나갈 때에는 공용 서브넷의 NAT 인스턴스 IP가 출발지 주소가 된다.
* 외부 인터넷으로부터 NAT 인스턴스에게 응답이 오면 사설 EC2 인스턴스로 트래픽을 포워딩한다. 이 때의 동작 방식은 **Bastion Host와 동일**하다.
* NAT EC2 인스턴스는 소스 및 목적지 확인 설정을 비활성화해야 한다. NAT 인스턴스가 소스 및 목적지가 아니더라도 트래픽을 허용해야 하기 때문이다.
* 가용성이 높지 않고 초기화 설정으로 복원할 수 없어서 여러 AZ에 ASG를 생성해야 하고, 복원되는 사용자 데이터 스크립트가 필요하기에 복잡하다.
* 대역폭은 EC2 사양에 따라 달라진다.
* 포트 포워딩을 지원한다.
* **보안 그룹과 규칙을 관리**해주어야 한다.
  * 인바운드에서는, 사설 서브넷의 HTTP/HTTPS 트래픽을 허용하고 홈 네트워크의 SSH도 허용해야 한다.
  * 아웃바운드에서는 HTTP/HTTPS 트래픽을 허용해야 한다.

## Bastion Host

* **외부 인터넷으로부터 사설 서브넷에 접근하기 위해 사용되는 공용 서브넷에 있는 EC2 인스턴스**이다.
* 배스천 호스트로 사설 서브넷에 있는 EC2 인스턴스에 접근할 수 있다.
* 외부 사용자는 배스천 호스트에 SSH 연결하고, 배스천 호스트를 통해 사설 서브넷의 EC2 인스턴스에 연결하는 형태로 접근할 수 있다.
* 배스천 호스트 보안 그룹이라는 자체 보안 그룹이 있다. 보안 그룹에서는 인터넷으로부터의 SSH 접근을 허용해야 한다. 이 때 특정 공인 CIDR 범위에 대해서만 22번 포트를 허용해야 보안 상 안전하다.
* 사설 서브넷의 EC2 인스턴스 보안 그룹에서는 반드시 SSH 액세스를 허용해야 한다. 배스천 호스트의 사설 IP나 배스천 호스트의 보안 그룹만이 22번 포트에 접근할 수 있도록 해야 한다.

## 네트워크 보안

### 공용 서브넷 방어

* NACL
  * Network ACL
  * 방화벽과 같이 서브넷을 오고 가는 트래픽을 제어한다.
  * 허용 규칙과 거부 규칙을 통해 트래픽을 명시적으로 허용/거부할 수 있다.
  * 서브넷 수준에서 동작한다. 따라서 연결된 서브넷의 모든 EC2 인스턴스에 적용된다.
  * 규칙은 IP 주소를 대상으로 한다. 따라서 특정 IP 주소를 거부할 수 있다.
  * 기본 VPC가 있으면 **기본 NACL이 모든 트래픽의 인입을 허용**한다.
  * 보안 그룹과 달리 상태를 유지하지 않는다. 따라서 인바운드, 아웃바운드 규칙을 각각 설정해주어야 한다.
  * NACL 규칙
    * 1 \~ 32766 까지 우선 순위를 정할 수 있다. 100씩 증가시키면서 규칙을 정의하는 것을 권장한다.
    * 여러 규칙 중 가장 우선순위가 높아 처음 매칭되는 규칙을 적용한다.
    * 가장 마지막 규칙은 `*` 이고, 규칙에 매칭되지 않는 트래픽을 차단하도록 한다.
  * 새롭게 생성된 NACL은 기본적으로 모든 트래픽을 차단한다.
  * 임시 포트
    * 서버가 서비스를 제공할 때 클라이언트는 규정된 포트에 접속한다. 서버가 클라이언트에게 응답을 하려면 클라이언트에 연결해야 하는데, 클라이언트는 기본적으로 개방된 포트가 없다.
    * 따라서, 클라이언트가 서버에 접속 시 자체적으로 특정한 포트를 개방한다. 이 포트는 임시라서 클라이언트와 서버 간 연결이 유지되는 동안만 열려 있다.
    * 운영 체제에 따라 임시 포트 범위가 달라진다.
      * Windows 10: 49152 \~ 65,535
      * Linux: 32768 \~ 60999
    * 예를 들어 웹 계층 서브넷들에 매핑되는 웹 NACL과 DB 계층 서브넷들에 매핑되는 DB NACL가 있을 때 웹이 데이터베이스에 요청을 보낸 후 데이터베이스가 다시 웹으로 응답을 보낼 때, 웹 NACL의 인바운드는 DB 서브넷 CIDR의 임시 포트 범위의 TCP 트래픽을 허용해야 한다.
  * 다중 NACL 및 서브넷이 있다면 각 NACL의 조합이 NACL 내에서 허용되어야 한다. NACL에 서브넷을 추가하면 NACL 규칙도 업데이트해서 연결 조합이 유효한지 확인해야 한다.
* 보안 그룹
  * ENI라는 탄력적 네트워크 인터페이스 또는 EC2 인스턴스를 오고 가는 트래픽을 제어한다.
  * 허용 규칙만 존재한다.
  * 모든 규칙이 평가되고 난 후 트래픽 허용 여부를 결정한다.
  * IP 주소들이나 다른 보안 그룹들만 참조할 수 있다.
  * 인스턴스나 ENI에 연결된다.
  * 상태를 유지한다. 트래픽이 외부에서 시작해서 인바운드로 허용된다면, 아웃바운드 역시 허용되어 반환 트래픽이 전달된다.

## VPC 피어링

* 내부적으로 여러 VPC들을 AWS network를 이용해 하나의 VPC처럼 사용할 수 있게 연결할 수 있다.
* 즉, 여러 VPC를 묶어 같은 네트워크에 있는 것처럼 동작하게 할 수 있다. 서로 다른 VPC에 연결이 가능하게 되는 것이다.
* **VPC 간에 할당된 CIDR 범위가 겹치면 통신이 불가능**하므로 안된다.
* 통신하려는 각 VPC에 모두 VPC 피어링이 활성화되어 있어야 한다. **A, B, C 세 개의 VPC가 있고 A와 B, B와 C가 연결되어 있더라도, A와 C가 통신하도록 하려면 각각 VPC 피어링 연결을 활성화해야 한다.**
* 서로 다른 VPC의 EC2 인스턴스가 서로 통신할 수 있도록 각 VPC 서브넷의 모든 라우팅 테이블을 업데이트해야 한다.
* 서로 다른 AWS 계정이나 리전 사이의 VPC들을 피어링할 수 있다.
* 보안 그룹에서 다른 보안 그룹을 참조할 수 있다. 동일한 지역의 계정 전체에서 피어링된 VPC의 보안 그룹을 참조하는 것도 가능하다.
* 소스를 CIDR 또는 IP로 가질 필요가 없다. AWS 계정 혹은 보안 그룹을 소스로 사용할 수 있다.

## VPC 공유

* VPC 공유(리소스 액세스 매니저의 일부)를 이용하면 다수의 AWS 계정이 EC2 인스턴스, RDS 데이터베이스, Redshift 클러스터, 람다 함수 같은 애플리케이션 리소스를 공유되고 일원적으로 관리되는 Amazon 가상 사설 클라우드(VPC)로 만들 수 있다.
* VPC를 소유한 계정은 AWS 조직의 동일한 조직에 속한 다른 계정과 하나 이상의 서브넷을 공유할 수 있다.
* 서브넷이 공유된 후에 참여자는 공유된 서브넷에서 자신들의 애플리케이션 리소스를 열람, 생성, 변경, 삭제할 수 있다.
* 참여자들은 다른 참여자 또는 VPC 오너에게 속한 리소스를 열람, 변경 또는 삭제할 수 없다.
* 높은 수준의 상호연결성을 요구하고 동일한 신뢰성 경계 안에 있는 애플리케이션들을 위해 VPC 안의 암묵적 라우팅(implicit routing)을 활용할 수 있다.
* 청구와 액세스 통제를 위해 별도의 계정을 사용하므로 각 계정이 생성하고 관리하는 VPC의 개수가 감소한다.

## VPC 엔드포인트

* 모든 AWS 서비스는 퍼블릭에 노출되어있고, 퍼블릭 URL을 갖는다.
* AWS 내부에서 또다른 AWS의 서비스에 접근할 때 퍼블릭 인터넷을 거치지 않고 AWS 내부 네트워크만을 거칠 수 있는 기능을 VPC 엔드포인트가 제공한다.
* AWS에서 DynamoDB와 같은 서비스에 퍼블릭하게 접근 시 NAT Gateway나 인터넷 게이트웨이 또는 인터넷 게이트웨이를 통해 DynamoDB에 액세스하게 된다. AWS 내부 네트워크만을 통해 접근하지 않고 NAT, 인터넷 게이트웨이를 거치므로 비용이 발생하거나 비효율적이다.
* 프라이빗 서브넷에 있는 EC2 인스턴스가 VPC 내에 배포된 VPC 엔드포인트를 통해 직접 AWS 서비스에 접근하면, 네트워크가 AWS 내에서만 이루어지므로 효율적이다.
* VPC 엔드포인트는 AWS PrivateLink를 통해 내부 네트워크를 사용하게 된다.
* 중복과 수평 확장이 가능하다.
* 인터넷 게이트웨이나 NAT Gateway 없이도 AWS 서비스에 액세스할 수 있게 해주므로 네트워크 인프라를 간단하게 만들 수 있다.
* 문제가 발생하면 VPC에서 DNS 설정 해석이나 라우팅 테이블을 확인하면 된다.

### 유형

* 인터페이스(Interface) 엔드포인트
  * PrivateLink를 이용한다.
  *   엔트리 포인트로 ENI를 프로비저닝한다. ENI에는 반드시 보안 그룹을 연결해야 한다.

      > ENI: VPC의 프라이빗 IP 주소이자 AWS의 엔트리 포인트

      * 대부분의 AWS 서비스를 지원하고 시간 혹은 GB 단위로 요금이 청구된다.
  * 온프레미스에서 접근(Site to Site VPN, Direct Connect)하거나 다른 VPC, 리전에서 접근해야 할 때 적합하다. (인터넷을 거치지 않는 방식)
* 게이트웨이(Gateway) 엔드포인트
  * 게이트웨이를 프로비저닝한다.
  * 게이트웨이는 **반드시 라우팅 테이블의 대상이 되어야 한다. 대신 IP 주소를 사용하거나 보안 그룹을 사용하지 않는다.**
  * **Amazon S3와 DynamoDB만 지원한다.**
  * 무료이다.
  * 단순히 라우팅 테이블 접근이므로 자동으로 확장된다.

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (199).png" alt="" width="375"><figcaption></figcaption></figure></div>

## VPC Flow 로그

* 인터페이스로 들어오는 IP 트래픽에서 정보를 캡처할 수 있다.
* VPC 수준이나 서브넷 수준 또는 탄력적 네트워크 인터페이스 수준에서 확인할 수 있다.
* VPC에서 일어나는 연결 문제를 모니터링하고 해결하는 데 유용하다. 사용 패턴을 분석하거나 악의적인 행동이나 포트 스캔 등을 탐지할 수 있다.
* Flow 로그를 Amazon S3, CloudWatch Logs, Kinesis Data Firehose에 전송할 수 있다.&#x20;
* ELB, RDS, ElastiCache, Redshift, WorkSpaces, NATGW, Transit Gateway 등 AWS의 관리형 인터페이스에서 발생하는 네트워크 정보 역시 캡처 가능하다.

### 구조

<figure><img src="../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

* 버전, 인터페이스 ID, 소스 주소, 대상 주소, 소스 포트, 대상 포트, 프로토콜, 패킷, 바이트 수, 시작, 끝, 액션, 로그 상태를 담는다.
  * 소스 주소와 대상 주소는 문제가 있는 IP를 식별하기 위해 사용된다. 특정 IP가 반복적으로 거부된다면, 그 IP에 문제가 있을 수 있다.
  * 소스 포트와 대상 포트는 여러분이 문제가 있는 포트를 식별하기 위해 사용된다.
  * 액션은 트래픽 허용 또는 거부를 의미하는데, 보안 그룹 또는 NACL 수준에서 성공 혹은 실패 여부를 알려준다.
* Flow Logs를 쿼리하는 방법
  * Athena를 S3에 사용하는 방법
  * CloudWatch Logs Insights 사용
    * 스트리밍 분석
* 인바운드/아웃바운드 둘 중 하나만 ACCEPT되고 하나는 REJECT 되었다면 NACL 관련 이슈이다. (보안 그룹은 인바운드/아웃바운드 중 하나가 ACCEPT 되면 나머지도 ACCEPT 처리되기 때문이다.)

### 아키텍처

* Flow 로그를 CloudWatch Logs로 보내고, CloudWatch Contributor Insights라는 걸 이용해 분석 가능하다. 예를 들어 네트워크에 가장 많이 기여하는 상위 10개 IP 주소를 확인할 수도 있다.
* Flow 로그를 CloudWatch Logs로 보내고, 메트릭 필터를 설정해서 특정 프로토콜을 검색할 수 있다. 평소보다 특정 메트릭이 많이 왔다면, CloudWatch Alarm을 트리거하고 Amazon SNS 토픽에 경보를 전송할 수 있다.
* Flow 로그를 S3 버킷에 전송해서 저장한 후 Amazon Athena를 사용해서 SQL로 된 VPC Flow Logs를 분석할 수 있다. Amazon QuickSight로 시각화도 가능하다.

<figure><img src="../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>

## 사이트 간 VPN

* AWS와 별개로 구축된 데이터 센터와 AWS를 프라이빗하게 연결하려면 기업은 CGW(Customer Gateway)를, VPC는 VPN 게이트웨이를 갖춰야 한다.
* 그리고 공용 인터넷을 통해 사설 Site-to-Site VPN을 연결한다. VPN 연결이라서 암호화된다.
* Site-to-Site VPN 연결의 최대 처리량은 1.25Gbps이다. 따라서 대역폭 요구사항이 낮거나 중간 정도인 경우에 적합하다.
* &#x20;더 높은 처리량이 필요한 경우 Equal Cost Multipath(ECMP) 라우팅을 사용할 수 있다. ECMP 라우팅은 전송 게이트웨이에 연결된 VPN 연결에 사용할 수 있다. ECMP 라우팅을 사용하면 여러 VPN 연결을 집계하여 보다 효과적인 처리량을 달성할 수 있다.
* VGW
  * Virtual Private Gateway
  * AWS 측에 있는 VPN concentrator
  * VGW는 Site-to-Site VPN 연결을 생성하려는 VPC에 연결된다.
  * ASN을 지정할 수도 있다.
* CGW
  * Custom Gateway
  * 외부에 존재하는 소프트웨어 혹은 물리적 장치에 연동되는 게이트웨이
  * 외부 장비의 IP와 연동된다.
  * ARN을 통해 AWS에서 온프레미스 VPN 기기에 접근 시 보안 연결을 설정할 수 있다.
* CGW가 public이라면 인터넷 라우팅이 가능한 public IP 주소가 존재하므로 이를 사용해 VGW와 연결하면 된다.
* NAT-T를 활성화하는 NAT 장치를 두어 기본적으로 private IP를 갖도록 하고 CGW에서 public IP를 사용할 수 있다.

<figure><img src="../.gitbook/assets/image (202).png" alt="" width="375"><figcaption></figcaption></figure>

* 서브넷의 VPC에서 라우트 전파를 활성화해야 Site-to-Site VPN 연결이 실제로 작동한다.
* 온프레미스에서 AWS로 ping 명령을 보내 EC2 인스턴스 상태를 진단할 때, 보안 그룹 인바운드 ICMP 프로토콜이 활성화해야 한다.

### AWS VPN CloudHub

* VGW를 갖춘 VPC가 있고 고객 네트워크와 데이터 센터마다 CGW를 갖추었을 때, CloudHub는 여러 VPN 연결을 통해 모든 사이트 간 안전한 소통을 보장한다.
* 비용이 적게 드는 허브 앤 스포크 모델(hub\&spoke)로 VPN만을 활용해 서로 다른 지역 사이 기본 및 보조 네트워크 연결을 지원한다.
* 고객 네트워크는 VPN 연결을 통해 서로 소통할 수 있게 된다.
* VGW 하나에 Site-to-Site VPN 연결을 여러 개 만들어 동적 라우팅을 활성화하고 라우팅 테이블을 구성하면 된다.

<figure><img src="../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

## 다이렉트 커넥트(DX)

* 원격 네트워크로부터 VPC로의 전용 프라이빗 연결을 제공한다.
* 전용 연결을 생성해야 하고 AWS Direct Connect 로케이션을 사용한다.
* VPC에는 VGW를 설치해야 온프레미스 데이터 센터와 AWS 간 연결이 가능하다.
* 동일한 연결으로 Amazon S3 등의 퍼블릭 리소스와 EC2 인스턴스 등의 프라이빗 리소스에 퍼블릭 및 프라이빗 VIF를 사용해 액세스할 수 있다.
* **대역폭 처리량이 증가할 때, 큰 데이터 세트를 처리할 때 퍼블릭 인터넷을 거치지 않아 속도가 빠르다.**
* 프라이빗 연결을 사용하므로 비용이 절약된다.
* 퍼블릭 인터넷 연결에 문제가 발생해도 Direct Connect를 사용하면 연결 상태를 유지할 수 있다.
  * 실시간 데이터 피드를 사용하는 애플리케이션에 유용하다.
* 온프레미스 데이터 센터와 클라우드가 연결되어 있어 하이브리드 환경을 지원한다.
* IPv4와 IPv6 둘 다 지원한다.

### 아키텍처

<figure><img src="../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

* 리전과 기업 데이터 센터를 연결하려면 AWS Direct Connect 로케이션을 요청한다.
* 고객 혹은 파트너 라우터는 고객이나 파트너 케이지(Cage)에서 빌려 와야 한다.
* Direct Connect 로케이션에 케이지 두 개가 있고, 온프레미스 데이터 센터에는 방화벽이 있는 고객 라우터를 등록한다.
* 프라이빗 VIF(Virtual Interface)를 생성하여 VPC로 프라이빗 리소스에 접근한다. 따라서 기본적으로 인터넷을 거치지 않는다. 이 때 모든 로케이션을 연결하는 프라이빗 VIF를 생성하여 VGW에 연결해야 한다.
* 수동으로 연결해야 하므로 설치 과정이 한달 정도로 오래 걸릴 수 있다. 일주일 안에 빠르게 데이터를 전송하고 싶다면 Direct Connect를 사용할 수 없다.
* 퍼블릭 VIF를 설치하면 AWS 내 퍼블릭 서비스 (Amazon Glacier 또는 Amazon S3)에 연결할 수 있다.

### Direct Connect Gateway

<figure><img src="../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

* 여러 리전에 각기 다른 VPC와 CIDR가 존재하고 온프레미스 데이터 센터를 여러 VPC에 연결하는 경우 사용된다.
* Direct Connect 연결을 생성한 뒤 프라이빗 VIF를 사용해 Direct Connect Gateway에 연결한다.
* Direct Connect Gateway는 프라이빗 가상 인터페이스를 통해 VPC의 VGW에 연결된다.

### Direct Connect 타입

* 전용 연결
  * 용량은 초당 1Gbp 혹은 10Gbp이 한도이다.
  * AWS에 요청을 보내면 AWS Direct Connect 파트너가 물리적 전용 이더넷 포트를 할당해준다.
* 호스팅 연결
  * 초당 50Mbp, 500Mbp에서 10Gbp까지 다양하게 쓸 수 있다.
  * AWS Direct Connect 파트너를 통해 연결을 요청해야 한다.
  * 필요 시 용량을 추가하거나 제거하면 된다.
* 프라이빗 연결이므로 암호화가 필수는 아니지만, Direct Connect과 함께 VPN을 설치해서 IPsec으로 암호화된 프라이빗 연결을 할 수 있다.
  * Direct Connect 로케이션을 가져와 해당 연결에 VPN 연결을 구축하면 Direct Connect에 암호화가 구성된다.

### Direct Connect 복원력

* 복원력을 높이기 위해서는 여러 Direct Connect를 설치하는 것이 좋다.

<figure><img src="../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

* 예를 들어 기업 데이터 센터가 두 개 있고 Direct Connect 로케이션도 둘프라이빗 VIF가 하나 있는데&#x20;
* 하나의 연결을 여러 Direct Connect 로케이션에 수립하여 Direct Connect 하나가 망가져도 다른 하나가 남아 있으므로 복원력이 강해진다.
* 최대의 복원력을 달성하려면 여러 독립적인 연결을 하나 이상의 로케이션에서 각기 다른 장치에 도달하도록 구성하면 된다.

### Direct Connect 백업 연결

* Direct Connect를 통해 VPC에 연결할 때 연결에 문제가 발생할 경우를 대비해 보조 연결을 준비해두어야 한다.
* Direct Connect를 보조 연결로 두면 비용이 많이 든다.
* Site to Site VPN을 백업 연결로 두면 저렴하고 퍼블릭 인터넷에 연결할 수 있다.

## Transit Gateway

<figure><img src="../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

* &#x20;여러 개의 VPC를 피어링으로 연결하고, VPN 연결과 Direct Connect를 구성해 Direct Connect Gateway로 VPC들을 연결하면 네트워크 토폴로지가 복잡해진다.
* 이를 해결하기 위해 Transit Gateway를 도입할 수 있다.
* **전이적 피어링 연결이 VPC 수천 개, 온프레미스 데이터 센터 Site-to-Site VPN, Direct Connect 허브와 스포크 간 스타형 연결 사이에 생긴다.**
* Transit Gateway를 통해 VPC 여러 개를 연결하면, Transit Gateway를 통해 전이적으로 연결되므로 VPC를 서로 모두 피어링할 필요가 없다.
  * Direct Connect Gateway를 Transit Gateway에 연결하면 Direct Connect 연결이 각기 다른 VPC에 직접 연결된다.
  * 고객 게이트웨이와 VPN을 Transit Gateway에 연결하면 Transit Gateway 일부로 모든 VPC에 액세스하도록 지원할 수 있다.
* Transit Gateway는 리전 리소스이며 리전 간에 작동한다. 리전 간 Transit Gateway를 피어링할 수도 있다.
* Resource Access Manager를 사용해 Transit Gateway를 계정 간에 공유할 수 있다.
* 라우팅 테이블을 생성해 VPC가 통신 가능한 다른 VPC를 제한할 수 있다. 이를 통해 Transit Gateway 내 모든 트래픽 경로를 제어할 수 있다.
* **AWS에서 유일하게 IP 멀티캐스트를 지원하는 서비스이다.**
*   Site-to-Site VPN 연결 대역폭을 ECMP를 사용해 늘릴 수 있다.

    <figure><img src="../.gitbook/assets/image (190).png" alt=""><figcaption></figcaption></figure>

    > ECMP
    >
    > * Equal Cost Multi Path Routing, 등가 다중 경로 라우팅
    > * 여러 최적 경로를 통해 패킷을 전달하는 라우팅 전략이다.

    * Transit Gateway를 이용해 Site-to-Site VPN 연결을 구축하면 두 개의 터널이 각각의 연결을 통해 구성된다. VPC를 이용해 Site-to-Site 연결을 구축하면 하나의 연결에 두 터널이 구성된다.
      * Transit Gateway로 여러 Site-to-Site VPN을 만들면, Site-to-Site VPN에서 사용되는 터널이 증가해 연결 처리량이 증가한다.
      * VPN 터널 하나당 1.25Gbps 대역폭이 최대 처리량이 된다.
        * VPC를 이용해 Site-to-Site VPN 연결을 구축하면 최대 처리량은 그대로 1.25Gbps가 된다.
        * Transit Gateway에서 Site-to-Site VPN 연결을 구축하면 ECMP를 통해 최대 처리량이 2.5Gbps가 된다.
      * Transit Gateway를 사용하면 Site-to-Site VPN 하나가 여러 VPC에 생성된다.
* Direct Connect 연결을 여러 계정에서 공유할 때 Transit Gateway를 사용할 수 있다.
  *   외부 데이터 센터와 Direct Connect 로케이션 간에 Direct Connect 연결을 생성하고, Transit Gateway를

      서로 다른 계정이 소유하는 VPC들에 연결한다. 그리고 Direct Connection Gateway를 통해 Transit Gateway와 Direct Connection Loaction을 연결하면 된다.

## VPC 트래픽 미러링

<figure><img src="../.gitbook/assets/image (192).png" alt="" width="312"><figcaption></figcaption></figure>

* VPC에서 네트워크 기능을 방해하지 않으면서 트래픽을 수집하고 검사하는 기능
* 트래픽을 수집해 관리 중인 보안 어플라이언스(ENI, NLB 등)로 라우팅할 수 있다.
* 예를 들어 위 그림과 같이 ENI와 EC2 인스턴스 사이의 인바운드 및 아웃바운드 트래픽을 분석하려면 트래픽 미러링 기능을 이용해 NLB로 필터링된 트래픽을 보낼 수 있다. NLB에는 트래픽을 분석할 ASG를 매핑해 가용성을 보장한다.
* 동일한 VPC에 source와 target이 있어야 한다. VPC 피어링을 활성화한 경우라면 다른 VPC에 걸쳐 있을 수도 있다.
* 콘텐츠 검사와 위협 모니터링, 네트워킹 문제 해결 등에 활용할 수 있다.

## IPv6

* IPv4는 VPC와 서브넷에서 비활성화할 수 없다.
* IPv6의 경우 공개 IP 주소를 듀얼 스택 모드로 활성화할 수 있다.
* EC2 인스턴스에는 사설 IP가 있고 기본적으로 공용인 IPv6가 있다.
* 공용 IPv6를 지원하는 VPC의 서브넷에서 EC2 인스턴스를 생성할 수 없는 경우, 원인은 서브넷에 사용 가능한 사설 IPv4가 없기 때문이다. 따라서 서브넷에 IPv4 CIDR을 추가로 매핑해주어야 한다.

## Egress-only Internet Gateway

* NAT Gateway + Internet Gateway 조합과 동일한 역할을 하지만 NAT Gateway는 IPv4만 지원하고, Egress-only Internet Gateway는 IPv6를 지원한다.
* 송신 전용 인터넷 게이트웨이를 생성하면 사설 서브넷의 EC2 인스턴스가 송신 전용 인터넷 게이트웨이를 통해 IPv6로 인터넷에 접근할 수 있다. 하지만 인터넷에서 EC2 인스턴스로는 접근할 수 없다.
* `::/0`로 라우팅 테이블을 작성하면 모든 IPv6에 대해 웹 서버가 인터넷 게이트웨이를 통해 인터넷에 액세스하도록 허용한다.

## AWS 네트워크 비용

* EC2로 들어오는 트래픽은 모두 무료이다.
* 같은 AZ에 위치한 두 EC2 인스턴스 간 트래픽은 사설 IP로 통신할 시 전부 무료이다.
* 같은 리전 내 다른 두 AZ에 있는 EC2 인스턴스 간 트래픽은 공용 IP나 탄력적 IP를 통해 통신할 수 있다. 즉, 트래픽이 AWS 네트워크 밖으로 나갔다 다시 들어오므로 이때 청구되는 비용은 GB당 2센트이다.
* 두 AZ를 연결하는 데 내부 AWS 네트워크를 사용한다면 비용이 반으로 절감된다.
* 트래픽이 다른 리전으로 이동할 때는 GB당 0.02달러가 청구된다.
* 가용성과 비용 사이에서 균형을 유지해야 한다. 다중 AZ, 다중 리전을 구성해 가용성을 확보하면서도 최대한 사설 IP를 활용해야 한다.

> **인스턴스끼리 소통할 때 속도는 높이고 비용은 줄이려면 공용 IP보다는 사설 IP를 사용해야 한다.**

* RDS 데이터베이스의 읽기 전용 복제본을 생성하고 분석을 해야 하는 경우, 가장 저렴한 방식은 같은 AZ에 복제본을 생성하는 것이다. 다른 AZ에 읽기 전용 복제본을 생성한다면 GB당 1센트를 지불해야 한다.
* 외부에서 AWS로 들어오는 트래픽으로 보통 무료이다. 외부로 나가는 트래픽의 크기와 횟수를 최소한으로 하기 위해 최대한 AWS 내부에서 작업을 처리해야 한다.
* Direct Connect를 사용할 경우 Direct Connect 로케이션을 동일한 AWS 리전으로 설정해 비용을 아낄 수 있다.
* Amazon S3에서 컴퓨터로 인터넷을 통해 데이터를 내려받는다면, 송신 트래픽 1GB마다 9센트를 지불하게 된다. S3 전송 가속화를 사용한다면 전송 속도가 50-500% 정도 빨라지지만 GB당 4-8센트가 청구된다.
* S3에서 CloudFront로 발생하는 트래픽은 무료이다. CloudFront에서 인터넷으로 전송할 때는 GB당 8.5센트가 청구된다. 그래도 캐싱 기능도 지원하고 데이터 액세스 지연 시간도 짧아지고 Amazon S3보다는 저렴하다.
* Amazon S3 버킷에 리전 간 복제를 수행한다면 GB당 2센트를 지불해야 한다.
* EC2 인스턴스에서 NAT Gateway와 인터넷 게이트웨이를 지나 인터넷과 직접 연결할 때 NAT Gateway 요금은 시간당 0.045달러이고, NAT Gateway에서 처리되는 데이터 1GB마다 0.045달러가 든다.&#x20;
* VPC 엔드 포인트를 사용 시 NAT Gateway보다 훨씬 저렴하다.

## AWS 네트워크 방화벽

* AWS WAF는 HTTP를 통하여 특정 서비스에 유입되는 악성 요청을 막는 방화벽이다.
* AWS Shield와 Shield Advanced를 통해 DDoS를 막을 수 있다.
* AWS Firewall Manager를 통해  여러 계정에 적용할 수 있는 WAF, Shield 등에 대한 규칙을 관리할 수 있다.
* AWS Network Firewall을 사용하면 전체 VPC를 방화벽으로 보호할 수 있다.
* 모든 방향에서 들어오는 모든 트래픽을 검사하여 3\~7계층을 보호한다.
* VPC 간 트래픽 인터넷 아웃바운드, 인터넷 인바운드, Direct Connect나 Site to Site VPN 연결까지 검사한다.
* Network Firewall은 내부적으로 AWS Gateway Load Balancer를 사용한다. AWS에서 자체 어플라이언스를 통해 트래픽을 관리하므로 Network Firewall에 자체 규칙을 갖추고 있다. 규칙들은 AWS Firewall Manager에 의해 중앙 집중식으로 관리되며 여러 계정과 VPC에 적용된다.
* Network Firewall에서는 네트워크 트래픽을 세부적으로 관리할 수 있다.
  * VPC 수준에서 수천 개의 규칙을 지원하고, 수만 IP를 IP와 포트별로 필터링할 수 있다.
  * 프로토콜, 도메인 별로도 필터링할 수 있다.
  * regex 등을 이용해서 일반 패턴 매칭도 가능하고 트래픽 허용, 차단, 알림 등을 설정해서 설정한 규칙에 맞는 트래픽을 필터링할 수도 있다.
  * **활성 플로우 검사**를 통해 침입 방지 능력을 갖출 수 있다.
* Amazon S3와 CloudWatch Logs 및 Kinesis Data Firehose로 전송해 분석할 수 있다.
