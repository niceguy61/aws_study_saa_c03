# Day 3-4: 페더레이션 및 SSO

## 📚 학습 목표
- 외부 자격 증명 공급자와의 연동 방법 이해
- AWS SSO(Single Sign-On) 기본 설정 실습
- 기업 Active Directory와 AWS 연동 개념 파악
- SAML 및 OIDC 기반 인증 메커니즘 습득

---

## 🔐 페더레이션 개념 이해 (30분)

### 페더레이션 = 신뢰 관계 네트워크

#### 전통적인 인증 문제
```
문제 상황:
- 회사: Active Directory로 사용자 관리
- AWS: 별도의 IAM 사용자 필요
- 결과: 이중 계정 관리, 비밀번호 복잡성

예시:
직원 김개발:
- 회사 계정: kim.developer@company.com
- AWS 계정: kim-developer (별도 생성 필요)
- 문제: 두 개의 비밀번호, 두 번의 로그인
```

#### 페더레이션의 해결책
```
통합 인증:
- 회사 계정으로 AWS 접근
- 단일 로그인 (Single Sign-On)
- 중앙 집중식 사용자 관리

비유:
- 대학교 학생증으로 도서관, 체육관, 식당 이용
- 한 번의 인증으로 모든 서비스 접근
- 학교에서 중앙 관리, 각 시설은 신뢰
```

### 페더레이션 유형

#### 1. SAML 2.0 페더레이션
```
특징:
- 기업 환경에서 널리 사용
- XML 기반 표준
- Active Directory와 호환성 우수

사용 사례:
- 기업 직원의 AWS 콘솔 접근
- 온프레미스 AD와 AWS 연동
- 대규모 조직의 중앙 집중식 관리
```

#### 2. OpenID Connect (OIDC)
```
특징:
- 웹 애플리케이션에 적합
- JSON 기반, REST API 친화적
- 소셜 로그인 지원

사용 사례:
- Google, Facebook 계정으로 AWS 접근
- 모바일 앱에서 AWS 서비스 사용
- 웹 애플리케이션 사용자 인증
```

#### 3. AWS SSO (Identity Center)
```
특징:
- AWS 네이티브 SSO 솔루션
- 다중 계정 환경에 최적화
- 외부 IdP와 연동 가능

사용 사례:
- AWS Organizations와 통합
- 여러 AWS 계정 통합 접근
- 비즈니스 애플리케이션 SSO
```

---

## 🏢 AWS SSO (Identity Center) 실습 (45분)

### 실습 시나리오
```
가상 회사: "TechCorp"
요구사항:
- 직원들이 단일 로그인으로 여러 AWS 계정 접근
- 부서별 차등 권한 부여
- 외부 IdP (Google Workspace) 연동
```

### 1단계: AWS SSO 활성화

#### 실습: SSO 설정 시작
```
주의사항:
- Organizations가 활성화되어 있어야 함
- 마스터 계정에서만 설정 가능
- 리전별로 하나의 SSO 인스턴스만 가능

설정 과정:
1. AWS SSO 콘솔 접속
2. "Enable AWS SSO" 클릭
3. 리전 선택 (ap-northeast-2 권장)
4. SSO 활성화 확인
```

### 2단계: 사용자 및 그룹 생성

#### 실습: 내부 사용자 디렉터리 구성
```
1단계: 그룹 생성
- Groups → "Create group"
- 그룹 생성:
  * Developers: 개발팀
  * Operations: 운영팀  
  * Managers: 관리자팀

2단계: 사용자 생성
- Users → "Add user"
- 사용자 정보:
  * Username: john.developer
  * Email: john@company.com
  * First name: John
  * Last name: Developer
  * Display name: John Developer

3단계: 그룹 멤버십
- 사용자를 적절한 그룹에 할당
- john.developer → Developers 그룹
```

### 3단계: 권한 세트 생성

#### 실습: 역할 기반 권한 정의
```
1단계: 개발자 권한 세트
- Permission sets → "Create permission set"
- Name: DeveloperAccess
- Description: 개발 환경 접근 권한
- Session duration: 8 hours
- Policies:
  * PowerUserAccess (AWS managed)
  * 커스텀 정책 (필요시)

2단계: 운영자 권한 세트  
- Name: OperationsAccess
- Policies:
  * ReadOnlyAccess
  * CloudWatchFullAccess
  * EC2InstanceConnect

3단계: 관리자 권한 세트
- Name: AdminAccess
- Policies:
  * AdministratorAccess
- MFA requirement: Required
```

### 4단계: 계정 할당

#### 실습: 그룹별 계정 접근 권한 설정
```
1단계: 계정 선택
- AWS accounts → 대상 계정 선택
- 예: Development Account

2단계: 그룹 및 권한 세트 할당
- Assign users or groups
- Groups: Developers
- Permission sets: DeveloperAccess

3단계: 할당 확인
- 개발자 그룹이 개발 계정에 접근 가능
- DeveloperAccess 권한으로 제한
```

---

## 🔗 외부 IdP 연동 실습 (30분)

### Google Workspace 연동 (시뮬레이션)

#### 1단계: 외부 IdP 설정
```
실제 환경에서의 과정:
1. Settings → Identity source
2. "Change identity source" 클릭
3. "External identity provider" 선택
4. IdP 유형: "Google Workspace"
5. 메타데이터 파일 업로드 또는 URL 입력
```

#### 2단계: SAML 속성 매핑
```
속성 매핑 설정:
- Subject: ${path:enterprise.employeeNumber}
- Email: ${path:enterprise.emails[primary eq true].value}
- FirstName: ${path:name.givenName}  
- LastName: ${path:name.familyName}
- DisplayName: ${path:displayName}

목적:
- Google Workspace 사용자 정보를 AWS SSO로 매핑
- 일관된 사용자 식별 및 속성 전달
```

### Active Directory 연동 개념

#### AWS Directory Service 활용
```
연동 옵션:
1. AWS Managed Microsoft AD
   - AWS에서 관리하는 AD
   - 온프레미스 AD와 신뢰 관계 설정

2. AD Connector  
   - 온프레미스 AD로 프록시
   - 사용자 정보는 온프레미스에 유지

3. Simple AD
   - 소규모 환경용 경량 AD
   - 기본적인 AD 기능만 제공
```

---

## 🌐 웹 애플리케이션 페더레이션 (30분)

### Amazon Cognito 활용

#### Cognito User Pools
```
용도: 웹/모바일 앱 사용자 관리
특징:
- 사용자 등록/로그인 기능
- 소셜 로그인 연동 (Google, Facebook)
- MFA 지원
- 사용자 속성 관리

사용 사례:
- 고객 대상 웹 애플리케이션
- 모바일 앱 사용자 인증
- B2C 서비스 사용자 관리
```

#### Cognito Identity Pools
```
용도: AWS 리소스 접근을 위한 임시 자격 증명
특징:
- 인증된/비인증된 사용자 모두 지원
- 역할 기반 접근 제어
- 소셜 IdP와 연동

사용 사례:
- 모바일 앱에서 S3 직접 업로드
- 웹 앱에서 DynamoDB 직접 접근
- IoT 디바이스 인증
```

### 실습: 웹 애플리케이션 인증 플로우

#### 시나리오
```
웹 애플리케이션:
- 사용자가 Google 계정으로 로그인
- 로그인 후 S3 버킷에 파일 업로드
- DynamoDB에 사용자 활동 기록
```

#### 구현 개념
```
1단계: 사용자 인증
- 사용자가 Google 로그인 클릭
- Google OAuth 2.0 플로우 실행
- ID 토큰 획득

2단계: AWS 자격 증명 획득
- Cognito Identity Pool에 ID 토큰 제출
- STS AssumeRoleWithWebIdentity 호출
- 임시 AWS 자격 증명 획득

3단계: AWS 서비스 접근
- 임시 자격 증명으로 S3 업로드
- DynamoDB에 활동 로그 기록
```

---

## 🔧 고급 페더레이션 패턴 (30분)

### 1. 조건부 접근 제어

#### SAML 속성 기반 접근
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::company-data/*",
      "Condition": {
        "StringEquals": {
          "saml:department": "Engineering"
        },
        "ForAllValues:StringEquals": {
          "saml:project": ["ProjectA", "ProjectB"]
        }
      }
    }
  ]
}
```

#### 시간 기반 접근 제한
```json
{
  "Condition": {
    "DateGreaterThan": {
      "aws:CurrentTime": "09:00Z"
    },
    "DateLessThan": {
      "aws:CurrentTime": "18:00Z"
    },
    "StringEquals": {
      "saml:location": "Seoul-Office"
    }
  }
}
```

### 2. 다중 IdP 지원

#### 시나리오
```
요구사항:
- 직원: Active Directory 인증
- 파트너: Google Workspace 인증  
- 고객: Amazon Cognito 인증
- 각각 다른 권한 레벨
```

#### 구현 전략
```
1. AWS SSO: 직원 및 파트너 관리
2. Cognito: 고객 관리
3. IAM 역할: IdP별 차등 권한
4. 조건부 정책: 세밀한 접근 제어
```

---

## 📊 페더레이션 모니터링 (15분)

### CloudTrail을 통한 추적

#### 모니터링 항목
```
페더레이션 이벤트:
- AssumeRoleWithSAML
- AssumeRoleWithWebIdentity  
- GetFederationToken
- AssumeRole (SSO)

분석 포인트:
- 누가 언제 로그인했는가?
- 어떤 IdP를 통해 인증했는가?
- 어떤 역할로 전환했는가?
- 비정상적인 접근 패턴은 없는가?
```

### 보안 모니터링

#### 알림 설정
```
CloudWatch 알람:
- 실패한 페더레이션 시도 급증
- 비정상적인 시간대 접근
- 새로운 IdP 설정 변경
- 권한 상승 시도

대응 방안:
- 자동 계정 잠금
- 보안팀 즉시 알림
- 상세 로그 수집 및 분석
```

---

## 📝 핵심 정리

### 페더레이션 핵심 개념
1. **신뢰 관계**: 외부 IdP와 AWS 간의 신뢰 설정
2. **속성 매핑**: 외부 사용자 정보를 AWS 역할로 매핑
3. **임시 자격 증명**: STS를 통한 시간 제한 접근
4. **조건부 접근**: 속성 기반 세밀한 권한 제어

### AWS SSO 장점
- **중앙 집중식 관리**: 단일 지점에서 모든 접근 관리
- **다중 계정 지원**: Organizations와 완벽 통합
- **외부 IdP 연동**: 기존 기업 인증 시스템 활용
- **사용자 경험**: 단일 로그인으로 모든 서비스 접근

### 페더레이션 선택 가이드
- **기업 환경**: AWS SSO + SAML
- **웹 애플리케이션**: Cognito + OIDC
- **모바일 앱**: Cognito Identity Pools
- **하이브리드**: 다중 IdP 조합 활용

---

## 🤔 토론 주제

1. **언제 페더레이션을 도입해야 할까?**
   - 조직 규모, 보안 요구사항, 사용자 경험 고려

2. **내부 IdP vs 외부 IdP, 어떤 선택이 좋을까?**
   - 관리 복잡성, 보안, 비용 측면에서 비교

---

## 📚 다음 시간 예고

**IAM 모니터링 및 감사**
- CloudTrail을 통한 API 호출 추적
- IAM 액세스 분석기 활용
- 권한 경계(Permission Boundary) 설정
- 정기적 권한 검토 및 정리

---

> 💡 **오늘의 핵심**: 페더레이션은 기존 기업 인증 시스템과 AWS를 연결하는 다리 역할을 합니다. AWS SSO를 통해 단일 로그인으로 여러 AWS 계정에 접근할 수 있고, 외부 IdP 연동으로 사용자 관리 복잡성을 크게 줄일 수 있습니다!