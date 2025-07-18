# 실습: ElastiCache Redis 클러스터 구성 및 캐싱 구현

## 개요
이 실습에서는 Amazon ElastiCache for Redis 클러스터를 구성하고, 캐싱 전략을 구현하여 애플리케이션 성능을 향상시키는 방법을 학습합니다. Redis 클러스터 모드, 복제, 샤딩 등의 고급 기능을 활용하여 고성능 캐싱 계층을 구축합니다.

**소요 시간:** 60분
**난이도:** 중급

## 학습 목표
이 실습을 완료하면 다음을 수행할 수 있게 됩니다:
- ElastiCache for Redis 클러스터 생성 및 구성
- Redis 클러스터 모드와 복제 그룹 설정
- 샤딩을 통한 수평적 확장 구현
- 애플리케이션에서 Redis 캐싱 전략 구현
- Redis 클러스터 성능 모니터링 및 최적화

## 사전 요구사항
이 실습을 시작하기 전에 다음이 필요합니다:
- AWS 계정 액세스
- IAM 사용자 (ElastiCache 관련 권한 필요)
- 기본 VPC 및 서브넷 구성
- AWS CLI 설치 (선택 사항)

## 아키텍처 개요

### 구현할 아키텍처
![ElastiCache 아키텍처](https://d1.awsstatic.com/product-marketing/ElastiCache/ElastiCache_Redis_Cluster.78dc3b2e8af2c0b826d8a22999bb0e8760715391.png)

### 구성 요소
- **ElastiCache for Redis 클러스터**: 인메모리 캐싱 서비스
- **기본 노드**: 읽기 및 쓰기 작업을 처리하는 주 노드
- **복제본 노드**: 읽기 작업을 처리하고 고가용성을 제공하는 보조 노드
- **샤드**: 데이터를 분할하여 저장하는 단위
- **파라미터 그룹**: Redis 구성 파라미터 설정
- **서브넷 그룹**: 클러스터가 배포될 서브넷 정의

### 작동 방식
1. 클라이언트가 Redis 클러스터에 데이터 요청
2. 데이터가 캐시에 있으면 즉시 반환 (캐시 히트)
3. 데이터가 캐시에 없으면 원본 데이터 소스에서 조회 후 캐시에 저장 (캐시 미스)
4. 쓰기 작업은 기본 노드로 전달되고, 복제본 노드에 비동기적으로 복제
5. 읽기 작업은 기본 노드 또는 복제본 노드에서 처리 가능

## 실습 단계

### 1단계: ElastiCache 서브넷 그룹 생성
**목적:** Redis 클러스터가 배포될 서브넷 그룹을 정의합니다.

1. AWS Management Console에 로그인합니다.
2. 상단 검색창에 "ElastiCache"를 입력하고 서비스를 선택합니다.
3. 왼쪽 탐색 창에서 "서브넷 그룹"을 선택합니다.
4. "서브넷 그룹 생성" 버튼을 클릭합니다.
5. 다음 정보를 입력합니다:
   - 이름: `redis-cache-subnet-group`
   - 설명: `Subnet group for Redis cache cluster`
   - VPC ID: 기본 VPC 선택
   - 가용 영역 및 서브넷: 최소 2개의 서로 다른 가용 영역에 있는 서브넷 선택
   
6. "생성" 버튼을 클릭합니다.

**확인 포인트:** 서브넷 그룹 목록에 새로 생성된 `redis-cache-subnet-group`이 표시되는지 확인합니다.

### 2단계: Redis 파라미터 그룹 생성
**목적:** Redis 구성 파라미터를 사용자 지정하기 위한 파라미터 그룹을 생성합니다.

1. 왼쪽 탐색 창에서 "파라미터 그룹"을 선택합니다.
2. "파라미터 그룹 생성" 버튼을 클릭합니다.
3. 다음 정보를 입력합니다:
   - 패밀리: `redis6.x` (또는 최신 버전)
   - 이름: `redis-cache-params`
   - 설명: `Custom parameters for Redis cache`
   
4. "생성" 버튼을 클릭합니다.
5. 생성된 파라미터 그룹을 선택하고 "파라미터 편집" 버튼을 클릭합니다.
6. 다음 파라미터를 수정합니다:
   - `maxmemory-policy`: `allkeys-lru` (가장 오래된 키부터 제거)
   - `timeout`: `300` (연결 유휴 시간 제한, 초 단위)
   - `notify-keyspace-events`: `Ex` (만료 이벤트 알림 활성화)
   
7. "변경 사항 저장" 버튼을 클릭합니다.

**확인 포인트:** 파라미터 그룹 목록에 새로 생성된 `redis-cache-params`가 표시되고, 수정된 파라미터 값이 적용되었는지 확인합니다.

### 3단계: Redis 클러스터 생성 (클러스터 모드 비활성화)
**목적:** 기본 노드와 복제본으로 구성된 Redis 클러스터를 생성합니다.

1. 왼쪽 탐색 창에서 "Redis 클러스터"를 선택합니다.
2. "클러스터 생성" 버튼을 클릭합니다.
3. "클러스터 생성 방법"에서 "클러스터 설계자 사용"을 선택합니다.
4. "클러스터 정보" 섹션에서 다음 정보를 입력합니다:
   - 이름: `redis-basic-cluster`
   - 설명: `Basic Redis cluster with replicas`
   - 엔진 버전: 최신 버전 선택
   - 포트: `6379` (기본값)
   - 파라미터 그룹: `redis-cache-params` (2단계에서 생성한 그룹)
   - 노드 유형: `cache.t3.small` (실습용)
   - 복제본 수: `1` (총 2개 노드: 1개 기본 + 1개 복제본)
   
5. "클러스터 설정" 섹션에서 다음 정보를 입력합니다:
   - 다중 AZ: `활성화` (고가용성을 위해)
   - 자동 장애 조치: `활성화`
   - 클러스터 모드: `비활성화`
   
6. "고급 설정" 섹션에서 다음 정보를 입력합니다:
   - 서브넷 그룹: `redis-cache-subnet-group` (1단계에서 생성한 그룹)
   - 보안 그룹: 기본 보안 그룹 선택
   - 백업 보존 기간: `1일`
   - 백업 기간: "기본값 없음" 선택
   - 유지 관리 기간: "기본값 없음" 선택
   - 암호화: 기본값 유지
   
7. "클러스터 생성" 버튼을 클릭합니다.

**확인 포인트:** Redis 클러스터 목록에 새로 생성된 `redis-basic-cluster`가 표시되고, 상태가 "생성 중"에서 "사용 가능"으로 변경되는지 확인합니다. 이 과정은 약 5-10분 정도 소요될 수 있습니다.

> **참고:** 클러스터 생성 중에는 다음 단계를 진행할 수 있습니다.

### 4단계: Redis 클러스터 생성 (클러스터 모드 활성화)
**목적:** 샤딩을 지원하는 클러스터 모드 Redis 클러스터를 생성합니다.

1. 왼쪽 탐색 창에서 "Redis 클러스터"를 선택합니다.
2. "클러스터 생성" 버튼을 클릭합니다.
3. "클러스터 생성 방법"에서 "클러스터 설계자 사용"을 선택합니다.
4. "클러스터 정보" 섹션에서 다음 정보를 입력합니다:
   - 이름: `redis-cluster-mode`
   - 설명: `Redis cluster with sharding enabled`
   - 엔진 버전: 최신 버전 선택
   - 포트: `6379` (기본값)
   - 파라미터 그룹: `redis-cache-params` (2단계에서 생성한 그룹)
   - 노드 유형: `cache.t3.small` (실습용)
   
5. "클러스터 설정" 섹션에서 다음 정보를 입력합니다:
   - 다중 AZ: `활성화` (고가용성을 위해)
   - 자동 장애 조치: `활성화`
   - 클러스터 모드: `활성화`
   - 샤드 수: `2`
   - 샤드당 복제본: `1`
   
6. "고급 설정" 섹션에서 다음 정보를 입력합니다:
   - 서브넷 그룹: `redis-cache-subnet-group` (1단계에서 생성한 그룹)
   - 보안 그룹: 기본 보안 그룹 선택
   - 백업 보존 기간: `1일`
   - 백업 기간: "기본값 없음" 선택
   - 유지 관리 기간: "기본값 없음" 선택
   - 암호화: 기본값 유지
   
7. "클러스터 생성" 버튼을 클릭합니다.

**확인 포인트:** Redis 클러스터 목록에 새로 생성된 `redis-cluster-mode`가 표시되고, 상태가 "생성 중"에서 "사용 가능"으로 변경되는지 확인합니다. 이 과정은 약 10-15분 정도 소요될 수 있습니다.

### 5단계: Redis 클러스터 연결 정보 확인
**목적:** 생성된 Redis 클러스터의 엔드포인트 정보를 확인합니다.

1. Redis 클러스터 목록에서 `redis-basic-cluster`를 선택합니다.
2. "클러스터 세부 정보" 탭에서 다음 정보를 확인합니다:
   - 기본 엔드포인트: 읽기 및 쓰기 작업을 위한 엔드포인트
   - 리더 엔드포인트: 읽기 작업을 위한 엔드포인트
   - 포트: 연결 포트 (기본값 6379)
   
3. Redis 클러스터 목록에서 `redis-cluster-mode`를 선택합니다.
4. "클러스터 세부 정보" 탭에서 다음 정보를 확인합니다:
   - 구성 엔드포인트: 클러스터 모드 연결을 위한 엔드포인트
   - 샤드 정보: 각 샤드의 기본 노드 및 복제본 노드 정보

**확인 포인트:** 두 클러스터의 엔드포인트 정보를 기록해 둡니다. 이 정보는 다음 단계에서 클러스터에 연결할 때 필요합니다.

### 6단계: EC2 인스턴스 생성 및 Redis 클라이언트 설치
**목적:** Redis 클러스터에 연결하고 테스트할 EC2 인스턴스를 생성합니다.

1. AWS Management Console에서 EC2 서비스로 이동합니다.
2. "인스턴스 시작" 버튼을 클릭합니다.
3. 다음 정보를 입력합니다:
   - 이름: `redis-client`
   - AMI: Amazon Linux 2 선택
   - 인스턴스 유형: `t2.micro` (프리 티어 적격)
   - 키 페어: 기존 키 페어 선택 또는 새 키 페어 생성
   - 네트워크 설정: ElastiCache 클러스터와 동일한 VPC 선택
   - 보안 그룹: 기본 보안 그룹 선택 (SSH 액세스 허용)
   
4. "인스턴스 시작" 버튼을 클릭합니다.
5. 인스턴스가 "실행 중" 상태가 되면, SSH를 사용하여 인스턴스에 연결합니다.
6. 다음 명령을 실행하여 Redis 클라이언트를 설치합니다:
   ```bash
   sudo yum update -y
   sudo yum install gcc make -y
   wget http://download.redis.io/redis-stable.tar.gz
   tar xvzf redis-stable.tar.gz
   cd redis-stable
   make
   ```

**확인 포인트:** Redis 클라이언트가 성공적으로 설치되었는지 확인합니다. `src/redis-cli --version` 명령을 실행하여 버전 정보가 표시되는지 확인합니다.

### 7단계: Redis 클러스터 연결 및 기본 작업 테스트
**목적:** Redis 클라이언트를 사용하여 클러스터에 연결하고 기본 작업을 테스트합니다.

1. EC2 인스턴스에서 다음 명령을 실행하여 `redis-basic-cluster`에 연결합니다 (엔드포인트를 5단계에서 확인한 값으로 대체):
   ```bash
   cd redis-stable
   src/redis-cli -h <primary-endpoint> -p 6379
   ```

2. 연결이 성공하면 다음 Redis 명령을 실행하여 기본 작업을 테스트합니다:
   ```
   SET user:1 "John Doe"
   GET user:1
   SET product:1 "AWS Certified Solutions Architect Study Guide"
   GET product:1
   KEYS *
   TTL user:1
   EXPIRE user:1 60
   TTL user:1
   ```

3. 다음 명령을 실행하여 Redis 정보를 확인합니다:
   ```
   INFO
   INFO replication
   INFO memory
   ```

4. `exit` 명령을 입력하여 Redis CLI를 종료합니다.

5. 다음 명령을 실행하여 `redis-cluster-mode`에 연결합니다 (엔드포인트를 5단계에서 확인한 값으로 대체):
   ```bash
   src/redis-cli -h <configuration-endpoint> -p 6379 -c
   ```
   > **참고:** `-c` 옵션은 클러스터 모드를 활성화합니다.

6. 연결이 성공하면 다음 Redis 명령을 실행하여 클러스터 모드 작업을 테스트합니다:
   ```
   CLUSTER INFO
   CLUSTER NODES
   SET user:2 "Jane Smith"
   GET user:2
   SET product:2 "AWS Certified Developer Study Guide"
   GET product:2
   ```

7. `exit` 명령을 입력하여 Redis CLI를 종료합니다.

**확인 포인트:** 두 Redis 클러스터에 성공적으로 연결되고, 기본 작업이 정상적으로 수행되는지 확인합니다.

### 8단계: 캐싱 성능 테스트
**목적:** Redis 캐싱의 성능 이점을 테스트합니다.

1. EC2 인스턴스에서 다음 내용으로 `redis_benchmark.py` 파일을 생성합니다:
   ```python
   import redis
   import time
   import random
   import string

   # Redis 연결 설정 (엔드포인트를 실제 값으로 대체)
   redis_host = "<primary-endpoint>"
   redis_port = 6379
   redis_client = redis.Redis(host=redis_host, port=redis_port)

   # 임의의 문자열 생성 함수
   def generate_random_string(length=10):
       return ''.join(random.choice(string.ascii_letters) for _ in range(length))

   # 캐시 없이 데이터 조회 시뮬레이션 (데이터베이스 조회 가정)
   def get_data_without_cache(key):
       # 데이터베이스 조회 시뮬레이션 (0.1초 지연)
       time.sleep(0.1)
       return generate_random_string(50)

   # 캐시를 사용한 데이터 조회
   def get_data_with_cache(key):
       # 캐시에서 데이터 조회 시도
       data = redis_client.get(key)
       if data:
           # 캐시 히트
           return data.decode('utf-8')
       else:
           # 캐시 미스, 데이터베이스에서 조회
           data = get_data_without_cache(key)
           # 캐시에 저장 (60초 만료)
           redis_client.setex(key, 60, data)
           return data

   # 성능 테스트
   def run_performance_test(num_requests=100):
       # 테스트 키 생성
       keys = [f"test:key:{i}" for i in range(num_requests)]
       
       # 캐시 없이 테스트
       start_time = time.time()
       for key in keys:
           get_data_without_cache(key)
       no_cache_time = time.time() - start_time
       print(f"캐시 없이 {num_requests}개 요청 처리 시간: {no_cache_time:.2f}초")
       
       # 캐시 사용 테스트 (첫 번째 실행 - 캐시 미스)
       start_time = time.time()
       for key in keys:
           get_data_with_cache(key)
       first_cache_time = time.time() - start_time
       print(f"캐시 첫 사용 {num_requests}개 요청 처리 시간: {first_cache_time:.2f}초")
       
       # 캐시 사용 테스트 (두 번째 실행 - 캐시 히트)
       start_time = time.time()
       for key in keys:
           get_data_with_cache(key)
       second_cache_time = time.time() - start_time
       print(f"캐시 히트 {num_requests}개 요청 처리 시간: {second_cache_time:.2f}초")
       
       # 성능 향상 계산
       improvement = (no_cache_time - second_cache_time) / no_cache_time * 100
       print(f"캐시 사용 시 성능 향상: {improvement:.2f}%")

   # 메인 실행
   if __name__ == "__main__":
       # Redis 연결 테스트
       try:
           redis_client.ping()
           print("Redis 서버에 연결되었습니다.")
           
           # 성능 테스트 실행
           run_performance_test(100)
           
       except redis.ConnectionError:
           print("Redis 서버에 연결할 수 없습니다. 엔드포인트와 포트를 확인하세요.")
   ```

2. 다음 명령을 실행하여 필요한 패키지를 설치하고 스크립트를 실행합니다:
   ```bash
   sudo pip3 install redis
   python3 redis_benchmark.py
   ```

3. 출력 결과를 확인하여 캐시 사용 시 성능 향상을 확인합니다.

**확인 포인트:** 캐시 히트 시 데이터 조회 속도가 크게 향상되는 것을 확인합니다. 일반적으로 90% 이상의 성능 향상이 관찰됩니다.

### 9단계: Redis 클러스터 모니터링
**목적:** CloudWatch를 통해 Redis 클러스터의 성능을 모니터링합니다.

1. AWS Management Console에서 ElastiCache 서비스로 돌아갑니다.
2. Redis 클러스터 목록에서 `redis-basic-cluster`를 선택합니다.
3. "모니터링" 탭을 클릭합니다.
4. 다음 지표를 확인합니다:
   - CPU 사용률
   - 메모리 사용량
   - 캐시 히트 및 미스
   - 네트워크 바이트 입/출력
   - 연결 수

5. "CloudWatch에서 보기" 링크를 클릭하여 더 자세한 지표를 확인합니다.
6. CloudWatch 대시보드에서 다음 작업을 수행합니다:
   - 시간 범위를 "최근 1시간"으로 설정
   - "지표 추가"를 클릭하여 관심 있는 추가 지표 선택
   - 그래프 형식을 "선" 또는 "누적 영역"으로 변경

**확인 포인트:** Redis 클러스터의 주요 성능 지표를 확인하고, 모니터링 대시보드를 통해 클러스터 상태를 파악할 수 있습니다.

### 10단계: 클러스터 스케일링 테스트 (선택 사항)
**목적:** Redis 클러스터의 수직 및 수평 확장을 테스트합니다.

1. AWS Management Console에서 ElastiCache 서비스로 이동합니다.
2. Redis 클러스터 목록에서 `redis-cluster-mode`를 선택합니다.
3. "작업" 드롭다운 메뉴에서 "수정"을 선택합니다.
4. "샤드 수정" 섹션에서 다음 작업을 수행합니다:
   - "샤드 추가"를 클릭하여 새 샤드 추가
   - 노드 유형을 변경하여 수직 확장 (예: `cache.t3.small`에서 `cache.t3.medium`으로)
   
5. "수정 사항 미리 보기"를 클릭하여 변경 사항을 검토합니다.
6. "수정" 버튼을 클릭하여 변경 사항을 적용합니다.

**확인 포인트:** 클러스터 수정이 진행 중임을 확인하고, 상태가 "수정 중"에서 "사용 가능"으로 변경되는지 확인합니다. 이 과정은 약 10-15분 정도 소요될 수 있습니다.

> **참고:** 실습 비용을 고려하여 이 단계는 선택 사항입니다. 실제 변경을 적용하지 않고 "수정 사항 미리 보기"까지만 진행해도 됩니다.

## 결과 검증

1. Redis 클러스터가 성공적으로 생성되었는지 확인합니다.
   - 두 클러스터 모두 "사용 가능" 상태인지 확인
   - 엔드포인트 정보가 올바르게 표시되는지 확인

2. Redis 클라이언트를 통해 클러스터에 연결하고 기본 작업이 정상적으로 수행되는지 확인합니다.
   - 데이터 쓰기 및 읽기 작업 성공
   - 클러스터 정보 조회 성공

3. 캐싱 성능 테스트 결과를 확인합니다.
   - 캐시 히트 시 성능이 크게 향상되는지 확인
   - 캐시 미스와 히트의 차이 분석

4. CloudWatch 모니터링을 통해 클러스터 성능 지표를 확인합니다.
   - CPU 및 메모리 사용량이 정상 범위인지 확인
   - 캐시 히트율이 높은지 확인

## 리소스 정리

1. AWS Management Console에서 ElastiCache 서비스로 이동합니다.
2. Redis 클러스터 목록에서 `redis-cluster-mode`를 선택합니다.
3. "작업" 드롭다운 메뉴에서 "삭제"를 선택합니다.
4. 확인 대화 상자에서 "삭제"를 클릭합니다.
5. 같은 방법으로 `redis-basic-cluster`도 삭제합니다.
6. 왼쪽 탐색 창에서 "파라미터 그룹"을 선택합니다.
7. `redis-cache-params`를 선택하고 "삭제" 버튼을 클릭합니다.
8. 왼쪽 탐색 창에서 "서브넷 그룹"을 선택합니다.
9. `redis-cache-subnet-group`을 선택하고 "삭제" 버튼을 클릭합니다.
10. EC2 서비스로 이동하여 `redis-client` 인스턴스를 종료합니다.

> **중요:** 리소스를 정리하지 않으면 추가 비용이 발생할 수 있습니다.

## 도전 과제 (선택 사항)

1. **Redis 복제 그룹 장애 조치 테스트:** 기본 노드 장애 시 복제본 노드로의 자동 장애 조치를 테스트해 보세요.
   - 힌트: AWS CLI를 사용하여 장애 조치를 시뮬레이션할 수 있습니다.

2. **Redis 클러스터에 암호화 구성:** 전송 중 암호화 및 저장 데이터 암호화를 활성화하여 보안을 강화해 보세요.
   - 힌트: 새 클러스터를 생성할 때 암호화 옵션을 활성화해야 합니다.

3. **Redis 백업 및 복원 테스트:** 수동 스냅샷을 생성하고 새 클러스터로 복원하는 과정을 테스트해 보세요.
   - 힌트: ElastiCache 콘솔의 "백업" 탭에서 수동 스냅샷을 생성할 수 있습니다.

## 추가 학습 자료

- [Amazon ElastiCache for Redis 사용 설명서](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html)
- [Redis 공식 문서](https://redis.io/documentation)
- [AWS 데이터베이스 블로그 - ElastiCache 관련 글](https://aws.amazon.com/blogs/database/category/database/amazon-elasticache/)
- [AWS Well-Architected Framework - 성능 효율성 기둥](https://docs.aws.amazon.com/wellarchitected/latest/performance-efficiency-pillar/welcome.html)
- [Redis 캐싱 전략 및 모범 사례](https://aws.amazon.com/caching/best-practices/)