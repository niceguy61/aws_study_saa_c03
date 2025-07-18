# Day 2-3: IAM 정책 설계

## 📚 학습 목표
- IAM 정책의 구조와 작성 방법 이해
- 관리형 정책과 인라인 정책의 차이점 파악
- JSON 정책 문서 작성 및 적용 실습
- 정책 시뮬레이터를 활용한 권한 테스트

---

## 📋 IAM 정책 구조 이해 (30분)

### 정책 = 권한 규칙서

#### 정책의 기본 구조
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {
          "s3:ExistingObjectTag/Department": "Finance"
        }
      }
    }
  ]
}
```

#### 구성 요소 설명
```
Version: 정책 언어 버전
- "2012-10-17" (현재 최신 버전)

Statement: 권한 규칙 배열
- 하나 이상의 권한 규칙 포함

Effect: 허용/거부 결정
- "Allow": 허용
- "Deny": 거부 (우선순위 높음)

Action: 수행할 작업
- "s3:GetObject": S3 객체 읽기
- "ec2:DescribeInstances": EC2 인스턴스 조회
- "*": 모든 작업

Resource: 대상 리소스
- ARN (Amazon Resource Name) 형식
- "*": 모든 리소스

Condition: 추가 조건 (선택사항)
- 시간, IP, 태그 등 조건부 접근
```

---

## 🔍 정책 유형별 특징 (30분)

### 1. AWS 관리형 정책

#### 특징
```
AWS에서 제공:
- AWS가 생성하고 관리
- 일반적인 사용 사례 커버
- 자동 업데이트
- 버전 관리 자동

장점:
- 즉시 사용 가능
- 모범 사례 적용
- 유지보수 불필요

단점:
- 세밀한 제어 어려움
- 과도한 권한 부여 가능
```

#### 주요 AWS 관리형 정책
```
PowerUserAccess:
- 거의 모든 서비스 접근
- IAM 관리 권한 제외
- 개발자에게 적합

ReadOnlyAccess:
- 모든 서비스 읽기 권한
- 생성/수정/삭제 불가
- 감사자, 모니터링용

AmazonS3FullAccess:
- S3 완전 관리 권한
- 모든 버킷과 객체 접근

AmazonEC2ReadOnlyAccess:
- EC2 서비스 읽기 전용
- 인스턴스 조회만 가능
```

### 2. 고객 관리형 정책

#### 특징
```
사용자가 생성:
- 특정 요구사항에 맞춤
- 세밀한 권한 제어
- 재사용 가능

장점:
- 최소 권한 원칙 적용
- 비즈니스 요구사항 반영
- 버전 관리 가능

단점:
- 작성 복잡성
- 유지보수 필요
- 전문 지식 요구
```

### 3. 인라인 정책

#### 특징
```
직접 연결:
- 특정 사용자/그룹/역할에만 연결
- 일대일 관계
- 재사용 불가

사용 사례:
- 특수한 일회성 권한
- 임시 권한 부여
- 매우 제한적 접근

주의사항:
- 관리 복잡성 증가
- 권한 추적 어려움
- 일반적으로 비추천
```

---

## ✍️ JSON 정책 작성 실습 (45분)

### 실습 1: S3 버킷 읽기 전용 정책

#### 시나리오
```
요구사항:
- 특정 S3 버킷의 객체만 읽기 가능
- 다른 버킷 접근 불가
- 객체 업로드/삭제 불가
```

#### 정책 작성
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::company-documents",
        "arn:aws:s3:::company-documents/*"
      ]
    }
  ]
}
```

#### 실습 과정
```
1단계: 정책 생성
- IAM → Policies → "Create policy"
- JSON 탭 선택
- 위 정책 내용 입력

2단계: 정책 검토
- Policy name: S3-CompanyDocuments-ReadOnly
- Description: 회사 문서 버킷 읽기 전용 접근

3단계: 정책 생성 완료
- "Create policy" 클릭
- 정책 목록에서 확인
```

### 실습 2: EC2 개발 환경 관리 정책

#### 시나리오
```
요구사항:
- 개발용 EC2 인스턴스만 관리
- 태그가 "Environment=Development"인 인스턴스만
- 프로덕션 인스턴스 접근 불가
```

#### 정책 작성
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeImages",
        "ec2:DescribeKeyPairs",
        "ec2:DescribeSecurityGroups"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RebootInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Environment": "Development"
        }
      }
    }
  ]
}
```

### 실습 3: 시간 제한 접근 정책

#### 시나리오
```
요구사항:
- 업무 시간에만 접근 가능 (09:00-18:00)
- 주말 접근 불가
- 특정 IP 대역에서만 접근
```

#### 정책 작성
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "DateGreaterThan": {
          "aws:CurrentTime": "09:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "18:00Z"
        },
        "ForAllValues:StringEquals": {
          "aws:RequestedRegion": "ap-northeast-2"
        },
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

---

## 🧪 정책 시뮬레이터 활용 (30분)

### IAM Policy Simulator란?

#### 기능
```
정책 테스트:
- 실제 실행 없이 권한 확인
- 여러 정책 조합 테스트
- 조건부 접근 시뮬레이션

장점:
- 안전한 테스트 환경
- 실시간 결과 확인
- 디버깅 정보 제공
```

### 실습: 정책 시뮬레이터 사용

#### 1단계: 시뮬레이터 접속
```
URL: https://policysim.aws.amazon.com/
또는: IAM 콘솔 → Policy simulator
```

#### 2단계: 사용자/역할 선택
```
테스트 대상:
- 앞서 생성한 개발자 사용자 선택
- 또는 특정 역할 선택
```

#### 3단계: 서비스 및 작업 선택
```
테스트할 작업:
- Service: Amazon S3
- Action: GetObject
- Resource: arn:aws:s3:::company-documents/test.txt
```

#### 4단계: 시뮬레이션 실행
```
결과 확인:
- Allowed/Denied 결과
- 적용된 정책 목록
- 거부 이유 (있는 경우)
```

---

## 🔒 정책 보안 모범 사례 (30분)

### 1. 최소 권한 원칙

#### 구현 방법
```
단계적 접근:
1. 필요한 최소 권한부터 시작
2. 사용자 요청 시 점진적 확대
3. 정기적 권한 검토 및 정리

예시:
- 처음: ReadOnlyAccess
- 필요 시: 특정 서비스 쓰기 권한 추가
- 검토: 실제 사용하지 않는 권한 제거
```

### 2. 조건부 접근 활용

#### 보안 강화 조건
```
시간 기반:
- 업무 시간에만 접근
- 특정 날짜 범위 제한

위치 기반:
- 특정 IP 대역에서만
- 특정 국가에서만

MFA 기반:
- 중요 작업 시 MFA 필수
- 관리자 권한 시 MFA 필수
```

#### 조건 예시
```json
{
  "Condition": {
    "Bool": {
      "aws:MultiFactorAuthPresent": "true"
    },
    "NumericLessThan": {
      "aws:MultiFactorAuthAge": "3600"
    },
    "IpAddress": {
      "aws:SourceIp": "203.0.113.0/24"
    }
  }
}
```

### 3. 정책 버전 관리

#### 관리 전략
```
버전 관리:
- 정책 변경 시 새 버전 생성
- 이전 버전 백업 유지
- 변경 사유 문서화

롤백 계획:
- 문제 발생 시 이전 버전 복원
- 테스트 환경에서 먼저 검증
- 단계적 배포
```

---

## 📊 정책 관리 및 모니터링 (15분)

### Access Analyzer 활용

#### 기능
```
정책 분석:
- 과도한 권한 식별
- 사용하지 않는 권한 탐지
- 외부 접근 권한 분석

권장사항:
- 정책 최적화 제안
- 보안 위험 알림
- 규정 준수 확인
```

### CloudTrail을 통한 권한 사용 추적

#### 모니터링 항목
```
권한 사용 패턴:
- 자주 사용하는 권한
- 사용하지 않는 권한
- 실패한 권한 요청

보안 이벤트:
- 권한 상승 시도
- 비정상적 접근 패턴
- 새로운 권한 부여
```

---

## 📝 핵심 정리

### IAM 정책 설계 원칙
1. **최소 권한**: 필요한 최소한의 권한만 부여
2. **명시적 허용**: 기본은 거부, 필요한 것만 허용
3. **조건부 접근**: 시간, 위치, MFA 등 조건 활용
4. **정기적 검토**: 권한 사용 현황 정기 점검

### 정책 유형 선택 가이드
- **AWS 관리형**: 일반적인 사용 사례, 빠른 구현
- **고객 관리형**: 세밀한 제어, 재사용 필요
- **인라인**: 특수한 일회성 권한, 최소 사용

### 정책 작성 팁
- **명확한 리소스 지정**: 와일드카드(*) 최소 사용
- **조건 활용**: 보안 강화를 위한 조건부 접근
- **테스트**: 정책 시뮬레이터로 사전 검증
- **문서화**: 정책 목적과 변경 이력 기록

---

## 🤔 토론 주제

1. **정책 설계 시 보안과 편의성의 균형점은?**
   - 너무 제한적 vs 너무 관대한 권한

2. **조건부 접근 정책의 실제 적용 사례는?**
   - 업무 환경에서 활용 가능한 조건들

---

## 📚 다음 시간 예고

**MFA 및 보안 강화**
- 다중 인증(MFA) 설정 및 관리
- 액세스 키 로테이션
- IAM 모범 사례 적용
- 보안 강화 실습

---

## 📖 참고 자료

### AWS 공식 문서
- [IAM 정책](https://docs.aws.amazon.com/iam/latest/userguide/access_policies.html)
- [IAM JSON 정책 요소](https://docs.aws.amazon.com/iam/latest/userguide/reference_policies_elements.html)
- [IAM 정책 예시](https://docs.aws.amazon.com/iam/latest/userguide/access_policies_examples.html)
- [IAM 모범 사례](https://docs.aws.amazon.com/iam/latest/userguide/best-practices.html)

---

> 💡 **오늘의 핵심**: IAM 정책은 AWS 보안의 핵심입니다. JSON 형태의 정책 문서를 통해 세밀한 권한 제어가 가능하며, 최소 권한 원칙과 조건부 접근을 활용하여 보안을 강화할 수 있습니다!