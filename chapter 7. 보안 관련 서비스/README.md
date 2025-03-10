# 보안 관련 서비스

## 정보보안 3대 요소
1.  <mark style="background: #DCFF8C;">기밀성</mark> - 정보가 유출되지 않도록 관리
2. <mark style="background: #DCFF8C;">무결성</mark> - 정보가 손상되거나 손실되지 않고 최신의 상태인 것
3. <mark style="background: #DCFF8C;">가용성</mark> - 정보를 언제나 사용할 수 있게 하는 것

> root 사용자 계정은 처음 설정할 떄만 사용하고 2FA 를 설정해 보안을 강화해야한다
> 2FA란 이중 인증을 말하면 계정 보안을 강화하는 방법이다.

## <mark style="background: #DCFF8C;">IAM</mark>
- AWS 를 사용하는 계정의 권한을 관리하는 서비스
- 각종 서비스를 사용할 수 있는 사용자(IAM 사용자)와 그룹 (IAM 그룹)를 만들고 해당 사용자에게 어떤 기능을 사용할 수 있는지)와 같은 권한 설정 가능
	- 권한도 정책(IAM역할)으로 따로 생성해 각 사용자와 그룹에 부여 가능하며 IAM 역할은 재사용이 된다.
- IAM 사용자 생성시 액세스 키 생성 가능
	- <mark style="background: #DCFF8C;">액세스 키</mark> - CLI로 AWS 자원을 조작하거나 프로그램을 통한 제어를 할 때 사용하는 인증 정보
	- AWS 계정 과 동일 권한이므로 액세스 키 유출을 주의해야한다.
 - <mark style="background: #DCFF8C;">IAM 역할</mark> - 특정 서비스나 사용자가 AWS 리소스에 접근할 수 있도록 임시 권한을 부여하는 것
	 - 해당 서비스를 사용할 수 있는 사용자를 지정하는 것 
	 - <mark style="background: #DCFF8C;">역할을 받는 주체는 IAM 사용자일 수도 있고, AWS 서비스(Lambda, EC2 등)일 수도 있음</mark>
	 - 특정 주체(사용자, 서비스, 계정 등)가 "필요할 때" 해당 역할을 "사용하도록 허용"하는 개념
 - IAM 사용자가 많아지면 IAM 역할만 생성하고IAM 사용자가 필요할 때만 역할을 부여해 적절한 권한으로 작업할 수 있게 설정 가능
 -  ![|537](https://i.imgur.com/6iTW2Vh.png)


## AWS CloudTrail
- AWS 계정을 만들 때부터 자동으로 모든 작업을 기록하는 서비스
- 관리 콘솔이나 프로그램에서의 작업, AWS 서비에서 수행한 모든 작업을 기록
- 기본적으로 90일 저장
	- 설정에따라 S3와 같은 별도 스토리지에 저장 가능
- **SSH, 원격 데스크톤으로 직업 EC2에 접속해 작업을 수행하면 CloudTrail에 기록되지 않는다**
	- **기본적으로 EC2에서 실행한 명령어는 ~/.bash_history 에 저장되므로 여기서 체크 가능**
	- CloudWatch Agent 를 사용해 EC2에서 실행된 명령어 로그를 실시간으로 CloudWatch Logs에 전송 가능
		- <mark style="background: #DCFF8C;">CloudWatch</mark> - AWS 리소스와 애플리케이션 성능 모니터링
- CloudWatch Logs 를 통해 로그 내용을 확인하고 특정 문자열을 찾을 때 이벤트를 발생 시키는 기능이 있어 알림을 보낼 수 있다.
- AWS **CloudWatch**와 **CloudTrail**은 기본적인 기능은 무료로 제공되지만, 사용량에 따라 비용이 발생한다.


## AWS Config
- EC2 인스턴스나 보안 그룹과 같은 AWS 자원에 대한 구성 정보와 변경 이력을 남기는 서비스
- AWS 자원의 설정 내용 기반으로 이력을 저장
- AWS Config 에서 추척하는 설정 변경 예시
- 관리 콘솔에서 쿼리를 통해 자원수 파악도 가능
	- AWS Config 관리 콘솔에서 왼쪽 메뉴의 Advanced queries 에서 쿼리 생성
	- 예시
```
SELECT COUNT(*) FROM aws.ec2.instance
```

- 추적하는 변경사항 정리

| <center><font color="#fac08f">리소스</font></center> | <center><font color="#fac08f">추적하는 변경 사항</font></center> |
| ------------------------------------------------- | --------------------------------------------------------- |
| EC2 인스턴스                                          | 시작, 중지, 종료, 인스턴스 유형 변경, 보안 그룹 변경                          |
| S3                                                | 공개 액세스 설정 변경, 버킷 정책 변경, 버전 관리 활성화/비활성화                    |
| IAM 사용자 & 역할                                      | 정책 변경, 새로운 사용자 생성, 역할 권한 수정                               |
| VPC (네트워크 설정)                                     | 서브넷 추가, 보안 그룹 규칙 변경, 라우트 테이블 변경                           |
| RDS (DB 설정)                                       | 백업 설정 변경, 보안 그룹 변경, 스토리지 크기 조정                            |
- AWS Config 규칙
	- AWS의 각 설정이 규칙을 준수하는지 확인 가능
		- 예) EBS가 암호화가 되어있는가
		- 예) ClroudTrail이 사용 설정되어 있는가
		- 예) S3 버킷이 공개적으로 읽고 쓸 수 없는가
		- 예) 보안 그룹에 SSH 포트(22번)가 공개적으로 게시되지 않았는가
	- AWS 에서 제공하는 규칙인 **AWS 관리형 규칙** 이 있고, 사용자가 자신만의 규칙을 만드는 **사용자 지정 규칙** 이 있다.

## <mark style="background: #DCFF8C;">Amazon EventBridge, AWS SNS 를 통해 이슈 발생시 메일 전송</mark>
(위 방법 이외에도 메일 보내는 방법도 있다 - AWS SES)
- 해당 방법은 AWS 서비스에서 이벤트 발생시 자동으로 이메일을 보내는 구조
- Amazon EventBridge 에서 이벤트 규칙 생성
    - 특정 AWS 서비스에서 이벤트(예: EC2 상태 변경, S3 파일 업로드 등)가 발생하면 감지
- 이벤트가 발생하면 AWS SNS로 전송
- <mark style="background: #DCFF8C;">AWS SNS가 이메일로 알림을 발송</mark>

><번외 지식>
> <mark style="background: #DCFF8C;">Amazon SQS (Simple Queue Service)</mark>
> 생성자(Producer)가 보낸 메시지가 하나의 큐에 쌓이고 여러 개의 소비자(Consumer)가 순차적으로 병렬로 가져가서 처리하는 비동기 메시지 큐 서비스
> 
> <mark style="background: #DCFF8C;">AWS SNS</mark>
> Email, SMS, HTTP, Lambda 등으로 메시지를 전송하는 서비스, 알림 또는 메시지 브로커 역할을 수행
> -> CloudWatch, EventBridge, Firebase(앱 푸시) 등 과 연동해 사용가능
> 
> <mark style="background: #DCFF8C;">AWS EventBridge</mark>
> AWS 서비스 간 이벤트 트리거를 자동으로 연결해주는 서비스
> (SaaS App, 사용자 정의 App 간 이벤트를 연결하고 처리하는 이벤트 버스 서비스)


## 그외의 서비스
- AWS GuardDuty
	- AWS 계정 내의 모든 활동을 감시하는 위협 탐지 서비스
	- AWS 계정의 보안 상태를 즉시 분석할 수 있다.
	- CloudTaril, VPC 흐름 로그 및 DNS Logs 의 세가지 정보를 기반으로 탐지를 수행하며, EC2 에서 실행되는 응용 프로그램이나 Lambda 함수에서 발생하는 위협 이벤트는 탐지할 수 없다.
	- 예) 루트 사용자 사용(사용 안하는게 맞음), IAM 액세스 키가 대량으로 생성됨, EC2가 DDoS 공격을 위한 좀비 PC 가 됐을 가능성

## WAF (웹 방화벽)
![|700x225](https://i.imgur.com/39elHOz.png)
- AWS WAF 서비스 제공중, 간단히 적용 가능
- 서비스 혹은 ALB에 적용 가능
- AWS 에서 제공하는 기본 보안규칙도 있고 사용자 정의 규칙도 추가가 가능하다.

><번외 지식>
><mark style="background: #DCFF8C;">SQL 인젝션</mark> 공격이란?
>- WEB APP의 DB 쿼리를 조작해 비정상적인 데이터 접근, 변경, 삭제 등을 수행하는 해킹 기법
>- 일반적으로 사용자 입력이 포함된 SQL 쿼리가 검증 없이 실행될 때 발생
>	- 사용자 입력값을 직접 쿼리에 포함하면 공격자가 **SQL 코드를 삽입할 수 있음**.
>	- 예) SELECT * FROM users WHERE username = 'admin' --' AND password = '1234';
>	- 위 예시와 같은 경우 --는 SQL 에서 주석 처리를 의미하므로 비밀번호 체크 부분이 무시된다.

