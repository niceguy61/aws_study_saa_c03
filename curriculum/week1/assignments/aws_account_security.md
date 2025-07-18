# 주간 과제: AWS 계정 설정 및 보안 강화

## 개요
이 과제에서는 AWS 계정의 기본 보안 설정을 구성하고, 루트 계정 보호, IAM 사용자 생성, MFA 설정 등 AWS 계정 사용을 위한 필수적인 보안 조치를 적용합니다. 또한 강사의 계정에 ReadOnlyAccess 권한을 부여하여 아키텍처 점검 및 비상시 가이드라인 제공을 위한 환경을 구성합니다.

## 학습 목표
- AWS 계정의 기본 보안 설정 방법 습득
- 루트 계정 보호 및 MFA 설정 방법 이해
- IAM 사용자 생성 및 관리자 권한 부여 방법 학습
- 교차 계정 액세스를 위한 IAM 역할 구성 방법 습득

## 요구 사항

### 1. GitHub 저장소 설정
- AWS 학습을 위한 Private GitHub 저장소 생성
- 저장소 이름: aws_study_saa 또는 원하는 이름
- 강사를 팀원으로 초대 (이메일: niceguy61@naver.com)
- README.md 파일에 본인 이름과 학습 목표 간단히 작성

### 2. AWS 계정 생성 및 루트 계정 보안 강화
- AWS 계정 생성 (Free Tier 계정)
- 루트 사용자에 MFA(Multi-Factor Authentication) 설정
  - 가상 MFA 디바이스(스마트폰 앱) 또는 하드웨어 MFA 디바이스 사용
- 루트 사용자 액세스 키 삭제 (존재하는 경우)
- 강력한 암호 설정 (최소 12자 이상, 대소문자, 숫자, 특수문자 포함)
- 계정 연락처 정보 업데이트 및 대체 연락처 설정

### 3. 관리자 IAM 사용자 생성
- 관리자 권한을 가진 IAM 사용자 생성 (사용자명: AdminUser)
- AdministratorAccess 관리형 정책 연결
- 관리자 사용자에 MFA 설정
- 관리자 사용자의 로그인 URL 저장 및 로그인 테스트
- 이후 모든 작업은 루트 계정이 아닌 관리자 사용자로 수행

### 4. IAM 비밀번호 정책 설정
- 최소 12자 이상의 비밀번호 길이 요구
- 대문자, 소문자, 숫자, 특수 문자 포함 요구
- 비밀번호 재사용 금지 (최소 5개 이전 비밀번호)
- 최대 90일의 비밀번호 만료 기간 설정
- 사용자가 자신의 비밀번호를 변경할 수 있도록 허용

### 5. 강사 계정을 위한 교차 계정 액세스 역할 생성
- 강사의 AWS 계정에 ReadOnlyAccess 권한을 부여하는 IAM 역할 생성
- 역할 이름: InstructorReadOnlyRole
- 신뢰 관계 설정: 강사가 제공한 AWS 계정 ID 입력
- AWS 관리형 정책 "ReadOnlyAccess" 연결
- 역할 ARN 기록 및 강사에게 제공
- 이 역할은 아키텍처 점검 및 비상시 가이드라인 제공을 위해 사용됩니다

### 6. 계정 보안 상태 확인
- IAM 보안 권장 사항 검토
- IAM 자격 증명 보고서 생성 및 검토
- AWS Trusted Advisor 기본 보안 점검 실행 (가능한 경우)
- 발견된 보안 문제 해결

## 제출 요구 사항

### 1. GitHub 저장소에 문서화
- 각 단계별 설정 과정 스크린샷 및 설명을 마크다운 파일로 작성
- 파일명: aws_account_security_setup.md
- 민감한 정보(계정 ID 외의 개인정보, 비밀번호 등)는 반드시 가려서 제출

### 2. 설정 문서에 포함할 내용
- IAM 대시보드 보안 상태 스크린샷
- 생성한 IAM 사용자 목록 스크린샷 (민감한 정보 제외)
- MFA 설정 화면 스크린샷
- 비밀번호 정책 설정 스크린샷
- 생성한 InstructorReadOnlyRole의 ARN
- 역할 신뢰 정책 JSON 내용
- 역할에 연결된 정책 목록
- IAM 보안 권장 사항 점검 결과
- 발견된 보안 문제 및 해결 방법
- 추가 보안 개선 권장 사항

## 교차 계정 액세스 역할 생성 가이드

### 1. IAM 콘솔 접속
- AWS Management Console에 로그인
- IAM 서비스로 이동

### 2. 역할 생성
- 왼쪽 메뉴에서 "역할(Roles)" 선택
- "역할 생성(Create role)" 버튼 클릭

### 3. 신뢰할 수 있는 엔터티 유형 선택
- "다른 AWS 계정(Another AWS account)" 선택
- "계정 ID(Account ID)" 필드에 강사가 제공한 AWS 계정 ID 입력
- (선택 사항) "MFA 필요(Require MFA)" 옵션 체크
- "다음: 권한(Next: Permissions)" 클릭

### 4. 권한 정책 연결
- 검색 창에 "ReadOnlyAccess" 입력
- "ReadOnlyAccess" 관리형 정책 체크
- "다음: 태그(Next: Tags)" 클릭

### 5. 태그 추가 (선택 사항)
- 키: "Purpose", 값: "ArchitectureReview" 태그 추가
- "다음: 검토(Next: Review)" 클릭

### 6. 역할 정보 검토 및 생성
- 역할 이름: "InstructorReadOnlyRole" 입력
- 역할 설명: "강사의 아키텍처 점검 및 가이드라인 제공을 위한 읽기 전용 액세스 역할" 입력
- 정보 확인 후 "역할 생성(Create role)" 클릭

### 7. 역할 ARN 확인
- 생성된 역할 목록에서 "InstructorReadOnlyRole" 클릭
- 역할 ARN 복사 (형식: arn:aws:iam::123456789012:role/InstructorReadOnlyRole)
- 강사에게 역할 ARN 제공 (GitHub 저장소의 README.md 또는 별도 문서에 기록)

## 강사의 역할 수임 방법 안내

강사가 여러분의 계정에 접근하기 위해 다음 단계를 수행합니다:

1. 강사는 자신의 AWS 계정에 로그인
2. AWS Management Console 오른쪽 상단의 계정 이름 클릭
3. "역할 전환(Switch Role)" 선택
4. 다음 정보 입력:
   - 계정: 여러분의 AWS 계정 ID
   - 역할: InstructorReadOnlyRole
   - 표시 이름: 원하는 표시 이름 (예: 학생이름-계정)
   - (선택 사항) 색상 선택
5. "역할 전환(Switch Role)" 버튼 클릭

이 과정을 통해 강사는 여러분의 AWS 계정 리소스를 읽기 전용으로 접근하여 아키텍처를 검토하고 필요시 가이드라인을 제공할 수 있습니다.

## 완료 기준

- Private GitHub 저장소 생성 및 강사 초대 완료
- 루트 사용자 MFA 설정 완료
- 강력한 비밀번호 정책 구성
- 관리자 IAM 사용자 생성 및 MFA 설정
- 루트 사용자 액세스 키 삭제
- InstructorReadOnlyRole 역할 올바르게 생성
- 적절한 신뢰 정책 설정
- ReadOnlyAccess 정책 연결
- IAM 보안 권장 사항 적용
- 최소 권한 원칙 적용
- 보안 상태 모니터링 구성
- 모든 과정 문서화 및 GitHub 저장소에 업로드

## 보너스 과제 (선택 사항)

### 1. AWS CloudShell 설정
- AWS CloudShell 액세스 및 기본 설정
- AWS CLI를 사용한 계정 정보 확인 명령어 실행
- CloudShell 사용 경험 문서화

### 2. 비용 알림 설정
- AWS Budgets를 사용한 비용 알림 설정
- 월별 예산 설정 및 알림 구성
- 비용 모니터링 대시보드 설정

### 3. AWS CLI 프로필 구성
- 로컬 환경에 AWS CLI 설치
- 관리자 사용자를 위한 CLI 프로필 구성
- 기본 AWS CLI 명령어 테스트 및 결과 문서화

## 참고 자료
- [AWS 계정 생성 가이드](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)
- [IAM 사용자 생성 가이드](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)
- [MFA 설정 가이드](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable.html)
- [교차 계정 액세스 가이드](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html)
- [AWS 보안 모범 사례](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [GitHub 저장소 생성 및 팀원 초대 가이드](https://docs.github.com/ko/repositories/creating-and-managing-repositories/creating-a-new-repository)

## 제출 마감일
2주차 월요일 오전 9시까지 GitHub 저장소 URL을 제출하세요.

## 질문 및 지원
과제 관련 질문이나 기술적 문제가 있는 경우, 강의 시간 중에 질문하거나 이메일로 문의하세요.