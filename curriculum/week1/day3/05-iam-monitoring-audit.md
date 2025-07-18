# Day 3-5: IAM 모니터링 및 감사

## 📚 학습 목표
- CloudTrail을 통한 API 호출 추적 방법 습득
- IAM 액세스 분석기 활용 실습
- 권한 경계(Permission Boundary) 설정 및 활용
- 정기적 권한 검토 및 정리 프로세스 구축

---

## 📊 CloudTrail을 통한 IAM 활동 추적 (30분)

### CloudTrail = 보안 카메라

#### CloudTrail의 역할
```
기능:
- 모든 AWS API 호출 기록
- 누가, 언제, 어디서, 무엇을 했는지 추적
- 보안 감사 및 규정 준수 지원

비유:
- 건물의 CCTV 시스템
- 모든 출입 기록 저장
- 사고 발생 시 추적 가능
- 예방 효과 및 사후 분석
```

### IAM 관련 주요 이벤트

#### 1. 사용자 관리 이벤트
```
추적 대상:
- CreateUser: 새 사용자 생성
- DeleteUser: 사용자 삭제
- AttachUserPolicy: 사용자에게 정책 연결
- DetachUserPolicy: 사용자에서 정책 분리
- CreateAccessKey: 액세스 키 생성
- DeleteAccessKey: 액세스 키 삭제

중요도: 높음
이유: 권한 변경이 보안에 직접적 영향
```

#### 2. 역할 관리 이벤트
```
추적 대상:
- CreateRole: 새 역할 생성
- DeleteRole: 역할 삭제
- AssumeRole: 역할 전환
- AttachRolePolicy: 역할에 정책 연결
- UpdateAssumeRolePolicy: 신뢰 정책 변경

특별 주의:
- AssumeRole: 가장 빈번한 이벤트
- 비정상적 패턴 탐지 중요
```

#### 3. 정책 관리 이벤트
```
추적 대상:
- CreatePolicy: 새 정책 생성
- DeletePolicy: 정책 삭제
- CreatePolicyVersion: 정책 버전 생성
- SetDefaultPolicyVersion: 기본 버전 변경

분석 포인트:
- 정책 변경 빈도
- 권한 확대 패턴
- 승인되지 않은 변경
```

---

## 🔍 실습: CloudTrail 로그 분석 (45분)

### 1단계: CloudTrail 이벤트 조회

#### 실습: 최근 IAM 활동 확인
```
1. CloudTrail 콘솔 접속
2. Event history 메뉴 선택
3. 필터 설정:
   - Event name: CreateUser
   - Time range: Last 24 hours
   - User name: (특정 사용자)

4. 결과 분석:
   - 누가 사용자를 생성했는가?
   - 언제 생성했는가?
   - 어떤 IP에서 접근했는가?
   - 어떤 권한을 부여했는가?
```

### 2단계: 의심스러운 활동 탐지

#### 실습: 비정상적 패턴 찾기
```
탐지 시나리오:
1. 업무 시간 외 관리자 권한 사용
2. 새로운 지역에서의 접근
3. 대량의 권한 변경
4. 실패한 인증 시도 급증

CloudTrail 쿼리:
- Event name: AssumeRole
- Time: 18:00 - 09:00 (업무 시간 외)
- Source IP: 회사 IP 대역 외부
- Error code: AccessDenied (실패한 시도)
```

### 3단계: 자동화된 모니터링 설정

#### 실습: CloudWatch 알람 생성
```
1단계: 메트릭 필터 생성
- CloudWatch → Logs → Log groups
- CloudTrail 로그 그룹 선택
- "Create metric filter"

필터 패턴:
{ ($.eventName = CreateUser) || ($.eventName = DeleteUser) || ($.eventName = AttachUserPolicy) }

2단계: 알람 설정
- Metric name: IAMUserChanges
- Threshold: > 0 (변경 발생 시)
- Action: SNS 토픽으로 이메일 알림

3단계: 테스트
- 테스트용 사용자 생성
- 알람 발생 확인
- 이메일 수신 확인
```

---

## 🔬 IAM 액세스 분석기 활용 (30분)

### Access Analyzer = 권한 탐정

#### 기능 개요
```
분석 대상:
- 과도한 권한 식별
- 사용하지 않는 권한 탐지
- 외부 접근 가능한 리소스 발견
- 정책 최적화 제안

작동 원리:
- 실제 API 호출 로그 분석
- 부여된 권한 vs 사용된 권한 비교
- 머신러닝 기반 패턴 분석
```

### 실습: Access Analyzer 설정 및 활용

#### 1단계: Access Analyzer 활성화
```
1. IAM 콘솔 → Access analyzer
2. "Create analyzer" 클릭
3. Analyzer name: "OrganizationAnalyzer"
4. Zone of trust: "Current account" 또는 "Organization"
5. "Create analyzer" 클릭
```

#### 2단계: 분석 결과 검토
```
분석 항목:
1. External access findings
   - 외부에서 접근 가능한 S3 버킷
   - 공개된 IAM 역할
   - 교차 계정 접근 권한

2. Unused access findings  
   - 90일 이상 사용하지 않은 권한
   - 불필요한 정책 연결
   - 과도한 권한 부여

3. Policy validation
   - 정책 문법 오류
   - 보안 모범 사례 위반
   - 권장 개선사항
```

#### 3단계: 권한 최적화 실습
```
시나리오: 개발자 사용자의 권한 최적화

1단계: 현재 권한 분석
- 부여된 정책: PowerUserAccess
- 실제 사용: EC2, S3, CloudWatch만 사용

2단계: 최적화된 정책 생성
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:StartInstances",
        "ec2:StopInstances",
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket",
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:ListMetrics"
      ],
      "Resource": "*"
    }
  ]
}

3단계: 정책 적용 및 테스트
- 기존 PowerUserAccess 정책 분리
- 새 최적화 정책 연결
- 기능 정상 동작 확인
```

---

## 🚧 권한 경계(Permission Boundary) 실습 (30분)

### Permission Boundary = 안전 울타리

#### 개념 이해
```
목적:
- 사용자/역할의 최대 권한 제한
- 권한 상승 공격 방지
- 위임된 관리자의 권한 제한

작동 방식:
실제 권한 = IAM 정책 ∩ Permission Boundary

예시:
- IAM 정책: S3, EC2, RDS 권한
- Permission Boundary: S3, EC2만 허용
- 결과: S3, EC2만 실제 사용 가능
```

### 실습: 위임된 관리자 권한 제한

#### 시나리오
```
요구사항:
- 팀 리더에게 팀원 관리 권한 위임
- 단, 관리자 권한은 부여할 수 없음
- 특정 서비스만 접근 가능하도록 제한
```

#### 1단계: Permission Boundary 정책 생성
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:CreateUser",
        "iam:DeleteUser",
        "iam:AttachUserPolicy",
        "iam:DetachUserPolicy",
        "iam:ListUsers",
        "iam:GetUser",
        "ec2:*",
        "s3:*",
        "cloudwatch:*"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": [
        "iam:CreateRole",
        "iam:DeleteRole",
        "iam:AttachRolePolicy",
        "iam:CreatePolicy"
      ],
      "Resource": "*"
    }
  ]
}
```

#### 2단계: 팀 리더 사용자에게 적용
```
1. IAM → Users → 팀 리더 사용자 선택
2. Permissions boundary → "Set boundary"
3. 위에서 생성한 정책 선택
4. "Set boundary" 클릭

결과:
- 팀 리더는 팀원 관리 가능
- 하지만 역할 생성/정책 생성 불가
- EC2, S3, CloudWatch만 접근 가능
```

#### 3단계: 권한 경계 테스트
```
테스트 시나리오:
1. 팀원 사용자 생성 → 성공
2. 팀원에게 EC2 권한 부여 → 성공
3. 팀원에게 관리자 권한 부여 시도 → 실패
4. 새로운 역할 생성 시도 → 실패

확인 방법:
- 팀 리더 계정으로 로그인
- 위 작업들을 실제 수행
- 성공/실패 결과 확인
```

---

## 📋 정기적 권한 검토 프로세스 (30분)

### 권한 검토 = 정기 건강검진

#### 검토 주기별 활동

##### 주간 검토
```
확인 항목:
- 새로 생성된 사용자/역할
- 권한 변경 요청 처리
- 보안 알림 확인
- 실패한 인증 시도 분석

자동화:
- Lambda 함수로 주간 보고서 생성
- 새 사용자 목록 이메일 발송
- 권한 변경 요약 리포트
```

##### 월간 검토
```
확인 항목:
- 사용자별 권한 사용 현황
- 미사용 권한 식별
- 액세스 키 로테이션 상태
- 정책 최적화 기회

도구 활용:
- Access Analyzer 보고서
- Credential Report 분석
- CloudTrail 로그 분석
```

##### 분기별 검토
```
확인 항목:
- 전체 권한 구조 재평가
- 조직 변경사항 반영
- 보안 정책 업데이트
- 규정 준수 상태 점검

활동:
- 부서별 권한 검토 회의
- 불필요한 권한 정리
- 새로운 보안 요구사항 적용
```

### 실습: 자동화된 권한 검토 시스템

#### Lambda 함수를 활용한 자동 보고서
```python
import boto3
import json
from datetime import datetime, timedelta

def lambda_handler(event, context):
    iam = boto3.client('iam')
    
    # 지난 30일간 사용하지 않은 사용자 찾기
    unused_users = []
    
    # Credential Report 생성
    iam.generate_credential_report()
    
    # 보고서 다운로드 및 분석
    report = iam.get_credential_report()
    
    # 분석 로직 (예시)
    for line in report['Content'].decode('utf-8').split('\n')[1:]:
        if line:
            fields = line.split(',')
            username = fields[0]
            last_used = fields[4]  # password_last_used
            
            if last_used == 'N/A' or is_old_date(last_used):
                unused_users.append(username)
    
    # 결과를 SNS로 발송
    sns = boto3.client('sns')
    message = f"미사용 사용자 목록: {unused_users}"
    
    sns.publish(
        TopicArn='arn:aws:sns:region:account:iam-review',
        Message=message,
        Subject='월간 IAM 권한 검토 보고서'
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps('보고서 발송 완료')
    }

def is_old_date(date_str):
    # 30일 이상 된 날짜인지 확인
    try:
        last_date = datetime.strptime(date_str, '%Y-%m-%dT%H:%M:%S+00:00')
        return (datetime.now() - last_date).days > 30
    except:
        return True
```

---

## 🛡️ 보안 사고 대응 프로세스 (15분)

### 사고 대응 단계

#### 1. 탐지 (Detection)
```
자동 탐지:
- CloudWatch 알람
- GuardDuty 위협 탐지
- Config 규칙 위반
- Access Analyzer 외부 접근 탐지

수동 탐지:
- 정기 검토 중 발견
- 사용자 신고
- 외부 보안 업체 알림
```

#### 2. 격리 (Containment)
```
즉시 조치:
- 의심스러운 사용자 계정 비활성화
- 액세스 키 즉시 삭제
- 역할 신뢰 정책 수정
- 보안 그룹 규칙 차단

임시 조치:
- 영향 범위 제한
- 추가 피해 방지
- 증거 보전
```

#### 3. 조사 (Investigation)
```
로그 분석:
- CloudTrail 이벤트 상세 분석
- 공격자 행동 패턴 파악
- 영향받은 리소스 식별
- 공격 경로 추적

증거 수집:
- 관련 로그 백업
- 스냅샷 생성
- 네트워크 트래픽 분석
```

#### 4. 복구 (Recovery)
```
시스템 복구:
- 손상된 리소스 복원
- 정상 권한 재설정
- 보안 설정 강화
- 모니터링 강화

프로세스 개선:
- 보안 정책 업데이트
- 모니터링 규칙 추가
- 교육 및 인식 개선
```

---

## 📝 핵심 정리

### IAM 모니터링 핵심 요소
1. **CloudTrail**: 모든 API 호출 추적 및 기록
2. **Access Analyzer**: 권한 사용 패턴 분석 및 최적화
3. **Permission Boundary**: 권한 상한선 설정
4. **정기 검토**: 체계적인 권한 관리 프로세스

### 보안 모니터링 모범 사례
- **실시간 알림**: 중요한 변경사항 즉시 탐지
- **자동화**: 정기적 검토 및 보고서 자동 생성
- **최소 권한**: 지속적인 권한 최적화
- **사고 대응**: 신속한 탐지 및 대응 체계

### 권한 관리 성숙도
- **Level 1**: 기본 CloudTrail 로깅
- **Level 2**: 자동화된 모니터링 및 알림
- **Level 3**: 지능형 분석 및 예측적 보안
- **Level 4**: 완전 자동화된 권한 관리

---

## 🤔 토론 주제

1. **어느 정도까지 모니터링해야 할까?**
   - 보안 vs 프라이버시의 균형점

2. **자동화된 권한 관리의 장단점은?**
   - 효율성 vs 통제력의 트레이드오프

---

## 📚 다음 시간 예고

**Day 3 종합 정리 및 퀴즈**
- IAM 고급 개념 전체 복습
- 실제 시나리오 기반 문제 해결
- 다중 계정 환경 권한 설계 과제
- 내일 VPC 학습 준비

---

> 💡 **오늘의 핵심**: IAM 모니터링과 감사는 지속적인 보안 유지의 핵심입니다. CloudTrail로 모든 활동을 추적하고, Access Analyzer로 권한을 최적화하며, Permission Boundary로 안전장치를 설정하는 것이 현대적인 클라우드 보안 관리의 기본입니다!