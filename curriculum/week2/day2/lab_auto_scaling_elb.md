# 실습: Auto Scaling 그룹 및 로드 밸런서 구성

## 개요
이 실습에서는 Amazon EC2 Auto Scaling과 Elastic Load Balancing을 함께 구성하여 고가용성과 확장성을 갖춘 웹 애플리케이션 아키텍처를 구축합니다. 부하에 따라 자동으로 확장 및 축소되는 웹 서버 환경을 만들고, 여러 가용 영역에 걸쳐 트래픽을 분산하는 방법을 학습합니다.

## 학습 목표
- Application Load Balancer 생성 및 구성 방법 습득
- Auto Scaling 그룹 설정 및 조정 정책 구성 방법 이해
- 부하 테스트를 통한 Auto Scaling 동작 확인
- 고가용성을 위한 다중 가용 영역 아키텍처 구현
- CloudWatch를 활용한 모니터링 및 경보 설정 방법 학습

## 사전 준비 사항
- AWS 계정
- 기본 네트워크 지식
- EC2 인스턴스 생성 경험
- 웹 브라우저

## 실습 시간
약 2시간

## 실습 단계

### 1. VPC 및 서브넷 확인

#### 1.1 VPC 확인
1. AWS 관리 콘솔에 로그인합니다.
2. 상단 검색창에 "VPC"를 입력하거나 서비스 메뉴에서 "VPC"를 선택합니다.
3. 왼쪽 메뉴에서 "VPC"를 클릭합니다.
4. 기본 VPC를 확인하고 VPC ID를 메모합니다.

#### 1.2 서브넷 확인
1. 왼쪽 메뉴에서 "서브넷"을 클릭합니다.
2. 서로 다른 가용 영역에 있는 최소 2개의 퍼블릭 서브넷을 확인하고 ID를 메모합니다.
   - 퍼블릭 서브넷은 "라우팅 테이블" 열에서 인터넷 게이트웨이로의 라우팅이 있는 서브넷입니다.
   - 각 서브넷의 가용 영역(AZ)을 확인합니다.

### 2. 보안 그룹 생성

#### 2.1 로드 밸런서 보안 그룹 생성
1. 왼쪽 메뉴에서 "보안 그룹"을 클릭합니다.
2. "보안 그룹 생성" 버튼을 클릭합니다.
3. 다음 정보를 입력합니다:
   - 보안 그룹 이름: `lb-sg`
   - 설명: Security group for load balancer
   - VPC: 기본 VPC 선택
   - 인바운드 규칙:
     - 유형: HTTP
     - 포트 범위: 80
     - 소스: Anywhere (0.0.0.0/0)
4. "보안 그룹 생성" 버튼을 클릭합니다.

#### 2.2 웹 서버 보안 그룹 생성
1. "보안 그룹 생성" 버튼을 다시 클릭합니다.
2. 다음 정보를 입력합니다:
   - 보안 그룹 이름: `web-server-sg`
   - 설명: Security group for web servers
   - VPC: 기본 VPC 선택
   - 인바운드 규칙:
     - 유형: HTTP
     - 포트 범위: 80
     - 소스: 로드 밸런서 보안 그룹 ID 선택 (lb-sg)
     - 유형: SSH
     - 포트 범위: 22
     - 소스: 내 IP
3. "보안 그룹 생성" 버튼을 클릭합니다.

### 3. 시작 템플릿 생성

#### 3.1 AMI 선택
1. 상단 검색창에 "EC2"를 입력하거나 서비스 메뉴에서 "EC2"를 선택합니다.
2. 왼쪽 메뉴에서 "시작 템플릿"을 클릭합니다.
3. "시작 템플릿 생성" 버튼을 클릭합니다.
4. 다음 정보를 입력합니다:
   - 시작 템플릿 이름: `web-server-template`
   - 템플릿 버전 설명: Web server with Apache
   - Auto Scaling 지침: 체크

#### 3.2 템플릿 구성
1. "애플리케이션 및 OS 이미지"에서 "Amazon Linux 2023 AMI"를 선택합니다.
2. "인스턴스 유형"에서 "t2.micro"를 선택합니다.
3. "키 페어"에서 기존 키 페어를 선택하거나 새 키 페어를 생성합니다.
4. "네트워크 설정"에서 "고급"을 클릭하고 다음을 설정합니다:
   - 보안 그룹: 앞서 생성한 `web-server-sg` 선택
5. "스토리지"는 기본값을 유지합니다.
6. "고급 세부 정보"에서 "사용자 데이터"에 다음 스크립트를 입력합니다:

```bash
#!/bin/bash
# 패키지 업데이트 및 Apache 설치
yum update -y
yum install -y httpd stress

# Apache 서비스 시작 및 부팅 시 자동 시작 설정
systemctl start httpd
systemctl enable httpd

# 인스턴스 메타데이터에서 인스턴스 ID 및 가용 영역 가져오기
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
AVAILABILITY_ZONE=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/placement/availability-zone)

# 간단한 웹 페이지 생성
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Auto Scaling 데모</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #f0f0f0;
        }
        .container {
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            padding: 20px;
            max-width: 600px;
            width: 100%;
            text-align: center;
        }
        h1 {
            color: #232f3e;
        }
        .info {
            margin: 20px 0;
            padding: 15px;
            background-color: #f8f8f8;
            border-radius: 4px;
            text-align: left;
        }
        .footer {
            margin-top: 20px;
            font-size: 0.8em;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Auto Scaling 데모</h1>
        <div class="info">
            <p><strong>인스턴스 ID:</strong> $INSTANCE_ID</p>
            <p><strong>가용 영역:</strong> $AVAILABILITY_ZONE</p>
            <p><strong>시간:</strong> <span id="current-time"></span></p>
        </div>
        <div class="footer">
            AWS SAA 교육과정 - Auto Scaling 및 ELB 실습
        </div>
    </div>
    <script>
        function updateTime() {
            document.getElementById('current-time').textContent = new Date().toLocaleString();
        }
        updateTime();
        setInterval(updateTime, 1000);
    </script>
</body>
</html>
EOF
```

7. "시작 템플릿 생성" 버튼을 클릭합니다.

### 4. Application Load Balancer 생성

#### 4.1 로드 밸런서 기본 구성
1. 왼쪽 메뉴에서 "로드 밸런서"를 클릭합니다.
2. "로드 밸런서 생성" 버튼을 클릭합니다.
3. "Application Load Balancer"에서 "생성" 버튼을 클릭합니다.
4. 다음 정보를 입력합니다:
   - 로드 밸런서 이름: `web-alb`
   - 체계: 인터넷 연결
   - IP 주소 유형: IPv4
   - 네트워크 매핑:
     - VPC: 기본 VPC 선택
     - 매핑: 앞서 확인한 두 개의 퍼블릭 서브넷이 있는 가용 영역 선택
   - 보안 그룹: 앞서 생성한 `lb-sg` 선택

#### 4.2 리스너 및 대상 그룹 구성
1. "리스너 및 라우팅" 섹션에서:
   - 프로토콜: HTTP
   - 포트: 80
   - 기본 작업: 새 대상 그룹 생성
2. "대상 그룹 생성" 페이지에서:
   - 대상 유형 선택: 인스턴스
   - 대상 그룹 이름: `web-target-group`
   - 프로토콜: HTTP
   - 포트: 80
   - VPC: 기본 VPC 선택
   - 프로토콜 버전: HTTP1
   - 상태 확인:
     - 프로토콜: HTTP
     - 경로: /
     - 정상 임계값: 2
     - 비정상 임계값: 2
     - 제한 시간: 5초
     - 간격: 10초
     - 성공 코드: 200
3. "다음" 버튼을 클릭합니다.
4. 대상 등록 페이지에서 "대상 그룹만 생성"을 선택하고 "대상 그룹 생성" 버튼을 클릭합니다.
5. 로드 밸런서 생성 페이지로 돌아가서 "대상 그룹" 드롭다운에서 방금 생성한 `web-target-group`을 선택합니다.
6. "로드 밸런서 생성" 버튼을 클릭합니다.

### 5. Auto Scaling 그룹 생성

#### 5.1 기본 구성
1. 왼쪽 메뉴에서 "Auto Scaling 그룹"을 클릭합니다.
2. "Auto Scaling 그룹 생성" 버튼을 클릭합니다.
3. 다음 정보를 입력합니다:
   - Auto Scaling 그룹 이름: `web-asg`
   - 시작 템플릿: 앞서 생성한 `web-server-template` 선택
   - 버전: 최신
4. "다음" 버튼을 클릭합니다.

#### 5.2 네트워크 구성
1. "인스턴스 시작 옵션 선택" 페이지에서:
   - VPC: 기본 VPC 선택
   - 가용 영역 및 서브넷: 앞서 확인한 두 개의 퍼블릭 서브넷 선택
2. "다음" 버튼을 클릭합니다.

#### 5.3 로드 밸런서 연결
1. "고급 옵션 구성" 페이지에서:
   - 로드 밸런싱: "기존 로드 밸런서에 연결" 선택
   - 기존 로드 밸런서 대상 그룹에서 연결 선택: `web-target-group` 선택
   - 상태 확인:
     - 상태 확인 유형: ELB
     - 상태 확인 유예 기간: 120초
2. "다음" 버튼을 클릭합니다.

#### 5.4 그룹 크기 및 조정 정책 구성
1. "그룹 크기 및 조정 정책 구성" 페이지에서:
   - 원하는 용량: 2
   - 최소 용량: 2
   - 최대 용량: 6
   - 조정 정책: "대상 추적 조정 정책"
   - 지표 유형: 평균 CPU 사용률
   - 대상 값: 50
   - 인스턴스 워밍업: 120초
2. "다음" 버튼을 클릭합니다.

#### 5.5 알림 구성 (선택 사항)
1. "알림 추가" 페이지에서 필요에 따라 SNS 알림을 구성하거나 "다음" 버튼을 클릭합니다.

#### 5.6 태그 구성
1. "태그 추가" 페이지에서:
   - 키: Name
   - 값: web-server
   - "인스턴스에 태그 전파" 체크
2. "다음" 버튼을 클릭합니다.

#### 5.7 검토 및 생성
1. 구성 내용을 검토합니다.
2. "Auto Scaling 그룹 생성" 버튼을 클릭합니다.

### 6. 로드 밸런서 및 Auto Scaling 그룹 테스트

#### 6.1 로드 밸런서 접속 확인
1. EC2 대시보드에서 "로드 밸런서"를 클릭합니다.
2. 생성한 `web-alb`를 선택합니다.
3. "설명" 탭에서 "DNS 이름"을 복사합니다.
4. 새 브라우저 탭에서 복사한 DNS 이름을 붙여넣고 접속합니다.
5. 웹 페이지가 표시되고 인스턴스 ID와 가용 영역 정보가 보이는지 확인합니다.
6. 페이지를 여러 번 새로고침하여 요청이 여러 인스턴스로 분산되는지 확인합니다(인스턴스 ID가 변경됨).

#### 6.2 Auto Scaling 동작 확인
1. EC2 대시보드에서 "인스턴스"를 클릭합니다.
2. Auto Scaling 그룹에 의해 생성된 두 개의 인스턴스가 실행 중인지 확인합니다.
3. 인스턴스 중 하나에 SSH로 연결합니다:
   ```bash
   ssh -i your-key.pem ec2-user@instance-public-ip
   ```
4. CPU 부하를 생성하여 Auto Scaling 트리거:
   ```bash
   sudo stress --cpu 2 --timeout 600
   ```
5. 다른 SSH 세션을 열어 두 번째 인스턴스에도 연결하여 동일한 명령을 실행합니다.
6. EC2 대시보드에서 "Auto Scaling 그룹"을 클릭하고 `web-asg`를 선택합니다.
7. "활동" 탭을 모니터링하여 새 인스턴스가 시작되는지 확인합니다.
8. "인스턴스" 탭을 확인하여 추가 인스턴스가 시작되는지 확인합니다.
9. "모니터링" 탭에서 CPU 사용률 그래프를 확인합니다.

#### 6.3 CloudWatch 지표 확인
1. 상단 검색창에 "CloudWatch"를 입력하거나 서비스 메뉴에서 "CloudWatch"를 선택합니다.
2. 왼쪽 메뉴에서 "지표"를 클릭합니다.
3. "모든 지표"에서 "EC2"를 클릭합니다.
4. "Auto Scaling 그룹별"을 클릭합니다.
5. `web-asg` 그룹의 "CPUUtilization" 지표를 선택합니다.
6. 그래프에서 CPU 사용률이 증가하는 것을 확인합니다.

#### 6.4 스케일 인 확인
1. SSH 세션에서 실행 중인 stress 명령을 중지합니다(Ctrl+C).
2. CloudWatch에서 CPU 사용률이 감소하는 것을 확인합니다.
3. Auto Scaling 그룹의 "활동" 탭을 모니터링하여 일정 시간 후 인스턴스가 종료되는지 확인합니다.
4. "인스턴스" 탭에서 인스턴스 수가 다시 2개로 감소하는지 확인합니다.

### 7. 정리

#### 7.1 Auto Scaling 그룹 삭제
1. EC2 대시보드에서 "Auto Scaling 그룹"을 클릭합니다.
2. `web-asg`를 선택합니다.
3. "작업" 버튼을 클릭하고 "삭제"를 선택합니다.
4. 확인 대화 상자에서 "삭제"를 클릭합니다.

#### 7.2 로드 밸런서 삭제
1. EC2 대시보드에서 "로드 밸런서"를 클릭합니다.
2. `web-alb`를 선택합니다.
3. "작업" 버튼을 클릭하고 "삭제"를 선택합니다.
4. 확인 대화 상자에서 "예, 삭제합니다"를 클릭합니다.

#### 7.3 대상 그룹 삭제
1. EC2 대시보드에서 "대상 그룹"을 클릭합니다.
2. `web-target-group`을 선택합니다.
3. "작업" 버튼을 클릭하고 "삭제"를 선택합니다.
4. 확인 대화 상자에서 "예, 삭제합니다"를 클릭합니다.

#### 7.4 시작 템플릿 삭제
1. EC2 대시보드에서 "시작 템플릿"을 클릭합니다.
2. `web-server-template`을 선택합니다.
3. "작업" 버튼을 클릭하고 "템플릿 삭제"를 선택합니다.
4. 확인 대화 상자에서 "삭제"를 클릭합니다.

#### 7.5 보안 그룹 삭제
1. EC2 대시보드에서 "보안 그룹"을 클릭합니다.
2. `web-server-sg`를 선택합니다.
3. "작업" 버튼을 클릭하고 "보안 그룹 삭제"를 선택합니다.
4. 확인 대화 상자에서 "삭제"를 클릭합니다.
5. `lb-sg`에 대해서도 동일한 과정을 반복합니다.

## 도전 과제

### 도전 과제 1: 단계 조정 정책 구현
1. 대상 추적 조정 정책 대신 단계 조정 정책을 사용하여 새 Auto Scaling 그룹을 생성합니다.
2. 다음과 같은 조정 정책을 구성합니다:
   - CPU 사용률 > 70%: 2개 인스턴스 추가
   - CPU 사용률 > 85%: 3개 인스턴스 추가
   - CPU 사용률 < 40%: 1개 인스턴스 제거
   - CPU 사용률 < 20%: 2개 인스턴스 제거
3. stress 명령을 사용하여 다양한 수준의 CPU 부하를 생성하고 정책이 예상대로 작동하는지 확인합니다.

### 도전 과제 2: 사용자 정의 지표 기반 조정
1. CloudWatch 사용자 정의 지표를 생성하여 메모리 사용률을 모니터링하는 스크립트를 작성합니다.
2. 이 사용자 정의 지표를 기반으로 조정 정책을 구성합니다.
3. 메모리 집약적인 작업을 실행하여 조정 정책이 작동하는지 확인합니다.

### 도전 과제 3: 다중 리전 배포
1. 두 번째 AWS 리전에 동일한 아키텍처를 배포합니다.
2. Route 53을 사용하여 두 리전의 로드 밸런서 간에 트래픽을 분산합니다.
3. 한 리전의 로드 밸런서를 비활성화하여 장애 조치가 작동하는지 확인합니다.

## 결론
이 실습을 통해 Amazon EC2 Auto Scaling과 Elastic Load Balancing을 함께 구성하여 고가용성과 확장성을 갖춘 웹 애플리케이션 아키텍처를 구축하는 방법을 학습했습니다. 부하에 따라 자동으로 확장 및 축소되는 웹 서버 환경을 만들고, 여러 가용 영역에 걸쳐 트래픽을 분산하는 방법을 실습했습니다. 이러한 기술은 AWS에서 안정적이고 확장 가능한 애플리케이션을 구축하는 데 필수적입니다.

## 참고 자료
- [Amazon EC2 Auto Scaling 사용 설명서](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)
- [Elastic Load Balancing 사용 설명서](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html)
- [Application Load Balancer 사용 설명서](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
- [Auto Scaling 그룹에서 조정 정책 사용](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-simple-step.html)