# Amazon ECS 및 EKS

## 개요
Amazon Elastic Container Service(ECS)와 Amazon Elastic Kubernetes Service(EKS)는 AWS에서 제공하는 컨테이너 오케스트레이션 서비스입니다. 이 강의에서는 ECS와 EKS의 기본 개념, 아키텍처, 주요 기능, 그리고 각 서비스를 언제 어떻게 사용해야 하는지 알아봅니다.

## 학습 목표
- Amazon ECS의 개념, 구성 요소 및 작동 방식 이해
- Amazon EKS의 개념, 구성 요소 및 작동 방식 이해
- ECS와 EKS의 차이점 및 각각의 사용 사례 학습
- AWS Fargate를 활용한 서버리스 컨테이너 실행 방법 이해
- 컨테이너 워크로드를 위한 고가용성 아키텍처 설계 방법 습득

## Amazon ECS (Elastic Container Service)

### ECS의 기본 개념
Amazon ECS는 AWS에서 제공하는 완전 관리형 컨테이너 오케스트레이션 서비스로, Docker 컨테이너를 쉽게 실행, 중지 및 관리할 수 있게 해줍니다.

**비유**: ECS는 마치 공장 관리자와 같습니다. 작업(컨테이너)을 어떤 기계(인스턴스)에서 실행할지 결정하고, 작업이 올바르게 수행되는지 모니터링하며, 필요에 따라 더 많은 작업을 시작하거나 중지합니다.

### ECS의 주요 구성 요소

#### 1. 클러스터
컨테이너를 실행하는 인프라의 논리적 그룹입니다.

**특징**:
- 여러 가용 영역에 걸쳐 있을 수 있음
- EC2 인스턴스 또는 Fargate 작업을 포함
- 리소스 격리 및 관리 단위

**비유**: 클러스터는 마치 공장 건물과 같습니다. 여러 기계와 작업자가 함께 모여 있는 공간입니다.

#### 2. 작업 정의 (Task Definition)
애플리케이션을 구성하는 하나 이상의 컨테이너를 정의하는 JSON 파일입니다.

**주요 설정**:
- 컨테이너 이미지
- CPU 및 메모리 할당
- 포트 매핑
- 환경 변수
- 볼륨
- 네트워킹 모드
- IAM 역할

**비유**: 작업 정의는 마치 제품 설계도와 같습니다. 어떤 부품(컨테이너)이 필요하고, 어떻게 조립해야 하는지 상세히 설명합니다.

**예시 작업 정의**:
```json
{
  "family": "webapp",
  "requiresCompatibilities": ["FARGATE"],
  "networkMode": "awsvpc",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "web",
      "image": "nginx:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "ENV_VAR",
          "value": "value"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/webapp",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "web"
        }
      }
    }
  ]
}
```

#### 3. 작업 (Task)
작업 정의의 인스턴스로, 클러스터 내에서 실행되는 컨테이너 세트입니다.

**특징**:
- 작업 정의에 지정된 컨테이너 실행
- 일회성 또는 지속적인 작업 가능
- 독립적인 실행 단위

**비유**: 작업은 마치 설계도(작업 정의)에 따라 실제로 만들어진 제품과 같습니다.

#### 4. 서비스 (Service)
지정된 수의 작업 인스턴스를 유지하고 관리합니다.

**주요 기능**:
- 작업 수 유지
- 로드 밸런서 통합
- 자동 복구
- 롤링 업데이트
- 서비스 검색

**비유**: 서비스는 마치 생산 라인 관리자와 같습니다. 항상 일정 수의 제품(작업)이 생산되고 있는지 확인하고, 문제가 있는 제품은 새 것으로 교체합니다.

#### 5. 컨테이너 에이전트
각 컨테이너 인스턴스에서 실행되며 ECS 서비스와 통신합니다.

**역할**:
- 컨테이너 시작 및 중지
- 상태 정보 보고
- 볼륨 관리
- 로깅

**비유**: 컨테이너 에이전트는 마치 공장 현장 감독관과 같습니다. 중앙 관리자(ECS 서비스)의 지시를 받아 작업자(컨테이너)를 관리합니다.

### ECS 시작 유형

#### 1. EC2 시작 유형
사용자가 관리하는 EC2 인스턴스에서 컨테이너를 실행합니다.

**특징**:
- 인프라 직접 관리
- 인스턴스 유형 선택 가능
- 비용 최적화 옵션 (예약 인스턴스, 스팟 인스턴스)
- 더 많은 제어 및 사용자 지정

**사용 사례**:
- 대규모 워크로드
- 특정 인스턴스 유형 요구
- 기존 EC2 인프라 활용
- 비용 최적화 필요

**비유**: EC2 시작 유형은 마치 자신의 공장 건물과 기계를 소유하고 관리하는 것과 같습니다. 초기 설정과 유지 관리에 더 많은 노력이 필요하지만, 더 많은 제어와 비용 최적화 옵션을 제공합니다.

#### 2. Fargate 시작 유형
서버를 관리하지 않고 컨테이너를 실행합니다.

**특징**:
- 서버리스 컨테이너 실행
- 인프라 관리 불필요
- 컨테이너 수준 리소스 할당
- 사용한 만큼만 지불

**사용 사례**:
- 인프라 관리 최소화
- 가변적인 워크로드
- 빠른 배포 및 확장
- 소규모 또는 중간 규모 애플리케이션

**비유**: Fargate 시작 유형은 마치 공유 작업 공간을 임대하는 것과 같습니다. 건물이나 기계를 관리할 필요 없이 필요한 공간과 도구만 빌려 사용합니다.

### ECS 네트워킹 모드

#### 1. awsvpc 모드
각 작업에 고유한 탄력적 네트워크 인터페이스(ENI)와 프라이빗 IP 주소를 할당합니다.

**특징**:
- 작업별 보안 그룹
- VPC 기능 완전 활용
- Fargate에 필수
- 향상된 네트워크 격리

**비유**: awsvpc 모드는 마치 각 아파트에 전용 현관과 우편함이 있는 것과 같습니다. 각 작업은 자체 네트워크 "주소"를 가집니다.

#### 2. bridge 모드
Docker의 기본 네트워크 모드로, 호스트의 Docker 브리지 네트워크를 사용합니다.

**특징**:
- 호스트 포트 매핑 필요
- 동일한 컨테이너 인스턴스의 작업 간 통신 가능
- EC2 시작 유형에서만 사용 가능

**비유**: bridge 모드는 마치 여러 사무실이 공유 로비를 통해 외부와 연결되는 것과 같습니다.

#### 3. host 모드
컨테이너가 호스트의 네트워크 인터페이스를 직접 사용합니다.

**특징**:
- 최고의 네트워크 성능
- 포트 충돌 가능성
- 보안 그룹 공유
- EC2 시작 유형에서만 사용 가능

**비유**: host 모드는 마치 집 안에 있는 모든 방이 외부로 직접 연결된 문을 가지고 있는 것과 같습니다.

### ECS 서비스 검색

ECS 서비스 검색은 AWS Cloud Map과 통합되어 서비스 간 통신을 간소화합니다.

**작동 방식**:
1. 네임스페이스 생성 (예: myapp.local)
2. 서비스 생성 시 서비스 검색 구성
3. 각 작업에 대한 DNS 레코드 자동 생성
4. 서비스 간 통신에 DNS 이름 사용

**이점**:
- 동적 환경에서 서비스 위치 자동 업데이트
- 서비스 간 통신 간소화
- 하드코딩된 IP 주소 불필요
- 상태 확인 통합

**비유**: 서비스 검색은 마치 자동 업데이트되는 전화번호부와 같습니다. 서비스가 이동하거나 변경되어도 항상 최신 "연락처 정보"를 제공합니다.

### ECS 자동 확장

ECS는 여러 자동 확장 옵션을 제공하여 트래픽 변화에 대응합니다.

#### 1. 서비스 자동 확장
서비스의 원하는 작업 수를 자동으로 조정합니다.

**유형**:
- **대상 추적 조정**: 특정 지표(예: CPU 사용률)의 목표값 유지
- **단계 조정**: 경보 위반 시 지정된 단계에 따라 조정
- **예약된 조정**: 예측 가능한 패턴에 따라 미리 정의된 시간에 조정

**비유**: 서비스 자동 확장은 마치 고객 수에 따라 계산대를 열거나 닫는 슈퍼마켓과 같습니다.

#### 2. 클러스터 자동 확장
EC2 시작 유형에서 클러스터의 EC2 인스턴스 수를 자동으로 조정합니다.

**작동 방식**:
1. 용량 공급자 생성
2. Auto Scaling 그룹 연결
3. ECS가 작업 배치 요구 사항에 따라 인스턴스 수 조정

**비유**: 클러스터 자동 확장은 마치 작업량에 따라 공장에 더 많은 기계를 추가하거나 제거하는 것과 같습니다.

## Amazon EKS (Elastic Kubernetes Service)

### EKS의 기본 개념
Amazon EKS는 AWS에서 Kubernetes를 실행하기 위한 관리형 서비스로, Kubernetes 컨트롤 플레인을 관리하여 사용자가 워커 노드와 워크로드에만 집중할 수 있게 해줍니다.

**비유**: EKS는 마치 전문 오케스트라 지휘자를 고용하는 것과 같습니다. 지휘자(컨트롤 플레인)는 AWS가 관리하므로, 사용자는 악기 연주자(워커 노드)와 음악(워크로드)에만 집중할 수 있습니다.

### EKS의 주요 구성 요소

#### 1. 컨트롤 플레인
Kubernetes 클러스터를 관리하는 구성 요소 집합입니다.

**구성 요소**:
- API 서버: Kubernetes API 제공
- etcd: 클러스터 데이터 저장소
- 스케줄러: 포드 배치 결정
- 컨트롤러 관리자: 컨트롤러 실행

**특징**:
- AWS에서 관리
- 여러 가용 영역에 걸쳐 고가용성
- 자동 버전 업그레이드
- 보안 패치 자동 적용

**비유**: 컨트롤 플레인은 마치 도시의 중앙 관제 센터와 같습니다. 교통 흐름을 모니터링하고, 신호등을 제어하며, 비상 상황에 대응합니다.

#### 2. 워커 노드
컨테이너화된 애플리케이션을 실행하는 EC2 인스턴스입니다.

**유형**:
- 자체 관리형 노드: 사용자가 직접 생성 및 관리
- 관리형 노드 그룹: EKS가 노드 생성 및 관리 지원
- Fargate 프로필: 서버리스 컨테이너 실행

**비유**: 워커 노드는 마치 공장의 작업 라인과 같습니다. 실제 제품(컨테이너)이 만들어지는 곳입니다.

#### 3. 포드 (Pod)
Kubernetes에서 가장 작은 배포 단위로, 하나 이상의 컨테이너 그룹입니다.

**특징**:
- 동일한 네트워크 네임스페이스 공유
- 동일한 스토리지 볼륨 액세스
- 함께 스케줄링 및 배포
- 동일한 생명주기

**비유**: 포드는 마치 아파트의 한 세대와 같습니다. 여러 방(컨테이너)이 있지만 같은 주소와 공용 공간을 공유합니다.

#### 4. 배포 (Deployment)
포드의 선언적 업데이트를 관리합니다.

**기능**:
- 원하는 포드 수 유지
- 롤링 업데이트
- 롤백
- 스케일링
- 자동 복구

**비유**: 배포는 마치 제품 생산 계획과 같습니다. 얼마나 많은 제품을 생산할지, 어떻게 업데이트할지 정의합니다.

#### 5. 서비스 (Service)
포드 집합에 대한 안정적인 네트워크 엔드포인트를 제공합니다.

**유형**:
- ClusterIP: 클러스터 내부 통신
- NodePort: 노드 포트를 통한 외부 액세스
- LoadBalancer: 외부 로드 밸런서를 통한 액세스
- ExternalName: 외부 서비스에 대한 별칭

**비유**: 서비스는 마치 호텔 프론트 데스크와 같습니다. 손님(요청)을 적절한 객실(포드)로 안내합니다.

### EKS 클러스터 네트워킹

#### 1. VPC CNI 플러그인
Amazon VPC 네트워킹을 Kubernetes 포드 네트워킹과 통합합니다.

**작동 방식**:
- 각 포드에 VPC IP 주소 할당
- 보안 그룹 통합
- 기존 VPC 네트워킹 도구 활용

**이점**:
- 네이티브 VPC 네트워킹
- 향상된 보안
- 성능 최적화
- AWS 서비스와의 통합

#### 2. 클러스터 엔드포인트 액세스
Kubernetes API 서버에 대한 액세스를 제어합니다.

**옵션**:
- 공개 액세스: 인터넷에서 API 서버 액세스 가능
- 프라이빗 액세스: VPC 내에서만 API 서버 액세스 가능
- 둘 다: 공개 및 프라이빗 액세스 모두 활성화

**보안 모범 사례**:
- 프라이빗 액세스만 활성화
- 필요한 경우 VPN 또는 AWS Direct Connect 사용
- CIDR 블록으로 공개 액세스 제한

### EKS 스토리지 옵션

#### 1. 영구 볼륨 (Persistent Volume)
포드의 수명주기와 독립적인 스토리지 리소스입니다.

**지원되는 AWS 스토리지 유형**:
- Amazon EBS
- Amazon EFS
- Amazon FSx for Lustre
- Amazon FSx for NetApp ONTAP

**비유**: 영구 볼륨은 마치 외부 하드 드라이브와 같습니다. 컴퓨터(포드)를 바꾸더라도 데이터는 유지됩니다.

#### 2. 스토리지 클래스 (Storage Class)
스토리지 유형 및 구성을 정의합니다.

**기능**:
- 동적 볼륨 프로비저닝
- 스토리지 유형 추상화
- 다양한 성능 및 비용 옵션

**예시**:
- `gp2`: 범용 EBS 볼륨
- `io1`: 프로비저닝된 IOPS EBS 볼륨
- `efs`: Amazon EFS 파일 시스템

### EKS 보안

#### 1. IAM 통합
AWS IAM과 Kubernetes RBAC를 통합하여 클러스터 액세스를 제어합니다.

**작동 방식**:
1. IAM 엔터티(사용자 또는 역할)에 EKS 클러스터 액세스 권한 부여
2. `aws-auth` ConfigMap을 통해 IAM 엔터티를 Kubernetes 사용자 또는 그룹에 매핑
3. Kubernetes RBAC를 사용하여 세부적인 권한 정의

**이점**:
- 중앙 집중식 ID 관리
- 세분화된 액세스 제어
- AWS 서비스와의 통합
- 감사 및 규정 준수

#### 2. 포드 IAM 역할
개별 Kubernetes 포드에 특정 IAM 권한을 할당합니다.

**작동 방식**:
1. 서비스 계정에 대한 IAM 역할 생성
2. 서비스 계정과 IAM 역할 연결
3. 포드에 서비스 계정 할당

**이점**:
- 최소 권한 원칙 적용
- 포드별 권한 분리
- AWS 서비스에 대한 안전한 액세스

#### 3. 네트워크 정책
포드 간 통신을 제어하는 Kubernetes 리소스입니다.

**기능**:
- 인그레스 및 이그레스 트래픽 제어
- 레이블 기반 선택
- 프로토콜 및 포트 필터링

**구현 옵션**:
- Calico
- Cilium
- AWS Network Firewall 통합

## AWS Fargate

### Fargate의 기본 개념
AWS Fargate는 서버를 관리하지 않고도 컨테이너를 실행할 수 있는 서버리스 컴퓨팅 엔진입니다. ECS 및 EKS와 함께 사용할 수 있습니다.

**비유**: Fargate는 마치 택시 서비스와 같습니다. 차량(서버)을 소유하고 유지 관리할 필요 없이 필요할 때 타고(컨테이너 실행) 목적지에 도착하면 비용을 지불합니다.

### Fargate의 작동 방식

#### ECS와 함께 사용
1. 작업 정의 생성 (Fargate 호환성 지정)
2. 작업 또는 서비스 시작
3. AWS가 컨테이너를 실행할 인프라 관리
4. 사용한 vCPU 및 메모리 리소스에 대해서만 비용 지불

#### EKS와 함께 사용
1. Fargate 프로필 생성
2. 네임스페이스 및 레이블 선택기 지정
3. 일치하는 포드 배포
4. AWS가 포드를 실행할 인프라 관리
5. 사용한 vCPU 및 메모리 리소스에 대해서만 비용 지불

### Fargate의 이점

#### 1. 인프라 관리 불필요
서버 프로비저닝, 패치, 보안 및 유지 관리 작업이 필요 없습니다.

**비유**: 이는 마치 관리형 아파트에 사는 것과 같습니다. 건물 유지 관리, 보안, 수리 등을 걱정할 필요 없이 생활 공간만 사용합니다.

#### 2. 컨테이너 수준 격리
각 Fargate 작업 또는 포드는 전용 커널 런타임 환경에서 실행됩니다.

**이점**:
- 향상된 보안
- 예측 가능한 성능
- 리소스 경합 감소

**비유**: 이는 마치 각 세입자가 자신만의 유틸리티(전기, 수도 등)를 가지는 것과 같습니다. 다른 세입자의 사용량에 영향을 받지 않습니다.

#### 3. 정확한 리소스 할당
작업 또는 포드 수준에서 CPU 및 메모리 리소스를 정확하게 지정할 수 있습니다.

**이점**:
- 비용 최적화
- 성능 예측 가능성
- 세밀한 리소스 제어

**비유**: 이는 마치 필요한 크기의 회의실만 예약하는 것과 같습니다. 작은 회의에는 작은 방을, 큰 회의에는 큰 방을 사용합니다.

#### 4. 사용한 만큼만 지불
실행 중인 작업 또는 포드에 할당된 리소스에 대해서만 비용을 지불합니다.

**비용 구조**:
- vCPU-시간당 요금
- 메모리-시간당 요금
- 최소 1분 청구, 이후 1초 단위

**비유**: 이는 마치 주차장에서 실제 주차한 시간만큼만 요금을 지불하는 것과 같습니다.

### Fargate의 제한 사항

#### 1. 리소스 제한
- 최대 4 vCPU 및 30GB 메모리 (작업/포드당)
- 제한된 임시 스토리지 (기본 20GB)
- GPU 지원 없음

#### 2. 네트워킹 제한
- ECS: awsvpc 네트워크 모드만 지원
- EKS: 특정 CNI 플러그인 버전 필요
- 호스트 네트워크 모드 미지원

#### 3. 볼륨 제한
- 제한된 볼륨 유형 지원
- 호스트 경로 볼륨 미지원
- 직접적인 디바이스 마운트 불가

## ECS와 EKS 비교

### 주요 차이점

#### 1. 관리 복잡성
- **ECS**: AWS 네이티브 서비스로 더 간단한 관리
- **EKS**: Kubernetes의 복잡성과 유연성 제공

**비유**: ECS는 마치 자동차의 자동 변속기와 같고, EKS는 수동 변속기와 같습니다. ECS는 더 쉽게 시작할 수 있지만 EKS는 더 많은 제어와 유연성을 제공합니다.

#### 2. 생태계 및 도구
- **ECS**: AWS 중심 생태계, AWS 서비스와의 긴밀한 통합
- **EKS**: 광범위한 오픈 소스 생태계, 다양한 도구 및 확장 기능

**비유**: ECS는 마치 단일 브랜드의 가전제품 세트와 같고, EKS는 다양한 제조업체의 호환 가능한 제품을 사용할 수 있는 표준화된 시스템과 같습니다.

#### 3. 이식성 및 벤더 종속성
- **ECS**: AWS에 특화된 서비스, 다른 클라우드로 마이그레이션 어려움
- **EKS**: 표준 Kubernetes, 다른 클라우드 또는 온프레미스로 이식 가능

**비유**: ECS는 마치 특정 게임 콘솔용 게임과 같고, EKS는 여러 플랫폼에서 실행되는 크로스 플랫폼 게임과 같습니다.

#### 4. 학습 곡선
- **ECS**: AWS 사용자에게 더 친숙하고 배우기 쉬움
- **EKS**: Kubernetes 지식 필요, 더 가파른 학습 곡선

**비유**: ECS는 마치 기본 기능이 있는 디지털 카메라와 같고, EKS는 다양한 설정과 기능이 있는 전문가용 DSLR 카메라와 같습니다.

#### 5. 비용 구조
- **ECS**: 컨테이너 인프라에 대한 비용만 지불 (EC2 또는 Fargate)
- **EKS**: 컨테이너 인프라 비용 + 클러스터 관리 비용 ($0.10/시간)

**비유**: ECS는 마치 기본 임대료만 지불하는 것과 같고, EKS는 기본 임대료에 추가 관리비를 지불하는 것과 같습니다.

### 선택 기준

#### ECS가 적합한 경우
- AWS 서비스 생태계에 깊이 통합된 경우
- 간단한 컨테이너 오케스트레이션 요구 사항
- AWS에 대한 기존 지식 활용
- 빠른 시작 및 간단한 관리 선호
- 비용 최적화 중요

**사용 사례**:
- 간단한 웹 애플리케이션
- 마이크로서비스 아키텍처
- 배치 처리 작업
- AWS 서비스와 긴밀하게 통합된 워크로드

#### EKS가 적합한 경우
- 기존 Kubernetes 지식 또는 워크로드 보유
- 고급 오케스트레이션 기능 필요
- 멀티 클라우드 또는 하이브리드 전략
- 광범위한 오픈 소스 생태계 활용
- 세밀한 제어 및 사용자 지정 필요

**사용 사례**:
- 복잡한 마이크로서비스 아키텍처
- 멀티 클라우드 배포
- 고급 네트워킹 및 보안 요구 사항
- 기존 Kubernetes 워크로드 마이그레이션

## 컨테이너 워크로드를 위한 고가용성 아키텍처

### 다중 가용 영역 배포

#### ECS에서의 구현
- 여러 가용 영역에 걸쳐 작업 배치
- 서비스 수준에서 작업 분산 구성
- 용량 공급자 전략을 통한 인스턴스 분산

**구성 예시**:
```json
{
  "serviceName": "webapp-service",
  "taskDefinition": "webapp",
  "desiredCount": 6,
  "deploymentConfiguration": {
    "maximumPercent": 200,
    "minimumHealthyPercent": 100
  },
  "placementConstraints": [
    {
      "type": "distinctInstance"
    }
  ],
  "placementStrategy": [
    {
      "type": "spread",
      "field": "attribute:ecs.availability-zone"
    }
  ]
}
```

#### EKS에서의 구현
- 여러 가용 영역에 워커 노드 배포
- 포드 안티어피니티 규칙 사용
- 토폴로지 분산 제약 조건 구성

**구성 예시**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 6
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - webapp
              topologyKey: "topology.kubernetes.io/zone"
```

### 자동 확장 구성

#### ECS 자동 확장
- 대상 추적 조정 정책 구성
- 여러 지표 기반 확장 (CPU, 메모리, 요청 수)
- 최소 및 최대 작업 수 설정

**구성 예시**:
```json
{
  "policyName": "webapp-scaling",
  "policyType": "TargetTrackingScaling",
  "resourceId": "service/my-cluster/webapp-service",
  "scalableDimension": "ecs:service:DesiredCount",
  "serviceNamespace": "ecs",
  "targetTrackingScalingPolicyConfiguration": {
    "targetValue": 70.0,
    "predefinedMetricSpecification": {
      "predefinedMetricType": "ECSServiceAverageCPUUtilization"
    },
    "scaleOutCooldown": 60,
    "scaleInCooldown": 60
  }
}
```

#### EKS 자동 확장
- Horizontal Pod Autoscaler (HPA) 구성
- Cluster Autoscaler 배포
- 사용자 지정 지표 기반 확장 (Prometheus 통합)

**HPA 구성 예시**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### 로드 밸런싱 통합

#### ECS와 로드 밸런서 통합
- Application Load Balancer 또는 Network Load Balancer 연결
- 동적 포트 매핑 활용
- 상태 확인 및 연결 드레이닝 구성

**구성 예시**:
```json
{
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:region:account-id:targetgroup/webapp-tg/1234567890",
      "containerName": "webapp",
      "containerPort": 80
    }
  ]
}
```

#### EKS와 로드 밸런서 통합
- AWS Load Balancer Controller 배포
- Ingress 리소스 또는 Service 유형 LoadBalancer 사용
- 주석을 통한 로드 밸런서 구성

**Ingress 구성 예시**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: webapp-service
                port:
                  number: 80
```

### 데이터 지속성 및 상태 관리

#### ECS에서의 데이터 지속성
- EBS 볼륨 마운트
- EFS 파일 시스템 통합
- S3 및 DynamoDB와 같은 관리형 서비스 활용

**작업 정의 예시**:
```json
{
  "family": "webapp",
  "volumes": [
    {
      "name": "efs-volume",
      "efsVolumeConfiguration": {
        "fileSystemId": "fs-1234",
        "transitEncryption": "ENABLED"
      }
    }
  ],
  "containerDefinitions": [
    {
      "name": "webapp",
      "image": "webapp:latest",
      "mountPoints": [
        {
          "sourceVolume": "efs-volume",
          "containerPath": "/data"
        }
      ]
    }
  ]
}
```

#### EKS에서의 데이터 지속성
- 영구 볼륨 및 영구 볼륨 클레임 사용
- 스토리지 클래스 구성
- StatefulSet 리소스 활용

**PVC 및 StatefulSet 예시**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: webapp
spec:
  serviceName: webapp
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: webapp:latest
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: data-pvc
```

## 결론
Amazon ECS와 EKS는 AWS에서 컨테이너화된 애플리케이션을 실행하기 위한 강력한 오케스트레이션 서비스입니다. ECS는 AWS 네이티브 서비스로 더 간단하고 AWS 서비스와 긴밀하게 통합되어 있으며, EKS는 표준 Kubernetes를 제공하여 더 많은 유연성과 이식성을 제공합니다. AWS Fargate를 사용하면 두 서비스 모두에서 서버리스 컨테이너 실행이 가능하여 인프라 관리 부담을 줄일 수 있습니다. 고가용성 아키텍처를 설계할 때는 다중 가용 영역 배포, 자동 확장, 로드 밸런싱 통합, 데이터 지속성 등의 요소를 고려해야 합니다. 워크로드 요구 사항, 팀의 기술 세트, 기존 인프라 및 장기 전략에 따라 ECS와 EKS 중 적합한 서비스를 선택할 수 있습니다.

## 참고 자료
- [Amazon ECS 개발자 안내서](https://docs.aws.amazon.com/ecs/index.html)
- [Amazon EKS 사용 설명서](https://docs.aws.amazon.com/eks/index.html)
- [AWS Fargate 사용 설명서](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [ECS와 EKS 비교](https://aws.amazon.com/blogs/containers/amazon-ecs-vs-amazon-eks-making-sense-of-aws-container-services/)
- [Kubernetes 공식 문서](https://kubernetes.io/docs/home/)