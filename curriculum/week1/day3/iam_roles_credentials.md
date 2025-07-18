# IAM 역할 및 임시 자격 증명

## 개요
AWS Identity and Access Management(IAM) 역할은 AWS 환경에서 권한을 안전하게 위임하는 강력한 메커니즘입니다. 이 강의에서는 IAM 역할의 개념, 생성 및 관리 방법, 역할 전환 및 위임 프로세스, 그리고 AWS Security Token Service(STS)를 통한 임시 자격 증명 발급에 대해 알아봅니다. IAM 역할을 효과적으로 활용하면 장기 자격 증명의 위험을 줄이고 보안을 강화할 수 있습니다.

## 학습 목표
- IAM 역할의 개념과 사용 사례 이해
- 역할 생성 및 관리 방법 습득
- 역할 전환 및 위임 프로세스 이해
- 서비스 역할의 목적과 구성 방법 학습
- AWS STS 및 임시 자격 증명의 작동 방식 이해

## IAM 역할 개요

### IAM 역할이란?
IAM 역할은 특정 권한을 가진 AWS 자격 증명으로, 필요한 엔터티가 맡을 수 있습니다. 역할은 특정 사용자나 서비스에 영구적으로 연결되지 않으며, 필요할 때 맡았다가 사용이 끝나면 반환할 수 있습니다.

**비유**: IAM 역할은 호텔 객실 키카드와 같습니다. 체크인할 때 특정 객실에 대한 액세스 권한이 있는 키카드를 받고, 체크아웃할 때 반납합니다. 키카드는 특정 기간 동안만 유효하며, 여러 사람이 같은 객실에 머물 때 동일한 액세스 권한을 가질 수 있습니다.

### IAM 역할과 IAM 사용자의 차이점
IAM 역할과 IAM 사용자는 모두 AWS 리소스에 액세스할 수 있는 자격 증명이지만, 몇 가지 중요한 차이점이 있습니다:

| 특성 | IAM 역할 | IAM 사용자 |
|------|---------|----------|
| 자격 증명 유형 | 임시 | 장기 |
| 연결 대상 | 여러 엔터티가 맡을 수 있음 | 단일 개인 또는 서비스 |
| 로그인 가능 여부 | 직접 로그인 불가 | 콘솔 로그인 가능 |
| 자격 증명 관리 | AWS STS에서 자동 관리 | 수동 관리 필요 |
| 보안 자격 증명 | 자동 교체 | 수동 교체 필요 |
| 주요 사용 사례 | 권한 위임, 서비스 간 통신 | 개인 사용자, 애플리케이션 |

### IAM 역할의 주요 구성 요소
IAM 역할은 다음과 같은 주요 구성 요소로 이루어져 있습니다:

1. **신뢰 정책(Trust Policy)**: 누가 이 역할을 맡을 수 있는지 정의
   - 역할을 맡을 수 있는 주체(Principal) 지정
   - AWS 계정, IAM 사용자, AWS 서비스, SAML 제공자 등이 주체가 될 수 있음

2. **권한 정책(Permission Policy)**: 역할이 수행할 수 있는 작업 정의
   - 역할이 액세스할 수 있는 AWS 리소스 및 작업 지정
   - 자격 증명 기반 정책과 동일한 형식 사용

3. **세션 기간**: 역할 세션의 최대 지속 시간
   - 기본값: 1시간
   - 최소값: 15분
   - 최대값: 12시간 (AWS 계정 설정에 따라 다를 수 있음)

4. **세션 태그**: 역할 세션에 연결할 수 있는 키-값 쌍
   - 역할을 맡을 때 전달되는 속성
   - 속성 기반 액세스 제어(ABAC)에 사용 가능

### IAM 역할의 주요 이점
IAM 역할을 사용하면 다음과 같은 이점을 얻을 수 있습니다:

1. **향상된 보안**:
   - 장기 자격 증명 불필요
   - 자동 자격 증명 교체
   - 최소 권한 원칙 적용 용이

2. **간소화된 관리**:
   - 중앙 집중식 권한 관리
   - 자격 증명 수명 주기 자동화
   - 권한 위임 간소화

3. **유연한 액세스 제어**:
   - 다양한 엔터티에 동일한 권한 세트 적용
   - 조건부 액세스 구현
   - 임시 권한 부여

4. **비용 효율성**:
   - 사용자별 자격 증명 관리 오버헤드 감소
   - 자격 증명 교체 자동화로 운영 비용 절감
   - 보안 인시던트 위험 및 관련 비용 감소

## 역할 생성 및 관리

### 역할 생성 프로세스
AWS Management Console, AWS CLI 또는 AWS SDK를 사용하여 IAM 역할을 생성할 수 있습니다. 다음은 역할 생성의 일반적인 단계입니다:

1. **신뢰할 수 있는 엔터티 유형 선택**:
   - AWS 계정
   - AWS 서비스
   - 웹 자격 증명
   - SAML 2.0 페더레이션
   - 사용자 지정 신뢰 정책

2. **권한 정책 연결**:
   - AWS 관리형 정책
   - 고객 관리형 정책
   - 인라인 정책

3. **역할 세부 정보 설정**:
   - 역할 이름
   - 설명
   - 세션 기간
   - 태그

### 일반적인 역할 유형

#### AWS 서비스 역할
AWS 서비스가 사용자를 대신하여 작업을 수행할 수 있도록 하는 역할입니다.

**예시**:
- EC2 인스턴스 역할: EC2 인스턴스가 S3 버킷에 액세스
- Lambda 실행 역할: Lambda 함수가 DynamoDB 테이블에 액세스
- CloudFormation 서비스 역할: CloudFormation이 스택 리소스를 생성 및 관리

**신뢰 정책 예시**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### 교차 계정 역할
다른 AWS 계정의 사용자가 현재 계정의 리소스에 액세스할 수 있도록 하는 역할입니다.

**예시**:
- 개발 계정의 사용자가 운영 계정의 리소스에 액세스
- 중앙 보안 팀이 여러 계정의 보안 설정을 감사
- 관리 계정이 멤버 계정의 리소스를 관리

**신뢰 정책 예시**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalOrgID": "o-exampleorgid"
        }
      }
    }
  ]
}
```

#### 자격 증명 제공자 역할
외부 자격 증명 제공자를 통해 인증된 사용자가 AWS 리소스에 액세스할 수 있도록 하는 역할입니다.

**예시**:
- SAML 2.0 기반 페더레이션 (Active Directory, Okta, OneLogin 등)
- 웹 자격 증명 페더레이션 (Google, Facebook, Amazon 등)
- OpenID Connect(OIDC) 제공자

**SAML 신뢰 정책 예시**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:saml-provider/ADFS"
      },
      "Action": "sts:AssumeRoleWithSAML",
      "Condition": {
        "StringEquals": {
          "SAML:aud": "https://signin.aws.amazon.com/saml"
        }
      }
    }
  ]
}
```

**웹 자격 증명 신뢰 정책 예시**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "accounts.google.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "accounts.google.com:aud": "123456789012.apps.googleusercontent.com"
        }
      }
    }
  ]
}
```

### 역할 관리 모범 사례

#### 역할 명명 규칙
일관된 명명 규칙을 사용하면 역할을 쉽게 식별하고 관리할 수 있습니다.

**권장 형식**:
- `<목적>-<권한 수준>-Role`
- `<서비스>-<기능>-Role`
- `<계정 유형>-<사용 사례>-Role`

**예시**:
- `EC2-S3ReadOnly-Role`
- `Lambda-DynamoDBFull-Role`
- `CrossAccount-Audit-Role`
- `SAML-Developer-Role`

#### 역할 권한 경계
권한 경계는 역할이 가질 수 있는 최대 권한을 설정하는 고급 기능입니다.

**주요 특징**:
- 역할에 연결된 정책이 부여할 수 있는 최대 권한 제한
- 권한을 부여하지 않고 가드레일 역할
- 권한 위임 시나리오에서 유용

**사용 사례**:
- 개발자가 자체 역할을 생성할 수 있지만 특정 서비스로 제한
- 특정 리전 또는 리소스로 액세스 제한
- 특정 작업(예: 리소스 삭제) 방지

#### 역할 세션 기간 및 태그
역할 세션 기간과 태그를 적절히 구성하여 보안을 강화하고 감사를 용이하게 할 수 있습니다.

**세션 기간 모범 사례**:
- 필요한 최소 시간으로 설정
- 사용 사례에 따라 다르게 설정 (예: 인간 사용자 vs 자동화)
- 중요한 작업에는 더 짧은 세션 기간 적용

**세션 태그 모범 사례**:
- 감사 및 추적을 위한 태그 사용 (예: `RequestID`, `ProjectName`)
- ABAC 구현을 위한 태그 사용 (예: `Department`, `Environment`)
- 태그 값 검증을 위한 조건 사용

#### 역할 정기 검토 및 정리
미사용 역할과 과도한 권한을 식별하고 제거하여 보안 위험을 줄입니다.

**모범 사례**:
- 정기적인 역할 액세스 검토 수행
- AWS IAM Access Analyzer를 사용하여 외부 액세스 식별
- 마지막 사용 정보를 기반으로 미사용 역할 식별
- 최소 권한 원칙에 따라 과도한 권한 제거

## 역할 전환 및 위임

### 역할 전환 프로세스
역할 전환은 한 자격 증명(사용자, 역할, 서비스)이 다른 역할을 맡는 프로세스입니다.

**역할 전환 단계**:
1. 주체(사용자, 역할, 서비스)가 AWS STS에 역할 수임 요청
2. AWS STS가 주체의 권한 확인 (신뢰 정책 평가)
3. 권한이 있으면 AWS STS가 임시 보안 자격 증명 발급
4. 주체가 임시 자격 증명을 사용하여 AWS 작업 수행
5. 세션 만료 시 자격 증명 자동 무효화

**역할 전환 다이어그램**:
```
사용자/서비스 → STS:AssumeRole 요청 → AWS STS → 임시 자격 증명 발급 → AWS 서비스 액세스
```

### AWS Management Console에서 역할 전환
AWS Management Console에서 역할을 전환하는 방법입니다.

**단계**:
1. AWS Management Console에 로그인
2. 오른쪽 상단의 사용자 이름 드롭다운 메뉴 클릭
3. "역할 전환" 선택
4. 계정 ID, 역할 이름, 표시 이름 입력
5. "역할 전환" 버튼 클릭
6. 역할 권한으로 작업 수행
7. 원래 자격 증명으로 돌아가려면 역할 표시 이름 드롭다운에서 "돌아가기" 선택

**역할 전환 URL 형식**:
```
https://signin.aws.amazon.com/switchrole?account=ACCOUNT_ID&roleName=ROLE_NAME&displayName=DISPLAY_NAME
```

### AWS CLI 및 SDK에서 역할 전환
프로그래밍 방식으로 역할을 전환하는 방법입니다.

**AWS CLI 예시**:
```bash
# 역할 수임하여 임시 자격 증명 얻기
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/S3AccessRole --role-session-name MyRoleSession

# 임시 자격 증명으로 프로필 구성
aws configure set aws_access_key_id ASIA... --profile assumed-role
aws configure set aws_secret_access_key wJalr... --profile assumed-role
aws configure set aws_session_token IQoJb... --profile assumed-role

# 임시 자격 증명으로 명령 실행
aws s3 ls --profile assumed-role
```

**AWS SDK (Python) 예시**:
```python
import boto3

# STS 클라이언트 생성
sts_client = boto3.client('sts')

# 역할 수임
response = sts_client.assume_role(
    RoleArn='arn:aws:iam::123456789012:role/S3AccessRole',
    RoleSessionName='MyRoleSession'
)

# 임시 자격 증명 추출
credentials = response['Credentials']

# 임시 자격 증명으로 S3 클라이언트 생성
s3_client = boto3.client(
    's3',
    aws_access_key_id=credentials['AccessKeyId'],
    aws_secret_access_key=credentials['SecretAccessKey'],
    aws_session_token=credentials['SessionToken']
)

# S3 작업 수행
response = s3_client.list_buckets()
```

### 역할 체인
역할 체인은 하나의 역할을 사용하여 다른 역할을 맡는 프로세스입니다.

**역할 체인 예시**:
1. 사용자가 역할 A를 맡음
2. 역할 A의 자격 증명을 사용하여 역할 B를 맡음
3. 역할 B의 자격 증명을 사용하여 작업 수행

**제한 사항**:
- 최대 체인 깊이: AWS API 호출의 경우 일반적으로 7개 역할
- 각 역할 전환은 이전 자격 증명의 권한 세트를 상속하지 않음
- 각 역할은 독립적인 권한 세트를 가짐

**AWS CLI 역할 체인 예시**:
```bash
# 첫 번째 역할 수임
CREDENTIALS=$(aws sts assume-role --role-arn arn:aws:iam::123456789012:role/RoleA --role-session-name SessionA)

# 환경 변수 설정
export AWS_ACCESS_KEY_ID=$(echo $CREDENTIALS | jq -r .Credentials.AccessKeyId)
export AWS_SECRET_ACCESS_KEY=$(echo $CREDENTIALS | jq -r .Credentials.SecretAccessKey)
export AWS_SESSION_TOKEN=$(echo $CREDENTIALS | jq -r .Credentials.SessionToken)

# 두 번째 역할 수임
CREDENTIALS=$(aws sts assume-role --role-arn arn:aws:iam::123456789012:role/RoleB --role-session-name SessionB)

# 새 자격 증명으로 환경 변수 업데이트
export AWS_ACCESS_KEY_ID=$(echo $CREDENTIALS | jq -r .Credentials.AccessKeyId)
export AWS_SECRET_ACCESS_KEY=$(echo $CREDENTIALS | jq -r .Credentials.SecretAccessKey)
export AWS_SESSION_TOKEN=$(echo $CREDENTIALS | jq -r .Credentials.SessionToken)

# 최종 역할 자격 증명으로 명령 실행
aws s3 ls
```

## 서비스 역할

### 서비스 역할의 목적
서비스 역할은 AWS 서비스가 사용자를 대신하여 작업을 수행할 수 있도록 하는 특수한 유형의 IAM 역할입니다.

**주요 목적**:
- AWS 서비스에 필요한 권한 부여
- 사용자 자격 증명 공유 없이 서비스 작업 수행
- 최소 권한 원칙 적용
- 서비스 간 통신 활성화

### 일반적인 서비스 역할 유형

#### EC2 인스턴스 프로필
EC2 인스턴스에 IAM 역할을 연결하여 인스턴스에서 실행되는 애플리케이션에 AWS 리소스에 대한 액세스 권한을 부여합니다.

**작동 방식**:
1. IAM 역할 생성 및 필요한 권한 정책 연결
2. 역할을 EC2 인스턴스 프로필로 변환
3. EC2 인스턴스 시작 시 인스턴스 프로필 연결
4. EC2 인스턴스 메타데이터 서비스를 통해 임시 자격 증명 제공
5. 인스턴스에서 실행되는 애플리케이션이 자격 증명을 자동으로 사용

**이점**:
- 액세스 키를 EC2 인스턴스에 저장할 필요 없음
- 자격 증명 자동 교체
- 인스턴스 중지/시작 없이 권한 업데이트 가능

**예시 사용 사례**:
- EC2에서 실행되는 웹 서버가 S3 버킷에서 이미지 검색
- EC2에서 실행되는 애플리케이션이 DynamoDB 테이블에 데이터 저장
- EC2에서 실행되는 모니터링 에이전트가 CloudWatch에 지표 게시

#### Lambda 실행 역할
Lambda 함수가 다른 AWS 서비스와 상호 작용하는 데 필요한 권한을 제공합니다.

**작동 방식**:
1. IAM 역할 생성 및 필요한 권한 정책 연결
2. Lambda 함수 생성 또는 업데이트 시 실행 역할 지정
3. Lambda 서비스가 함수 실행 시 역할 수임
4. 함수 코드가 AWS SDK를 통해 다른 서비스에 액세스

**일반적인 권한**:
- 기본 Lambda 실행 권한 (CloudWatch Logs 액세스)
- 이벤트 소스에 따른 권한 (S3, DynamoDB, SQS 등)
- 함수가 상호 작용하는 다른 AWS 서비스에 대한 권한

**예시 사용 사례**:
- S3 버킷에 업로드된 이미지 처리
- DynamoDB 스트림의 데이터 변경 처리
- API Gateway 요청에 대한 백엔드 로직 실행

#### 기타 일반적인 서비스 역할
다양한 AWS 서비스가 특정 작업을 수행하기 위해 서비스 역할을 사용합니다.

**CloudFormation 서비스 역할**:
- CloudFormation이 스택 리소스를 생성, 업데이트, 삭제하는 데 사용
- 템플릿에 정의된 모든 리소스에 대한 권한 필요

**AWS Config 서비스 역할**:
- AWS Config가 리소스 구성을 기록하고 평가하는 데 사용
- 리소스 구성 읽기 및 S3 버킷에 구성 기록 저장 권한 필요

**Amazon EMR 서비스 역할**:
- EMR 클러스터가 다른 AWS 서비스와 상호 작용하는 데 사용
- EC2 인스턴스 프로비저닝, S3 데이터 액세스, CloudWatch 로깅 권한 필요

### 서비스 연결 역할
서비스 연결 역할은 AWS 서비스에 직접 연결된 고유한 유형의 IAM 역할입니다.

**주요 특징**:
- AWS 서비스에 의해 사전 정의됨
- 서비스에 필요한 모든 권한 포함
- 서비스에 의해 자동 생성 가능
- 사용자가 직접 권한 수정 불가
- 서비스에서만 삭제 가능

**서비스 연결 역할을 지원하는 일반적인 서비스**:
- Amazon RDS
- AWS Auto Scaling
- AWS Organizations
- Amazon GuardDuty
- AWS Security Hub

**예시 (Auto Scaling 서비스 연결 역할)**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "ec2:TerminateInstances",
        "ec2:DescribeLaunchTemplates",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    }
  ]
}
```

## AWS STS 및 임시 자격 증명

### AWS Security Token Service(STS) 개요
AWS STS는 AWS 리소스에 대한 제한된 권한을 가진 임시 자격 증명을 생성하는 웹 서비스입니다.

**주요 기능**:
- 임시 보안 자격 증명 발급
- 제한된 권한 및 수명을 가진 자격 증명 제공
- 다양한 자격 증명 시나리오 지원
- AWS 서비스 및 리소스에 대한 안전한 액세스 제공

**임시 자격 증명 구성 요소**:
- 액세스 키 ID
- 비밀 액세스 키
- 세션 토큰
- 만료 시간

### 주요 AWS STS API 작업

#### AssumeRole
현재 자격 증명을 사용하여 IAM 역할을 맡고 임시 자격 증명을 얻습니다.

**주요 파라미터**:
- `RoleArn`: 맡을 역할의 ARN
- `RoleSessionName`: 감사 및 로깅을 위한 세션 식별자
- `DurationSeconds`: 세션 지속 시간 (기본값: 1시간)
- `Policy`: 역할 권한을 추가로 제한하는 인라인 정책 (선택 사항)
- `Tags`: 세션에 연결할 태그 (선택 사항)

**사용 사례**:
- 교차 계정 액세스
- 권한 제한
- 임시 액세스 제공

**예시**:
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/S3AccessRole \
  --role-session-name MyRoleSession \
  --duration-seconds 3600
```

#### AssumeRoleWithSAML
SAML 2.0 기반 페더레이션을 통해 역할을 맡고 임시 자격 증명을 얻습니다.

**주요 파라미터**:
- `RoleArn`: 맡을 역할의 ARN
- `PrincipalArn`: SAML 제공자의 ARN
- `SAMLAssertion`: 자격 증명 제공자의 SAML 어설션
- `DurationSeconds`: 세션 지속 시간

**사용 사례**:
- 기업 디렉터리(Active Directory) 통합
- Single Sign-On(SSO) 구현
- 타사 자격 증명 제공자 통합

**작동 방식**:
1. 사용자가 기업 포털에 로그인
2. 자격 증명 제공자가 SAML 어설션 생성
3. 브라우저가 SAML 어설션을 AWS SAML 엔드포인트로 전송
4. AWS STS가 SAML 어설션을 검증하고 임시 자격 증명 발급
5. 사용자가 AWS Management Console에 액세스하거나 API 호출 수행

#### AssumeRoleWithWebIdentity
웹 자격 증명 제공자를 통해 역할을 맡고 임시 자격 증명을 얻습니다.

**주요 파라미터**:
- `RoleArn`: 맡을 역할의 ARN
- `WebIdentityToken`: 자격 증명 제공자의 토큰
- `ProviderId`: 자격 증명 제공자 식별자 (선택 사항)
- `DurationSeconds`: 세션 지속 시간

**사용 사례**:
- 모바일 애플리케이션 인증
- 소셜 자격 증명 제공자 통합 (Google, Facebook, Amazon)
- 웹 애플리케이션 인증

**작동 방식**:
1. 사용자가 웹 자격 증명 제공자에 로그인
2. 제공자가 ID 토큰 발급
3. 애플리케이션이 ID 토큰으로 AWS STS API 호출
4. AWS STS가 토큰을 검증하고 임시 자격 증명 발급
5. 애플리케이션이 임시 자격 증명으로 AWS 서비스 액세스

#### GetSessionToken
MFA 인증을 위한 임시 자격 증명을 얻습니다.

**주요 파라미터**:
- `DurationSeconds`: 세션 지속 시간
- `SerialNumber`: MFA 디바이스 일련 번호
- `TokenCode`: MFA 디바이스에서 생성된 코드

**사용 사례**:
- MFA 보호 API 액세스
- 루트 사용자 또는 IAM 사용자의 임시 자격 증명 생성
- 민감한 작업에 대한 추가 보안 계층 제공

**예시**:
```bash
aws sts get-session-token \
  --duration-seconds 3600 \
  --serial-number arn:aws:iam::123456789012:mfa/user \
  --token-code 123456
```

### 임시 자격 증명 사용 모범 사례

#### 세션 기간 최적화
세션 기간을 사용 사례에 맞게 최적화하여 보안과 사용자 경험 사이의 균형을 맞춥니다.

**모범 사례**:
- 인간 사용자: 4-8시간 (작업 세션 기간에 맞춤)
- 자동화 프로세스: 15분-1시간 (작업 완료에 필요한 최소 시간)
- 중요한 작업: 최소 기간 (15분)
- 장기 실행 프로세스: 자격 증명 갱신 메커니즘 구현

#### 외부 ID 사용
교차 계정 시나리오에서 외부 ID를 사용하여 "혼동된 대리자" 문제를 방지합니다.

**외부 ID란?**:
- 제3자와 고객 간에 합의된 고유 식별자
- 역할 수임 요청에 포함되어야 하는 추가 조건
- 신뢰 정책에서 확인되는 비밀 값

**신뢰 정책 예시**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-id-assigned-by-third-party"
        }
      }
    }
  ]
}
```

**AWS CLI 예시**:
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/S3AccessRole \
  --role-session-name MyRoleSession \
  --external-id unique-id-assigned-by-third-party
```

#### 세션 정책 사용
세션 정책을 사용하여 역할이 부여하는 권한을 추가로 제한합니다.

**세션 정책이란?**:
- 역할 수임 시 전달되는 인라인 IAM 정책
- 역할 권한을 추가로 제한 (확장 불가)
- 임시 세션에만 적용

**사용 사례**:
- 특정 리소스로 액세스 제한
- 특정 작업만 허용
- 특정 조건에서만 액세스 허용

**AWS CLI 예시**:
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/S3AccessRole \
  --role-session-name MyRoleSession \
  --policy '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"s3:*","Resource":"arn:aws:s3:::example-bucket/*"}]}'
```

#### 자격 증명 캐싱 및 갱신
애플리케이션에서 임시 자격 증명을 효율적으로 관리하는 방법입니다.

**모범 사례**:
- 자격 증명 캐싱으로 불필요한 STS 호출 감소
- 만료 전에 자격 증명 갱신
- AWS SDK 자격 증명 공급자 체인 활용
- 오류 처리 및 재시도 로직 구현

**AWS SDK 자격 증명 공급자 예시 (Python)**:
```python
import boto3
from botocore.credentials import RefreshableCredentials
from botocore.session import get_session

def get_refreshable_credentials():
    def refresh_credentials():
        # STS 클라이언트 생성
        sts_client = boto3.client('sts')
        
        # 역할 수임
        response = sts_client.assume_role(
            RoleArn='arn:aws:iam::123456789012:role/S3AccessRole',
            RoleSessionName='RefreshableSession'
        )
        
        # 자격 증명 반환
        credentials = response['Credentials']
        return {
            'access_key': credentials['AccessKeyId'],
            'secret_key': credentials['SecretAccessKey'],
            'token': credentials['SessionToken'],
            'expiry_time': credentials['Expiration'].isoformat()
        }
    
    # 갱신 가능한 자격 증명 생성
    refreshable_credentials = RefreshableCredentials.create_from_metadata(
        metadata=refresh_credentials(),
        refresh_using=refresh_credentials,
        method='sts-assume-role'
    )
    
    # 세션 생성
    session = get_session()
    session._credentials = refreshable_credentials
    
    # 갱신 가능한 자격 증명으로 클라이언트 생성
    s3_client = session.create_client('s3')
    return s3_client
```

## IAM 역할 모니터링 및 감사

### IAM 자격 증명 보고서
계정의 모든 사용자와 자격 증명 상태에 대한 보고서를 생성합니다.

**포함 정보**:
- 사용자 생성 시간
- 암호 상태 및 마지막 사용 시간
- MFA 상태
- 액세스 키 상태 및 마지막 사용 시간
- 인증서 상태

**생성 방법**:
```bash
# AWS CLI를 사용하여 보고서 생성
aws iam generate-credential-report

# 보고서 다운로드
aws iam get-credential-report
```

### IAM 액세스 분석기
리소스에 대한 외부 액세스를 식별하는 도구입니다.

**주요 기능**:
- 외부 엔터티에 액세스 권한을 부여하는 정책 식별
- 정책 검증 및 권장 사항 제공
- 새로운 정책 변경 사항 지속적 모니터링
- 규정 준수 감사 지원

**사용 방법**:
1. AWS Management Console에서 IAM 액세스 분석기 활성화
2. 분석기 유형 및 신뢰 영역 선택
3. 결과 검토 및 조치

### CloudTrail을 통한 IAM 활동 로깅
AWS CloudTrail을 사용하여 IAM 역할 사용 및 권한 변경을 모니터링합니다.

**로깅되는 주요 이벤트**:
- 역할 생성, 수정, 삭제
- 역할 수임 작업
- 정책 변경
- 권한 사용

**CloudTrail 로그 예시 (AssumeRole)**:
```json
{
  "eventVersion": "1.05",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAEXAMPLE",
    "arn": "arn:aws:iam::123456789012:user/Alice",
    "accountId": "123456789012",
    "userName": "Alice"
  },
  "eventTime": "2023-07-18T15:30:00Z",
  "eventSource": "sts.amazonaws.com",
  "eventName": "AssumeRole",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "192.0.2.1",
  "userAgent": "aws-cli/2.0.0",
  "requestParameters": {
    "roleArn": "arn:aws:iam::123456789012:role/S3AccessRole",
    "roleSessionName": "AliceSession"
  },
  "responseElements": {
    "credentials": {
      "accessKeyId": "ASIAEXAMPLE",
      "sessionToken": "TOKEN",
      "expiration": "2023-07-18T16:30:00Z"
    },
    "assumedRoleUser": {
      "assumedRoleId": "AROAEXAMPLE:AliceSession",
      "arn": "arn:aws:sts::123456789012:assumed-role/S3AccessRole/AliceSession"
    }
  },
  "requestID": "request-id",
  "eventID": "event-id",
  "eventType": "AwsApiCall",
  "recipientAccountId": "123456789012"
}
```

### IAM 보안 평가
IAM 구성 및 사용을 정기적으로 평가하여 보안 위험을 식별하고 해결합니다.

**평가 영역**:
- 미사용 역할 및 권한
- 과도한 권한
- 정책 복잡성 및 중복
- 보안 모범 사례 준수

**평가 도구**:
- AWS Trusted Advisor
- IAM Access Analyzer
- AWS Config
- AWS Security Hub

**자동화된 평가 예시 (AWS Config 규칙)**:
- `iam-role-managed-policy-check`: 역할에 연결된 관리형 정책 확인
- `iam-policy-no-statements-with-admin-access`: 관리자 액세스 권한이 있는 정책 식별
- `iam-policy-no-statements-with-full-access`: 전체 액세스 권한이 있는 정책 식별
- `iam-root-access-key-check`: 루트 사용자 액세스 키 확인

## 결론
IAM 역할과 임시 자격 증명은 AWS 환경에서 보안 액세스 관리의 핵심 구성 요소입니다. 역할을 사용하면 장기 자격 증명의 위험을 줄이고, 최소 권한 원칙을 적용하며, 다양한 엔터티 간에 권한을 안전하게 위임할 수 있습니다. AWS STS를 통한 임시 자격 증명은 보안을 강화하고 자격 증명 관리를 간소화합니다. 역할 생성 및 관리, 역할 전환 및 위임, 서비스 역할 구성, 그리고 임시 자격 증명 사용에 대한 모범 사례를 적용함으로써 AWS 환경의 보안 태세를 크게 향상시킬 수 있습니다.

## 참고 자료
- [IAM 역할 사용 설명서](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [AWS STS 사용 설명서](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)
- [IAM 모범 사례](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS 보안 블로그 - IAM 역할 관련 게시물](https://aws.amazon.com/blogs/security/tag/iam-roles/)
- [AWS 임시 보안 자격 증명](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)