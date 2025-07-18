# Day 3-1: IAM 역할(Role) 심화

## 📚 학습 목표
- IAM 역할의 개념과 사용 시나리오 완전 이해
- EC2 인스턴스용 역할 생성 및 적용 실습
- 역할 전환(Switch Role) 메커니즘 이해
- 서비스 간 권한 위임 패턴 습득

---

## 🎭 IAM 역할 = 임시 출입증 (30분)

### 역할 vs 사용자 비교

#### 사용자 (User) = 정규직 사원
```
특징:
- 영구적 자격 증명 (Access Key)
- 개인 전용 계정
- 장기간 사용
- 직접적 권한 부여

비유:
- 정규직 사원의 사원증
- 개인 전용, 양도 불가
- 퇴사 전까지 계속 사용
```

#### 역할 (Role) = 임시 출입증
```
특징:
- 임시 자격 증명 (STS Token)
- 여러 개체가 공유 가능
- 단기간 사용 (기본 1시간)
- 조건부 권한 부여

비유:
- 방문자용 임시 출입증
- 여러 사람이 사용 가능
- 시간 제한 있음
- 특정 조건에서만 발급
```

### 역할 사용 시나리오

#### 1. EC2 인스턴스가 AWS 서비스 접근
```
문제 상황:
- EC2에서 S3 버킷에 파일 업로드 필요
- Access Key를 EC2에 저장? (보안 위험)
- 하드코딩? (매우 위험)

해결책:
- EC2용 IAM 역할 생성
- S3 접근 권한 부여
- EC2에 역할 연결
- 자동으로 임시 자격 증명 획득
```

#### 2. Lambda 함수가 DynamoDB 접근
```
시나리오:
- 사용자가 API 호출
- Lambda 함수 실행
- DynamoDB에 데이터 저장

역할 활용:
- Lambda 실행 역할 생성
- DynamoDB 쓰기 권한 부여
- Lambda에 역할 연결
- 함수 실행 시 자동 권한 획득
```

#### 3. 교차 계정 접근
```
상황:
- 개발 계정과 운영 계정 분리
- 개발자가 운영 계정 모니터링 필요
- 직접 계정 공유는 보안 위험

해결:
- 운영 계정에 모니터링 역할 생성
- 개발 계정 사용자에게 역할 전환 권한
- 필요 시에만 임시 접근
```

---

## 🖥️ EC2 인스턴스용 역할 실습 (45분)

### 실습 시나리오
```
목표:
- EC2 인스턴스에서 S3 버킷 접근
- 하드코딩된 키 없이 안전한 접근
- 자동으로 권한 획득 및 갱신
```

### 1단계: S3 버킷 생성

#### 실습: 테스트용 S3 버킷 생성
```
1. S3 콘솔 접속
2. "Create bucket" 클릭
3. 버킷 설정:
   - Bucket name: saa-study-[본인이름]-test
   - Region: ap-northeast-2 (Seoul)
   - 나머지 설정: 기본값 유지
4. "Create bucket" 클릭
```

### 2단계: IAM 역할 생성

#### 실습: EC2용 S3 접근 역할 생성
```
1단계: 역할 생성 시작
- IAM → Roles → "Create role"
- Trusted entity type: "AWS service"
- Use case: "EC2" 선택

2단계: 권한 정책 연결
- "AmazonS3ReadOnlyAccess" 검색 및 선택
- 또는 커스텀 정책 생성 (아래 참조)

3단계: 역할 이름 및 설명
- Role name: EC2-S3-ReadOnly-Role
- Description: EC2 인스턴스가 S3 버킷에 읽기 접근

4단계: 역할 생성 완료
- "Create role" 클릭
- 역할 ARN 확인 및 복사
```

#### 커스텀 정책 예시 (선택사항)
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
        "arn:aws:s3:::saa-study-*",
        "arn:aws:s3:::saa-study-*/*"
      ]
    }
  ]
}
```

### 3단계: EC2 인스턴스 생성

#### 실습: 역할이 연결된 EC2 인스턴스 생성
```
1단계: EC2 인스턴스 시작
- EC2 → Instances → "Launch instances"
- Name: saa-study-role-test
- AMI: Amazon Linux 2023 (Free tier)
- Instance type: t3.micro

2단계: IAM 역할 연결
- Advanced details 섹션 확장
- IAM instance profile: "EC2-S3-ReadOnly-Role" 선택

3단계: 보안 그룹 설정
- Security group: 새로 생성
- SSH (22): My IP에서만 접근 허용

4단계: 키 페어 설정
- 기존 키 페어 선택 또는 새로 생성
- 키 파일 안전하게 보관

5단계: 인스턴스 시작
- "Launch instance" 클릭
- 인스턴스 상태 확인 (Running)
```

### 4단계: 역할 동작 테스트

#### 실습: EC2에서 S3 접근 테스트
```
1단계: EC2 인스턴스 접속
- SSH를 통해 인스턴스 접속
- ssh -i your-key.pem ec2-user@[인스턴스-IP]

2단계: AWS CLI 설치 확인
- aws --version
- (이미 설치되어 있음)

3단계: 자격 증명 확인
- aws sts get-caller-identity
- 역할 ARN이 표시되는지 확인

4단계: S3 접근 테스트
- aws s3 ls (모든 버킷 목록)
- aws s3 ls s3://saa-study-[본인이름]-test/
- 성공적으로 접근되는지 확인

5단계: 권한 제한 테스트
- aws s3 cp test.txt s3://saa-study-[본인이름]-test/
- 쓰기 권한이 없어 실패하는지 확인
```

---

## 🔄 역할 전환(Switch Role) 실습 (30분)

### 역할 전환 메커니즘 이해

#### STS (Security Token Service)
```
동작 과정:
1. 사용자가 역할 전환 요청
2. STS가 신뢰 관계 확인
3. 임시 자격 증명 발급 (1시간 유효)
4. 임시 자격 증명으로 AWS 서비스 접근
5. 시간 만료 시 자동 갱신 또는 재요청
```

### 실습: 콘솔에서 역할 전환

#### 1단계: 테스트용 역할 생성
```
역할 설정:
- Role name: TestSwitchRole
- Trusted entity: 현재 AWS 계정
- Permissions: ReadOnlyAccess
- 본인 IAM 사용자가 이 역할로 전환 가능하도록 설정
```

#### 2단계: 역할 전환 실습
```
1. AWS 콘솔 우측 상단 계정명 클릭
2. "Switch Role" 선택
3. 역할 정보 입력:
   - Account: 현재 계정 ID
   - Role: TestSwitchRole
   - Display Name: Test Role (선택사항)
4. "Switch Role" 클릭
5. 역할 전환 확인 (상단에 역할명 표시)
```

#### 3단계: 권한 차이 확인
```
역할 전환 후 테스트:
- IAM 사용자 생성 시도 (실패해야 함)
- EC2 인스턴스 목록 조회 (성공)
- S3 버킷 목록 조회 (성공)
- 원래 역할로 복귀: "Back to [사용자명]" 클릭
```

---

## 🔗 서비스 간 권한 위임 패턴 (30분)

### Lambda 실행 역할 생성

#### 실습: Lambda용 DynamoDB 접근 역할
```
1단계: 역할 생성
- Trusted entity: AWS service → Lambda
- Role name: Lambda-DynamoDB-Role

2단계: 정책 연결
기본 정책:
☑ AWSLambdaBasicExecutionRole (로그 작성)

커스텀 정책:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:GetItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-northeast-2:*:table/saa-study-*"
    }
  ]
}
```

### API Gateway와 Lambda 통합

#### 권한 위임 체인
```
사용자 요청 → API Gateway → Lambda → DynamoDB

각 단계별 권한:
1. API Gateway: Lambda 함수 호출 권한
2. Lambda: DynamoDB 테이블 접근 권한
3. DynamoDB: 데이터 읽기/쓰기 권한
```

---

## 🛡️ 역할 보안 모범 사례 (30분)

### 1. 신뢰 관계 설정

#### 최소 신뢰 원칙
```
✅ 좋은 예:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "ap-northeast-2"
        }
      }
    }
  ]
}

❌ 나쁜 예:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### 2. 조건부 역할 전환

#### 시간 제한 역할
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/developer"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "DateGreaterThan": {
          "aws:CurrentTime": "2024-01-01T00:00:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "2024-12-31T23:59:59Z"
        },
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```

### 3. 역할 모니터링

#### CloudTrail을 통한 추적
```
모니터링 항목:
- AssumeRole 이벤트
- 역할 사용 빈도
- 비정상적 접근 패턴
- 권한 사용 현황

알림 설정:
- 새로운 역할 생성
- 권한 정책 변경
- 비정상적 역할 사용
```

---

## 📝 핵심 정리

### IAM 역할의 핵심 특징
1. **임시성**: 시간 제한이 있는 자격 증명
2. **공유성**: 여러 개체가 동일 역할 사용 가능
3. **조건부**: 특정 조건에서만 역할 전환 가능
4. **추적성**: 모든 역할 사용이 로그에 기록

### 역할 사용 시나리오
- **EC2 → AWS 서비스**: 하드코딩된 키 없이 안전한 접근
- **Lambda → 다른 서비스**: 서버리스 환경에서 권한 관리
- **교차 계정**: 계정 간 안전한 리소스 공유
- **임시 권한**: 특정 작업을 위한 일시적 권한 부여

### 보안 모범 사례
- **최소 신뢰**: 필요한 서비스/계정에만 신뢰 관계 설정
- **조건 활용**: MFA, 시간, IP 등 조건부 접근
- **정기 검토**: 역할 사용 현황 및 권한 정기 점검
- **모니터링**: CloudTrail을 통한 지속적 감시

---

## 🤔 토론 주제

1. **역할 vs 사용자, 언제 무엇을 사용할까?**
   - 각각의 적절한 사용 시나리오

2. **교차 계정 접근 시 보안 고려사항은?**
   - 신뢰 관계 설정 시 주의점

---

## 📚 다음 시간 예고

**교차 계정 액세스**
- 여러 AWS 계정 간 리소스 공유
- 교차 계정 역할 설정 및 전환 실습
- AWS STS(Security Token Service) 심화
- 실제 기업 환경의 다중 계정 전략

---

> 💡 **오늘의 핵심**: IAM 역할은 임시 출입증과 같습니다. 필요할 때만 권한을 빌려주고, 시간이 지나면 자동으로 만료되어 보안을 강화합니다. EC2나 Lambda 같은 AWS 서비스가 다른 서비스에 접근할 때 가장 안전한 방법입니다!