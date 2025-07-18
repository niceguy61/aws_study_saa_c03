# Day 4-5: 하이브리드 연결

## 📚 학습 목표
- VPN과 Direct Connect의 차이점과 사용 사례 이해
- Site-to-Site VPN 설정 및 구성 방법 습득
- 온프레미스와 클라우드 연결 시나리오 분석
- 하이브리드 아키텍처 설계 원칙 및 모범 사례 학습

---

## 🌉 하이브리드 클라우드 개념 (30분)

### 하이브리드 클라우드란?

#### 하이브리드 클라우드의 정의
```
하이브리드 클라우드 = 온프레미스 + 퍼블릭 클라우드
- 기존 데이터센터와 AWS 클라우드의 결합
- 워크로드를 적절히 분산 배치
- 점진적 클라우드 마이그레이션 지원

비유: 본사와 지사의 연결
- 본사 (온프레미스): 핵심 시스템, 민감한 데이터
- 지사 (클라우드): 확장 가능한 서비스, 새로운 애플리케이션
- 전용선 (연결): 안전하고 빠른 통신 경로
```

#### 하이브리드 클라우드 도입 이유
```
기술적 이유:
✅ 기존 투자 보호 (레거시 시스템)
✅ 점진적 마이그레이션 가능
✅ 데이터 지역성 요구사항 충족
✅ 네트워크 지연시간 최소화

비즈니스 이유:
✅ 규정 준수 요구사항
✅ 데이터 주권 및 보안 정책
✅ 비용 최적화 (기존 자산 활용)
✅ 위험 분산 (멀티 클라우드 전략)

운영적 이유:
✅ 기존 운영 프로세스 유지
✅ 직원 스킬셋 활용
✅ 단계적 변화 관리
✅ 백업 및 재해 복구
```

### 하이브리드 연결 옵션

#### AWS 하이브리드 연결 서비스
```
1. AWS Site-to-Site VPN
   - 인터넷 기반 암호화 연결
   - 빠른 구축, 저렴한 비용
   - 대역폭 제한, 지연시간 가변적

2. AWS Direct Connect
   - 전용 물리적 연결
   - 높은 대역폭, 일관된 성능
   - 구축 시간 오래, 높은 비용

3. AWS Client VPN
   - 개별 사용자 원격 접속
   - 재택근무, 모바일 접속 지원

4. AWS Transit Gateway
   - 중앙 집중식 연결 허브
   - VPN, Direct Connect 통합 관리
```

---

## 🔐 VPN vs Direct Connect 비교 (45분)

### Site-to-Site VPN

#### VPN 특징 및 장단점
```
AWS Site-to-Site VPN:
- IPSec 터널을 통한 암호화 통신
- 인터넷을 통한 연결
- 이중화 터널 자동 구성 (고가용성)

장점:
✅ 빠른 구축 (몇 분 내 설정 가능)
✅ 저렴한 비용 ($0.05/시간 + 데이터 전송비)
✅ 전 세계 어디서나 연결 가능
✅ 완전 관리형 서비스
✅ 강력한 암호화 (AES-256)

단점:
❌ 인터넷 품질에 의존적
❌ 대역폭 제한 (최대 1.25Gbps per 터널)
❌ 지연시간 가변적
❌ 패킷 손실 가능성
```

#### VPN 구성 요소
```
1. Virtual Private Gateway (VGW)
   - AWS 측 VPN 엔드포인트
   - VPC에 연결되는 게이트웨이
   - 이중화 구성으로 고가용성 제공

2. Customer Gateway (CGW)
   - 고객 측 VPN 디바이스 정보
   - 퍼블릭 IP 주소 및 BGP ASN
   - 물리적 또는 소프트웨어 디바이스

3. VPN Connection
   - VGW와 CGW 간의 IPSec 터널
   - 2개의 터널로 이중화 구성
   - BGP 또는 정적 라우팅 지원
```

### AWS Direct Connect

#### Direct Connect 특징 및 장단점
```
AWS Direct Connect:
- 전용 물리적 네트워크 연결
- AWS Direct Connect Location을 통한 연결
- 일관된 네트워크 성능 보장

장점:
✅ 높은 대역폭 (1Gbps ~ 100Gbps)
✅ 일관된 네트워크 성능
✅ 낮은 지연시간
✅ 데이터 전송 비용 절약
✅ 프라이빗 연결 (인터넷 경유 없음)

단점:
❌ 긴 구축 시간 (몇 주 ~ 몇 달)
❌ 높은 초기 비용
❌ 물리적 위치 제약
❌ 단일 장애점 (이중화 필요)
❌ 복잡한 설정 및 관리
```

#### Direct Connect 구성 요소
```
1. Direct Connect Gateway
   - 여러 VPC와 연결 가능한 글로벌 리소스
   - 리전 간 연결 지원
   - Transit Gateway와 통합 가능

2. Virtual Interface (VIF)
   - 논리적 연결 인터페이스
   - Public VIF: AWS 퍼블릭 서비스 접근
   - Private VIF: VPC 프라이빗 리소스 접근
   - Transit VIF: Transit Gateway 연결

3. Direct Connect Location
   - AWS 파트너 데이터센터
   - 물리적 연결 지점
   - 전 세계 주요 도시에 위치
```

### 연결 옵션 선택 기준

#### 상황별 최적 선택
```
VPN 선택 시나리오:
- 빠른 연결 구축이 필요한 경우
- 임시적 또는 테스트 목적
- 낮은 대역폭 요구사항 (< 1Gbps)
- 예산 제약이 있는 경우
- 지리적으로 분산된 여러 지점 연결

Direct Connect 선택 시나리오:
- 높은 대역폭 요구사항 (> 1Gbps)
- 일관된 네트워크 성능 필요
- 대용량 데이터 전송
- 미션 크리티컬 애플리케이션
- 장기적 안정적 연결 필요

하이브리드 접근:
- Direct Connect (주 연결) + VPN (백업)
- 최고의 성능과 가용성 확보
- 비용과 성능의 균형점
```

---

## 🛠️ Site-to-Site VPN 설정 실습 (60분)

### 실습 시나리오

#### 가상 온프레미스 환경 구성
```
목표: 온프레미스 데이터센터와 AWS VPC 연결 시뮬레이션

구성:
1. 온프레미스 시뮬레이션 VPC (On-Prem-VPC)
   - CIDR: 192.168.0.0/16
   - 퍼블릭 서브넷: 192.168.1.0/24
   - 프라이빗 서브넷: 192.168.10.0/24

2. AWS 클라우드 VPC (기존 SAA-Study-VPC)
   - CIDR: 10.0.0.0/16
   - 기존 서브넷 구성 활용

3. VPN 연결
   - Site-to-Site VPN으로 두 VPC 연결
   - 양방향 통신 테스트
```

### 1단계: 온프레미스 VPC 생성

#### 온프레미스 시뮬레이션 환경
```
1. VPC 생성
   - Name: OnPrem-Simulation-VPC
   - CIDR: 192.168.0.0/16
   - DNS resolution: Enable
   - DNS hostnames: Enable

2. 서브넷 생성
   - Public Subnet: 192.168.1.0/24 (AZ-a)
   - Private Subnet: 192.168.10.0/24 (AZ-a)

3. 인터넷 게이트웨이 연결
   - IGW 생성 및 VPC 연결
   - 퍼블릭 서브넷 라우팅 설정

4. NAT 게이트웨이 생성
   - 퍼블릭 서브넷에 NAT 게이트웨이 배치
   - 프라이빗 서브넷 인터넷 연결 설정
```

### 2단계: Customer Gateway 생성

#### Customer Gateway 설정
```
1. Customer Gateway 생성
   - VPC → Customer Gateways
   - "Create Customer Gateway" 클릭
   - Name: OnPrem-CGW
   - Routing: Dynamic (BGP 사용)
   - BGP ASN: 65000 (사설 ASN)
   - IP Address: [OnPrem VPC의 NAT Gateway 퍼블릭 IP]

2. 설정 확인
   - State: Available 확인
   - IP 주소 정확성 확인
```

### 3단계: Virtual Private Gateway 생성

#### VGW 생성 및 연결
```
1. Virtual Private Gateway 생성
   - VPC → Virtual Private Gateways
   - "Create Virtual Private Gateway" 클릭
   - Name: AWS-VGW
   - Amazon side ASN: 64512 (기본값)

2. VPC에 연결
   - 생성된 VGW 선택
   - Actions → "Attach to VPC"
   - VPC: SAA-Study-VPC 선택
   - "Attach to VPC" 클릭

3. 라우팅 전파 활성화
   - VPC → Route Tables
   - 프라이빗 라우팅 테이블들 선택
   - Route Propagation 탭
   - "Edit route propagation"
   - VGW 체크박스 활성화
```

### 4단계: VPN Connection 생성

#### Site-to-Site VPN 설정
```
1. VPN Connection 생성
   - VPC → Site-to-Site VPN Connections
   - "Create VPN Connection" 클릭
   - Name: OnPrem-to-AWS-VPN
   - Target Gateway Type: Virtual Private Gateway
   - Virtual Private Gateway: AWS-VGW
   - Customer Gateway: Existing → OnPrem-CGW
   - Routing Options: Dynamic (requires BGP)
   - Local IPv4 Network CIDR: 10.0.0.0/16
   - Remote IPv4 Network CIDR: 192.168.0.0/16

2. 터널 옵션 설정 (선택사항)
   - Pre-Shared Key: 자동 생성 또는 수동 입력
   - Advanced Options: 기본값 사용

3. VPN 생성 완료
   - "Create VPN Connection" 클릭
   - State: Available까지 대기 (약 5-10분)
```

### 5단계: 온프레미스 측 VPN 설정

#### strongSwan을 이용한 VPN 클라이언트 구성
```
1. 온프레미스 VPN 서버 인스턴스 생성
   - AMI: Amazon Linux 2023
   - Instance Type: t3.micro
   - VPC: OnPrem-Simulation-VPC
   - Subnet: Public Subnet
   - Security Group: VPN 트래픽 허용

2. strongSwan 설치 및 설정
   # strongSwan 설치
   sudo yum update -y
   sudo yum install -y strongswan

   # 설정 파일 다운로드 (AWS 콘솔에서 제공)
   # VPN Connection → Download Configuration
   # Vendor: Generic, Platform: Generic, Software: Vendor Agnostic

3. 설정 파일 적용
   # /etc/strongswan/ipsec.conf 편집
   # /etc/strongswan/ipsec.secrets 편집
   # AWS에서 제공한 설정 내용 적용

4. VPN 서비스 시작
   sudo systemctl enable strongswan
   sudo systemctl start strongswan
   sudo systemctl status strongswan
```

### 6단계: 연결 테스트 및 검증

#### VPN 터널 상태 확인
```
1. AWS 콘솔에서 터널 상태 확인
   - Site-to-Site VPN Connections
   - 생성한 VPN 선택
   - Tunnel Details 탭
   - Status: UP 확인 (최소 1개 터널)

2. 라우팅 테이블 확인
   - Route Tables에서 BGP 라우팅 전파 확인
   - 192.168.0.0/16 → vgw-xxx 경로 자동 추가 확인

3. 보안 그룹 설정
   - 양쪽 VPC의 보안 그룹에서 상대방 CIDR 허용
   - ICMP, SSH 등 테스트 트래픽 허용
```

#### 연결 테스트 수행
```
1. 인스턴스 간 통신 테스트
   # AWS VPC 인스턴스에서 온프레미스로 ping
   ping 192.168.10.5

   # 온프레미스 인스턴스에서 AWS로 ping
   ping 10.0.11.5

2. 애플리케이션 레벨 테스트
   # SSH 연결 테스트
   ssh ec2-user@192.168.10.5

   # HTTP 서비스 테스트
   curl http://192.168.10.5:80

3. 대역폭 및 지연시간 테스트
   # iperf3를 이용한 성능 테스트
   # 서버 측: iperf3 -s
   # 클라이언트 측: iperf3 -c [서버IP]
```

---

## 🏗️ 하이브리드 아키텍처 설계 (30분)

### 일반적인 하이브리드 패턴

#### 1. 확장형 패턴 (Extension Pattern)
```
구조:
온프레미스 데이터센터
├── 핵심 시스템 (ERP, 데이터베이스)
├── 레거시 애플리케이션
└── 기존 보안 인프라
    │
    │ VPN/Direct Connect
    │
AWS 클라우드
├── 웹 프론트엔드
├── API 게이트웨이
├── 마이크로서비스
└── 데이터 분석 플랫폼

특징:
- 기존 시스템은 온프레미스 유지
- 새로운 기능은 클라우드에서 개발
- 점진적 마이그레이션 가능
```

#### 2. 버스팅 패턴 (Bursting Pattern)
```
구조:
평상시: 온프레미스에서 모든 워크로드 처리
피크시: 클라우드로 트래픽 오버플로우

온프레미스 (기본 용량)
├── 웹 서버 (고정 용량)
├── 애플리케이션 서버
└── 데이터베이스
    │
    │ 로드 밸런서
    │
AWS 클라우드 (확장 용량)
├── Auto Scaling 웹 서버
├── 탄력적 애플리케이션 서버
└── 읽기 전용 복제본

특징:
- 비용 효율적 확장
- 피크 트래픽 대응
- 기존 투자 보호
```

#### 3. 재해 복구 패턴 (DR Pattern)
```
구조:
Primary Site (온프레미스)
├── 프로덕션 시스템
├── 실시간 데이터
└── 사용자 트래픽
    │
    │ 데이터 복제
    │
DR Site (AWS 클라우드)
├── 대기 상태 인프라
├── 복제된 데이터
└── 장애 시 활성화

특징:
- RTO/RPO 목표 달성
- 비용 효율적 DR 환경
- 자동화된 장애 조치
```

### 네트워크 설계 고려사항

#### 대역폭 계획
```
대역폭 요구사항 분석:

1. 일반적인 트래픽
   - 사용자 요청/응답
   - API 호출
   - 모니터링 데이터

2. 데이터 동기화
   - 데이터베이스 복제
   - 파일 동기화
   - 백업 데이터 전송

3. 피크 트래픽
   - 배치 작업
   - 대용량 파일 전송
   - 재해 복구 시나리오

권장 대역폭:
- 소규모: 100Mbps ~ 1Gbps
- 중간규모: 1Gbps ~ 10Gbps  
- 대규모: 10Gbps 이상
```

#### 보안 설계
```
하이브리드 보안 모델:

1. 네트워크 보안
   - VPN/Direct Connect 암호화
   - 네트워크 분할 및 격리
   - 트래픽 모니터링

2. 자격 증명 통합
   - Active Directory 연동
   - Single Sign-On (SSO)
   - 다중 인증 (MFA)

3. 데이터 보안
   - 전송 중 암호화
   - 저장 시 암호화
   - 키 관리 통합

4. 규정 준수
   - 데이터 거버넌스
   - 감사 로그 통합
   - 규정 준수 모니터링
```

---

## 📊 성능 최적화 및 모니터링 (15분)

### 하이브리드 연결 성능 최적화

#### VPN 성능 최적화
```
최적화 기법:

1. 터널 본딩
   - 여러 VPN 터널 동시 사용
   - 대역폭 증가 및 가용성 향상
   - ECMP (Equal Cost Multi-Path) 활용

2. 압축 및 최적화
   - 데이터 압축 활성화
   - TCP 최적화 설정
   - MTU 크기 조정

3. 라우팅 최적화
   - BGP 경로 최적화
   - 트래픽 엔지니어링
   - QoS 정책 적용
```

#### 모니터링 및 알림
```
핵심 모니터링 지표:

1. 연결 상태
   - VPN 터널 상태
   - BGP 세션 상태
   - 연결 가용성

2. 성능 지표
   - 대역폭 사용률
   - 지연시간
   - 패킷 손실률

3. 보안 지표
   - 인증 실패
   - 비정상 트래픽
   - 보안 이벤트

CloudWatch 알림 설정:
- VPN 터널 다운 시 즉시 알림
- 대역폭 임계값 초과 시 알림
- 지연시간 증가 시 알림
```

---

## 📝 핵심 정리

### 하이브리드 연결 선택 기준
1. **VPN**: 빠른 구축, 저비용, 가변 성능
2. **Direct Connect**: 고성능, 일관성, 높은 비용
3. **하이브리드**: VPN + Direct Connect로 최적화

### 설계 원칙
- **점진적 접근**: 작게 시작해서 단계적 확장
- **이중화**: 단일 장애점 제거
- **보안 우선**: 네트워크 및 데이터 보안 강화
- **모니터링**: 지속적인 성능 및 상태 모니터링

### 운영 고려사항
- **변경 관리**: 네트워크 변경 시 영향도 분석
- **재해 복구**: 연결 장애 시 대응 계획
- **비용 최적화**: 트래픽 패턴 기반 최적화
- **규정 준수**: 데이터 주권 및 보안 요구사항

---

## 🤔 토론 주제

1. **하이브리드 클라우드가 정말 필요할까?**
   - 완전 클라우드 vs 하이브리드 접근법

2. **VPN과 Direct Connect 중 어떤 것을 선택할까?**
   - 비용, 성능, 복잡성 관점에서

---

## 📚 다음 시간 예고

**일일 정리 & Q&A**
- VPC 네트워킹 개념 확인 퀴즈
- 3-tier 아키텍처 네트워크 설계 과제
- 실습 내용 종합 정리
- 질의응답 및 문제 해결

---

> 💡 **오늘의 핵심**: 하이브리드 연결은 온프레미스와 클라우드를 연결하는 다리입니다. 비즈니스 요구사항에 맞는 적절한 연결 방식을 선택하고, 보안과 성능을 모두 고려한 설계가 중요합니다. 작은 것부터 시작해서 점진적으로 확장하세요!