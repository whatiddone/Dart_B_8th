# Seoul Running Recommendation — Air Quality Pipeline

서울 러닝 코스 추천 프로젝트의 **대기질(Air Quality) 데이터 파이프라인** 정리 문서입니다.

이 파트의 최종 목표는 서울 25개 자치구의 과거 대기질 데이터를 이용해,

```text
러닝 월 + 출발 시간 + 코스가 지나는 자치구 + 구간 거리 + 러닝 페이스
```

를 입력했을 때 해당 코스의 **Route AirScore (0~100)** 를 계산하는 것입니다.

---

# 1. 전체 흐름

```text
서울시 대기질 원본 데이터 (2019~2026)
        ↓
연도별 파일 통합 / 컬럼 표준화 / 데이터 품질 점검
        ↓
시간별 · 일별 · 월별 EDA
        ↓
장기 추세 / 계절성 / 시간대 / 지역 차이 분석
        ↓
2019~2025 historical climatology 생성
        ↓
district × month × hour feature table
        ↓
오염물질별 percentile scaling
        ↓
AirScore v1
        ↓
여러 자치구를 지나는 코스의 시간가중 Route AirScore
```

---

# 2. 데이터 기간

## 분석 데이터

```text
2019 ~ 2026
```

EDA와 최근 추세 확인에는 2026년 데이터도 포함합니다.

## AirScore baseline

```text
2019 ~ 2025
```

2026년은 아직 완전연도가 아니므로 historical baseline에는 포함하지 않습니다.

즉:

```text
2019~2025
→ historical climatology / percentile / AirScore baseline

2026
→ 최근 추세 및 drift 확인
```

---

# 3. 분석 대상 오염물질

| 내부 코드 | 오염물질 | 단위 | AirScore 사용 |
|---|---|---:|---|
| `pm10_1h` | 미세먼지 1시간 | ㎍/㎥ | O |
| `pm10_24h` | 미세먼지 24시간 | ㎍/㎥ | X |
| `pm25` | 초미세먼지 | ㎍/㎥ | O |
| `o3` | 오존 | ppm | O |
| `no2` | 이산화질소 | ppm | O |
| `co` | 일산화탄소 | ppm | O |
| `so2` | 아황산가스 | ppm | O |

`pm10_24h`는 분석에는 사용하지만 최종 AirScore에는 넣지 않습니다.

`pm10_1h`과 `pm10_24h`를 동시에 사용하면 PM10 영향이 중복될 수 있기 때문입니다.

---

# 4. 폴더 구조

프로젝트 기본 경로:

```text
C:\Users\Luke\Desktop\DArtB\Toyproject
```

권장 구조:

```text
Toyproject/
│
├─ raw_air/
│  └─ yearly/
│     ├─ seoul_air_2019.csv
│     ├─ seoul_air_2020.csv
│     ├─ seoul_air_2021.csv
│     ├─ seoul_air_2022.csv
│     ├─ seoul_air_2023.csv
│     ├─ seoul_air_2024.csv
│     ├─ seoul_air_2025.csv
│     └─ seoul_air_2026.csv
│
└─ air_quality_analysis/
   ├─ air_hourly_clean_2019_2026.csv
   ├─ air_daily_mean_2019_2026.csv
   ├─ air_monthly_mean_2019_2026.csv
   ├─ air_quality_summary_2019_2026.csv
   ├─ air_year_station_summary_2019_2026.csv
   │
   ├─ conclusion/
   │  ├─ 01_year_coverage.csv
   │  ├─ 02_quality_summary.csv
   │  ├─ 03_long_term_trend.csv
   │  ├─ 04_2026_same_period_comparison.csv
   │  ├─ 05_seasonality_summary.csv
   │  ├─ 06_diurnal_summary.csv
   │  ├─ 07_regional_summary.csv
   │  ├─ 08_month_hour_station_variation.csv
   │  ├─ 09_final_summary.csv
   │  └─ 10_final_conclusion.md
   │
   ├─ feature_table/
   │  ├─ historical_air_climatology_full_2019_2025.csv
   │  ├─ historical_air_climatology_compact_2019_2025.csv
   │  └─ station_code_map.csv
   │
   └─ airscore_v1/
      ├─ example_route_airscore_summary.csv
      └─ example_route_airscore_chunks.csv
```

원본 데이터는 `raw_air/yearly/`에 보관하고, 분석 과정에서 새로 생성되는 데이터는 모두 `air_quality_analysis/` 아래에 저장합니다.

---

# 5. 실행 순서

아래 4개 노트북을 순서대로 실행합니다.

## 1) `seoul_air_trend_2019_2026_eda_revised.ipynb`

### 역할

2019~2026 연도별 원본 데이터를 불러와 하나의 표준 형식으로 통합하고 EDA를 수행합니다.

### 주요 처리

- 2019~2026 CSV 자동 탐색
- 서로 다른 컬럼명 표준화
- datetime 변환
- 25개 측정소 확인
- 결측값 / 음수 / 0값 / 중복 확인
- 시간별(hourly), 일별(daily), 월별(monthly) 집계
- 자치구 × 오염물질 시각화
- 24시간 패턴
- 월별 계절성
- 연도별 월 패턴
- 측정소 × 월 heatmap
- 서울 평균 대비 지역 편차

### 주요 생성 파일

```text
air_quality_analysis/
├─ air_hourly_clean_2019_2026.csv
├─ air_daily_mean_2019_2026.csv
├─ air_monthly_mean_2019_2026.csv
├─ air_quality_summary_2019_2026.csv
└─ air_year_station_summary_2019_2026.csv
```

---

## 2) `seoul_air_trend_2019_2026_analysis_revised.ipynb`

### 역할

EDA에서 확인한 패턴을 수치화하고 최종 해석을 정리합니다.

### 분석 순서

```text
A. 데이터 품질
B. 장기 추세
C. 계절성
D. 시간대 패턴
E. 지역 차이
F. 기존 가설 재검증
```

### 주요 분석 내용

#### 데이터 품질

- 연도별 25개 측정소 존재 여부
- 연도 × 측정소 관측 커버리지
- 결측률
- 0값 비율
- 2026년 데이터 범위

#### 장기 추세

- 2019~2025 완전연도 추세
- 오염물질별 연간 기울기
- 2026년과 과거 동일기간 비교

#### 계절성

- 1~12월 평균 패턴
- 최고 / 최저 월
- 월별 변동 크기

#### 시간대 패턴

- 0~23시 평균 profile
- 최고 / 최저 시간
- O3와 NO2의 시간대 차이

#### 지역 차이

```text
지역 상대편차
=
해당 측정소 월평균
-
같은 달 서울 전체 평균
```

#### 월 / 시간 / 지역 효과 비교

탐색적 비교를 위해 다음 CV를 계산합니다.

```text
month_cv
hour_cv
station_cv
```

이는 변수별 패턴의 상대적인 크기를 비교하기 위한 지표이며, 인과효과나 eta²와 동일한 개념은 아닙니다.

### 생성 파일

```text
air_quality_analysis/conclusion/
```

아래에 분석 결과 CSV와 최종 Markdown 결론이 저장됩니다.

---

## 3) `seoul_air_historical_climatology_2019_2025_revised.ipynb`

### 역할

분석 결과를 실제 추천 시스템에서 조회할 수 있는 historical feature table로 변환합니다.

핵심 key:

```text
district × month × hour
```

예:

```text
송파구 × 3월 × 16시
강남구 × 7월 × 06시
마포구 × 10월 × 20시
```

각 조합마다 2019~2025 과거 대기질의 통계값을 계산합니다.

### 저장 통계

각 오염물질에 대해:

```text
mean
median
p90
observation_count
```

를 저장합니다.

예:

```text
pm25_mean
pm25_median
pm25_p90
pm25_observation_count
```

### 의미

| 통계 | 의미 |
|---|---|
| `mean` | 장기 평균 |
| `median` | 평소 / 전형적인 historical 수준 |
| `p90` | historical 고농도 상황 |
| `observation_count` | 해당 통계의 표본 수 |

기본 AirScore 계산에는 `median`을 사용합니다.

`p90`은 보수적인 고농도 시나리오를 확인할 때 사용합니다.

---

# 6. 서울 평균 대비 Relative Feature

같은 시각의 서울 전체 평균과 비교한 지역 편차도 별도로 계산합니다.

```text
relative
=
해당 구 농도
-
같은 시각 서울 전체 평균 농도
```

예:

```text
송파구 PM2.5 = 20
같은 시각 서울 평균 PM2.5 = 27

relative = -7
```

해석:

```text
relative < 0
→ 같은 시각 서울 평균보다 낮은 편

relative > 0
→ 같은 시각 서울 평균보다 높은 편
```

Relative feature는 최종 AirScore에 직접 다시 더하지 않습니다.

이미 `district × month × hour`의 절대 농도 자체에 지역 차이가 들어 있으므로, relative를 다시 가중하면 지역 효과를 중복 반영할 가능성이 있기 때문입니다.

Relative feature는 다음 용도로 남겨둡니다.

- 결과 설명
- 비슷한 점수 코스 비교
- 이후 ML feature
- 서울 평균 대비 지역 특성 해석

---

# 7. Historical Climatology 최종 산출물

Full table:

```text
air_quality_analysis/
└─ feature_table/
   └─ historical_air_climatology_full_2019_2025.csv
```

구조:

```text
district_code
district
month
hour

pm10_1h_mean
pm10_1h_median
pm10_1h_p90
...

pm25_mean
pm25_median
pm25_p90
...

o3_mean
o3_median
o3_p90
...

relative features
observation counts
```

서울 25개 구 × 12개월 × 24시간이 모두 존재하면 이론상:

```text
25 × 12 × 24 = 7,200 rows
```

가 됩니다.

---

# 8. AirScore v1

`seoul_airscore_v1_route_scoring_revised.ipynb`에서 실제 AirScore를 계산합니다.

AirScore는 법적 AQI나 의료용 위험점수가 아닙니다.

**러닝 코스끼리 historical air condition을 비교하기 위한 프로젝트용 heuristic score**입니다.

---

# 9. AirScore Scaling

오염물질마다 단위와 값의 범위가 다르기 때문에 원 농도를 바로 더하지 않습니다.

2019~2025 서울 전체 hourly 데이터에서 각 값이 어느 percentile에 있는지 계산합니다.

```text
PollutantScore
=
100 - Historical Percentile
```

예:

```text
PM2.5 percentile = 75

PM2.5 Score
= 100 - 75
= 25
```

반대로:

```text
PM2.5 percentile = 20

PM2.5 Score
= 100 - 20
= 80
```

따라서:

```text
Score ↑
→ 과거 서울 데이터 기준 상대적으로 낮은 오염 농도

Score ↓
→ 과거 서울 데이터 기준 상대적으로 높은 오염 농도
```

---

# 10. AirScore v1 고정 가중치

현재 프로젝트에서는 다음 가중치를 사용합니다.

| 오염물질 | 가중치 |
|---|---:|
| PM2.5 | 40% |
| O3 | 25% |
| NO2 | 15% |
| PM10 | 10% |
| CO | 5% |
| SO2 | 5% |
| **합계** | **100%** |

공식:

```text
AirScore
=
0.40 × PM2.5 Score
+ 0.25 × O3 Score
+ 0.15 × NO2 Score
+ 0.10 × PM10 Score
+ 0.05 × CO Score
+ 0.05 × SO2 Score
```

이 비율은 특정 논문에서 그대로 가져온 공식이 아니라, 러닝 추천용으로 설계한 **AirScore v1 heuristic weighting**입니다.

향후 검증 결과에 따라 가중치는 조정할 수 있습니다.

---

# 11. AirScore는 '구별 고정 점수'가 아님

잘못된 형태:

```text
강남구 = 70점
송파구 = 80점
```

현재 구조:

```text
강남구 × 3월 × 16시
강남구 × 7월 × 06시
송파구 × 3월 × 16시
송파구 × 7월 × 06시
```

즉 같은 자치구라도 **월과 시간에 따라 점수가 달라집니다.**

---

# 12. 여러 자치구를 지나는 러닝 코스

예:

```text
3월
16:00 출발
pace = 6 min/km

강남구 4 km
송파구 6 km
```

러닝 코스를 구간별로 나눠 실제 예상 통과 시간을 계산합니다.

예를 들어 한 시간 경계를 넘는다면:

```text
16:00~16:24
강남구
→ 강남구 × 3월 × 16시

16:24~17:00
송파구
→ 송파구 × 3월 × 16시

17:00 이후
송파구
→ 송파구 × 3월 × 17시
```

처럼 자동으로 분리합니다.

---

# 13. Route AirScore

각 시간 chunk의 AirScore를 구한 뒤 실제 노출 시간으로 가중평균합니다.

```text
Route AirScore
=
Σ(chunk AirScore × chunk minutes)
/
Σ(chunk minutes)
```

거리보다 **실제 달리는 시간**을 가중치로 사용합니다.

페이스가 일정하다면 거리 가중과 거의 동일하지만, 구간별 페이스가 다를 때는 시간 가중이 실제 노출 개념에 더 적합합니다.

---

# 14. Route AirScore 사용 예

```python
ROUTE_SEGMENTS = [
    {
        "district": "강남구",
        "distance_km": 4.0
    },
    {
        "district": "송파구",
        "distance_km": 6.0
    }
]

result = run_airscore(
    route_segments=ROUTE_SEGMENTS,
    month=3,
    start_hour=16,
    start_minute=0,
    pace_min_per_km=6.0,
    statistic="median"
)
```

구간별 pace가 다르면 각 segment에 따로 넣을 수 있습니다.

```python
ROUTE_SEGMENTS = [
    {
        "district": "강남구",
        "distance_km": 4.0,
        "pace_min_per_km": 6.0
    },
    {
        "district": "송파구",
        "distance_km": 6.0,
        "pace_min_per_km": 6.5
    }
]
```

---

# 15. Median / P90 사용

기본:

```python
statistic="median"
```

의미:

```text
평소 / 전형적인 historical air condition
```

보수적 시나리오:

```python
statistic="p90"
```

의미:

```text
historical 고농도 조건을 가정한 시나리오
```

---

# 16. AirScore 해석

현재 UI용 설명 구간:

| AirScore | 설명 |
|---:|---|
| 80~100 | historical 기준 매우 양호 |
| 60~80 | 비교적 양호 |
| 40~60 | 중간 수준 |
| 20~40 | 비교적 불리 |
| 0~20 | historical 기준 매우 불리 |

주의:

이 구간은 법적 대기질 등급이나 건강 안전 기준이 아닙니다.

프로젝트 내 상대 비교를 쉽게 하기 위한 heuristic label입니다.

---

# 17. 현재 대기질 파트에서 완료된 것

현재까지 완료된 범위:

```text
[완료] 2019~2026 원본 데이터 통합
[완료] 25개 측정소 기준 표준화
[완료] 데이터 품질 점검
[완료] 시간별 / 일별 / 월별 EDA
[완료] 장기 추세 분석
[완료] 월별 계절성 분석
[완료] 24시간 패턴 분석
[완료] 지역 차이 분석
[완료] 서울 평균 대비 relative feature
[완료] 2019~2025 climatology
[완료] district × month × hour feature table
[완료] empirical percentile scaling
[완료] AirScore v1
[완료] multi-gu route 시간가중 Route AirScore
```

---

# 18. 아직 하지 않은 것

대기질 자체 분석은 거의 완료되었지만 최종 러닝 추천 시스템은 아직 완성되지 않았습니다.

다음 단계에서는:

```text
Route Feature(경사도, 인구밀도, 수변, 녹지 등)
+
AirScore
+
기상상황
```

를 하나의 추천 데이터셋으로 결합해야 합니다.

예:

```text
route_id
distance
elevation
green_ratio
shade_ratio
surface
...

AirScore

temperature
humidity
rain
wind
solar
visibility
...
```

이후 최종 추천식 또는 ranking logic을 정의합니다.

---

# 19. 기상 데이터와의 관계

대기질과 기상은 서로 독립적인 입력 feature로 사용합니다.

```text
Air
→ 자치구별 공간 차이 존재

Weather
→ 현재 프로젝트에서는 서울 ASOS 108 기준
   서울 공통 기상값
```

따라서 같은 시각의 코스 간 차이는:

```text
대기질
→ 코스가 지나는 자치구에 따라 차이가 날 수 있음

기상
→ 같은 시각 서울 전체에 동일한 기본값
→ route 특성과 interaction으로 차이를 만듦
```

예:

```text
temperature × shade
solar radiation × shade
rain × surface
wind × river
```

---

# 20. 핵심 설계 결정 요약

## Air historical baseline

```text
2019~2025
```

## EDA 범위

```text
2019~2026
```

## Air feature key

```text
district × month × hour
```

## 대표 historical statistic

```text
median
```

## 보수적 scenario

```text
p90
```

## Scaling

```text
2019~2025 서울 전체 empirical percentile
```

## Final AirScore direction

```text
0   = historical 기준 매우 불리
100 = historical 기준 매우 양호
```

## Multi-gu route aggregation

```text
시간가중 평균
```

---

# 21. 빠른 실행 순서

처음 프로젝트를 받았다면 아래 순서대로 실행하면 됩니다.

```text
1.
seoul_air_trend_2019_2026_eda_revised.ipynb

        ↓ 생성

air_hourly_clean_2019_2026.csv
air_daily_mean_2019_2026.csv
air_monthly_mean_2019_2026.csv
...

2.
seoul_air_trend_2019_2026_analysis_revised.ipynb

        ↓ 생성

conclusion/*.csv
10_final_conclusion.md

3.
seoul_air_historical_climatology_2019_2025_revised.ipynb

        ↓ 생성

feature_table/
historical_air_climatology_full_2019_2025.csv

4.
seoul_airscore_v1_route_scoring_revised.ipynb

        ↓

Route AirScore 계산
```

---

# 22. 주의사항

### 2026년

2026년 데이터는 EDA와 최근 추세에는 사용하지만 historical baseline에는 넣지 않습니다.

### 0값

원본의 `0`이 실제 측정값인지 결측/장비 상태를 의미하는지 확정되지 않은 경우가 있으므로 임의로 제거하지 않습니다.

### AirScore

AirScore는:

- 법적 AQI가 아님
- 의료 진단 또는 안전 판정 점수가 아님
- 실시간 예보가 아님
- 2019~2025 서울 historical data를 이용한 러닝 코스 상대 비교 점수임

### 가중치

현재 가중치는 AirScore v1의 고정 heuristic입니다.

```text
PM2.5 40%
O3    25%
NO2   15%
PM10  10%
CO     5%
SO2    5%
```

향후 외부 검증, 실제 사용자 피드백, 모델 성능 검증이 가능해지면 변경할 수 있습니다.

---

# 23. 최종 개념

현재 대기질 파이프라인은 다음 질문에 답하기 위한 구조입니다.

> "이 사람이 이 달, 이 시간에 이 코스를 달린다면, 과거 서울 대기질 패턴을 기준으로 이 코스의 대기질 조건은 상대적으로 어느 정도인가?"

최종적으로:

```text
사용자 러닝 조건
        ↓
month / start time / pace
        ↓
코스별 district segment
        ↓
district × month × hour climatology
        ↓
pollutant percentile score
        ↓
AirScore v1
        ↓
시간가중 Route AirScore
```

를 계산합니다.

이 Route AirScore가 이후 **Route Feature + Weather Feature**와 함께 최종 러닝 추천 점수에 들어갑니다.
