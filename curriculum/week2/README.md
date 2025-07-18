# 2주차: 복원력을 갖춘 아키텍처 설계

## 학습 목표
- 고가용성 및 내결함성 설계 원칙 이해
- 확장 가능한 아키텍처 설계 방법 습득
- AWS의 다양한 컴퓨팅 서비스 활용 방법 학습
- 서버리스 아키텍처 및 컨테이너 기술 이해
- 재해 복구 전략 수립 방법 습득

## 주간 일정

### 월요일: 고가용성 설계 원칙 및 EC2 기초
- [고가용성 설계 원칙](./day1/high_availability_principles.md)
- [EC2 기초 및 인스턴스 유형](./day1/ec2_basics.md)
- [실습: EC2 인스턴스 배포 및 관리](./day1/lab_ec2_deployment.md)

### 화요일: Auto Scaling 및 로드 밸런싱
- [Auto Scaling 개념 및 구성](./day2/auto_scaling.md)
- [Elastic Load Balancing 서비스](./day2/elastic_load_balancing.md)
- [실습: Auto Scaling 그룹 및 로드 밸런서 구성](./day2/lab_auto_scaling_elb.md)

### 수요일: 서버리스 아키텍처
- [서버리스 컴퓨팅 개념](./day3/serverless_computing.md)
- [AWS Lambda 및 API Gateway](./day3/lambda_api_gateway.md)
- [실습: 서버리스 API 구축](./day3/lab_serverless_api.md)

### 목요일: 컨테이너 서비스
- [컨테이너 기술 개요](./day4/container_overview.md)
- [Amazon ECS 및 EKS](./day4/ecs_eks.md)
- [실습: 컨테이너 배포 및 관리](./day4/lab_container_deployment.md)

### 금요일: 재해 복구 전략 및 백업 솔루션
- [재해 복구 전략 및 설계](./day5/disaster_recovery.md)
- [AWS 백업 서비스](./day5/aws_backup_services.md)
- [실습: 재해 복구 계획 수립 및 백업 구성](./day5/lab_dr_backup.md)
- [주간 복습 및 퀴즈](./day5/weekly_review.md)

## 주간 과제
- [고가용성 웹 애플리케이션 배포](./assignments/ha_web_app_deployment.md)

## 주간 챌린지
- [멀티 리전 재해 복구 솔루션](./challenges/multi_region_dr.md)

## 참고 자료
- [AWS 공식 문서: 고가용성 아키텍처](https://docs.aws.amazon.com/whitepapers/latest/real-time-communication-on-aws/high-availability-architecture.html)
- [AWS 공식 문서: EC2 사용 설명서](https://docs.aws.amazon.com/ec2/index.html)
- [AWS 공식 문서: Auto Scaling 사용 설명서](https://docs.aws.amazon.com/autoscaling/index.html)
- [AWS 공식 문서: Elastic Load Balancing 사용 설명서](https://docs.aws.amazon.com/elasticloadbalancing/index.html)
- [AWS 공식 문서: Lambda 개발자 안내서](https://docs.aws.amazon.com/lambda/index.html)
- [AWS 공식 문서: ECS 개발자 안내서](https://docs.aws.amazon.com/ecs/index.html)
- [AWS 공식 문서: 재해 복구 백서](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html)