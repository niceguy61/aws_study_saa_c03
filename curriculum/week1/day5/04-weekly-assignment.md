# Day 5-4: 주간 과제 설명 및 시작

## 📚 학습 목표
- 1주차 종합 과제 "안전한 웹 서비스 환경 구축" 이해
- 온라인 쇼핑몰 인프라 설계 시나리오 분석
- ReadOnlyAccess 역할 부여 방법 실습
- 과제 진행 가이드라인 및 평가 기준 파악

---

## 🎯 과제 개요 (30분)

### 과제명: "안전한 웹 서비스 환경 구축"

#### 비즈니스 시나리오
```
회사: TechMart (온라인 쇼핑몰 스타트업)
상황: AWS 클라우드로 인프라 마이그레이션 결정
목표: 안전하고 확장 가능한 웹 서비스 환경 구축

현재 상황:
- 온프레미스 서버 3대 운영 중
- 월 평균 방문자 10만명, 피크 시 50만명
- 개인정보 및 결제 정보 처리 (보안 중요)
- 24/7 서비스 가용성 필요
- 향후 글로벌 확장 계획
```

#### 기술 요구사항
```
인프라 요구사항:
✅ 3-tier 아키텍처 (Web, App, DB)
✅ 고가용성 (99.9% 이상)
✅ 자동 확장 (트래픽 변화 대응)
✅ 보안 강화 (PCI DSS 수준)
✅ 비용 최적화
✅ 모니터링 및 로깅

성능 요구사항:
✅ 응답시간 2초 이내
✅ 동시 접속자 1만명 지원
✅ 데이터베이스 백업 및 복구
✅ CDN을 통한 글로벌 서비스

보안 요구사항:
✅ 네트워크 격리 및 접근 제어
✅ 데이터 암호화 (전송/저장)
✅ 로그 수집 및 모니터링
✅ 침입 탐지 및 대응
```

### 과제 범위 및 제한사항

#### 이번 주차 범위 (1주차 학습 내용)
```
필수 구현 요소:
1. VPC 네트워크 설계
   - 적절한 CIDR 블록 선택
   - 3-tier 서브넷 구성 (퍼블릭/프라이빗/DB)
   - 다중 AZ 배치

2. 보안 설정
   - IAM 역할 및 정책 설계
   - 보안 그룹 및 NACL 구성
   - 최소 권한 원칙 적용

3. 네트워크 연결
   - 인터넷 게이트웨이 설정
   - NAT 게이트웨이 구성
   - VPC 엔드포인트 활용

4. 모니터링 설정
   - VPC Flow Logs 활성화
   - CloudWatch 모니터링
   - 기본 알림 설정

향후 주차에서 추가될 요소:
- EC2 인스턴스 및 Auto Scaling (2주차)
- 로드 밸런서 및 CDN (2주차)
- RDS 데이터베이스 (2주차)
- 보안 서비스 (AWS WAF, Shield) (추후)
```

---

## 🏗️ 아키텍처 설계 가이드 (45분)

### 네트워크 아키텍처 설계

#### 1단계: VPC 및 CIDR 설계
```
VPC 설계 고려사항:
- 현재 요구사항: 인스턴스 50개 내외
- 향후 확장: 인스턴스 500개까지 확장 가능
- 서브넷 분할: 3-tier × 3-AZ = 9개 서브넷
- 여유 공간: 50% 이상 확보

권장 CIDR 설계:
VPC CIDR: 10.0.0.0/16 (65,536개 IP)

서브넷 설계:
AZ-A (ap-northeast-2a):
- 퍼블릭: 10.0.1.0/24 (256개 IP)
- 프라이빗: 10.0.11.0/24 (256개 IP)
- DB: 10.0.21.0/24 (256개 IP)

AZ-B (ap-northeast-2b):
- 퍼블릭: 10.0.2.0/24
- 프라이빗: 10.0.12.0/24
- DB: 10.0.22.0/24

AZ-C (ap-northeast-2c):
- 퍼블릭: 10.0.3.0/24
- 프라이빗: 10.0.13.0/24
- DB: 10.0.23.0/24

예비 서브넷:
- 관리용: 10.0.100.0/24
- 확장용: 10.0.101.0/24 ~ 10.0.200.0/24
```

#### 2단계: 보안 그룹 설계
```
보안 그룹 구조:

1. ALB-SG (Application Load Balancer)
인바운드:
- HTTP (80): 0.0.0.0/0
- HTTPS (443): 0.0.0.0/0
아웃바운드:
- HTTP (80): WebServer-SG
- HTTPS (443): WebServer-SG

2. WebServer-SG
인바운드:
- HTTP (80): ALB-SG
- HTTPS (443): ALB-SG
- SSH (22): Bastion-SG
아웃바운드:
- HTTP (8080): AppServer-SG
- HTTPS (443): 0.0.0.0/0 (외부 API)

3. AppServer-SG
인바운드:
- HTTP (8080): WebServer-SG
- SSH (22): Bastion-SG
아웃바운드:
- MySQL (3306): Database-SG
- HTTPS (443): 0.0.0.0/0 (외부 API)

4. Database-SG
인바운드:
- MySQL (3306): AppServer-SG
아웃바운드:
- 없음 (기본 거부)

5. Bastion-SG
인바운드:
- SSH (22): [관리자 IP]/32
아웃바운드:
- SSH (22): 10.0.0.0/16
```

### IAM 설계

#### 역할 기반 접근 제어
```
IAM 역할 설계:

1. EC2-WebServer-Role
정책:
- CloudWatchAgentServerPolicy (로그 수집)
- AmazonS3ReadOnlyAccess (정적 파일 접근)
- 커스텀 정책 (필요한 최소 권한)

2. EC2-AppServer-Role
정책:
- CloudWatchAgentServerPolicy
- AmazonS3ReadOnlyAccess
- SecretsManagerReadWrite (DB 연결 정보)
- 커스텀 정책 (외부 API 호출)

3. RDS-MonitoringRole
정책:
- AmazonRDSEnhancedMonitoringRole

4. Lambda-ExecutionRole (향후 사용)
정책:
- AWSLambdaBasicExecutionRole
- VPC 접근 권한

IAM 사용자 그룹:
1. Developers
   - EC2 읽기 권한
   - CloudWatch 로그 읽기
   - 개발 환경 리소스 관리

2. Operators
   - 모든 리소스 읽기 권한
   - 인스턴스 시작/중지
   - 모니터링 및 알림 관리

3. Administrators
   - 전체 권한 (제한적)
   - 보안 정책 관리
   - 비용 관리
```

---

## 🔧 실습 가이드 (60분)

### 실습 1: VPC 및 네트워크 구성

#### VPC 생성 체크리스트
```
□ VPC 생성 (10.0.0.0/16)
□ 인터넷 게이트웨이 생성 및 연결
□ 서브넷 9개 생성 (3-tier × 3-AZ)
□ 라우팅 테이블 생성 및 연결
  - 퍼블릭 라우팅 테이블 (IGW 연결)
  - 프라이빗 라우팅 테이블 (NAT 연결)
  - DB 라우팅 테이블 (로컬만)
□ NAT 게이트웨이 생성 (고가용성 고려)
□ VPC 엔드포인트 생성 (S3, EC2)
```

#### 네트워크 설정 스크립트 (참고용)
```bash
#!/bin/bash
# VPC 생성 스크립트 (AWS CLI 사용)

# VPC 생성
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=TechMart-VPC}]' \
  --query 'Vpc.VpcId' --output text)

echo "Created VPC: $VPC_ID"

# 인터넷 게이트웨이 생성
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=TechMart-IGW}]' \
  --query 'InternetGateway.InternetGatewayId' --output text)

# IGW를 VPC에 연결
aws ec2 attach-internet-gateway \
  --internet-gateway-id $IGW_ID \
  --vpc-id $VPC_ID

echo "Created and attached IGW: $IGW_ID"

# 서브넷 생성 (예시: AZ-A 퍼블릭 서브넷)
SUBNET_PUBLIC_A=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone ap-northeast-2a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Public-Subnet-AZ-A}]' \
  --query 'Subnet.SubnetId' --output text)

echo "Created subnet: $SUBNET_PUBLIC_A"

# 추가 서브넷들도 동일한 방식으로 생성...
```

### 실습 2: 보안 설정

#### 보안 그룹 생성 실습
```
1. ALB 보안 그룹 생성
   - Name: TechMart-ALB-SG
   - Description: Security group for Application Load Balancer
   - VPC: TechMart-VPC
   - 인바운드 규칙:
     * HTTP (80): 0.0.0.0/0
     * HTTPS (443): 0.0.0.0/0

2. 웹 서버 보안 그룹 생성
   - Name: TechMart-WebServer-SG
   - 인바운드 규칙:
     * HTTP (80): TechMart-ALB-SG
     * HTTPS (443): TechMart-ALB-SG
     * SSH (22): TechMart-Bastion-SG

3. 보안 그룹 체이닝 테스트
   # 보안 그룹 참조가 올바르게 작동하는지 확인
   # 각 계층 간 통신 테스트
```

#### IAM 역할 생성 실습
```
1. EC2 웹 서버 역할 생성
   - 역할 이름: TechMart-WebServer-Role
   - 신뢰 관계: EC2 서비스
   - 정책 연결:
     * CloudWatchAgentServerPolicy
     * AmazonS3ReadOnlyAccess

2. 커스텀 정책 생성
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::techmart-static-assets/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:ap-northeast-2:*:*"
    }
  ]
}
```

### 실습 3: 모니터링 설정

#### VPC Flow Logs 및 CloudWatch 설정
```
1. Flow Logs 활성화
   - 대상: VPC 레벨
   - 필터: All (Accept + Reject)
   - 목적지: CloudWatch Logs
   - 로그 그룹: /aws/vpc/techmart-flowlogs
   - 보존 기간: 7일

2. CloudWatch 대시보드 생성
   - 대시보드 이름: TechMart-Network-Dashboard
   - 위젯 구성:
     * NAT Gateway 트래픽
     * VPC Flow Logs 볼륨
     * 네트워크 연결 수

3. 기본 알림 설정
   - 높은 NAT 트래픽 알림
   - Flow Logs 중단 알림
   - 보안 이벤트 알림
```

---

## 👨‍🏫 강사 접근 권한 설정 (30분)

### ReadOnlyAccess 역할 부여 방법

#### 교차 계정 역할 생성
```
목적: 강사가 학생의 AWS 계정에 읽기 전용으로 접근하여 과제 검토

1. IAM 역할 생성
   - 역할 이름: SAA-Instructor-ReadOnly-Role
   - 신뢰 관계: 다른 AWS 계정
   - 강사 계정 ID: [강사가 제공하는 계정 ID]
   - 외부 ID: [강사가 제공하는 고유 ID]

2. 정책 연결
   - ReadOnlyAccess (AWS 관리형 정책)
   - ViewOnlyAccess (추가 읽기 권한)

3. 신뢰 관계 정책 예시
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::[강사-계정-ID]:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "[고유-외부-ID]"
        }
      }
    }
  ]
}
```

#### 역할 ARN 제출 방법
```
1. 생성된 역할 ARN 확인
   - IAM → Roles → SAA-Instructor-ReadOnly-Role
   - Role ARN 복사: arn:aws:iam::[학생-계정-ID]:role/SAA-Instructor-ReadOnly-Role

2. 과제 제출 시 포함 정보
   - 학생 이름 및 학번
   - AWS 계정 ID
   - 역할 ARN
   - 과제 완료 일시
   - 주요 리소스 목록

3. 제출 양식 예시
---
과제 제출 정보
- 이름: 홍길동
- 학번: 2024001
- AWS 계정 ID: 123456789012
- 강사 접근 역할 ARN: arn:aws:iam::123456789012:role/SAA-Instructor-ReadOnly-Role
- 제출 일시: 2024-01-05 18:00
- VPC ID: vpc-0123456789abcdef0
- 주요 리소스:
  * 서브넷 9개 생성 완료
  * 보안 그룹 5개 설정 완료
  * NAT 게이트웨이 2개 구성 완료
  * VPC Flow Logs 활성화 완료
---
```

### 강사 검토 포인트

#### 평가 기준 및 체크리스트
```
네트워크 설계 (30점):
□ VPC CIDR 적절성 (5점)
□ 서브넷 구성 정확성 (10점)
□ 라우팅 테이블 설정 (10점)
□ NAT 게이트웨이 고가용성 (5점)

보안 설정 (30점):
□ 보안 그룹 계층적 설계 (15점)
□ IAM 역할 최소 권한 적용 (10점)
□ 네트워크 ACL 활용 (5점)

모니터링 (20점):
□ VPC Flow Logs 설정 (10점)
□ CloudWatch 대시보드 (5점)
□ 기본 알림 설정 (5점)

문서화 (20점):
□ 아키텍처 다이어그램 (10점)
□ 설정 근거 설명 (5점)
□ 보안 고려사항 (5점)

총점: 100점
합격 기준: 70점 이상
```

---

## 📋 과제 진행 가이드라인 (15분)

### 진행 일정

#### 주간 일정
```
Day 5 (오늘): 과제 시작
- 요구사항 분석 및 설계
- VPC 및 기본 네트워크 구성
- 진행률: 30%

주말: 개별 작업
- 보안 설정 완료
- 모니터링 구성
- 문서화 작업
- 진행률: 80%

다음 주 월요일: 최종 제출
- 최종 검토 및 테스트
- 제출 문서 완성
- 강사 접근 권한 설정
- 진행률: 100%
```

### 제출 요구사항

#### 필수 제출 문서
```
1. 아키텍처 다이어그램
   - 네트워크 구성도
   - 보안 그룹 관계도
   - 트래픽 흐름도

2. 설정 문서
   - 리소스 목록 및 설정값
   - 보안 정책 설명
   - 모니터링 구성 내역

3. 테스트 결과
   - 네트워크 연결 테스트
   - 보안 설정 검증
   - 모니터링 동작 확인

4. 학습 보고서
   - 설계 결정 근거
   - 어려웠던 점과 해결 방법
   - 추가 개선 아이디어
```

### 도움 받기

#### 지원 채널
```
1. 실시간 질의응답
   - 카카오톡 오픈채팅: [링크]
   - 응답 시간: 평일 09:00-18:00

2. 이메일 문의
   - 이메일: instructor@example.com
   - 상세한 기술 문의 시 활용

3. 동료 학습
   - 스터디 그룹 구성 권장
   - 서로 리뷰 및 피드백

4. 참고 자료
   - AWS 공식 문서
   - Well-Architected Framework
   - 이전 실습 자료
```

---

## 📝 핵심 정리

### 과제 성공 포인트
1. **요구사항 이해**: 비즈니스 요구사항을 기술적으로 해석
2. **체계적 설계**: 네트워크부터 보안까지 단계적 접근
3. **모범 사례 적용**: AWS Well-Architected 원칙 준수
4. **문서화**: 설계 결정의 근거와 과정 기록

### 평가 관점
- **기능성**: 요구사항 충족 여부
- **보안성**: 보안 모범 사례 적용
- **확장성**: 미래 성장 고려
- **경제성**: 비용 효율적 설계

### 학습 효과
- **실무 경험**: 실제 프로젝트와 유사한 경험
- **통합 사고**: 개별 서비스를 통합한 솔루션 설계
- **문제 해결**: 제약 조건 하에서 최적해 도출
- **커뮤니케이션**: 기술적 내용의 문서화 및 발표

---

## 🚀 과제 시작!

### 지금 시작할 일
1. **요구사항 재검토**: 비즈니스 시나리오 완전 이해
2. **설계 스케치**: 종이에 먼저 아키텍처 그려보기
3. **VPC 생성**: 기본 네트워크 인프라 구축 시작
4. **진행 상황 기록**: 각 단계별 스크린샷 및 메모

### 성공을 위한 팁
- **작은 단위로 진행**: 한 번에 모든 것을 하려 하지 말기
- **테스트 중심**: 각 단계마다 동작 확인
- **문서화 병행**: 나중에 몰아서 하지 말고 진행하면서 기록
- **도움 요청**: 막히면 주저하지 말고 질문하기

---

> 💡 **과제의 의미**: 이 과제는 단순한 실습이 아닙니다. 실제 기업에서 클라우드 인프라를 설계하고 구축하는 과정을 경험하는 소중한 기회입니다. 완벽하지 않아도 괜찮으니 최선을 다해 도전해보세요!

**화이팅! 🎯**