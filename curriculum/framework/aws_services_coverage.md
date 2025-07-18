# AWS SAA 교육과정 서비스 커버리지

## 개요

본 문서는 AWS Certified Solutions Architect - Associate (SAA-C03) 5주 교육과정에서 다루는 AWS 서비스 목록과 각 서비스의 학습 깊이를 정의합니다. SAA 시험 출제 범위를 기준으로 서비스의 중요도를 구분하여 교육 과정에 반영합니다.

## 서비스 중요도 분류

### 핵심 서비스 (Core)
- SAA 시험에서 높은 비중으로 출제되는 서비스
- 심층적인 이해와 실습이 필요한 서비스
- 교육 과정에서 상세하게 다루고 실습 필수

### 주요 서비스 (Major)
- SAA 시험에서 중간 비중으로 출제되는 서비스
- 기본 개념과 주요 기능에 대한 이해가 필요한 서비스
- 교육 과정에서 주요 기능 위주로 다루고 일부 실습 진행

### 기본 서비스 (Basic)
- SAA 시험에서 낮은 비중으로 출제되는 서비스
- 개념적 이해와 사용 사례 인지가 필요한 서비스
- 교육 과정에서 개념과 사용 사례 위주로 소개

## 주차별 서비스 커버리지

### 1주차: 클라우드 기초 및 보안 아키텍처 설계

#### 핵심 서비스
- **IAM (Identity and Access Management)**
  - 사용자, 그룹, 역할, 정책
  - 최소 권한의 원칙
  - 다중 인증(MFA)
  - 자격 증명 페더레이션
  - 교차 계정 액세스

- **VPC (Virtual Private Cloud)**
  - 서브넷 설계 (퍼블릭/프라이빗)
  - 라우팅 테이블
  - 인터넷 게이트웨이
  - NAT 게이트웨이/인스턴스
  - 보안 그룹 및 NACL
  - VPC 엔드포인트

- **KMS (Key Management Service)**
  - 암호화 키 생성 및 관리
  - 키 정책 및 권한
  - 서비스 통합

#### 주요 서비스
- **AWS Organizations**
  - 다중 계정 관리
  - 서비스 제어 정책(SCP)
  - 조직 단위(OU)

- **AWS Config**
  - 리소스 구성 모니터링
  - 규정 준수 평가
  - 구성 변경 추적

- **CloudTrail**
  - API 활동 로깅
  - 이벤트 기록
  - 감사 및 규정 준수

- **Secrets Manager**
  - 비밀 저장 및 관리
  - 자동 교체
  - 서비스 통합

#### 기본 서비스
- **AWS Shield**
  - DDoS 보호
  - Shield Standard vs Advanced

- **AWS WAF (Web Application Firewall)**
  - 웹 애플리케이션 보호
  - 규칙 및 규칙 그룹

- **GuardDuty**
  - 위협 탐지
  - 보안 모니터링

- **Security Hub**
  - 보안 상태 통합 보기
  - 규정 준수 확인

### 2주차: 복원력을 갖춘 아키텍처 설계

#### 핵심 서비스
- **EC2 (Elastic Compute Cloud)**
  - 인스턴스 유형 및 패밀리
  - AMI 및 사용자 데이터
  - 인스턴스 스토어 vs EBS
  - 배치 그룹
  - 인스턴스 구매 옵션

- **Auto Scaling**
  - Auto Scaling 그룹
  - 시작 템플릿/구성
  - 스케일링 정책
  - 예측 스케일링

- **ELB (Elastic Load Balancing)**
  - ALB, NLB, CLB 비교
  - 대상 그룹 및 상태 확인
  - 리스너 및 규칙
  - SSL/TLS 종료

- **Lambda**
  - 서버리스 컴퓨팅
  - 트리거 및 이벤트 소스
  - 배포 패키지 및 계층
  - 동시성 및 제한 시간

#### 주요 서비스
- **ECS (Elastic Container Service)**
  - 컨테이너 오케스트레이션
  - 작업 정의 및 서비스
  - Fargate vs EC2 시작 유형

- **EKS (Elastic Kubernetes Service)**
  - 관리형 Kubernetes
  - 클러스터 및 노드 그룹
  - Kubernetes 통합

- **API Gateway**
  - REST 및 WebSocket API
  - 스테이지 및 배포
  - 권한 부여 및 스로틀링

- **Step Functions**
  - 워크플로 오케스트레이션
  - 상태 머신
  - 통합 패턴

#### 기본 서비스
- **Elastic Beanstalk**
  - PaaS 솔루션
  - 환경 및 애플리케이션 버전

- **App Runner**
  - 컨테이너화된 웹 애플리케이션 실행

- **AWS Batch**
  - 배치 처리 작업

- **EventBridge**
  - 이벤트 버스 및 규칙
  - 이벤트 패턴 매칭

### 3주차: 고성능 아키텍처 설계

#### 핵심 서비스
- **S3 (Simple Storage Service)**
  - 스토리지 클래스
  - 버킷 정책 및 ACL
  - 수명 주기 관리
  - 버전 관리
  - 암호화 옵션
  - 정적 웹 사이트 호스팅

- **EBS (Elastic Block Store)**
  - 볼륨 유형 및 성능 특성
  - 스냅샷 및 AMI
  - 암호화
  - IOPS 및 처리량 최적화

- **RDS (Relational Database Service)**
  - 지원되는 데이터베이스 엔진
  - 다중 AZ 배포
  - 읽기 전용 복제본
  - 백업 및 복원
  - 파라미터 그룹

- **DynamoDB**
  - NoSQL 데이터 모델
  - 파티션 키 및 정렬 키
  - 읽기/쓰기 용량 모드
  - 글로벌 테이블
  - DAX(DynamoDB Accelerator)

#### 주요 서비스
- **EFS (Elastic File System)**
  - 공유 파일 스토리지
  - 성능 모드 및 처리량 모드
  - 수명 주기 관리

- **Aurora**
  - 고성능 관계형 데이터베이스
  - 글로벌 데이터베이스
  - 서버리스
  - 병렬 쿼리

- **ElastiCache**
  - Redis 및 Memcached
  - 캐싱 전략
  - 클러스터 모드

- **CloudFront**
  - CDN 서비스
  - 오리진 및 동작
  - 캐싱 동작
  - Lambda@Edge

#### 기본 서비스
- **FSx**
  - Windows File Server
  - Lustre
  - NetApp ONTAP

- **Neptune**
  - 그래프 데이터베이스

- **DocumentDB**
  - MongoDB 호환 데이터베이스

- **Keyspaces**
  - Apache Cassandra 호환 데이터베이스

- **Global Accelerator**
  - 글로벌 애플리케이션 성능 개선

- **Kinesis**
  - 실시간 데이터 스트리밍
  - Data Streams, Firehose, Analytics

### 4주차: 비용에 최적화된 아키텍처 설계

#### 핵심 서비스
- **Cost Explorer**
  - 비용 분석 및 시각화
  - 비용 할당 태그
  - 예약 인스턴스 권장 사항

- **AWS Budgets**
  - 예산 설정 및 알림
  - 비용, 사용량, RI 커버리지 예산

- **EC2 구매 옵션**
  - 온디맨드 인스턴스
  - 예약 인스턴스
  - Savings Plans
  - 스팟 인스턴스

- **S3 스토리지 클래스**
  - Standard, Intelligent-Tiering
  - Standard-IA, One Zone-IA
  - Glacier, Glacier Deep Archive
  - 수명 주기 정책

#### 주요 서비스
- **Trusted Advisor**
  - 비용 최적화 권장 사항
  - 성능, 보안, 내결함성 검사

- **Compute Optimizer**
  - EC2 인스턴스 크기 최적화
  - 권장 사항 및 절감 기회

- **AWS Cost and Usage Report**
  - 상세 비용 및 사용량 데이터
  - Athena, QuickSight와 통합

- **AWS Organizations 통합 결제**
  - 다중 계정 비용 관리
  - 통합 결제 할인

#### 기본 서비스
- **AWS Pricing Calculator**
  - AWS 서비스 비용 추정

- **AWS Cost Anomaly Detection**
  - 비정상적인 지출 감지

- **AWS Purchase Order Management**
  - 구매 주문 관리

### 5주차: 통합 실습 및 시험 준비

#### 핵심 서비스 통합
- **Well-Architected Framework**
  - 5가지 기둥 (운영 우수성, 보안, 안정성, 성능 효율성, 비용 최적화)
  - 아키텍처 평가 및 개선

- **CloudFormation**
  - 인프라 코드화(IaC)
  - 템플릿 및 스택
  - 리소스 및 출력
  - 중첩 스택

- **Route 53**
  - DNS 서비스
  - 라우팅 정책
  - 상태 확인
  - 도메인 등록

#### 주요 서비스 통합
- **Systems Manager**
  - 리소스 관리
  - 자동화
  - 패치 관리

- **CloudWatch**
  - 지표 및 경보
  - 로그
  - 이벤트

- **Direct Connect**
  - 전용 네트워크 연결
  - 가상 인터페이스

- **Storage Gateway**
  - 하이브리드 스토리지 통합
  - 파일, 볼륨, 테이프 게이트웨이

#### 기본 서비스 통합
- **Control Tower**
  - 다중 계정 거버넌스

- **Service Catalog**
  - 승인된 서비스 포트폴리오

- **License Manager**
  - 소프트웨어 라이선스 관리

## 서비스별 학습 깊이

### 핵심 서비스 학습 깊이

| 서비스 | 개념 이해 | 구성 방법 | 모범 사례 | 실습 수준 | 시험 중요도 |
|-------|---------|---------|---------|---------|----------|
| IAM | 심층 | 상세 | 상세 | 고급 | 매우 높음 |
| VPC | 심층 | 상세 | 상세 | 고급 | 매우 높음 |
| EC2 | 심층 | 상세 | 상세 | 고급 | 매우 높음 |
| S3 | 심층 | 상세 | 상세 | 고급 | 매우 높음 |
| RDS | 심층 | 상세 | 상세 | 고급 | 매우 높음 |
| DynamoDB | 심층 | 상세 | 상세 | 중급 | 높음 |
| Auto Scaling | 심층 | 상세 | 상세 | 고급 | 높음 |
| ELB | 심층 | 상세 | 상세 | 고급 | 높음 |
| Lambda | 심층 | 상세 | 상세 | 중급 | 높음 |
| CloudFormation | 심층 | 상세 | 상세 | 중급 | 높음 |
| Route 53 | 심층 | 상세 | 상세 | 중급 | 높음 |
| KMS | 심층 | 상세 | 상세 | 중급 | 높음 |
| EBS | 심층 | 상세 | 상세 | 중급 | 높음 |

### 주요 서비스 학습 깊이

| 서비스 | 개념 이해 | 구성 방법 | 모범 사례 | 실습 수준 | 시험 중요도 |
|-------|---------|---------|---------|---------|----------|
| AWS Organizations | 중급 | 기본 | 중급 | 기본 | 중간 |
| CloudTrail | 중급 | 기본 | 중급 | 기본 | 중간 |
| AWS Config | 중급 | 기본 | 중급 | 기본 | 중간 |
| ECS | 중급 | 기본 | 중급 | 기본 | 중간 |
| EKS | 중급 | 기본 | 중급 | 기본 | 중간 |
| API Gateway | 중급 | 기본 | 중급 | 기본 | 중간 |
| Aurora | 중급 | 기본 | 중급 | 기본 | 중간 |
| ElastiCache | 중급 | 기본 | 중급 | 기본 | 중간 |
| CloudFront | 중급 | 기본 | 중급 | 기본 | 중간 |
| EFS | 중급 | 기본 | 중급 | 기본 | 중간 |
| Cost Explorer | 중급 | 기본 | 중급 | 기본 | 중간 |
| AWS Budgets | 중급 | 기본 | 중급 | 기본 | 중간 |
| Secrets Manager | 중급 | 기본 | 중급 | 기본 | 중간 |
| Systems Manager | 중급 | 기본 | 중급 | 기본 | 중간 |
| CloudWatch | 중급 | 기본 | 중급 | 기본 | 중간 |

### 기본 서비스 학습 깊이

| 서비스 | 개념 이해 | 구성 방법 | 모범 사례 | 실습 수준 | 시험 중요도 |
|-------|---------|---------|---------|---------|----------|
| AWS Shield | 기본 | 개요 | 기본 | 없음 | 낮음 |
| AWS WAF | 기본 | 개요 | 기본 | 없음 | 낮음 |
| GuardDuty | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Security Hub | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Elastic Beanstalk | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Step Functions | 기본 | 개요 | 기본 | 없음 | 낮음 |
| EventBridge | 기본 | 개요 | 기본 | 없음 | 낮음 |
| FSx | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Neptune | 기본 | 개요 | 기본 | 없음 | 낮음 |
| DocumentDB | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Keyspaces | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Global Accelerator | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Kinesis | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Trusted Advisor | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Compute Optimizer | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Direct Connect | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Storage Gateway | 기본 | 개요 | 기본 | 없음 | 낮음 |
| Control Tower | 기본 | 개요 | 기본 | 없음 | 낮음 |

## 서비스 학습 우선순위

### 최우선 학습 서비스 (Must Learn)
1. IAM (Identity and Access Management)
2. VPC (Virtual Private Cloud)
3. EC2 (Elastic Compute Cloud)
4. S3 (Simple Storage Service)
5. RDS (Relational Database Service)
6. DynamoDB
7. Auto Scaling
8. ELB (Elastic Load Balancing)
9. Lambda
10. CloudFormation
11. Route 53
12. KMS (Key Management Service)
13. EBS (Elastic Block Store)

### 중요 학습 서비스 (Should Learn)
1. AWS Organizations
2. CloudTrail
3. AWS Config
4. ECS (Elastic Container Service)
5. API Gateway
6. Aurora
7. ElastiCache
8. CloudFront
9. EFS (Elastic File System)
10. Cost Explorer
11. AWS Budgets
12. Secrets Manager
13. Systems Manager
14. CloudWatch

### 기본 학습 서비스 (Nice to Learn)
1. AWS Shield
2. AWS WAF
3. GuardDuty
4. Security Hub
5. Elastic Beanstalk
6. Step Functions
7. EventBridge
8. FSx
9. Neptune
10. DocumentDB
11. Global Accelerator
12. Kinesis
13. Trusted Advisor
14. Direct Connect
15. Storage Gateway

## 서비스 학습 로드맵

### 1주차: 클라우드 기초 및 보안 아키텍처 설계
- **월요일**: AWS 개요, IAM 기초
- **화요일**: IAM 심화, AWS Organizations
- **수요일**: VPC 기초, 서브넷 설계
- **목요일**: VPC 심화, 보안 그룹, NACL
- **금요일**: KMS, CloudTrail, AWS Config

### 2주차: 복원력을 갖춘 아키텍처 설계
- **월요일**: EC2 기초, AMI, 인스턴스 유형
- **화요일**: Auto Scaling, ELB
- **수요일**: Lambda, API Gateway
- **목요일**: ECS, EKS
- **금요일**: 재해 복구 전략, Route 53

### 3주차: 고성능 아키텍처 설계
- **월요일**: S3, EBS, EFS
- **화요일**: RDS, Aurora
- **수요일**: DynamoDB, ElastiCache
- **목요일**: CloudFront, Global Accelerator
- **금요일**: Kinesis, 데이터 처리 서비스

### 4주차: 비용에 최적화된 아키텍처 설계
- **월요일**: Cost Explorer, AWS Budgets
- **화요일**: S3 스토리지 클래스, 수명 주기 정책
- **수요일**: EC2 구매 옵션, Savings Plans
- **목요일**: RDS 및 DynamoDB 비용 최적화
- **금요일**: Well-Architected Framework

### 5주차: 통합 실습 및 시험 준비
- **월요일**: 종합 아키텍처 설계 실습 (1)
- **화요일**: 종합 아키텍처 설계 실습 (2)
- **수요일**: 모의고사 (1) 및 해설
- **목요일**: 취약 영역 보완 및 심화 학습
- **금요일**: 모의고사 (2) 및 최종 시험 전략