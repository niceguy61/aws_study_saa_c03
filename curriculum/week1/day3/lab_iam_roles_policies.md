# 실습: IAM 역할 및 정책 구성

## 개요
이 실습에서는 AWS Identity and Access Management(IAM)의 역할과 정책을 구성하는 방법을 학습합니다. IAM 역할을 생성하고, 다양한 권한 수준의 정책을 연결하며, 역할 전환 및 임시 자격 증명 사용 방법을 실습합니다. 이를 통해 AWS 환경에서 안전하게 권한을 위임하고 최소 권한 원칙을 적용하는 방법을 이해할 수 있습니다.

## 학습 목표
- IAM 역할 생성 및 구성 방법 습득
- 다양한 권한 수준의 IAM 정책 생성 및 테스트
- 역할 전환 및 임시 자격 증명 사용 방법 이해
- 서비스 역할 구성 및 활용 방법 학습
- IAM 정책 시뮬레이터를 사용한 권한 테스트 방법 습득

## 사전 준비 사항
- AWS 계정 액세스
- 관리자 권한이 있는 IAM 사용자
- AWS Management Console 액세스
- AWS CLI 설치 (선택 사항)

## 실습 1: 기본 IAM 역할 생성 및 구성

### 1.1 AWS 서비스를 위한 역할 생성
1. AWS Management Console에 로그인합니다.
2. IAM 서비스로 이동합니다.
3. 왼쪽 탐색 창에서 "역할"을 선택합니다.
4. "역할 생성" 버튼을 클릭합니다.
5. 신뢰할 수 있는 엔터티 유형으로 "AWS 서비스"를 선택합니다.
6. 사용 사례로 "EC2"를 선택합니다.
7. "다음: 권한"을 클릭합니다.
8. "AmazonS3ReadOnlyAccess" 정책을 검색하여 선택합니다.
9. "다음: 태그"를 클릭합니다.
10. (선택 사항) 태그를 추가합니다. 예: 키 = "Purpose", 값 = "Lab"
11. "다음: 검토"를 클릭합니다.
12. 역할 이름을 "EC2-S3ReadOnly-Role"로 입력하고 설명을 추가합니다.
13. "역할 생성"을 클릭합니다.

### 1.2 역할 세부 정보 검토
1. 생성된 역할 목록에서 "EC2-S3ReadOnly-Role"을 찾아 클릭합니다.
2. "신뢰 관계" 탭을 클릭하여 신뢰 정책을 검토합니다.
3. 신뢰 정책이 다음과 유사한지 확인합니다:
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
4. "권한" 탭을 클릭하여 연결된 권한 정책을 검토합니다.
5. "AmazonS3ReadOnlyAccess" 정책이 연결되어 있는지 확인합니다.

### 1.3 EC2 인스턴스 프로필 생성
1. 역할 세부 정보 페이지에서 상단의 "인스턴스 프로필에 추가" 버튼을 클릭합니다.
   - 참고: 역할을 생성할 때 EC2 사용 사례를 선택한 경우, 인스턴스 프로필이 자동으로 생성됩니다.
2. 인스턴스 프로필 이름을 확인합니다.
3. "인스턴스 프로필에 추가"를 클릭합니다.

## 실습 2: 사용자 지정 IAM 정책 생성

### 2.1 인라인 정책 생성
1. IAM 콘솔에서 "역할"로 이동합니다.
2. 이전에 생성한 "EC2-S3ReadOnly-Role"을 선택합니다.
3. "권한" 탭에서 "인라인 정책 추가" 버튼을 클릭합니다.
4. "JSON" 탭을 선택합니다.
5. 다음 정책을 입력합니다:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:PutMetricData",
        "cloudwatch:GetMetricData",
        "cloudwatch:ListMetrics"
      ],
      "Resource": "*"
    }
  ]
}
```
6. "정책 검토"를 클릭합니다.
7. 정책 이름을 "CloudWatchMetricsAccess"로 입력합니다.
8. "정책 생성"을 클릭합니다.

### 2.2 고객 관리형 정책 생성
1. IAM 콘솔의 왼쪽 탐색 창에서 "정책"을 선택합니다.
2. "정책 생성" 버튼을 클릭합니다.
3. "JSON" 탭을 선택합니다.
4. 다음 정책을 입력합니다:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:BatchGetItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/Lab-*"
    }
  ]
}
```
5. "다음: 태그"를 클릭합니다.
6. (선택 사항) 태그를 추가합니다.
7. "다음: 검토"를 클릭합니다.
8. 정책 이름을 "DynamoDBReadOnlyCustom"으로 입력하고 설명을 추가합니다.
9. "정책 생성"을 클릭합니다.

### 2.3 생성한 정책을 역할에 연결
1. IAM 콘솔의 왼쪽 탐색 창에서 "역할"을 선택합니다.
2. "EC2-S3ReadOnly-Role"을 선택합니다.
3. "권한" 탭에서 "권한 추가" > "정책 연결"을 클릭합니다.
4. 검색 상자에 "DynamoDBReadOnlyCustom"을 입력합니다.
5. 정책을 선택하고 "권한 추가"를 클릭합니다.

## 실습 3: 교차 계정 역할 생성 및 구성

### 3.1 교차 계정 역할 생성
1. IAM 콘솔에서 "역할"로 이동합니다.
2. "역할 생성" 버튼을 클릭합니다.
3. 신뢰할 수 있는 엔터티 유형으로 "AWS 계정"을 선택합니다.
4. "다른 AWS 계정"을 선택합니다.
5. 액세스를 제공할 계정 ID를 입력합니다.
   - 참고: 실습 환경에서는 동일한 계정 ID를 입력하거나, 강사가 제공한 계정 ID를 사용합니다.
6. (선택 사항) "MFA 필요" 옵션을 선택합니다.
7. "다음: 권한"을 클릭합니다.
8. "ReadOnlyAccess" 정책을 검색하여 선택합니다.
9. "다음: 태그"를 클릭합니다.
10. (선택 사항) 태그를 추가합니다.
11. "다음: 검토"를 클릭합니다.
12. 역할 이름을 "CrossAccount-ReadOnly-Role"로 입력하고 설명을 추가합니다.
13. "역할 생성"을 클릭합니다.

### 3.2 교차 계정 역할 신뢰 정책 수정
1. 생성된 역할 목록에서 "CrossAccount-ReadOnly-Role"을 찾아 클릭합니다.
2. "신뢰 관계" 탭을 클릭합니다.
3. "신뢰 정책 편집" 버튼을 클릭합니다.
4. 다음과 같이 정책을 수정하여 특정 사용자만 역할을 수임할 수 있도록 합니다:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:user/USERNAME"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```
5. ACCOUNT_ID와 USERNAME을 실제 값으로 바꿉니다.
6. "정책 업데이트"를 클릭합니다.

### 3.3 역할 전환 권한 구성
1. IAM 콘솔의 왼쪽 탐색 창에서 "사용자"를 선택합니다.
2. 역할을 수임할 사용자를 선택합니다.
3. "권한" 탭에서 "인라인 정책 추가"를 클릭합니다.
4. "JSON" 탭을 선택합니다.
5. 다음 정책을 입력합니다:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::ACCOUNT_ID:role/CrossAccount-ReadOnly-Role"
    }
  ]
}
```
6. ACCOUNT_ID를 실제 계정 ID로 바꿉니다.
7. "정책 검토"를 클릭합니다.
8. 정책 이름을 "AssumeRolePolicy"로 입력합니다.
9. "정책 생성"을 클릭합니다.

## 실습 4: 역할 전환 및 임시 자격 증명 사용

### 4.1 AWS Management Console에서 역할 전환
1. AWS Management Console에 로그인합니다.
2. 오른쪽 상단의 계정 이름/번호를 클릭합니다.
3. 드롭다운 메뉴에서 "역할 전환"을 선택합니다.
4. 다음 정보를 입력합니다:
   - 계정: 역할이 있는 계정 ID
   - 역할: CrossAccount-ReadOnly-Role
   - 표시 이름: 원하는 표시 이름 (예: "ReadOnly")
   - 표시 색상: 원하는 색상 선택
5. "역할 전환" 버튼을 클릭합니다.
6. 역할이 성공적으로 전환되면 콘솔 상단에 표시 이름과 색상이 나타납니다.
7. 다양한 서비스에 액세스하여 읽기 전용 권한을 테스트합니다.
8. 원래 사용자로 돌아가려면 현재 역할 이름을 클릭하고 "돌아가기"를 선택합니다.

### 4.2 AWS CLI를 사용한 역할 전환 (선택 사항)
1. AWS CLI가 설치되어 있고 기본 자격 증명이 구성되어 있는지 확인합니다.
2. 다음 명령을 실행하여 역할을 수임합니다:
```bash
aws sts assume-role --role-arn "arn:aws:iam::ACCOUNT_ID:role/CrossAccount-ReadOnly-Role" --role-session-name "TestSession"
```
3. 반환된 임시 자격 증명(AccessKeyId, SecretAccessKey, SessionToken)을 기록합니다.
4. 다음과 같이 환경 변수를 설정하여 임시 자격 증명을 사용합니다:
```bash
# Linux/macOS
export AWS_ACCESS_KEY_ID=<AccessKeyId>
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
export AWS_SESSION_TOKEN=<SessionToken>

# Windows (CMD)
set AWS_ACCESS_KEY_ID=<AccessKeyId>
set AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
set AWS_SESSION_TOKEN=<SessionToken>

# Windows (PowerShell)
$env:AWS_ACCESS_KEY_ID="<AccessKeyId>"
$env:AWS_SECRET_ACCESS_KEY="<SecretAccessKey>"
$env:AWS_SESSION_TOKEN="<SessionToken>"
```
5. 임시 자격 증명으로 AWS CLI 명령을 실행합니다:
```bash
aws s3 ls
```
6. 작업이 완료되면 환경 변수를 지웁니다:
```bash
# Linux/macOS
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN

# Windows (CMD)
set AWS_ACCESS_KEY_ID=
set AWS_SECRET_ACCESS_KEY=
set AWS_SESSION_TOKEN=

# Windows (PowerShell)
Remove-Item Env:\AWS_ACCESS_KEY_ID
Remove-Item Env:\AWS_SECRET_ACCESS_KEY
Remove-Item Env:\AWS_SESSION_TOKEN
```

## 실습 5: 서비스 역할 생성 및 사용

### 5.1 Lambda 함수용 서비스 역할 생성
1. IAM 콘솔에서 "역할"로 이동합니다.
2. "역할 생성" 버튼을 클릭합니다.
3. 신뢰할 수 있는 엔터티 유형으로 "AWS 서비스"를 선택합니다.
4. 사용 사례로 "Lambda"를 선택합니다.
5. "다음: 권한"을 클릭합니다.
6. 다음 정책을 검색하여 선택합니다:
   - "AWSLambdaBasicExecutionRole"
   - "AmazonS3ReadOnlyAccess"
7. "다음: 태그"를 클릭합니다.
8. (선택 사항) 태그를 추가합니다.
9. "다음: 검토"를 클릭합니다.
10. 역할 이름을 "Lambda-S3ReadOnly-Role"로 입력하고 설명을 추가합니다.
11. "역할 생성"을 클릭합니다.

### 5.2 Lambda 함수 생성 및 역할 연결
1. AWS Management Console에서 Lambda 서비스로 이동합니다.
2. "함수 생성" 버튼을 클릭합니다.
3. "새로 작성"을 선택합니다.
4. 함수 이름을 "S3ListFunction"으로 입력합니다.
5. 런타임으로 "Python 3.9"를 선택합니다.
6. "기존 역할 사용"을 선택하고 드롭다운 메뉴에서 "Lambda-S3ReadOnly-Role"을 선택합니다.
7. "함수 생성" 버튼을 클릭합니다.
8. 함수 코드 편집기에서 다음 코드를 입력합니다:
```python
import boto3
import json

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    # S3 버킷 목록 가져오기
    response = s3.list_buckets()
    
    # 버킷 이름 추출
    buckets = [bucket['Name'] for bucket in response['Buckets']]
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'S3 버킷 목록을 성공적으로 가져왔습니다.',
            'buckets': buckets
        })
    }
```
9. "배포" 버튼을 클릭합니다.
10. "테스트" 탭을 클릭합니다.
11. 새 테스트 이벤트를 구성합니다:
    - 이벤트 이름: "TestEvent"
    - 이벤트 본문: 기본값 유지
12. "저장" 버튼을 클릭합니다.
13. "테스트" 버튼을 클릭하여 함수를 실행합니다.
14. 실행 결과를 확인하고 S3 버킷 목록이 반환되는지 확인합니다.

## 실습 6: IAM 정책 시뮬레이터 사용

### 6.1 IAM 정책 시뮬레이터 액세스
1. AWS Management Console에서 IAM 서비스로 이동합니다.
2. 왼쪽 탐색 창에서 "액세스 관리" > "정책"을 선택합니다.
3. "정책 시뮬레이터" 버튼을 클릭합니다.
   - 참고: 또는 https://policysim.aws.amazon.com 으로 직접 접속할 수 있습니다.

### 6.2 사용자 권한 시뮬레이션
1. "사용자, 그룹 및 역할" 드롭다운에서 "역할"을 선택합니다.
2. 이전에 생성한 "EC2-S3ReadOnly-Role"을 선택합니다.
3. "시뮬레이션할 서비스 선택" 드롭다운에서 "S3"를 선택합니다.
4. 다음 작업을 선택합니다:
   - ListBucket
   - GetObject
   - PutObject
5. "시뮬레이션 실행" 버튼을 클릭합니다.
6. 결과를 검토합니다:
   - ListBucket, GetObject: 허용됨
   - PutObject: 거부됨
7. 다른 서비스와 작업에 대해서도 시뮬레이션을 반복합니다.

### 6.3 정책 수정 및 재시뮬레이션
1. "새 정책 추가" 버튼을 클릭합니다.
2. "JSON" 탭을 선택합니다.
3. 다음 정책을 입력합니다:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```
4. "정책 추가" 버튼을 클릭합니다.
5. "시뮬레이션 실행" 버튼을 클릭합니다.
6. 결과를 다시 검토합니다:
   - ListBucket, GetObject: 허용됨
   - PutObject: 특정 버킷에 대해 허용됨
7. 다양한 정책 조합을 시뮬레이션하여 권한 효과를 이해합니다.

## 실습 결과물
이 실습을 완료하면 다음과 같은 결과물을 얻게 됩니다:
1. EC2 인스턴스용 IAM 역할 및 인스턴스 프로필
2. 사용자 지정 IAM 정책 (인라인 및 고객 관리형)
3. 교차 계정 액세스를 위한 IAM 역할
4. Lambda 함수용 서비스 역할
5. IAM 정책 시뮬레이터를 사용한 권한 테스트 경험

## 정리 (선택 사항)
실습이 완료되면 생성한 리소스를 정리할 수 있습니다:
1. Lambda 함수 삭제
2. IAM 역할 삭제 (연결된 정책 먼저 분리)
3. 고객 관리형 정책 삭제

## 추가 학습 자료
- [IAM 역할 사용 설명서](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [IAM 정책 참조](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- [AWS STS 사용 설명서](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)
- [IAM 모범 사례](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM 정책 시뮬레이터 사용 설명서](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)