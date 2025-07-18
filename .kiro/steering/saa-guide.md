---
inclusion: always
---

# AWS Certified Solutions Architect - Associate (SAA-C03) 5주 교육과정

## 교육 개요

### 대상 응시자
- AWS 서비스를 사용하는 클라우드 솔루션 설계 1년 이상의 실무 경험자
- AWS Well-Architected Framework 기반 솔루션 설계 역량 보유 목표

### 핵심 학습 목표
- 현재 및 향후 비즈니스 요구사항을 충족하는 AWS 서비스 통합 솔루션 설계
- 안전하고 복원력이 뛰어나며 성능이 우수하고 비용에 최적화된 아키텍처 설계
- 기존 솔루션 검토 및 개선 사항 결정

## 시험 정보

### 출제 형식
- **선다형**: 정답 1개, 오답 3개
- **복수 응답형**: 5개 이상 선택지 중 2개 이상 정답
- **총 문항**: 65문항 (채점 대상 50문항 + 채점 비대상 15문항)
- **합격 기준**: 720점/1000점
- **시험 시간**: 130분

### 도메인별 가중치
1. **보안 아키텍처 설계**: 30%
2. **복원력을 갖춘 아키텍처 설계**: 26%
3. **고성능 아키텍처 설계**: 24%
4. **비용에 최적화된 아키텍처 설계**: 20%

## 5주 교육과정 구성

### 1주차: 보안 아키텍처 설계 (30%)

#### 1.1 AWS 리소스에 대한 보안 액세스 설계
**핵심 개념:**
- 여러 계정에 대한 액세스 제어 및 관리
- AWS 페더레이션 액세스 및 자격 증명 서비스
- AWS 글로벌 인프라 (가용 영역, AWS 리전)
- AWS 보안 모범 실무 (최소 권한의 원칙)
- AWS 공동 책임 모델

**실습 기술:**
- IAM 사용자 및 루트 사용자 보안 모범 실무 적용 (MFA)
- IAM 사용자, 그룹, 역할 및 정책을 포함하는 유연한 권한 부여 모델 설계
- 역할 기반 액세스 제어 전략 설계 (AWS STS, 역할 전환, 교차 계정 액세스)
- 여러 AWS 계정에 대한 보안 전략 설계 (AWS Control Tower, SCP)
- AWS 서비스에 적합한 리소스 정책 사용
- 디렉터리 서비스와 IAM 역할 연동

#### 1.2 안전한 워크로드 및 애플리케이션 설계
**핵심 개념:**
- 애플리케이션 구성 및 보안 인증 정보의 보안
- AWS 서비스 엔드포인트
- 포트, 프로토콜 및 네트워크 트래픽 제어
- 안전한 애플리케이션 액세스
- 보안 서비스와 적합한 사용 사례
- AWS 외부의 위협 벡터 (DDoS, SQL 명령어 삽입)

**실습 기술:**
- 보안 구성 요소로 VPC 아키텍처 설계
- 네트워크 분할 전략 결정
- 보안 애플리케이션에 AWS 서비스 통합
- AWS 클라우드와의 외부 네트워크 연결 보호

#### 1.3 적합한 데이터 보안 제어 결정
**핵심 개념:**
- 데이터 액세스 및 거버넌스
- 데이터 복구, 보존 및 분류
- 암호화 및 적합한 키 관리

**실습 기술:**
- 규정 준수 요구 사항을 충족하도록 AWS 기술 조율
- 저장 데이터 암호화 (AWS KMS)
- 전송 중 데이터 암호화 (TLS, ACM)
- 암호화 키 액세스 정책 구현
- 데이터 백업 및 복제 구현

### 2주차: 복원력을 갖춘 아키텍처 설계 (26%)

#### 2.1 확장 가능하고 느슨하게 결합된 아키텍처 설계
**핵심 개념:**
- API 생성 및 관리
- AWS Managed Services와 적합한 사용 사례
- 캐싱 전략
- 마이크로서비스의 설계 원칙
- 이벤트 기반 아키텍처
- 수평적/수직적 크기 조정
- 엣지 액셀러레이터 사용법
- 애플리케이션 컨테이너 마이그레이션
- 로드 밸런싱 개념
- 멀티 티어 아키텍처
- 대기열 및 메시징 개념
- 서버리스 기술 및 패턴
- 스토리지 유형 특성
- 컨테이너 오케스트레이션
- 읽기 전용 복제본 사용 시기
- 워크플로 오케스트레이션

**실습 기술:**
- 요구 사항에 따른 아키텍처 설계
- 구성 요소 크기 조정 전략 결정
- 느슨한 결합을 위한 AWS 서비스 결정
- 컨테이너 및 서버리스 기술 사용 시기 결정
- 적합한 컴퓨팅, 스토리지, 네트워킹, 데이터베이스 기술 권장

#### 2.2 고가용성 및 내결함성 아키텍처 설계
**핵심 개념:**
- AWS 글로벌 인프라
- AWS Managed Services와 적합한 사용 사례
- 기본 네트워킹 개념
- 재해 복구(DR) 전략
- 분산 설계 패턴
- 장애 조치 전략
- 변경 불가능한 인프라
- 서비스 할당량 및 제한
- 스토리지 옵션 및 특성
- 워크로드 가시성

**실습 기술:**
- 인프라 무결성을 위한 자동화 전략 결정
- 고가용성/내결함성 아키텍처를 위한 AWS 서비스 결정
- 고가용성 솔루션을 위한 지표 파악
- 단일 장애 지점 완화 설계
- 데이터 내구성과 가용성 보장 전략
- 적합한 DR 전략 선택

### 3주차: 고성능 아키텍처 설계 (24%)

#### 3.1 고성능 확장 가능한 스토리지 솔루션
**핵심 개념:**
- 비즈니스 요구 사항을 충족하는 하이브리드 스토리지 솔루션
- 스토리지 서비스와 적합한 사용 사례
- 스토리지 유형과 연관된 특성

**실습 기술:**
- 성능 요구 사항을 충족하는 스토리지 서비스 및 구성 결정
- 향후 요구 사항을 수용하도록 크기 조정 가능한 스토리지 서비스 결정

#### 3.2 고성능 탄력적인 컴퓨팅 솔루션 설계
**핵심 개념:**
- AWS 컴퓨팅 서비스와 적합한 사용 사례
- AWS 글로벌 인프라 및 엣지 서비스에서 지원하는 분산 컴퓨팅
- 대기열 및 메시징 개념
- 확장성 기능과 적합한 사용 사례
- 서버리스 기술 및 패턴
- 컨테이너 오케스트레이션

**실습 기술:**
- 워크로드 디커플링으로 구성 요소 크기 독립적 조정
- 크기 조정을 위한 지표 및 조건 식별
- 비즈니스 요구 사항을 충족하는 적합한 컴퓨팅 옵션 선택
- 적합한 리소스 유형 및 크기 선택

#### 3.3 고성능 데이터베이스 솔루션
**핵심 개념:**
- AWS 글로벌 인프라
- 캐싱 전략 및 서비스
- 데이터 액세스 패턴
- 데이터베이스 용량 계획
- 데이터베이스 연결 및 프록시
- 데이터베이스 엔진과 적합한 사용 사례
- 데이터베이스 복제
- 데이터베이스 유형 및 서비스

**실습 기술:**
- 비즈니스 요구 사항을 충족하도록 읽기 전용 복제본 구성
- 데이터베이스 아키텍처 설계
- 적합한 데이터베이스 엔진 결정
- 적합한 데이터베이스 유형 결정
- 캐싱 통합

#### 3.4 고성능 확장 가능한 네트워크 아키텍처
**핵심 개념:**
- 엣지 네트워킹 서비스와 적합한 사용 사례
- 네트워크 아키텍처 설계 방법
- 로드 밸런싱 개념
- 네트워크 연결 옵션

**실습 기술:**
- 다양한 아키텍처에 대한 네트워크 토폴로지 생성
- 크기 조정 가능한 네트워크 구성 결정
- 적합한 리소스 배치 결정
- 적합한 로드 밸런싱 전략 선택

#### 3.5 고성능 데이터 수집 및 변환 솔루션
**핵심 개념:**
- 데이터 분석 및 시각화 서비스와 적합한 사용 사례
- 데이터 수집 패턴
- 데이터 전송 서비스와 적합한 사용 사례
- 데이터 변환 서비스와 적합한 사용 사례
- 수집 액세스 포인트에 대한 보안 액세스
- 스트리밍 데이터 서비스와 적합한 사용 사례

**실습 기술:**
- 데이터 레이크 구축 및 보호
- 데이터 스트리밍 아키텍처 설계
- 데이터 전송 솔루션 설계
- 시각화 전략 구현
- 데이터 처리에 적합한 컴퓨팅 옵션 선택

### 4주차: 비용에 최적화된 아키텍처 설계 (20%)

#### 4.1 비용에 최적화된 스토리지 솔루션 설계
**핵심 개념:**
- 액세스 옵션
- AWS 비용 관리 서비스 기능
- AWS 비용 관리 도구와 적합한 사용 사례
- AWS 스토리지 서비스와 적합한 사용 사례
- 백업 전략
- 블록 스토리지 옵션
- 데이터 수명 주기
- 하이브리드 스토리지 옵션
- 스토리지 액세스 패턴
- 스토리지 계층화
- 스토리지 유형과 연관된 특성

**실습 기술:**
- 적합한 스토리지 전략 설계
- 워크로드에 올바른 스토리지 크기 결정
- 가장 저렴한 데이터 전송 방법 결정
- 스토리지 자동 크기조정 시점 결정
- S3 객체 수명 주기 관리
- 적합한 백업/아카이브 솔루션 선택

#### 4.2 비용에 최적화된 컴퓨팅 솔루션 설계
**핵심 개념:**
- AWS 비용 관리 서비스 기능
- AWS 비용 관리 도구와 적합한 사용 사례
- AWS 글로벌 인프라
- AWS 구매 옵션
- 분산 컴퓨팅 전략
- 하이브리드 컴퓨팅 옵션
- 인스턴스 유형, 패밀리 및 크기
- 컴퓨팅 사용률 최적화
- 크기 조정 전략

**실습 기술:**
- 적합한 로드 밸런싱 전략 결정
- 탄력적인 워크로드에 적합한 크기 조정 방법 및 전략 결정
- 비용 효율적인 AWS 컴퓨팅 서비스 결정
- 다양한 워크로드 클래스에 필요한 가용성 결정
- 워크로드에 적합한 인스턴스 패밀리 및 크기 선택

#### 4.3 비용에 최적화된 데이터베이스 솔루션 설계
**핵심 개념:**
- AWS 비용 관리 서비스 기능
- 캐싱 전략
- 데이터 보존 정책
- 데이터베이스 용량 계획
- 데이터베이스 연결 및 프록시
- 데이터베이스 엔진과 적합한 사용 사례
- 데이터베이스 복제
- 데이터베이스 유형 및 서비스

**실습 기술:**
- 적합한 백업 및 보존 정책 설계
- 적합한 데이터베이스 엔진 결정
- 비용 효율적인 AWS 데이터베이스 서비스 결정
- 비용 효율적인 AWS 데이터베이스 유형 결정
- 데이터베이스 스키마와 데이터 마이그레이션

#### 4.4 비용에 최적화된 네트워크 아키텍처 설계
**핵심 개념:**
- AWS 비용 관리 서비스 기능
- 로드 밸런싱 개념
- NAT 게이트웨이
- 네트워크 연결
- 네트워크 라우팅, 토폴로지 및 피어링
- 네트워크 서비스와 적합한 사용 사례

**실습 기술:**
- 네트워크에 적합한 NAT 게이트웨이 유형 구성
- 적합한 네트워크 연결 구성
- 네트워크 전송 비용을 최소화하는 네트워크 경로 구성
- CDN 및 엣지 캐싱 전략적 요구 사항 결정
- 네트워크 최적화를 위한 기존 워크로드 검토

### 5주차: 통합 실습 및 시험 준비

#### 5.1 종합 아키텍처 설계 실습
- 4개 도메인을 통합한 종합 시나리오 기반 실습
- 실제 비즈니스 요구사항을 반영한 아키텍처 설계
- 팀 프로젝트를 통한 협업 경험

#### 5.2 시험 전략 및 모의고사
- 시험 유형별 문제 해결 전략
- 시간 관리 기법
- 모의고사 및 해설
- 취약점 분석 및 보완

#### 5.3 실무 적용 케이스 스터디
- 실제 기업 사례를 통한 아키텍처 분석
- 기존 솔루션의 개선점 도출
- AWS Well-Architected Framework 적용

## 시험 범위 AWS 서비스 (주요 카테고리별)

### 분석
Amazon Athena, AWS Data Exchange, AWS Data Pipeline, Amazon EMR, AWS Glue, Amazon Kinesis, AWS Lake Formation, Amazon MSK, Amazon OpenSearch Service, Amazon QuickSight, Amazon Redshift

### 애플리케이션 통합
Amazon AppFlow, AWS AppSync, Amazon EventBridge, Amazon MQ, Amazon SNS, Amazon SQS, AWS Step Functions

### AWS 비용 관리
AWS Budgets, AWS Cost and Usage Report, AWS Cost Explorer, Savings Plans

### 컴퓨팅
AWS Batch, Amazon EC2, Amazon EC2 Auto Scaling, AWS Elastic Beanstalk, AWS Outposts, AWS Serverless Application Repository, VMware Cloud on AWS, AWS Wavelength

### 컨테이너
Amazon ECS Anywhere, Amazon EKS Anywhere, Amazon EKS Distro, Amazon ECR, Amazon ECS, Amazon EKS

### 데이터베이스
Amazon Aurora, Amazon Aurora Serverless, Amazon DocumentDB, Amazon DynamoDB, Amazon ElastiCache, Amazon Keyspaces, Amazon Neptune, Amazon QLDB, Amazon RDS, Amazon Redshift

### Machine Learning
Amazon Comprehend, Amazon Forecast, Amazon Fraud Detector, Amazon Kendra, Amazon Lex, Amazon Polly, Amazon Rekognition, Amazon SageMaker, Amazon Textract, Amazon Transcribe, Amazon Translate

### 관리 및 거버넌스
AWS Auto Scaling, AWS CloudFormation, AWS CloudTrail, Amazon CloudWatch, AWS CLI, AWS Compute Optimizer, AWS Config, AWS Control Tower, AWS Health Dashboard, AWS License Manager, Amazon Managed Grafana, Amazon Managed Service for Prometheus, AWS Management Console, AWS Organizations, AWS Proton, AWS Service Catalog, AWS Systems Manager, AWS Trusted Advisor, AWS Well-Architected Tool

### 마이그레이션 및 전송
AWS Application Discovery Service, AWS Application Migration Service, AWS DMS, AWS DataSync, AWS Migration Hub, AWS Snow Family, AWS Transfer Family

### 네트워킹 및 콘텐츠 전송
AWS Client VPN, Amazon CloudFront, AWS Direct Connect, Elastic Load Balancing, AWS Global Accelerator, AWS PrivateLink, Amazon Route 53, AWS Site-to-Site VPN, AWS Transit Gateway, Amazon VPC

### 보안, 자격 증명 및 규정 준수
AWS Artifact, AWS Audit Manager, AWS Certificate Manager, AWS CloudHSM, Amazon Cognito, Amazon Detective, AWS Directory Service, AWS Firewall Manager, Amazon GuardDuty, AWS IAM Identity Center, AWS IAM, Amazon Inspector, AWS KMS, Amazon Macie, AWS Network Firewall, AWS RAM, AWS Secrets Manager, AWS Security Hub, AWS Shield, AWS WAF

### 서버리스
AWS AppSync, AWS Fargate, AWS Lambda

### 스토리지
AWS Backup, Amazon EBS, Amazon EFS, Amazon FSx, Amazon S3, Amazon S3 Glacier, AWS Storage Gateway

## 학습 성과 평가

### 주차별 평가
- 1-4주차: 도메인별 이해도 평가 (퀴즈, 실습 과제)
- 5주차: 종합 평가 (모의고사, 프로젝트 발표)

### 최종 목표
- SAA-C03 시험 합격 (720점/1000점 이상)
- 실무에 적용 가능한 AWS 아키텍처 설계 역량 확보
- AWS Well-Architected Framework 기반 사고력 배양