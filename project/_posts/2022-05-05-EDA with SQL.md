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
<div>

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

> 결과

| **id** | **Year_Birth** | **Education** | **Marital_Status** | **Income** | **Kidhome** | **Teenhome** | **Dt_Customer** | **Recency** | **NumDealsPurchases** | **NumWebPurchases** | **NumCatalogPurchases** | **NumStorePurchases** | **NumWebVisitsMonth** | **AcceptedCmp3** | **AcceptedCmp4** | **AcceptedCmp5** | **AcceptedCmp1** | **AcceptedCmp2** | **Complain** | **Response** | **target** | **target** |
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

> 결과

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

> 결과

| count(*) |
|:--------:|
| 1108     |


## 2. 데이터 탐색






