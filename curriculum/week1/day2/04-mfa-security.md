# Day 2-4: MFA 및 보안 강화

## 📚 학습 목표
- 다중 인증(MFA)의 중요성과 설정 방법 이해
- 액세스 키 관리 및 로테이션 실습
- IAM 보안 모범 사례 적용
- 보안 강화를 위한 추가 설정 완료

---

## 🔐 MFA(다중 인증) 이해 (20분)

### MFA = 이중 잠금 장치

#### MFA란?
```
정의: 두 개 이상의 인증 요소를 조합한 보안 방법
비유: 은행 금고 (카드 + 비밀번호 + 지문)

인증 요소:
1. 알고 있는 것 (Something you know)
   - 비밀번호, PIN

2. 가지고 있는 것 (Something you have)
   - 스마트폰, 하드웨어 토큰

3. 자신의 것 (Something you are)
   - 지문, 얼굴 인식
```

#### MFA의 보안 효과
```
비밀번호만 사용:
- 비밀번호 유출 시 계정 탈취
- 무차별 대입 공격 취약
- 피싱 공격 취약

MFA 사용:
- 비밀번호 + 추가 인증
- 계정 탈취 확률 99.9% 감소
- 실시간 인증 코드로 보안 강화
```

---

## 📱 Virtual MFA 설정 실습 (30분)

### 사용자별 MFA 설정

#### 실습: 관리자 사용자 MFA 설정
```
1단계: 사용자 선택
- IAM → Users → "admin-[본인이름]" 클릭
- Security credentials 탭 선택

2단계: MFA 디바이스 할당
- "Assign MFA device" 클릭
- Device name: admin-mfa-device
- MFA device type: "Authenticator app" 선택

3단계: 앱 설정
- 스마트폰에 Google Authenticator 설치
- QR 코드 스캔 또는 시크릿 키 입력

4단계: 인증 코드 입력
- MFA code 1: 현재 표시된 6자리 코드
- MFA code 2: 30초 후 새로 표시된 코드
- "Add MFA" 클릭

5단계: 설정 완료 확인
- MFA device 목록에서 확인
- 상태: Active 확인
```

#### MFA 로그인 테스트
```
테스트 과정:
1. 현재 세션 로그아웃
2. 관리자 사용자로 다시 로그인
3. 비밀번호 입력 후 MFA 코드 요구 확인
4. Google Authenticator에서 코드 확인
5. 6자리 코드 입력하여 로그인 완료
```

---

## 🔑 액세스 키 관리 (30분)

### 액세스 키 보안 원칙

#### 보안 위험
```
위험 상황:
- GitHub에 키 업로드 (자동 스캔으로 즉시 탐지)
- 이메일로 키 전송
- 하드코딩된 키
- 장기간 동일 키 사용

실제 사례:
- 개발자가 실수로 GitHub에 키 업로드
- 몇 분 내에 악용자가 키 탈취
- 수백만원의 AWS 요금 폭탄
```

#### 안전한 키 관리
```
✅ 좋은 관리:
- 환경 변수 사용
- AWS CLI 프로필 활용
- IAM 역할 사용 (가능한 경우)
- 정기적 키 로테이션

❌ 위험한 관리:
- 코드에 하드코딩
- 공개 저장소에 업로드
- 이메일/메신저로 공유
- 장기간 동일 키 사용
```

### 실습: 액세스 키 로테이션

#### 1단계: 새 액세스 키 생성
```
과정:
1. IAM → Users → "developer-[본인이름]"
2. Security credentials 탭
3. "Create access key" 클릭
4. Use case: "Command Line Interface (CLI)"
5. 새 키 생성 및 안전하게 저장
```

#### 2단계: AWS CLI 설정 업데이트
```
명령어:
aws configure
- AWS Access Key ID: [새로운 키 입력]
- AWS Secret Access Key: [새로운 시크릿 키 입력]
- Default region: ap-northeast-2
- Default output format: json

테스트:
aws sts get-caller-identity
```

#### 3단계: 기존 키 비활성화
```
과정:
1. 새 키로 정상 작동 확인
2. 기존 키 "Make inactive" 클릭
3. 24-48시간 모니터링
4. 문제없으면 기존 키 삭제
```

---

## 🛡️ IAM 보안 모범 사례 적용 (45분)

### 1. 비밀번호 정책 설정

#### 실습: 계정 비밀번호 정책 설정
```
1단계: 비밀번호 정책 접속
- IAM → Account settings
- Password policy 섹션

2단계: 정책 설정
☑ Minimum password length: 12 characters
☑ Require at least one uppercase letter
☑ Require at least one lowercase letter  
☑ Require at least one number
☑ Require at least one non-alphanumeric character
☑ Allow users to change their own password
☑ Password expiration: 90 days
☑ Prevent password reuse: 5 passwords

3단계: 정책 적용
- "Save changes" 클릭
```

### 2. 계정 별칭 설정

#### 실습: 사용자 친화적 로그인 URL
```
1단계: 계정 별칭 설정
- IAM → Dashboard
- AWS Account 섹션에서 "Create" 클릭
- Account Alias: saa-study-[본인이름]

2단계: 새 로그인 URL 확인
- 기존: https://123456789012.signin.aws.amazon.com/console
- 새로운: https://saa-study-kimcheolsu.signin.aws.amazon.com/console

3단계: 별칭 URL 테스트
- 새 시크릿 브라우저에서 별칭 URL 접속
- 로그인 테스트
```

### 3. CloudTrail 로깅 활성화

#### 실습: API 호출 추적 설정
```
1단계: CloudTrail 콘솔 접속
- Services → CloudTrail

2단계: 트레일 생성
- "Create trail" 클릭
- Trail name: saa-study-trail
- S3 bucket: 새 버킷 생성 (자동)
- Log file SSE-KMS encryption: 활성화

3단계: 이벤트 선택
☑ Management events
☑ Data events (선택사항)
☑ Insight events (선택사항)

4단계: 트레일 생성 완료
- "Create trail" 클릭
- 로깅 시작 확인
```

### 4. 사용하지 않는 자격 증명 정리

#### 실습: 자격 증명 감사
```
1단계: Credential Report 생성
- IAM → Credential report
- "Download Report" 클릭

2단계: 보고서 분석
확인 항목:
- password_last_used: 비밀번호 마지막 사용
- access_key_1_last_used_date: 키 마지막 사용
- mfa_active: MFA 활성화 여부

3단계: 정리 작업
- 90일 이상 미사용 사용자 비활성화
- 미사용 액세스 키 삭제
- MFA 미설정 사용자 알림
```

---

## 🔍 보안 모니터링 설정 (30분)

### 1. 보안 알림 설정

#### CloudWatch 알람 생성
```
루트 계정 사용 알림:
1. CloudWatch → Alarms → "Create alarm"
2. Metric: CloudTrailMetrics → RootAccountUsageEventCount
3. Threshold: Greater than 0
4. Action: SNS 토픽으로 이메일 알림

IAM 정책 변경 알림:
1. Metric: IAMPolicyEventCount  
2. Threshold: Greater than 0
3. Action: 관리자에게 즉시 알림
```

### 2. AWS Config 규칙 설정

#### 실습: 규정 준수 모니터링
```
1단계: AWS Config 활성화
- Services → Config
- "Get started" 클릭
- S3 bucket: 새 버킷 생성

2단계: 규칙 추가
추가할 규칙:
☑ root-access-key-check (루트 액세스 키 확인)
☑ mfa-enabled-for-iam-console-access (MFA 필수)
☑ iam-password-policy (비밀번호 정책)
☑ access-keys-rotated (키 로테이션)

3단계: 규칙 평가
- 자동으로 규정 준수 상태 평가
- 위반 시 알림 발송
```

---

## 📊 보안 상태 점검 (15분)

### IAM 보안 체크리스트

#### 필수 보안 설정 확인
```
루트 계정:
☑ 강력한 비밀번호 설정
☑ MFA 활성화
☑ 액세스 키 삭제 (있는 경우)
☑ 일상적 사용 금지

IAM 사용자:
☑ 개별 사용자 계정 생성
☑ 최소 권한 원칙 적용
☑ MFA 설정 (중요 사용자)
☑ 정기적 권한 검토

액세스 키:
☑ 필요한 경우에만 생성
☑ 정기적 로테이션 (90일)
☑ 안전한 저장 및 관리
☑ 미사용 키 삭제

모니터링:
☑ CloudTrail 로깅 활성화
☑ 보안 알림 설정
☑ 정기적 보안 감사
☑ Credential Report 검토
```

### 보안 점수 확인

#### AWS Security Hub (선택사항)
```
기능:
- 종합적인 보안 상태 평가
- 보안 표준 준수 확인
- 보안 권장사항 제공
- 중앙화된 보안 대시보드

활성화:
- Services → Security Hub
- "Enable Security Hub" 클릭
- 보안 표준 선택 및 활성화
```

---

## 📝 핵심 정리

### MFA 보안 효과
- **계정 탈취 방지**: 비밀번호 유출 시에도 추가 보호
- **실시간 인증**: 30초마다 변경되는 인증 코드
- **다양한 옵션**: 앱, SMS, 하드웨어 토큰 지원
- **필수 적용**: 관리자 계정 및 중요 사용자

### 액세스 키 관리 원칙
- **최소 발급**: 꼭 필요한 경우에만 생성
- **안전한 저장**: 환경변수, 설정파일 활용
- **정기 로테이션**: 90일마다 새 키로 교체
- **즉시 삭제**: 퇴사자 또는 미사용 키 즉시 제거

### 보안 모니터링 체계
- **CloudTrail**: 모든 API 호출 로깅
- **CloudWatch**: 보안 이벤트 알림
- **Config**: 규정 준수 모니터링
- **정기 감사**: Credential Report 활용

---

## 🤔 토론 주제

1. **MFA 설정 시 사용자 편의성과 보안의 균형점은?**
   - 모든 사용자 vs 관리자만 MFA 적용

2. **액세스 키 vs IAM 역할, 언제 무엇을 사용할까?**
   - 각각의 적절한 사용 시나리오

---

## 📚 다음 시간 예고

**IAM 모범 사례 종합**
- 부서별 권한 그룹 설계
- 임시 계정 관리 전략
- 정기적 권한 검토 프로세스
- 일일 정리 및 퀴즈

---

> 💡 **오늘의 핵심**: MFA는 계정 보안의 필수 요소입니다. 비밀번호만으로는 충분하지 않으며, 다중 인증을 통해 보안을 크게 강화할 수 있습니다. 액세스 키는 안전하게 관리하고 정기적으로 로테이션하는 것이 중요합니다!