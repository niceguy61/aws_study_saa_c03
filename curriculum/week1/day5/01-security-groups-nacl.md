# Day 5-1: 보안 그룹 vs NACL

## 📚 학습 목표
- 보안 그룹과 네트워크 ACL의 차이점 완전 이해
- 각 보안 계층의 특성과 적용 시나리오 파악
- 실습을 통한 보안 규칙 설정 및 테스트
- 다층 보안 모델 설계 원칙 습득

---

## 🛡️ AWS 네트워크 보안 개요 (30분)

### 다층 보안 모델 (Defense in Depth)

#### AWS 네트워크 보안 계층
```
인터넷
    ↓
┌─────────────────────────────────────────┐
│ 1. AWS Shield (DDoS 보호)               │
├─────────────────────────────────────────┤
│ 2. AWS WAF (웹 애플리케이션 방화벽)      │
├─────────────────────────────────────────┤
│ 3. Network Load Balancer                │
├─────────────────────────────────────────┤
│ 4. Network ACL (서브넷 레벨)            │
├─────────────────────────────────────────┤
│ 5. Security Group (인스턴스 레벨)       │
├─────────────────────────────────────────┤
│ 6. Host-based Firewall (OS 레벨)       │
└─────────────────────────────────────────┘
    ↓
EC2 인스턴스

각 계층의 역할:
- 외부 → 내부로 갈수록 더 세밀한 제어
- 여러 계층이 함께 작동하여 보안 강화
- 한 계층이 뚫려도 다른 계층에서 차단
```

#### 보안 그룹 vs NACL 비교 개요
```
보안 그룹 (Security Group):
🎯 대상: EC2 인스턴스 (ENI 레벨)
🔄 상태: Stateful (상태 저장)
📝 규칙: Allow 규칙만 가능
🔢 평가: 모든 규칙 평가 후 허용 여부 결정
🏷️ 참조: 다른 보안 그룹 ID 참조 가능

네트워크 ACL (NACL):
🎯 대상: 서브넷 전체
🔄 상태: Stateless (상태 비저장)
📝 규칙: Allow/Deny 규칙 모두 가능
🔢 평가: 규칙 번호 순서대로 평가
🏷️ 참조: IP 주소/CIDR 블록만 가능
```

---

## 🔒 보안 그룹 심화 (45분)

### 보안 그룹의 작동 원리

#### Stateful 특성 이해
```
Stateful = 연결 상태 추적

예시: 웹 서버 HTTP 요청
1. 클라이언트 → 서버 (포트 80)
   - 인바운드 규칙: HTTP 80 허용 필요
   
2. 서버 → 클라이언트 (임시 포트 32768-65535)
   - 아웃바운드 규칙: 자동으로 허용됨 (상태 추적)
   - 별도 아웃바운드 규칙 불필요

장점:
✅ 관리 편의성 (응답 트래픽 자동 허용)
✅ 보안성 (연결 상태 기반 제어)
✅ 성능 (연결 추적으로 빠른 처리)
```

#### 보안 그룹 규칙 구성
```
인바운드 규칙 구성 요소:
- Type: 프로토콜 유형 (HTTP, HTTPS, SSH, Custom)
- Protocol: TCP, UDP, ICMP
- Port Range: 포트 번호 또는 범위
- Source: 트래픽 소스 (IP, CIDR, 다른 SG)
- Description: 규칙 설명 (권장)

아웃바운드 규칙 구성 요소:
- Type: 프로토콜 유형
- Protocol: TCP, UDP, ICMP
- Port Range: 포트 번호 또는 범위
- Destination: 트래픽 목적지
- Description: 규칙 설명
```

### 실습 1: 웹 서버 보안 그룹 설정

#### 웹 서버 보안 그룹 생성
```
1. 보안 그룹 생성
   - EC2 → Security Groups
   - "Create security group" 클릭
   - Name: WebServer-SG
   - Description: Security group for web servers
   - VPC: SAA-Study-VPC

2. 인바운드 규칙 설정
   Rule 1:
   - Type: HTTP
   - Port: 80
   - Source: 0.0.0.0/0
   - Description: Allow HTTP from anywhere
   
   Rule 2:
   - Type: HTTPS
   - Port: 443
   - Source: 0.0.0.0/0
   - Description: Allow HTTPS from anywhere
   
   Rule 3:
   - Type: SSH
   - Port: 22
   - Source: My IP
   - Description: SSH access from admin

3. 아웃바운드 규칙 확인
   - 기본값: All traffic to 0.0.0.0/0 (모든 아웃바운드 허용)
   - 필요시 제한적으로 변경 가능
```

#### 애플리케이션 서버 보안 그룹 생성
```
1. 보안 그룹 생성
   - Name: AppServer-SG
   - Description: Security group for application servers

2. 인바운드 규칙 설정
   Rule 1:
   - Type: Custom TCP
   - Port: 8080
   - Source: WebServer-SG (보안 그룹 ID 참조)
   - Description: Allow HTTP from web servers
   
   Rule 2:
   - Type: SSH
   - Port: 22
   - Source: WebServer-SG
   - Description: SSH from web servers only

3. 아웃바운드 규칙 설정
   Rule 1:
   - Type: HTTPS
   - Port: 443
   - Destination: 0.0.0.0/0
   - Description: External API calls
   
   Rule 2:
   - Type: MySQL/Aurora
   - Port: 3306
   - Destination: Database-SG
   - Description: Database connection
```

### 보안 그룹 고급 기능

#### 보안 그룹 체이닝
```
보안 그룹 참조의 장점:

1. 동적 관리
   - 참조된 보안 그룹의 인스턴스가 변경되어도 자동 적용
   - IP 주소 변경에 대한 걱정 없음

2. 확장성
   - 새로운 웹 서버 추가 시 자동으로 접근 허용
   - 수동 IP 관리 불필요

3. 보안성
   - 특정 역할의 인스턴스만 접근 허용
   - 최소 권한 원칙 구현

예시:
WebServer-SG → AppServer-SG → Database-SG
- 각 계층은 이전 계층에서만 접근 가능
- 계층적 보안 모델 구현
```

---

## 🚧 네트워크 ACL 심화 (45분)

### NACL의 작동 원리

#### Stateless 특성 이해
```
Stateless = 연결 상태 추적 안 함

예시: 웹 서버 HTTP 요청
1. 클라이언트 → 서버 (포트 80)
   - 인바운드 규칙: HTTP 80 허용 필요
   
2. 서버 → 클라이언트 (임시 포트 32768-65535)
   - 아웃바운드 규칙: 임시 포트 범위 허용 필요
   - 별도 아웃바운드 규칙 명시적 설정 필요

주의사항:
⚠️ 양방향 트래픽 모두 명시적 허용 필요
⚠️ 임시 포트 범위 고려 필요
⚠️ 규칙 순서 중요 (번호 순서대로 평가)
```

#### NACL 규칙 평가 순서
```
규칙 평가 방식:
1. 규칙 번호 오름차순으로 평가
2. 첫 번째 매칭되는 규칙 적용
3. 매칭되는 규칙이 없으면 기본 DENY

예시 NACL 규칙:
Rule #100: ALLOW HTTP (80) from 0.0.0.0/0
Rule #200: DENY HTTP (80) from 192.168.1.0/24
Rule #300: ALLOW ALL from 0.0.0.0/0
Rule #*: DENY ALL (기본 규칙)

결과:
- 192.168.1.0/24에서 HTTP 접근 → 허용 (Rule #100이 먼저 매칭)
- Rule #200은 실행되지 않음 (Dead Rule)
```

### 실습 2: NACL 설정 및 테스트

#### 커스텀 NACL 생성
```
1. NACL 생성
   - VPC → Network ACLs
   - "Create network ACL" 클릭
   - Name: WebTier-NACL
   - VPC: SAA-Study-VPC

2. 인바운드 규칙 설정
   Rule #100:
   - Type: HTTP (80)
   - Source: 0.0.0.0/0
   - Allow/Deny: ALLOW
   
   Rule #110:
   - Type: HTTPS (443)
   - Source: 0.0.0.0/0
   - Allow/Deny: ALLOW
   
   Rule #120:
   - Type: SSH (22)
   - Source: [관리자 IP]/32
   - Allow/Deny: ALLOW
   
   Rule #130:
   - Type: Custom TCP
   - Port: 32768-65535
   - Source: 0.0.0.0/0
   - Allow/Deny: ALLOW
   - Description: Ephemeral ports for responses

3. 아웃바운드 규칙 설정
   Rule #100:
   - Type: HTTP (80)
   - Destination: 0.0.0.0/0
   - Allow/Deny: ALLOW
   
   Rule #110:
   - Type: HTTPS (443)
   - Destination: 0.0.0.0/0
   - Allow/Deny: ALLOW
   
   Rule #120:
   - Type: Custom TCP
   - Port: 32768-65535
   - Destination: 0.0.0.0/0
   - Allow/Deny: ALLOW
   - Description: Ephemeral ports for responses
```

#### NACL 서브넷 연결
```
1. 서브넷 연결
   - 생성한 NACL 선택
   - Subnet associations 탭
   - "Edit subnet associations"
   - 퍼블릭 서브넷들 선택
   - "Save changes"

2. 기존 연결 확인
   - 서브넷은 하나의 NACL에만 연결 가능
   - 기존 기본 NACL에서 자동으로 해제됨
```

### NACL 고급 활용

#### 특정 IP 차단 시나리오
```
시나리오: 악성 IP 192.168.100.50 차단

NACL 규칙 설정:
Rule #50:
- Type: All Traffic
- Source: 192.168.100.50/32
- Allow/Deny: DENY
- Description: Block malicious IP

Rule #100:
- Type: HTTP (80)
- Source: 0.0.0.0/0
- Allow/Deny: ALLOW

결과:
- 192.168.100.50은 모든 접근 차단 (Rule #50)
- 다른 모든 IP는 HTTP 접근 허용 (Rule #100)

보안 그룹으로는 불가능:
- 보안 그룹은 ALLOW 규칙만 가능
- 특정 IP 차단은 NACL에서만 가능
```

---

## ⚖️ 보안 그룹 vs NACL 비교 분석 (30분)

### 상세 비교표

#### 기능별 상세 비교
```
┌─────────────────┬─────────────────┬─────────────────┐
│     특성        │   보안 그룹     │     NACL        │
├─────────────────┼─────────────────┼─────────────────┤
│ 적용 레벨       │ 인스턴스 (ENI)  │ 서브넷          │
│ 상태 관리       │ Stateful        │ Stateless       │
│ 규칙 유형       │ Allow만         │ Allow/Deny      │
│ 규칙 평가       │ 모든 규칙 평가  │ 순서대로 평가   │
│ 기본 정책       │ 모든 트래픽 거부│ 모든 트래픽 허용│
│ 규칙 수 제한    │ 60개 (기본)     │ 20개 (기본)     │
│ 참조 가능       │ SG ID, IP       │ IP/CIDR만       │
│ 변경 적용       │ 즉시 적용       │ 즉시 적용       │
│ 로깅            │ VPC Flow Logs   │ VPC Flow Logs   │
└─────────────────┴─────────────────┴─────────────────┘
```

#### 사용 시나리오별 선택 기준
```
보안 그룹 주 사용 시나리오:
✅ 애플리케이션별 세밀한 접근 제어
✅ 동적 환경 (Auto Scaling)
✅ 마이크로서비스 간 통신 제어
✅ 개발/테스트 환경의 빠른 변경

NACL 주 사용 시나리오:
✅ 서브넷 레벨 추가 보안 계층
✅ 특정 IP/대역 차단
✅ 규정 준수 요구사항
✅ 네트워크 레벨 로깅 및 감사

조합 사용 (권장):
🎯 NACL: 서브넷 레벨 기본 보안 + 악성 IP 차단
🎯 보안 그룹: 애플리케이션별 세밀한 제어
```

### 실습 3: 보안 계층 테스트

#### 테스트 시나리오 구성
```
테스트 환경:
- 웹 서버 인스턴스 (퍼블릭 서브넷)
- 앱 서버 인스턴스 (프라이빗 서브넷)
- 각각 다른 보안 그룹 적용
- 커스텀 NACL 적용

테스트 케이스:
1. 인터넷 → 웹 서버 HTTP 접근
2. 웹 서버 → 앱 서버 통신
3. 특정 IP 차단 테스트
4. 보안 그룹 vs NACL 우선순위 테스트
```

#### 테스트 수행 및 결과 분석
```
테스트 1: 정상 HTTP 접근
curl http://[웹서버-퍼블릭-IP]
예상 결과: 성공 (보안 그룹 + NACL 모두 허용)

테스트 2: SSH 접근 (허용된 IP)
ssh -i key.pem ec2-user@[웹서버-퍼블릭-IP]
예상 결과: 성공

테스트 3: SSH 접근 (차단된 IP)
# 다른 IP에서 SSH 시도
예상 결과: 실패 (보안 그룹에서 차단)

테스트 4: NACL 차단 테스트
# NACL에서 특정 IP 차단 후 테스트
예상 결과: 보안 그룹 허용이어도 NACL에서 차단

분석 포인트:
- 두 보안 계층 모두 허용해야 통신 가능
- NACL 차단이 보안 그룹 허용보다 우선
- Stateful vs Stateless 동작 차이 확인
```

---

## 🔧 보안 설정 모범 사례 (15분)

### 보안 그룹 모범 사례

#### 설계 원칙
```
1. 최소 권한 원칙
   - 필요한 포트만 개방
   - 소스를 최대한 제한적으로 설정
   - 정기적인 권한 검토

2. 명명 규칙 표준화
   - 역할별 명명: WebServer-SG, Database-SG
   - 환경별 구분: Prod-WebServer-SG, Dev-WebServer-SG
   - 설명 필드 활용

3. 그룹 참조 활용
   - IP 주소 대신 보안 그룹 ID 참조
   - 계층적 구조 설계
   - 동적 환경 대응

4. 정기적 감사
   - 사용하지 않는 규칙 정리
   - 과도한 권한 검토
   - 변경 이력 추적
```

### NACL 모범 사례

#### 설계 원칙
```
1. 기본 NACL 유지
   - 특별한 요구사항이 없으면 기본 NACL 사용
   - 모든 트래픽 허용으로 관리 편의성 확보

2. 커스텀 NACL 신중 사용
   - 특정 보안 요구사항이 있을 때만 사용
   - 임시 포트 범위 고려
   - 양방향 트래픽 규칙 설정

3. 규칙 번호 체계화
   - 10, 20, 30... 단위로 번호 할당
   - 중간 삽입 여유 공간 확보
   - 우선순위 명확히 설계

4. 문서화
   - 각 규칙의 목적 명시
   - 변경 이유 기록
   - 정기적 검토 일정 수립
```

---

## 📝 핵심 정리

### 보안 그룹 vs NACL 핵심 차이점
1. **적용 범위**: 인스턴스 vs 서브넷
2. **상태 관리**: Stateful vs Stateless
3. **규칙 유형**: Allow만 vs Allow/Deny
4. **평가 방식**: 모든 규칙 vs 순서대로

### 다층 보안 전략
- **1차 방어**: NACL (서브넷 레벨)
- **2차 방어**: 보안 그룹 (인스턴스 레벨)
- **3차 방어**: 호스트 기반 방화벽

### 실무 적용 가이드
- **일반적 사용**: 보안 그룹 중심 설계
- **특수 요구사항**: NACL 추가 활용
- **보안 강화**: 두 계층 모두 활용
- **관리 효율성**: 표준화된 명명 규칙

---

## 🤔 토론 주제

1. **언제 NACL을 사용해야 할까?**
   - 보안 그룹만으로 충분한 경우 vs NACL이 필요한 경우

2. **성능에 미치는 영향은?**
   - 보안 규칙이 네트워크 성능에 미치는 영향

---

## 📚 다음 시간 예고

**VPC 엔드포인트**
- 게이트웨이 엔드포인트 vs 인터페이스 엔드포인트
- S3 게이트웨이 엔드포인트 설정 실습
- EC2 인터페이스 엔드포인트 구성
- 비용 최적화를 위한 엔드포인트 활용

---

> 💡 **오늘의 핵심**: 보안 그룹과 NACL은 서로 다른 계층에서 작동하는 보완적 보안 도구입니다. 보안 그룹의 편의성과 NACL의 강력함을 적절히 조합하여 다층 보안 모델을 구축하세요!