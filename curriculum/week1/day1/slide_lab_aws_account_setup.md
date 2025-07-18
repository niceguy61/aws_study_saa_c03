# 실습: AWS 계정 생성 및 기본 설정

## 슬라이드 1: 실습 개요
- AWS 계정 생성 및 기본 보안 설정 구성
- AWS Management Console, CLI, SDK 사용법 학습
- 결제 알림 설정 및 비용 관리 도구 활용
- AWS 프리 티어 이해 및 활용

![AWS 계정 설정 단계](./images/aws_account_setup.svg)

## 슬라이드 2: 학습 목표
- AWS 계정 생성 프로세스 이해 및 실행
- 루트 사용자 보안 강화 (MFA 설정)
- 결제 알림 설정 및 비용 관리 도구 활용
- AWS Management Console 탐색 및 기본 사용법 학습
- AWS CLI 및 SDK 설치 및 구성

## 슬라이드 3: 사전 준비 사항
- 유효한 이메일 주소
- 신용카드 또는 체크카드 (확인용)
- 휴대전화 (SMS 인증용)
- 웹 브라우저 (Chrome, Firefox, Safari, Edge 최신 버전)
- 노트북 또는 데스크톱 컴퓨터

## 슬라이드 4: AWS 계정 생성 - 1
- AWS 웹사이트(aws.amazon.com) 접속
- "계정 생성" 버튼 클릭
- 이메일 주소 입력 (루트 사용자 ID로 사용)
- AWS 계정 이름 입력
- 이메일 인증 코드 확인

## 슬라이드 5: AWS 계정 생성 - 2
- 강력한 암호 설정 (8자 이상, 대소문자, 숫자, 특수문자 포함)
- 개인 또는 비즈니스 계정 유형 선택
- 연락처 정보 입력 (이름, 주소, 전화번호)
- AWS 고객 계약 동의

## 슬라이드 6: AWS 계정 생성 - 3
- 결제 정보 입력 (신용카드/체크카드)
- 전화번호 확인 (SMS 또는 음성 통화)
- 기본 지원 플랜(무료) 선택
- 계정 활성화 확인 (최대 24시간 소요 가능)

## 슬라이드 7: 루트 사용자 보안 강화
- AWS Management Console 로그인
- 다중 인증(MFA) 설정
  - 가상 MFA 디바이스 (Google Authenticator, Authy)
  - 하드웨어 TOTP 토큰
  - 보안 키 (YubiKey 등)
- 계정 연락처 정보 확인 및 업데이트

## 슬라이드 8: 결제 알림 설정
- 결제 기본 설정 구성
  - "무료 등급 사용량 알림 수신" 활성화
  - "결제 알림 수신" 활성화
  - "PDF 인보이스 수신" 활성화
- 예산 생성 (예: 월간 $10)
- 예산 알림 설정 (80%, 100% 도달 시)

## 슬라이드 9: 비용 관리 도구
- AWS Budgets: 예산 설정 및 알림
- AWS Cost Explorer: 비용 분석 및 시각화
- AWS Trusted Advisor: 비용 최적화 권장사항
- AWS Pricing Calculator: 예상 비용 계산
- 프리 티어 사용량 모니터링

## 슬라이드 10: AWS Management Console 탐색
- 콘솔 주요 구성 요소
  - 서비스 메뉴
  - 리전 선택기
  - 검색 바
  - 최근 방문한 서비스
  - 리소스 그룹
- 리전 선택 (서울 리전: ap-northeast-2)

## 슬라이드 11: 주요 서비스 콘솔 살펴보기
- EC2 (Elastic Compute Cloud): 가상 서버
- S3 (Simple Storage Service): 객체 스토리지
- IAM (Identity and Access Management): 접근 제어
- VPC (Virtual Private Cloud): 가상 네트워크
- CloudWatch: 모니터링 서비스

## 슬라이드 12: AWS CLI 설치 - Windows
```
# 설치 프로그램 다운로드 및 실행
# https://awscli.amazonaws.com/AWSCLIV2.msi

# 설치 확인
aws --version
```

## 슬라이드 13: AWS CLI 설치 - macOS/Linux
```
# macOS
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# 설치 확인
aws --version
```

## 슬라이드 14: AWS CLI 구성
```
# 기본 구성 설정
aws configure

# 입력 정보
AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name: ap-northeast-2
Default output format: json

# 구성 테스트
aws s3 ls
```

## 슬라이드 15: AWS SDK 살펴보기
- 주요 AWS SDK:
  - AWS SDK for Python (Boto3)
  - AWS SDK for JavaScript
  - AWS SDK for Java
  - AWS SDK for .NET
  - AWS SDK for Go
  - AWS SDK for Ruby

## 슬라이드 16: Python SDK 예제
```python
# Boto3 설치
# pip install boto3

import boto3

# S3 클라이언트 생성
s3 = boto3.client('s3')

# 버킷 목록 가져오기
response = s3.list_buckets()

# 결과 출력
print('S3 버킷 목록:')
for bucket in response['Buckets']:
    print(f'  {bucket["Name"]}')
```

## 슬라이드 17: AWS 프리 티어 이해
- 프리 티어 카테고리:
  - 12개월 무료: EC2, S3, RDS 등
  - 항상 무료: Lambda, DynamoDB 등 (제한된 사용량)
  - 단기 무료 평가판: 특정 서비스 단기 체험
- 프리 티어 사용량 모니터링 방법
- 프리 티어 한도 초과 방지 방법

## 슬라이드 18: 주요 프리 티어 한도
- EC2: 월 750시간의 t2.micro/t3.micro 인스턴스
- S3: 월 5GB의 표준 스토리지
- RDS: 월 750시간의 db.t2.micro/db.t3.micro 인스턴스
- Lambda: 월 1,000,000회 요청, 400,000GB-초 컴퓨팅
- CloudFront: 월 50GB 데이터 전송

## 슬라이드 19: 문제 해결 가이드
- 계정 생성 문제:
  - 신용카드 확인 실패
  - 전화번호 확인 실패
  - 이메일 인증 코드 미수신
- MFA 설정 문제:
  - QR 코드 스캔 실패
  - MFA 코드 불일치
- AWS CLI 문제:
  - 'aws' 명령 인식 실패
  - 인증 오류

## 슬라이드 20: 실습 결과물 및 다음 단계
- 실습 결과물:
  - 활성화된 AWS 계정
  - MFA가 설정된 보안 루트 사용자
  - 구성된 결제 알림 및 예산
  - AWS CLI 설치 및 구성
- 다음 단계:
  - IAM 사용자 및 그룹 관리
  - AWS 서비스 기본 사용법 (EC2, S3 등)