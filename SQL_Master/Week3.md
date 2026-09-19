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

- 액션 로그 테이블을 따로 만들고 내부에 내용을 적으면 별도의 JOIN과 UNION 없이 데이터를 다룰 수 있음
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
#### 1-1-2 로그인 사용자와 비로그인 사용자를 구분해서 집계하기
```sql
-- 로그인 상태를 판별하는 쿼리
WITH
action_log_with_status AS (
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
```sql
-- 로그인 상태에 따라 액션 수 등을 따로 집계하는 쿼리
WITH
-- 1. 로그인 여부 판정 플래그 추가
action_log_with_status AS (
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

-- BigQuery 표준 SQL은 ROLLUP(컬럼1, 컬럼2) 문법을 사용합니다.
GROUP BY ROLLUP(action, login_status)                                 -- (action, status)별 상세 -> action별 소계 -> 전체 총계 생성
ORDER BY
  action, login_status;
```
![img](../SQL_Master/image/Week3/11-3.png)

- 로그 정보의 user_id 정보를 기반으로 집계한 데이터이므로, 비로그인 사용자가 로그인하면 
각각의 액션에 1 씩 추가됨. 

#### 1-1-3 회원과 비회원을 구분해서 집계하기
### 1-2 연령별 구분 집계하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 1-3 연령별 구분의 특징 추출하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->
 
### 1-4 사용자의 방문 빈도 집계하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
여기에 코드를 적어주세요.
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

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
