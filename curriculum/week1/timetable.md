# 1주차 시간표: 클라우드 기초 및 보안 아키텍처 설계

## 📅 주간 개요
- **학습 시간**: 매일 09:00 ~ 18:00 (8시간, 점심시간 1시간 포함)
- **주요 도메인**: 보안 아키텍처 설계 (SAA-C03 30% 비중)
- **실습 환경**: AWS 프리티어 계정 활용

---

## 📚 Day 1: 클라우드 기초 & AWS 소개

### 🕘 09:00 - 10:30 | 오리엔테이션 & 클라우드 컴퓨팅 기초
- **오리엔테이션**: 과정 소개, 학습 목표, 5주 커리큘럼 안내
- **동기부여**: AWS 자격증 취득의 장점 (연봉 상승, 취업 기회, 실무 역량)
- **강의**: 온프레미스 vs 클라우드 컴퓨팅 (자가용 vs 택시)
- **강의**: 클라우드의 장점 - 비용 절감, 확장성, 유연성, 보안
- **📖 강의자료**: [01-orientation-cloud-basics.md](day1/01-orientation-cloud-basics.md)

### 🕘 10:30 - 10:45 | 휴식

### 🕘 10:45 - 12:00 | AWS 글로벌 인프라 & 계정 설정
- **강의**: 리전, 가용영역, 엣지 로케이션 (전국 체인점 네트워크)
- **실습**: AWS 계정 생성 가이드 (프리티어 활용)
- **실습**: 결제 정보 설정 및 비용 알림 설정
- **실습**: AWS 콘솔 첫 탐색 및 주요 서비스 위치 파악
- **📖 강의자료**: [02-aws-global-infrastructure.md](day1/02-aws-global-infrastructure.md)

### 🕘 12:00 - 13:00 | 점심시간

### 🕘 13:00 - 14:30 | AWS 핵심 서비스 개요
- **강의**: 컴퓨팅, 스토리지, 네트워킹, 데이터베이스 서비스 소개
- **실습**: AWS 콘솔에서 주요 서비스 둘러보기
- **사례**: 실제 비즈니스 시나리오에서의 서비스 활용
- **📖 강의자료**: [03-aws-core-services.md](day1/03-aws-core-services.md)

### 🕘 14:30 - 14:45 | 휴식

### 🕘 14:45 - 16:00 | 클라우드 경제학 & 요금 모델
- **강의**: 온디맨드, 예약 인스턴스, 스팟 인스턴스
- **실습**: AWS 요금 계산기 활용
- **과제**: 간단한 아키텍처 비용 계산
- **📖 강의자료**: [04-cloud-economics-pricing.md](day1/04-cloud-economics-pricing.md)

### 🕘 16:00 - 16:15 | 휴식

### 🕘 16:15 - 17:30 | AWS 공동 책임 모델
- **강의**: AWS와 고객의 보안 책임 분담
- **사례**: 실제 보안 사고 사례를 통한 책임 구분 이해
- **실습**: AWS 서비스별 책임 범위 확인
- **📖 강의자료**: [05-shared-responsibility-model.md](day1/05-shared-responsibility-model.md)

### 🕘 17:30 - 18:00 | 일일 정리 & 강사 접근 권한 설정
- **실습**: 강사용 ReadOnlyAccess 역할 생성 가이드
- **실습**: SwitchRole 설정으로 강사 계정 접근 허용
- **퀴즈**: 오늘 학습 내용 확인 (10문항)
- **정리**: 핵심 개념 요약 및 내일 학습 준비
- **📖 강의자료**: [06-instructor-access-setup.md](day1/06-instructor-access-setup.md)
- **📝 퀴즈**: [Day1 퀴즈](quizzes/day1-quiz.md)

---

## 📚 Day 2: IAM 기초

### 🕘 09:00 - 10:30 | IAM 개념 및 구성 요소
- **강의**: Identity and Access Management 개요 (회사 출입카드 시스템)
- **이론**: 사용자, 그룹, 역할, 정책의 관계
- **실습**: IAM 콘솔 둘러보기
- **📖 강의자료**: [01-iam-basics.md](day2/01-iam-basics.md)

### 🕘 10:30 - 10:45 | 휴식

### 🕘 10:45 - 12:00 | IAM 사용자 및 그룹 관리
- **실습**: IAM 사용자 생성 및 그룹 설정
- **실습**: 프로그래밍 방식 액세스 vs 콘솔 액세스
- **보안**: 루트 사용자 보안 모범 사례
- **📖 강의자료**: [02-iam-users-groups.md](day2/02-iam-users-groups.md)

### 🕘 12:00 - 13:00 | 점심시간

### 🕘 13:00 - 14:30 | IAM 정책 설계
- **강의**: 관리형 정책 vs 인라인 정책
- **실습**: JSON 정책 문서 작성 및 적용
- **실습**: 정책 시뮬레이터 활용
- **📖 강의자료**: [03-iam-policies.md](day2/03-iam-policies.md)

### 🕘 14:30 - 14:45 | 휴식

### 🕘 14:45 - 16:00 | MFA 및 보안 강화
- **강의**: 다중 인증(MFA)의 중요성
- **실습**: 가상 MFA 디바이스 설정
- **실습**: 액세스 키 로테이션
- **📖 강의자료**: [04-mfa-security.md](day2/04-mfa-security.md)

### 🕘 16:00 - 16:15 | 휴식

### 🕘 16:15 - 17:30 | IAM 모범 사례
- **강의**: 최소 권한 원칙 적용
- **사례**: 실제 기업의 IAM 권한 설계 사례
- **실습**: 부서별 권한 그룹 설계
- **📖 강의자료**: [05-iam-best-practices.md](day2/05-iam-best-practices.md)

### 🕘 17:30 - 18:00 | 일일 정리 & Q&A
- **퀴즈**: IAM 개념 확인 (15문항)
- **과제**: 소규모 조직의 IAM 구조 설계
- **📝 퀴즈**: [Day2 퀴즈](quizzes/day2-quiz.md)

---

## 📚 Day 3: IAM 고급 & 다중 계정 전략

### 🕘 09:00 - 10:30 | IAM 역할(Role) 심화
- **강의**: 역할의 개념과 사용 시나리오 (임시 출입증)
- **실습**: EC2 인스턴스용 역할 생성
- **실습**: 역할 전환 실습
- **📖 강의자료**: [01-iam-roles-advanced.md](day3/01-iam-roles-advanced.md)

### 🕘 10:30 - 10:45 | 휴식

### 🕘 10:45 - 12:00 | 교차 계정 액세스
- **강의**: 여러 AWS 계정 간 리소스 공유
- **실습**: 교차 계정 역할 설정 및 전환
- **실습**: AWS STS(Security Token Service) 활용
- **📖 강의자료**: [02-cross-account-access.md](day3/02-cross-account-access.md)

### 🕘 12:00 - 13:00 | 점심시간

### 🕘 13:00 - 14:30 | AWS Organizations
- **강의**: 다중 계정 관리 전략
- **실습**: 조직 단위(OU) 생성 및 계정 관리
- **실습**: 서비스 제어 정책(SCP) 적용
- **📖 강의자료**: [03-aws-organizations.md](day3/03-aws-organizations.md)

### 🕘 14:30 - 14:45 | 휴식

### 🕘 14:45 - 16:00 | 페더레이션 및 SSO
- **강의**: 외부 자격 증명 공급자와의 연동
- **실습**: AWS SSO(Single Sign-On) 기본 설정
- **사례**: 기업 Active Directory와 AWS 연동
- **📖 강의자료**: [04-federation-sso.md](day3/04-federation-sso.md)

### 🕘 16:00 - 16:15 | 휴식

### 🕘 16:15 - 17:30 | IAM 모니터링 및 감사
- **강의**: CloudTrail을 통한 API 호출 추적
- **실습**: IAM 액세스 분석기 활용
- **실습**: 권한 경계(Permission Boundary) 설정
- **📖 강의자료**: [05-iam-monitoring-audit.md](day3/05-iam-monitoring-audit.md)

### 🕘 17:30 - 18:00 | 일일 정리 & Q&A
- **퀴즈**: IAM 고급 개념 확인 (15문항)
- **과제**: 다중 계정 환경의 권한 설계
- **정리**: Day 3 전체 내용 요약 및 내일 VPC 학습 준비
- **📝 퀴즈**: [Day3 퀴즈](quizzes/day3-quiz.md)

---

## 📚 Day 4: VPC 기초

### 🕘 09:00 - 10:30 | VPC 개념 및 구성 요소
- **강의**: Virtual Private Cloud 개요 (아파트 단지)
- **이론**: CIDR 블록, 서브넷, 라우팅 테이블
- **실습**: IP 주소 체계 이해
- **📖 강의자료**: [01-vpc-concepts.md](day4/01-vpc-concepts.md)

### 🕘 10:30 - 10:45 | 휴식

### 🕘 10:45 - 12:00 | VPC 생성 및 서브넷 설계
- **실습**: 사용자 정의 VPC 생성
- **실습**: 퍼블릭/프라이빗 서브넷 구성
- **실습**: 인터넷 게이트웨이 연결
- **📖 강의자료**: [02-vpc-creation-subnets.md](day4/02-vpc-creation-subnets.md)

### 🕘 12:00 - 13:00 | 점심시간

### 🕘 13:00 - 14:30 | 라우팅 및 NAT
- **강의**: 라우팅 테이블 설정 원리
- **실습**: NAT 게이트웨이 vs NAT 인스턴스
- **실습**: 프라이빗 서브넷 인터넷 연결
- **📖 강의자료**: [03-routing-nat.md](day4/03-routing-nat.md)

### 🕘 14:30 - 14:45 | 휴식

### 🕘 14:45 - 16:00 | VPC 연결 옵션
- **강의**: VPC 피어링, Transit Gateway
- **실습**: VPC 피어링 연결 설정
- **사례**: 멀티 VPC 아키텍처 설계
- **📖 강의자료**: [04-vpc-connectivity.md](day4/04-vpc-connectivity.md)

### 🕘 16:00 - 16:15 | 휴식

### 🕘 16:15 - 17:30 | 하이브리드 연결
- **강의**: VPN vs Direct Connect
- **실습**: Site-to-Site VPN 설정
- **사례**: 온프레미스와 클라우드 연결 시나리오
- **📖 강의자료**: [05-hybrid-connectivity.md](day4/05-hybrid-connectivity.md)

### 🕘 17:30 - 18:00 | 일일 정리 & Q&A
- **퀴즈**: VPC 네트워킹 개념 확인 (15문항)
- **과제**: 3-tier 아키텍처 네트워크 설계
- **📖 강의자료**: [06-daily-summary.md](day4/06-daily-summary.md)
- **📝 퀴즈**: [Day4 퀴즈](quizzes/day4-quiz.md)

---

## 📚 Day 5: VPC 보안 & 종합 정리

### 🕘 09:00 - 10:30 | 보안 그룹 vs NACL
- **강의**: 보안 그룹과 네트워크 ACL 차이점
- **실습**: 보안 그룹 규칙 설정
- **실습**: NACL을 통한 서브넷 레벨 보안
- **📖 강의자료**: [01-security-groups-nacl.md](day5/01-security-groups-nacl.md)

### 🕘 10:30 - 10:45 | 휴식

### 🕘 10:45 - 12:00 | VPC 엔드포인트
- **강의**: 게이트웨이 엔드포인트 vs 인터페이스 엔드포인트
- **실습**: S3 게이트웨이 엔드포인트 설정
- **실습**: EC2 인터페이스 엔드포인트 구성
- **📖 강의자료**: [02-vpc-endpoints.md](day5/02-vpc-endpoints.md)

### 🕘 12:00 - 13:00 | 점심시간

### 🕘 13:00 - 14:30 | 네트워크 모니터링
- **강의**: VPC Flow Logs 활용
- **실습**: CloudWatch를 통한 네트워크 모니터링
- **실습**: 네트워크 트래픽 분석
- **📖 강의자료**: [03-network-monitoring.md](day5/03-network-monitoring.md)

### 🕘 14:30 - 14:45 | 휴식

### 🕘 14:45 - 16:00 | 주간 과제 설명 및 시작
- **과제**: "안전한 웹 서비스 환경 구축"
- **시나리오**: 온라인 쇼핑몰 인프라 설계
- **가이드**: ReadOnlyAccess 역할 부여 방법
- **📖 강의자료**: [04-weekly-assignment.md](day5/04-weekly-assignment.md)

### 🕘 16:00 - 16:15 | 휴식

### 🕘 16:15 - 17:30 | 1주차 종합 정리
- **복습**: 클라우드 기초부터 VPC 보안까지
- **실습**: 전체 아키텍처 통합 구현
- **토론**: 실제 기업 사례 분석
- **📖 강의자료**: [05-week1-summary.md](day5/05-week1-summary.md)

### 🕘 17:30 - 18:00 | 주간 평가 & 다음 주 준비
- **시험**: 1주차 종합 평가 (30문항)
- **피드백**: 개별 학습 상담
- **예고**: 2주차 학습 내용 소개
- **📖 강의자료**: [06-weekly-evaluation.md](day5/06-weekly-evaluation.md)
- **📝 종합시험**: [Day5 종합 퀴즈](quizzes/day5-quiz.md)

---

## 📋 주간 학습 체크리스트

### ✅ 필수 완료 항목
- [ ] AWS 계정 생성 및 MFA 설정
- [ ] 강사용 ReadOnlyAccess 역할 생성 및 SwitchRole 설정
- [ ] IAM 사용자/그룹/역할 생성 실습
- [ ] VPC 및 서브넷 구성 실습
- [ ] 보안 그룹 및 NACL 설정
- [ ] 주간 과제 완료

### 📚 추가 학습 권장
- [ ] AWS Well-Architected Framework 보안 기둥 읽기
- [ ] IAM 정책 예제 추가 연습
- [ ] VPC 설계 패턴 사례 연구

### 🎯 다음 주 준비
- [ ] EC2 인스턴스 유형 미리 살펴보기
- [ ] Auto Scaling 개념 예습
- [ ] 서버리스 컴퓨팅 개념 조사