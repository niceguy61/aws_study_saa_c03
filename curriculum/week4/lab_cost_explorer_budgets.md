# 실습: AWS Cost Explorer 및 Budgets를 활용한 비용 관리

## 개요
이 실습에서는 AWS Cost Explorer와 AWS Budgets를 활용하여 클라우드 비용을 분석하고 관리하는 방법을 학습합니다. 비용 데이터를 다양한 관점에서 분석하고, 예산을 설정하며, 비용 이상을 탐지하는 방법을 실습합니다.

**소요 시간:** 60분
**난이도:** 초급

## 학습 목표
이 실습을 완료하면 다음을 수행할 수 있게 됩니다:
- AWS Cost Explorer를 사용하여 비용 데이터를 분석하고 시각화하기
- 다양한 필터와 그룹화 옵션을 활용하여 비용 인사이트 얻기
- AWS Budgets를 설정하여 비용 한도를 모니터링하기
- 비용 알림을 구성하여 예산 초과 시 알림 받기
- 비용 할당 태그를 활성화하고 태그 기반 비용 분석하기

## 사전 요구사항
이 실습을 시작하기 전에 다음이 필요합니다:
- AWS 계정 액세스 (관리자 권한 또는 비용 관리 권한)
- 최소 1개월 이상의 AWS 사용 기록 (더 많은 데이터가 있을수록 좋음)
- AWS Management Console 기본 사용법 이해

## 아키텍처 개요

### 구현할 아키텍처
이 실습에서는 다음과 같은 비용 관리 아키텍처를 구현합니다:

```
+----------------------------------+
|                                  |
|       AWS Cost Explorer          |
|   (비용 분석 및 시각화)           |
|                                  |
+----------------------------------+
              |
              v
+----------------------------------+
|                                  |
|         AWS Budgets              |
|   (예산 설정 및 알림 구성)        |
|                                  |
+----------------------------------+
              |
              v
+----------------------------------+
|                                  |
|      Amazon SNS Topic            |
|   (예산 알림 전달)               |
|                                  |
+----------------------------------+
              |
              v
+----------------------------------+
|                                  |
|      이메일 알림                  |
|                                  |
+----------------------------------+
```

### 구성 요소
- **AWS Cost Explorer**: 비용 및 사용량 데이터를 분석하고 시각화하는 도구
- **AWS Budgets**: 예산을 설정하고 실제 또는 예상 비용이 예산을 초과할 때 알림을 보내는 서비스
- **Amazon SNS**: 예산 알림을 이메일로 전달하는 알림 서비스
- **비용 할당 태그**: 리소스에 태그를 지정하여 비용을 분류하고 추적하는 메커니즘

### 작동 방식
1. Cost Explorer를 통해 과거 및 현재 비용 데이터를 분석합니다.
2. 분석 결과를 바탕으로 AWS Budgets에서 예산을 설정합니다.
3. 예산 알림을 SNS 주제에 연결하여 이메일 알림을 구성합니다.
4. 비용이 설정한 임계값을 초과하면 자동으로 알림을 받습니다.
5. 비용 할당 태그를 활용하여 세분화된 비용 분석을 수행합니다.

## 실습 단계

### 1단계: AWS Cost Explorer 액세스 및 탐색
**목적:** AWS Cost Explorer에 액세스하고 기본 기능을 탐색합니다.

1. AWS Management Console에 로그인합니다.

2. 상단 검색창에 "Cost Explorer"를 입력하고 서비스를 선택하거나, 상단 오른쪽의 계정 이름을 클릭한 후 "Billing Dashboard"를 선택합니다.
   ![Cost Explorer 액세스](images/cost-explorer-access.png)

3. 왼쪽 탐색 메뉴에서 "Cost Explorer"를 클릭합니다.

4. "Launch Cost Explorer" 버튼을 클릭합니다. Cost Explorer가 처음 활성화되는 경우 데이터를 처리하는 데 최대 24시간이 걸릴 수 있습니다.
   ![Cost Explorer 시작](images/cost-explorer-launch.png)

5. Cost Explorer 인터페이스를 탐색합니다:
   - 상단의 시간 범위 선택기
   - 그래프 유형 선택 (막대, 선, 누적 영역)
   - 그룹화 및 필터 옵션
   - 저장된 보고서 목록

**확인 포인트:** Cost Explorer 대시보드가 표시되고 비용 데이터가 그래프로 시각화되는지 확인합니다.

### 2단계: 비용 데이터 분석
**목적:** 다양한 필터와 그룹화 옵션을 사용하여 비용 데이터를 분석합니다.

1. 시간 범위를 "Last 3 Months"로 설정합니다.
   ![시간 범위 설정](images/cost-explorer-time-range.png)

2. 그룹화 옵션을 "Service"로 설정하여 서비스별 비용을 확인합니다.
   ![서비스별 그룹화](images/cost-explorer-group-by-service.png)

3. 상위 5개 서비스를 식별하고 기록합니다.

4. 그룹화 옵션을 "Region"으로 변경하여 리전별 비용을 확인합니다.
   ![리전별 그룹화](images/cost-explorer-group-by-region.png)

5. 필터를 추가하여 특정 서비스(예: Amazon EC2)의 비용만 표시합니다:
   - "Filter" 버튼 클릭
   - "Service" 필터 추가
   - "Amazon Elastic Compute Cloud - Compute" 선택
   - "Apply" 클릭
   ![서비스 필터 적용](images/cost-explorer-service-filter.png)

6. 이제 그룹화 옵션을 "Instance Type"으로 변경하여 EC2 인스턴스 유형별 비용을 확인합니다.
   ![인스턴스 유형별 그룹화](images/cost-explorer-group-by-instance-type.png)

7. "Advanced Options"를 클릭하고 "Show costs as" 옵션을 "Daily"로 변경하여 일별 비용 추세를 확인합니다.
   ![일별 비용 보기](images/cost-explorer-daily-costs.png)

**확인 포인트:** 다양한 그룹화 및 필터 옵션을 적용하여 비용 데이터를 다양한 관점에서 분석할 수 있는지 확인합니다.

### 3단계: 저장된 보고서 생성
**목적:** 자주 사용하는 비용 보고서를 저장하고 관리합니다.

1. 이전 단계에서 구성한 EC2 인스턴스 유형별 비용 보고서를 저장합니다:
   - "Save as" 버튼 클릭
   - 보고서 이름: "EC2 비용 분석 - 인스턴스 유형별"
   - "Save report" 클릭
   ![보고서 저장](images/cost-explorer-save-report.png)

2. 새로운 보고서를 생성합니다:
   - "Clear" 버튼을 클릭하여 필터를 초기화합니다
   - 시간 범위를 "Last 6 Months"로 설정합니다
   - 그룹화 옵션을 "Service"로 설정합니다
   - "Advanced Options"에서 "Show costs as"를 "Monthly"로 설정합니다
   - "Save as" 버튼 클릭
   - 보고서 이름: "월별 서비스 비용 추세"
   - "Save report" 클릭

3. 왼쪽 탐색 메뉴에서 "Saved Reports"를 클릭하여 저장된 보고서 목록을 확인합니다.
   ![저장된 보고서](images/cost-explorer-saved-reports.png)

4. 저장한 "EC2 비용 분석 - 인스턴스 유형별" 보고서를 클릭하여 다시 로드합니다.

**확인 포인트:** 저장된 보고서가 목록에 표시되고, 클릭하면 해당 보고서 설정이 로드되는지 확인합니다.

### 4단계: AWS Budgets 설정
**목적:** AWS Budgets를 사용하여 비용 예산을 설정하고 알림을 구성합니다.

1. 왼쪽 탐색 메뉴에서 "Budgets"를 클릭합니다.

2. "Create budget" 버튼을 클릭합니다.
   ![예산 생성](images/aws-budgets-create.png)

3. "Use a template (simplified)" 옵션을 선택하고 "Monthly cost budget"을 선택한 후 "Configure budget" 버튼을 클릭합니다.
   ![예산 템플릿 선택](images/aws-budgets-template.png)

4. 예산 세부 정보를 입력합니다:
   - 예산 이름: "월별 총 비용 예산"
   - 예산 금액: 이전 달 비용의 110% 또는 적절한 금액 설정
   - 이메일 연락처: 알림을 받을 이메일 주소 입력
   - "Create budget" 버튼 클릭
   ![예산 세부 정보](images/aws-budgets-details.png)

5. 생성된 예산을 클릭하여 세부 정보를 확인합니다.
   ![예산 세부 정보 보기](images/aws-budgets-view.png)

6. 이제 좀 더 세부적인 예산을 생성합니다. "Create budget" 버튼을 다시 클릭합니다.

7. 이번에는 "Create budget" 페이지에서 "Customize (advanced)" 옵션을 선택하고 "Cost budget"을 선택한 후 "Next" 버튼을 클릭합니다.
   ![고급 예산 생성](images/aws-budgets-advanced.png)

8. 예산 세부 정보를 입력합니다:
   - 예산 이름: "EC2 서비스 비용 예산"
   - 기간: Monthly
   - 예산 갱신 유형: Recurring budget
   - 시작 날짜: 현재 월
   - 예산 금액 유형: Fixed
   - 예산 금액: EC2 서비스에 적절한 금액 설정
   - "Next" 버튼 클릭

9. 필터를 추가하여 EC2 서비스에 대한 예산을 설정합니다:
   - "Add filter" 클릭
   - "Service" 필터 선택
   - "Amazon Elastic Compute Cloud - Compute" 선택
   - "Next" 버튼 클릭
   ![예산 필터](images/aws-budgets-filter.png)

10. 알림 임계값을 구성합니다:
    - "Add an alert threshold" 클릭
    - 임계값 유형: Actual
    - 임계값: 80%
    - 이메일 수신자: 알림을 받을 이메일 주소 입력
    - "Next" 버튼 클릭
    ![알림 임계값](images/aws-budgets-alert.png)

11. 예산 검토 페이지에서 "Create budget" 버튼을 클릭합니다.

**확인 포인트:** 두 개의 예산이 생성되고 예산 대시보드에 표시되는지 확인합니다.

### 5단계: 비용 할당 태그 활성화 및 사용
**목적:** 비용 할당 태그를 활성화하고 태그 기반 비용 분석을 수행합니다.

1. 왼쪽 탐색 메뉴에서 "Cost allocation tags"를 클릭합니다.
   ![비용 할당 태그](images/cost-allocation-tags.png)

2. AWS 생성 태그 섹션에서 "aws:createdBy" 태그를 찾아 "Activate" 버튼을 클릭합니다.
   ![AWS 생성 태그 활성화](images/cost-allocation-aws-tags.png)

3. 사용자 정의 태그 섹션에서 사용 가능한 태그(예: "Environment", "Project" 등)를 찾아 "Activate" 버튼을 클릭합니다.
   ![사용자 정의 태그 활성화](images/cost-allocation-user-tags.png)

   > **참고:** 태그 활성화 후 비용 데이터에 태그가 표시되기까지 최대 24시간이 걸릴 수 있습니다. 이미 활성화된 태그가 있다면 다음 단계로 진행합니다.

4. Cost Explorer로 돌아가서 그룹화 옵션을 "Tag"로 설정하고 활성화한 태그 중 하나를 선택합니다.
   ![태그별 그룹화](images/cost-explorer-group-by-tag.png)

5. 태그 값별로 비용이 어떻게 분포되어 있는지 분석합니다.

6. 특정 태그 값에 대한 필터를 추가하여 더 세부적인 분석을 수행합니다:
   - "Filter" 버튼 클릭
   - 태그 필터 추가
   - 특정 태그 값 선택
   - "Apply" 클릭
   ![태그 필터 적용](images/cost-explorer-tag-filter.png)

**확인 포인트:** 비용 할당 태그가 활성화되고, Cost Explorer에서 태그 기반 그룹화 및 필터링이 가능한지 확인합니다.

### 6단계: 비용 이상 탐지 구성
**목적:** AWS Cost Anomaly Detection을 구성하여 비정상적인 비용 증가를 탐지합니다.

1. 왼쪽 탐색 메뉴에서 "Cost Anomaly Detection"을 클릭합니다.

2. "Cost monitors" 탭을 클릭한 후 "Create monitor" 버튼을 클릭합니다.
   ![이상 탐지 모니터 생성](images/cost-anomaly-create-monitor.png)

3. 모니터 유형으로 "AWS services"를 선택하고 "Next" 버튼을 클릭합니다.
   ![모니터 유형 선택](images/cost-anomaly-monitor-type.png)

4. 모니터 이름을 "AWS 서비스 비용 이상 탐지"로 입력하고 "Next" 버튼을 클릭합니다.

5. 알림 기본 설정을 구성합니다:
   - "Create new alert subscription" 선택
   - 구독 이름: "비용 이상 알림"
   - 임계값: Individual alerts
   - 알림 빈도: Daily
   - 알림 수신자: 알림을 받을 이메일 주소 입력
   - "Next" 버튼 클릭
   ![알림 구성](images/cost-anomaly-alert.png)

6. 검토 페이지에서 "Create monitor" 버튼을 클릭합니다.

7. 생성된 모니터를 확인하고 "Alert history" 탭을 클릭하여 이상 탐지 기록을 확인합니다.
   ![이상 탐지 기록](images/cost-anomaly-history.png)

**확인 포인트:** 비용 이상 탐지 모니터가 생성되고 알림 구독이 구성되었는지 확인합니다.

## 결과 검증
실습이 성공적으로 완료되었는지 확인하려면:

1. Cost Explorer에서 다양한 그룹화 및 필터 옵션을 사용하여 비용 데이터를 분석할 수 있는지 확인합니다.
   - 예상 결과: 서비스별, 리전별, 인스턴스 유형별 등 다양한 관점에서 비용 데이터 시각화

2. 저장된 보고서가 생성되고 액세스할 수 있는지 확인합니다.
   - 예상 결과: 저장된 보고서 목록에서 생성한 보고서 확인 가능

3. AWS Budgets에서 생성한 예산이 표시되는지 확인합니다.
   - 예상 결과: 예산 대시보드에 "월별 총 비용 예산"과 "EC2 서비스 비용 예산" 표시

4. 비용 할당 태그가 활성화되고 Cost Explorer에서 사용 가능한지 확인합니다.
   - 예상 결과: Cost Explorer의 그룹화 옵션에서 활성화한 태그 선택 가능

5. 비용 이상 탐지 모니터가 생성되고 알림 구독이 구성되었는지 확인합니다.
   - 예상 결과: Cost Anomaly Detection 대시보드에서 생성한 모니터 확인 가능

## 리소스 정리
이 실습에서 생성한 리소스는 추가 비용을 발생시키지 않지만, 더 이상 필요하지 않은 경우 다음과 같이 정리할 수 있습니다:

1. AWS Budgets에서 생성한 예산을 삭제하려면:
   - Budgets 대시보드로 이동
   - 삭제할 예산 선택
   - "Actions" 드롭다운 메뉴에서 "Delete" 선택
   - 삭제 확인

2. Cost Anomaly Detection에서 생성한 모니터를 삭제하려면:
   - Cost Anomaly Detection 대시보드로 이동
   - "Cost monitors" 탭 선택
   - 삭제할 모니터 선택
   - "Delete" 버튼 클릭
   - 삭제 확인

3. 저장된 Cost Explorer 보고서를 삭제하려면:
   - Cost Explorer의 "Saved Reports" 섹션으로 이동
   - 삭제할 보고서 옆의 "X" 아이콘 클릭
   - 삭제 확인

> **참고:** 비용 할당 태그 활성화는 비용이 발생하지 않으므로 비활성화할 필요가 없습니다.

## 도전 과제 (선택 사항)
1. **예측 보고서 생성:** Cost Explorer에서 향후 3개월 동안의 비용 예측 보고서를 생성하고 저장하세요.
   - 힌트: Cost Explorer의 "Forecasting" 옵션을 활용하세요.

2. **비용 효율성 보고서 생성:** EC2 예약 인스턴스 또는 Savings Plans 권장사항 보고서를 생성하여 잠재적인 비용 절감 기회를 식별하세요.
   - 힌트: Cost Explorer의 "Recommendations" 섹션을 확인하세요.

3. **고급 예산 설정:** 사용량 기반 예산(예: EC2 인스턴스 시간)을 설정하고 알림을 구성하세요.
   - 힌트: AWS Budgets에서 "Usage budget" 유형을 선택하세요.

## 추가 학습 자료
- [AWS Cost Explorer 사용 설명서](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
- [AWS Budgets 사용 설명서](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [AWS 비용 할당 태그 사용 설명서](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html)
- [AWS Cost Anomaly Detection 사용 설명서](https://docs.aws.amazon.com/cost-management/latest/userguide/getting-started-ad.html)
- [AWS Well-Architected Framework - 비용 최적화 원칙](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)