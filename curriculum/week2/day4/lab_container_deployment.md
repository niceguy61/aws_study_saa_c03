# 실습: 컨테이너 배포 및 관리

## 개요
이 실습에서는 AWS에서 컨테이너를 배포하고 관리하는 방법을 학습합니다. Docker를 사용하여 간단한 애플리케이션을 컨테이너화하고, Amazon ECR에 이미지를 푸시한 다음, Amazon ECS를 사용하여 컨테이너를 배포하고 관리하는 과정을 경험합니다. 또한 AWS Fargate를 사용하여 서버리스 컨테이너 배포를 구성하고, 고가용성을 위한 다중 가용 영역 배포 및 자동 확장을 설정합니다.

## 학습 목표
- Docker를 사용하여 애플리케이션을 컨테이너화하는 방법 습득
- Amazon ECR에 컨테이너 이미지를 저장하고 관리하는 방법 이해
- Amazon ECS를 사용하여 컨테이너를 배포하고 관리하는 방법 학습
- AWS Fargate를 활용한 서버리스 컨테이너 배포 구성 방법 이해
- 고가용성 및 확장성을 위한 ECS 서비스 구성 방법 습득

## 사전 준비 사항
- AWS 계정
- AWS CLI 설치 및 구성
- Docker 설치 (로컬 개발 환경)
- 기본 Node.js 지식 (예제 애플리케이션용)
- 웹 브라우저

## 실습 시간
약 2시간

## 실습 단계

### 1. 간단한 웹 애플리케이션 생성

#### 1.1 프로젝트 디렉토리 설정
1. 로컬 컴퓨터에서 새 디렉토리를 생성합니다:
   ```bash
   mkdir container-lab
   cd container-lab
   ```

2. 간단한 Node.js 애플리케이션을 위한 파일을 생성합니다:

   **package.json**:
   ```json
   {
     "name": "container-demo",
     "version": "1.0.0",
     "description": "Simple container demo app",
     "main": "app.js",
     "scripts": {
       "start": "node app.js"
     },
     "dependencies": {
       "express": "^4.17.1"
     }
   }
   ```

   **app.js**:
   ```javascript
   const express = require('express');
   const app = express();
   const port = process.env.PORT || 3000;
   
   app.get('/', (req, res) => {
     const hostname = require('os').hostname();
     res.send(`
       <html>
         <head>
           <title>Container Demo</title>
           <style>
             body {
               font-family: Arial, sans-serif;
               max-width: 800px;
               margin: 0 auto;
               padding: 20px;
               text-align: center;
             }
             .container {
               background-color: #f0f0f0;
               border-radius: 10px;
               padding: 20px;
               box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
             }
             h1 {
               color: #333;
             }
             .info {
               background-color: #e0e0e0;
               border-radius: 5px;
               padding: 10px;
               margin-top: 20px;
             }
           </style>
         </head>
         <body>
           <div class="container">
             <h1>Hello from Container!</h1>
             <p>This is a simple Node.js application running in a container.</p>
             <div class="info">
               <p><strong>Container ID:</strong> ${hostname}</p>
               <p><strong>Time:</strong> ${new Date().toLocaleString()}</p>
             </div>
           </div>
         </body>
       </html>
     `);
   });
   
   app.listen(port, () => {
     console.log(`App listening at http://localhost:${port}`);
   });
   ```

#### 1.2 Dockerfile 생성
애플리케이션을 컨테이너화하기 위한 Dockerfile을 생성합니다:

```dockerfile
FROM node:14-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### 2. 로컬에서 컨테이너 빌드 및 테스트

#### 2.1 Docker 이미지 빌드
1. 프로젝트 디렉토리에서 다음 명령을 실행하여 Docker 이미지를 빌드합니다:
   ```bash
   docker build -t container-demo:latest .
   ```

2. 빌드된 이미지를 확인합니다:
   ```bash
   docker images
   ```

#### 2.2 로컬에서 컨테이너 실행
1. 빌드된 이미지를 사용하여 컨테이너를 실행합니다:
   ```bash
   docker run -p 3000:3000 container-demo:latest
   ```

2. 웹 브라우저를 열고 `http://localhost:3000`에 접속하여 애플리케이션이 제대로 작동하는지 확인합니다.

3. 컨테이너를 중지하려면 터미널에서 `Ctrl+C`를 누릅니다.

### 3. Amazon ECR 리포지토리 생성 및 이미지 푸시

#### 3.1 ECR 리포지토리 생성
1. AWS Management Console에 로그인합니다.
2. 상단 검색창에 "ECR"을 입력하거나 서비스 메뉴에서 "Elastic Container Registry"를 선택합니다.
3. "리포지토리 생성" 버튼을 클릭합니다.
4. 다음 정보를 입력합니다:
   - 리포지토리 이름: `container-demo`
   - 태그 불변성: 비활성화
   - 스캔 설정: 기본값 사용
5. "리포지토리 생성" 버튼을 클릭합니다.

#### 3.2 AWS CLI를 사용하여 ECR에 로그인
1. 터미널에서 다음 명령을 실행하여 ECR에 로그인합니다:
   ```bash
   aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-account-id>.dkr.ecr.<your-region>.amazonaws.com
   ```
   
   **참고**: `<your-region>`을 사용 중인 AWS 리전(예: us-east-1)으로, `<your-account-id>`를 AWS 계정 ID로 바꿉니다.

#### 3.3 이미지 태그 지정 및 푸시
1. 로컬 이미지에 ECR 리포지토리 URL로 태그를 지정합니다:
   ```bash
   docker tag container-demo:latest <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/container-demo:latest
   ```

2. 태그가 지정된 이미지를 ECR 리포지토리에 푸시합니다:
   ```bash
   docker push <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/container-demo:latest
   ```

3. ECR 콘솔에서 이미지가 성공적으로 푸시되었는지 확인합니다.

### 4. Amazon ECS 클러스터 생성

#### 4.1 ECS 클러스터 생성
1. AWS Management Console에서 "ECS"를 검색하거나 서비스 메뉴에서 "Elastic Container Service"를 선택합니다.
2. "클러스터" 탭을 클릭하고 "클러스터 생성" 버튼을 클릭합니다.
3. "Fargate"를 선택합니다.
4. 다음 정보를 입력합니다:
   - 클러스터 이름: `container-demo-cluster`
   - 네트워킹: 기본 VPC 및 서브넷 사용
   - CloudWatch Container Insights: 활성화
5. "생성" 버튼을 클릭합니다.

### 5. 작업 정의 생성

#### 5.1 Fargate 작업 정의 생성
1. ECS 콘솔에서 "작업 정의" 탭을 클릭하고 "새 작업 정의 생성" 버튼을 클릭합니다.
2. "Fargate"를 선택하고 "다음 단계" 버튼을 클릭합니다.
3. 다음 정보를 입력합니다:
   - 작업 정의 이름: `container-demo-task`
   - 작업 역할: `ecsTaskExecutionRole` (없는 경우 새로 생성)
   - 작업 실행 역할: `ecsTaskExecutionRole`
   - 작업 메모리: 0.5GB
   - 작업 CPU: 0.25 vCPU
4. "컨테이너 추가" 버튼을 클릭하고 다음 정보를 입력합니다:
   - 컨테이너 이름: `container-demo`
   - 이미지: `<your-account-id>.dkr.ecr.<your-region>.amazonaws.com/container-demo:latest`
   - 메모리 제한: 소프트 제한 - 256
   - 포트 매핑: 3000
5. "생성" 버튼을 클릭합니다.

### 6. ECS 서비스 생성 및 배포

#### 6.1 ECS 서비스 생성
1. ECS 콘솔에서 `container-demo-cluster`를 선택합니다.
2. "서비스" 탭을 클릭하고 "생성" 버튼을 클릭합니다.
3. 다음 정보를 입력합니다:
   - 시작 유형: Fargate
   - 작업 정의: `container-demo-task`
   - 서비스 이름: `container-demo-service`
   - 작업 수: 2
   - 최소 정상 백분율: 100
   - 최대 백분율: 200
4. "다음 단계" 버튼을 클릭합니다.
5. 네트워킹 구성:
   - VPC: 기본 VPC 선택
   - 서브넷: 여러 가용 영역에 걸쳐 최소 2개의 서브넷 선택
   - 보안 그룹: 새 보안 그룹 생성
     - 이름: `container-demo-sg`
     - 인바운드 규칙: TCP 3000 포트 허용 (0.0.0.0/0)
   - 퍼블릭 IP 자동 할당: 활성화
6. "다음 단계" 버튼을 클릭합니다.
7. 로드 밸런서 구성:
   - 로드 밸런서 유형: Application Load Balancer
   - 로드 밸런서 이름: 새 로드 밸런서 생성 - `container-demo-alb`
   - 컨테이너: `container-demo:3000:3000`
   - 대상 그룹 이름: 새 대상 그룹 생성 - `container-demo-tg`
8. "다음 단계" 버튼을 클릭합니다.
9. Auto Scaling 구성:
   - 서비스 Auto Scaling: 활성화
   - 최소 작업 수: 2
   - 원하는 작업 수: 2
   - 최대 작업 수: 6
   - 조정 정책: 대상 추적
   - 정책 이름: `cpu-tracking`
   - 지표: ECSServiceAverageCPUUtilization
   - 대상 값: 70
10. "다음 단계" 버튼을 클릭하고 설정을 검토한 후 "서비스 생성" 버튼을 클릭합니다.

#### 6.2 서비스 배포 확인
1. ECS 콘솔에서 `container-demo-cluster` > `container-demo-service`를 선택합니다.
2. "작업" 탭을 클릭하여 실행 중인 작업을 확인합니다.
3. 모든 작업이 "RUNNING" 상태가 될 때까지 기다립니다.
4. "세부 정보" 탭에서 로드 밸런서 섹션을 찾아 로드 밸런서 DNS 이름을 확인합니다.
5. 웹 브라우저에서 로드 밸런서 DNS 이름을 입력하여 애플리케이션에 접속합니다.
6. 페이지를 여러 번 새로고침하여 요청이 여러 컨테이너 인스턴스에 분산되는지 확인합니다(컨테이너 ID가 변경됨).

### 7. 서비스 확장 및 업데이트 테스트

#### 7.1 서비스 수동 확장
1. ECS 콘솔에서 `container-demo-service`를 선택합니다.
2. "업데이트" 버튼을 클릭합니다.
3. "원하는 작업 수"를 4로 변경합니다.
4. "서비스 업데이트" 버튼을 클릭합니다.
5. "작업" 탭에서 새 작업이 시작되는 것을 확인합니다.

#### 7.2 애플리케이션 업데이트 및 배포
1. 로컬 프로젝트에서 `app.js` 파일을 수정하여 메시지를 변경합니다:
   ```javascript
   res.send(`
     <html>
       <head>
         <title>Container Demo - Updated</title>
         <style>
           body {
             font-family: Arial, sans-serif;
             max-width: 800px;
             margin: 0 auto;
             padding: 20px;
             text-align: center;
           }
           .container {
             background-color: #e6f7ff;
             border-radius: 10px;
             padding: 20px;
             box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
           }
           h1 {
             color: #0066cc;
           }
           .info {
             background-color: #cce5ff;
             border-radius: 5px;
             padding: 10px;
             margin-top: 20px;
           }
         </style>
       </head>
       <body>
         <div class="container">
           <h1>Hello from Updated Container!</h1>
           <p>This is the updated version of our application.</p>
           <div class="info">
             <p><strong>Container ID:</strong> ${hostname}</p>
             <p><strong>Time:</strong> ${new Date().toLocaleString()}</p>
             <p><strong>Version:</strong> 2.0</p>
           </div>
         </div>
       </body>
     </html>
   `);
   ```

2. 새 Docker 이미지를 빌드하고 태그를 지정합니다:
   ```bash
   docker build -t container-demo:v2 .
   docker tag container-demo:v2 <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/container-demo:v2
   ```

3. 새 이미지를 ECR에 푸시합니다:
   ```bash
   docker push <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/container-demo:v2
   ```

4. 새 작업 정의 개정을 생성합니다:
   - ECS 콘솔에서 "작업 정의" > `container-demo-task`를 선택합니다.
   - "새 개정 생성" 버튼을 클릭합니다.
   - 컨테이너 정의를 클릭하고 이미지 URL을 `<your-account-id>.dkr.ecr.<your-region>.amazonaws.com/container-demo:v2`로 업데이트합니다.
   - "생성" 버튼을 클릭합니다.

5. 서비스를 업데이트하여 새 작업 정의를 사용합니다:
   - ECS 콘솔에서 `container-demo-service`를 선택합니다.
   - "업데이트" 버튼을 클릭합니다.
   - "작업 정의"에서 "새 개정"을 선택하고 최신 개정을 선택합니다.
   - "서비스 업데이트" 버튼을 클릭합니다.

6. 배포 진행 상황을 모니터링하고 웹 브라우저에서 애플리케이션을 새로고침하여 업데이트된 버전이 표시되는지 확인합니다.

### 8. 모니터링 및 로깅 구성

#### 8.1 CloudWatch 로그 확인
1. ECS 콘솔에서 실행 중인 작업 중 하나를 선택합니다.
2. "로그" 탭을 클릭하여 컨테이너 로그를 확인합니다.
3. 또는 CloudWatch 콘솔로 이동하여 로그 그룹 `/ecs/container-demo-task`를 찾아 로그 스트림을 확인합니다.

#### 8.2 CloudWatch 지표 확인
1. CloudWatch 콘솔로 이동합니다.
2. "지표" > "모든 지표"를 클릭합니다.
3. "ECS" 네임스페이스를 선택합니다.
4. "ClusterName, ServiceName"을 클릭하여 서비스 수준 지표를 확인합니다.
5. `container-demo-cluster`와 `container-demo-service`에 해당하는 지표를 선택하여 그래프로 확인합니다.

#### 8.3 경보 생성
1. CloudWatch 콘솔에서 "경보" > "경보 생성"을 클릭합니다.
2. "지표 선택"을 클릭합니다.
3. "ECS" > "ClusterName, ServiceName"을 선택합니다.
4. `container-demo-cluster`와 `container-demo-service`에 해당하는 "CPUUtilization" 지표를 선택하고 "지표 선택" 버튼을 클릭합니다.
5. 다음 정보를 입력합니다:
   - 통계: Average
   - 기간: 1분
   - 임계값 유형: 정적
   - 조건: 보다 큼
   - 임계값: 80
6. "다음" 버튼을 클릭합니다.
7. 알림 구성:
   - 경보 상태 트리거: 경보 상태
   - SNS 주제 선택: 새 주제 생성
   - 주제 이름: `container-demo-alarm`
   - 이메일 엔드포인트: 자신의 이메일 주소 입력
8. "주제 생성" 버튼을 클릭합니다.
9. "다음" 버튼을 클릭합니다.
10. 경보 이름과 설명을 입력하고 "다음" 버튼을 클릭합니다.
11. 설정을 검토하고 "경보 생성" 버튼을 클릭합니다.
12. 이메일로 SNS 주제 구독 확인 메시지가 오면 확인 링크를 클릭합니다.

### 9. 부하 테스트 및 자동 확장 확인

#### 9.1 부하 테스트 도구 설치
1. 로컬 컴퓨터에 Apache Bench(ab) 또는 다른 부하 테스트 도구를 설치합니다.
   - Ubuntu/Debian: `sudo apt-get install apache2-utils`
   - macOS: `brew install httpd`
   - Windows: Apache Lounge에서 Apache 바이너리 다운로드 또는 WSL 사용

#### 9.2 부하 테스트 실행
1. 터미널에서 다음 명령을 실행하여 부하 테스트를 시작합니다:
   ```bash
   ab -n 10000 -c 100 http://<your-load-balancer-dns>/
   ```
   
   **참고**: `<your-load-balancer-dns>`를 실제 로드 밸런서 DNS 이름으로 바꿉니다.

2. CloudWatch 콘솔에서 CPU 사용률 지표를 모니터링합니다.
3. ECS 콘솔에서 서비스의 "작업" 탭을 모니터링하여 Auto Scaling이 추가 작업을 시작하는지 확인합니다.
4. 부하 테스트가 완료된 후 CPU 사용률이 감소하면 서비스가 다시 축소되는지 확인합니다.

### 10. 정리

#### 10.1 ECS 서비스 삭제
1. ECS 콘솔에서 `container-demo-service`를 선택합니다.
2. "삭제" 버튼을 클릭합니다.
3. 확인 대화 상자에서 "예, 삭제합니다"를 클릭합니다.
4. 서비스가 삭제될 때까지 기다립니다.

#### 10.2 로드 밸런서 및 대상 그룹 삭제
1. EC2 콘솔로 이동합니다.
2. "로드 밸런서"를 클릭하고 `container-demo-alb`를 선택합니다.
3. "작업" > "삭제"를 클릭합니다.
4. 확인 대화 상자에서 "예, 삭제합니다"를 클릭합니다.
5. "대상 그룹"을 클릭하고 `container-demo-tg`를 선택합니다.
6. "작업" > "삭제"를 클릭합니다.
7. 확인 대화 상자에서 "삭제"를 클릭합니다.

#### 10.3 ECS 클러스터 삭제
1. ECS 콘솔에서 `container-demo-cluster`를 선택합니다.
2. "클러스터 삭제" 버튼을 클릭합니다.
3. 확인 대화 상자에서 "삭제"를 클릭합니다.

#### 10.4 ECR 리포지토리 삭제
1. ECR 콘솔에서 `container-demo` 리포지토리를 선택합니다.
2. "삭제" 버튼을 클릭합니다.
3. 확인 대화 상자에 리포지토리 이름을 입력하고 "삭제"를 클릭합니다.

#### 10.5 CloudWatch 경보 삭제
1. CloudWatch 콘솔에서 "경보"를 클릭합니다.
2. 생성한 경보를 선택합니다.
3. "작업" > "삭제"를 클릭합니다.
4. 확인 대화 상자에서 "삭제"를 클릭합니다.

## 도전 과제

### 도전 과제 1: 다중 컨테이너 애플리케이션 배포
1. 프론트엔드와 백엔드로 구성된 간단한 다중 컨테이너 애플리케이션을 생성합니다.
2. 각 컴포넌트에 대한 Dockerfile을 작성합니다.
3. 두 컨테이너를 포함하는 ECS 작업 정의를 생성합니다.
4. 서비스를 배포하고 컨테이너 간 통신을 테스트합니다.

### 도전 과제 2: ECS 작업 예약
1. 정기적으로 실행되는 배치 작업을 위한 ECS 작업 정의를 생성합니다.
2. CloudWatch Events(EventBridge) 규칙을 사용하여 작업을 예약합니다.
3. 작업 실행 결과를 CloudWatch Logs에서 확인합니다.

### 도전 과제 3: 블루/그린 배포 구성
1. AWS CodeDeploy를 사용하여 ECS 서비스에 대한 블루/그린 배포를 구성합니다.
2. 애플리케이션의 새 버전을 생성하고 배포합니다.
3. 트래픽이 새 버전으로 점진적으로 전환되는 것을 확인합니다.

## 결론
이 실습을 통해 AWS에서 컨테이너를 배포하고 관리하는 방법을 학습했습니다. Docker를 사용하여 애플리케이션을 컨테이너화하고, Amazon ECR에 이미지를 저장하고, Amazon ECS와 Fargate를 사용하여 컨테이너를 배포하고 관리하는 과정을 경험했습니다. 또한 로드 밸런싱, 자동 확장, 모니터링 및 로깅과 같은 고가용성 기능을 구성하는 방법도 배웠습니다. 이러한 기술은 확장 가능하고 안정적인 컨테이너 기반 애플리케이션을 AWS에서 운영하는 데 필수적입니다.

## 참고 자료
- [Docker 공식 문서](https://docs.docker.com/)
- [Amazon ECR 사용 설명서](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [Amazon ECS 개발자 안내서](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [AWS Fargate 사용 설명서](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [ECS 서비스 Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)