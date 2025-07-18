# Day 4-3: 라우팅 및 NAT

## 📚 학습 목표
- 라우팅 테이블 설정 원리 이해
- NAT 게이트웨이와 NAT 인스턴스 비교 분석
- 프라이빗 서브넷 인터넷 연결 구성
- 네트워크 트래픽 흐름 분석 및 최적화

---

## 🛣️ 라우팅 테이블 심화 (45분)

### 라우팅의 작동 원리

#### 라우팅 결정 과정
```
패킷 전송 시 라우팅 결정:
1. 목적지 IP 주소 확인
2. 라우팅 테이블에서 매칭되는 경로 검색
3. 가장 구체적인 경로 선택 (Longest Prefix Match)
4. 해당 타겟으로 패킷 전송

예시:
목적지: 10.0.11.5
라우팅 테이블:
- 10.0.0.0/16 → local (매칭)
- 10.0.11.0/24 → local (더 구체적, 선택됨)
- 0.0.0.0/0 → igw-xxx (매칭되지만 덜 구체적)
```

#### 라우팅 우선순위
```
우선순위 순서 (높음 → 낮음):
1. /32 (호스트 라우팅)
2. /24, /20, /16 등 (서브넷 라우팅)
3. /0 (기본 라우팅)

실제 예시:
- 10.0.1.100/32 → 특정 서버로 직접 라우팅
- 10.0.1.0/24 → 서브넷 라우팅
- 10.0.0.0/16 → VPC 로컬 라우팅
- 0.0.0.0/0 → 기본 게이트웨이
```

### 라우팅 테이블 고급 설정

#### 실습: 세밀한 라우팅 제어
```
시나리오: 특정 서비스만 다른 경로로 라우팅

1. 관리용 서브넷 생성
   - CIDR: 10.0.100.0/24
   - 용도: 관리 서버, 모니터링 도구

2. 관리용 라우팅 테이블 생성
   - Name: Management-Route-Table
   - 특별한 라우팅 규칙 적용

3. 세밀한 라우팅 규칙
   - 10.0.0.0/16 → local (VPC 내부)
   - 172.16.0.0/12 → vpn-xxx (본사 네트워크)
   - 0.0.0.0/0 → igw-xxx (일반 인터넷)
```

#### 라우팅 테이블 모범 사례
```
설계 원칙:
1. 최소 권한 원칙
   - 필요한 경로만 추가
   - 불필요한 라우팅 제거

2. 명확한 명명 규칙
   - Public-RT, Private-RT-AZ-A 등
   - 용도와 위치를 명시

3. 문서화
   - 각 라우팅 규칙의 목적 기록
   - 변경 이력 관리

4. 정기적 검토
   - 사용하지 않는 라우팅 정리
   - 보안 정책 준수 확인
```

---

## 🌐 NAT 게이트웨이 vs NAT 인스턴스 (45분)

### NAT의 필요성

#### 프라이빗 서브넷의 딜레마
```
문제 상황:
- 프라이빗 서브넷 = 보안을 위해 인터넷 직접 접근 차단
- 하지만 소프트웨어 업데이트, API 호출 등은 필요
- 인바운드는 차단하되 아웃바운드는 허용해야 함

해결책: NAT (Network Address Translation)
- 프라이빗 IP → 퍼블릭 IP 변환
- 아웃바운드 연결만 허용
- 인바운드 연결은 차단 (보안 유지)
```

#### NAT 작동 원리
```
아웃바운드 트래픽 흐름:
1. 프라이빗 서버 (10.0.11.5) → 외부 API (8.8.8.8)
2. NAT 게이트웨이에서 IP 변환
   - Source: 10.0.11.5 → NAT의 퍼블릭 IP
   - Destination: 8.8.8.8 (그대로)
3. 인터넷으로 전송
4. 응답 패킷은 NAT를 통해 원래 서버로 전달

보안 효과:
- 외부에서는 프라이빗 서버 IP를 알 수 없음
- 직접적인 인바운드 연결 불가능
- 연결 상태 추적으로 관련 응답만 허용
```

### NAT 게이트웨이 (AWS 관리형)

#### NAT 게이트웨이 특징
```
장점:
✅ AWS 완전 관리형 서비스
✅ 고가용성 자동 보장
✅ 자동 스케일링 (최대 45Gbps)
✅ 패치 및 유지보수 불필요
✅ 보안 그룹 설정 불필요

단점:
❌ 시간당 고정 비용 ($0.045/시간)
❌ 데이터 처리 비용 ($0.045/GB)
❌ 커스터마이징 제한
❌ 포트 포워딩 불가능
```

#### 실습: NAT 게이트웨이 생성
```
1. NAT 게이트웨이 생성
   - VPC → NAT Gateways
   - "Create NAT gateway" 클릭
   - Name: SAA-Study-NAT-AZ-A
   - Subnet: Public-Subnet-AZ-A (퍼블릭 서브넷에 생성)
   - Connectivity type: Public
   - Elastic IP allocation: "Allocate Elastic IP" 클릭

2. 추가 NAT 게이트웨이 생성 (고가용성)
   - Name: SAA-Study-NAT-AZ-B
   - Subnet: Public-Subnet-AZ-B
   - 새로운 Elastic IP 할당

3. 생성 확인
   - State: Available 확인
   - Elastic IP 할당 확인
```

### NAT 인스턴스 (사용자 관리형)

#### NAT 인스턴스 특징
```
장점:
✅ 비용 효율적 (인스턴스 비용만)
✅ 완전한 커스터마이징 가능
✅ 포트 포워딩, VPN 서버 등 추가 기능
✅ 보안 그룹으로 세밀한 제어

단점:
❌ 직접 관리 필요 (패치, 업데이트)
❌ 단일 장애점 (고가용성 직접 구현)
❌ 성능 제한 (인스턴스 타입에 의존)
❌ 복잡한 설정 및 운영
```

#### 실습: NAT 인스턴스 구성 (참고용)
```
1. NAT 전용 AMI 선택
   - Community AMIs 검색: "amzn-ami-vpc-nat"
   - 또는 일반 Amazon Linux에서 직접 구성

2. 인스턴스 설정
   - Instance type: t3.micro (테스트용)
   - Subnet: Public-Subnet-AZ-A
   - Auto-assign Public IP: Enable
   - Source/Destination Check: Disable (중요!)

3. 보안 그룹 설정
   - Name: NAT-Instance-SG
   - Inbound: 프라이빗 서브넷에서의 모든 트래픽
   - Outbound: 모든 트래픽 허용

4. NAT 기능 활성화 (Amazon Linux)
   sudo sysctl -w net.ipv4.ip_forward=1
   sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

### NAT 비교 및 선택 기준

#### 상황별 선택 가이드
```
NAT 게이트웨이 선택 시:
- 운영 부담을 최소화하고 싶은 경우
- 고가용성이 중요한 프로덕션 환경
- 높은 네트워크 성능이 필요한 경우
- AWS 관리형 서비스를 선호하는 경우

NAT 인스턴스 선택 시:
- 비용 최적화가 중요한 경우
- 특별한 네트워크 기능이 필요한 경우
- 완전한 제어권이 필요한 경우
- 개발/테스트 환경에서 실험하는 경우
```

---

## 🔧 프라이빗 서브넷 인터넷 연결 설정 (45분)

### 라우팅 테이블 업데이트

#### 실습: NAT 게이트웨이 라우팅 추가
```
1. AZ-A 프라이빗 라우팅 테이블 수정
   - Route Tables → Private-Route-Table-AZ-A 선택
   - Routes 탭 → "Edit routes"
   - "Add route" 클릭
   - Destination: 0.0.0.0/0
   - Target: NAT Gateway → SAA-Study-NAT-AZ-A 선택
   - "Save changes" 클릭

2. AZ-B 프라이빗 라우팅 테이블 수정
   - Private-Route-Table-AZ-B
   - 0.0.0.0/0 → SAA-Study-NAT-AZ-B

3. AZ-C 프라이빗 라우팅 테이블 수정
   - Private-Route-Table-AZ-C
   - 0.0.0.0/0 → SAA-Study-NAT-AZ-A (또는 AZ-B)
   - 비용 절약을 위해 하나의 NAT 공유 가능
```

#### 고가용성 vs 비용 최적화
```
고가용성 우선 (권장):
AZ-A Private → NAT-AZ-A
AZ-B Private → NAT-AZ-B
AZ-C Private → NAT-AZ-C
- 각 AZ별 독립적인 NAT
- 한 AZ 장애 시 다른 AZ 영향 없음
- 비용: NAT 게이트웨이 3개

비용 최적화:
AZ-A Private → NAT-AZ-A
AZ-B Private → NAT-AZ-A
AZ-C Private → NAT-AZ-A
- 하나의 NAT로 모든 트래픽 처리
- NAT AZ 장애 시 모든 프라이빗 서브넷 영향
- 비용: NAT 게이트웨이 1개
```

### 연결 테스트 및 검증

#### 실습: 프라이빗 인스턴스 인터넷 연결 테스트
```
1. 프라이빗 인스턴스 접속
   # 퍼블릭 인스턴스를 통해 접속
   ssh -i key.pem ec2-user@[퍼블릭-IP]
   ssh -i key.pem ec2-user@[프라이빗-IP]

2. 인터넷 연결 테스트
   # DNS 해석 테스트
   nslookup google.com
   
   # 외부 연결 테스트
   ping 8.8.8.8
   curl http://httpbin.org/ip
   
   # 패키지 업데이트 테스트
   sudo yum update -y

3. 연결 상태 확인
   # 현재 연결 상태 확인
   netstat -rn
   
   # NAT 게이트웨이 IP 확인
   curl http://checkip.amazonaws.com
```

#### 트래픽 흐름 분석
```
아웃바운드 트래픽 경로:
프라이빗 인스턴스 (10.0.11.5)
↓
프라이빗 서브넷 라우팅 테이블
↓
NAT 게이트웨이 (10.0.1.100)
↓
퍼블릭 서브넷 라우팅 테이블
↓
인터넷 게이트웨이
↓
인터넷

응답 트래픽 경로:
인터넷
↓
인터넷 게이트웨이
↓
NAT 게이트웨이 (연결 상태 추적)
↓
원래 프라이빗 인스턴스
```

---

## 📊 네트워크 성능 최적화 (30분)

### NAT 게이트웨이 성능 특성

#### 성능 지표 이해
```
NAT 게이트웨이 성능:
- 대역폭: 최대 45Gbps
- 동시 연결: 최대 55,000개
- 패킷 처리: 초당 수백만 패킷
- 지연시간: 매우 낮음 (< 1ms)

성능에 영향을 주는 요소:
- 동시 연결 수
- 패킷 크기
- 연결 지속 시간
- 대상 서비스의 응답 시간
```

#### 성능 모니터링
```
CloudWatch 메트릭:
- BytesInFromDestination: 인터넷에서 받은 바이트
- BytesInFromSource: 프라이빗 서브넷에서 받은 바이트
- BytesOutToDestination: 인터넷으로 보낸 바이트
- BytesOutToSource: 프라이빗 서브넷으로 보낸 바이트
- ActiveConnectionCount: 활성 연결 수
- ConnectionAttemptCount: 연결 시도 수
- ConnectionEstablishedCount: 성공한 연결 수
- ErrorPortAllocation: 포트 할당 오류
- PacketsDropCount: 드롭된 패킷 수
```

### 비용 최적화 전략

#### NAT 비용 구조 이해
```
NAT 게이트웨이 비용 (서울 리전):
- 시간당 요금: $0.059
- 데이터 처리 요금: $0.059/GB
- 월 예상 비용 (24시간 운영): ~$43

비용 절약 방법:
1. 필요한 AZ에만 NAT 배치
2. 트래픽 패턴 분석 후 최적화
3. VPC 엔드포인트 활용 (AWS 서비스 접근)
4. 개발 환경에서는 NAT 인스턴스 고려
```

#### VPC 엔드포인트를 통한 비용 절약
```
VPC 엔드포인트 활용 시나리오:
- S3 접근: Gateway Endpoint (무료)
- EC2, SSM 등: Interface Endpoint (유료)
- DynamoDB: Gateway Endpoint (무료)

예시: S3 접근 최적화
기존: 프라이빗 서브넷 → NAT → 인터넷 → S3
최적화: 프라이빗 서브넷 → S3 Gateway Endpoint → S3
- NAT 트래픽 비용 절약
- 더 빠른 접근 속도
- 보안 향상 (인터넷 경유 없음)
```

---

## 🔍 네트워크 트래픽 분석 (15분)

### VPC Flow Logs 활용

#### Flow Logs 설정
```
1. VPC Flow Logs 활성화
   - VPC → Your VPCs → SAA-Study-VPC 선택
   - Flow logs 탭 → "Create flow log"
   - Filter: All (Accept, Reject 모두)
   - Destination: CloudWatch Logs
   - Log group: /aws/vpc/flowlogs

2. 로그 분석
   - CloudWatch → Logs → Log groups
   - /aws/vpc/flowlogs 선택
   - 실시간 트래픽 패턴 확인
```

#### 트래픽 패턴 분석
```
Flow Log 필드 이해:
- srcaddr: 소스 IP 주소
- dstaddr: 목적지 IP 주소
- srcport: 소스 포트
- dstport: 목적지 포트
- protocol: 프로토콜 (6=TCP, 17=UDP)
- action: ACCEPT 또는 REJECT

분석 예시:
10.0.11.5 → 8.8.8.8:53 (DNS 쿼리)
10.0.11.5 → 52.95.110.1:443 (S3 접근)
10.0.11.5 → 169.254.169.254:80 (메타데이터 서비스)
```

---

## 📝 핵심 정리

### 라우팅 핵심 원칙
1. **Longest Prefix Match**: 가장 구체적인 경로 우선
2. **로컬 라우팅**: VPC 내부 통신은 항상 로컬
3. **기본 라우팅**: 0.0.0.0/0으로 모든 외부 트래픽 처리
4. **라우팅 테이블 연결**: 서브넷별 적절한 라우팅 테이블 연결

### NAT 선택 기준
- **NAT 게이트웨이**: 프로덕션, 고가용성, 관리 편의성
- **NAT 인스턴스**: 개발/테스트, 비용 최적화, 커스터마이징

### 네트워크 보안
- **아웃바운드만 허용**: NAT를 통한 단방향 통신
- **연결 상태 추적**: 관련 응답만 허용
- **최소 권한**: 필요한 포트와 프로토콜만 허용

---

## 🤔 토론 주제

1. **모든 AZ에 NAT 게이트웨이가 필요할까?**
   - 고가용성 vs 비용 효율성

2. **VPC 엔드포인트를 언제 사용해야 할까?**
   - NAT 비용 vs 엔드포인트 비용 비교

---

## 📚 다음 시간 예고

**VPC 연결 옵션**
- VPC 피어링 설정 및 활용
- Transit Gateway 개념 및 구성
- 멀티 VPC 아키텍처 설계
- 네트워크 연결 최적화 전략

---

> 💡 **오늘의 핵심**: NAT는 프라이빗 서브넷의 보안을 유지하면서 필요한 외부 통신을 가능하게 하는 핵심 구성 요소입니다. 라우팅 테이블을 통해 트래픽 흐름을 정확히 제어하는 것이 안전한 네트워크 설계의 기본입니다!