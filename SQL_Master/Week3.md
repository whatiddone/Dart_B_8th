# SQL_MASTER 3주차 정규과제

📌SQL MASTER 정규과제는 매주 정해진 분량의 『*데이터 분석을 위한 SQL 레시피*』 를 읽고 학습하는 것입니다. 이번 주는 아래의 **SQL_MASTER_3rd_TIL**에 나열된 분량을 읽고 공부하시면 됩니다.

아래 실습을 수행하며 학습 내용을 직접 적용해보세요. 단순히 결과를 재현하는 것이 아니라, SQL을 직접 작성하는 과정에서 개념을 스스로 정리하는 것이 중요합니다.

필요한 경우 교재와 추가 자료를 참고하여 이해를 보완하시기 바랍니다.

## SQL_MASTER_3rd_TIL

### 5장 사용자를 파악하기 위한 데이터 추출
#### 1. 사용자 전체의 특징과 경향 찾기


## Study Schedule

| 주차  | 공부 범위     | 완료 여부 |
| ----- | ------------- | --------- |
| 1주차 | p.52~136   | ✅         |
| 2주차 | p.138~184  | ✅         |
| 3주차 | p.186~232 | ✅         |
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

## 1. 사용자 전체의 특징과 경향 찾기

**핵심 원리 및 요약**
- 사용자가 서비스 내부에서 제공되는 기능 등을 얼마나 이용하는지 집계하는 작업은 사용자의 행동 패턴을 파악할 때와 어떤 대책의 효과를 확인할 때 굉장히 중요하며, 매우 자주 하게 되는 작업
- 특정 액션의 사용률과 사용자가 평균적으로 액션을 몇 번이나 사용했는지 확인 가능

### 1-1 사용자의 액션 수 집계하기
#### 1-1-1 액션과 관련된 지표 집계하기
- 사용률(usage_rate): 특정 액션 UU/전체 액션 UU
```sql
-- 액션 수와 비율을 계산하는 쿼리
WITH stats AS (          
  SELECT  
    COUNT(DISTINCT session) AS total_uu 
  FROM `3rd_week.action_log`
)
SELECT
  l.action,
  COUNT(DISTINCT l.session) AS action_uu,
  COUNT(1) AS action_count,
  s.total_uu,
  100.0 * COUNT(DISTINCT l.session) / s.total_uu AS usage_rate, -- 전체 사용자 대비 액션 사용률(%): <액션 UU> / <전체 UU>
  1.0 * COUNT(1) / COUNT(DISTINCT l.session) AS count_per_user  -- 해당 액션을 수행한 유저 1인당 평균 실행 횟수: <액션 수> / <액션 UU>
FROM `3rd_week.action_log` AS l
CROSS JOIN stats AS s                                           -- 로그 전체의 유니크 사용자 수를 모든 레코드에 결합하기
GROUP BY
  l.action,
  s.total_uu    
```
![img](../SQL_Master/image/Week3/11-1.png)

**결과 설명 / 주의점**
- `stats` CTE를 사용하여 전체 유니크 사용자 수를 계산합니다.
- `action_log` 테이블에서 각 액션의 유니크 사용자 수와 전체 액션 수를 계산합니다.
- 사용률과 사용자가 평균적으로 액션을 몇 번이나 사용했는지 계산하여 결과를 출력합니다.
- 주의점: `COUNT(DISTINCT session)`은 유니크 사용자 수를 계산하고, `COUNT(action)`은 전체 액션 수를 계산합니다.

#### 1-1-2 로그인 사용자와 비로그인 사용자를 구분해서 집계하기

**핵심 원리 및 요약**
- `user_id`가 존재하면 `login_status`를 `login`으로, 그렇지 않으면 `guest`로 설정
- `action_log_with_status`라는 CTE를 생성하여 로그인 상태 판별

```sql
-- 로그인 상태를 판별하는 쿼리
WITH action_log_with_status AS (
  SELECT
    session,
    user_id, 
    action,
    CASE
      WHEN COALESCE(user_id, '') <> '' THEN 'login' -- user_id가 NULL 또는 빈 문자가 아닌 경우 'login' 판정
      ELSE 'guest' -- NULL이거나 빈 문자열('')이면 'guest' 판정
    END AS login_status 
  FROM `3rd_week.action_log` 
)
SELECT *
FROM action_log_with_status;
```
![img](../SQL_Master/image/Week3/11-2.png)

**결과 설명 / 주의점**
- `user_id`가 존재하면 `login_status`는 `login`으로 설정됩니다.
- `user_id`가 없으면 `login_status`는 `guest`로 설정됩니다.
- 결과는 `session`, `user.id`, `action`, `login_status`를 포함합니다.

---

**핵심 원리 및 요약**
- `login_status`를 기반으로 액션 수와 UU(Unique Users: 중복 없이 집계된 사용자 수)를 집계합니다.
- `ROLLUP` 구문을 사용하여 `action`과 `login_status`의 모든 조합을 포함합니다.
- `ROLLUP` 구문이 지원되지 않는 경우 `UNION ALL`을 사용하여 같은 결과를 얻을 수 있습니다.

```sql
-- 로그인 상태에 따라 액션 수 등을 따로 집계하는 쿼리
WITH action_log_with_status AS (
  SELECT
    session,
    user_id, 
    action,
    CASE
      WHEN COALESCE(user_id, '') <> '' THEN 'login'
      ELSE 'guest'
    END AS login_status  
  FROM `3rd_week.action_log` 
)

-- 2. ROLLUP을 통한 계층별 소계/총계 집계
SELECT
  -- ROLLUP에 의해 합산되어 NULL로 표시되는 행을 'all'로 치환
  COALESCE(action, 'all')       AS action,
  COALESCE(login_status, 'all') AS login_status,
  COUNT(DISTINCT session)       AS action_uu,  
  COUNT(1)                      AS action_count 

FROM action_log_with_status  

-- BigQuery 표준 SQL은 ROLLUP(컬럼1, 컬럼2) 문법을 사용
GROUP BY ROLLUP(action, login_status)                                 -- (action, status)별 상세 -> action별 소계 -> 전체 총계 생성
ORDER BY
  action, login_status;
```
![img](../SQL_Master/image/Week3/11-3.png)

- 로그 정보의 user_id 정보를 기반으로 집계한 데이터이므로, 비로그인 사용자가 로그인하면 
각각의 액션에 1 씩 추가됨. 

**결과 설명 / 주의점**
- `action`과 `login_status`의 모든 조합에 대한 액션 수와 UU를 집계합니다.
- `action`과 `login_status`가 모두 `all`인 경우 전체 데이터를 포함합니다.
- 결과는 `action`, `login_status`, `action_uu`, `action_count`를 포함합니다.

#### 1-1-3 회원과 비회원을 구분해서 집계하기

**핵심 원리 및 요약**
- `user_id`가 존재하면 `login_status`를 `login`으로, 그렇지 않으면 `guest`로 설정합니다.
- `action_log_with_status`라는 CTE를 생성하여 로그인 상태를 판별합니다.
- `member_status`를 추가하여 한 번이라도 로그인한 사용자를 `member`로 설정합니다.

```sql
WITH action_log_with_status AS (
  SELECT
    session,
    user_id,
    action,

    -- [회원 여부(member_status) 판정 로직]
    CASE 
      WHEN COALESCE(
        -- 동일 세션 내에서 시간(stamp) 순서대로 첫 행부터 현재 행까지 user_id의 최댓값을 누적 탐색
        -- (한 번이라도 로그인하여 user_id가 찍혔다면 그 이후 행들에도 해당 user_id가 계속 유지되어 전파됨)
        MAX(user_id) OVER(
          PARTITION BY session 
          ORDER BY stamp 
          ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ), 
        '' -- 누적된 user_id가 없으면(NULL) 빈 문자열('')로 치환
      ) <> '' THEN 'member' -- 현재 행 시점까지 로그인한 이력이 존재하면 'member'
      ELSE 'none'           -- 아직 로그인한 적이 없다면 'none'
    END AS member_status,

    stamp
  FROM
    `sql-master-507514.3rd_week.action_log`
)
SELECT *
FROM action_log_with_status;
```
![img](../SQL_Master/image/Week3/11-4.png)

**결과 설명 / 주의점**
- `user_id`가 존재하면 `login_status`는 `login`으로 설정됩니다.
- `user_id`가 없으면 `login_status`는 `guest`로 설정됩니다.
- 한 번이라도 로그인한 사용자를 `member_status`로 설정합니다.
- 결과는 `session`, `user.id`, `action`, `member_status`, `stamp`를 포함합니다.

> 원포인트
```
로그인 하지 않은 상태일 경우, 사용자 ID 컬럼의 값이 빈 문자열 또는 NULL 일 수 있기 때문에, COALESCE 함수를 사용해 빈 문자열로 전환하여 해결 가능

로그인하지 않은 때의 사용자 ID를 빈 문자열로 저장했다면, COUNT(DISTINCT user_id)의 결과에 1이 추가된다. 따라서, COUNT(DISTINCT user_id)를 정확하게 추출하려면 사용자 ID를 NULL로 지정하는 것이 좋다.
```
### 1-2 연령별 구분 집계하기

**핵심 원리 및 요약**
- 연령별 구분을 사용자에 추가하여 다양한 리포트를 만들 수 있음
- 생일을 기반으로 특정 날짜의 나이를 계산하고, 연령별 구분을 계산하여 사용자 정보에 추가
- 나이는 생일과 특정 날짜를 정수로 표현하고, 이 차이를 10,000으로 나누는 방법으로 간단하게 계산 가능

```sql
-- 사용자의 생일을 계산하는 쿼리
WITH mst_users_with_int_birth_date AS (
  SELECT
    *,
    20170101 AS int_specific_date,   -- 특정 날짜(2017년 1월 1일)의 정수 표현
    CAST(REPLACE(SUBSTR(birth_date, 1, 10), '-', '') AS INT64) AS int_birth_date  -- 문자열로 구성된 생년월일을 정수 표현으로 변환 (BigQuery 규격: SUBSTR + CAST AS INT64)
  FROM
    `3rd_week.mst_users`
)
, mst_users_with_age AS (
  SELECT
    *,
    FLOOR((int_specific_date - int_birth_date) / 10000) AS age  -- 특정 날짜(2017년 1월 1일)의 나이
  FROM
    mst_users_with_int_birth_date
)
SELECT
  user_id,
  sex,
  birth_date,
  age
FROM
  mst_users_with_age
;
```
![img](../SQL_Master/image/Week3/11-5.png)

**결과 설명 / 주의점**
- `user_id`가 존재하면 `login_status`는 `'login'`으로 설정됩니다[cite: 3].
- `user_id`가 없거나 빈 문자열이면 `login_status`는 `'guest'`로 설정됩니다[cite: 3].
- 세션 내에서 한 번이라도 로그인한 이력이 있는 행부터는 누적 윈도우 함수를 통해 `member_status`가 `'member'`로 설정됩니다.
- 로그인한 적이 없는 이전 행들은 `member_status`가 `'none'`으로 유지됩니다.
- 결과는 `session`, `user_id`, `action`, `member_status`, `stamp`를 포함합니다.
---
```sql
-- 성별과 연령으로 연령별 구분을 계산하는 쿼리
WITH mst_users_with_int_birth_date AS (
  SELECT
    *,
    20170101 AS int_specific_date,                                              -- 1. 나이 계산의 기준일(2017년 1월 1일)을 8자리 정수(YYYYMMDD)로 지정
    CAST(REPLACE(SUBSTR(birth_date, 1, 10), '-', '') AS INT64) AS int_birth_date -- 2. 생년월일 문자열에서 하이픈(-)을 제거하고 정수형(INT64)으로 변환
  FROM
    `3rd_week.mst_users`
)
, mst_users_with_age AS (
  SELECT
    *,
    -- 3. 기준일 정수에서 생년월일 정수를 뺀 후 10000으로 나누어 내림(FLOOR) 처리 (생일 경과 여부가 완벽히 반영된 만 나이 산출)
    FLOOR((int_specific_date - int_birth_date) / 10000) AS age
  FROM
    mst_users_with_int_birth_date
)
, mst_users_with_category AS (
  SELECT
    user_id, -- 사용자 식별자 ID
    sex,     -- 성별 ('M' 또는 'F')
    age,     -- 위에서 계산된 만 나이

    -- 4. 성별과 연령대 코드를 하나로 연결하여 마케팅 세분화 카테고리(예: M1, F2, C, T 등) 생성
    CONCAT(
      -- 4-1. 성별 접두사 부여: 20세 이상 성인인 경우에만 성별('M'/'F')을 붙이고, 미성년자(20세 미만)는 빈 문자열('') 처리
      CASE
        WHEN 20 <= age THEN sex
        ELSE ''
      END,

      -- 4-2. 연령대 구간 분류 코드 부여 (광고/마케팅 표준 타깃 구분)
      CASE
        WHEN age BETWEEN 4 AND 12  THEN 'C'  -- Child (어린이: 4~12세, 성별 무관 단독 'C')
        WHEN age BETWEEN 13 AND 19 THEN 'T'  -- Teen (청소년: 13~19세, 성별 무관 단독 'T')
        WHEN age BETWEEN 20 AND 34 THEN 'M1' -- 20~34세 청년층 (앞의 성별과 합쳐져 'MM1'/'FM1' 형태 생성)
        WHEN age BETWEEN 35 AND 49 THEN 'M2' -- 35~49세 중년층 (앞의 성별과 합쳐져 'MM2'/'FM2' 형태 생성)
        WHEN age >= 50             THEN 'M3' -- 50세 이상 장년·노년층 (앞의 성별과 합쳐져 'MM3'/'FM3' 형태 생성)
      END
    ) AS category

  FROM
    mst_users_with_age
)
SELECT *
FROM mst_users_with_category;
```
![img](../SQL_Master/image/Week3/11-6.png)

**결과 설명 / 주의점**
- 2017년 1월 1일 기준 만 나이를 계산한 후, 성별(`sex`)과 연령대(`age`)를 조합하여 세분화 카테고리(`category`)를 생성합니다.
- **성인(20세 이상)**: 성별 접두사(`M`/`F`)와 연령 구간 코드가 합쳐져 `MM1`, `FM1`, `MM2`, `FM2`, `MM3`, `FM3` 형태로 부여됩니다.
- **미성년자(20세 미만)**: 성별 구분 없이 어린이(`C`, 4~12세) 또는 청소년(`T`, 13~19세) 단일 코드로만 부여됩니다.
- 만 나이가 **4세 미만(0~3세)**인 영유아는 두 번째 `CASE` 조건식에 정의되어 있지 않아 카테고리가 `NULL`이 되어 `CONCAT`의 결과도 `NULL`이 됩니다.
- `birth_date`가 `NULL`인 데이터가 존재할 경우 나이 및 최종 카테고리 연산 결과가 모두 `NULL`이 되므로 사전 정제가 필요합니다.
- 최종 결과는 `user_id`, `sex`, `age`, `category` 컬럼을 포함합니다.

```sql
-- 연령별 구분의 사람 수를 계산하는 쿼리
WITH mst_users_with_age AS (
  SELECT
    *,
    FLOOR((int_specific_date - int_birth_date) / 10000) AS age
  FROM
    mst_users_with_int_birth_date
)
, mst_users_with_category AS (
  SELECT
    user_id,
    sex,
    age, 
    CONCAT(
      CASE
        WHEN 20 <= age THEN sex
        ELSE ''
      END,
      CASE
        WHEN age BETWEEN 4 AND 12  THEN 'C'
        WHEN age BETWEEN 13 AND 19 THEN 'T'
        WHEN age BETWEEN 20 AND 34 THEN 'M1'
        WHEN age BETWEEN 35 AND 49 THEN 'M2'
        WHEN age >= 50             THEN 'M3'
      END
    ) AS category

  FROM
    mst_users_with_age
)
SELECT
  category,
  COUNT(1) AS user_count
FROM
  mst_users_with_category
GROUP BY
  category;
```
![img](../SQL_Master/image/Week3/11-7.png)
**결과 설명 / 주의점**
- `category` 컬럼은 연령과 성별을 결합한 구분 코드입니다.
- `COUNT(*)`는 각 구분 코드별 사용자 수를 계산합니다.
- `WHERE category IS NOT NULL`로 NULL 값을 제외하여 계산합니다.

### 1-3 연령별 구분의 특징 추출하기

**핵심 원리 및 요약**
- `action_log` 테이블에서 구매 로그만 선택합니다.
- `mst_users_with_category` 테이블과 조인하여 구매한 상품의 카테고리를 집계합니다.
- `GROUP BY`를 사용하여 카테고리별 구매 수를 계산합니다.


```sql
WITH mst_users_with_int_birth_date AS (
  SELECT
    user_id,
    sex,
    birth_date,
    CAST(REPLACE(SUBSTR(birth_date, 1, 10), '-', '') AS INT64) AS int_birth_date
  FROM
    `sql-master-507514.3rd_week.mst_users`
),
mst_users_with_age AS (
  SELECT
    user_id,
    sex,
    birth_date,
    int_birth_date,
    FLOOR((20170101 - int_birth_date) / 10000) AS age
  FROM
    mst_users_with_int_birth_date
),
mst_users_with_category AS (
  SELECT
    user_id,
    sex,
    age,
    CONCAT(
      CASE
        WHEN 20 <= age THEN sex
        ELSE ''
      END,
      CASE
        WHEN age BETWEEN 4 AND 12 THEN 'C'
        WHEN age BETWEEN 13 AND 19 THEN 'T'
        WHEN age BETWEEN 20 AND 34 THEN 'M1'
        WHEN age BETWEEN 35 AND 49 THEN 'M2'
        WHEN age >= 50 THEN 'M3'
      END
    ) AS category
  FROM
    mst_users_with_age
)
SELECT
  p.category AS product_category,
  u.category AS user_category,
  COUNT(*) AS purchase_count
FROM
  `sql-master-507514.3rd_week.action_log` AS p
JOIN
  mst_users_with_category AS u
ON
  p.user_id = u.user_id
WHERE
  action = 'purchase'
GROUP BY
  p.category, u.category
ORDER BY
  p.category, u.category;
```

![img](../SQL_Master/image/Week3/11-8.png)

 **결과 설명 / 주의점**
- `product_category`와 `user_category`는 각각 구매한 상품의 카테고리와 사용자의 연령별 구분 코드입니다.
- `COUNT(*)`는 각 카테고리 내에서의 구매 수를 계산합니다.

### 1-4 사용자의 방문 빈도 집계하기

**핵심 원리 및 요약**
- 사용자 ID, 액션, 날짜가 기록되어 있는 `action_log` 테이블에 사용자 ID 별로 날짜에 DISTINCT를 적용하여 사용 일수를 집계
- 구성비와 구성비누계를 계산하여 사용 일수에 따른 사용자 수를 분석

```sql
-- 한 주에 며칠 사용되었는지를 집계하는 쿼리
WITH action_log_with_dt AS (
  SELECT
    user_id,
    action,
    SUBSTR(stamp, 1, 10) AS dt
  FROM
    `sql-master-507514.3rd_week.action_log`
),
action_day_count_per_user AS (
  SELECT
    user_id,
    COUNT(DISTINCT dt) AS action_day_count     -- 해당 기간 내 사용자가 실제로 활동(접속/액션)한 고유 일수 집계
  FROM
  FROM
    action_log_with_dt
  WHERE
    dt BETWEEN '2016-11-01' AND '2016-11-07'
  GROUP BY
    user_id
)
SELECT
  action_day_count,
  COUNT(DISTINCT user_id) AS user_count    -- 해당 활동 일수 구간에 속하는 고유 사용자 수 집계
FROM
  action_day_count_per_user
GROUP BY
  action_day_count
ORDER BY
  action_day_count;
```
![img](../SQL_Master/image/Week3/11-9.png)

```sql
WITH
-- 1. 타임스탬프에서 일자(YYYY-MM-DD) 추출
action_log_with_dt AS (
  SELECT
    user_id,
    action,
    SUBSTR(stamp, 1, 10) AS dt
  FROM
    `3rd_week.action_log`
),

-- 2. 대상 기간(11-01 ~ 11-07) 동안 각 사용자별 활동 일수(방문 빈도) 집계
action_day_count_per_user AS (
  SELECT
    user_id,
    COUNT(DISTINCT dt) AS action_day_count
  FROM
    action_log_with_dt
  WHERE
    dt BETWEEN '2016-11-01' AND '2016-11-07'
  GROUP BY
    user_id
)

-- 3. 활동 일수별 유저 수, 구성비(%), 구성비 누계(%) 산출
SELECT
  action_day_count,                                      -- 활동 일수 (계급)
  COUNT(DISTINCT user_id) AS user_count,                 -- 해당 활동 일수에 속한 고유 사용자 수

  -- 구성비: 100.0 * <해당 일수 유저 수> / <전체 유저 수>
  100.0
  * COUNT(DISTINCT user_id)
  / SUM(COUNT(DISTINCT user_id)) OVER() AS composition_ratio,

  -- 구성비 누계: 100.0 * <1일부터 현재 일수까지 누적 유저 수> / <전체 유저 수>
  100.0
  * SUM(COUNT(DISTINCT user_id)) OVER(
      ORDER BY action_day_count
      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )
  / SUM(COUNT(DISTINCT user_id)) OVER() AS cumulative_ratio

FROM
  action_day_count_per_user
GROUP BY
  action_day_count
ORDER BY
  action_day_count;
```
![img](../SQL_Master/image/Week3/11-10.png)

**결과 설명 / 주의점**
- 구성비는 각 사용 일수별 사용자 수를 전체 사용자 수에 대한 비율로 나타냅니다.
- 구성비누계는 각 사용 일수별 사용자 수를 전체 사용자 수에 대한 누적 비율로 나타냅니다.
- 결과는 사용 일수와 해당 사용 일수에 따른 사용자 수를 정렬하여 출력됩니다.
### 1-5 벤 다이어그램으로 사용자 액션 집계하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 1-6 Decile 분석을 사용해 사용자를 10단계 그룹으로 나누기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 1-7 RFM 분석으로 사용자를 3가지 관점의 그룹으로 나누기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->


### 🎉 수고하셨습니다.
