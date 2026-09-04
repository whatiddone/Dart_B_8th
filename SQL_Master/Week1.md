# SQL_MASTER 1주차 정규과제

📌SQL MASTER 정규과제는 매주 정해진 분량의 『*데이터 분석을 위한 SQL 레시피*』 를 읽고 학습하는 것입니다. 이번 주는 아래의 **SQL_MASTER_1st_TIL**에 나열된 분량을 읽고 공부하시면 됩니다.

아래 실습을 수행하며 학습 내용을 직접 적용해보세요. 단순히 결과를 재현하는 것이 아니라, SQL을 직접 작성하는 과정에서 개념을 스스로 정리하는 것이 중요합니다.

필요한 경우 교재와 추가 자료를 참고하여 이해를 보완하시기 바랍니다.

## SQL_MASTER_1st_TIL

### 3장 데이터 가공믈 위한 SQL
#### 1. 하나의 값 조작하기
#### 2. 여러 개의 값에 대한 조작
#### 3. 하나의 테이블에 대한 조작
#### 4. 여러 개의 테이블 조작하기


## Study Schedule

| 주차  | 공부 범위     | 완료 여부 |
| ----- | ------------- | --------- |
| 1주차 | p.52~136   | ✅         |
| 2주차 | p.138~184  | 🍽️         |
| 3주차 | p.186~232 | 🍽️         |
| 4주차 | p.233~321 | 🍽️         |
| 5주차 | p.324~406 | 🍽️         |
| 6주차 | p.408~464 | 🍽️         |
| 7주차 | p.466~566 | 🍽️         |

<br>

<!-- 여기까진 그대로 둬 주세요-->


# 실습

## 0. 실습 규칙

1. 샘플 데이터 생성 코드는 **08_SQL_MASTER_Template/src** 경로에 장별로 정리되어 있습니다.
2. 아래 목차에 맞춰 해당 코드를 실행하여 샘플 데이터를 생성한 후, 각 장에서 요구하는 쿼리를 직접 작성해보시기 바랍니다.
3. 작성한 쿼리의 **실행 결과 화면도 함께 제출**해 주세요.
4. 단순히 교재의 예시 코드를 그대로 작성하는 것이 아니라, **제시된 로직을 충분히 이해한 뒤 교재를 보지 않고 스스로 쿼리를 구성**해보는 것을 권장합니다.
5. 교재 예시는 PostgreSQL, Hive, BigQuery 등 다양한 DBMS 기준으로 제시되어 있기 때문에, **MySQL이 아닌 다른 SQL 환경을 사용하여 실습을 진행해도 무방합니다.**
6. 다만, 사용 중인 DBMS에 맞는 문법으로 적절히 변환하여 작성하시기 바랍니다.

## 1. 하나의 값 조작하기 

### 1-1 코드 값을 레이블로 변경하기

- 리포트 작성 시 가독성의 위해 코드 값을 레이블로 변경
- 특정 조건을 기반으로 값을 결정할 때는 **CASE 식**을 사용

    - CASE 식
    ```sql
    CASE
        WHEN <조건식 1> THEN <조건을 만족할 때의 값 1>
        WHEN <조건식 2> THEN <조건을 만족할 때의 값 2>
        WHEN <조건식 3> THEN <조건을 만족할 때의 값 3>
        ELSE <그 외의 값>
        END AS <새_컬럼이름> -- 컬럼명(Alias) 변경
    ```
---

#### 코드를 레이블로 변경
```sql
SELECT
    user_id,
    CASE
        WHEN register_device = 1 THEN '데스크톱'
        WHEN register_device = 2 THEN '스마트폰'
        WHEN register_device = 3 THEN '애플리케이션'
        END AS device_name
FROM `1st_week.mst_users`
ORDER BY user_id ASC;
```

![img](../SQL_Master/image/Week1/1.png)

### 1-2 URL에서 요소 추출하기

- 어떤 웹 페이지를 거쳐 넘어왔는지를 판별할 때는 레퍼러(지금 이 페이지에 오기 직전에 머물렀던 이전 페이지의 주소)를 집계
- URL 주소는 가독성이 좋지 않기 떄문에, **특정 값을 추출**하여 사용 

#### 1-2-1 레퍼러로 어떤 웹 페이지를 거쳐 넘어왔는지 판별하기
```sql
-- 레퍼러 도메인을 호스트 단위로 집계
-- 1. REGEXP_EXTRACT 사용
SELECT
    stamp,
    
    -- https:// 또는 http:// 뒤에 오는 첫 번째 슬래시(/) 전까지의 호스트명(도메인)만 괄호 ( )로 묶어 추출
    REGEXP_EXTRACT(referrer, r'https?://([^/]+)') AS referrer_host
FROM `1st_week.access_log`;
-- 2. NET.HOST 사용(Bigquery)
SELECT
    stamp,
    -- URL에서 호스트(도메인) 부분 추출
    NET.HOST(referrer) AS referrer_host
FROM `1st_week.access_log`;
```
```
r'https?://([^/]+)'
│ └──┬─┘ │ └──┬──┘
│    │   │    └─ ③ 괄호 안: 슬래시(/)가 아닌 글자들을 1개 이상 묶어서 '추출'
│    │   └────── ② 문자 그대로 콜론과 슬래시 두 개 (://)
│    └────────── ① http 또는 https 로 시작
└─────────────── (참고: 문자열 앞 'r'은 백슬래시를 있는 그대로 쓰겠다는 의미)
```
- REGEXP_EXTRACT 함수란?
```
1. 형식: REGEXP_EXTRACT(대상_컬럼, r'정규표현식_패턴')
2. 역할: 패턴 내에서 괄호 ( )로 묶은 그룹(Capture Group)에 해당하는 문자열만 추출
3. 일치하는 패턴이 없으면 NULL을 반환함
```
- **상황별 권장 사용 기준**
```
'NET.HOST'를 사용하는 경우
  * Referrer, 접속 URL 로그에서 **순수 도메인/호스트명**만 빠르고 간결하게 뽑아낼 때
  * 포트 번호(`:8080`), 불완전한 URL 구조 등을 내장 파서가 자체 처리해주어 정규식보다 속도가 빠르고 에러 위험이 적음

'REGEXP_EXTRACT'를 사용하는 경우
  * 호스트명 외에 특정 경로 세그먼트(`video/detail`), 쿼리 매개변수 값, 특정 텍스트 패턴 등 '규칙 기반의 유연한 문자열 분리가 필요할 때'
  * 타 데이터베이스(PostgreSQL 등)의 정규식 쿼리를 BigQuery로 이식할 때
```
#### 1-2-2 URL에서 경로와 요청 매개변수 값 추출하기
```sql
-- URL 경로와 GET 매개변수에 있는 특정 키 
-- 1. REGEXP_EXTRACT 사용
SELECT
    stamp,
    url,
    -- 1. URL 경로(Path) 추출 (도메인 뒤부터 ? 또는 # 전까지)
    REGEXP_EXTRACT(url, r'//[^/]+([^?#]+)') AS path,

    -- 2. GET 쿼리 매개변수 중 id의 값 추출
    REGEXP_EXTRACT(url, r'id=([^&]*)') AS id
FROM `1st_week.access_log`;

-- 2. NET 계열 함수 활용
SELECT
    stamp,
    url,
    
    -- 1. 호스트명(도메인) 추출
    NET.HOST(url) AS host,

    -- 2. URL 경로(Path) 추출
    -- 도메인(NET.HOST) 뒤에 붙은 문자열 중 '?' 앞부분까지 잘라내기
    REGEXP_EXTRACT(url, CONCAT('https?://', NET.HOST(url), '([^?#]*)')) AS path,

    -- 3. GET 매개변수 id 값 추출
    REGEXP_EXTRACT(url, r'[?&]id=([^&#]+)') AS id
FROM `1st_week.access_log`; 
```
<p align="center">
  <img src="image/Week1/2.png" height="200" alt="2">
  <img src="image/Week1/3.png" height="200" alt="3">
  <img src="image/Week1/4.png" height="200" alt="4">
  <img src="image/Week1/5.png" height="200" alt="5">
</p>

### 1-3 문자열을 배열로 분해하기

- 문자열 자료형은 더 **세부적으로 분해해서** 사용하는 경우가 많음


```sql
-- URL 경로를 슬래시로 분할해서 계층 추출
SELECT
  stamp,
  url,
  SPLIT(REGEXP_EXTRACT(url, r'//[^/]+([^?#]+)'), '/')[SAFE_ORDINAL(2)] AS path1,
  -- 1) REGEXP_EXTRACT: 도메인 뒤의 세부 경로(Path) 추출
  -- 2) SPLIT: '/' 기준으로 경로를 쪼개어 배열(List)로 반환
  -- 3) [SAFE_ORDINAL()]: n번째 요소 추출 (범위 초과 시 NULL 반환)
  SPLIT(REGEXP_EXTRACT(url, r'//[^/]+([^?#]+)'), '/')[SAFE_ORDINAL(3)] AS path2
FROM `1st_week.access_log`;
```

![img](../SQL_Master/image/Week1/6.png)

 - SPLIT 함수란?
```
1. 형식: SPLIT(문자열_컬럼, '구분자')
2. 역할: 지정한 구분자(delimiter)를 기준으로 문자열을 잘라 배열(Array) 형태로 반환함
3. 구분자가 포함되어 있지 않으면 원본 문자열 1개만 담긴 배열을 반환함
```

 - SAFE_ORDINAL(n) 인덱싱이란?
```
1. 형식: 배열_컬럼[SAFE_ORDINAL(n)]
2. 역할: 배열의 n번째 요소를 1부터 시작하는 순번 기준으로 추출함
3. 해당 위치(인덱스)에 데이터가 존재하지 않아도 오류(Error)를 내지 않고 안전하게 NULL을 반환함
```

### 1-4 날짜와 타임스탬프 다루기
#### 1-4-1 현재 날짜와 타임스탬프 추출하기

BigQuery는 UTC 시간 리턴 → CURRENT_TIMESTAMP로 리턴되는 시간이 한국 시간과 다름
PostegreSQL에서는 타임존이 적용된 타임스탬프 자료형 리턴


```sql
-- 현재 날짜와 타임스탬프 추출
SELECT
    CURRENT_DATE as dt,
    CURRENT_TIMESTAMP as STAMP
;
```
<figure>
  <img src="../SQL_Master/image/Week1/7.png" alt="결과 이미지" width="500">
  <figcaption><em>▲ BigQuery 실행 결과 화면: UTC 시간을 리턴한다.</em></figcaption>
</figure>

#### 1-4-2 지정한 값의 날짜/시각 데이터 추출하기

**CAST 함수를 사용하는 방법이 가장 범용적**

- CAST 함수란?: 데이터 타입 변환하는 함수
```
1. 형식: CAST(표현식 AS 목표_데이터타입)
2. 역할: 특정 컬럼이나 값의 데이터 타입을 다른 타입(INT64, STRING, DATE 등)으로 명시적 변환함
3. 변환이 불가능한 형태일 경우 쿼리 실행 에러가 발생함 (오류 대신 NULL을 원하면 SAFE_CAST 사용)
```

```sql
-- 문자열을 날짜/타임스탬프로 변환하기
 1. 명시적 형변환 (`CAST`)
 SELECT
  CAST ('2026-09-04' AS date) AS dt,
  CAST ('2026-09-04 15:47:45' AS timestamp) AS stamp
;
 2. 타입 생성자 함수 (`DATE()`, `TIMESTAMP()`)
 SELECT
  DATE('2026-09-04')AS dt,
  timestamp('2026-09-04 15:47:45') AS stamp
;
 3. 리터럴 표기법 (`DATE '...'`, `TIMESTAMP '...'`)
 SELECT
  DATE '2026-09-04'AS dt,
  timestamp '2026-09-04 15:47:45' AS stamp
;
```

<p align="center">
  <img src="image/Week1/8-1.png" height="200" alt="2">
  <img src="image/Week1/8-2.png" height="200" alt="3">
  <img src="image/Week1/8-3.png" height="200" alt="4">
</p>

📌 BigQuery 날짜·시간 타입 변환/생성 방식 3종 비교

| 구분 | 1. 명시적 형변환 (`CAST`) | 2. 타입 생성자 함수 (`DATE()`, `TIMESTAMP()`) | 3. 리터럴 표기법 (`DATE '...'`, `TIMESTAMP '...'`) |
| :--- | :--- | :--- | :--- |
| **작성 예시** | `CAST('2026-09-04' AS DATE)` | `DATE('2026-09-04')` | `DATE '2026-09-04'` |
| **문법 분류** | 표준 명시적 형변환 연산자 | 전용 데이터 생성/변환 내장 함수 | 데이터 타입 리터럴(고정 상수 표기법) |
| **적용 대상** | 모든 컬럼, 변수, 계산식 | 문자열 리터럴, 컬럼, 타임스탬프 객체 | **순수 고정 문자열 상수만 가능** |
| **추가 매개변수** | 불가 | **타임존(`time_zone`) 지정 가능** | 불가 |
| **변환 실패 시** | 쿼리 실행 에러 (대안: `SAFE_CAST` 사용 시 `NULL`) | 쿼리 실행 에러 | 쿼리 파싱(컴파일) 단계에서 에러 |
| **주요 활용 상황** | 테이블 내 기존 컬럼의 데이터 타입을 변경할 때 | 타임존을 반영해 변환하거나 다른 시간 타입끼리 맞출 때 | `WHERE` 조건절 등에서 특정 기준일을 상수로 비교/필터링할 때 |

```
💡 한 줄 요약 및 사용 가이드
*`CAST`: 표준 ANSI SQL 방식으로, **컬럼 데이터의 자료형을 변환**할 때 기본으로 사용합니다.
* 생성자 함수: 시차(`Asia/Seoul` 등)를 고려해 **타임존 기반 날짜/시간을 뽑아내야 할 때** 주로 사용합니다.
* 리터럴 표기: 함수 연산 오버헤드 없이 **고정된 기준 일자/시간을 직관적으로 필터링**할 때 가장 깔끔합니다.
```

#### 1-4-3 날짜/시각에서 특정 필드 추출하기
**EXTRACT 함수 사용**

 - EXTRACT 함수란?
```
1. 형식: EXTRACT(추출할_부분 FROM 대상_컬럼)
2. 역할: DATE, DATETIME, TIMESTAMP 등의 날짜/시간 데이터에서 원하는 특정 요소(연, 월, 일, 요일, 시간 등)만 숫자로 추출함
3. 대상 값이 NULL이면 NULL을 반환함
```
```sql
-- 타임스탬프 자료형의 데이터에서 연, 월, 일 등을 추출하는 쿼리
SELECT
  stamp,
  EXTRACT(YEAR FROM stamp)  AS year,
  EXTRACT(MONTH FROM stamp) AS month,
  EXTRACT(DAY FROM stamp)   AS day,
  EXTRACT(HOUR FROM stamp)  AS hour
FROM (
  SELECT CAST('2026-09-04 12:00:00' AS TIMESTAMP) AS stamp -- SELECT TIMESTAMP '2016-01-30 12:00:00' AS stamp 로도 가능
) AS t
;
```
![img](../SQL_Master/image/Week1/9-1.png)

**SUBSTRING(SUBSTR) 함수 사용**
 - SUBSTRING(SUBSTR)란?
```
1. 형식: SUBSTR(문자열_컬럼, 시작_위치, [추출할_길이])
2. 역할: 문자열의 특정 시작 위치부터 지정한 글자 수(길이)만큼 잘라내어 반환함 (시작 위치는 1부터 카운트)
3. 길이를 생략하면 시작 위치부터 문자열 끝까지 추출하며, 위치가 문자열 범위를 벗어나면 빈 문자열('') 또는 NULL을 반환함
```
 ```sql
SELECT
  stamp,
  SUBSTR(stamp, 1, 4)  AS year,
  SUBSTR(stamp, 6, 2)  AS month,
  SUBSTR(stamp, 9, 2)  AS day,
  SUBSTR(stamp, 12, 2) AS hour,
  SUBSTR(stamp, 1, 7)  AS year_month
FROM (
  SELECT CAST('2026-09-04 12:00:00' AS STRING) AS stamp --문자열 자료형인 string 사용
    -- 작은따옴표로 감싼 값 자체가 BigQuery에서는 기본 STRING 타입이므로 SELECT '2016-01-30 12:00:00' AS stamp 로도 가능
) AS t
;
 ```

 ![img](../SQL_Master/image/Week1/9-2.png)

#### 1-4-4 보충
- DATE, DATETIME, TIME, TIMESTAMP
    - DATE: 2002-03-10 (DATE 만 표시하는 데이터)
    - DATETIME: 2002-03-10 16:00:00 (DATE와 TIME까지 표시하는 데이터, Time Zone 정보 없음)
    - TIME: 23:59:59.00 (날짜와 무관하게 시간만 표시하는 데이터)
    - TIMESTAMP: 2023-12-31 14:00:00 UTC (UTC부터 경과한 시간을 나타내는 값, Time Zone 정보 있음)

```
timestamp: 주로 UTC 기준으로 시간 데이터를 저장하고, 데이터베이스 서버의 시간대에 따라 값을 자동으로 변환해줘요. 따라서 시간대 차이가 있는 환경에서 사용할 때 유리하고, 특히 로그 데이터나 시스템 관련 데이터를 다룰 때 유용해요. 다만, 일반적으로 1970년 1월 1일 이후의 시간을 기록할 수 있는 범위가 있어요(2038년까지).

datetime: 특정 시간대에 상관없이 그 자체로 시간을 기록하고, 시간대에 의한 자동 변환이 없어요. 특정 시간대를 고정해놓고 데이터에 사용해야 하는 경우에 유리해요. 예를 들어, 과거 이벤트나 로컬 시간대 기반의 일정을 기록하는 경우 적합해요.

요약
timestamp는 시간대 변환이 필요하거나 서버의 시간이 변동될 가능성이 있는 경우 적합해요.
datetime은 특정 시간대와 무관하게 고정된 시간 정보를 저장하고 싶을 때 좋아요.
```

### 1-5 결손 값을 디폴트 값으로 대치하기

문자열 또는 숫자를 다룰 때는 **중간에 NULL이 들어있는 경우를 주의**

```sql
-- 구매액에서 할인 쿠폰 값을 제외한 매출 금액을 구하는 쿼리
SELECT
  purchase_id,
  amount,
  coupon,
  amount - coupon AS discount_amount1,
  amount - COALESCE(coupon, 0) AS discount_amount2
FROM `1st_week.purchase_log_with_coupon`
ORDER BY purchase_id ASC
;
```

![img](../SQL_Master/image/Week1/10.png)

- COALESCE 함수란?
```
1. 형식: COALESCE(값1, 값2, 값3, ...)
2. 역할: 나열된 인자들을 왼쪽부터 순서대로 확인하여, '처음으로 NULL이 아닌 값'을 찾아 반환함
3. 모든 인자가 NULL이면 NULL을 반환함 (주로 NULL 값을 0이나 기본값으로 대체할 때 필수 사용)
```


## 2. 여러 개의 값에 대한 조작 

### 2-1 문자열을 연결하기

앞에서 REGEXP_EXTRACT 함수로 특정 값을 추출했다면 CONCAT 함수로는 단일 값으로 반환

```SQL![alt text](image.png)
SELECT
  user_id,
  CONCAT(pref_name, city_name) AS pref_city  -- CONCAT을 사용해 도/시(pref_name)와 시/군/구(city_name) 문자열을 연결
FROM `1st_week.mst_user_location`
ORDER BY user_id ASC;
```

![img](../SQL_Master/image/Week1/11.png)

### 2-2 여러 개의 값을 비교하기
#### 2-2-1 분기별 매출 증감 판정하기

**CASE 함수**를 이용하여 조건을 기술하고 조건에 맞는 값을 지정하거나, **SIGN 함수**를 이용하여 간단하게 증감 판정.

- SIGN 함수란?
```
1. 형식: SIGN(계산식)
2. 역할: 숫자의 부호(양수, 0, 음수)를 판별하여 각각 1, 0, -1 중 하나를 반환함
3. 입력값이 양수면 1, 0이면 0, 음수면 -1을 반환하며, NULL이면 NULL을 반환함
```

```SQL
SELECT
  year,
  q1,
  q2,
  CASE
    WHEN q1 < q2 THEN '+'
    WHEN q1 = q2 THEN ' '
    ELSE '-'
  END AS judge_q1_q2,  -- 1) CASE 문으로 Q1과 Q2의 매출 증감 여부를 기호(+, ' ', -)로 평가
  q2 - q1 AS diff_q2_q1, -- 2) Q2와 Q1의 매출 차이 계산
  SIGN(q2 - q1) AS sign_q2_q1 -- 3) SIGN 함수로 부호 판별 (증가: 1, 유지: 0, 감소: -1)
FROM `1st_week.quarterly_sales`
ORDER BY year;
```
![img](../SQL_Master/image/Week1/12.png)

#### 2-2-2 연간 최대/최소 4분기 매출 맞기

컬럼 값에서 최댓값 또는 최솟값을 찾을 때는 **GREATEST 함수** 또는 **LEAST 함수**를 사용

- GREATEST/LEAST 함수란?
```
1. 형식: GREATEST(LEAST)(값1, 값2, 값3, ...)
2. 역할: 나열된 여러 인자(컬럼 또는 값) 중 '최댓값(최솟값)'을 반환함
3. 인자 중 하나라도 NULL이 포함되어 있으면 NULL을 반환함
```

```SQL
SELECT
  year,
  -- NULL을 0으로 치환하여 존재하는 분기 중 최대 매출 계산
  GREATEST(
    COALESCE(q1, 0),
    COALESCE(q2, 0),
    COALESCE(q3, 0),
    COALESCE(q4, 0)
  ) AS greatest_sales,

  -- 최솟값은 0으로 치환 시 왜곡되므로 유효한 값만 골라내거나 기본 LEAST 적용
  LEAST(
    COALESCE(q1, 999999999),
    COALESCE(q2, 999999999),
    COALESCE(q3, 999999999),
    COALESCE(q4, 999999999)
  ) AS least_sales
FROM `1st_week.quarterly_sales`
ORDER BY year;
```

![img](../SQL_Master/image/Week1/13.png)

``` SQL
--보충: CONCAT 함수 활용하여 매출액(분기)' 형태 문자열 생성
SELECT
  year,
  -- 1) 순수 최댓값 숫자 구하기
  GREATEST(COALESCE(q1, 0), COALESCE(q2, 0), COALESCE(q3, 0), COALESCE(q4, 0)) AS greatest_sales,

  -- 2) CONCAT으로 '매출액(분기)' 형태 문자열 만들기
  CONCAT(
    CAST(GREATEST(COALESCE(q1, 0), COALESCE(q2, 0), COALESCE(q3, 0), COALESCE(q4, 0)) AS STRING),
    CASE GREATEST(COALESCE(q1, 0), COALESCE(q2, 0), COALESCE(q3, 0), COALESCE(q4, 0))
      WHEN q1 THEN '(Q1)'
      WHEN q2 THEN '(Q2)'
      WHEN q3 THEN '(Q3)'
      WHEN q4 THEN '(Q4)'
    END
  ) AS max_sales_with_quarter
FROM `1st_week.quarterly_sales`
ORDER BY year;
```
#### 2-2-3 연간 평균 4분기 매출 
```SQL
-- 단순 연산
SELECT
  year,
  (q1 + q2 + q3 + q4) / 4 AS average
FROM `1st_week.quarterly_sales`
ORDER BY year;
```
![img](../SQL_Master/image/Week1/14-1.png)<br>
⚠️ NULL 값이 있으므로 사칙연산이 제대로 작동하지 않음
```SQL
-- COALESCE를 사용해 NULL을 0으로변환
SELECT
  year,
  (COALESCE(q1, 0) + COALESCE(q2, 0) + COALESCE(q3, 0) + COALESCE(q4, 0)) / 4 AS average
FROM `1st_week.quarterly_sales`
ORDER BY year;
```

![img](../SQL_Master/image/Week1/14-2.png)<br>
⚠️ 분자가 0이므로 2017년의 평균이 낮아짐
```SQL
-- NULL이 아닌 컬럼만을 사용
SELECT
  year,
  (COALESCE(q1, 0) + COALESCE(q2, 0) + COALESCE(q3, 0) + COALESCE(q4, 0))
  / (
      SIGN(COALESCE(q1, 0)) + SIGN(COALESCE(q2, 0))
    + SIGN(COALESCE(q3, 0)) + SIGN(COALESCE(q4, 0)) -- NULL이 아닌 유효한 분기 개수의 합으로 나누기
  ) AS average
FROM `1st_week.quarterly_sales`
ORDER BY year;
```
![img](../SQL_Master/image/Week1/14-3.png)
```SQL
```

### 2-3 2개의 값 비율 계산하기
#### 2-3-1 정수 자료형의 데이터 나누기

```sql
여기에 코드를 적어주세요.
```
#### 2-3-2 0으로 나누는 것 피하기
<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 2-4 두 값의 거리 계산하기
#### 2-4-1 숫자 데이터의 절댓값, 제곱 평균 제곱근(RMS) 계산하기
#### xy 평면 위에 있는 두 점의 유클리드 거리 계산하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 2-5 날짜/시간을 계산하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 2-6 IP 주소 다루기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

## 03. 하나의 테이블에 대한 조작 

### 3-1 그룹의 특징 잡기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 3-2 그룹 내부의 순서

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 3-3 세로 기반 데이터를 가로 기반으로 변환하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 3-4 가로 기반 데이터를 세로 기반으로 변환하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->


## 04. 여러 개의 테이블 조작하기

### 4-1 여러 개의 테이블을 세로로 결합하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 4-2 여러 개의 테이블을 가로로 정렬하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 4-3 조건 플래그를 0과 1로 표현하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 4-4 계산한 테이블에 이름 붙여 재사용하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 4-5 유사 테이블 만들기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->



### 🎉 수고하셨습니다.
