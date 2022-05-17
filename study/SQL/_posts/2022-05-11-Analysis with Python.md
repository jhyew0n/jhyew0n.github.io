---
layout: post
title: 소비자 데이터를 이용한 고객 분석 with SQL
description: >
  데이터 분석 스터디 실습, DACON
sitemap: false
hide_last_modified: true
hide_description: true
categories:
  - study
  - sql
---

# 소비자 데이터 기반 소비 예측 with SQL

* toc
{:toc .large-only}


[ 👉 DACON : 소비자 데이터 기반 소비 예측 경진대회 바로가기](https://dacon.io/competitions/official/235893/overview/description)
<br>

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 🔎  지난 EDA로 궁금한 것 </span>

<b> 1. 주 타겟층은 누구? </b>

  - 주 타겟 성별+연령
  - 구매횟수가 많은 사람들은?
  - 1회당 구매 액수가 많은 사람들은?

<b> 2. 프로모션의 효과 </b>

  - 캠페인을 n번째 수용한 사람들의 특징
  - 캠페인을 한번도 수용하지 않은 사람들의 최근 구매일과 구매횟수
  - 할인된 구매가 많은 사람들의 특징과 소비
  - 결혼 상태와 교육 수준이 target에 미치는 영향

<b> 3. 구매 방식별 타겟 맞춤형 전략 </b>

  - 구매 빈도 제일 높은 것끼리 묶어 비교해보기.
  - 카탈로그를 주로 이용하는 사람들의 특징
  - 자녀가 있는/많은 사람들의 구매 경향은?

<b> 4. 웹 방문 빈도수가 오히려 target을 떨어뜨리는 이유? </b>

  - 누가 web 방문 빈도가 높은가?
  - web 방문 빈도가 높은 사람들의 성별+연령
  - web 방문 빈도가 높은 사람들의 구매 패턴

---

## 1. 주 타겟층 분석

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 1) 주 타겟층의 성별 + 연령 </span>

```sql
WITH
 users_with_age AS (
  SELECT
    *
    , 2023 - Year_Birth AS age
  FROM train
)
, users_with_category AS (
  SELECT
    id
    , Marital_Status
    , age
    , CONCAT (
      Marital_Status
      , CASE
        WHEN age BETWEEN 20 AND 30 THEN '20대'
        WHEN age BETWEEN 30 AND 40 THEN '30대'
        WHEN age BETWEEN 40 AND 50 THEN '40대'
        WHEN age BETWEEN 50 AND 60 THEN '50대'
        WHEN age BETWEEN 60 AND 70 THEN '60대'
        WHEN age >=71 THEN '70대 이상'
      END
    ) AS category
  FROM
    users_with_age
)
SELECT
  , category
  , COUNT(1) AS user_count
FROM
  users_with_category
GROUP BY
  category
;

```


<span style="color:#268C81; font-size:120%; font-weight:bold;"> 2) 구매 횟수 많은 유형 </span>

```sql
SELECT DISTINCT Education
FROM train;
```

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 3) 회당 구매 액수가 많은 유형 </span>

```sql
SELECT DISTINCT Education
FROM train;
```

## 2. 프로모션의 효과

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 1) 캠페인을 n번째 수용한 사람들의 특징 </span>

```sql
SELECT DISTINCT Education
FROM train;
```

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 2) 캠페인을 한번도 수용하지 않은 사람들의 최근 구매일과 구매횟수 </span>

```sql
SELECT DISTINCT Education
FROM train;
```

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 3) 할인된 구매가 많은 사람들의 특징과 소비 </span>

```sql
SELECT DISTINCT Education
FROM train;
```

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 4) 결혼 상태와 교육 수준이 target에 미치는 영향 </span>

```sql
SELECT DISTINCT Education
FROM train;
```

## 3. 구매 방식별 타겟 맞춤형 전략

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 1) 캠매 빈도 제일 높은 것끼리 묶어 비교해보기 </span>

```sql
SELECT DISTINCT Education
FROM train;
```

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 2) 카탈로그를 주로 이용하는 사람들의 특징 </span>

```sql
SELECT DISTINCT Education
FROM train;
```

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 3) 자녀가 있는/많은 사람들의 구매 경향은? </span>

```sql
SELECT DISTINCT Education
FROM train;
```

## 4. 웹 방문 빈도수와 소비량의 관계

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 1) Webpage 방문 빈도 높은 유형 </span>

```sql
SELECT DISTINCT Education
FROM train;
```

<span style="color:#268C81; font-size:120%; font-weight:bold;"> 2) Webpage 방문빈도가 높은 사람들의 구매 패턴 </span>

```sql
SELECT DISTINCT Education
FROM train;
```
