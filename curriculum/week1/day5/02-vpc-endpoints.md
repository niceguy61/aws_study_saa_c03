# Day 5-2: VPC 엔드포인트

## 📚 학습 목표
- VPC 엔드포인트의 개념과 필요성 이해
- 게이트웨이 엔드포인트와 인터페이스 엔드포인트 차이점 파악
- S3 게이트웨이 엔드포인트 설정 실습
- EC2 인터페이스 엔드포인트 구성 및 활용

---

## 🔗 VPC 엔드포인트 개념 (30분)

### VPC 엔드포인트란?

#### 기존 AWS 서비스 접근 방식의 문제점
```
기존 방식: 프라이빗 서브넷 → NAT → 인터넷 → AWS 서비스

문제점:
❌ 불필요한 인터넷 경유 (보안 위험)
❌ NAT 게이트웨이 비용 발생
❌ 대역폭 제한 및 지연시간 증가
❌ 데이터 전송 비용 추가 발생
❌ 외부 네트워크 의존성

예시: S3 접근
프라이빗 EC2 → NAT Gateway → Internet Gateway → S3
- NAT 비용: $0.059/시간 + $0.059/GB
- 데이터 전송 비용: 추가 발생
- 지연시간: 인터넷 경유로 증가
```

#### VPC 엔드포인트 해결책
```
VPC 엔드포인트 방식: 프라이빗 서브넷 → VPC 엔드포인트 → AWS 서비스

장점:
✅ 프라이빗 연결 (인터넷 경유 없음)
✅ NAT 게이트웨이 비용 절약
✅ 향상된 보안 (AWS 백본 네트워크 사용)
✅ 낮은 지연시간
✅ 높은 가용성 (AWS 관리)
✅ 확장성 (자동 스케일링)

비유: 전용 엘리베이터
- 일반 엘리베이터 (인터넷): 여러 층 경유, 대기시간
- 전용 엘리베이터 (엔드포인트): 직접 연결, 빠른 접근
```

### VPC 엔드포인트 유형

#### 1. 게이트웨이 엔드포인트 (Gateway Endpoint)
```
지원 서비스:
- Amazon S3
- Amazon DynamoDB

특징:
✅ 무료 (데이터 전송 비용만 발생)
✅ 라우팅 테이블 기반 동작
✅ 높은 성능 (제한 없음)
✅ 간단한 설정

작동 방식:
1. 라우팅 테이블에 엔드포인트 경로 자동 추가
2. S3/DynamoDB 트래픽을 엔드포인트로 라우팅
3. AWS 백본 네트워크를 통한 직접 연결

제한사항:
❌ S3, DynamoDB만 지원
❌ 온프레미스에서 접근 불가
❌ 다른 VPC에서 접근 불가
```

#### 2. 인터페이스 엔드포인트 (Interface Endpoint)
```
지원 서비스:
- 대부분의 AWS 서비스 (EC2, SSM, KMS, Secrets Manager 등)
- 100개 이상의 서비스 지원

특징:
💰 시간당 비용 발생 ($0.014/시간 per AZ)
💰 데이터 처리 비용 ($0.014/GB)
🔌 ENI (Elastic Network Interface) 기반
🌐 DNS 기반 접근
🔒 보안 그룹 적용 가능

작동 방식:
1. 서브넷에 ENI 생성
2. 프라이빗 IP 주소 할당
3. DNS 이름을 통한 서비스 접근
4. PrivateLink 기술 활용

장점:
✅ 다양한 AWS 서비스 지원
✅ 온프레미스에서 접근 가능
✅ 다른 VPC에서 접근 가능
✅ 세밀한 보안 제어
```

---

## 🪣 S3 게이트웨이 엔드포인트 실습 (45분)

### 실습 시나리오

#### 비용 절약 시나리오
```
현재 상황:
- 프라이빗 서브넷의 EC2에서 S3 접근
- NAT 게이트웨이를 통한 인터넷 경유
- 월 1TB 데이터 전송 시 비용

기존 비용 (월):
- NAT 게이트웨이: $43 (24시간 운영)
- 데이터 처리: $60 (1TB × $0.059/GB)
- 총 비용: $103/월

엔드포인트 적용 후:
- S3 게이트웨이 엔드포인트: $0
- 데이터 전송: 리전 내 무료
- 총 비용: $0/월
- 절약 효과: $103/월 (100% 절약)
```

### 실습 1: S3 게이트웨이 엔드포인트 생성

#### 1단계: 엔드포인트 생성
```
1. VPC 엔드포인트 생성
   - VPC → Endpoints
   - "Create endpoint" 클릭
   - Name: S3-Gateway-Endpoint
   - Service category: AWS services
   - Service name: com.amazonaws.ap-northeast-2.s3
   - VPC: SAA-Study-VPC
   - Route tables: 프라이빗 라우팅 테이블들 선택
   - Policy: Full access (기본값)

2. 설정 확인
   - Service name에서 Type: Gateway 확인
   - 선택한 라우팅 테이블 확인
   - "Create endpoint" 클릭
```

#### 2단계: 라우팅 테이블 확인
```
1. 라우팅 테이블 자동 업데이트 확인
   - Route Tables → Private-Route-Table-AZ-A 선택
   - Routes 탭에서 새로운 경로 확인:
     * Destination: pl-78a54011 (S3 prefix list)
     * Target: vpce-xxxxxxxxx (VPC endpoint)

2. Prefix List 확인
   - Managed prefix lists 메뉴
   - S3 서비스의 IP 범위 자동 관리
   - 리전별 S3 IP 주소 포함
```

### 실습 2: S3 접근 테스트

#### 테스트 환경 준비
```
1. 테스트용 S3 버킷 생성
   - S3 → Buckets → "Create bucket"
   - Bucket name: saa-study-endpoint-test-[랜덤숫자]
   - Region: Asia Pacific (Seoul)
   - 기본 설정으로 생성

2. 테스트 파일 업로드
   - 간단한 텍스트 파일 업로드
   - 퍼블릭 액세스 차단 유지 (기본값)
```

#### 프라이빗 인스턴스에서 S3 접근 테스트
```
1. 프라이빗 인스턴스 접속
   # 퍼블릭 인스턴스를 통해 접속
   ssh -i key.pem ec2-user@[퍼블릭-IP]
   ssh -i key.pem ec2-user@[프라이빗-IP]

2. AWS CLI 설정 (IAM 역할 사용 권장)
   # EC2 인스턴스에 S3 접근 역할 부여
   aws configure list
   aws sts get-caller-identity

3. S3 접근 테스트
   # 버킷 목록 조회
   aws s3 ls
   
   # 특정 버킷 내용 조회
   aws s3 ls s3://saa-study-endpoint-test-[번호]/
   
   # 파일 업로드 테스트
   echo "Hello from private subnet" > test.txt
   aws s3 cp test.txt s3://saa-study-endpoint-test-[번호]/
   
   # 파일 다운로드 테스트
   aws s3 cp s3://saa-study-endpoint-test-[번호]/test.txt downloaded.txt
   cat downloaded.txt

4. 네트워크 경로 확인
   # S3 IP 주소 확인
   nslookup s3.ap-northeast-2.amazonaws.com
   
   # 라우팅 경로 확인
   ip route get [S3-IP-주소]
   # VPC 엔드포인트를 통한 라우팅 확인
```

### 실습 3: 엔드포인트 정책 설정

#### 세밀한 접근 제어
```
1. 엔드포인트 정책 수정
   - Endpoints → S3-Gateway-Endpoint 선택
   - Policy 탭 → "Edit policy"

2. 제한적 정책 예시
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::saa-study-endpoint-test-*",
        "arn:aws:s3:::saa-study-endpoint-test-*/*"
      ]
    }
  ]
}

3. 정책 적용 및 테스트
   - 허용된 버킷: 접근 가능
   - 다른 버킷: 접근 차단 확인
```

---

## 🔌 인터페이스 엔드포인트 실습 (45분)

### EC2 인터페이스 엔드포인트 구성

#### 실습 시나리오
```
목표: 프라이빗 서브넷에서 EC2 API 직접 접근
- EC2 인스턴스 정보 조회
- 인스턴스 시작/중지 (권한 있는 경우)
- Systems Manager 연결

기존 방식: 프라이빗 EC2 → NAT → 인터넷 → EC2 API
엔드포인트 방식: 프라이빗 EC2 → Interface Endpoint → EC2 API
```

### 실습 1: EC2 인터페이스 엔드포인트 생성

#### 1단계: 엔드포인트 생성
```
1. 인터페이스 엔드포인트 생성
   - VPC → Endpoints → "Create endpoint"
   - Name: EC2-Interface-Endpoint
   - Service category: AWS services
   - Service name: com.amazonaws.ap-northeast-2.ec2
   - VPC: SAA-Study-VPC
   - Subnets: 프라이빗 서브넷들 선택 (각 AZ)
   - Security groups: 새 보안 그룹 생성

2. 보안 그룹 설정
   - Name: VPC-Endpoint-SG
   - Inbound rules:
     * HTTPS (443): 10.0.0.0/16 (VPC CIDR)
     * Description: Allow HTTPS from VPC
   - Outbound rules: 기본값 유지

3. DNS 설정
   - Enable DNS hostnames: 체크
   - Enable private DNS names: 체크 (권장)
   - "Create endpoint" 클릭
```

#### 2단계: 엔드포인트 상태 확인
```
1. 엔드포인트 상태 확인
   - State: Available 확인 (몇 분 소요)
   - DNS names 확인:
     * Regional DNS name
     * Zonal DNS names (AZ별)

2. ENI 확인
   - EC2 → Network Interfaces
   - VPC endpoint ENI 확인
   - 각 AZ별 프라이빗 IP 할당 확인

3. DNS 해석 테스트
   # 프라이빗 인스턴스에서 실행
   nslookup ec2.ap-northeast-2.amazonaws.com
   # 프라이빗 IP 반환 확인 (엔드포인트 IP)
```

### 실습 2: EC2 API 접근 테스트

#### API 호출 테스트
```
1. 기본 EC2 정보 조회
   # 현재 인스턴스 정보
   aws ec2 describe-instances --instance-ids $(curl -s http://169.254.169.254/latest/meta-data/instance-id)
   
   # 모든 인스턴스 정보 (권한 있는 경우)
   aws ec2 describe-instances
   
   # 리전 정보 확인
   aws ec2 describe-regions

2. 네트워크 경로 확인
   # EC2 API 엔드포인트 IP 확인
   nslookup ec2.ap-northeast-2.amazonaws.com
   
   # 프라이빗 IP 반환 확인 (10.0.x.x)
   # 인터넷 경유 없이 직접 접근 확인

3. 성능 비교
   # 엔드포인트 사용 시 응답 시간 측정
   time aws ec2 describe-regions
   
   # NAT 경유 시와 비교 (참고용)
```

### 실습 3: Systems Manager 엔드포인트

#### SSM 엔드포인트 생성 (선택사항)
```
1. 필요한 SSM 엔드포인트들
   - com.amazonaws.ap-northeast-2.ssm
   - com.amazonaws.ap-northeast-2.ssmmessages
   - com.amazonaws.ap-northeast-2.ec2messages

2. SSM 엔드포인트 생성
   # 각각 별도로 생성하거나 한 번에 생성
   - 동일한 보안 그룹 사용
   - 동일한 서브넷 선택
   - Private DNS names 활성화

3. Session Manager 테스트
   # AWS 콘솔에서 Session Manager 접속 테스트
   # 프라이빗 인스턴스에 직접 접속 가능 확인
```

---

## 💰 비용 최적화 전략 (30분)

### 엔드포인트 비용 분석

#### 게이트웨이 엔드포인트 비용
```
S3 게이트웨이 엔드포인트:
- 엔드포인트 자체: 무료
- 데이터 전송: 리전 내 무료
- 총 비용: $0

DynamoDB 게이트웨이 엔드포인트:
- 엔드포인트 자체: 무료
- 데이터 전송: 리전 내 무료
- 총 비용: $0

ROI 계산:
- NAT 게이트웨이 절약: $43/월
- 데이터 처리 절약: $0.059/GB
- 즉시 100% 비용 절약
```

#### 인터페이스 엔드포인트 비용
```
인터페이스 엔드포인트 비용 (서울 리전):
- 시간당 요금: $0.014/시간 per AZ
- 데이터 처리: $0.014/GB
- 월 비용 (1개 AZ): ~$10

비용 분석 예시:
시나리오: EC2 API 월 100GB 사용

기존 비용 (NAT 경유):
- NAT 게이트웨이: $43/월
- 데이터 처리: $5.9/월 (100GB × $0.059)
- 총 비용: $48.9/월

엔드포인트 비용:
- 엔드포인트: $10/월 (1개 AZ)
- 데이터 처리: $1.4/월 (100GB × $0.014)
- 총 비용: $11.4/월

절약 효과: $37.5/월 (77% 절약)
```

### 최적화 전략

#### 엔드포인트 선택 기준
```
게이트웨이 엔드포인트 우선 고려:
✅ S3, DynamoDB 사용 시
✅ 대용량 데이터 전송
✅ 비용 최적화 우선
✅ 간단한 설정 선호

인터페이스 엔드포인트 고려:
✅ 다양한 AWS 서비스 사용
✅ 온프레미스 연결 필요
✅ 세밀한 보안 제어 필요
✅ 성능보다 기능 우선

하이브리드 접근:
🎯 S3/DynamoDB: 게이트웨이 엔드포인트
🎯 기타 서비스: 필요시 인터페이스 엔드포인트
🎯 사용량 기반 비용 분석 후 결정
```

#### 비용 모니터링
```
비용 추적 방법:
1. CloudWatch 메트릭 활용
   - 엔드포인트 사용량 모니터링
   - 데이터 전송량 추적

2. Cost Explorer 분석
   - VPC 엔드포인트 비용 분석
   - NAT 게이트웨이 비용과 비교

3. 정기적 검토
   - 월별 비용 분석
   - 사용하지 않는 엔드포인트 정리
   - 트래픽 패턴 변화 반영
```

---

## 🔒 보안 고려사항 (15분)

### 엔드포인트 보안 설정

#### 게이트웨이 엔드포인트 보안
```
보안 요소:
1. 엔드포인트 정책
   - 접근 가능한 리소스 제한
   - 허용된 작업 제한
   - 조건부 접근 제어

2. 라우팅 제어
   - 특정 라우팅 테이블에만 연결
   - 불필요한 서브넷 접근 차단

3. IAM 정책 연동
   - 사용자/역할별 접근 제어
   - 리소스 기반 정책 활용
```

#### 인터페이스 엔드포인트 보안
```
보안 요소:
1. 보안 그룹
   - HTTPS(443) 포트만 허용
   - 소스 IP 제한 (VPC CIDR)
   - 최소 권한 원칙 적용

2. 엔드포인트 정책
   - API 작업 제한
   - 리소스 접근 제한
   - 시간/조건 기반 제어

3. DNS 보안
   - Private DNS 활성화
   - DNS 하이재킹 방지
   - 내부 DNS 해석 보장

4. 네트워크 격리
   - 전용 서브넷 사용 고려
   - 트래픽 모니터링
   - 로그 분석
```

---

## 📝 핵심 정리

### VPC 엔드포인트 핵심 개념
1. **게이트웨이 엔드포인트**: S3/DynamoDB 전용, 무료, 라우팅 기반
2. **인터페이스 엔드포인트**: 대부분 서비스 지원, 유료, ENI 기반
3. **비용 절약**: NAT 게이트웨이 대체로 상당한 비용 절약
4. **보안 강화**: 인터넷 경유 없는 프라이빗 연결

### 설계 원칙
- **우선순위**: 게이트웨이 엔드포인트 우선 고려
- **비용 분석**: 사용량 기반 ROI 계산
- **보안 설정**: 최소 권한 원칙 적용
- **모니터링**: 지속적인 비용 및 사용량 추적

### 실무 적용 포인트
- **S3 접근**: 게이트웨이 엔드포인트로 100% 비용 절약
- **API 호출**: 인터페이스 엔드포인트로 성능 향상
- **하이브리드**: 서비스별 최적 엔드포인트 선택
- **정기 검토**: 사용 패턴 변화에 따른 최적화

---

## 🤔 토론 주제

1. **모든 AWS 서비스에 엔드포인트가 필요할까?**
   - 비용 vs 성능 vs 보안의 균형점

2. **온프레미스 환경에서 엔드포인트 활용 방안은?**
   - Direct Connect와 엔드포인트 조합

---

## 📚 다음 시간 예고

**네트워크 모니터링**
- VPC Flow Logs 설정 및 활용
- CloudWatch를 통한 네트워크 모니터링
- 네트워크 트래픽 분석 및 최적화
- 보안 이벤트 탐지 및 대응

---

> 💡 **오늘의 핵심**: VPC 엔드포인트는 비용 절약과 보안 강화를 동시에 달성할 수 있는 핵심 기술입니다. 특히 S3 게이트웨이 엔드포인트는 즉시 100% 비용 절약 효과를 가져다줍니다. 서비스별 특성을 고려하여 적절한 엔드포인트를 선택하세요!