# Day 1-4: 클라우드 경제학 & 요금 모델

## 📚 학습 목표
- AWS 요금 체계의 기본 원리 이해
- 온디맨드, 예약 인스턴스, 스팟 인스턴스 차이점 파악
- AWS 요금 계산기 활용 방법 습득
- 비용 최적화 전략 및 모범 사례 학습

---

## 💰 AWS 요금 체계 기본 원리 (30분)

![AWS Pricing Models](../images/aws-pricing-models.svg)

### Pay-as-you-go = 사용한 만큼만 지불

#### 전통적 IT vs 클라우드 비용 구조
```
전통적 IT (CAPEX 모델):
초기 투자 → 서버 구매 (3,000만원)
         → 네트워크 장비 (1,000만원)
         → 소프트웨어 라이선스 (500만원)
         → 총 4,500만원 선투자

운영 비용 → 전기료 (월 100만원)
         → 인건비 (월 500만원)
         → 유지보수 (월 200만원)
         → 월 800만원 고정비
```

```
클라우드 (OPEX 모델):
초기 투자 → 0원
운영 비용 → 사용한 리소스만큼
         → 트래픽 적으면 적게
         → 트래픽 많으면 많게
         → 변동비 구조
```

#### AWS 요금 구성 요소
```
컴퓨팅 비용:
- 인스턴스 실행 시간
- 인스턴스 유형별 시간당 요금
- 예: t3.micro = $0.0104/시간

스토리지 비용:
- 저장 용량 (GB당)
- 요청 횟수 (GET, PUT)
- 데이터 전송량

네트워크 비용:
- 데이터 전송량 (GB당)
- 리전 간 전송
- 인터넷으로 나가는 트래픽
```

---

## 🖥️ EC2 요금 모델 (45분)

### 1. 온디맨드 인스턴스 = 택시

#### 특징
```
장점:
- 선불 결제 없음
- 장기 약정 없음
- 언제든 시작/중지 가능
- 예측 불가능한 워크로드에 적합

단점:
- 가장 비싼 요금
- 장기 사용 시 비효율적

사용 사례:
- 개발/테스트 환경
- 단기 프로젝트
- 예측 불가능한 워크로드
```

#### 요금 예시
```
t3.micro (1 vCPU, 1GB RAM):
- 시간당: $0.0104
- 월간 (24시간 × 30일): $7.49
- 연간: $91.13
```

### 2. 예약 인스턴스 = 월세 계약

#### 특징
```
약정 기간:
- 1년 또는 3년 약정
- 중도 해지 불가
- 약정 기간 동안 할인

결제 옵션:
- 선불 없음 (No Upfront)
- 부분 선불 (Partial Upfront)
- 전액 선불 (All Upfront)

할인율:
- 1년 약정: 최대 40% 할인
- 3년 약정: 최대 60% 할인
```

#### 요금 비교 (t3.micro 기준)
```
온디맨드: $91.13/년

예약 인스턴스 (1년):
- 선불 없음: $62.05/년 (32% 할인)
- 부분 선불: $59.00/년 (35% 할인)
- 전액 선불: $56.00/년 (39% 할인)

예약 인스턴스 (3년):
- 전액 선불: $37.00/년 (59% 할인)
```

### 3. 스팟 인스턴스 = 경매

#### 특징
```
동작 원리:
- AWS 여유 용량 활용
- 시장 가격으로 경매
- 최대 90% 할인 가능
- 언제든 중단될 수 있음

중단 조건:
- 스팟 가격이 입찰가 초과
- AWS 용량 부족
- 2분 전 중단 알림

적합한 워크로드:
- 배치 처리
- 빅데이터 분석
- CI/CD 파이프라인
- 내결함성 애플리케이션
```

#### 스팟 인스턴스 전략
```
다양화 전략:
- 여러 인스턴스 유형 사용
- 여러 가용영역 활용
- 스팟 플릿 구성

체크포인트 전략:
- 정기적 작업 상태 저장
- 중단 시 재시작 가능
- S3에 중간 결과 저장
```

---

## 💾 스토리지 요금 모델 (30분)

### S3 스토리지 클래스별 요금

#### 스토리지 클래스 비교
```
S3 Standard:
- 자주 접근하는 데이터
- $0.023/GB/월
- 즉시 접근 가능

S3 Standard-IA (Infrequent Access):
- 가끔 접근하는 데이터
- $0.0125/GB/월 (45% 저렴)
- 검색 시 추가 요금

S3 Glacier:
- 아카이브 데이터
- $0.004/GB/월 (83% 저렴)
- 검색에 몇 분~몇 시간

S3 Glacier Deep Archive:
- 장기 보관 데이터
- $0.00099/GB/월 (96% 저렴)
- 검색에 12시간
```

#### 수명 주기 정책 활용
```
자동 전환 설정:
Day 0-30: S3 Standard
Day 31-90: S3 Standard-IA
Day 91-365: S3 Glacier
Day 366+: S3 Glacier Deep Archive

비용 절감 효과:
- 1TB 데이터 1년 보관
- Standard만 사용: $276
- 수명주기 적용: $156 (43% 절약)
```

---

## 🌐 네트워크 요금 모델 (20분)

### 데이터 전송 요금

#### 기본 원칙
```
무료 전송:
- AWS 서비스로 들어오는 데이터
- 같은 AZ 내 전송
- CloudFront에서 오리진으로

유료 전송:
- 인터넷으로 나가는 데이터
- 리전 간 전송
- AZ 간 전송
```

#### 요금 구조 (미국 동부 기준)
```
인터넷 아웃바운드:
- 첫 1GB/월: 무료
- 다음 9.999TB: $0.09/GB
- 다음 40TB: $0.085/GB
- 다음 100TB: $0.07/GB

리전 간 전송:
- $0.02/GB (일반적)

AZ 간 전송:
- $0.01/GB (in/out)
```

---

## 🧮 AWS 요금 계산기 실습 (30분)

### AWS Pricing Calculator 사용법

#### 실습: 웹 애플리케이션 비용 계산
```
시나리오:
- 웹 서버 2대 (t3.medium)
- 데이터베이스 1대 (db.t3.micro)
- 로드 밸런서 1개
- S3 스토리지 100GB
- 월 트래픽 1TB
```

#### 단계별 계산 과정
```
1단계: AWS Pricing Calculator 접속
- https://calculator.aws/

2단계: 리전 선택
- Asia Pacific (Seoul)

3단계: EC2 인스턴스 추가
- Service: Amazon EC2
- Instance type: t3.medium
- Quantity: 2
- Usage: 24시간 × 30일 = 720시간

4단계: RDS 추가
- Service: Amazon RDS
- Engine: MySQL
- Instance type: db.t3.micro
- Usage: 720시간

5단계: Application Load Balancer 추가
- Service: Elastic Load Balancing
- Type: Application Load Balancer
- Usage: 720시간

6단계: S3 추가
- Service: Amazon S3
- Storage: 100GB Standard
- Requests: 10,000 GET, 1,000 PUT

7단계: 데이터 전송 추가
- Outbound data transfer: 1TB/월
```

#### 예상 비용 결과
```
월간 예상 비용:
- EC2 (t3.medium × 2): $67.07
- RDS (db.t3.micro): $16.79
- ALB: $22.27
- S3: $2.30
- Data Transfer: $90.00
- 총합: $198.43/월

연간 예상 비용: $2,381.16
```

---

## 📊 비용 최적화 전략 (30분)

### 1. Right Sizing = 적정 크기 선택

#### 모니터링 기반 최적화
```
과정:
1. CloudWatch로 리소스 사용률 모니터링
2. CPU, 메모리, 네트워크 사용률 분석
3. 과소/과대 사용 인스턴스 식별
4. 적절한 크기로 조정

예시:
- 현재: t3.large (CPU 사용률 20%)
- 최적화: t3.medium (CPU 사용률 40%)
- 절약: 50% 비용 절감
```

### 2. 예약 인스턴스 활용

#### 구매 전략
```
분석 단계:
1. 지난 12개월 사용 패턴 분석
2. 안정적으로 사용되는 인스턴스 식별
3. 예약 인스턴스 구매 계획 수립

구매 전략:
- 70% 예약 인스턴스 (안정적 워크로드)
- 20% 온디맨드 (변동 워크로드)
- 10% 스팟 인스턴스 (배치 작업)
```

### 3. 자동화를 통한 비용 절감

#### 스케줄링 자동화
```
개발/테스트 환경:
- 평일 09:00 시작
- 평일 18:00 종료
- 주말 완전 종료
- 절약 효과: 65% 비용 절감

Lambda 함수로 자동화:
- CloudWatch Events로 스케줄링
- EC2 인스턴스 자동 시작/종료
- RDS 인스턴스 자동 시작/종료
```

### 4. 스토리지 최적화

#### 수명 주기 관리
```
자동 전환 정책:
- 30일 후: Standard → Standard-IA
- 90일 후: Standard-IA → Glacier
- 365일 후: Glacier → Deep Archive
- 7년 후: 자동 삭제

중복 제거:
- S3 Intelligent Tiering 활용
- 불필요한 스냅샷 정리
- 미사용 EBS 볼륨 삭제
```

---

## 📈 비용 모니터링 및 알림 (15분)

### AWS Cost Explorer 활용

#### 비용 분석 기능
```
기본 분석:
- 월별/일별 비용 추이
- 서비스별 비용 분해
- 리전별 비용 분석

고급 분석:
- 태그별 비용 분석
- 예약 인스턴스 활용률
- 비용 예측
```

### AWS Budgets 설정

#### 예산 알림 설정
```
예산 유형:
- 비용 예산: 월 $200 한도
- 사용량 예산: EC2 시간 한도
- 예약 인스턴스 예산: 활용률 목표

알림 설정:
- 80% 도달 시: 이메일 알림
- 100% 도달 시: 이메일 + SMS
- 120% 예상 시: 관리자 알림
```

---

## 📝 핵심 정리

### AWS 요금 모델 특징
1. **사용량 기반**: 사용한 만큼만 지불
2. **선택의 다양성**: 다양한 요금 옵션 제공
3. **투명성**: 모든 요금 공개
4. **최적화 도구**: 비용 관리 도구 제공

### 비용 최적화 핵심 전략
- **Right Sizing**: 적절한 리소스 크기 선택
- **예약 구매**: 안정적 워크로드는 예약 인스턴스
- **자동화**: 스케줄링을 통한 자동 관리
- **모니터링**: 지속적인 비용 추적 및 분석

### 요금 모델 선택 가이드
- **온디맨드**: 예측 불가능한 워크로드
- **예약 인스턴스**: 안정적이고 예측 가능한 워크로드
- **스팟 인스턴스**: 중단 가능한 배치 작업
- **하이브리드**: 여러 모델의 조합 활용

---

## 🤔 토론 주제

1. **스타트업 vs 대기업의 AWS 비용 전략 차이는?**
   - 각각의 상황에 맞는 최적화 방법

2. **비용과 성능의 트레이드오프를 어떻게 결정할까?**
   - 비즈니스 가치 기준의 의사결정

---

## 📚 다음 시간 예고

**AWS 공동 책임 모델**
- AWS와 고객의 보안 책임 분담
- 서비스별 책임 범위 차이
- 실제 보안 사고 사례 분석
- 고객 책임 영역 보안 체크리스트

---

## 📖 참고 자료

### AWS 공식 문서
- [AWS 요금](https://aws.amazon.com/pricing/)
- [EC2 요금](https://aws.amazon.com/ec2/pricing/)
- [EC2 예약 인스턴스](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html)
- [EC2 스팟 인스턴스](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html)
- [AWS 요금 계산기](https://calculator.aws/)
- [AWS 비용 관리](https://docs.aws.amazon.com/awsaccountbilling/)

---

> 💡 **오늘의 핵심**: AWS는 사용한 만큼만 지불하는 유연한 요금 체계를 제공합니다. 다양한 요금 모델을 이해하고 워크로드 특성에 맞게 선택하면 상당한 비용 절감이 가능합니다. 비용 최적화는 일회성이 아닌 지속적인 과정입니다!