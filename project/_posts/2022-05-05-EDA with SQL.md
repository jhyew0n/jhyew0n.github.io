---
layout: post
title: 소비자 데이터를 이용한 소비 예측 with SQL
description: >
  데이터 분석 스터디 실습, DACON
sitemap: false
hide_last_modified: true
hide_description: true
categories:
  - project
---

# 소비자 데이터 기반 소비 예측 with SQL

* toc
{:toc .large-only}


[ 👉 DACON : 소비자 데이터 기반 소비 예측 경진대회 바로가기](https://dacon.io/competitions/official/235893/overview/description)


## 1. 데이터 확인

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

### 2) Python을 이용해 csv 파일을 Database에 올려줌

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

  
    
[코드 참고 블로그](https://velog.io/@actpjk/21.2.14-pandas-pymysql-sqlalchemy-csv%ED%8C%8C%EC%9D%BC%EC%9D%84-MySQL%EC%97%90-%EC%A0%80%EC%9E%A5%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)

  

```sql
SELECT *
FROM train
```
  


| **id** | **Year_Birth** | **Education** | **Marital_Status** | **Income** | **Kidhome** | **Teenhome** | **Dt_Customer** | **Recency** | **NumDealsPurchases** | **NumWebPurchases** | **NumCatalogPurchases** | **NumStorePurchases** | **NumWebVisitsMonth** | **AcceptedCmp3** | **AcceptedCmp4** | **AcceptedCmp5** | **AcceptedCmp1** | **AcceptedCmp2** | **Complain** | **Response** | **target** | |
|-------:|---------------:|--------------:|-------------------:|-----------:|------------:|-------------:|----------------:|------------:|----------------------:|--------------------:|------------------------:|----------------------:|----------------------:|-----------------:|-----------------:|-----------------:|-----------------:|-----------------:|-------------:|-------------:|-----------:|-----------:|
| 0      | 1974           | Master        | Together           | 46014      | 1           | 1            | 21-01-2013      | 21          | 10                    | 7                   | 1                       | 8                     | 7                     | 0                | 0                | 0                | 0                | 0                | 0            | 0            | 541        |            |
| 1      | 1962           | Graduation    | Single             | 76624      | 0           | 1            | 24-05-2014      | 68          | 1                     | 5                   | 10                      | 7                     | 1                     | 1                | 0                | 0                | 0                | 0                | 0            | 0            | 899        |            |
| 2      | 1951           | Graduation    | Married            | 75903      | 0           | 1            | 08-04-2013      | 50          | 2                     | 6                   | 6                       | 9                     | 3                     | 0                | 0                | 0                | 0                | 0                | 0            | 0            | 901        |            |
| 3      | 1974           | Basic         | Married            | 18393      | 1           | 0            | 29-03-2014      | 2           | 2                     | 3                   | 0                       | 3                     | 8                     | 0                | 0                | 0                | 0                | 0                | 0            | 0            | 50         |            |
| 4      | 1946           | PhD           | Together           | 64014      | 2           | 1            | 10-06-2014      | 56          | 7                     | 8                   | 2                       | 5                     | 7                     | 0                | 0                | 0                | 1                | 0                | 0            | 0            | 444        |            |
{:.smaller}
{:.scroll-table}


## 3) 테이블 구조 확인

```sql
DESC train
```


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


## 4) 데이터 크기 확인


```sql
SELECT count(*)
FROM train
```

| count(*) |
|:--------:|
| 1108     |


## 2. 데이터 탐색

- **카테고리형 변수** : ['Education', 'Marital_Status', 'Kidhome', 'Teenhome'
               'AcceptedCmp1', 'AcceptedCmp2', 'AcceptedCmp3', 'AcceptedCmp4', 'AcceptedCmp5', 
               'Complain', 'Response']

- **수치형 변수** : ['Year_Birth', 'Income', 'Recency', 'NumDealsPurchases', 'NumWebPurchases',
       'NumCatalogPurchases', 'NumStorePurchases', 'NumWebVisitsMonth']

### 2-1) 카테고리형 변수 탐색

```sql
SELECT DISTINCT Education
FROM train;
```
> 'Master', 'Graduation', 'Basic', 'PhD', '2n Cycle'

찾아보니 
Basic(중등 졸업), Graduation(학사), Master(석사), PhD(박사), 2n Cycle(?)

2n Cycle은  뭔지 모르겠다. 

<span style="color:green; font-size:110%; font-weight:bold;"> Education 데이터 분포</span>

```sql
SELECT Education, count(id), count(id) / sum(count(*)) OVER() AS RAT
FROM train
GROUP BY Education;
```

|  Education | count(id) | RAT     |
|:----------:|-----------|---------|
| Master     | 173       | 15.6137 |
| Graduation | 570       | 51.4440 |
| Basic      | 22        | 1.9856  |
| PhD        | 254       | 22.9242 |
| 2n Cycle   | 89        | 8.0325  |
{:.smaller}


<span style="color:green; font-size:110%; font-weight:bold;"> Marital_Status 데이터 분포</span>

```sql
SELECT DISTINCT Marital_Status
FROM train;
```
> 'Together', 'Single', 'Married', 'Widow', 'Divorced', 'Alone', 'YOLO', 'Absurd'


Together(동거), Widow(과부), Divorced(이혼), YOLO, Absurd(having no rational or orderly relationship to hyman life)


Absurd가 뭔지 한참 찾았다...



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


Alone, YOLO, Absurd는 값이 너무 적어서 Single에 포함시킨다.


```sql
UPDATE train
SET Marital_Status = "Single"
WHERE Marital_Status = "YOLO" 
	or Marital_Status = "Absurd"
	or Marital_Status = "Alone" ;
```


UPDATE 구문으로 데이터를 바꾸고 다시 확인해봤다. 이번에는 소비량 합계도 함께 출력해봤다.
UPDATA 구문처럼 원본 테이블을 수정하는 경우에는 


> BEGIN tran
> COMMIT tran


혹시 모를 상황을 대비해 트랜잭션을 설정하고 commit하는게 좋은데,
mysql은 기본적으로 자동 commit이 설정되어 있는 것 같다. 알아서 반영됐다.



```sql
SELECT Marital_Status, count(id), count(id)*100 / sum(count(*)) OVER() AS RAT, sum(target)
FROM train
GROUP BY Marital_Status;



| Education | count(id) |   RAT   | sum(target) |
|:---------:|:---------:|:-------:|:-----------:|
| Together  |       296 | 26.7148 |      176101 |
| Single    |       238 | 21.4801 |      148846 |
| Married   |       415 | 37.4549 |      251653 |
| Widow     |        39 |  3.5199 |       28150 |
| Divorced  |       120 | 10.8303 |       79021 |
{:.smaller}


<span style="color:green; font-size:110%; font-weight:bold;"> Kidhome, Teenhome 데이터 분포</span>


다음으로

Kidhome, Teenhome은 수치형 변수로 보는게 맞을 것 같은데 값의 범위가 좁아서 카테고리형으로 분류한 것 같다.
이거를 카테고리형으로 바꾸려면 3명 이상 이라는 옵션이 있어야 할 것 같은데, 없다.


```sql
SELECT Kidhome, count(id), count(id)*100 / sum(count(*)) OVER() AS RAT, sum(target)
FROM train
GROUP BY Kidhome;
```

> 0, 1, 2


| Kidhome | count(id) |   RAT   | sum(target) |
|:-------:|:---------:|:-------:|:-----------:|
|    0    |       661 | 59.6570 |      590377 |
|    1    |       418 | 37.7256 |       89295 |
|    2    |        29 |  2.6173 |        4099 |
{:.smaller}


```sql
SELECT Teenhome, count(id), count(id)*100 / sum(count(*)) OVER() AS RAT, sum(target)
FROM train
GROUP BY Kidhome;
```

> 0, 1, 2


| Teenhome | count(id) |   RAT   | sum(target) |
|:--------:|:---------:|:-------:|:-----------:|
|     0    |       571 | 51.5343 |      393832 |
|     1    |       507 | 45.7581 |      270224 |
|     2    |        30 |  2.7076 |       19715 |
{:.smaller}


*참고로 Kid는 성인 이전의 아이들을 포괄적으로 의미하고 Teen은 그 중 청소년기의 아이들만 지칭하는 것 같은데
왜 Kidhome 1,2 < Teenhome 1,2 일까...?*


<span style="color:green; font-size:110%; font-weight:bold;"> 그 외 데이터 분포</span>

나머지는 0 또는 1 인 변수 : 'AcceptedCmp1', 'AcceptedCmp2', 'AcceptedCmp3', 'AcceptedCmp4', 'AcceptedCmp5', 
                      'Complain', 'Response



```sql
SELECT sum(AcceptedCmp1), sum(AcceptedCmp2), sum(AcceptedCmp3), sum(AcceptedCmp4), sum(AcceptedCmp5), sum(Complain), sum(Response)
FROM train;
```



| sum(AcceptedCmp1) | sum(AcceptedCmp2) | sum(AcceptedCmp3) | sum(AcceptedCmp4) | sum(AcceptedCmp5) | sum(Complain) | sum(Response) |
|:-----------------:|:-----------------:|:-----------------:|-------------------|-------------------|---------------|---------------|
|         76        |         17        |         77        |         95        |         80        |       10      |      157      |
{:.smaller}



```sql
SELECT sum(AcceptedCmp1)/1108, sum(AcceptedCmp2)/1108, sum(AcceptedCmp3)/1108, sum(AcceptedCmp4)/1108, sum(AcceptedCmp5)/1108, sum(Complain)/1108, sum(Response)/1108
FROM train;
```


두번째 캠페인 수용률이 현저히 낮다.
그 외에는 뚜렷한 경향성은 없는 것 같다.




### 2-2) 수치형 변수 탐색


<span style="color:green; font-size:110%; font-weight:bold;"> Year_Birth </span>




연령대를 구간으로 나누어 세어보기.

일단 범위를 알아보자


```sql
SELECT min(year_birth) , max(year_birth)
FROM train
```

> 1893 \<= age \< 1996

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

| age_range | user_count | user_ratio |
|:---:|---|---|
| 60대 이상 | 347 | 31.3177 |
| 50대 | 311 | 28.0686 |
| 40대 | 296 | 26.7148 |
| 30대 | 147 | 13.2671 |
| 20대 | 7 | 0.6318 |
{:.smaller}


연령대가 확실히 높은 편.



<span style="color:green; font-size:110%; font-weight:bold;"> Income </span>

대략적인 소득수준을 파악하기 위해 고객을 5등분 하여 상위 20%, 40%, 60%, 80%, 100%로 나누어 소득 수준을 파악해보았다.


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


| decile | id_count | avg(income) | min(income) | max(income) |
|:---:|---|---|---|---|
| 1 | 222 | 81502.17117117117 | 71488 | 162397 |
| 2 | 222 | 65154.792792792796 | 58607 | 71427 |
| 3 | 222 | 51822.63063063063 | 44802 | 58582 |
| 4 | 221 | 38516.61990950226 | 32880 | 44794 |
| 5 | 221 | 23191.647058823528 | 1730 | 32644 |
{:.smaller}

> 이렇게 보는 것보다 소득 구간을 직접 보는게 나으려나?



<span style="color:green; font-size:110%; font-weight:bold;"> Recency </span>


'Year_Birth', 'Income', 'Recency', 'NumDealsPurchases', 'NumWebPurchases',
       'NumCatalogPurchases', 'NumStorePurchases', 'NumWebVisitsMonth'










