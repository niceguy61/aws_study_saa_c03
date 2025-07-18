# AWS SAA-C03 용어집

이 용어집은 AWS Solutions Architect Associate(SAA-C03) 학습 과정에서 나오는 주요 용어와 개념을 정리한 것입니다. 주차별로 학습하는 내용에 따라 용어가 추가됩니다.

## 1주차: 클라우드 기초 및 보안 아키텍처

| 용어 | 설명 | 관련 서비스 |
|------|------|------------|
| 클라우드 컴퓨팅 | 인터넷을 통해 IT 리소스를 온디맨드로 제공하는 서비스 | AWS 전체 |
| CAPEX | 자본 지출(Capital Expenditure). 서버 구매와 같은 초기 투자 비용 | - |
| OPEX | 운영 지출(Operational Expenditure). 사용한 만큼 지불하는 클라우드 비용 모델 | - |
| 확장성 | 필요에 따라 리소스를 확장하거나 축소할 수 있는 능력 | Auto Scaling |
| 탄력성 | 수요 변화에 자동으로 대응하여 리소스를 조정하는 능력 | EC2, Lambda |
| IaaS | Infrastructure as a Service. 가상 서버, 스토리지 등 인프라 제공 | EC2, S3 |
| PaaS | Platform as a Service. 개발 플랫폼과 런타임 환경 제공 | Elastic Beanstalk |
| SaaS | Software as a Service. 완성된 소프트웨어 서비스 제공 | WorkMail |
| 리전 | AWS 데이터 센터가 위치한 지리적 영역 | 글로벌 인프라 |
| 가용 영역 | 각 리전 내의 물리적으로 분리된 데이터 센터 | 글로벌 인프라 |
| 엣지 로케이션 | CloudFront 콘텐츠 캐싱을 위한 전 세계 지점 | CloudFront |
| 공동 책임 모델 | AWS와 고객 간의 보안 책임 분담 구조 | 보안 |
| IAM | Identity and Access Management. AWS 리소스 접근 제어 서비스 | IAM |
| 루트 사용자 | AWS 계정 생성 시 만들어지는 모든 권한을 가진 사용자 | IAM |
| IAM 사용자 | AWS 서비스와 리소스에 접근하기 위해 생성하는 개체 | IAM |
| IAM 그룹 | IAM 사용자들의 집합으로 권한 관리 효율화 | IAM |
| IAM 역할 | 임시로 권한을 빌려주는 시스템 | IAM |
| IAM 정책 | 권한을 정의하는 JSON 문서 | IAM |
| MFA | Multi-Factor Authentication. 다중 인증 | IAM |
| VPC | Virtual Private Cloud. AWS 클라우드 내 논리적으로 격리된 네트워크 | VPC |
| 서브넷 | VPC 내의 IP 주소 범위 | VPC |
| CIDR | Classless Inter-Domain Routing. IP 주소 할당 방법 | VPC |
| 라우팅 테이블 | 네트워크 트래픽 경로를 결정하는 규칙 집합 | VPC |
| 인터넷 게이트웨이 | VPC와 인터넷 간의 통신을 가능하게 하는 구성 요소 | VPC |
| NAT 게이트웨이 | 프라이빗 서브넷의 인스턴스가 인터넷에 접근할 수 있게 해주는 서비스 | VPC |
| 보안 그룹 | 인스턴스 레벨의 방화벽 | VPC |
| 네트워크 ACL | 서브넷 레벨의 방화벽 | VPC |

## 2주차: 복원력 + 고성능 아키텍처

| 용어 | 설명 | 관련 서비스 |
|------|------|------------|
| EC2 | Elastic Compute Cloud. 가상 서버 서비스 | EC2 |
| AMI | Amazon Machine Image. EC2 인스턴스 생성을 위한 템플릿 | EC2 |
| EBS | Elastic Block Store. EC2용 블록 스토리지 | EC2 |
| 인스턴스 유형 | EC2 인스턴스의 CPU, 메모리, 스토리지, 네트워킹 용량 조합 | EC2 |
| Auto Scaling | 수요에 따라 EC2 인스턴스 수를 자동으로 조정 | EC2 |
| 로드 밸런서 | 여러 컴퓨팅 리소스에 네트워크 트래픽 분산 | ELB |
| Lambda | 서버리스 컴퓨팅 서비스 | Lambda |
| S3 | Simple Storage Service. 객체 스토리지 서비스 | S3 |
| S3 스토리지 클래스 | 데이터 접근 빈도와 중요도에 따른 스토리지 옵션 | S3 |
| RDS | Relational Database Service. 관계형 데이터베이스 서비스 | RDS |
| DynamoDB | 완전 관리형 NoSQL 데이터베이스 서비스 | DynamoDB |
| ElastiCache | 인메모리 캐싱 서비스 | ElastiCache |
| CloudFront | 콘텐츠 전송 네트워크(CDN) 서비스 | CloudFront |
| Route 53 | DNS 웹 서비스 | Route 53 |

## 3주차: 비용 최적화 + 종합 정리

| 용어 | 설명 | 관련 서비스 |
|------|------|------------|
| 온디맨드 인스턴스 | 필요할 때 인스턴스를 시작하고 사용한 시간만큼 지불 | EC2 |
| 예약 인스턴스 | 1년 또는 3년 약정으로 할인된 가격에 인스턴스 예약 | EC2 |
| 스팟 인스턴스 | AWS의 여유 컴퓨팅 용량을 경매 방식으로 이용 | EC2 |
| 비용 최적화 | 불필요한 비용을 줄이면서 비즈니스 결과 개선 | AWS 비용 관리 |
| Cost Explorer | AWS 비용 및 사용량 분석 도구 | AWS 비용 관리 |
| AWS Budgets | 비용 및 사용량 예산 설정 및 추적 | AWS 비용 관리 |
| Well-Architected Framework | AWS 클라우드에서 안전하고 효율적인 시스템 설계 원칙 | - |

## 4주차: 집중 문제풀이 및 시험 준비

| 용어 | 설명 | 관련 서비스 |
|------|------|------------|
| 보안 아키텍처 | AWS 리소스에 대한 보안 액세스 설계 | IAM, KMS, WAF |
| 복원력 아키텍처 | 장애 발생 시에도 서비스를 지속할 수 있는 아키텍처 | Auto Scaling, Multi-AZ |
| 고성능 아키텍처 | 워크로드 요구사항에 맞게 컴퓨팅 리소스를 효율적으로 사용 | EC2, RDS, ElastiCache |
| 비용 최적화 아키텍처 | 불필요한 비용을 피하면서 비즈니스 요구사항 충족 | 예약 인스턴스, S3 수명주기 |