# Day 2-2: IAM 사용자 및 그룹 관리

## 📚 학습 목표
- IAM 사용자 생성 및 권한 설정 실습
- IAM 그룹 생성 및 사용자 할당 실습
- 프로그래밍 방식 액세스와 콘솔 액세스 차이점 이해
- 루트 사용자 보안 모범 사례 적용

---

## 🔐 루트 사용자 보안 강화 (30분)

### 루트 사용자 = 회사 마스터키

#### 루트 사용자의 특징
```
절대 권한:
- 계정 삭제 가능
- 결제 정보 변경
- 지원 플랜 변경
- 모든 IAM 권한 무시

위험성:
- 해킹 시 전체 계정 탈취
- 실수로 전체 리소스 삭제 가능
- 추적 및 제어 어려움
```

### 루트 사용자 보안 설정

#### 1단계: 강력한 비밀번호 설정
```
비밀번호 요구사항:
- 최소 8자 이상
- 대문자, 소문자, 숫자, 특수문자 포함
- 개인정보와 무관한 복잡한 조합
- 정기적 변경 (3-6개월)

예시 (사용하지 마세요):
- 약한 비밀번호: password123, 생년월일
- 강한 비밀번호: MyAws#2024$Secure!
```

#### 2단계: MFA(다중 인증) 설정 실습
```
MFA 설정 과정:
1. IAM 대시보드 → 루트 사용자 MFA 설정
2. "Activate MFA" 클릭
3. MFA 디바이스 유형 선택:
   - Virtual MFA device (추천)
   - Hardware MFA device
   - SMS text message (비추천)

Virtual MFA 앱:
- Google Authenticator
- Microsoft Authenticator
- Authy
```

#### 실습: Virtual MFA 설정
```
1. 스마트폰에 Google Authenticator 설치
2. AWS 콘솔에서 QR 코드 표시
3. 앱으로 QR 코드 스캔
4. 연속된 두 개의 MFA 코드 입력
5. MFA 설정 완료 확인
```

---

## 👤 첫 번째 IAM 사용자 생성 (45분)

![IAM User Group Relationship](../images/iam-user-group-relationship.svg)

### 관리자 사용자 생성

#### 왜 관리자 사용자가 필요한가?
```
루트 사용자 문제점:
- 일상적 사용 시 보안 위험
- 세밀한 권한 제어 불가
- 활동 추적 어려움

관리자 사용자 장점:
- 필요한 권한만 부여
- 활동 로그 추적 가능
- 필요 시 권한 회수 가능
- 여러 관리자 계정 생성 가능
```

#### 실습: 관리자 사용자 생성
```
1단계: 사용자 생성 시작
- IAM → Users → "Create user"
- User name: admin-[본인이름]
- 예: admin-kimcheolsu

2단계: 액세스 유형 선택
☑ Provide user access to the AWS Management Console
☑ I want to create an IAM user
- Console password: Custom password
- 비밀번호: 강력한 비밀번호 입력
☑ Users must create a new password at next sign-in

3단계: 권한 설정
- "Attach policies directly" 선택
- "AdministratorAccess" 정책 검색 및 선택
- 이 정책의 의미: 거의 모든 AWS 서비스 관리 권한
```

#### 사용자 생성 완료 및 확인
```
생성 완료 후 확인사항:
1. 사용자 이름 확인
2. 콘솔 로그인 URL 복사
3. 임시 비밀번호 확인 (있는 경우)
4. 사용자 ARN 확인

콘솔 로그인 URL 형식:
https://[계정ID].signin.aws.amazon.com/console
```

---

## 👥 IAM 그룹 생성 및 관리 (30분)

### 개발팀 그룹 생성

#### 실습: 개발자 그룹 생성
```
1단계: 그룹 생성 시작
- IAM → User groups → "Create group"
- Group name: Developers

2단계: 권한 정책 선택
검색 및 선택할 정책들:
☑ AmazonEC2FullAccess (EC2 완전 관리)
☑ AmazonS3FullAccess (S3 완전 관리)
☑ CloudWatchReadOnlyAccess (모니터링 조회)

3단계: 그룹 생성 완료
- "Create group" 클릭
- 그룹 생성 확인
```

### 읽기 전용 그룹 생성

#### 실습: 읽기 전용 그룹 생성
```
1단계: 그룹 생성
- Group name: ReadOnlyUsers

2단계: 권한 정책 선택
☑ ReadOnlyAccess
- 모든 AWS 서비스 읽기 권한
- 생성/수정/삭제 권한 없음

3단계: 그룹 생성 완료
```

---

## 🔧 사용자 생성 및 그룹 할당 실습 (45분)

### 개발자 사용자 생성

#### 실습: 개발자 사용자 생성
```
1단계: 사용자 기본 정보
- User name: developer-[본인이름]
- 예: developer-parkdesign
- Console access: Enable

2단계: 권한 설정 방법 선택
- "Add user to group" 선택
- "Developers" 그룹 선택
- 직접 정책 연결하지 않음 (그룹을 통해 권한 상속)

3단계: 사용자 생성 완료
```

### 읽기 전용 사용자 생성

#### 실습: 읽기 전용 사용자 생성
```
1단계: 사용자 기본 정보
- User name: readonly-[본인이름]
- Console access: Enable

2단계: 그룹 할당
- "ReadOnlyUsers" 그룹 선택

3단계: 생성 완료
```

---

## 🔑 프로그래밍 방식 액세스 설정 (30분)

### Access Key란?

#### 개념 이해
```
Access Key 구성:
- Access Key ID: 공개 식별자 (사용자명 같은 역할)
- Secret Access Key: 비밀 키 (비밀번호 같은 역할)

사용 목적:
- AWS CLI 사용
- SDK를 통한 프로그래밍 접근
- API 호출
- 자동화 스크립트
```

#### 보안 고려사항
```
✅ 좋은 관리:
- 필요한 사용자에게만 발급
- 정기적 키 순환 (90일 권장)
- 코드에 하드코딩 금지
- 환경변수 또는 설정파일 사용

❌ 위험한 관리:
- GitHub 등 공개 저장소에 업로드
- 이메일로 키 전송
- 여러 사람이 동일 키 공유
- 장기간 동일 키 사용
```

### 실습: Access Key 생성

#### 개발자 사용자에게 Access Key 발급
```
1단계: 사용자 선택
- IAM → Users → "developer-[본인이름]" 클릭

2단계: Security credentials 탭
- "Create access key" 클릭

3단계: 사용 사례 선택
- "Command Line Interface (CLI)" 선택
- 확인 체크박스 선택

4단계: 설명 태그 (선택사항)
- Description: "개발용 CLI 접근"

5단계: 키 생성 완료
- Access Key ID 복사
- Secret Access Key 복사 (한 번만 표시됨!)
- CSV 파일 다운로드 권장
```

---

## 🧪 권한 테스트 실습 (30분)

### 사용자별 권한 차이 확인

#### 실습 1: 관리자 사용자 테스트
```
1. 새 시크릿 브라우저 창 열기
2. 관리자 사용자로 로그인
3. 다음 작업 시도:
   - EC2 대시보드 접근 ✅
   - S3 버킷 생성 시도 ✅
   - IAM 사용자 목록 조회 ✅
   - Billing 정보 접근 ❌ (루트 권한 필요)
```

#### 실습 2: 개발자 사용자 테스트
```
1. 다른 시크릿 브라우저 창 열기
2. 개발자 사용자로 로그인
3. 다음 작업 시도:
   - EC2 대시보드 접근 ✅
   - EC2 인스턴스 생성 시도 ✅
   - S3 버킷 생성 시도 ✅
   - IAM 사용자 생성 시도 ❌
```

#### 실습 3: 읽기 전용 사용자 테스트
```
1. 또 다른 시크릿 브라우저 창 열기
2. 읽기 전용 사용자로 로그인
3. 다음 작업 시도:
   - 모든 서비스 대시보드 조회 ✅
   - EC2 인스턴스 목록 조회 ✅
   - S3 버킷 목록 조회 ✅
   - 리소스 생성/수정 시도 ❌
```

---

## 📊 사용자 활동 모니터링 (15분)

### CloudTrail을 통한 활동 추적

#### 실습: 사용자 활동 확인
```
1. CloudTrail 콘솔 접속
2. Event history 메뉴 선택
3. 필터 설정:
   - User name: 특정 사용자 선택
   - Time range: 최근 1시간
4. 활동 로그 확인:
   - 로그인 시간
   - 수행한 작업
   - 접근한 서비스
```

### 보안 모니터링 설정
```
권장 모니터링 항목:
- 루트 계정 사용 알림
- 새 IAM 사용자 생성 알림
- 권한 변경 알림
- 비정상적 로그인 시도 알림
```

---

## 📝 핵심 정리

### IAM 사용자 관리 모범 사례
1. **루트 사용자 보호**: MFA 설정, 일상 사용 금지
2. **개별 사용자 계정**: 공유 계정 사용 금지
3. **그룹 기반 권한**: 개별 권한보다 그룹 활용
4. **최소 권한 원칙**: 필요한 최소한의 권한만 부여
5. **정기적 검토**: 사용자 및 권한 정기 점검

### 액세스 키 관리 원칙
- **필요 시에만 발급**: 콘솔 접근만으로 충분한 경우 발급 안 함
- **안전한 보관**: 코드에 하드코딩 금지
- **정기적 순환**: 90일마다 새 키로 교체
- **즉시 비활성화**: 퇴사자 또는 불필요 시 즉시 삭제

### 권한 설계 전략
- **역할 기반**: 업무 역할에 따른 그룹 설계
- **환경별 분리**: 개발/테스트/운영 환경별 권한 분리
- **임시 권한**: 일시적 작업은 역할 활용
- **정기 감사**: 권한 사용 현황 정기 검토

---

## 🤔 토론 주제

1. **회사에서 IAM 사용자를 어떻게 관리할까?**
   - 신입사원 온보딩 프로세스
   - 퇴사자 오프보딩 프로세스

2. **개발팀과 운영팀의 권한 차이는?**
   - 각 팀에 필요한 최소 권한은 무엇일까?

---

## 📚 다음 시간 예고

**IAM 고급 & 다중 계정 전략**
- IAM 역할(Role) 심화 학습
- 교차 계정 액세스 설정
- AWS STS(Security Token Service) 활용
- AWS Organizations를 통한 다중 계정 관리

---

## 📖 참고 자료

### AWS 공식 문서
- [IAM 사용자 생성](https://docs.aws.amazon.com/iam/latest/userguide/id_users_create.html)
- [IAM 그룹 관리](https://docs.aws.amazon.com/iam/latest/userguide/id_groups_manage.html)
- [IAM 액세스 키](https://docs.aws.amazon.com/iam/latest/userguide/id_credentials_access-keys.html)
- [MFA 설정](https://docs.aws.amazon.com/iam/latest/userguide/id_credentials_mfa_enable.html)
- [IAM 모범 사례](https://docs.aws.amazon.com/iam/latest/userguide/best-practices.html)

---

> 💡 **오늘의 핵심**: IAM 사용자와 그룹을 통해 체계적인 권한 관리가 가능합니다. 루트 사용자는 응급상황에만 사용하고, 일상적인 작업은 적절한 권한을 가진 IAM 사용자로 수행하는 것이 보안의 기본입니다!