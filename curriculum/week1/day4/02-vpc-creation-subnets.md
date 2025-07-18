# Day 4-2: VPC 생성 및 서브넷 설계

## 📚 학습 목표
- 사용자 정의 VPC 생성 실습
- 퍼블릭/프라이빗 서브넷 구성 방법 습득
- 인터넷 게이트웨이 연결 및 설정
- 라우팅 테이블 생성 및 연결 실습

---

## 🏗️ VPC 생성 실습 (45분)

### 실습 시나리오
```
목표: 3-tier 웹 애플리케이션을 위한 VPC 구축

요구사항:
- 고가용성을 위한 다중 AZ 구성
- 웹 서버, 앱 서버, DB 서버 분리
- 보안을 위한 계층적 네트워크 설계
- 확장 가능한 IP 주소 체계

아키텍처:
퍼블릭 서브넷: 로드 밸런서, NAT 게이트웨이
프라이빗 서브넷: 웹/앱 서버
DB 서브넷: 데이터베이스
```

### 1단계: VPC 생성

#### 실습: 기본 VPC 생성
```
1. VPC 콘솔 접속
   - Services → VPC

2. VPC 생성 시작
   - "Create VPC" 클릭
   - "VPC only" 선택 (수동 구성을 위해)

3. VPC 기본 설정
   - Name tag: SAA-Study-VPC
   - IPv4 CIDR block: 10.0.0.0/16
   - IPv6 CIDR block: No IPv6 CIDR block
   - Tenancy: Default

4. VPC 생성 완료
   - "Create VPC" 클릭
   - VPC ID 확인 및 기록
```

#### VPC 기본 구성 요소 확인
```
자동 생성되는 요소들:
1. 기본 라우팅 테이블
   - 로컬 라우팅 (10.0.0.0/16 → local)
   
2. 기본 네트워크 ACL
   - 모든 트래픽 허용 (기본값)
   
3. 기본 보안 그룹
   - 아웃바운드: 모든 트래픽 허용
   - 인바운드: 동일 보안 그룹 내에서만 허용

주의사항:
- 인터넷 게이트웨이는 자동 생성되지 않음
- 서브넷도 별도로 생성해야 함
```

### 2단계: 인터넷 게이트웨이 생성 및 연결

#### 실습: 인터넷 게이트웨이 설정
```
1. 인터넷 게이트웨이 생성
   - VPC → Internet Gateways
   - "Create internet gateway" 클릭
   - Name tag: SAA-Study-IGW
   - "Create internet gateway" 클릭

2. VPC에 연결
   - 생성된 IGW 선택
   - Actions → "Attach to VPC"
   - VPC 선택: SAA-Study-VPC
   - "Attach internet gateway" 클릭

3. 연결 상태 확인
   - State: Attached 확인
   - VPC ID 일치 확인
```

---

## 🏢 서브넷 생성 및 구성 (60분)

### 서브넷 설계 계획

#### IP 주소 할당 계획
```
VPC CIDR: 10.0.0.0/16 (65,536개 IP)

퍼블릭 서브넷 (각 AZ당 256개 IP):
- AZ-a: 10.0.1.0/24 (10.0.1.1 ~ 10.0.1.254)
- AZ-b: 10.0.2.0/24 (10.0.2.1 ~ 10.0.2.254)
- AZ-c: 10.0.3.0/24 (10.0.3.1 ~ 10.0.3.254)

프라이빗 서브넷 (각 AZ당 256개 IP):
- AZ-a: 10.0.11.0/24 (10.0.11.1 ~ 10.0.11.254)
- AZ-b: 10.0.12.0/24 (10.0.12.1 ~ 10.0.12.254)
- AZ-c: 10.0.13.0/24 (10.0.13.1 ~ 10.0.13.254)

DB 서브넷 (각 AZ당 256개 IP):
- AZ-a: 10.0.21.0/24 (10.0.21.1 ~ 10.0.21.254)
- AZ-b: 10.0.22.0/24 (10.0.22.1 ~ 10.0.22.254)
- AZ-c: 10.0.23.0/24 (10.0.23.1 ~ 10.0.23.254)
```

### 실습 1: 퍼블릭 서브넷 생성

#### AZ-a 퍼블릭 서브넷
```
1. 서브넷 생성 시작
   - VPC → Subnets
   - "Create subnet" 클릭

2. 서브넷 설정
   - VPC ID: SAA-Study-VPC 선택
   - Subnet name: Public-Subnet-AZ-A
   - Availability Zone: ap-northeast-2a
   - IPv4 CIDR block: 10.0.1.0/24

3. 퍼블릭 IP 자동 할당 설정
   - 생성된 서브넷 선택
   - Actions → "Modify auto-assign IP settings"
   - "Enable auto-assign public IPv4 address" 체크
   - "Save" 클릭
```

#### AZ-b, AZ-c 퍼블릭 서브넷
```
동일한 방식으로 생성:

AZ-b 퍼블릭 서브넷:
- Name: Public-Subnet-AZ-B
- AZ: ap-northeast-2b
- CIDR: 10.0.2.0/24
- 퍼블릭 IP 자동 할당: 활성화

AZ-c 퍼블릭 서브넷:
- Name: Public-Subnet-AZ-C
- AZ: ap-northeast-2c
- CIDR: 10.0.3.0/24
- 퍼블릭 IP 자동 할당: 활성화
```

### 실습 2: 프라이빗 서브넷 생성

#### 프라이빗 서브넷 생성 (3개 AZ)
```
AZ-a 프라이빗 서브넷:
- Name: Private-Subnet-AZ-A
- AZ: ap-northeast-2a
- CIDR: 10.0.11.0/24
- 퍼블릭 IP 자동 할당: 비활성화 (기본값)

AZ-b 프라이빗 서브넷:
- Name: Private-Subnet-AZ-B
- AZ: ap-northeast-2b
- CIDR: 10.0.12.0/24

AZ-c 프라이빗 서브넷:
- Name: Private-Subnet-AZ-C
- AZ: ap-northeast-2c
- CIDR: 10.0.13.0/24
```

### 실습 3: DB 서브넷 생성

#### DB 서브넷 생성 (3개 AZ)
```
AZ-a DB 서브넷:
- Name: DB-Subnet-AZ-A
- AZ: ap-northeast-2a
- CIDR: 10.0.21.0/24

AZ-b DB 서브넷:
- Name: DB-Subnet-AZ-B
- AZ: ap-northeast-2b
- CIDR: 10.0.22.0/24

AZ-c DB 서브넷:
- Name: DB-Subnet-AZ-C
- AZ: ap-northeast-2c
- CIDR: 10.0.23.0/24
```

---

## 🛣️ 라우팅 테이블 설정 (45분)

### 라우팅 테이블 설계 전략

#### 라우팅 테이블 구성 계획
```
1. 퍼블릭 라우팅 테이블
   - 인터넷 게이트웨이로 라우팅
   - 퍼블릭 서브넷들과 연결

2. 프라이빗 라우팅 테이블 (AZ별)
   - NAT 게이트웨이로 라우팅
   - 각 AZ의 프라이빗 서브넷과 연결

3. DB 라우팅 테이블
   - 인터넷 접근 차단
   - DB 서브넷들과 연결
```

### 실습 1: 퍼블릭 라우팅 테이블 생성

#### 퍼블릭 라우팅 테이블 설정
```
1. 라우팅 테이블 생성
   - VPC → Route Tables
   - "Create route table" 클릭
   - Name: Public-Route-Table
   - VPC: SAA-Study-VPC 선택
   - "Create route table" 클릭

2. 인터넷 라우팅 추가
   - 생성된 라우팅 테이블 선택
   - Routes 탭 → "Edit routes"
   - "Add route" 클릭
   - Destination: 0.0.0.0/0
   - Target: Internet Gateway → SAA-Study-IGW 선택
   - "Save changes" 클릭

3. 서브넷 연결
   - Subnet associations 탭 → "Edit subnet associations"
   - 퍼블릭 서브넷 3개 모두 선택:
     * Public-Subnet-AZ-A
     * Public-Subnet-AZ-B  
     * Public-Subnet-AZ-C
   - "Save associations" 클릭
```

### 실습 2: 프라이빗 라우팅 테이블 생성

#### AZ-A 프라이빗 라우팅 테이블
```
1. 라우팅 테이블 생성
   - Name: Private-Route-Table-AZ-A
   - VPC: SAA-Study-VPC

2. 서브넷 연결
   - Private-Subnet-AZ-A 연결
   
참고: NAT 게이트웨이는 다음 시간에 생성 후 라우팅 추가
```

#### AZ-B, AZ-C 프라이빗 라우팅 테이블
```
동일한 방식으로 생성:

AZ-B:
- Name: Private-Route-Table-AZ-B
- 연결: Private-Subnet-AZ-B

AZ-C:
- Name: Private-Route-Table-AZ-C
- 연결: Private-Subnet-AZ-C
```

### 실습 3: DB 라우팅 테이블 생성

#### DB 전용 라우팅 테이블
```
1. 라우팅 테이블 생성
   - Name: DB-Route-Table
   - VPC: SAA-Study-VPC

2. 서브넷 연결
   - DB 서브넷 3개 모두 연결:
     * DB-Subnet-AZ-A
     * DB-Subnet-AZ-B
     * DB-Subnet-AZ-C

3. 라우팅 확인
   - 기본 로컬 라우팅만 존재 (10.0.0.0/16 → local)
   - 인터넷 라우팅 없음 (보안 강화)
```

---

## 🧪 네트워크 연결 테스트 (30분)

### 테스트 인스턴스 생성

#### 퍼블릭 서브넷 테스트 인스턴스
```
1. EC2 인스턴스 시작
   - AMI: Amazon Linux 2023
   - Instance type: t3.micro
   - Network: SAA-Study-VPC
   - Subnet: Public-Subnet-AZ-A
   - Auto-assign Public IP: Enable

2. 보안 그룹 설정
   - Name: Public-Test-SG
   - Inbound rules:
     * SSH (22): My IP
     * HTTP (80): 0.0.0.0/0
     * ICMP: 0.0.0.0/0 (ping 테스트용)

3. 키 페어 설정
   - 기존 키 페어 선택 또는 새로 생성
```

#### 프라이빗 서브넷 테스트 인스턴스
```
1. EC2 인스턴스 시작
   - AMI: Amazon Linux 2023
   - Instance type: t3.micro
   - Network: SAA-Study-VPC
   - Subnet: Private-Subnet-AZ-A
   - Auto-assign Public IP: Disable

2. 보안 그룹 설정
   - Name: Private-Test-SG
   - Inbound rules:
     * SSH (22): 10.0.0.0/16 (VPC 내부에서만)
     * ICMP: 10.0.0.0/16

3. 동일한 키 페어 사용
```

### 연결 테스트 수행

#### 1. 퍼블릭 인스턴스 접근 테스트
```
1. SSH 접속 테스트
   ssh -i your-key.pem ec2-user@[퍼블릭-IP]

2. 인터넷 연결 테스트
   ping 8.8.8.8
   curl http://httpbin.org/ip

3. VPC 내부 통신 테스트
   ping [프라이빗-인스턴스-IP]
```

#### 2. 프라이빗 인스턴스 접근 테스트
```
1. 퍼블릭 인스턴스를 통한 접속 (점프 서버)
   # 퍼블릭 인스턴스에서 실행
   ssh -i your-key.pem ec2-user@[프라이빗-IP]

2. 인터넷 연결 테스트 (실패해야 정상)
   ping 8.8.8.8  # 실패 예상
   curl http://httpbin.org/ip  # 실패 예상

3. VPC 내부 통신 테스트
   ping [퍼블릭-인스턴스-IP]  # 성공
```

---

## 📊 VPC 구성 검증 (15분)

### 네트워크 구성 확인 체크리스트

#### VPC 레벨 확인
```
✅ VPC 생성 확인
- VPC ID: vpc-xxxxxxxxx
- CIDR: 10.0.0.0/16
- State: Available

✅ 인터넷 게이트웨이 연결 확인
- IGW ID: igw-xxxxxxxxx
- State: Attached
- VPC 연결 상태 확인
```

#### 서브넷 레벨 확인
```
✅ 서브넷 생성 확인 (총 9개)
퍼블릭 서브넷 (3개):
- 10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24
- 퍼블릭 IP 자동 할당: 활성화

프라이빗 서브넷 (3개):
- 10.0.11.0/24, 10.0.12.0/24, 10.0.13.0/24
- 퍼블릭 IP 자동 할당: 비활성화

DB 서브넷 (3개):
- 10.0.21.0/24, 10.0.22.0/24, 10.0.23.0/24
- 퍼블릭 IP 자동 할당: 비활성화
```

#### 라우팅 테이블 확인
```
✅ 라우팅 테이블 생성 확인 (총 5개)
1. 기본 라우팅 테이블 (사용 안 함)
2. Public-Route-Table
   - 0.0.0.0/0 → IGW
   - 퍼블릭 서브넷 3개 연결
3. Private-Route-Table-AZ-A/B/C (3개)
   - 로컬 라우팅만 (NAT는 다음 시간에 추가)
4. DB-Route-Table
   - 로컬 라우팅만
   - DB 서브넷 3개 연결
```

### 네트워크 다이어그램 확인

#### 현재 구성된 아키텍처
```
Internet
    |
Internet Gateway (IGW)
    |
┌─────────────────────────────────────────────────────────┐
│                    VPC (10.0.0.0/16)                   │
├─────────────────────────────────────────────────────────┤
│  Public Subnets                                        │
│  ┌─────────────┬─────────────┬─────────────┐            │
│  │10.0.1.0/24  │10.0.2.0/24  │10.0.3.0/24  │            │
│  │   AZ-A      │   AZ-B      │   AZ-C      │            │
│  └─────────────┴─────────────┴─────────────┘            │
├─────────────────────────────────────────────────────────┤
│  Private Subnets                                       │
│  ┌─────────────┬─────────────┬─────────────┐            │
│  │10.0.11.0/24 │10.0.12.0/24 │10.0.13.0/24 │            │
│  │   AZ-A      │   AZ-B      │   AZ-C      │            │
│  └─────────────┴─────────────┴─────────────┘            │
├─────────────────────────────────────────────────────────┤
│  DB Subnets                                            │
│  ┌─────────────┬─────────────┬─────────────┐            │
│  │10.0.21.0/24 │10.0.22.0/24 │10.0.23.0/24 │            │
│  │   AZ-A      │   AZ-B      │   AZ-C      │            │
│  └─────────────┴─────────────┴─────────────┘            │
└─────────────────────────────────────────────────────────┘
```

---

## 📝 핵심 정리

### VPC 생성 핵심 단계
1. **VPC 생성**: CIDR 블록 설정 및 기본 구성
2. **IGW 연결**: 인터넷 접근을 위한 게이트웨이
3. **서브넷 생성**: 용도별, AZ별 네트워크 분할
4. **라우팅 설정**: 트래픽 흐름 제어

### 서브넷 설계 원칙
- **다중 AZ**: 고가용성을 위한 분산 배치
- **계층적 분리**: 퍼블릭 → 프라이빗 → DB 순서
- **IP 체계**: 체계적인 CIDR 할당
- **보안 고려**: 각 계층별 적절한 접근 제어

### 라우팅 전략
- **퍼블릭**: 인터넷 게이트웨이 직접 연결
- **프라이빗**: NAT 게이트웨이 경유 (다음 시간)
- **DB**: 외부 접근 완전 차단
- **로컬**: VPC 내부 통신은 항상 로컬 라우팅

---

## 🤔 토론 주제

1. **서브넷을 AZ별로 나누는 이유는?**
   - 고가용성과 장애 격리 관점에서

2. **프라이빗 서브넷이 정말 필요할까?**
   - 보안 vs 관리 복잡성 측면에서

---

## 📚 다음 시간 예고

**라우팅 및 NAT**
- NAT 게이트웨이 vs NAT 인스턴스 비교
- 프라이빗 서브넷 인터넷 연결 설정
- 라우팅 테이블 고급 설정
- 네트워크 트래픽 흐름 분석

---

> 💡 **오늘의 핵심**: VPC는 클라우드 네트워킹의 기초입니다. 체계적인 CIDR 설계와 용도별 서브넷 분리, 그리고 적절한 라우팅 설정이 안전하고 확장 가능한 네트워크의 핵심입니다!