---
layout: post
title: 소비자 데이터를 이용한 데이터 탐색 with SQL
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

## 1. 데이터 확인
<br>
### 1) DACON 제공 데이터 설명 확인

|      col_names      | dtype   |                             description                            |
|:-------------------:|:---------:|:------------------------------------------------------------------|
|          id         | int64   |                             샘플 아이디                            |
|      Year_Birth     | **int64**   |                            고객 생년월일                           |
|      Education      | object(범주형)  |                              고객 학력                             |
|    Marital_status   | object(범주형)  |                           고객 결혼 상태                           |
|        Income       | float64 |                         고객 연간 가구 소득                        |
|       Kidhome       | int64   |                         고객 가구의 자녀 수                        |
|       Teenhome      | int64   |                        고객 가구의 청소년 수                       |
|     Dt_Customer     | **object**  |                      고객이 회사에 등록한 날짜                     |
|       Recency       | int64   |                    고객의 마지막 구매 이후 일수                    |
|  NumDealsPurchases  | int64   |                          할인된 구매 횟수                          |
|   NumWebPurchases   | int64   |                   회사 웹사이트를 통한 구매 건수                   |
| NumCatalogPurchases | int64   |                      카탈로그를 사용한 구매 수                     |
|   NumStorePuchases  | int64   |                      매장에서 직접 구매한 횟수                     |
|  NumWebVisitsMonth  | int64   |                   지난 달 회사 웹사이트 방문 횟수                  |
|     AcceptedCmp1    | int64   | 고객이 첫 번째 캠페인에서 제안을 수락한 경우 1, 그렇지 않은 경우 0 |
|     AcceptedCmp2    | int64   | 고객이 두 번째 캠페인에서 제안을 수락한 경우 1, 그렇지 않은 경우 0 |
|     AcceptedCmp3    | int64   | 고객이 세 번째 캠페인에서 제안을 수락한 경우 1, 그렇지 않은 경우 0 |
|     AcceptedCmp4    | int64   | 고객이 네 번째 캠페인에서 제안을 수락한 경우 1, 그렇지 않은 경우 0 |
|     AcceptedCmp5    | int64   |  고객이 5번째 캠페인에서 제안을 수락한 경우 1, 그렇지 않은 경우 0  |
|       Complain      | int64   |    고객이 지난 2년 동안 불만을 제기한 경우 1, 그렇지 않은 경우 0   |
|       Response      | int64   |  고객이 마지막 캠페인에서 제안을 수락한 경우 1, 그렇지 않은 경우 0 |
|        target       | int64   |                        고객의 제품 총 소비량                       |
{:.smaller}
<br><br>

### 2) Python을 이용해 csv 파일을 Database에 올려줌
<br>
<details>
<summary>python 코드 확인</summary>
<div markdown="1">

    ```python

    # 라이브러리 설치 (주피터 노트북)
    !pip install pandas
    !pip install sqlalchemy
    !pip install pymysql

    # 라이브러리 임포트
    import pandas as pd
    import pymysql
    from sqlalchemy import create_engine

    # pymysql 세팅
    db = pymysql.connect(user = 'root', host = 'localhost', passwd = '비밀번호', port = 3306, db = '데이터베이스이름')
    cursor = db.cursor()

    # csv파일 불러오기
    df = pd.read_csv('파일명.csv',encoding = 'utf-8-sig')
    df.columns = ['테이블과 동일한 컬럼명 사용하도록 수정']

    # sqlalchemy를 사용해 원하는 database에 csv파일 저장
    engine = create_engine("mysql+pymysql://유저이름:"+"비밀번호"+"@호스트주소:포트숫자/데이터베이스이름?charset=utf8", encoding = "utf-8")
    conn = engine.connect()
    df.to_sql(name = "테이블이름", con = engine, if_exist = 'append', index = False)
    conn.close()

    # 저장 확인
    sql = "select * from 테이블이름 limit 5"
    pd.read_sql(sql,db)
    ```

</div>
</details>
<br>


[코드 참고 블로그](https://velog.io/@actpjk/21.2.14-pandas-pymysql-sqlalchemy-csv%ED%8C%8C%EC%9D%BC%EC%9D%84-MySQL%EC%97%90-%EC%A0%80%EC%9E%A5%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)

<br>  

```sql
SELECT *
FROM train
```
<br>


| **id** | **Year_Birth** | **Education** | **Marital_Status** | **Income** | **Kidhome** | **Teenhome** | **Dt_Customer** | **Recency** | **NumDealsPurchases** | **NumWebPurchases** | **NumCatalogPurchases** | **NumStorePurchases** | **NumWebVisitsMonth** | **AcceptedCmp3** | **AcceptedCmp4** | **AcceptedCmp5** | **AcceptedCmp1** | **AcceptedCmp2** | **Complain** | **Response** | **target** | |
|-------:|---------------:|--------------:|-------------------:|-----------:|------------:|-------------:|----------------:|------------:|----------------------:|--------------------:|------------------------:|----------------------:|----------------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-------------:|-------------:|-----------:|-----------:|
| 0      | 1974           | Master        | Together           | 46014      | 1           | 1            | 21-01-2013      | 21          | 10                    | 7                   | 1                       | 8                     | 7                     | 0                | 0                | 0                | 0                | 0                | 0            | 0            | 541        |            |
| 1      | 1962           | Graduation    | Single             | 76624      | 0           | 1            | 24-05-2014      | 68          | 1                     | 5                   | 10                      | 7                     | 1                     | 1                | 0                | 0                | 0                | 0                | 0            | 0            | 899        |            |
| 2      | 1951           | Graduation    | Married            | 75903      | 0           | 1            | 08-04-2013      | 50          | 2                     | 6                   | 6                       | 9                     | 3                     | 0                | 0                | 0                | 0                | 0                | 0            | 0            | 901        |            |
| 3      | 1974           | Basic         | Married            | 18393      | 1           | 0            | 29-03-2014      | 2           | 2                     | 3                   | 0                       | 3                     | 8                     | 0                | 0                | 0                | 0                | 0                | 0            | 0            | 50         |            |
| 4      | 1946           | PhD           | Together           | 64014      | 2           | 1            | 10-06-2014      | 56          | 7                     | 8                   | 2                       | 5                     | 7                     | 0                | 0                | 0                | 1                | 0                | 0            | 0            | 444        |            |
{:.smaller}
{:.scroll-table}
<br><br>


## 3) 테이블 구조 확인
<br>

```sql
DESC train
```
<br>

|      **Field**      | **Type** | **Null** | **Default** |
|:-------------------:|:--------:|:--------:|:-----------:|
| id                  |  bigint  |    YES   |     NULL    |
| Year_Birth          |  bigint  |    YES   |     NULL    |
| Education           |   text   |    YES   |     NULL    |
| Marital_Status      |   text   |    YES   |     NULL    |
| Income              |  double  |    YES   |     NULL    |
| Kidhome             |  bigint  |    YES   |     NULL    |
| Teenhome            |  bigint  |    YES   |     NULL    |
| Dt_Customer         |   text   |    YES   |     NULL    |
| Recency             |  bigint  |    YES   |     NULL    |
| NumDealsPurchases   |  bigint  |    YES   |     NULL    |
| NumWebPurchases     |  bigint  |    YES   |     NULL    |
| NumCatalogPurchases |  bigint  |    YES   |     NULL    |
| NumStorePurchases   |  bigint  |    YES   |     NULL    |
| NumWebVisitsMonth   |  bigint  |    YES   |     NULL    |
| AcceptedCmp3        |  bigint  |    YES   |     NULL    |
| AcceptedCmp4        |  bigint  |    YES   |     NULL    |
| AcceptedCmp5        |  bigint  |    YES   |     NULL    |
| AcceptedCmp1        |  bigint  |    YES   |     NULL    |
| AcceptedCmp2        |  bigint  |    YES   |     NULL    |
| Complain            |  bigint  |    YES   |     NULL    |
| Response            |  bigint  |    YES   |     NULL    |
| target              |  bigint  |    YES   |     NULL    |
{:.smaller}
{:.scroll-table}
<br><br>


## 4) 데이터 크기 확인
<br>


```sql
SELECT count(*)
FROM train
```
<br>

| count(*) |
|:--------:|
| 1108     |

<br>

## 2. 데이터 탐색

- **카테고리형 변수** : ['Education', 'Marital_Status', 'Kidhome', 'Teenhome'
               'AcceptedCmp1', 'AcceptedCmp2', 'AcceptedCmp3', 'AcceptedCmp4', 'AcceptedCmp5',
               'Complain', 'Response']

- **수치형 변수** : ['Year_Birth', 'Income', 'Recency', 'NumDealsPurchases', 'NumWebPurchases',
       'NumCatalogPurchases', 'NumStorePurchases', 'NumWebVisitsMonth']

<br>

### 2-1) 카테고리형 변수 탐색

<br><br>

<span style="color:green; font-size:110%; font-weight:bold;"> Education 데이터 분포</span>


<br>

```sql
SELECT DISTINCT Education
FROM train;
```
> 'Master', 'Graduation', 'Basic', 'PhD', '2n Cycle'
<br>
찾아보니
Basic(중등 졸업), Graduation(학사), Master(석사), PhD(박사), 2n Cycle(?)
<br>
2n Cycle은  뭔지 모르겠다.

<br>

```sql
SELECT Education, count(id), count(id) / sum(count(*)) OVER() AS RAT
FROM train
GROUP BY Education;
```
<br>
|  Education | count(id) | RAT     |
|:----------:|-----------|---------|
| Master     | 173       | 15.6137 |
| Graduation | 570       | 51.4440 |
| Basic      | 22        | 1.9856  |
| PhD        | 254       | 22.9242 |
| 2n Cycle   | 89        | 8.0325  |
{:.smaller}

<br><br>

<span style="color:green; font-size:110%; font-weight:bold;"> Marital_Status 데이터 분포</span>

<br>

```sql
SELECT DISTINCT Marital_Status
FROM train;
```
> 'Together', 'Single', 'Married', 'Widow', 'Divorced', 'Alone', 'YOLO', 'Absurd'

<br>

Together(동거), Widow(과부), Divorced(이혼), YOLO, Absurd(having no rational or orderly relationship to hyman life)

<br>

Absurd가 뭔지 한참 찾았다...

<br>


| Education | count(id) | RAT     |
|:---------:|-----------|---------|
| Together  | 296       | 26.7148 |
| Single    | 234       | 21.1191 |
| Married   | 415       | 37.4549 |
| Widow     | 39        | 3.5199  |
| Divorced  | 120       | 10.8303 |
| Alone     | 2         | 0.1805  |
| YOLO      | 1         | 0.0903  |
| Absurd    | 1         | 0.0903  |
{:.smaller}

<br>
Alone, YOLO, Absurd는 값이 너무 적어서 Single에 포함시킨다.
<br>

```sql
UPDATE train
SET Marital_Status = "Single"
WHERE Marital_Status = "YOLO"
	or Marital_Status = "Absurd"
	or Marital_Status = "Alone" ;
```
<br>

UPDATE 구문으로 데이터를 바꾸고 다시 확인해봤다. 이번에는 소비량 합계도 함께 출력해봤다.
UPDATA 구문처럼 원본 테이블을 수정하는 경우에는
<br>

> BEGIN tran
> COMMIT tran

<br>
혹시 모를 상황을 대비해 트랜잭션을 설정하고 commit하는게 좋은데,
mysql은 기본적으로 자동 commit이 설정되어 있는 것 같다. 알아서 반영됐다.


<br>

```sql
SELECT Marital_Status, count(id), count(id)*100 / sum(count(*)) OVER() AS RAT, sum(target)
FROM train
GROUP BY Marital_Status;
```
<br>

| Education | count(id) |   RAT   | sum(target) |
|:---------:|:---------:|:-------:|:-----------:|
| Together  |       296 | 26.7148 |      176101 |
| Single    |       238 | 21.4801 |      148846 |
| Married   |       415 | 37.4549 |      251653 |
| Widow     |        39 |  3.5199 |       28150 |
| Divorced  |       120 | 10.8303 |       79021 |
{:.smaller}

<br>

<span style="color:green; font-size:110%; font-weight:bold;"> Kidhome, Teenhome 데이터 분포</span>

<br>

다음으로
<br>
Kidhome, Teenhome은 수치형 변수로 보는게 맞을 것 같은데 값의 범위가 좁아서 카테고리형으로 분류한 것 같다.
이거를 카테고리형으로 바꾸려면 3명 이상 이라는 옵션이 있어야 할 것 같은데, 없다.

<br>

```sql
SELECT Kidhome, count(id), count(id)*100 / sum(count(*)) OVER() AS RAT, sum(target)
FROM train
GROUP BY Kidhome;
```

> 0, 1, 2

<br>

| Kidhome | count(id) |   RAT   | sum(target) |
|:-------:|:---------:|:-------:|:-----------:|
|    0    |       661 | 59.6570 |      590377 |
|    1    |       418 | 37.7256 |       89295 |
|    2    |        29 |  2.6173 |        4099 |
{:.smaller}
<br>

```sql
SELECT Teenhome, count(id), count(id)*100 / sum(count(*)) OVER() AS RAT, sum(target)
FROM train
GROUP BY Kidhome;
```

> 0, 1, 2

<br>

| Teenhome | count(id) |   RAT   | sum(target) |
|:--------:|:---------:|:-------:|:-----------:|
|     0    |       571 | 51.5343 |      393832 |
|     1    |       507 | 45.7581 |      270224 |
|     2    |        30 |  2.7076 |       19715 |
{:.smaller}
<br>

*참고로 Kid는 성인 이전의 아이들을 포괄적으로 의미하고 Teen은 그 중 청소년기의 아이들만 지칭하는 것 같은데
왜 Kidhome 1,2 < Teenhome 1,2 일까...?*
<br><br>


<span style="color:green; font-size:110%; font-weight:bold;"> 그 외 데이터 분포</span>
<br>

나머지는 0 또는 1 인 변수 : 'AcceptedCmp1', 'AcceptedCmp2', 'AcceptedCmp3', 'AcceptedCmp4', 'AcceptedCmp5',
                      'Complain', 'Response

<br>

```sql
SELECT
  sum(AcceptedCmp1),
  sum(AcceptedCmp2),
  sum(AcceptedCmp3),
  sum(AcceptedCmp4),
  sum(AcceptedCmp5),
  sum(Complain),
  sum(Response)
FROM train;
```

<br>

| sum(AcceptedCmp1) | sum(AcceptedCmp2) | sum(AcceptedCmp3) | sum(AcceptedCmp4) | sum(AcceptedCmp5) | sum(Complain) | sum(Response) |
|:-----------------:|:-----------------:|:-----------------:|-------------------|-------------------|---------------|---------------|
|         76        |         17        |         77        |         95        |         80        |       10      |      157      |
{:.smaller}

<br>

```sql
SELECT
  sum(AcceptedCmp1)*100/1108,
  sum(AcceptedCmp2)*100/1108,
  sum(AcceptedCmp3)*100/1108,
  sum(AcceptedCmp4)*100/1108,
  sum(AcceptedCmp5)*100/1108,
  sum(Complain)*100/1108,
  sum(Response)*100/1108
FROM train;
```

<br>

| sum(AcceptedCmp1) | sum(AcceptedCmp2) | sum(AcceptedCmp3) | sum(AcceptedCmp4) | sum(AcceptedCmp5) | sum(Complain) | sum(Response) |
|:---:|---|---|---|---|---|---|
| 6.8592 | 1.5343 | 6.9495 | 8.5740 | 7.2202 | 0.9025 | 14.1697 |
{:.smaller}

<br>

두번째 캠페인 수용률이 유독 낮다.
그 외에는 뚜렷한 경향성은 없는 것 같다.
<br><br>



### 2-2) 수치형 변수 탐색

<br>

<span style="color:green; font-size:110%; font-weight:bold;"> Year_Birth </span>

<br>

연령대를 구간으로 나누어 세어보기.

일단 범위를 알아보자


```sql
SELECT min(year_birth) , max(year_birth)
FROM train
```

> 1893 \<= age \<= 1996

<br>

```sql
WITH users_with_age AS (
  SELECT
    *
    , 2023 - year_birth AS age
  FROM train
)
, users_with_age_range AS (
  SELECT
    id
    , age
    , CASE
        WHEN age BETWEEN 20 AND 30 THEN '20대'
        WHEN age BETWEEN 30 AND 40 THEN '30대'
        WHEN age BETWEEN 40 AND 50 THEN '40대'
        WHEN age BETWEEN 50 AND 60 THEN '50대'
        WHEN age >= 61 THEN '60대 이상'
      END
     AS age_range
  FROM
    users_with_age
)

SELECT
  age_range
  , COUNT(1) AS user_count
  , COUNT(1)*100/1108 AS user_ratio
FROM
  users_with_age_range
GROUP BY
  age_range
;
```
<br>

| age_range | user_count | user_ratio |
|:---:|---|---|
| 60대 이상 | 347 | 31.3177 |
| 50대 | 311 | 28.0686 |
| 40대 | 296 | 26.7148 |
| 30대 | 147 | 13.2671 |
| 20대 | 7 | 0.6318 |
{:.smaller}

<br>

연령대가 확실히 높은 편.

<br>

<span style="color:green; font-size:110%; font-weight:bold;"> Income </span>

<br>
대략적인 소득수준을 파악하기 위해 고객을 5등분 하여 상위 20%, 40%, 60%, 80%, 100%로 나누어 소득 수준을 파악해보았다.
<br>

```sql
WITH users_with_decile AS (
  SELECT
    id
    , income
    , ntile(5) OVER (ORDER BY income DESC) AS decile
  FROM
    train
)
, decile_with_income AS (
  SELECT
    decile
    ,count(1) AS id_count
    , AVG(income) AS avg_amount
    , MIN(income) AS min_amount
    , MAX(income) AS max_amount
  FROM
    users_with_decile
  GROUP BY
    decile
)
SELECT *
FROM
  decile_with_income
;
```

<br>

| decile | id_count | avg(income) | min(income) | max(income) |
|:---:|---|---|---|---|
| 1 | 222 | 81502.17117117117 | 71488 | 162397 |
| 2 | 222 | 65154.792792792796 | 58607 | 71427 |
| 3 | 222 | 51822.63063063063 | 44802 | 58582 |
| 4 | 221 | 38516.61990950226 | 32880 | 44794 |
| 5 | 221 | 23191.647058823528 | 1730 | 32644 |
{:.smaller}

> 이렇게 보는 것보다 소득 구간을 직접 보는게 나으려나?


<br>
<span style="color:green; font-size:110%; font-weight:bold;"> Recency </span>
<br>

마지막 구매일 이후 일수

<br>

```sql
WITH users_with_decile_recency AS (
  SELECT
    id
    , recency
    , ntile(5) OVER (ORDER BY recency DESC) AS decile
  FROM
    train
)
, decile_with_recency AS (
  SELECT
    decile
    ,count(1) AS id_count
    , AVG(recency) AS avg_amount
    , MIN(recency) AS min_amount
    , MAX(recency) AS max_amount
  FROM
    users_with_decile_recency
  GROUP BY
    decile
)
SELECT *
FROM
  decile_with_recency
;
```

<br>

| decile | id_count | avg_amount | min_amount | max_amount |
|:---:|---|---|---|---|
| 1 | 222 | 89.5631 | 80 | 99 |
| 2 | 222 | 70.6802 | 60 | 80 |
| 3 | 222 | 51.1667 | 41 | 60 |
| 4 | 221 | 30.0045 | 20 | 41 |
| 5 | 221 | 9.0905 | 0 | 20 |
{:.smaller}

<br>

대충 기록이 100일 안에 구매 기록이 있는 사람들의 데이터인 것 같다.
일정하게 나뉜 걸 보니 이걸 기준으로 데이터를 추출한 거 같다.

<br>

<span style="color:green; font-size:110%; font-weight:bold;"> 그 외 데이터 분포 </span>

<br>

```sql
WITH sum_table AS (
SELECT
	SUM(SIGN(NumDealsPurchases)) AS Deals_sum,
	SUM(SIGN(NumWebPurchases)) AS Web_sum,
	SUM(SIGN(NumCatalogPurchases)) AS Catal_sum,
	SUM(SIGN(NumStorePurchases)) AS Store_sum
FROM train
)
SELECT
	Deals_sum*100/(Deals_sum+Web_sum+Catal_sum+Store_sum) AS Deals,
	Web_sum*100/(Deals_sum+Web_sum+Catal_sum+Store_sum) AS Web,
	Catal_sum*100/(Deals_sum+Web_sum+Catal_sum+Store_sum) AS Catal,
	Store_sum*100/(Deals_sum+Web_sum+Catal_sum+Store_sum)  AS Store
FROM sum_table ;
```

<br>


| Deals | Web | Catal | Store |
|---|---|---|---|
| 15.4728 | 27.6743 | 17.7949 | 39.0580 |
{:.smaller}

<br>

Store 구매 비율이 제일 높다.
Web , Catal, Deals 순.
그런데 생각해보니, 일부 인원이 일부 카테고리에서 구매양이 월등히 높으면 정보가 왜곡될 수 있을 것 같다.
그래서 사람 수대로 계산 해보기로 했다.

<br>


```sql
WITH sum_table AS (
SELECT
	SUM(SIGN(NumDealsPurchases)) AS Deals_sum,
	SUM(SIGN(NumWebPurchases)) AS Web_sum,
	SUM(SIGN(NumCatalogPurchases)) AS Catal_sum,
	SUM(SIGN(NumStorePurchases)) AS Store_sum
FROM train
)
SELECT
	Deals_sum*100/1108 AS Deals,
	Web_sum*100/1108 AS Web,
	Catal_sum*100/1108 AS Catal,
	Store_sum*100/1108  AS Store
FROM sum_table ;

```

<br>


| Deals | Web | Catal | Store |
|---|---|---|---|
| 97.5632 | 97.6534 | 74.8195 | 99.4585 |
{:.smaller}

<br>

사람수대로 보니 매장에서 대부분 구매 경험이 있다.
할인된 딜 구매, 웹 구매도 대부분이 경험이 있다.

<br>

혹시 아무데서도 구매 안한 사람이 있으려나...?

<br>

```sql
WITH sign_table AS (
SELECT *,
	SIGN(NumDealsPurchases)+SIGN(NumWebPurchases)+SIGN(NumCatalogPurchases)+SIGN(NumStorePurchases) AS sign_sum
FROM train
)
SELECT id, Year_Birth, Education, Marital_Status, Income, Kidhome, Teenhome, Dt_Customer, Recency, target,sign_sum
FROM sign_table
WHERE sign_sum <3
ORDER BY sign_sum;
```

<br>

| id | Year_Birth | Education | Marital_Status | Income | Kidhome | Teenhome | Dt_Customer | Recency | target | sign_sum |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 686 | 1975 | Graduation | Divorced | 153924 | 0 | 0 | 07-02-2014 | 81 | 6 | 0 |
| 1097 | 1965 | Graduation | Divorced | 4861 | 0 | 0 | 22-06-2014 | 20 | 6 | 0 |
| 91 | 1963 | PhD | Married | 4023 | 1 | 1 | 23-06-2014 | 29 | 9 | 1 |
| 182 | 1971 | Graduation | Divorced | 1730 | 0 | 0 | 18-05-2014 | 65 | 8 | 1 |
| 194 | 1976 | Graduation | Married | 5305 | 0 | 1 | 30-07-2013 | 12 | 32 | 1 |
| 246 | 1976 | PhD | Together | 162397 | 1 | 1 | 03-06-2013 | 31 | 107 | 1 |
| 297 | 1957 | PhD | Together | 6835 | 0 | 1 | 08-12-2012 | 76 | 137 | 1 |
| 892 | 1945 | PhD | Single | 113734 | 0 | 0 | 28-05-2014 | 9 | 277 | 1 |
| 98 | 1986 | Graduation | Single | 26816 | 0 | 0 | 17-08-2012 | 50 | 22 | 2 |
| 112 | 1965 | Graduation | Together | 29672 | 1 | 1 | 12-03-2013 | 6 | 25 | 2 |
| 140 | 1982 | Graduation | Together | 40479 | 1 | 0 | 17-08-2013 | 95 | 15 | 2 |
| 218 | 1960 | Master | Divorced | 39228 | 0 | 0 | 10-05-2013 | 1 | 20 | 2 |
| 275 | 1957 | Graduation | Together | 21994 | 0 | 1 | 24-12-2012 | 4 | 22 | 2 |
| 324 | 1950 | PhD | Married | 41145 | 1 | 1 | 08-02-2014 | 20 | 13 | 2 |
| 374 | 1975 | Graduation | Divorced | 36627 | 2 | 0 | 23-07-2013 | 78 | 16 | 2 |
| 450 | 1946 | Graduation | Married | 18100 | 0 | 0 | 06-08-2013 | 14 | 14 | 2 |
| 472 | 1985 | Graduation | Married | 19986 | 1 | 0 | 14-11-2013 | 74 | 22 | 2 |
| 616 | 1978 | Graduation | Single | 60199 | 1 | 2 | 12-09-2013 | 49 | 18 | 2 |
| 618 | 1975 | 2n Cycle | Married | 37284 | 1 | 1 | 29-03-2013 | 46 | 23 | 2 |
| 651 | 1983 | PhD | Married | 23536 | 1 | 0 | 04-06-2014 | 53 | 10 | 2 |
| 730 | 1971 | Master | Together | 157733 | 1 | 0 | 04-06-2013 | 37 | 59 | 2 |
| 811 | 1959 | Master | Married | 34242 | 0 | 1 | 23-03-2014 | 25 | 15 | 2 |
| 901 | 1985 | Basic | Single | 16581 | 0 | 0 | 12-01-2013 | 51 | 24 | 2 |
| 1073 | 1987 | Graduation | Married | 7500 | 0 | 0 | 09-01-2013 | 94 | 15 | 2 |
| 1082 | 1971 | Master | Married | 34109 | 0 | 1 | 06-11-2013 | 39 | 22 | 2 |
{:.smaller}
{:.scroll-table}

<br>



지난달에 web에 방문한 사람 중,
web 구매도 한 사람은..?

<br>

```sql
SELECT id, NumWebPurchases, NumWebVisitsMonth, Recency,
	ROUND(NumWebPurchases*100/NumWebVisitsMonth,2) AS purch_ratio
FROM train
WHERE NumWebVisitsMonth >0
	and Recency <32
ORDER BY purch_ratio ;
```

<br>

| id | NumWebPurchases | NumWebVisitsMonth | Recency | purch_ratio |
|:---:|:---:|:---:|:---:|:---:|
| 91 | 0 | 19 | 29 | 0.00 |
| 112 | 0 | 6 | 6 | 0.00 |
| 218 | 0 | 4 | 1 | 0.00 |
| 246 | 0 | 1 | 31 | 0.00 |
| 275 | 0 | 5 | 4 | 0.00 |
| 324 | 0 | 3 | 20 | 0.00 |
| 350 | 0 | 7 | 16 | 0.00 |
| 450 | 0 | 5 | 14 | 0.00 |
| 811 | 0 | 5 | 25 | 0.00 |
| 1097 | 0 | 14 | 20 | 0.00 |
| 194 | 1 | 13 | 12 | 7.69 |
| 19 | 1 | 8 | 24 | 12.50 |
| 39 | 1 | 8 | 24 | 12.50 |
| 93 | 1 | 8 | 13 | 12.50 |
| 240 | 1 | 8 | 13 | 12.50 |
| 309 | 1 | 8 | 13 | 12.50 |
| 330 | 1 | 8 | 8 | 12.50 |
| 369 | 1 | 8 | 26 | 12.50 |
| 557 | 1 | 8 | 18 | 12.50 |
| 709 | 1 | 8 | 2 | 12.50 |
| 881 | 1 | 8 | 18 | 12.50 |
| 916 | 1 | 8 | 24 | 12.50 |
| 1064 | 1 | 8 | 13 | 12.50 |
| 30 | 1 | 7 | 10 | 14.29 |
| 76 | 1 | 7 | 10 | 14.29 |
| 114 | 1 | 7 | 17 | 14.29 |
| 265 | 1 | 7 | 13 | 14.29 |
| 495 | 1 | 7 | 19 | 14.29 |
| 581 | 1 | 7 | 5 | 14.29 |
| 584 | 1 | 7 | 3 | 14.29 |
| 694 | 1 | 7 | 24 | 14.29 |
| 776 | 1 | 7 | 10 | 14.29 |
| 807 | 1 | 7 | 8 | 14.29 |
| 818 | 1 | 7 | 23 | 14.29 |
| 888 | 1 | 7 | 10 | 14.29 |
| 895 | 1 | 7 | 11 | 14.29 |
| 1026 | 1 | 7 | 25 | 14.29 |
| 1042 | 1 | 7 | 27 | 14.29 |
| 1069 | 1 | 7 | 2 | 14.29 |
| 1079 | 1 | 7 | 10 | 14.29 |
| 113 | 1 | 6 | 2 | 16.67 |
| 175 | 1 | 6 | 21 | 16.67 |
| 208 | 1 | 6 | 29 | 16.67 |
| 256 | 1 | 6 | 15 | 16.67 |
| 396 | 1 | 6 | 23 | 16.67 |
| 416 | 1 | 6 | 0 | 16.67 |
| 481 | 1 | 6 | 15 | 16.67 |
| 685 | 1 | 6 | 31 | 16.67 |
| 702 | 1 | 6 | 1 | 16.67 |
| 1105 | 1 | 6 | 4 | 16.67 |
| 592 | 1 | 5 | 28 | 20.00 |
| 247 | 2 | 9 | 24 | 22.22 |
| 1028 | 2 | 9 | 14 | 22.22 |
| 7 | 1 | 4 | 31 | 25.00 |
| 25 | 1 | 4 | 3 | 25.00 |
| 84 | 2 | 8 | 27 | 25.00 |
| 156 | 1 | 4 | 24 | 25.00 |
| 199 | 1 | 4 | 0 | 25.00 |
| 231 | 1 | 4 | 15 | 25.00 |
| 259 | 1 | 4 | 9 | 25.00 |
| 339 | 2 | 8 | 25 | 25.00 |
| 378 | 1 | 4 | 29 | 25.00 |
| 630 | 1 | 4 | 18 | 25.00 |
| 824 | 2 | 8 | 7 | 25.00 |
| 865 | 1 | 4 | 23 | 25.00 |
| 925 | 2 | 8 | 19 | 25.00 |
| 69 | 2 | 7 | 3 | 28.57 |
| 77 | 2 | 7 | 28 | 28.57 |
| 132 | 2 | 7 | 28 | 28.57 |
| 203 | 2 | 7 | 14 | 28.57 |
| 325 | 2 | 7 | 14 | 28.57 |
| 332 | 2 | 7 | 5 | 28.57 |
| 347 | 2 | 7 | 27 | 28.57 |
| 359 | 2 | 7 | 27 | 28.57 |
| 437 | 2 | 7 | 1 | 28.57 |
| 451 | 2 | 7 | 10 | 28.57 |
| 467 | 2 | 7 | 9 | 28.57 |
| 502 | 2 | 7 | 12 | 28.57 |
| 752 | 2 | 7 | 13 | 28.57 |
| 761 | 2 | 7 | 28 | 28.57 |
| 826 | 2 | 7 | 19 | 28.57 |
| 846 | 2 | 7 | 2 | 28.57 |
| 969 | 2 | 7 | 25 | 28.57 |
| 23 | 3 | 9 | 19 | 33.33 |
| 31 | 2 | 6 | 11 | 33.33 |
| 33 | 2 | 6 | 30 | 33.33 |
| 68 | 3 | 9 | 11 | 33.33 |
| 71 | 2 | 6 | 28 | 33.33 |
| 166 | 2 | 6 | 11 | 33.33 |
| 245 | 3 | 9 | 15 | 33.33 |
| 474 | 2 | 6 | 11 | 33.33 |
| 486 | 1 | 3 | 9 | 33.33 |
| 586 | 2 | 6 | 3 | 33.33 |
| 648 | 2 | 6 | 11 | 33.33 |
| 650 | 2 | 6 | 12 | 33.33 |
| 674 | 1 | 3 | 31 | 33.33 |
| 713 | 1 | 3 | 7 | 33.33 |
| 751 | 3 | 9 | 24 | 33.33 |
| 789 | 2 | 6 | 27 | 33.33 |
| 792 | 2 | 6 | 26 | 33.33 |
| 953 | 2 | 6 | 5 | 33.33 |
| 1041 | 1 | 3 | 13 | 33.33 |
{:.smaller}
{:.scroll-table}

<br>

추출하고 보니 웹 구매는 지난달만 포함한게 아니라 인사이트를 얻을 수 없음
<br>


### 3. 상관관계 테이블

<br>

![png](/assets/img/post/EDA/output_48_1.png)

<br>


> SQL에서는 구현이 어려울 것 같다.....


<br>

income, Web 구매, Catalog 구매, Store 구매 모두 target과 관련이 있다.
Web 방문은 오히려 많을 수록 음의 상관관계를 가지는게 인상적이다.
Catalog 구매와 income이 가장 강한 상관관계를 가지고 있음을 알 수 있다.


<br>

다음 데이터 분석으로 알아보고 싶은 것

<br>

- 성별/연령별 분포 확인해보기 -> 주 타겟층 확인
- 자녀가 있는/많은 사람들의 구매 경향은?
- 구매 횟수와 target 크기를 비교하여 한번에 큰 금액을 구매하는 사람들은 누구일까? -> 맞춤형 서비스
- catalog를 주로 이용하는 사람들은 누구인가? -> catalog 활성화는 유믜미한가?
- 웹사이트를 방문하지만, 구매는 안하는 사람들의 특징은 무엇일까? -> 캠페인을 통해 구매를 유도하는 것이 좋을까?
- Deals 구매를 많이 하는 사람들은 누구인가? -> 혼자인 사람이 많은가?
- 결혼 상태, 교육 수준별 소비 크기가 다른가? 얼만큼 다른가? -> 멤버십을 상태에 따라 다른 프로그램을 운영할 수 있지 않을까?
- 캠페인을 아무것도 수용하지 않는 사람의 행동 패턴은 무엇인가? -> 캠페인이 문제였는가? 사람이 문제였는가?
- 5번째 캠페인을 수용한 사람들은 어떤 특징이 있는가? -> 여러번 캠페인을 던져주는 것이 유의미한가?

<br>
<br>

등의 질문을 던지며
알아보기
