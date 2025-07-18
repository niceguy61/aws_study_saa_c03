# Day 3-3: AWS Organizations

## 📚 학습 목표
- AWS Organizations의 개념과 다중 계정 관리 전략 이해
- 조직 단위(OU) 생성 및 계정 관리 실습
- 서비스 제어 정책(SCP) 적용 방법 습득
- 통합 결제 및 비용 관리 기능 활용

---

## 🏢 AWS Organizations 개념 (30분)

### Organizations = 기업 조직도

#### 전통적인 다중 계정 관리 문제
```
문제점:
- 각 계정별 개별 관리
- 일관성 없는 보안 정책
- 복잡한 비용 추적
- 계정 간 협업 어려움

예시:
개발팀 계정: 각자 다른 보안 설정
운영팀 계정: 서로 다른 명명 규칙
데이터팀 계정: 독립적인 비용 관리
→ 관리 복잡성 증가
```

#### Organizations의 해결책
```
중앙 집중식 관리:
- 단일 지점에서 모든 계정 관리
- 일관된 정책 적용
- 통합 결제 및 비용 관리
- 계정 간 리소스 공유 간소화

비유:
- 대기업의 본사와 지사 관계
- 본사: 전체 정책 수립 및 관리
- 지사: 정책 범위 내에서 자율 운영
```

### Organizations 핵심 구성 요소

#### 1. 조직 (Organization)
```
정의: 여러 AWS 계정을 관리하는 최상위 컨테이너
특징:
- 마스터 계정 1개 + 멤버 계정 여러 개
- 계층적 구조 (조직도와 유사)
- 중앙 집중식 정책 관리
```

#### 2. 조직 단위 (Organizational Unit, OU)
```
정의: 계정들을 논리적으로 그룹화하는 컨테이너
용도:
- 부서별 그룹화 (개발팀, 운영팀, 데이터팀)
- 환경별 그룹화 (개발, 테스트, 운영)
- 프로젝트별 그룹화 (프로젝트A, 프로젝트B)

특징:
- 중첩 가능 (OU 안에 OU)
- 정책 상속 (상위 OU 정책이 하위로 전파)
```

#### 3. 서비스 제어 정책 (Service Control Policy, SCP)
```
정의: 조직 내 계정들이 사용할 수 있는 AWS 서비스 제한
특징:
- 허용 목록 (Allow List) 또는 거부 목록 (Deny List)
- 계정의 최대 권한 설정 (실제 권한은 IAM에서 결정)
- 상속 구조 (상위에서 하위로 전파)
```

---

## 🏗️ Organizations 설정 실습 (45분)

### 실습 시나리오
```
가상 회사: "CloudTech Solutions"
구조:
├── 루트 (Root)
├── 보안 OU (Security)
│   └── 보안 계정
├── 개발 OU (Development)  
│   ├── 개발 계정
│   └── 테스트 계정
└── 운영 OU (Production)
    └── 운영 계정
```

### 1단계: Organizations 활성화

#### 실습: 조직 생성
```
주의사항:
- 실제 환경에서는 신중하게 진행
- 마스터 계정은 변경 불가
- 조직 해체 시 복잡한 과정 필요

시뮬레이션 과정:
1. Organizations 콘솔 접속
2. "Create organization" 클릭
3. 기능 선택:
   - "All features" (권장)
   - "Consolidated billing only" (제한적)
4. 조직 생성 확인
```

### 2단계: 조직 단위(OU) 생성

#### 실습: OU 구조 생성
```
1단계: 보안 OU 생성
- OU name: Security
- Description: 보안 관련 계정들

2단계: 개발 OU 생성  
- OU name: Development
- Description: 개발 및 테스트 환경

3단계: 운영 OU 생성
- OU name: Production  
- Description: 운영 환경 계정들

4단계: 중첩 OU 생성 (선택사항)
Development OU 하위에:
- Dev-Frontend
- Dev-Backend
- Dev-Data
```

### 3단계: 계정 초대 (시뮬레이션)

#### 실제 환경에서의 과정
```
새 계정 생성:
1. "Create account" 클릭
2. 계정 정보 입력:
   - Account name: Dev-Account
   - Email: dev-account@company.com
   - IAM role name: OrganizationAccountAccessRole
3. 적절한 OU로 이동

기존 계정 초대:
1. "Invite account" 클릭
2. 계정 ID 또는 이메일 입력
3. 초대 메시지 작성
4. 대상 계정에서 초대 수락
5. 적절한 OU로 이동
```

---

## 🛡️ 서비스 제어 정책(SCP) 실습 (45분)

### SCP 이해

#### SCP vs IAM 정책 차이
```
IAM 정책:
- 사용자/역할에게 권한 부여
- "무엇을 할 수 있는가?"
- 허용 중심 (Allow)

SCP:
- 계정의 최대 권한 제한
- "무엇을 할 수 없는가?"
- 제한 중심 (Deny)

관계:
실제 권한 = IAM 권한 ∩ SCP 허용 범위
```

#### SCP 작동 방식
```
예시:
SCP: EC2, S3만 허용
IAM: EC2, S3, RDS 권한 부여
결과: EC2, S3만 실제 사용 가능 (RDS 차단)
```

### 실습 1: 개발 환경 제한 정책

#### 시나리오
```
요구사항:
- 개발 계정에서는 비용이 많이 드는 서비스 제한
- 특정 인스턴스 유형만 허용
- 특정 리전에서만 리소스 생성 허용
```

#### SCP 정책 작성
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowedServices",
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "s3:*",
        "iam:*",
        "cloudwatch:*",
        "logs:*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "DenyExpensiveInstances",
      "Effect": "Deny",
      "Action": [
        "ec2:RunInstances"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "ForAllValues:StringNotEquals": {
          "ec2:InstanceType": [
            "t3.micro",
            "t3.small",
            "t3.medium"
          ]
        }
      }
    },
    {
      "Sid": "RestrictRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "ap-northeast-2",
            "us-east-1"
          ]
        }
      }
    }
  ]
}
```

### 실습 2: 운영 환경 보안 정책

#### 시나리오
```
요구사항:
- 운영 계정에서는 보안 관련 서비스 변경 금지
- 루트 사용자 사용 금지
- 특정 시간대에만 민감한 작업 허용
```

#### SCP 정책 작성
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAllExceptRestricted",
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    },
    {
      "Sid": "DenyRootUser",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:userid": "root"
        }
      }
    },
    {
      "Sid": "DenySecurityChanges",
      "Effect": "Deny",
      "Action": [
        "iam:DeleteRole",
        "iam:DeletePolicy",
        "iam:CreateRole",
        "iam:AttachRolePolicy",
        "iam:DetachRolePolicy"
      ],
      "Resource": "*",
      "Condition": {
        "DateGreaterThan": {
          "aws:CurrentTime": "18:00Z"
        }
      }
    },
    {
      "Sid": "DenyDangerousActions",
      "Effect": "Deny",
      "Action": [
        "organizations:LeaveOrganization",
        "account:CloseAccount"
      ],
      "Resource": "*"
    }
  ]
}
```

### 실습 3: SCP 적용 및 테스트

#### SCP 적용 과정
```
1단계: 정책 생성
- Organizations → Policies → "Create policy"
- Policy name: "DevelopmentRestrictions"
- 위에서 작성한 JSON 정책 입력

2단계: OU에 정책 연결
- Development OU 선택
- Policies 탭 → "Attach policy"
- "DevelopmentRestrictions" 선택

3단계: 정책 상속 확인
- 하위 계정들이 정책을 상속받는지 확인
- 정책 적용 상태 모니터링
```

---

## 💰 통합 결제 및 비용 관리 (30분)

### 통합 결제 (Consolidated Billing)

#### 장점
```
비용 절감:
- 볼륨 할인 혜택
- 예약 인스턴스 공유
- 무료 티어 통합 활용

관리 효율성:
- 단일 청구서
- 중앙 집중식 결제
- 계정별 비용 추적

예시:
계정 A: EC2 사용량 500시간
계정 B: EC2 사용량 300시간
→ 총 800시간으로 볼륨 할인 적용
```

### 비용 관리 기능

#### 1. 비용 할당 태그
```
목적: 계정별, 프로젝트별 비용 추적

설정:
1. Organizations → Settings
2. "Cost allocation tags" 활성화
3. 표준 태그 정의:
   - Department: Engineering, Marketing, Sales
   - Project: ProjectA, ProjectB
   - Environment: Dev, Test, Prod

활용:
- Cost Explorer에서 태그별 비용 분석
- 부서별 비용 배분
- 프로젝트 ROI 계산
```

#### 2. 예산 및 알림
```
마스터 계정에서 설정:
- 전체 조직 예산
- OU별 예산
- 계정별 예산

알림 설정:
- 예산 80% 도달 시 경고
- 예산 100% 도달 시 알림
- 예상 초과 시 관리자 통보
```

---

## 🔍 Organizations 모니터링 및 거버넌스 (30분)

### CloudTrail 통합 로깅

#### 조직 전체 로깅 설정
```
마스터 계정에서 설정:
1. CloudTrail → "Create trail"
2. Trail name: "OrganizationTrail"
3. Apply trail to organization: ✓
4. S3 bucket: 중앙 로깅 버킷
5. 모든 멤버 계정의 활동 자동 수집
```

### Config 규칙 관리

#### 조직 전체 규정 준수
```
설정 가능한 규칙:
- 루트 액세스 키 존재 여부
- MFA 활성화 상태
- 보안 그룹 설정 준수
- 암호화 정책 준수

적용 방법:
1. Config → Rules → "Add rule"
2. "Deploy to organization" 선택
3. 적용할 OU 또는 계정 선택
4. 자동 평가 및 알림 설정
```

---

## 🚀 고급 Organizations 패턴 (15분)

### 1. 계정 팩토리 자동화

#### AWS Control Tower 활용
```
기능:
- 표준화된 계정 생성
- 자동 보안 설정
- 규정 준수 모니터링
- 가드레일 자동 적용

장점:
- 일관된 계정 설정
- 보안 모범 사례 자동 적용
- 규정 준수 자동 확인
```

### 2. 리소스 공유

#### AWS Resource Access Manager (RAM)
```
공유 가능한 리소스:
- VPC 서브넷
- Transit Gateway
- Route 53 Resolver 규칙
- License Manager 라이선스

활용 사례:
- 중앙 네트워크 허브 공유
- 공통 DNS 설정 공유
- 라이선스 풀 공유
```

---

## 📝 핵심 정리

### AWS Organizations 핵심 개념
1. **중앙 집중식 관리**: 단일 지점에서 모든 계정 관리
2. **계층적 구조**: OU를 통한 논리적 그룹화
3. **정책 상속**: 상위에서 하위로 정책 전파
4. **통합 결제**: 볼륨 할인 및 중앙 집중식 비용 관리

### SCP 핵심 원칙
- **가드레일 역할**: 계정의 최대 권한 제한
- **IAM과 조합**: 실제 권한은 IAM ∩ SCP
- **상속 구조**: 상위 OU 정책이 하위로 전파
- **명시적 거부**: Deny 정책이 Allow보다 우선

### 실제 적용 시 고려사항
- **계정 구조**: 비즈니스 요구사항에 맞는 OU 설계
- **정책 설계**: 너무 제한적이지 않으면서도 보안 확보
- **모니터링**: 조직 전체 활동 추적 및 분석
- **자동화**: 계정 생성 및 관리 프로세스 자동화

---

## 🤔 토론 주제

1. **어떤 기준으로 OU를 구성해야 할까?**
   - 부서별 vs 환경별 vs 프로젝트별

2. **SCP 정책은 얼마나 제한적이어야 할까?**
   - 보안 vs 개발 생산성의 균형점

---

## 📚 다음 시간 예고

**페더레이션 및 SSO**
- 외부 자격 증명 공급자와의 연동
- AWS SSO(Single Sign-On) 기본 설정
- 기업 Active Directory와 AWS 연동
- SAML 및 OIDC 기반 인증

---

> 💡 **오늘의 핵심**: AWS Organizations는 다중 계정 환경의 중앙 집중식 관리를 가능하게 합니다. OU와 SCP를 통해 계층적 구조와 정책 관리를 구현하고, 통합 결제로 비용 효율성을 높일 수 있습니다!