# Day 5-3: 네트워크 모니터링

## 📚 학습 목표
- VPC Flow Logs의 개념과 활용 방법 이해
- CloudWatch를 통한 네트워크 모니터링 구성
- 네트워크 트래픽 분석 및 패턴 파악
- 보안 이벤트 탐지 및 대응 방법 습득

---

## 📊 VPC Flow Logs 개요 (30분)

### VPC Flow Logs란?

#### Flow Logs의 개념
```
VPC Flow Logs = 네트워크 트래픽 기록
- VPC, 서브넷, ENI 레벨에서 IP 트래픽 캡처
- 연결 허용/거부 정보 기록
- 네트워크 문제 해결 및 보안 분석 도구

비유: 건물 출입 기록부
- 누가 (소스 IP)
- 언제 (타임스탬프)
- 어디로 (목적지 IP)
- 무엇을 (포트, 프로토콜)
- 결과는 (허용/거부)
```

#### Flow Logs 수집 레벨
```
1. VPC 레벨
   - VPC 내 모든 ENI 트래픽 캡처
   - 가장 포괄적인 수집
   - 전체 네트워크 가시성 확보

2. 서브넷 레벨
   - 특정 서브넷 내 ENI 트래픽만 캡처
   - 계층별 분석 가능
   - 세밀한 모니터링

3. ENI 레벨
   - 특정 네트워크 인터페이스만 캡처
   - 인스턴스별 상세 분석
   - 문제 해결 시 유용

선택 기준:
- 전체 모니터링: VPC 레벨
- 계층별 분석: 서브넷 레벨
- 특정 인스턴스: ENI 레벨
```

### Flow Logs 필드 구조

#### 기본 필드 설명
```
Flow Log 레코드 예시:
2 123456789012 eni-1235b8ca123456789 172.31.16.139 172.31.16.21 20641 22 6 20 4249 1418530010 1418530070 ACCEPT OK

필드 구조:
1. version: 2 (Flow Logs 버전)
2. account-id: 123456789012 (AWS 계정 ID)
3. interface-id: eni-1235b8ca123456789 (네트워크 인터페이스 ID)
4. srcaddr: 172.31.16.139 (소스 IP 주소)
5. dstaddr: 172.31.16.21 (목적지 IP 주소)
6. srcport: 20641 (소스 포트)
7. dstport: 22 (목적지 포트)
8. protocol: 6 (프로토콜 번호, 6=TCP)
9. packets: 20 (패킷 수)
10. bytes: 4249 (바이트 수)
11. windowstart: 1418530010 (시작 시간)
12. windowend: 1418530070 (종료 시간)
13. action: ACCEPT (허용/거부)
14. flowlogstatus: OK (로그 상태)
```

#### 프로토콜 번호 참조
```
주요 프로토콜 번호:
- 1: ICMP (ping)
- 6: TCP (HTTP, HTTPS, SSH)
- 17: UDP (DNS)
- 47: GRE
- 50: ESP (IPSec)

포트 번호 참조:
- 22: SSH
- 53: DNS
- 80: HTTP
- 443: HTTPS
- 3306: MySQL
- 5432: PostgreSQL
```

---

## 🔧 VPC Flow Logs 설정 실습 (45분)

### 실습 1: VPC Flow Logs 활성화

#### CloudWatch Logs 대상으로 설정
```
1. CloudWatch 로그 그룹 생성
   - CloudWatch → Logs → Log groups
   - "Create log group" 클릭
   - Log group name: /aws/vpc/flowlogs
   - Retention setting: 7 days (비용 절약)
   - "Create" 클릭

2. VPC Flow Logs 생성
   - VPC → Your VPCs → SAA-Study-VPC 선택
   - Flow logs 탭 → "Create flow log"
   - Filter: All (Accept와 Reject 모두)
   - Destination: Send to CloudWatch Logs
   - Destination log group: /aws/vpc/flowlogs
   - IAM role: 새 역할 생성 또는 기존 역할 선택
   - Log record format: AWS default format
   - "Create flow log" 클릭

3. IAM 역할 설정 (자동 생성 시)
   - 역할 이름: flowlogsRole
   - 정책: CloudWatchLogsFullAccess
   - 신뢰 관계: VPC Flow Logs 서비스
```

#### S3 대상으로 설정 (선택사항)
```
1. S3 버킷 생성
   - 버킷 이름: saa-study-flowlogs-[랜덤숫자]
   - 리전: ap-northeast-2
   - 기본 설정으로 생성

2. VPC Flow Logs 생성 (S3)
   - Destination: Send to S3 bucket
   - S3 bucket ARN: arn:aws:s3:::saa-study-flowlogs-[번호]
   - Log file format: Parquet (분석 최적화)
   - Hive-compatible S3 prefixes: 활성화
   - Partition logs by time: Every 1 hour

장점:
- 장기 보관 비용 효율적
- 대용량 데이터 분석 가능
- Athena, QuickSight 연동 가능
```

### 실습 2: 트래픽 생성 및 로그 확인

#### 테스트 트래픽 생성
```
1. 정상 트래픽 생성
   # 퍼블릭 인스턴스에서 실행
   curl http://httpbin.org/ip
   ping -c 5 8.8.8.8
   
   # 프라이빗 인스턴스 접속
   ssh ec2-user@[프라이빗-IP]

2. 차단 트래픽 생성
   # 허용되지 않은 포트로 접속 시도
   telnet [프라이빗-IP] 3306
   nc -zv [프라이빗-IP] 80

3. 대용량 트래픽 생성 (선택사항)
   # 큰 파일 다운로드
   wget https://releases.ubuntu.com/20.04/ubuntu-20.04.6-desktop-amd64.iso
```

#### Flow Logs 확인
```
1. CloudWatch Logs에서 확인
   - CloudWatch → Logs → Log groups
   - /aws/vpc/flowlogs 선택
   - Log streams 확인 (ENI별로 생성)
   - 최신 로그 스트림 선택

2. 로그 내용 분석
   # SSH 연결 로그 예시
   2 123456789012 eni-xxx 10.0.1.100 10.0.11.50 54321 22 6 10 520 1640000000 1640000060 ACCEPT OK
   
   해석:
   - 소스: 10.0.1.100 (퍼블릭 서브넷)
   - 목적지: 10.0.11.50 (프라이빗 서브넷)
   - 포트: 22 (SSH)
   - 결과: ACCEPT (허용)

3. 거부된 트래픽 확인
   # 차단된 연결 시도
   2 123456789012 eni-xxx 10.0.1.100 10.0.11.50 54322 3306 6 0 0 1640000000 1640000060 REJECT OK
   
   해석:
   - 포트: 3306 (MySQL)
   - 패킷/바이트: 0 (연결 실패)
   - 결과: REJECT (거부)
```

### 실습 3: 커스텀 Flow Logs 형식

#### 필요한 필드만 선택
```
1. 커스텀 형식 Flow Logs 생성
   - Log record format: Custom format 선택
   - 필요한 필드만 선택:
     * srcaddr (소스 IP)
     * dstaddr (목적지 IP)
     * srcport (소스 포트)
     * dstport (목적지 포트)
     * protocol (프로토콜)
     * action (허용/거부)
     * start (시작 시간)
     * end (종료 시간)

2. 커스텀 형식의 장점
   - 로그 크기 감소 (비용 절약)
   - 분석 속도 향상
   - 필요한 정보만 집중

3. 로그 형식 예시
   10.0.1.100 10.0.11.50 54321 22 6 ACCEPT 1640000000 1640000060
   
   간결하고 분석하기 쉬운 형태
```

---

## 📈 CloudWatch 네트워크 모니터링 (45분)

### CloudWatch 메트릭 활용

#### VPC 관련 주요 메트릭
```
NAT Gateway 메트릭:
- BytesInFromDestination: 인터넷에서 받은 바이트
- BytesInFromSource: 프라이빗 서브넷에서 받은 바이트
- BytesOutToDestination: 인터넷으로 보낸 바이트
- BytesOutToSource: 프라이빗 서브넷으로 보낸 바이트
- ActiveConnectionCount: 활성 연결 수
- ConnectionAttemptCount: 연결 시도 수
- ErrorPortAllocation: 포트 할당 오류
- PacketsDropCount: 드롭된 패킷 수

VPC Endpoint 메트릭:
- PacketsIn/PacketsOut: 패킷 수
- BytesIn/BytesOut: 바이트 수
- 서비스별 API 호출 수
```

### 실습 1: CloudWatch 대시보드 생성

#### 네트워크 모니터링 대시보드
```
1. 대시보드 생성
   - CloudWatch → Dashboards → "Create dashboard"
   - Dashboard name: VPC-Network-Monitoring
   - "Create dashboard" 클릭

2. NAT Gateway 위젯 추가
   - "Add widget" → Line graph
   - Metrics → AWS/NatGateway
   - NAT Gateway 선택
   - 메트릭 선택:
     * BytesInFromSource
     * BytesOutToDestination
   - Period: 5 minutes
   - Widget title: NAT Gateway Traffic

3. VPC Flow Logs 위젯 추가
   - "Add widget" → Number
   - Metrics → CWLogs → Log Group Metrics
   - /aws/vpc/flowlogs 선택
   - IncomingLogEvents 메트릭
   - Widget title: Flow Logs Volume

4. 네트워크 연결 위젯
   - "Add widget" → Line graph
   - NAT Gateway ActiveConnectionCount
   - Widget title: Active Connections
```

### 실습 2: CloudWatch Alarms 설정

#### 네트워크 이상 상황 알림
```
1. 높은 트래픽 알림
   - CloudWatch → Alarms → "Create alarm"
   - Metric: NAT Gateway BytesOutToDestination
   - Statistic: Sum
   - Period: 5 minutes
   - Threshold: > 1GB (1073741824 bytes)
   - Alarm name: High-NAT-Traffic
   - Action: SNS 토픽으로 이메일 발송

2. 연결 실패 알림
   - Metric: NAT Gateway ErrorPortAllocation
   - Threshold: > 10 (5분간)
   - Alarm name: NAT-Connection-Errors

3. Flow Logs 중단 알림
   - Metric: CWLogs IncomingLogEvents
   - Threshold: < 1 (10분간)
   - Alarm name: FlowLogs-Missing
   - 네트워크 모니터링 중단 감지
```

### 실습 3: CloudWatch Insights 쿼리

#### Flow Logs 분석 쿼리
```
1. 상위 트래픽 소스 분석
fields @timestamp, srcaddr, dstaddr, bytes
| filter action = "ACCEPT"
| stats sum(bytes) as total_bytes by srcaddr
| sort total_bytes desc
| limit 10

2. 거부된 연결 분석
fields @timestamp, srcaddr, dstaddr, dstport, protocol
| filter action = "REJECT"
| stats count() as reject_count by srcaddr, dstport
| sort reject_count desc
| limit 20

3. 시간대별 트래픽 패턴
fields @timestamp, bytes
| filter action = "ACCEPT"
| bin(@timestamp, 1h) as hour
| stats sum(bytes) as hourly_bytes by hour
| sort hour

4. 보안 이벤트 탐지
fields @timestamp, srcaddr, dstaddr, dstport
| filter action = "REJECT" and dstport in [22, 3389, 1433, 3306]
| stats count() as attempts by srcaddr, dstport
| sort attempts desc
```

---

## 🔍 네트워크 트래픽 분석 (30분)

### 트래픽 패턴 분석

#### 정상 vs 비정상 패턴 식별
```
정상 트래픽 패턴:
✅ 예측 가능한 시간대별 변화
✅ 비즈니스 로직에 맞는 포트 사용
✅ 내부 통신이 대부분
✅ 낮은 거부율 (< 5%)

비정상 트래픽 패턴:
🚨 갑작스러운 트래픽 급증
🚨 비정상적인 포트 스캔
🚨 외부에서 내부 포트 접근 시도
🚨 높은 거부율 (> 20%)
🚨 알려진 악성 IP에서 접근
```

#### 실습: 트래픽 분석 시나리오
```
1. 정상 트래픽 분석
   # 웹 서버 트래픽 패턴
   - 80, 443 포트로 인바운드 트래픽
   - 임시 포트로 아웃바운드 응답
   - 대부분 ACCEPT 상태

2. 의심스러운 활동 탐지
   # 포트 스캔 탐지
   - 동일 소스에서 여러 포트 접근 시도
   - 대부분 REJECT 상태
   - 짧은 시간 내 대량 시도

3. DDoS 공격 패턴
   # 대량 트래픽 패턴
   - 다수 소스에서 동일 목적지
   - 비정상적으로 높은 패킷 수
   - 네트워크 리소스 고갈

CloudWatch Insights 쿼리 예시:
# 포트 스캔 탐지
fields @timestamp, srcaddr, dstport
| filter action = "REJECT"
| stats count_distinct(dstport) as scanned_ports by srcaddr
| sort scanned_ports desc
| limit 10
```

### 성능 최적화 분석

#### 네트워크 병목 지점 식별
```
분석 포인트:
1. 대역폭 사용률
   - NAT Gateway 처리량 확인
   - 인스턴스별 네트워크 사용률
   - 피크 시간대 패턴 분석

2. 연결 패턴
   - 동시 연결 수 모니터링
   - 연결 지속 시간 분석
   - 연결 실패율 추적

3. 지연시간 분석
   - 응답 시간 패턴
   - 네트워크 홉 수 최적화
   - 지역별 성능 차이

최적화 방안:
- 트래픽이 많은 서비스: VPC 엔드포인트 적용
- 지연시간 민감: 배치 그룹 활용
- 대역폭 부족: 인스턴스 타입 업그레이드
```

---

## 🛡️ 보안 이벤트 탐지 및 대응 (30분)

### 보안 이벤트 탐지

#### 자동화된 보안 모니터링
```
1. CloudWatch 알림 기반 탐지
   # 의심스러운 활동 알림
   - 높은 거부율 (> 100회/분)
   - 알려진 악성 IP 접근
   - 비정상적인 포트 접근

2. Lambda 기반 자동 대응
   # 실시간 분석 및 대응
   - Flow Logs 실시간 분석
   - 임계값 초과 시 자동 차단
   - 보안 그룹 규칙 자동 업데이트

3. 써드파티 도구 연동
   - SIEM 도구와 연동
   - 위협 인텔리전스 활용
   - 머신러닝 기반 이상 탐지
```

#### 실습: 보안 이벤트 대응 자동화
```
1. Lambda 함수 생성 (개념적 예시)
import json
import boto3

def lambda_handler(event, context):
    # CloudWatch 알림에서 트리거
    ec2 = boto3.client('ec2')
    
    # 악성 IP 추출
    malicious_ip = extract_ip_from_alarm(event)
    
    # 보안 그룹에 차단 규칙 추가
    response = ec2.authorize_security_group_ingress(
        GroupId='sg-xxxxxxxxx',
        IpPermissions=[
            {
                'IpProtocol': '-1',
                'IpRanges': [
                    {
                        'CidrIp': f'{malicious_ip}/32',
                        'Description': 'Auto-blocked malicious IP'
                    }
                ]
            }
        ]
    )
    
    # 알림 발송
    send_notification(f"Blocked malicious IP: {malicious_ip}")
    
    return {
        'statusCode': 200,
        'body': json.dumps('Security response completed')
    }

2. CloudWatch 알림과 연동
   - 알림 액션: Lambda 함수 호출
   - 자동 차단 및 알림 발송
   - 로그 기록 및 추적
```

### 인시던트 대응 절차

#### 보안 사고 대응 플레이북
```
1단계: 탐지 및 분석
- Flow Logs에서 이상 패턴 확인
- 영향 범위 및 심각도 평가
- 공격 벡터 및 방법 분석

2단계: 격리 및 차단
- 악성 IP 즉시 차단
- 영향받은 인스턴스 격리
- 추가 확산 방지 조치

3단계: 제거 및 복구
- 악성 코드 제거
- 시스템 무결성 확인
- 서비스 정상화

4단계: 사후 분석
- 근본 원인 분석
- 보안 정책 개선
- 재발 방지 대책 수립

문서화 요소:
- 사고 발생 시간 및 경위
- 대응 조치 내역
- 영향 범위 및 피해 규모
- 개선 사항 및 교훈
```

---

## 📊 모니터링 최적화 및 비용 관리 (15분)

### Flow Logs 비용 최적화

#### 비용 구조 이해
```
Flow Logs 비용 요소:
1. 로그 수집 비용
   - CloudWatch Logs: $0.76/GB (수집)
   - S3: $0.023/GB (저장)

2. 로그 저장 비용
   - CloudWatch Logs: $0.76/GB/월
   - S3 Standard: $0.025/GB/월
   - S3 IA: $0.0138/GB/월

3. 로그 분석 비용
   - CloudWatch Insights: $0.0076/GB (스캔)
   - Athena: $6.25/TB (스캔)

비용 최적화 전략:
- 필요한 필드만 수집 (커스텀 형식)
- 적절한 보존 기간 설정
- S3로 장기 보관
- 샘플링 활용 (전체 트래픽의 일부만)
```

#### 모니터링 효율성 향상
```
효율적인 모니터링 전략:
1. 계층적 모니터링
   - 전체: VPC 레벨 기본 모니터링
   - 중요: 서브넷 레벨 상세 모니터링
   - 특별: ENI 레벨 집중 모니터링

2. 알림 최적화
   - 중요도별 알림 분류
   - 임계값 조정으로 노이즈 감소
   - 알림 피로도 방지

3. 자동화 활용
   - 정기적인 보고서 자동 생성
   - 이상 패턴 자동 탐지
   - 대응 조치 자동화
```

---

## 📝 핵심 정리

### VPC Flow Logs 핵심 개념
1. **수집 레벨**: VPC, 서브넷, ENI 레벨별 선택 가능
2. **로그 형식**: 기본 형식 vs 커스텀 형식
3. **저장 대상**: CloudWatch Logs vs S3
4. **분석 도구**: CloudWatch Insights, Athena

### 네트워크 모니터링 전략
- **실시간 모니터링**: CloudWatch 메트릭 및 알림
- **트래픽 분석**: Flow Logs 기반 패턴 분석
- **보안 감시**: 이상 행동 탐지 및 자동 대응
- **성능 최적화**: 병목 지점 식별 및 개선

### 실무 적용 포인트
- **비용 효율성**: 필요한 정보만 수집하여 비용 최적화
- **보안 강화**: 자동화된 위협 탐지 및 대응
- **운영 효율성**: 대시보드와 알림으로 능동적 관리
- **규정 준수**: 로그 보관 및 감사 추적 체계

---

## 🤔 토론 주제

1. **모든 트래픽을 로깅해야 할까?**
   - 보안 vs 비용 vs 성능의 균형점

2. **실시간 대응 vs 사후 분석?**
   - 자동화 수준과 인간 개입의 적절한 조합

---

## 📚 다음 시간 예고

**주간 과제 설명 및 시작**
- "안전한 웹 서비스 환경 구축" 과제 상세 설명
- 온라인 쇼핑몰 인프라 설계 시나리오
- ReadOnlyAccess 역할 부여 방법
- 과제 진행 가이드라인

---

> 💡 **오늘의 핵심**: 네트워크 모니터링은 보안과 성능 최적화의 핵심입니다. VPC Flow Logs를 통해 네트워크 가시성을 확보하고, CloudWatch를 활용한 실시간 모니터링으로 능동적인 네트워크 관리를 구현하세요!