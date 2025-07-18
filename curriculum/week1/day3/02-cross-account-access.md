# Day 3-2: 교차 계정 액세스

## 📚 학습 목표
- 여러 AWS 계정 간 리소스 공유 방법 이해
- 교차 계정 역할 설정 및 전환 실습
- AWS STS(Security Token Service) 심화 학습
- 실제 기업 환경의 다중 계정 전략 파악

---

## 🏢 다중 계정 전략 이해 (30분)

### 왜 계정을 분리하는가?

#### 단일 계정의 문제점
```
보안 위험:
- 모든 리소스가 한 계정에 집중
- 권한 관리 복잡성 증가
- 실수로 인한 전체 시스템 영향

관리 어려움:
- 개발/테스트/운영 환경 혼재
- 비용 추적 어려움
- 규정 준수 복잡성
```

#### 다중 계정의 장점
```
보안 강화:
- 계정 레벨 격리
- 폭발 반경(Blast Radius) 제한
- 세밀한 권한 제어

운영 효율성:
- 환경별 명확한 분리
- 팀별 독립적 관리
- 비용 추적 용이성

규정 준수:
- 감사 범위 명확화
- 데이터 주권 준수
- 규제 요구사항 충족
```

### 일반적인 계정 분리 패턴

#### 환경별 분리
```
개발 계정 (Development):
- 개발자 실험 환경
- 새로운 기능 테스트
- 비용 제한 설정

스테이징 계정 (Staging):
- 운영 환경과 유사한 테스트
- 성능 테스트
- 배포 전 최종 검증

운영 계정 (Production):
- 실제 서비스 운영
- 엄격한 접근 제어
- 고가용성 구성
```

#### 팀별 분리
```
프론트엔드 팀 계정:
- 웹 애플리케이션 리소스
- CDN 및 정적 콘텐츠

백엔드 팀 계정:
- API 서버 및 데이터베이스
- 마이크로서비스 인프라

데이터 팀 계정:
- 데이터 레이크 및 분석 도구
- 머신러닝 파이프라인
```

---

## 🔗 교차 계정 역할 설정 실습 (45분)

### 실습 시나리오
```
상황:
- 계정 A: 개발 계정 (본인 계정)
- 계정 B: 운영 계정 (시뮬레이션)
- 개발자가 운영 계정의 리소스를 읽기 전용으로 모니터링

목표:
- 개발 계정에서 운영 계정으로 역할 전환
- 안전한 교차 계정 접근 구현
```

### 1단계: 신뢰 관계 이해

#### 신뢰 정책 (Trust Policy)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT-A:user/developer"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-external-id"
        },
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```

#### 구성 요소 설명
```
Principal: 누가 이 역할을 사용할 수 있는가?
- AWS 계정, 사용자, 역할, 서비스 지정

Action: 어떤 작업을 허용하는가?
- sts:AssumeRole (역할 전환)

Condition: 어떤 조건에서 허용하는가?
- ExternalId: 추가 보안 계층
- MFA: 다중 인증 필수
- 시간, IP, 태그 등 다양한 조건
```

### 2단계: 교차 계정 역할 생성 (시뮬레이션)

#### 실습: 운영 계정 역할 시뮬레이션
```
실제로는 다른 계정이 필요하지만, 
동일 계정 내에서 교차 계정 패턴을 시뮬레이션

1단계: 운영팀 역할 생성
- Role name: ProductionMonitoringRole
- Trusted entity: AWS account (현재 계정)
- Principal: 특정 사용자 또는 역할

2단계: 권한 정책 연결
- CloudWatchReadOnlyAccess
- AmazonEC2ReadOnlyAccess
- AmazonS3ReadOnlyAccess

3단계: 신뢰 정책 설정
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::[현재계정ID]:user/developer-[본인이름]"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "prod-monitoring-2024"
        }
      }
    }
  ]
}
```

### 3단계: 개발자 사용자 권한 설정

#### 역할 전환 권한 부여
```
개발자 사용자에게 연결할 정책:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::[계정ID]:role/ProductionMonitoringRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "prod-monitoring-2024"
        }
      }
    }
  ]
}
```

### 4단계: 역할 전환 테스트

#### AWS CLI를 통한 역할 전환
```bash
# 1. 현재 자격 증명 확인
aws sts get-caller-identity

# 2. 역할 전환
aws sts assume-role \
  --role-arn "arn:aws:iam::[계정ID]:role/ProductionMonitoringRole" \
  --role-session-name "dev-monitoring-session" \
  --external-id "prod-monitoring-2024"

# 3. 반환된 임시 자격 증명을 환경변수로 설정
export AWS_ACCESS_KEY_ID="[임시 액세스 키]"
export AWS_SECRET_ACCESS_KEY="[임시 시크릿 키]"
export AWS_SESSION_TOKEN="[세션 토큰]"

# 4. 역할 전환 확인
aws sts get-caller-identity

# 5. 권한 테스트
aws ec2 describe-instances
aws s3 ls
```

---

## 🔐 AWS STS 심화 학습 (30분)

### STS (Security Token Service) 이해

#### STS의 핵심 기능
```
임시 자격 증명 발급:
- AssumeRole: 역할 전환
- GetSessionToken: 세션 토큰 획득
- AssumeRoleWithWebIdentity: 웹 자격 증명 연동
- AssumeRoleWithSAML: SAML 연동

보안 기능:
- 시간 제한 (15분 ~ 12시간)
- 조건부 접근 제어
- 외부 ID 검증
- MFA 토큰 검증
```

#### 임시 자격 증명 구조
```
반환되는 정보:
- AccessKeyId: 임시 액세스 키
- SecretAccessKey: 임시 시크릿 키
- SessionToken: 세션 토큰
- Expiration: 만료 시간

특징:
- 모든 API 호출에 SessionToken 필요
- 만료 시간 후 자동 무효화
- 갱신 불가 (새로 발급 필요)
```

### STS 보안 모범 사례

#### 1. 최소 권한 원칙
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::specific-bucket/*",
      "Condition": {
        "DateLessThan": {
          "aws:CurrentTime": "2024-12-31T23:59:59Z"
        }
      }
    }
  ]
}
```

#### 2. 외부 ID 활용
```
목적: 혼동된 대리인 공격 방지
사용법: 
- 역할 생성 시 외부 ID 설정
- 역할 전환 시 동일한 외부 ID 제공
- 예측 불가능한 값 사용 (UUID 등)

예시:
ExternalId: "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
```

#### 3. 세션 지속 시간 제한
```
기본값: 1시간
최소값: 15분
최대값: 12시간 (역할 설정에 따라)

권장사항:
- 일반 작업: 1-2시간
- 민감한 작업: 15-30분
- 자동화 작업: 필요에 따라 조정
```

---

## 🏗️ 실제 기업 환경 사례 (30분)

### 대기업 다중 계정 전략

#### 계정 구조 예시
```
루트 조직 (Root Organization)
├── 보안 계정 (Security)
│   ├── CloudTrail 중앙 로깅
│   ├── Config 규칙 관리
│   └── GuardDuty 마스터
├── 로그 계정 (Logging)
│   ├── 모든 계정의 로그 수집
│   └── 장기 보관 및 분석
├── 공유 서비스 계정 (Shared Services)
│   ├── DNS (Route 53)
│   ├── 네트워크 허브 (Transit Gateway)
│   └── 공통 도구
└── 워크로드 계정들
    ├── 개발 계정 (Dev)
    ├── 테스트 계정 (Test)
    ├── 스테이징 계정 (Staging)
    └── 운영 계정 (Production)
```

### 권한 관리 전략

#### 중앙 집중식 권한 관리
```
Identity Account (신원 계정):
- 모든 사용자 계정 중앙 관리
- SSO (Single Sign-On) 구성
- 그룹 기반 권한 관리

각 워크로드 계정:
- 신원 계정의 그룹별 역할 생성
- 최소 권한 원칙 적용
- 환경별 차등 권한
```

#### 역할 명명 규칙
```
패턴: [Environment]-[Team]-[Permission Level]-Role

예시:
- Prod-Backend-ReadOnly-Role
- Dev-Frontend-PowerUser-Role
- Staging-Data-Admin-Role
- Shared-Network-Operator-Role
```

---

## 🔧 고급 교차 계정 패턴 (30분)

### 1. 역할 체인 (Role Chaining)

#### 시나리오
```
사용자 → 중간 역할 → 최종 역할 → 리소스 접근

예시:
개발자 → DevOpsRole → ProductionRole → 운영 리소스
```

#### 구현 방법
```
1단계: 중간 역할 생성 (DevOpsRole)
- 개발자가 전환 가능
- ProductionRole 전환 권한 보유

2단계: 최종 역할 생성 (ProductionRole)
- DevOpsRole이 전환 가능
- 실제 리소스 접근 권한

3단계: 체인 실행
- 개발자 → DevOpsRole 전환
- DevOpsRole → ProductionRole 전환
- ProductionRole로 리소스 접근
```

### 2. 조건부 교차 계정 접근

#### 시간 기반 접근
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::DEV-ACCOUNT:user/developer"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "DateGreaterThan": {
          "aws:CurrentTime": "09:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "18:00Z"
        },
        "ForAllValues:StringEquals": {
          "aws:RequestedRegion": "ap-northeast-2"
        }
      }
    }
  ]
}
```

#### IP 기반 접근 제한
```json
{
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": [
        "203.0.113.0/24",
        "198.51.100.0/24"
      ]
    }
  }
}
```

---

## 📊 교차 계정 모니터링 (15분)

### CloudTrail을 통한 추적

#### 모니터링 항목
```
AssumeRole 이벤트:
- 누가 언제 역할을 전환했는가?
- 어떤 역할로 전환했는가?
- 어디서 접근했는가? (IP, 지역)

권한 사용 패턴:
- 전환된 역할로 어떤 작업을 수행했는가?
- 비정상적인 접근 패턴은 없는가?
- 권한 남용 징후는 없는가?
```

#### 알림 설정
```
CloudWatch 알람:
- 새로운 교차 계정 역할 생성
- 비정상적인 시간대 접근
- 실패한 역할 전환 시도 급증
- 민감한 리소스 접근
```

---

## 📝 핵심 정리

### 교차 계정 액세스 핵심 개념
1. **계정 분리**: 보안, 관리, 규정 준수를 위한 논리적 격리
2. **신뢰 관계**: Principal을 통한 명시적 신뢰 설정
3. **임시 자격 증명**: STS를 통한 시간 제한 접근
4. **조건부 접근**: 시간, 위치, MFA 등 추가 보안 조건

### 보안 모범 사례
- **최소 권한**: 필요한 최소한의 권한만 부여
- **외부 ID**: 혼동된 대리인 공격 방지
- **시간 제한**: 적절한 세션 지속 시간 설정
- **모니터링**: 모든 교차 계정 활동 추적

### 실제 적용 시 고려사항
- **계정 구조**: 비즈니스 요구사항에 맞는 계정 분리
- **권한 관리**: 중앙 집중식 vs 분산식 관리 전략
- **자동화**: 역할 생성 및 관리 자동화
- **거버넌스**: 명명 규칙 및 정책 표준화

---

## 🤔 토론 주제

1. **어떤 기준으로 계정을 분리해야 할까?**
   - 환경별 vs 팀별 vs 프로젝트별

2. **교차 계정 접근 시 보안과 편의성의 균형점은?**
   - 너무 제한적 vs 너무 관대한 접근

---

## 📚 다음 시간 예고

**AWS Organizations**
- 다중 계정 중앙 관리
- 조직 단위(OU) 생성 및 계정 관리
- 서비스 제어 정책(SCP) 적용
- 통합 결제 및 비용 관리

---

> 💡 **오늘의 핵심**: 교차 계정 액세스는 다중 계정 환경에서 안전하게 리소스를 공유하는 핵심 메커니즘입니다. 신뢰 관계와 조건부 접근을 통해 보안을 유지하면서도 필요한 협업을 가능하게 합니다!