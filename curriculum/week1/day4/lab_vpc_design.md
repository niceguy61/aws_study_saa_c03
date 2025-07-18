# VPC 설계 및 구현 실습

## 개요
이 실습에서는 AWS VPC(Virtual Private Cloud)를 설계하고 구현하는 과정을 단계별로 진행합니다. 이론적으로 학습한 VPC 개념과 네트워킹 지식을 실제 환경에 적용하여 안전하고 확장 가능한 네트워크 아키텍처를 구축하는 방법을 배웁니다.

## 학습 목표
- 요구 사항에 맞는 VPC 아키텍처 설계
- 다중 가용 영역 구성을 통한 고가용성 네트워크 구현
- 퍼블릭 및 프라이빗 서브넷 구성 및 라우팅 설정
- 보안 그룹 및 네트워크 ACL을 활용한 네트워크 보안 구현
- VPC 엔드포인트를 통한 AWS 서비스 프라이빗 액세스 구성

## 사전 준비 사항
- AWS 계정 액세스
- AWS Management Console 기본 사용법 숙지
- 기본적인 네트워킹 개념 이해
- AWS CLI 설치 (선택 사항)

## 실습 시나리오
여러분은 스타트업 기업의 클라우드 인프라 엔지니어로서, 회사의 새로운 웹 애플리케이션을 위한 AWS 네트워크 인프라를 설계하고 구축해야 합니다. 이 애플리케이션은 다음과 같은 요구 사항을 가지고 있습니다:

1. 고가용성 및 내결함성을 위한 다중 가용 영역 구성
2. 웹 서버는 인터넷에서 접근 가능해야 함
3. 애플리케이션 서버와 데이터베이스는 인터넷에서 직접 접근할 수 없어야 함
4. 모든 서버는 소프트웨어 업데이트를 위해 인터넷에 접근할 수 있어야 함
5. AWS 서비스(S3, DynamoDB 등)에 대한 안전한 액세스 필요

## 실습 1: VPC 및 서브넷 설계

### 1.1 VPC 설계 계획
먼저 VPC 및 서브넷 구성을 계획합니다.

**VPC 설계 사양**:
- VPC CIDR: 10.0.0.0/16 (65,536개 IP 주소)
- 리전: ap-northeast-2 (서울)
- 가용 영역: ap-northeast-2a, ap-northeast-2c (2개 AZ 사용)
- 서브넷 구성:
  - 퍼블릭 서브넷 (각 AZ당 1개): 웹 서버, NAT 게이트웨이 배치
  - 프라이빗 애플리케이션 서브넷 (각 AZ당 1개): 애플리케이션 서버 배치
  - 프라이빗 데이터베이스 서브넷 (각 AZ당 1개): 데이터베이스 서버 배치

**서브넷 CIDR 계획**:
```
ap-northeast-2a:
- 퍼블릭 서브넷: 10.0.0.0/24 (256개 IP)
- 프라이빗 애플리케이션 서브넷: 10.0.1.0/24 (256개 IP)
- 프라이빗 데이터베이스 서브넷: 10.0.2.0/24 (256개 IP)

ap-northeast-2c:
- 퍼블릭 서브넷: 10.0.3.0/24 (256개 IP)
- 프라이빗 애플리케이션 서브넷: 10.0.4.0/24 (256개 IP)
- 프라이빗 데이터베이스 서브넷: 10.0.5.0/24 (256개 IP)
```
### 1.2 VPC 생성
AWS Management Console을 사용하여 VPC를 생성합니다.

1. AWS Management Console에 로그인합니다.
2. 리전 선택기에서 "아시아 태평양(서울) ap-northeast-2"를 선택합니다.
3. 서비스 메뉴에서 "VPC"를 선택합니다.
4. "VPC 생성" 버튼을 클릭합니다.
5. "VPC 등" 옵션을 선택합니다.
6. 다음 설정을 구성합니다:
   - 생성할 리소스: "VPC 등"
   - 이름 태그 자동 생성: "Lab-VPC"
   - IPv4 CIDR 블록: "10.0.0.0/16"
   - IPv6 CIDR 블록: "IPv6 CIDR 블록 없음"
   - 테넌시: "기본값"
   - 가용 영역(AZ) 수: "2"
   - 퍼블릭 서브넷 수: "2"
   - 프라이빗 서브넷 수: "4"
   - NAT 게이트웨이: "AZ당 1개"
   - VPC 엔드포인트: "S3 게이트웨이"
7. "VPC 생성" 버튼을 클릭합니다.
8. VPC 및 관련 리소스가 생성될 때까지 기다립니다.

### 1.3 생성된 리소스 확인
VPC 생성이 완료되면 다음 리소스가 생성되었는지 확인합니다:

1. VPC (10.0.0.0/16)
2. 서브넷:
   - 퍼블릭 서브넷 2개
   - 프라이빗 서브넷 4개
3. 라우팅 테이블:
   - 퍼블릭 라우팅 테이블
   - 프라이빗 라우팅 테이블
4. 인터넷 게이트웨이
5. NAT 게이트웨이 2개
6. S3 게이트웨이 엔드포인트

### 1.4 서브넷 태그 지정
생성된 서브넷에 적절한 태그를 지정하여 용도를 명확히 합니다.

1. VPC 대시보드에서 "서브넷"을 선택합니다.
2. 각 서브넷을 선택하고 "작업" > "태그 관리"를 클릭합니다.
3. 다음과 같이 태그를 추가합니다:
   - 퍼블릭 서브넷: Name = "Public-Subnet-AZ1", "Public-Subnet-AZ2"
   - 프라이빗 애플리케이션 서브넷: Name = "Private-App-Subnet-AZ1", "Private-App-Subnet-AZ2"
   - 프라이빗 데이터베이스 서브넷: Name = "Private-DB-Subnet-AZ1", "Private-DB-Subnet-AZ2"
4. "저장"을 클릭합니다.

## 실습 2: 보안 구성

### 2.1 보안 그룹 생성
애플리케이션의 각 계층에 대한 보안 그룹을 생성합니다.

#### 웹 서버 보안 그룹
1. VPC 대시보드에서 "보안 그룹"을 선택합니다.
2. "보안 그룹 생성" 버튼을 클릭합니다.
3. 다음 설정을 구성합니다:
   - 보안 그룹 이름: "Web-Server-SG"
   - 설명: "Web servers security group"
   - VPC: 생성한 VPC 선택
4. 인바운드 규칙:
   - HTTP(80): 소스 0.0.0.0/0
   - HTTPS(443): 소스 0.0.0.0/0
   - SSH(22): 소스 [관리자 IP]/32 (보안을 위해 특정 IP만 허용)
5. 아웃바운드 규칙: 기본값 유지 (모든 트래픽 허용)
6. "보안 그룹 생성" 버튼을 클릭합니다.

#### 애플리케이션 서버 보안 그룹
1. "보안 그룹 생성" 버튼을 클릭합니다.
2. 다음 설정을 구성합니다:
   - 보안 그룹 이름: "App-Server-SG"
   - 설명: "Application servers security group"
   - VPC: 생성한 VPC 선택
3. 인바운드 규칙:
   - 사용자 지정 TCP(8080): 소스 Web-Server-SG
   - SSH(22): 소스 [관리자 IP]/32 (보안을 위해 특정 IP만 허용)
4. 아웃바운드 규칙: 기본값 유지 (모든 트래픽 허용)
5. "보안 그룹 생성" 버튼을 클릭합니다.

#### 데이터베이스 보안 그룹
1. "보안 그룹 생성" 버튼을 클릭합니다.
2. 다음 설정을 구성합니다:
   - 보안 그룹 이름: "Database-SG"
   - 설명: "Database servers security group"
   - VPC: 생성한 VPC 선택
3. 인바운드 규칙:
   - MySQL/Aurora(3306): 소스 App-Server-SG
4. 아웃바운드 규칙: 기본값 유지 (모든 트래픽 허용)
5. "보안 그룹 생성" 버튼을 클릭합니다.

### 2.2 네트워크 ACL 구성
기본 네트워크 ACL을 검토하고 필요한 경우 사용자 지정 네트워크 ACL을 구성합니다.

1. VPC 대시보드에서 "네트워크 ACL"을 선택합니다.
2. 기본 네트워크 ACL을 선택하고 규칙을 검토합니다.
3. 필요한 경우 사용자 지정 네트워크 ACL을 생성합니다:
   - "네트워크 ACL 생성" 버튼 클릭
   - 이름: "Custom-NACL"
   - VPC: 생성한 VPC 선택
   - 인바운드 규칙:
     - 100: HTTP(80) 허용, 소스 0.0.0.0/0
     - 110: HTTPS(443) 허용, 소스 0.0.0.0/0
     - 120: SSH(22) 허용, 소스 [관리자 IP]/32
     - 130: 임시 포트(1024-65535) 허용, 소스 0.0.0.0/0
   - 아웃바운드 규칙:
     - 100: HTTP(80) 허용, 대상 0.0.0.0/0
     - 110: HTTPS(443) 허용, 대상 0.0.0.0/0
     - 120: 임시 포트(1024-65535) 허용, 대상 0.0.0.0/0
4. "생성" 버튼을 클릭합니다.
5. 필요한 서브넷과 사용자 지정 네트워크 ACL을 연결합니다.

## 실습 3: EC2 인스턴스 배포

### 3.1 웹 서버 인스턴스 배포
퍼블릭 서브넷에 웹 서버 인스턴스를 배포합니다.

1. AWS Management Console에서 "EC2" 서비스로 이동합니다.
2. "인스턴스 시작" 버튼을 클릭합니다.
3. 다음 설정을 구성합니다:
   - 이름: "Web-Server-1"
   - AMI: Amazon Linux 2 선택
   - 인스턴스 유형: t2.micro 선택
   - 키 페어: 기존 키 페어 선택 또는 새 키 페어 생성
   - 네트워크 설정:
     - VPC: 생성한 VPC 선택
     - 서브넷: Public-Subnet-AZ1 선택
     - 퍼블릭 IP 자동 할당: 활성화
     - 보안 그룹: Web-Server-SG 선택
   - 고급 세부 정보:
     - 사용자 데이터:
     ```bash
     #!/bin/bash
     yum update -y
     yum install -y httpd
     systemctl start httpd
     systemctl enable httpd
     echo "<h1>Hello from Web Server 1</h1>" > /var/www/html/index.html
     ```
4. "인스턴스 시작" 버튼을 클릭합니다.

### 3.2 애플리케이션 서버 인스턴스 배포
프라이빗 애플리케이션 서브넷에 애플리케이션 서버 인스턴스를 배포합니다.

1. "인스턴스 시작" 버튼을 클릭합니다.
2. 다음 설정을 구성합니다:
   - 이름: "App-Server-1"
   - AMI: Amazon Linux 2 선택
   - 인스턴스 유형: t2.micro 선택
   - 키 페어: 기존 키 페어 선택
   - 네트워크 설정:
     - VPC: 생성한 VPC 선택
     - 서브넷: Private-App-Subnet-AZ1 선택
     - 퍼블릭 IP 자동 할당: 비활성화
     - 보안 그룹: App-Server-SG 선택
   - 고급 세부 정보:
     - 사용자 데이터:
     ```bash
     #!/bin/bash
     yum update -y
     yum install -y java-1.8.0-openjdk
     echo "#!/bin/bash" > /home/ec2-user/run-app.sh
     echo "java -jar /home/ec2-user/app.jar" >> /home/ec2-user/run-app.sh
     chmod +x /home/ec2-user/run-app.sh
     ```
3. "인스턴스 시작" 버튼을 클릭합니다.

### 3.3 데이터베이스 인스턴스 배포
이 실습에서는 EC2 인스턴스를 사용하여 데이터베이스를 시뮬레이션합니다. 실제 환경에서는 Amazon RDS를 사용하는 것이 좋습니다.

1. "인스턴스 시작" 버튼을 클릭합니다.
2. 다음 설정을 구성합니다:
   - 이름: "DB-Server-1"
   - AMI: Amazon Linux 2 선택
   - 인스턴스 유형: t2.micro 선택
   - 키 페어: 기존 키 페어 선택
   - 네트워크 설정:
     - VPC: 생성한 VPC 선택
     - 서브넷: Private-DB-Subnet-AZ1 선택
     - 퍼블릭 IP 자동 할당: 비활성화
     - 보안 그룹: Database-SG 선택
   - 고급 세부 정보:
     - 사용자 데이터:
     ```bash
     #!/bin/bash
     yum update -y
     yum install -y mariadb-server
     systemctl start mariadb
     systemctl enable mariadb
     mysql -e "CREATE DATABASE testdb;"
     mysql -e "CREATE USER 'testuser'@'%' IDENTIFIED BY 'password';"
     mysql -e "GRANT ALL PRIVILEGES ON testdb.* TO 'testuser'@'%';"
     mysql -e "FLUSH PRIVILEGES;"
     ```
3. "인스턴스 시작" 버튼을 클릭합니다.

## 실습 4: VPC 엔드포인트 구성

### 4.1 S3 게이트웨이 엔드포인트 구성
프라이빗 서브넷의 인스턴스가 인터넷을 통하지 않고 Amazon S3에 액세스할 수 있도록 S3 게이트웨이 엔드포인트를 구성합니다.

1. VPC 대시보드에서 "엔드포인트"를 선택합니다.
2. "엔드포인트 생성" 버튼을 클릭합니다.
3. 다음 설정을 구성합니다:
   - 서비스 카테고리: AWS 서비스
   - 서비스 이름: com.amazonaws.ap-northeast-2.s3 (게이트웨이 유형)
   - VPC: 생성한 VPC 선택
   - 라우팅 테이블: 프라이빗 서브넷과 연결된 라우팅 테이블 선택
   - 정책: 전체 액세스
4. "엔드포인트 생성" 버튼을 클릭합니다.

### 4.2 DynamoDB 게이트웨이 엔드포인트 구성
프라이빗 서브넷의 인스턴스가 인터넷을 통하지 않고 Amazon DynamoDB에 액세스할 수 있도록 DynamoDB 게이트웨이 엔드포인트를 구성합니다.

1. "엔드포인트 생성" 버튼을 클릭합니다.
2. 다음 설정을 구성합니다:
   - 서비스 카테고리: AWS 서비스
   - 서비스 이름: com.amazonaws.ap-northeast-2.dynamodb (게이트웨이 유형)
   - VPC: 생성한 VPC 선택
   - 라우팅 테이블: 프라이빗 서브넷과 연결된 라우팅 테이블 선택
   - 정책: 전체 액세스
3. "엔드포인트 생성" 버튼을 클릭합니다.

### 4.3 SSM 인터페이스 엔드포인트 구성
프라이빗 서브넷의 인스턴스가 인터넷을 통하지 않고 AWS Systems Manager에 액세스할 수 있도록 SSM 관련 인터페이스 엔드포인트를 구성합니다.

1. "엔드포인트 생성" 버튼을 클릭합니다.
2. 다음 설정을 구성합니다:
   - 서비스 카테고리: AWS 서비스
   - 서비스 이름: com.amazonaws.ap-northeast-2.ssm (인터페이스 유형)
   - VPC: 생성한 VPC 선택
   - 서브넷: 프라이빗 애플리케이션 서브넷 선택
   - 보안 그룹: 기본 보안 그룹 선택
   - 프라이빗 DNS 이름 활성화: 체크
3. "엔드포인트 생성" 버튼을 클릭합니다.

4. 추가로 다음 SSM 관련 엔드포인트도 동일한 방식으로 생성합니다:
   - com.amazonaws.ap-northeast-2.ssmmessages
   - com.amazonaws.ap-northeast-2.ec2messages

## 실습 5: 연결 테스트 및 검증

### 5.1 웹 서버 접근 테스트
1. EC2 대시보드에서 웹 서버 인스턴스의 퍼블릭 IP 주소를 확인합니다.
2. 웹 브라우저에서 해당 IP 주소로 접속하여 웹 페이지가 표시되는지 확인합니다.
3. 성공적으로 접속되면 "Hello from Web Server 1" 메시지가 표시됩니다.

### 5.2 웹 서버에서 애플리케이션 서버 접근 테스트
1. EC2 대시보드에서 웹 서버 인스턴스를 선택합니다.
2. "연결" 버튼을 클릭하고 "EC2 인스턴스 연결"을 선택합니다.
3. 브라우저 기반 SSH 연결이 열리면 다음 명령을 실행하여 애플리케이션 서버에 접근합니다:
   ```bash
   # 애플리케이션 서버의 프라이빗 IP 주소 확인
   APPSERVER_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=App-Server-1" --query "Reservations[0].Instances[0].PrivateIpAddress" --output text)
   
   # 애플리케이션 서버의 8080 포트에 접근 테스트
   curl -v $APPSERVER_IP:8080
   ```
4. 연결이 성공하면 애플리케이션 서버와의 통신이 가능함을 확인할 수 있습니다.

### 5.3 애플리케이션 서버에서 데이터베이스 서버 접근 테스트
1. 웹 서버에서 애플리케이션 서버로 SSH 접속합니다:
   ```bash
   # 프라이빗 키를 웹 서버에 복사 (실제 환경에서는 보안상 권장되지 않음)
   # 이 단계는 실습 목적으로만 수행합니다
   
   # 애플리케이션 서버에 SSH 접속
   ssh -i key.pem ec2-user@$APPSERVER_IP
   ```
2. 애플리케이션 서버에서 데이터베이스 서버에 접근합니다:
   ```bash
   # 데이터베이스 서버의 프라이빗 IP 주소 확인
   DBSERVER_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=DB-Server-1" --query "Reservations[0].Instances[0].PrivateIpAddress" --output text)
   
   # MySQL 클라이언트 설치
   sudo yum install -y mysql
   
   # 데이터베이스 서버에 접속
   mysql -h $DBSERVER_IP -u testuser -p'password' -e "SHOW DATABASES;"
   ```
3. 연결이 성공하면 데이터베이스 목록이 표시됩니다.

### 5.4 VPC 엔드포인트 테스트
프라이빗 서브넷의 인스턴스에서 VPC 엔드포인트를 통해 AWS 서비스에 액세스할 수 있는지 테스트합니다.

1. 애플리케이션 서버에서 다음 명령을 실행하여 S3 액세스를 테스트합니다:
   ```bash
   # S3 버킷 목록 조회
   aws s3 ls
   
   # 테스트 파일 생성
   echo "VPC Endpoint Test" > test.txt
   
   # 테스트 파일을 S3에 업로드 (버킷이 있다고 가정)
   aws s3 cp test.txt s3://your-bucket-name/
   ```

2. DynamoDB 액세스를 테스트합니다:
   ```bash
   # DynamoDB 테이블 목록 조회
   aws dynamodb list-tables
   ```

3. SSM 엔드포인트 테스트:
   ```bash
   # SSM 파라미터 목록 조회
   aws ssm describe-parameters
   ```## 실습
 6: 고가용성 구성 확장

### 6.1 두 번째 가용 영역에 웹 서버 배포
첫 번째 가용 영역과 동일한 방식으로 두 번째 가용 영역에 웹 서버를 배포합니다.

1. EC2 대시보드에서 "인스턴스 시작" 버튼을 클릭합니다.
2. 다음 설정을 구성합니다:
   - 이름: "Web-Server-2"
   - AMI: Amazon Linux 2 선택
   - 인스턴스 유형: t2.micro 선택
   - 키 페어: 기존 키 페어 선택
   - 네트워크 설정:
     - VPC: 생성한 VPC 선택
     - 서브넷: Public-Subnet-AZ2 선택
     - 퍼블릭 IP 자동 할당: 활성화
     - 보안 그룹: Web-Server-SG 선택
   - 고급 세부 정보:
     - 사용자 데이터:
     ```bash
     #!/bin/bash
     yum update -y
     yum install -y httpd
     systemctl start httpd
     systemctl enable httpd
     echo "<h1>Hello from Web Server 2</h1>" > /var/www/html/index.html
     ```
3. "인스턴스 시작" 버튼을 클릭합니다.

### 6.2 Application Load Balancer 구성
두 가용 영역의 웹 서버 앞에 Application Load Balancer를 배치하여 고가용성을 확보합니다.

1. EC2 대시보드에서 "로드 밸런서" 섹션으로 이동합니다.
2. "로드 밸런서 생성" 버튼을 클릭합니다.
3. "Application Load Balancer" 유형을 선택하고 "생성" 버튼을 클릭합니다.
4. 다음 설정을 구성합니다:
   - 이름: "Web-ALB"
   - 체계: 인터넷 연결
   - IP 주소 유형: IPv4
   - 네트워크 매핑:
     - VPC: 생성한 VPC 선택
     - 매핑: 두 가용 영역 모두 선택
     - 서브넷: 각 가용 영역의 퍼블릭 서브넷 선택
   - 보안 그룹:
     - 새 보안 그룹 생성
     - 이름: "ALB-SG"
     - 규칙: HTTP(80) 및 HTTPS(443)에 대해 0.0.0.0/0 허용
   - 리스너 및 라우팅:
     - 프로토콜: HTTP
     - 포트: 80
     - 대상 그룹: 새 대상 그룹 생성
       - 이름: "Web-TG"
       - 프로토콜: HTTP
       - 포트: 80
       - 대상 유형: 인스턴스
       - 상태 확인 경로: "/"
   - 대상 등록:
     - Web-Server-1 및 Web-Server-2 인스턴스 선택
5. "로드 밸런서 생성" 버튼을 클릭합니다.

### 6.3 Auto Scaling 그룹 구성
웹 서버의 자동 확장 및 복구를 위해 Auto Scaling 그룹을 구성합니다.

1. EC2 대시보드에서 "Auto Scaling 그룹" 섹션으로 이동합니다.
2. "Auto Scaling 그룹 생성" 버튼을 클릭합니다.
3. 다음 설정을 구성합니다:
   - 시작 템플릿 생성:
     - 이름: "Web-Server-Template"
     - AMI: Amazon Linux 2 선택
     - 인스턴스 유형: t2.micro 선택
     - 키 페어: 기존 키 페어 선택
     - 네트워크 설정:
       - 보안 그룹: Web-Server-SG 선택
     - 고급 세부 정보:
       - 사용자 데이터:
       ```bash
       #!/bin/bash
       yum update -y
       yum install -y httpd
       systemctl start httpd
       systemctl enable httpd
       echo "<h1>Hello from Auto Scaling Web Server</h1>" > /var/www/html/index.html
       ```
   - Auto Scaling 그룹 세부 정보:
     - 이름: "Web-ASG"
     - 시작 템플릿: Web-Server-Template 선택
     - VPC: 생성한 VPC 선택
     - 가용 영역 및 서브넷: 두 가용 영역의 퍼블릭 서브넷 선택
   - 로드 밸런싱:
     - 기존 로드 밸런서에 연결: 선택
     - 대상 그룹 선택: Web-TG 선택
   - 그룹 크기:
     - 원하는 용량: 2
     - 최소 용량: 2
     - 최대 용량: 4
   - 조정 정책:
     - 대상 추적 조정 정책 추가
     - 지표 유형: 평균 CPU 사용률
     - 대상 값: 70%
4. "Auto Scaling 그룹 생성" 버튼을 클릭합니다.

## 실습 7: 모니터링 및 로깅 구성

### 7.1 VPC 흐름 로그 활성화
VPC 네트워크 트래픽을 모니터링하기 위해 VPC 흐름 로그를 활성화합니다.

1. VPC 대시보드에서 생성한 VPC를 선택합니다.
2. "작업" 메뉴에서 "흐름 로그 생성"을 선택합니다.
3. 다음 설정을 구성합니다:
   - 이름: "VPC-Flow-Log"
   - 필터: "모든 트래픽"
   - 최대 집계 간격: "1분"
   - 대상: "CloudWatch Logs"
   - 로그 그룹: 새 로그 그룹 생성 (예: "/aws/vpc/flowlogs")
   - IAM 역할: 새 역할 생성
4. "흐름 로그 생성" 버튼을 클릭합니다.

### 7.2 CloudWatch 경보 설정
주요 지표에 대한 CloudWatch 경보를 설정합니다.

1. CloudWatch 대시보드로 이동합니다.
2. "경보" 섹션에서 "경보 생성" 버튼을 클릭합니다.
3. 다음 설정을 구성합니다:
   - 지표 선택: EC2 > 인스턴스별 지표 > CPUUtilization
   - 인스턴스: Web-Server-1 선택
   - 통계: 평균
   - 기간: 5분
   - 임계값 유형: 정적
   - 조건: 보다 큼
   - 임계값: 80%
   - 알림:
     - 경보 상태 트리거: 경보 상태
     - SNS 주제 선택 또는 생성
   - 이름: "High-CPU-Alarm-Web-Server-1"
4. "경보 생성" 버튼을 클릭합니다.

### 7.3 대시보드 생성
주요 지표를 모니터링하기 위한 CloudWatch 대시보드를 생성합니다.

1. CloudWatch 대시보드에서 "대시보드 생성" 버튼을 클릭합니다.
2. 대시보드 이름을 "VPC-Lab-Dashboard"로 입력합니다.
3. 다음 위젯을 추가합니다:
   - EC2 인스턴스 CPU 사용률
   - EC2 인스턴스 네트워크 입/출력
   - ALB 요청 수
   - ALB 대상 응답 시간
   - VPC 흐름 로그 지표
4. "대시보드 저장" 버튼을 클릭합니다.

## 실습 8: 리소스 정리

실습이 완료되면 생성한 리소스를 삭제하여 불필요한 비용이 발생하지 않도록 합니다.

1. Auto Scaling 그룹 삭제:
   - Auto Scaling 그룹 대시보드에서 "Web-ASG"를 선택합니다.
   - "작업" > "삭제"를 클릭합니다.
   - 확인 대화 상자에서 "삭제"를 클릭합니다.

2. 로드 밸런서 삭제:
   - 로드 밸런서 대시보드에서 "Web-ALB"를 선택합니다.
   - "작업" > "삭제"를 클릭합니다.
   - 확인 대화 상자에서 "예, 삭제합니다"를 클릭합니다.

3. EC2 인스턴스 종료:
   - EC2 대시보드에서 모든 인스턴스를 선택합니다.
   - "인스턴스 상태" > "인스턴스 종료"를 클릭합니다.
   - 확인 대화 상자에서 "종료"를 클릭합니다.

4. NAT 게이트웨이 삭제:
   - VPC 대시보드에서 "NAT 게이트웨이"를 선택합니다.
   - 각 NAT 게이트웨이를 선택하고 "작업" > "NAT 게이트웨이 삭제"를 클릭합니다.
   - 확인 대화 상자에서 "삭제"를 클릭합니다.

5. VPC 엔드포인트 삭제:
   - VPC 대시보드에서 "엔드포인트"를 선택합니다.
   - 각 엔드포인트를 선택하고 "작업" > "엔드포인트 삭제"를 클릭합니다.
   - 확인 대화 상자에서 "예, 삭제"를 클릭합니다.

6. VPC 삭제:
   - VPC 대시보드에서 생성한 VPC를 선택합니다.
   - "작업" > "VPC 삭제"를 클릭합니다.
   - 확인 대화 상자에서 "삭제"를 클릭합니다.

## 결론

이 실습을 통해 AWS VPC를 설계하고 구현하는 방법을 배웠습니다. 다중 가용 영역 아키텍처, 퍼블릭 및 프라이빗 서브넷 구성, 보안 그룹 및 네트워크 ACL 설정, VPC 엔드포인트 구성, 그리고 고가용성을 위한 로드 밸런서 및 Auto Scaling 그룹 설정 등 다양한 VPC 기능을 실습했습니다.

이러한 지식과 경험은 실제 프로덕션 환경에서 안전하고 확장 가능한 네트워크 아키텍처를 설계하는 데 큰 도움이 될 것입니다.

## 추가 학습 자료
- [Amazon VPC 사용 설명서](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [AWS 아키텍처 센터 - VPC 설계 패턴](https://aws.amazon.com/architecture/)
- [AWS Well-Architected Framework - 보안 원칙](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [AWS 네트워킹 및 콘텐츠 전송 블로그](https://aws.amazon.com/blogs/networking-and-content-delivery/)