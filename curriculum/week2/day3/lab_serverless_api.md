# 실습: 서버리스 API 구축

## 개요
이 실습에서는 AWS Lambda와 Amazon API Gateway를 사용하여 간단한 서버리스 API를 구축합니다. 사용자 정보를 관리하는 RESTful API를 만들고, DynamoDB를 데이터 저장소로 사용하여 완전한 서버리스 아키텍처를 구현합니다. 이를 통해 서버리스 애플리케이션의 개발, 배포, 테스트 및 모니터링 방법을 학습합니다.

## 학습 목표
- AWS Lambda 함수 생성 및 구성 방법 습득
- Amazon API Gateway를 사용한 RESTful API 설계 및 구현
- DynamoDB 테이블 생성 및 Lambda 함수와의 통합
- API Gateway와 Lambda 함수 간의 통합 구성
- 서버리스 API 테스트 및 모니터링 방법 이해

## 사전 준비 사항
- AWS 계정
- 기본 JavaScript(Node.js) 지식
- REST API에 대한 기본 이해
- 웹 브라우저 및 텍스트 에디터

## 실습 시간
약 2시간

## 실습 단계

### 1. DynamoDB 테이블 생성

#### 1.1 DynamoDB 콘솔 접속
1. AWS 관리 콘솔에 로그인합니다.
2. 상단 검색창에 "DynamoDB"를 입력하거나 서비스 메뉴에서 "DynamoDB"를 선택합니다.
3. DynamoDB 대시보드가 표시됩니다.

#### 1.2 테이블 생성
1. 왼쪽 메뉴에서 "테이블"을 클릭합니다.
2. "테이블 생성" 버튼을 클릭합니다.
3. 다음 정보를 입력합니다:
   - 테이블 이름: `Users`
   - 파티션 키: `userId` (문자열)
   - 정렬 키: (비워둠)
   - 테이블 설정: 기본 설정 사용
4. "테이블 생성" 버튼을 클릭합니다.
5. 테이블이 생성될 때까지 기다립니다(상태가 "활성"으로 변경됨).

### 2. IAM 역할 생성

#### 2.1 IAM 콘솔 접속
1. 상단 검색창에 "IAM"을 입력하거나 서비스 메뉴에서 "IAM"을 선택합니다.
2. IAM 대시보드가 표시됩니다.

#### 2.2 Lambda 실행 역할 생성
1. 왼쪽 메뉴에서 "역할"을 클릭합니다.
2. "역할 생성" 버튼을 클릭합니다.
3. 신뢰할 수 있는 엔터티 유형으로 "AWS 서비스"를 선택합니다.
4. 사용 사례로 "Lambda"를 선택합니다.
5. "다음: 권한" 버튼을 클릭합니다.
6. 다음 정책을 검색하여 선택합니다:
   - `AmazonDynamoDBFullAccess`
   - `AWSLambdaBasicExecutionRole`
7. "다음: 태그" 버튼을 클릭합니다.
8. 태그 추가는 선택 사항이므로 "다음: 검토" 버튼을 클릭합니다.
9. 역할 이름에 `LambdaDynamoDBRole`을 입력합니다.
10. "역할 생성" 버튼을 클릭합니다.

### 3. Lambda 함수 생성

#### 3.1 Lambda 콘솔 접속
1. 상단 검색창에 "Lambda"를 입력하거나 서비스 메뉴에서 "Lambda"를 선택합니다.
2. Lambda 대시보드가 표시됩니다.

#### 3.2 사용자 조회 함수 생성
1. "함수 생성" 버튼을 클릭합니다.
2. "처음부터 작성"을 선택합니다.
3. 다음 정보를 입력합니다:
   - 함수 이름: `getUsers`
   - 런타임: Node.js 18.x
   - 아키텍처: x86_64
   - 실행 역할: 기존 역할 사용
   - 기존 역할: `LambdaDynamoDBRole`
4. "함수 생성" 버튼을 클릭합니다.
5. 함수 코드 편집기에 다음 코드를 입력합니다:

```javascript
const AWS = require('aws-sdk');
const docClient = new AWS.DynamoDB.DocumentClient();
const tableName = 'Users';

exports.handler = async (event) => {
    console.log('Received event:', JSON.stringify(event, null, 2));
    
    // 기본 응답 헤더
    const headers = {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'
    };
    
    try {
        // 경로 파라미터에서 userId 추출
        const userId = event.pathParameters ? event.pathParameters.userId : null;
        
        // 특정 사용자 조회
        if (userId) {
            const params = {
                TableName: tableName,
                Key: {
                    userId: userId
                }
            };
            
            const result = await docClient.get(params).promise();
            
            if (!result.Item) {
                return {
                    statusCode: 404,
                    headers: headers,
                    body: JSON.stringify({ message: 'User not found' })
                };
            }
            
            return {
                statusCode: 200,
                headers: headers,
                body: JSON.stringify(result.Item)
            };
        }
        
        // 모든 사용자 조회
        const params = {
            TableName: tableName
        };
        
        const result = await docClient.scan(params).promise();
        
        return {
            statusCode: 200,
            headers: headers,
            body: JSON.stringify(result.Items)
        };
    } catch (error) {
        console.error('Error:', error);
        
        return {
            statusCode: 500,
            headers: headers,
            body: JSON.stringify({ message: 'Internal server error', error: error.message })
        };
    }
};
```

6. "배포" 버튼을 클릭하여 함수를 저장합니다.

#### 3.3 사용자 생성 함수 생성
1. Lambda 대시보드로 돌아가서 "함수 생성" 버튼을 클릭합니다.
2. "처음부터 작성"을 선택합니다.
3. 다음 정보를 입력합니다:
   - 함수 이름: `createUser`
   - 런타임: Node.js 18.x
   - 아키텍처: x86_64
   - 실행 역할: 기존 역할 사용
   - 기존 역할: `LambdaDynamoDBRole`
4. "함수 생성" 버튼을 클릭합니다.
5. 함수 코드 편집기에 다음 코드를 입력합니다:

```javascript
const AWS = require('aws-sdk');
const { v4: uuidv4 } = require('uuid');
const docClient = new AWS.DynamoDB.DocumentClient();
const tableName = 'Users';

exports.handler = async (event) => {
    console.log('Received event:', JSON.stringify(event, null, 2));
    
    // 기본 응답 헤더
    const headers = {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'POST, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'
    };
    
    // OPTIONS 요청 처리 (CORS)
    if (event.httpMethod === 'OPTIONS') {
        return {
            statusCode: 200,
            headers: headers,
            body: ''
        };
    }
    
    try {
        // 요청 본문 파싱
        const requestBody = JSON.parse(event.body);
        
        // 필수 필드 검증
        if (!requestBody.name || !requestBody.email) {
            return {
                statusCode: 400,
                headers: headers,
                body: JSON.stringify({ message: 'Name and email are required' })
            };
        }
        
        // 사용자 객체 생성
        const user = {
            userId: uuidv4(),
            name: requestBody.name,
            email: requestBody.email,
            createdAt: new Date().toISOString()
        };
        
        // DynamoDB에 사용자 저장
        const params = {
            TableName: tableName,
            Item: user
        };
        
        await docClient.put(params).promise();
        
        return {
            statusCode: 201,
            headers: headers,
            body: JSON.stringify(user)
        };
    } catch (error) {
        console.error('Error:', error);
        
        return {
            statusCode: 500,
            headers: headers,
            body: JSON.stringify({ message: 'Internal server error', error: error.message })
        };
    }
};
```

6. "배포" 버튼을 클릭하여 함수를 저장합니다.

#### 3.4 사용자 삭제 함수 생성
1. Lambda 대시보드로 돌아가서 "함수 생성" 버튼을 클릭합니다.
2. "처음부터 작성"을 선택합니다.
3. 다음 정보를 입력합니다:
   - 함수 이름: `deleteUser`
   - 런타임: Node.js 18.x
   - 아키텍처: x86_64
   - 실행 역할: 기존 역할 사용
   - 기존 역할: `LambdaDynamoDBRole`
4. "함수 생성" 버튼을 클릭합니다.
5. 함수 코드 편집기에 다음 코드를 입력합니다:

```javascript
const AWS = require('aws-sdk');
const docClient = new AWS.DynamoDB.DocumentClient();
const tableName = 'Users';

exports.handler = async (event) => {
    console.log('Received event:', JSON.stringify(event, null, 2));
    
    // 기본 응답 헤더
    const headers = {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'DELETE, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'
    };
    
    // OPTIONS 요청 처리 (CORS)
    if (event.httpMethod === 'OPTIONS') {
        return {
            statusCode: 200,
            headers: headers,
            body: ''
        };
    }
    
    try {
        // 경로 파라미터에서 userId 추출
        const userId = event.pathParameters ? event.pathParameters.userId : null;
        
        if (!userId) {
            return {
                statusCode: 400,
                headers: headers,
                body: JSON.stringify({ message: 'userId is required' })
            };
        }
        
        // 사용자 존재 여부 확인
        const getParams = {
            TableName: tableName,
            Key: {
                userId: userId
            }
        };
        
        const getResult = await docClient.get(getParams).promise();
        
        if (!getResult.Item) {
            return {
                statusCode: 404,
                headers: headers,
                body: JSON.stringify({ message: 'User not found' })
            };
        }
        
        // 사용자 삭제
        const deleteParams = {
            TableName: tableName,
            Key: {
                userId: userId
            }
        };
        
        await docClient.delete(deleteParams).promise();
        
        return {
            statusCode: 200,
            headers: headers,
            body: JSON.stringify({ message: 'User deleted successfully' })
        };
    } catch (error) {
        console.error('Error:', error);
        
        return {
            statusCode: 500,
            headers: headers,
            body: JSON.stringify({ message: 'Internal server error', error: error.message })
        };
    }
};
```

6. "배포" 버튼을 클릭하여 함수를 저장합니다.

### 4. API Gateway 생성 및 구성

#### 4.1 API Gateway 콘솔 접속
1. 상단 검색창에 "API Gateway"를 입력하거나 서비스 메뉴에서 "API Gateway"를 선택합니다.
2. API Gateway 대시보드가 표시됩니다.

#### 4.2 REST API 생성
1. "API 생성" 버튼을 클릭합니다.
2. "REST API" 카드에서 "구축" 버튼을 클릭합니다.
3. "새 API" 옵션을 선택합니다.
4. 다음 정보를 입력합니다:
   - API 이름: `UsersAPI`
   - 설명: API for managing users
   - 엔드포인트 유형: 리전
5. "API 생성" 버튼을 클릭합니다.

#### 4.3 리소스 및 메서드 구성
1. "리소스" 탭에서 "/" 리소스를 선택합니다.
2. "작업" 드롭다운 메뉴에서 "리소스 생성"을 선택합니다.
3. 다음 정보를 입력합니다:
   - 리소스 이름: `users`
   - 리소스 경로: `/users`
   - CORS 활성화: 체크
4. "리소스 생성" 버튼을 클릭합니다.

5. `/users` 리소스를 선택하고 "작업" 드롭다운 메뉴에서 "메서드 생성"을 선택합니다.
6. 드롭다운 메뉴에서 "GET"을 선택하고 체크 버튼을 클릭합니다.
7. 다음 정보를 입력합니다:
   - 통합 유형: Lambda 함수
   - Lambda 프록시 통합 사용: 체크
   - Lambda 리전: 함수가 있는 리전 선택
   - Lambda 함수: `getUsers`
8. "저장" 버튼을 클릭합니다.
9. Lambda 함수에 권한을 추가하라는 팝업이 표시되면 "확인"을 클릭합니다.

10. `/users` 리소스를 선택하고 "작업" 드롭다운 메뉴에서 "메서드 생성"을 선택합니다.
11. 드롭다운 메뉴에서 "POST"를 선택하고 체크 버튼을 클릭합니다.
12. 다음 정보를 입력합니다:
    - 통합 유형: Lambda 함수
    - Lambda 프록시 통합 사용: 체크
    - Lambda 리전: 함수가 있는 리전 선택
    - Lambda 함수: `createUser`
13. "저장" 버튼을 클릭합니다.
14. Lambda 함수에 권한을 추가하라는 팝업이 표시되면 "확인"을 클릭합니다.

15. `/users` 리소스를 선택하고 "작업" 드롭다운 메뉴에서 "리소스 생성"을 선택합니다.
16. 다음 정보를 입력합니다:
    - 리소스 이름: `userId`
    - 리소스 경로: `/{userId}`
    - CORS 활성화: 체크
17. "리소스 생성" 버튼을 클릭합니다.

18. `/{userId}` 리소스를 선택하고 "작업" 드롭다운 메뉴에서 "메서드 생성"을 선택합니다.
19. 드롭다운 메뉴에서 "GET"을 선택하고 체크 버튼을 클릭합니다.
20. 다음 정보를 입력합니다:
    - 통합 유형: Lambda 함수
    - Lambda 프록시 통합 사용: 체크
    - Lambda 리전: 함수가 있는 리전 선택
    - Lambda 함수: `getUsers`
21. "저장" 버튼을 클릭합니다.
22. Lambda 함수에 권한을 추가하라는 팝업이 표시되면 "확인"을 클릭합니다.

23. `/{userId}` 리소스를 선택하고 "작업" 드롭다운 메뉴에서 "메서드 생성"을 선택합니다.
24. 드롭다운 메뉴에서 "DELETE"를 선택하고 체크 버튼을 클릭합니다.
25. 다음 정보를 입력합니다:
    - 통합 유형: Lambda 함수
    - Lambda 프록시 통합 사용: 체크
    - Lambda 리전: 함수가 있는 리전 선택
    - Lambda 함수: `deleteUser`
26. "저장" 버튼을 클릭합니다.
27. Lambda 함수에 권한을 추가하라는 팝업이 표시되면 "확인"을 클릭합니다.

#### 4.4 API 배포
1. "작업" 드롭다운 메뉴에서 "API 배포"를 선택합니다.
2. 다음 정보를 입력합니다:
   - 배포 스테이지: [새 스테이지]
   - 스테이지 이름: `dev`
   - 스테이지 설명: Development stage
   - 배포 설명: Initial deployment
3. "배포" 버튼을 클릭합니다.
4. 스테이지 편집기가 표시되면 "스테이지 세부 정보" 탭에서 "호출 URL"을 확인하고 메모합니다.

### 5. API 테스트

#### 5.1 API Gateway 콘솔에서 테스트
1. API Gateway 콘솔의 왼쪽 메뉴에서 "리소스"를 클릭합니다.
2. `/users` 리소스의 "POST" 메서드를 선택합니다.
3. "테스트" 탭을 클릭합니다.
4. 요청 본문에 다음 JSON을 입력합니다:

```json
{
  "name": "John Doe",
  "email": "john.doe@example.com"
}
```

5. "테스트" 버튼을 클릭합니다.
6. 응답 본문에 생성된 사용자 정보가 표시되는지 확인합니다.
7. 반환된 `userId`를 메모합니다.

8. `/users` 리소스의 "GET" 메서드를 선택합니다.
9. "테스트" 탭을 클릭합니다.
10. "테스트" 버튼을 클릭합니다.
11. 응답 본문에 방금 생성한 사용자가 포함된 목록이 표시되는지 확인합니다.

12. `/{userId}` 리소스의 "GET" 메서드를 선택합니다.
13. "테스트" 탭을 클릭합니다.
14. 경로에서 `userId`에 앞서 메모한 ID를 입력합니다.
15. "테스트" 버튼을 클릭합니다.
16. 응답 본문에 해당 사용자의 정보가 표시되는지 확인합니다.

17. `/{userId}` 리소스의 "DELETE" 메서드를 선택합니다.
18. "테스트" 탭을 클릭합니다.
19. 경로에서 `userId`에 앞서 메모한 ID를 입력합니다.
20. "테스트" 버튼을 클릭합니다.
21. 응답 본문에 삭제 성공 메시지가 표시되는지 확인합니다.

#### 5.2 Postman 또는 curl을 사용한 테스트
1. 앞서 메모한 API 호출 URL을 사용하여 외부에서 API를 테스트합니다.

**curl 명령어 예시**:

사용자 생성:
```bash
curl -X POST \
  https://your-api-id.execute-api.region.amazonaws.com/dev/users \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "Jane Smith",
    "email": "jane.smith@example.com"
  }'
```

모든 사용자 조회:
```bash
curl -X GET \
  https://your-api-id.execute-api.region.amazonaws.com/dev/users
```

특정 사용자 조회:
```bash
curl -X GET \
  https://your-api-id.execute-api.region.amazonaws.com/dev/users/user-id
```

사용자 삭제:
```bash
curl -X DELETE \
  https://your-api-id.execute-api.region.amazonaws.com/dev/users/user-id
```

### 6. 모니터링 및 로깅 구성

#### 6.1 CloudWatch 로그 확인
1. Lambda 콘솔에서 함수 중 하나를 선택합니다.
2. "모니터링" 탭을 클릭합니다.
3. "로그" 섹션에서 "CloudWatch에서 로그 보기" 링크를 클릭합니다.
4. 로그 스트림 목록이 표시됩니다.
5. 최신 로그 스트림을 클릭하여 함수 실행 로그를 확인합니다.

#### 6.2 API Gateway 로깅 활성화
1. API Gateway 콘솔에서 "스테이지"를 클릭합니다.
2. `dev` 스테이지를 선택합니다.
3. "로그/추적" 탭을 클릭합니다.
4. "CloudWatch 로그" 섹션에서 "CloudWatch 로그 활성화"를 체크합니다.
5. 로그 수준으로 "INFO"를 선택합니다.
6. "액세스 로깅" 섹션에서 "CloudWatch 로그 활성화"를 체크합니다.
7. "변경 사항 저장" 버튼을 클릭합니다.

#### 6.3 CloudWatch 지표 확인
1. CloudWatch 콘솔로 이동합니다.
2. 왼쪽 메뉴에서 "지표"를 클릭합니다.
3. "모든 지표" 탭에서 "API Gateway"를 클릭합니다.
4. "API별, 스테이지별, 리소스별"을 클릭합니다.
5. API 및 스테이지에 해당하는 지표를 선택하여 그래프로 확인합니다.

### 7. 정리

#### 7.1 API Gateway API 삭제
1. API Gateway 콘솔에서 `UsersAPI`를 선택합니다.
2. "작업" 드롭다운 메뉴에서 "API 삭제"를 선택합니다.
3. 확인 대화 상자에 "confirm"을 입력하고 "삭제" 버튼을 클릭합니다.

#### 7.2 Lambda 함수 삭제
1. Lambda 콘솔에서 생성한 세 함수(`getUsers`, `createUser`, `deleteUser`)를 각각 선택합니다.
2. "작업" 드롭다운 메뉴에서 "함수 삭제"를 선택합니다.
3. 확인 대화 상자에서 "삭제" 버튼을 클릭합니다.

#### 7.3 DynamoDB 테이블 삭제
1. DynamoDB 콘솔에서 `Users` 테이블을 선택합니다.
2. "작업" 드롭다운 메뉴에서 "테이블 삭제"를 선택합니다.
3. 확인 대화 상자에서 "삭제" 버튼을 클릭합니다.

#### 7.4 IAM 역할 삭제
1. IAM 콘솔에서 "역할"을 클릭합니다.
2. `LambdaDynamoDBRole`을 검색하여 선택합니다.
3. "역할 삭제" 버튼을 클릭합니다.
4. 확인 대화 상자에서 "예, 삭제합니다"를 클릭합니다.

## 도전 과제

### 도전 과제 1: 사용자 업데이트 기능 추가
1. 사용자 정보를 업데이트하는 Lambda 함수(`updateUser`)를 생성합니다.
2. API Gateway에 `/{userId}` 리소스에 대한 "PUT" 메서드를 추가합니다.
3. Lambda 함수와 API Gateway 메서드를 통합합니다.
4. 새로운 엔드포인트를 테스트합니다.

### 도전 과제 2: 인증 추가
1. Amazon Cognito 사용자 풀을 생성합니다.
2. API Gateway에 Cognito 권한 부여자를 구성합니다.
3. API 엔드포인트에 권한 부여자를 연결합니다.
4. 인증된 요청으로 API를 테스트합니다.

### 도전 과제 3: 페이지네이션 구현
1. `getUsers` Lambda 함수를 수정하여 페이지네이션을 지원합니다.
2. 쿼리 파라미터로 `limit`와 `nextToken`을 처리하도록 구현합니다.
3. DynamoDB의 `Scan` 작업에 페이지네이션 파라미터를 적용합니다.
4. 페이지네이션이 적용된 API를 테스트합니다.

## 결론
이 실습을 통해 AWS Lambda, Amazon API Gateway, Amazon DynamoDB를 사용하여 완전한 서버리스 API를 구축하는 방법을 학습했습니다. 서버리스 아키텍처는 서버 관리 없이 확장 가능하고 비용 효율적인 애플리케이션을 개발할 수 있게 해줍니다. 이러한 서비스를 조합하여 다양한 유형의 애플리케이션을 구축할 수 있으며, 필요에 따라 인증, 권한 부여, 캐싱, 스로틀링 등의 기능을 추가하여 더욱 강력한 API를 만들 수 있습니다.

## 참고 자료
- [AWS Lambda 개발자 안내서](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Amazon API Gateway 개발자 안내서](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [Amazon DynamoDB 개발자 안내서](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [서버리스 애플리케이션 렌즈 - AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/welcome.html)