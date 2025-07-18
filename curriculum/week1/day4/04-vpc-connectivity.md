# Day 4-4: VPC 연결 옵션

## 📚 학습 목표
- VPC 피어링의 개념과 구성 방법 이해
- Transit Gateway를 통한 중앙 집중식 연결 관리
- 멀티 VPC 아키텍처 설계 원칙 습득
- 네트워크 연결 옵션별 특징과 사용 사례 파악

---

## 🔗 VPC 피어링 기초 (45분)

### VPC 피어링이란?

#### 피어링의 개념
```
VPC 피어링 = 두 VPC 간의 직접 연결
- 마치 같은 네트워크처럼 통신 가능
- AWS 백본 네트워크를 통한 프라이빗 연결
- 인터넷을 경유하지 않는 안전한 통신

비유: 아파트 단지 간 전용 통로
- A단지와 B단지를 직접 연결하는 다리
- 외부 도로를 거치지 않고 직접 이동
- 각 단지의 보안은 그대로 유지
```

#### 피어링 연결 특징
```
연결 특성:
✅ 1:1 연결 (Point-to-Point)
✅ 양방향 통신 가능
✅ 프라이빗 IP로 직접 통신
✅ 지연시간 최소화
✅ 데이터 전송 비용 절약

제한사항:
❌ 전이적 라우팅 불가 (A-B-C 연결 시 A-C 직접 통신 불가)
❌ CIDR 블록 중복 불가
❌ 복잡한 네트워크에서 관리 어려움
❌ 피어링 연결 수 제한 (VPC당 최대 125개)
```

### VPC 피어링 사용 사례

#### 일반적인 활용 시나리오
```
1. 개발/스테이징/프로덕션 환경 분리
   Dev VPC ←→ Staging VPC ←→ Prod VPC
   - 환경별 독립성 유지
   - 필요시 데이터 공유

2. 서비스별 VPC 분리
   Web VPC ←→ API VPC ←→ DB VPC
   - 마이크로서비스 아키텍처
   - 서비스별 보안 정책 적용

3. 계정 간 리소스 공유
   Account A VPC ←→ Account B VPC
   - 조직 내 부서별 계정 분리
   - 공통 서비스 공유 (DNS, 모니터링)

4. 지역별 VPC 연결
   Seoul VPC ←→ Tokyo VPC
   - 글로벌 서비스 구축
   - 재해 복구 환경 구성
```

### 실습: VPC 피어링 설정

#### 1단계: 두 번째 VPC 생성
```
1. 새 VPC 생성
   - Name: SAA-Study-VPC-B
   - CIDR: 172.16.0.0/16 (기존 VPC와 중복 방지)
   - DNS resolution: Enable
   - DNS hostnames: Enable

2. 서브넷 생성
   - Public Subnet: 172.16.1.0/24
   - Private Subnet: 172.16.11.0/24

3. 인터넷 게이트웨이 연결
   - IGW 생성 및 VPC-B 연결
   - 퍼블릭 서브넷 라우팅 설정
```

#### 2단계: 피어링 연결 생성
```
1. 피어링 연결 요청
   - VPC → Peering Connections
   - "Create Peering Connection" 클릭
   - Name: SAA-Study-Peering
   - VPC (Requester): SAA-Study-VPC (10.0.0.0/16)
   - VPC (Accepter): SAA-Study-VPC-B (172.16.0.0/16)
   - "Create Peering Connection" 클릭

2. 피어링 연결 수락
   - 생성된 피어링 연결 선택
   - Actions → "Accept Request"
   - "Accept Request" 클릭
   - Status: Active 확인
```

#### 3단계: 라우팅 테이블 업데이트
```
1. VPC-A 라우팅 테이블 수정
   - Private-Route-Table-AZ-A 선택
   - Routes → "Edit routes"
   - Add route:
     * Destination: 172.16.0.0/16
     * Target: Peering Connection → SAA-Study-Peering
   - "Save changes"

2. VPC-B 라우팅 테이블 수정
   - VPC-B의 프라이빗 라우팅 테이블 선택
   - Add route:
     * Destination: 10.0.0.0/16
     * Target: Peering Connection → SAA-Study-Peering
```

#### 4단계: 보안 그룹 설정
```
1. VPC-A 보안 그룹 수정
   - Private-Test-SG 선택
   - Inbound rules → "Edit inbound rules"
   - Add rule:
     * Type: All ICMP - IPv4
     * Source: 172.16.0.0/16
     * Description: From VPC-B

2. VPC-B 보안 그룹 수정
   - 동일하게 10.0.0.0/16에서의 접근 허용
```

---

## 🌐 Transit Gateway 개념 (45분)

### Transit Gateway란?

#### 중앙 집중식 연결 허브
```
Transit Gateway = 네트워크 허브
- 여러 VPC, VPN, Direct Connect를 중앙에서 관리
- 스타 토폴로지로 복잡성 해결
- 확장 가능하고 관리하기 쉬운 구조

기존 피어링 문제:
VPC-A ←→ VPC-B
  ↕        ↕
VPC-D ←→ VPC-C
- N개 VPC 연결 시 N(N-1)/2개 피어링 필요
- 복잡한 라우팅 관리
- 전이적 라우팅 불가

Transit Gateway 해결:
    VPC-A
      ↑
VPC-D ← TGW → VPC-B
      ↓
    VPC-C
- 모든 VPC가 TGW를 통해 연결
- 중앙 집중식 라우팅 관리
- 전이적 라우팅 지원
```

#### Transit Gateway 특징
```
장점:
✅ 중앙 집중식 관리
✅ 확장성 (최대 5,000개 VPC 연결)
✅ 전이적 라우팅 지원
✅ 멀티캐스트 지원
✅ VPN, Direct Connect 통합 연결
✅ 리전 간 피어링 지원

고려사항:
⚠️ 추가 비용 발생 (시간당 + 데이터 처리)
⚠️ 약간의 지연시간 추가
⚠️ 복잡한 라우팅 정책 설정 가능
```

### Transit Gateway 구성 요소

#### 핵심 구성 요소
```
1. Transit Gateway
   - 중앙 라우팅 허브
   - 리전별로 생성

2. Attachment
   - VPC, VPN, Direct Connect 연결점
   - 각 연결마다 하나의 Attachment

3. Route Table
   - 트래픽 라우팅 규칙 정의
   - 여러 Route Table로 세밀한 제어 가능

4. Propagation
   - 동적 라우팅 정보 전파
   - BGP를 통한 자동 라우팅 업데이트
```

#### 라우팅 제어 전략
```
시나리오별 라우팅 설계:

1. 모든 VPC 간 통신 허용
   - 단일 Route Table 사용
   - 모든 Attachment 연결

2. 보안 격리 (Dev/Prod 분리)
   - Dev Route Table: Dev VPC들만 연결
   - Prod Route Table: Prod VPC들만 연결
   - 공통 서비스는 별도 Route Table

3. 허브-스포크 모델
   - 중앙 서비스 VPC (허브)
   - 각 서비스 VPC (스포크)
   - 스포크 간 직접 통신 차단
```

### 실습: Transit Gateway 기본 설정 (데모)

#### Transit Gateway 생성 (참고용)
```
1. Transit Gateway 생성
   - VPC → Transit Gateways
   - "Create Transit Gateway" 클릭
   - Name: SAA-Study-TGW
   - Description: Study purpose TGW
   - Amazon side ASN: 64512 (기본값)
   - Default route table association: Enable
   - Default route table propagation: Enable
   - DNS support: Enable
   - Multicast support: Disable

2. VPC Attachment 생성
   - Transit Gateway Attachments
   - "Create Transit Gateway Attachment"
   - Transit Gateway ID: SAA-Study-TGW
   - Attachment type: VPC
   - VPC ID: SAA-Study-VPC
   - Subnet IDs: 각 AZ의 프라이빗 서브넷 선택

3. 라우팅 테이블 업데이트
   - VPC 라우팅 테이블에 TGW로의 경로 추가
   - 대상 네트워크 → Transit Gateway
```

---

## 🏗️ 멀티 VPC 아키텍처 설계 (45분)

### 아키텍처 패턴

#### 1. 환경별 분리 패턴
```
구조:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Dev VPC   │    │ Stage VPC   │    │  Prod VPC   │
│ 10.1.0.0/16 │    │ 10.2.0.0/16 │    │ 10.3.0.0/16 │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                  ┌─────────────┐
                  │ Shared VPC  │
                  │ 10.0.0.0/16 │
                  │(DNS, 모니터링)│
                  └─────────────┘

특징:
- 환경별 완전 격리
- 공통 서비스는 Shared VPC에서 제공
- 개발 환경 실험이 프로덕션에 영향 없음
```

#### 2. 서비스별 분리 패턴
```
구조:
┌─────────────┐    ┌─────────────┐
│  Web VPC    │    │  API VPC    │
│ 10.1.0.0/16 │←──→│ 10.2.0.0/16 │
└─────────────┘    └─────────────┘
       │                   │
       └───────────────────┼───────────────────┐
                           │                   │
                  ┌─────────────┐    ┌─────────────┐
                  │  Data VPC   │    │ Admin VPC   │
                  │ 10.3.0.0/16 │    │ 10.4.0.0/16 │
                  └─────────────┘    └─────────────┘

특징:
- 마이크로서비스 아키텍처 지원
- 서비스별 독립적 스케일링
- 장애 격리 및 보안 강화
```

#### 3. 지역별 분리 패턴
```
구조:
Seoul Region                    Tokyo Region
┌─────────────┐                ┌─────────────┐
│ Seoul VPC   │                │ Tokyo VPC   │
│ 10.1.0.0/16 │←──────────────→│ 10.2.0.0/16 │
└─────────────┘                └─────────────┘
       │                              │
       │        Cross-Region          │
       │        Peering or TGW        │
       │                              │
┌─────────────┐                ┌─────────────┐
│   DR VPC    │                │   DR VPC    │
│ 10.3.0.0/16 │                │ 10.4.0.0/16 │
└─────────────┘                └─────────────┘

특징:
- 글로벌 서비스 지원
- 재해 복구 환경 구성
- 지역별 데이터 주권 준수
```

### 네트워크 설계 원칙

#### CIDR 블록 계획
```
체계적인 IP 주소 할당:

조직 전체: 10.0.0.0/8
├── 서울 리전: 10.1.0.0/12
│   ├── Prod VPC: 10.1.0.0/16
│   ├── Stage VPC: 10.1.16.0/16
│   └── Dev VPC: 10.1.32.0/16
├── 도쿄 리전: 10.2.0.0/12
│   ├── Prod VPC: 10.2.0.0/16
│   └── DR VPC: 10.2.16.0/16
└── 공통 서비스: 10.0.0.0/16
    ├── DNS VPC: 10.0.0.0/20
    ├── 모니터링 VPC: 10.0.16.0/20
    └── 보안 VPC: 10.0.32.0/20

설계 원칙:
- 계층적 구조로 관리 용이성 확보
- 충분한 여유 공간 확보
- 미래 확장 고려
- 표준화된 명명 규칙 적용
```

#### 보안 경계 설정
```
네트워크 보안 계층:

1. VPC 레벨 격리
   - 환경별/서비스별 VPC 분리
   - 최소 권한 원칙 적용

2. 서브넷 레벨 분할
   - 퍼블릭/프라이빗/DB 서브넷 분리
   - 용도별 네트워크 ACL 적용

3. 인스턴스 레벨 제어
   - 보안 그룹으로 세밀한 제어
   - 애플리케이션별 규칙 적용

4. 연결 레벨 모니터링
   - VPC Flow Logs 활성화
   - 이상 트래픽 탐지 및 알림
```

---

## 💰 비용 최적화 전략 (30분)

### 연결 옵션별 비용 비교

#### VPC 피어링 비용
```
VPC 피어링:
- 연결 자체: 무료
- 데이터 전송: 리전 내 $0.01/GB, 리전 간 $0.02/GB
- 관리 비용: 없음

장점:
✅ 가장 저렴한 옵션
✅ 직접 연결로 최고 성능
✅ 단순한 구조에서 관리 용이

단점:
❌ 복잡한 네트워크에서 관리 어려움
❌ 전이적 라우팅 불가
❌ 연결 수 제한
```

#### Transit Gateway 비용
```
Transit Gateway:
- 시간당 요금: $0.07/시간 (월 ~$50)
- 데이터 처리: $0.02/GB
- Attachment 요금: 없음

장점:
✅ 중앙 집중식 관리
✅ 확장성 우수
✅ 복잡한 네트워크에 적합

단점:
❌ 고정 비용 발생
❌ 소규모 환경에서는 과도한 비용
❌ 약간의 성능 오버헤드
```

### 비용 최적화 전략

#### 상황별 최적 선택
```
소규모 환경 (VPC 5개 이하):
- VPC 피어링 사용
- 필요한 연결만 구성
- 단순한 라우팅 구조 유지

중간 규모 환경 (VPC 5-20개):
- 하이브리드 접근
- 핵심 서비스는 Transit Gateway
- 부가 서비스는 피어링

대규모 환경 (VPC 20개 이상):
- Transit Gateway 중심 설계
- 계층적 네트워크 구조
- 자동화된 관리 도구 활용
```

#### 트래픽 패턴 분석
```
비용 최적화를 위한 분석:

1. 트래픽 볼륨 측정
   - CloudWatch 메트릭 활용
   - 월별 데이터 전송량 확인
   - 피크 시간대 패턴 분석

2. 연결 패턴 분석
   - 자주 통신하는 VPC 쌍 식별
   - 일시적 연결 vs 상시 연결 구분
   - 대역폭 요구사항 확인

3. 비용 시뮬레이션
   - 피어링 vs TGW 비용 비교
   - 성장 시나리오별 비용 예측
   - ROI 계산 및 의사결정
```

---

## 📊 연결 성능 및 모니터링 (15분)

### 성능 특성 비교

#### 지연시간 및 대역폭
```
VPC 피어링:
- 지연시간: 최소 (직접 연결)
- 대역폭: 제한 없음
- 패킷 손실: 거의 없음

Transit Gateway:
- 지연시간: 약간 증가 (~1-2ms)
- 대역폭: 50Gbps per Attachment
- 패킷 손실: 매우 낮음

인터넷 경유:
- 지연시간: 높음 (가변적)
- 대역폭: 인터넷 품질에 의존
- 패킷 손실: 상대적으로 높음
```

### 모니터링 및 문제 해결

#### 핵심 모니터링 지표
```
VPC 피어링 모니터링:
- VPC Flow Logs로 트래픽 패턴 확인
- CloudWatch 메트릭으로 데이터 전송량 추적
- 연결 상태 및 라우팅 테이블 정기 점검

Transit Gateway 모니터링:
- TGW 전용 CloudWatch 메트릭
- Attachment별 트래픽 분석
- 라우팅 테이블 전파 상태 확인
```

---

## 📝 핵심 정리

### VPC 연결 옵션 선택 기준
1. **네트워크 규모**: 소규모는 피어링, 대규모는 TGW
2. **복잡성**: 단순한 구조는 피어링, 복잡한 구조는 TGW
3. **비용**: 피어링이 저렴, TGW는 관리 비용 절약
4. **성능**: 피어링이 최고 성능, TGW는 약간의 오버헤드

### 설계 원칙
- **CIDR 계획**: 체계적이고 확장 가능한 IP 주소 체계
- **보안 경계**: 환경별, 서비스별 적절한 격리
- **확장성**: 미래 성장을 고려한 아키텍처
- **비용 효율성**: 트래픽 패턴 기반 최적화

### 운영 고려사항
- **모니터링**: 지속적인 성능 및 비용 모니터링
- **자동화**: 라우팅 및 보안 정책 자동화
- **문서화**: 네트워크 구성 및 변경 이력 관리

---

## 🤔 토론 주제

1. **언제 VPC를 분리해야 할까?**
   - 보안 vs 관리 복잡성의 균형점

2. **Transit Gateway의 도입 시점은?**
   - 비용 대비 효과 분석

---

## 📚 다음 시간 예고

**하이브리드 연결**
- VPN vs Direct Connect 비교
- Site-to-Site VPN 설정 실습
- 온프레미스와 클라우드 연결 시나리오
- 하이브리드 아키텍처 설계 원칙

---

> 💡 **오늘의 핵심**: VPC 연결은 단순한 네트워크 연결이 아닙니다. 비즈니스 요구사항, 보안 정책, 비용 효율성을 모두 고려한 전략적 설계가 필요합니다. 작게 시작해서 점진적으로 확장하는 것이 현명한 접근법입니다!