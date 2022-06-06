---
layout: post
title: 소비자 데이터 기반 소비 예측 with SQL, Python
description: >
  데이터 분석 스터디 실습, DACON
sitemap: false
hide_last_modified: true
hide_description: true
categories:
  - study
  - sql
---


# 소비자 데이터 기반 소비 예측 with SQL, Python


* toc
{:toc .large-only}


## 데이터 

### train.csv : 학습 데이터

|      col_names      | dtype   |                             description                            |
|:-------------------:|---------|:------------------------------------------------------------------:|
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


<span style="color:#268C81; font-size:120%; font-weight:bold;"> 🔎  지난 EDA로 궁금한 것 </span>

<b> 1. 주 타겟층은 누구? </b>ß

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


## import


```python
import pandas as pd
import pymysql
from sqlalchemy import create_engine
import seaborn as sns
import matplotlib.pyplot as plt
```

## pymysql 세팅


```python
db = pymysql.connect(host='localhost', user='root', db='python', password=pw, charset='utf8')
curs = db.cursor()
```

## 1. 주 타겟층 분석

  - 주 타겟 성별+연령


```python
db = pymysql.connect(host='localhost', user='root', db='python', password=pw, charset='utf8')
curs = db.cursor()
 
sql = '''WITH
             users_with_age AS (
              SELECT
                *
                , 2015 - Year_Birth AS age
              FROM train
            )
            , users_with_category AS (
              SELECT
                id
                , Marital_Status
                , age
                , CASE
                    WHEN age BETWEEN 20 AND 30 THEN '20대'
                    WHEN age BETWEEN 30 AND 40 THEN '30대'
                    WHEN age BETWEEN 40 AND 50 THEN '40대'
                    WHEN age BETWEEN 50 AND 60 THEN '50대'
                    WHEN age BETWEEN 60 AND 70 THEN '60대'
                    WHEN age >=71 THEN '70대 이상'
                  END
                 AS category
              FROM
                users_with_age
            )

            SELECT Marital_Status,
                COALESCE( SUM(if(category='20대',1,0) ), 0) as 20s,
                COALESCE( SUM(if(category='30대',1,0) ), 0) as 30s,
                COALESCE( SUM(if(category='40대',1,0) ), 0) as 40s,
                COALESCE( SUM(if(category='50대',1,0) ), 0) as 50s,
                COALESCE( SUM(if(category='60대',1,0) ), 0) as 60s,
                COALESCE( SUM(if(category='70대 이상',1,0) ), 0) as 70s
            FROM users_with_category
            GROUP BY Marital_Status; '''
 
curs.execute(sql)
```



```python
# postgre SQL
SELECT Marital_Status,
    COALESCE( SUM(CASE WHEN category='20대' THEN 1 END), 0) as 20s,
    COALESCE( SUM(CASE WHEN category='30대' THEN  1 END), 0) as 30s,
    COALESCE( SUM(CASE WHEN category='40대' THEN 1 END), 0) as 40s,
    COALESCE( SUM(CASE WHEN category='50대' THEN 1 END), 0) as 50s,
    COALESCE( SUM(CASE WHEN category='60대' THEN 1 END), 0) as 60s,
    COALESCE( SUM(CASE WHEN category='70대 이상' THEN 1 END), 0) as 70s,
FROM users_with_category
GROUP BY Marital_Status ; 
```



```python
df = pd.read_sql(sql, db)
print(df)
```

| Marital_Status | 20s | 30s | 40s | 50s | 60s | 70s |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Together | 25 | 72 | 88 | 68 | 43 | 0 |
| Single | 53 | 55 | 57 | 49 | 21 | 3 |
| Married | 34 | 108 | 138 | 74 | 54 | 6 |
| Widow | 0 | 3 | 9 | 9 | 17 | 1 |
| Divorced | 4 | 26 | 44 | 31 | 12 | 3 |
{:.smaller}



```python
df['Marital_Status']=df['Marital_Status'].replace('Married','Together').replace('Widow','Single').replace('Divorced','Single')

custom = df.groupby('Marital_Status').sum().reset_index()

custom.index = custom['Marital_Status']

del custom['Marital_Status']

for i in custom.columns :
    custom[i]=custom[i].astype(int)

custom.style.background_gradient(cmap='Oranges')
```




<style type="text/css">
#T_92565_row0_col0, #T_92565_row0_col1, #T_92565_row0_col2, #T_92565_row0_col3, #T_92565_row0_col4, #T_92565_row1_col5 {
  background-color: #fff5eb;
  color: #000000;
}
#T_92565_row0_col5, #T_92565_row1_col0, #T_92565_row1_col1, #T_92565_row1_col2, #T_92565_row1_col3, #T_92565_row1_col4 {
  background-color: #7f2704;
  color: #f1f1f1;
}
</style>
<table id="T_92565">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_92565_level0_col0" class="col_heading level0 col0" >20s</th>
      <th id="T_92565_level0_col1" class="col_heading level0 col1" >30s</th>
      <th id="T_92565_level0_col2" class="col_heading level0 col2" >40s</th>
      <th id="T_92565_level0_col3" class="col_heading level0 col3" >50s</th>
      <th id="T_92565_level0_col4" class="col_heading level0 col4" >60s</th>
      <th id="T_92565_level0_col5" class="col_heading level0 col5" >70s</th>
    </tr>
    <tr>
      <th class="index_name level0" >Marital_Status</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
      <th class="blank col2" >&nbsp;</th>
      <th class="blank col3" >&nbsp;</th>
      <th class="blank col4" >&nbsp;</th>
      <th class="blank col5" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_92565_level0_row0" class="row_heading level0 row0" >Single</th>
      <td id="T_92565_row0_col0" class="data row0 col0" >57</td>
      <td id="T_92565_row0_col1" class="data row0 col1" >84</td>
      <td id="T_92565_row0_col2" class="data row0 col2" >110</td>
      <td id="T_92565_row0_col3" class="data row0 col3" >89</td>
      <td id="T_92565_row0_col4" class="data row0 col4" >50</td>
      <td id="T_92565_row0_col5" class="data row0 col5" >7</td>
    </tr>
    <tr>
      <th id="T_92565_level0_row1" class="row_heading level0 row1" >Together</th>
      <td id="T_92565_row1_col0" class="data row1 col0" >59</td>
      <td id="T_92565_row1_col1" class="data row1 col1" >180</td>
      <td id="T_92565_row1_col2" class="data row1 col2" >226</td>
      <td id="T_92565_row1_col3" class="data row1 col3" >142</td>
      <td id="T_92565_row1_col4" class="data row1 col4" >97</td>
      <td id="T_92565_row1_col5" class="data row1 col5" >6</td>
    </tr>
  </tbody>
</table>





```python
total = sum(list(custom.sum()))

round(custom*100/total,2).style.background_gradient(cmap='Oranges').set_precision(1)
```





<style type="text/css">
#T_0957d_row0_col0, #T_0957d_row0_col1, #T_0957d_row0_col2, #T_0957d_row0_col3, #T_0957d_row0_col4, #T_0957d_row1_col5 {
  background-color: #fff5eb;
  color: #000000;
}
#T_0957d_row0_col5, #T_0957d_row1_col0, #T_0957d_row1_col1, #T_0957d_row1_col2, #T_0957d_row1_col3, #T_0957d_row1_col4 {
  background-color: #7f2704;
  color: #f1f1f1;
}
</style>
<table id="T_0957d">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_0957d_level0_col0" class="col_heading level0 col0" >20s</th>
      <th id="T_0957d_level0_col1" class="col_heading level0 col1" >30s</th>
      <th id="T_0957d_level0_col2" class="col_heading level0 col2" >40s</th>
      <th id="T_0957d_level0_col3" class="col_heading level0 col3" >50s</th>
      <th id="T_0957d_level0_col4" class="col_heading level0 col4" >60s</th>
      <th id="T_0957d_level0_col5" class="col_heading level0 col5" >70s</th>
    </tr>
    <tr>
      <th class="index_name level0" >Marital_Status</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
      <th class="blank col2" >&nbsp;</th>
      <th class="blank col3" >&nbsp;</th>
      <th class="blank col4" >&nbsp;</th>
      <th class="blank col5" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_0957d_level0_row0" class="row_heading level0 row0" >Single</th>
      <td id="T_0957d_row0_col0" class="data row0 col0" >5.2</td>
      <td id="T_0957d_row0_col1" class="data row0 col1" >7.6</td>
      <td id="T_0957d_row0_col2" class="data row0 col2" >9.9</td>
      <td id="T_0957d_row0_col3" class="data row0 col3" >8.0</td>
      <td id="T_0957d_row0_col4" class="data row0 col4" >4.5</td>
      <td id="T_0957d_row0_col5" class="data row0 col5" >0.6</td>
    </tr>
    <tr>
      <th id="T_0957d_level0_row1" class="row_heading level0 row1" >Together</th>
      <td id="T_0957d_row1_col0" class="data row1 col0" >5.3</td>
      <td id="T_0957d_row1_col1" class="data row1 col1" >16.3</td>
      <td id="T_0957d_row1_col2" class="data row1 col2" >20.4</td>
      <td id="T_0957d_row1_col3" class="data row1 col3" >12.8</td>
      <td id="T_0957d_row1_col4" class="data row1 col4" >8.8</td>
      <td id="T_0957d_row1_col5" class="data row1 col5" >0.5</td>
    </tr>
  </tbody>
</table>





```python
pd.DataFrame(custom.sum(axis=0)*100/total).style.background_gradient(cmap='Oranges').set_precision(1)
```



<style type="text/css">
#T_9ed1a_row0_col0 {
  background-color: #fdbe84;
  color: #000000;
}
#T_9ed1a_row1_col0 {
  background-color: #ce4401;
  color: #f1f1f1;
}
#T_9ed1a_row2_col0 {
  background-color: #7f2704;
  color: #f1f1f1;
}
#T_9ed1a_row3_col0 {
  background-color: #e75c0c;
  color: #f1f1f1;
}
#T_9ed1a_row4_col0 {
  background-color: #fda35c;
  color: #000000;
}
#T_9ed1a_row5_col0 {
  background-color: #fff5eb;
  color: #000000;
}
</style>
<table id="T_9ed1a">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_9ed1a_level0_col0" class="col_heading level0 col0" >0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_9ed1a_level0_row0" class="row_heading level0 row0" >20s</th>
      <td id="T_9ed1a_row0_col0" class="data row0 col0" >10.5</td>
    </tr>
    <tr>
      <th id="T_9ed1a_level0_row1" class="row_heading level0 row1" >30s</th>
      <td id="T_9ed1a_row1_col0" class="data row1 col0" >23.8</td>
    </tr>
    <tr>
      <th id="T_9ed1a_level0_row2" class="row_heading level0 row2" >40s</th>
      <td id="T_9ed1a_row2_col0" class="data row2 col0" >30.4</td>
    </tr>
    <tr>
      <th id="T_9ed1a_level0_row3" class="row_heading level0 row3" >50s</th>
      <td id="T_9ed1a_row3_col0" class="data row3 col0" >20.9</td>
    </tr>
    <tr>
      <th id="T_9ed1a_level0_row4" class="row_heading level0 row4" >60s</th>
      <td id="T_9ed1a_row4_col0" class="data row4 col0" >13.3</td>
    </tr>
    <tr>
      <th id="T_9ed1a_level0_row5" class="row_heading level0 row5" >70s</th>
      <td id="T_9ed1a_row5_col0" class="data row5 col0" >1.2</td>
    </tr>
  </tbody>
</table>





```python
print(custom.sum(axis=1))
```

  Marital_Status  
  Single      397  
  Together    710  
  dtype: int64  

  - 구매횟수가 많은 사람들은?


```python
sql = '''WITH
             users_with_age AS (
              SELECT
                *
                , 2015 - Year_Birth AS age
              FROM train
            )
            , users_with_category AS (
              SELECT
                id
                , target
                , Marital_Status
                , age
                , CASE
                    WHEN Marital_Status="Married" THEN 'Together'
                    WHEN Marital_Status="Divorced" THEN 'Single'
                    WHEN Marital_Status="Widow" THEN 'Single'
                    ELSE Marital_Status
                  END
                    AS Marital
                , CASE
                    WHEN age BETWEEN 20 AND 30 THEN '20대'
                    WHEN age BETWEEN 30 AND 40 THEN '30대'
                    WHEN age BETWEEN 40 AND 50 THEN '40대'
                    WHEN age BETWEEN 50 AND 60 THEN '50대'
                    WHEN age BETWEEN 60 AND 70 THEN '60대'
                    WHEN age >=71 THEN '70대 이상'
                  END
                 AS category
              FROM
                users_with_age
            )

            SELECT Marital,
                COALESCE( SUM(if(category='20대',target,0) ), 0) as 20s,
                COALESCE( SUM(if(category='30대',target,0) ), 0) as 30s,
                COALESCE( SUM(if(category='40대',target,0) ), 0) as 40s,
                COALESCE( SUM(if(category='50대',target,0) ), 0) as 50s,
                COALESCE( SUM(if(category='60대',target,0) ), 0) as 60s,
                COALESCE( SUM(if(category='70대 이상',target,0) ), 0) as 70s
            FROM users_with_category
            GROUP BY Marital; '''
```


```python
df = pd.read_sql(sql, db)
print(df)
```

| Marital | 20s | 30s | 40s | 50s | 60s | 70s |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Together | 31490 | 94966 | 123795 | 93759 | 75285 | 8337 |
| Single | 32901 | 44112 | 63077 | 72645 | 38659 | 4623 |
{:.smaller}


```python
df.index = df['Marital']

del df['Marital']

total2 = sum(list(df.sum()))
```


```python
# 소비량 비율
(df*100/total2).style.background_gradient(cmap='Oranges').set_precision(1)
```



<style type="text/css">
#T_e1225_row0_col0, #T_e1225_row1_col1, #T_e1225_row1_col2, #T_e1225_row1_col3, #T_e1225_row1_col4, #T_e1225_row1_col5 {
  background-color: #fff5eb;
  color: #000000;
}
#T_e1225_row0_col1, #T_e1225_row0_col2, #T_e1225_row0_col3, #T_e1225_row0_col4, #T_e1225_row0_col5, #T_e1225_row1_col0 {
  background-color: #7f2704;
  color: #f1f1f1;
}
</style>
<table id="T_e1225">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_e1225_level0_col0" class="col_heading level0 col0" >20s</th>
      <th id="T_e1225_level0_col1" class="col_heading level0 col1" >30s</th>
      <th id="T_e1225_level0_col2" class="col_heading level0 col2" >40s</th>
      <th id="T_e1225_level0_col3" class="col_heading level0 col3" >50s</th>
      <th id="T_e1225_level0_col4" class="col_heading level0 col4" >60s</th>
      <th id="T_e1225_level0_col5" class="col_heading level0 col5" >70s</th>
    </tr>
    <tr>
      <th class="index_name level0" >Marital</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
      <th class="blank col2" >&nbsp;</th>
      <th class="blank col3" >&nbsp;</th>
      <th class="blank col4" >&nbsp;</th>
      <th class="blank col5" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_e1225_level0_row0" class="row_heading level0 row0" >Together</th>
      <td id="T_e1225_row0_col0" class="data row0 col0" >4.6</td>
      <td id="T_e1225_row0_col1" class="data row0 col1" >13.9</td>
      <td id="T_e1225_row0_col2" class="data row0 col2" >18.1</td>
      <td id="T_e1225_row0_col3" class="data row0 col3" >13.7</td>
      <td id="T_e1225_row0_col4" class="data row0 col4" >11.0</td>
      <td id="T_e1225_row0_col5" class="data row0 col5" >1.2</td>
    </tr>
    <tr>
      <th id="T_e1225_level0_row1" class="row_heading level0 row1" >Single</th>
      <td id="T_e1225_row1_col0" class="data row1 col0" >4.8</td>
      <td id="T_e1225_row1_col1" class="data row1 col1" >6.5</td>
      <td id="T_e1225_row1_col2" class="data row1 col2" >9.2</td>
      <td id="T_e1225_row1_col3" class="data row1 col3" >10.6</td>
      <td id="T_e1225_row1_col4" class="data row1 col4" >5.7</td>
      <td id="T_e1225_row1_col5" class="data row1 col5" >0.7</td>
    </tr>
  </tbody>
</table>



```python
# 고객수
round(custom*100/total,2).style.background_gradient(cmap='Oranges').set_precision(1)
```



<style type="text/css">
#T_bea05_row0_col0, #T_bea05_row0_col1, #T_bea05_row0_col2, #T_bea05_row0_col3, #T_bea05_row0_col4, #T_bea05_row1_col5 {
  background-color: #fff5eb;
  color: #000000;
}
#T_bea05_row0_col5, #T_bea05_row1_col0, #T_bea05_row1_col1, #T_bea05_row1_col2, #T_bea05_row1_col3, #T_bea05_row1_col4 {
  background-color: #7f2704;
  color: #f1f1f1;
}
</style>
<table id="T_bea05">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_bea05_level0_col0" class="col_heading level0 col0" >20s</th>
      <th id="T_bea05_level0_col1" class="col_heading level0 col1" >30s</th>
      <th id="T_bea05_level0_col2" class="col_heading level0 col2" >40s</th>
      <th id="T_bea05_level0_col3" class="col_heading level0 col3" >50s</th>
      <th id="T_bea05_level0_col4" class="col_heading level0 col4" >60s</th>
      <th id="T_bea05_level0_col5" class="col_heading level0 col5" >70s</th>
    </tr>
    <tr>
      <th class="index_name level0" >Marital_Status</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
      <th class="blank col2" >&nbsp;</th>
      <th class="blank col3" >&nbsp;</th>
      <th class="blank col4" >&nbsp;</th>
      <th class="blank col5" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_bea05_level0_row0" class="row_heading level0 row0" >Single</th>
      <td id="T_bea05_row0_col0" class="data row0 col0" >5.2</td>
      <td id="T_bea05_row0_col1" class="data row0 col1" >7.6</td>
      <td id="T_bea05_row0_col2" class="data row0 col2" >9.9</td>
      <td id="T_bea05_row0_col3" class="data row0 col3" >8.0</td>
      <td id="T_bea05_row0_col4" class="data row0 col4" >4.5</td>
      <td id="T_bea05_row0_col5" class="data row0 col5" >0.6</td>
    </tr>
    <tr>
      <th id="T_bea05_level0_row1" class="row_heading level0 row1" >Together</th>
      <td id="T_bea05_row1_col0" class="data row1 col0" >5.3</td>
      <td id="T_bea05_row1_col1" class="data row1 col1" >16.3</td>
      <td id="T_bea05_row1_col2" class="data row1 col2" >20.4</td>
      <td id="T_bea05_row1_col3" class="data row1 col3" >12.8</td>
      <td id="T_bea05_row1_col4" class="data row1 col4" >8.8</td>
      <td id="T_bea05_row1_col5" class="data row1 col5" >0.5</td>
    </tr>
  </tbody>
</table>



비교적 50대부터 고객수에 비해 쓰는 소비량이 큰 편

  - 구매 횟수가 많은 유형


```python
sql = '''WITH
             users_with_age AS (
              SELECT
                *
                , 2015 - Year_Birth AS age
              FROM train
            )
            , users_with_category AS (
              SELECT
                *
                , CASE
                    WHEN Marital_Status="Married" THEN 'Together'
                    WHEN Marital_Status="Divorced" THEN 'Single'
                    WHEN Marital_Status="Widow" THEN 'Single'
                    ELSE Marital_Status
                  END
                    AS Marital
                , CASE
                    WHEN age BETWEEN 20 AND 30 THEN '20대'
                    WHEN age BETWEEN 30 AND 40 THEN '30대'
                    WHEN age BETWEEN 40 AND 50 THEN '40대'
                    WHEN age BETWEEN 50 AND 60 THEN '50대'
                    WHEN age BETWEEN 60 AND 70 THEN '60대'
                    WHEN age >=71 THEN '70대 이상'
                  END
                 AS category
              FROM
                users_with_age
            )


        SELECT 
            NumWebPurchases, 
            NumCatalogPurchases, 
            NumStorePurchases,
            age,
            target,
            Marital,
            category
        FROM users_with_category
        ; '''
```


```python
df = pd.read_sql(sql, db)
```




```python
df['count']= df['NumWebPurchases']+df['NumCatalogPurchases']+df['NumStorePurchases']
df =df.iloc[:,3:]

CPP = df.groupby(['category','Marital']).agg({'age':'count', 'count':'sum'}).reset_index()
CPP['cpp']=CPP['count']/CPP['age']
CPP[['category','Marital','cpp']].style.background_gradient(cmap='Oranges')
```




<style type="text/css">
#T_4e571_row0_col2 {
  background-color: #fedebf;
  color: #000000;
}
#T_4e571_row1_col2 {
  background-color: #fff5eb;
  color: #000000;
}
#T_4e571_row2_col2 {
  background-color: #fee7d0;
  color: #000000;
}
#T_4e571_row3_col2 {
  background-color: #fee8d2;
  color: #000000;
}
#T_4e571_row4_col2 {
  background-color: #fdd2a6;
  color: #000000;
}
#T_4e571_row5_col2 {
  background-color: #feddbc;
  color: #000000;
}
#T_4e571_row6_col2 {
  background-color: #f98230;
  color: #f1f1f1;
}
#T_4e571_row7_col2 {
  background-color: #fd984b;
  color: #000000;
}
#T_4e571_row8_col2 {
  background-color: #e85d0c;
  color: #f1f1f1;
}
#T_4e571_row9_col2 {
  background-color: #fb8836;
  color: #f1f1f1;
}
#T_4e571_row10_col2 {
  background-color: #f57520;
  color: #f1f1f1;
}
#T_4e571_row11_col2 {
  background-color: #7f2704;
  color: #f1f1f1;
}
</style>
<table id="T_4e571">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_4e571_level0_col0" class="col_heading level0 col0" >category</th>
      <th id="T_4e571_level0_col1" class="col_heading level0 col1" >Marital</th>
      <th id="T_4e571_level0_col2" class="col_heading level0 col2" >cpp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_4e571_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_4e571_row0_col0" class="data row0 col0" >20대</td>
      <td id="T_4e571_row0_col1" class="data row0 col1" >Single</td>
      <td id="T_4e571_row0_col2" class="data row0 col2" >11.912281</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_4e571_row1_col0" class="data row1 col0" >20대</td>
      <td id="T_4e571_row1_col1" class="data row1 col1" >Together</td>
      <td id="T_4e571_row1_col2" class="data row1 col2" >10.525424</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_4e571_row2_col0" class="data row2 col0" >30대</td>
      <td id="T_4e571_row2_col1" class="data row2 col1" >Single</td>
      <td id="T_4e571_row2_col2" class="data row2 col2" >11.506024</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_4e571_row3_col0" class="data row3 col0" >30대</td>
      <td id="T_4e571_row3_col1" class="data row3 col1" >Together</td>
      <td id="T_4e571_row3_col2" class="data row3 col2" >11.427778</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row4" class="row_heading level0 row4" >4</th>
      <td id="T_4e571_row4_col0" class="data row4 col0" >40대</td>
      <td id="T_4e571_row4_col1" class="data row4 col1" >Single</td>
      <td id="T_4e571_row4_col2" class="data row4 col2" >12.472222</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row5" class="row_heading level0 row5" >5</th>
      <td id="T_4e571_row5_col0" class="data row5 col0" >40대</td>
      <td id="T_4e571_row5_col1" class="data row5 col1" >Together</td>
      <td id="T_4e571_row5_col2" class="data row5 col2" >11.986726</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row6" class="row_heading level0 row6" >6</th>
      <td id="T_4e571_row6_col0" class="data row6 col0" >50대</td>
      <td id="T_4e571_row6_col1" class="data row6 col1" >Single</td>
      <td id="T_4e571_row6_col2" class="data row6 col2" >14.910112</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row7" class="row_heading level0 row7" >7</th>
      <td id="T_4e571_row7_col0" class="data row7 col0" >50대</td>
      <td id="T_4e571_row7_col1" class="data row7 col1" >Together</td>
      <td id="T_4e571_row7_col2" class="data row7 col2" >14.262411</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row8" class="row_heading level0 row8" >8</th>
      <td id="T_4e571_row8_col0" class="data row8 col0" >60대</td>
      <td id="T_4e571_row8_col1" class="data row8 col1" >Single</td>
      <td id="T_4e571_row8_col2" class="data row8 col2" >15.980000</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row9" class="row_heading level0 row9" >9</th>
      <td id="T_4e571_row9_col0" class="data row9 col0" >60대</td>
      <td id="T_4e571_row9_col1" class="data row9 col1" >Together</td>
      <td id="T_4e571_row9_col2" class="data row9 col2" >14.731959</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row10" class="row_heading level0 row10" >10</th>
      <td id="T_4e571_row10_col0" class="data row10 col0" >70대 이상</td>
      <td id="T_4e571_row10_col1" class="data row10 col1" >Single</td>
      <td id="T_4e571_row10_col2" class="data row10 col2" >15.285714</td>
    </tr>
    <tr>
      <th id="T_4e571_level0_row11" class="row_heading level0 row11" >11</th>
      <td id="T_4e571_row11_col0" class="data row11 col0" >70대 이상</td>
      <td id="T_4e571_row11_col1" class="data row11 col1" >Together</td>
      <td id="T_4e571_row11_col2" class="data row11 col2" >18.666667</td>
    </tr>
  </tbody>
</table>




50대 이상부터 인당 구매횟수도 많은 편


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>age</th>
      <th>target</th>
      <th>Marital</th>
      <th>category</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>41</td>
      <td>541</td>
      <td>Together</td>
      <td>40대</td>
      <td>16</td>
    </tr>
    <tr>
      <th>1</th>
      <td>53</td>
      <td>899</td>
      <td>Single</td>
      <td>50대</td>
      <td>22</td>
    </tr>
    <tr>
      <th>2</th>
      <td>64</td>
      <td>901</td>
      <td>Together</td>
      <td>60대</td>
      <td>21</td>
    </tr>
    <tr>
      <th>3</th>
      <td>41</td>
      <td>50</td>
      <td>Together</td>
      <td>40대</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>69</td>
      <td>444</td>
      <td>Together</td>
      <td>60대</td>
      <td>15</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1103</th>
      <td>59</td>
      <td>241</td>
      <td>Together</td>
      <td>50대</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1104</th>
      <td>29</td>
      <td>147</td>
      <td>Together</td>
      <td>20대</td>
      <td>7</td>
    </tr>
    <tr>
      <th>1105</th>
      <td>40</td>
      <td>30</td>
      <td>Together</td>
      <td>30대</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1106</th>
      <td>41</td>
      <td>447</td>
      <td>Single</td>
      <td>40대</td>
      <td>14</td>
    </tr>
    <tr>
      <th>1107</th>
      <td>63</td>
      <td>302</td>
      <td>Single</td>
      <td>60대</td>
      <td>11</td>
    </tr>
  </tbody>
</table>
<p>1108 rows × 5 columns</p>
</div>




```python
df['tpc'] = df['target']/df['count']
df = df[df['count']>0]
```


```python
# 결혼 상태, 연령별 1회당 소비량
sns.barplot(data=df, x='category', y='tpc', hue='Marital', ci=None)
```


    
![png](/assets/img/post/Cus_Ana/output_46_2.png)
    



```python
pd.pivot_table(df, index='category', columns='Marital', values='tpc', aggfunc ='mean').style.background_gradient(cmap='Oranges')
```




<style type="text/css">
#T_10462_row0_col0 {
  background-color: #fdd3a7;
  color: #000000;
}
#T_10462_row0_col1, #T_10462_row1_col1, #T_10462_row5_col0 {
  background-color: #fff5eb;
  color: #000000;
}
#T_10462_row1_col0 {
  background-color: #fff1e4;
  color: #000000;
}
#T_10462_row2_col0 {
  background-color: #fdd5ab;
  color: #000000;
}
#T_10462_row2_col1 {
  background-color: #fff2e5;
  color: #000000;
}
#T_10462_row3_col0, #T_10462_row5_col1 {
  background-color: #7f2704;
  color: #f1f1f1;
}
#T_10462_row3_col1 {
  background-color: #fedebd;
  color: #000000;
}
#T_10462_row4_col0 {
  background-color: #e35608;
  color: #f1f1f1;
}
#T_10462_row4_col1 {
  background-color: #fdbd83;
  color: #000000;
}
</style>
<table id="T_10462">
  <thead>
    <tr>
      <th class="index_name level0" >Marital</th>
      <th id="T_10462_level0_col0" class="col_heading level0 col0" >Single</th>
      <th id="T_10462_level0_col1" class="col_heading level0 col1" >Together</th>
    </tr>
    <tr>
      <th class="index_name level0" >category</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_10462_level0_row0" class="row_heading level0 row0" >20대</th>
      <td id="T_10462_row0_col0" class="data row0 col0" >36.074107</td>
      <td id="T_10462_row0_col1" class="data row0 col1" >33.367509</td>
    </tr>
    <tr>
      <th id="T_10462_level0_row1" class="row_heading level0 row1" >30대</th>
      <td id="T_10462_row1_col0" class="data row1 col0" >33.123906</td>
      <td id="T_10462_row1_col1" class="data row1 col1" >33.256436</td>
    </tr>
    <tr>
      <th id="T_10462_level0_row2" class="row_heading level0 row2" >40대</th>
      <td id="T_10462_row2_col0" class="data row2 col0" >35.917083</td>
      <td id="T_10462_row2_col1" class="data row2 col1" >34.416119</td>
    </tr>
    <tr>
      <th id="T_10462_level0_row3" class="row_heading level0 row3" >50대</th>
      <td id="T_10462_row3_col0" class="data row3 col0" >47.131819</td>
      <td id="T_10462_row3_col1" class="data row3 col1" >40.320347</td>
    </tr>
    <tr>
      <th id="T_10462_level0_row4" class="row_heading level0 row4" >60대</th>
      <td id="T_10462_row4_col0" class="data row4 col0" >42.773245</td>
      <td id="T_10462_row4_col1" class="data row4 col1" >46.388702</td>
    </tr>
    <tr>
      <th id="T_10462_level0_row5" class="row_heading level0 row5" >70대 이상</th>
      <td id="T_10462_row5_col0" class="data row5 col0" >32.664724</td>
      <td id="T_10462_row5_col1" class="data row5 col1" >73.892369</td>
    </tr>
  </tbody>
</table>




연령대가 높을수록 인당 구매횟수도, 1회당 소비량도 크다.  
그런데, 50대까진 single들이, 60대부터는 함께 사는 사람들의 소비량이 크다


```python
#회당 소비량 TOP 30 분포
vis = df.sort_values(by='tpc', ascending=False).head(30)
sns.countplot(data=vis, x='category',hue='Marital')
# togeter가 확실히 많다.
```

    
![png](/assets/img/post/Cus_Ana/output_49_2.png)
    


주 타겟층은 30, 40대  
그러나 돈은 50대 이상이 많이 쓴다.  
결혼한 이들이 많은 편이나 인류학적 특성일 것 같다.

## 2. 프로모션의 효과

캠페인을 n번째 수용한 사람들의 특징  
캠페인을 한번도 수용하지 않은 사람들의 최근 구매일과 구매횟수  
할인된 구매가 많은 사람들의 특징과 소비  
결혼 상태와 교육 수준이 target에 미치는 영향  


```python
sql = '''WITH
             users_with_age AS (
              SELECT
                *
                , 2015 - Year_Birth AS age
              FROM train
            )
            , users_with_category AS (
              SELECT
                *
                , CASE
                    WHEN Marital_Status="Married" THEN 'Together'
                    WHEN Marital_Status="Divorced" THEN 'Single'
                    WHEN Marital_Status="Widow" THEN 'Single'
                    ELSE Marital_Status
                  END
                    AS Marital
                , CASE
                    WHEN age BETWEEN 20 AND 30 THEN '20대'
                    WHEN age BETWEEN 30 AND 40 THEN '30대'
                    WHEN age BETWEEN 40 AND 50 THEN '40대'
                    WHEN age BETWEEN 50 AND 60 THEN '50대'
                    WHEN age BETWEEN 60 AND 70 THEN '60대'
                    WHEN age >=71 THEN '70대 이상'
                  END
                 AS category
              FROM
                users_with_age
            )


        SELECT 
            AcceptedCmp1,
            AcceptedCmp2,
            AcceptedCmp3,
            AcceptedCmp4,
            AcceptedCmp5,
            Education,
            age,
            target,
            Marital,
            category
        FROM users_with_category
        ; '''
```


```python
df = pd.read_sql(sql, db)
```


```python
vis = df.groupby(['AcceptedCmp1','AcceptedCmp2','AcceptedCmp3','AcceptedCmp4','AcceptedCmp5'])\
.agg({'age':'mean','Education':'count', 'target':'mean'}).reset_index()
vis[vis['Education']>=10].style.background_gradient(cmap='Oranges')
```



<style type="text/css">
#T_52ba2_row0_col0, #T_52ba2_row0_col1, #T_52ba2_row0_col2, #T_52ba2_row0_col3, #T_52ba2_row0_col4, #T_52ba2_row0_col7, #T_52ba2_row1_col0, #T_52ba2_row1_col1, #T_52ba2_row1_col2, #T_52ba2_row1_col3, #T_52ba2_row2_col0, #T_52ba2_row2_col1, #T_52ba2_row2_col2, #T_52ba2_row2_col4, #T_52ba2_row3_col0, #T_52ba2_row3_col1, #T_52ba2_row3_col2, #T_52ba2_row3_col6, #T_52ba2_row4_col0, #T_52ba2_row4_col1, #T_52ba2_row4_col3, #T_52ba2_row4_col4, #T_52ba2_row4_col5, #T_52ba2_row5_col1, #T_52ba2_row5_col2, #T_52ba2_row5_col3, #T_52ba2_row5_col4, #T_52ba2_row6_col1, #T_52ba2_row6_col2, #T_52ba2_row6_col3, #T_52ba2_row6_col6 {
  background-color: #fff5eb;
  color: #000000;
}
#T_52ba2_row0_col5 {
  background-color: #fedcb9;
  color: #000000;
}
#T_52ba2_row0_col6, #T_52ba2_row1_col4, #T_52ba2_row1_col7, #T_52ba2_row2_col3, #T_52ba2_row2_col5, #T_52ba2_row3_col3, #T_52ba2_row3_col4, #T_52ba2_row4_col2, #T_52ba2_row5_col0, #T_52ba2_row6_col0, #T_52ba2_row6_col4 {
  background-color: #7f2704;
  color: #f1f1f1;
}
#T_52ba2_row1_col5 {
  background-color: #fd9141;
  color: #000000;
}
#T_52ba2_row1_col6 {
  background-color: #fff4e8;
  color: #000000;
}
#T_52ba2_row2_col6, #T_52ba2_row4_col7 {
  background-color: #fff0e2;
  color: #000000;
}
#T_52ba2_row2_col7 {
  background-color: #fdc48f;
  color: #000000;
}
#T_52ba2_row3_col5 {
  background-color: #c64102;
  color: #f1f1f1;
}
#T_52ba2_row3_col7 {
  background-color: #b33b02;
  color: #f1f1f1;
}
#T_52ba2_row4_col6 {
  background-color: #ffeede;
  color: #000000;
}
#T_52ba2_row5_col5 {
  background-color: #8a2b04;
  color: #f1f1f1;
}
#T_52ba2_row5_col6 {
  background-color: #fff3e6;
  color: #000000;
}
#T_52ba2_row5_col7 {
  background-color: #f06712;
  color: #f1f1f1;
}
#T_52ba2_row6_col5 {
  background-color: #fdbb81;
  color: #000000;
}
#T_52ba2_row6_col7 {
  background-color: #842904;
  color: #f1f1f1;
}
</style>
<table id="T_52ba2">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_52ba2_level0_col0" class="col_heading level0 col0" >AcceptedCmp1</th>
      <th id="T_52ba2_level0_col1" class="col_heading level0 col1" >AcceptedCmp2</th>
      <th id="T_52ba2_level0_col2" class="col_heading level0 col2" >AcceptedCmp3</th>
      <th id="T_52ba2_level0_col3" class="col_heading level0 col3" >AcceptedCmp4</th>
      <th id="T_52ba2_level0_col4" class="col_heading level0 col4" >AcceptedCmp5</th>
      <th id="T_52ba2_level0_col5" class="col_heading level0 col5" >age</th>
      <th id="T_52ba2_level0_col6" class="col_heading level0 col6" >Education</th>
      <th id="T_52ba2_level0_col7" class="col_heading level0 col7" >target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_52ba2_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_52ba2_row0_col0" class="data row0 col0" >0</td>
      <td id="T_52ba2_row0_col1" class="data row0 col1" >0</td>
      <td id="T_52ba2_row0_col2" class="data row0 col2" >0</td>
      <td id="T_52ba2_row0_col3" class="data row0 col3" >0</td>
      <td id="T_52ba2_row0_col4" class="data row0 col4" >0</td>
      <td id="T_52ba2_row0_col5" class="data row0 col5" >45.818707</td>
      <td id="T_52ba2_row0_col6" class="data row0 col6" >866</td>
      <td id="T_52ba2_row0_col7" class="data row0 col7" >484.207852</td>
    </tr>
    <tr>
      <th id="T_52ba2_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_52ba2_row1_col0" class="data row1 col0" >0</td>
      <td id="T_52ba2_row1_col1" class="data row1 col1" >0</td>
      <td id="T_52ba2_row1_col2" class="data row1 col2" >0</td>
      <td id="T_52ba2_row1_col3" class="data row1 col3" >0</td>
      <td id="T_52ba2_row1_col4" class="data row1 col4" >1</td>
      <td id="T_52ba2_row1_col5" class="data row1 col5" >48.083333</td>
      <td id="T_52ba2_row1_col6" class="data row1 col6" >24</td>
      <td id="T_52ba2_row1_col7" class="data row1 col7" >1681.583333</td>
    </tr>
    <tr>
      <th id="T_52ba2_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_52ba2_row2_col0" class="data row2 col0" >0</td>
      <td id="T_52ba2_row2_col1" class="data row2 col1" >0</td>
      <td id="T_52ba2_row2_col2" class="data row2 col2" >0</td>
      <td id="T_52ba2_row2_col3" class="data row2 col3" >1</td>
      <td id="T_52ba2_row2_col4" class="data row2 col4" >0</td>
      <td id="T_52ba2_row2_col5" class="data row2 col5" >51.960000</td>
      <td id="T_52ba2_row2_col6" class="data row2 col6" >50</td>
      <td id="T_52ba2_row2_col7" class="data row2 col7" >836.900000</td>
    </tr>
    <tr>
      <th id="T_52ba2_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_52ba2_row3_col0" class="data row3 col0" >0</td>
      <td id="T_52ba2_row3_col1" class="data row3 col1" >0</td>
      <td id="T_52ba2_row3_col2" class="data row3 col2" >0</td>
      <td id="T_52ba2_row3_col3" class="data row3 col3" >1</td>
      <td id="T_52ba2_row3_col4" class="data row3 col4" >1</td>
      <td id="T_52ba2_row3_col5" class="data row3 col5" >50.411765</td>
      <td id="T_52ba2_row3_col6" class="data row3 col6" >17</td>
      <td id="T_52ba2_row3_col7" class="data row3 col7" >1491.823529</td>
    </tr>
    <tr>
      <th id="T_52ba2_level0_row4" class="row_heading level0 row4" >4</th>
      <td id="T_52ba2_row4_col0" class="data row4 col0" >0</td>
      <td id="T_52ba2_row4_col1" class="data row4 col1" >0</td>
      <td id="T_52ba2_row4_col2" class="data row4 col2" >1</td>
      <td id="T_52ba2_row4_col3" class="data row4 col3" >0</td>
      <td id="T_52ba2_row4_col4" class="data row4 col4" >0</td>
      <td id="T_52ba2_row4_col5" class="data row4 col5" >44.403226</td>
      <td id="T_52ba2_row4_col6" class="data row4 col6" >62</td>
      <td id="T_52ba2_row4_col7" class="data row4 col7" >532.919355</td>
    </tr>
    <tr>
      <th id="T_52ba2_level0_row5" class="row_heading level0 row5" >10</th>
      <td id="T_52ba2_row5_col0" class="data row5 col0" >1</td>
      <td id="T_52ba2_row5_col1" class="data row5 col1" >0</td>
      <td id="T_52ba2_row5_col2" class="data row5 col2" >0</td>
      <td id="T_52ba2_row5_col3" class="data row5 col3" >0</td>
      <td id="T_52ba2_row5_col4" class="data row5 col4" >0</td>
      <td id="T_52ba2_row5_col5" class="data row5 col5" >51.687500</td>
      <td id="T_52ba2_row5_col6" class="data row5 col6" >32</td>
      <td id="T_52ba2_row5_col7" class="data row5 col7" >1240.062500</td>
    </tr>
    <tr>
      <th id="T_52ba2_level0_row6" class="row_heading level0 row6" >11</th>
      <td id="T_52ba2_row6_col0" class="data row6 col0" >1</td>
      <td id="T_52ba2_row6_col1" class="data row6 col1" >0</td>
      <td id="T_52ba2_row6_col2" class="data row6 col2" >0</td>
      <td id="T_52ba2_row6_col3" class="data row6 col3" >0</td>
      <td id="T_52ba2_row6_col4" class="data row6 col4" >1</td>
      <td id="T_52ba2_row6_col5" class="data row6 col5" >46.857143</td>
      <td id="T_52ba2_row6_col6" class="data row6 col6" >14</td>
      <td id="T_52ba2_row6_col7" class="data row6 col7" >1662.714286</td>
    </tr>
  </tbody>
</table>





```python
print(df[df['AcceptedCmp1']==1]['age'].mean())
print(df[df['AcceptedCmp2']==1]['age'].mean())
print(df[df['AcceptedCmp3']==1]['age'].mean())
print(df[df['AcceptedCmp4']==1]['age'].mean())
print(df[df['AcceptedCmp5']==1]['age'].mean())
```

  48.55263157894737
  49.64705882352941
  43.311688311688314
  50.747368421052634
  46.775


이걸 보고 고연령대는 상대적으로 빨리 캠페인에 반응하는데 어린 연령층은 그 뒤에 반응한다고 할 수 있을까.....?


```python
cmp_list=[]
for i in range(len(df)) :
    data= df.iloc[i,:]
    if data['AcceptedCmp1']==1 :
        Cmp = 1
    else : 
        if data['AcceptedCmp2']==1 :
            Cmp = 2
        else: 
            if data['AcceptedCmp3']==1 :
                Cmp = 3
            else :
                if data['AcceptedCmp4']==1 :
                    Cmp=4
                else :
                    if data['AcceptedCmp5']==1 :
                        Cmp=5
                    else :
                        Cmp=0
    cmp_list.append(Cmp)

```


```python
df['cmp']=cmp_list
```


```python
print(df[df['cmp']==1]['age'].mean())
print(df[df['cmp']==2]['age'].mean())
print(df[df['cmp']==3]['age'].mean())
print(df[df['cmp']==4]['age'].mean())
print(df[df['cmp']==5]['age'].mean())
```

  48.55263157894737
  48.55555555555556
  43.696969696969695
  51.56716417910448
  48.083333333333336


처음 반응한 캠페인은 큰 차이가 없는 것으로 보인다....

그렇다면 교육 수준과는 관계가 있을까


```python
df['total_cmp']=df['AcceptedCmp1']+df['AcceptedCmp2']+df['AcceptedCmp3']+df['AcceptedCmp4']+df['AcceptedCmp5']

df.groupby('Education').agg({'total_cmp':'mean','cmp':'mean','age':'mean', 'target':'count'})
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>total_cmp</th>
      <th>cmp</th>
      <th>age</th>
      <th>target</th>
    </tr>
    <tr>
      <th>Education</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2n Cycle</th>
      <td>0.247191</td>
      <td>0.359551</td>
      <td>45.348315</td>
      <td>89</td>
    </tr>
    <tr>
      <th>Basic</th>
      <td>0.136364</td>
      <td>0.409091</td>
      <td>37.909091</td>
      <td>22</td>
    </tr>
    <tr>
      <th>Graduation</th>
      <td>0.287719</td>
      <td>0.556140</td>
      <td>44.687719</td>
      <td>570</td>
    </tr>
    <tr>
      <th>Master</th>
      <td>0.352601</td>
      <td>0.820809</td>
      <td>48.641618</td>
      <td>173</td>
    </tr>
    <tr>
      <th>PhD</th>
      <td>0.374016</td>
      <td>0.708661</td>
      <td>49.377953</td>
      <td>254</td>
    </tr>
  </tbody>
</table>
</div>



**total__cmp : 캠페인 총 참여횟수**  
나이가 어려서 그런건진 모르겠지만, Basic이 확실히 Cmp 참여도가 낮다.   

 
**cmp : 첫번째로 참여한 캠페인 번호**  
꽤나 유의미하게, 고학력일수록 뒤의 캠페인에 참여한다.

 -> 고학력자일수록 캠페인을 적극 이용하지만, 초반 캠페인에는 반응도가 떨어지는 편

- 아무 캠페인에도 반응하지 않는 사람


```python
vis2 = pd.concat([df[df['total_cmp']==0]['Education'].value_counts(),df['Education'].value_counts()],axis=1)
vis2.columns = ['edu_0','edu_all']
vis2['ratio'] = vis2['edu_0']*100/vis2['edu_all']
vis2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>edu_0</th>
      <th>edu_all</th>
      <th>ratio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Graduation</th>
      <td>456</td>
      <td>570</td>
      <td>80.000000</td>
    </tr>
    <tr>
      <th>PhD</th>
      <td>190</td>
      <td>254</td>
      <td>74.803150</td>
    </tr>
    <tr>
      <th>Master</th>
      <td>128</td>
      <td>173</td>
      <td>73.988439</td>
    </tr>
    <tr>
      <th>2n Cycle</th>
      <td>73</td>
      <td>89</td>
      <td>82.022472</td>
    </tr>
    <tr>
      <th>Basic</th>
      <td>19</td>
      <td>22</td>
      <td>86.363636</td>
    </tr>
  </tbody>
</table>
</div>




```python
vis3 = pd.concat([df[df['total_cmp']==0]['category'].value_counts(),df['category'].value_counts()],axis=1)
vis3.columns = ['cate_0','cate_all']
vis3['ratio'] = vis3['cate_0']*100/vis3['cate_all']
vis3
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cate_0</th>
      <th>cate_all</th>
      <th>ratio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>40대</th>
      <td>269</td>
      <td>336</td>
      <td>80.059524</td>
    </tr>
    <tr>
      <th>30대</th>
      <td>215</td>
      <td>264</td>
      <td>81.439394</td>
    </tr>
    <tr>
      <th>50대</th>
      <td>170</td>
      <td>231</td>
      <td>73.593074</td>
    </tr>
    <tr>
      <th>60대</th>
      <td>110</td>
      <td>147</td>
      <td>74.829932</td>
    </tr>
    <tr>
      <th>20대</th>
      <td>94</td>
      <td>116</td>
      <td>81.034483</td>
    </tr>
    <tr>
      <th>70대 이상</th>
      <td>7</td>
      <td>13</td>
      <td>53.846154</td>
    </tr>
  </tbody>
</table>
</div>




```python
vis4 = pd.concat([df[df['total_cmp']==0]['Marital'].value_counts(),df['Marital'].value_counts()],axis=1)
vis4.columns = ['mar_0','mar_all']
vis4['ratio'] = vis4['mar_0']*100/vis4['mar_all']
vis4
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mar_0</th>
      <th>mar_all</th>
      <th>ratio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Together</th>
      <td>553</td>
      <td>711</td>
      <td>77.777778</td>
    </tr>
    <tr>
      <th>Single</th>
      <td>313</td>
      <td>397</td>
      <td>78.841310</td>
    </tr>
  </tbody>
</table>
</div>



확실히 상대적으로 고학력자, 고연령층이 캠페인을 더 수용하는 편.  
상관관계는 잘 모르겠다.


# 3. 구매 방식별 타겟 맞춤형 전략

## 1) 구매 빈도 제일 높은 것끼리 묶어 비교해보기.


```python
sql = '''WITH
             users_with_age AS (
              SELECT
                *
                , 2015 - Year_Birth AS age
              FROM train
            )
            , users_with_category AS (
              SELECT
                *
                , CASE
                    WHEN Marital_Status="Married" THEN 'Together'
                    WHEN Marital_Status="Divorced" THEN 'Single'
                    WHEN Marital_Status="Widow" THEN 'Single'
                    ELSE Marital_Status
                  END
                    AS Marital
                , CASE
                    WHEN age BETWEEN 20 AND 30 THEN '20대'
                    WHEN age BETWEEN 30 AND 40 THEN '30대'
                    WHEN age BETWEEN 40 AND 50 THEN '40대'
                    WHEN age BETWEEN 50 AND 60 THEN '50대'
                    WHEN age BETWEEN 60 AND 70 THEN '60대'
                    WHEN age >=71 THEN '70대 이상'
                  END
                 AS category
              FROM
                users_with_age
            )


        SELECT 
            NumDealsPurchases,
            age,
            Education,
            target,
            Marital,
            category
        FROM users_with_category
        ; '''
```


```python
df = pd.read_sql(sql, db)
print(df)
```

## 2) 카탈로그를 주로 이용하는 사람들의 특징



```python
sql = '''WITH
             users_with_age AS (
              SELECT
                *
                , 2015 - Year_Birth AS age
              FROM train
            )
            , users_with_category AS (
              SELECT
                *
                , CASE
                    WHEN Marital_Status="Married" THEN 'Together'
                    WHEN Marital_Status="Divorced" THEN 'Single'
                    WHEN Marital_Status="Widow" THEN 'Single'
                    ELSE Marital_Status
                  END
                    AS Marital
                , CASE
                    WHEN age BETWEEN 20 AND 30 THEN '20대'
                    WHEN age BETWEEN 30 AND 40 THEN '30대'
                    WHEN age BETWEEN 40 AND 50 THEN '40대'
                    WHEN age BETWEEN 50 AND 60 THEN '50대'
                    WHEN age BETWEEN 60 AND 70 THEN '60대'
                    WHEN age >=71 THEN '70대 이상'
                  END
                 AS category
              FROM
                users_with_age
            )


        SELECT 
            NumCatalogPurchases,
            age,
            Education,
            target,
            Marital,
            category
        FROM users_with_category
        ; '''
```


```python
df = pd.read_sql(sql, db)
df['NumCatalogPurchases'].describe()
```

    count    1108.000000
    mean        2.690433
    std         2.792236
    min         0.000000
    25%         0.000000
    50%         2.000000
    75%         4.000000
    max        11.000000
    Name: NumCatalogPurchases, dtype: float64




```python
# 2회 이상 Catalog 구매한 사람들의 수
df['Cata_cnt'] = df['NumCatalogPurchases'].apply(lambda x : 1 if (x>=2) else 0)
pd.pivot_table(df,values = 'Cata_cnt', index='category',
    columns='Marital',
    aggfunc = 'sum').style.background_gradient(cmap='Oranges')
```




<style type="text/css">
#T_58b28_row0_col0 {
  background-color: #fda762;
  color: #000000;
}
#T_58b28_row0_col1 {
  background-color: #fee1c4;
  color: #000000;
}
#T_58b28_row1_col0 {
  background-color: #fb8836;
  color: #f1f1f1;
}
#T_58b28_row1_col1 {
  background-color: #de4e05;
  color: #f1f1f1;
}
#T_58b28_row2_col0 {
  background-color: #bd3e02;
  color: #f1f1f1;
}
#T_58b28_row2_col1, #T_58b28_row3_col0 {
  background-color: #7f2704;
  color: #f1f1f1;
}
#T_58b28_row3_col1 {
  background-color: #a83703;
  color: #f1f1f1;
}
#T_58b28_row4_col0 {
  background-color: #fd9243;
  color: #000000;
}
#T_58b28_row4_col1 {
  background-color: #f36f1a;
  color: #f1f1f1;
}
#T_58b28_row5_col0, #T_58b28_row5_col1 {
  background-color: #fff5eb;
  color: #000000;
}
</style>
<table id="T_58b28">
  <thead>
    <tr>
      <th class="index_name level0" >Marital</th>
      <th id="T_58b28_level0_col0" class="col_heading level0 col0" >Single</th>
      <th id="T_58b28_level0_col1" class="col_heading level0 col1" >Together</th>
    </tr>
    <tr>
      <th class="index_name level0" >category</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_58b28_level0_row0" class="row_heading level0 row0" >20대</th>
      <td id="T_58b28_row0_col0" class="data row0 col0" >28</td>
      <td id="T_58b28_row0_col1" class="data row0 col1" >21</td>
    </tr>
    <tr>
      <th id="T_58b28_level0_row1" class="row_heading level0 row1" >30대</th>
      <td id="T_58b28_row1_col0" class="data row1 col0" >35</td>
      <td id="T_58b28_row1_col1" class="data row1 col1" >77</td>
    </tr>
    <tr>
      <th id="T_58b28_level0_row2" class="row_heading level0 row2" >40대</th>
      <td id="T_58b28_row2_col0" class="data row2 col0" >53</td>
      <td id="T_58b28_row2_col1" class="data row2 col1" >104</td>
    </tr>
    <tr>
      <th id="T_58b28_level0_row3" class="row_heading level0 row3" >50대</th>
      <td id="T_58b28_row3_col0" class="data row3 col0" >64</td>
      <td id="T_58b28_row3_col1" class="data row3 col1" >91</td>
    </tr>
    <tr>
      <th id="T_58b28_level0_row4" class="row_heading level0 row4" >60대</th>
      <td id="T_58b28_row4_col0" class="data row4 col0" >33</td>
      <td id="T_58b28_row4_col1" class="data row4 col1" >65</td>
    </tr>
    <tr>
      <th id="T_58b28_level0_row5" class="row_heading level0 row5" >70대 이상</th>
      <td id="T_58b28_row5_col0" class="data row5 col0" >4</td>
      <td id="T_58b28_row5_col1" class="data row5 col1" >6</td>
    </tr>
  </tbody>
</table>





```python
# 비율로 확인
df['Cata_ratio'] = df['Cata_cnt']*100/df['Cata_cnt'].sum()
pd.pivot_table(df,values = 'Cata_ratio', index='category',
    columns='Marital',
    aggfunc = 'sum').style.background_gradient(cmap='Oranges').format(precision=1)
```




<style type="text/css">
#T_a8d9c_row0_col0 {
  background-color: #fda762;
  color: #000000;
}
#T_a8d9c_row0_col1 {
  background-color: #fee1c4;
  color: #000000;
}
#T_a8d9c_row1_col0 {
  background-color: #fb8836;
  color: #f1f1f1;
}
#T_a8d9c_row1_col1 {
  background-color: #de4e05;
  color: #f1f1f1;
}
#T_a8d9c_row2_col0 {
  background-color: #bd3e02;
  color: #f1f1f1;
}
#T_a8d9c_row2_col1, #T_a8d9c_row3_col0 {
  background-color: #7f2704;
  color: #f1f1f1;
}
#T_a8d9c_row3_col1 {
  background-color: #a83703;
  color: #f1f1f1;
}
#T_a8d9c_row4_col0 {
  background-color: #fd9243;
  color: #000000;
}
#T_a8d9c_row4_col1 {
  background-color: #f36f1a;
  color: #f1f1f1;
}
#T_a8d9c_row5_col0, #T_a8d9c_row5_col1 {
  background-color: #fff5eb;
  color: #000000;
}
</style>
<table id="T_a8d9c">
  <thead>
    <tr>
      <th class="index_name level0" >Marital</th>
      <th id="T_a8d9c_level0_col0" class="col_heading level0 col0" >Single</th>
      <th id="T_a8d9c_level0_col1" class="col_heading level0 col1" >Together</th>
    </tr>
    <tr>
      <th class="index_name level0" >category</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_a8d9c_level0_row0" class="row_heading level0 row0" >20대</th>
      <td id="T_a8d9c_row0_col0" class="data row0 col0" >4.8</td>
      <td id="T_a8d9c_row0_col1" class="data row0 col1" >3.6</td>
    </tr>
    <tr>
      <th id="T_a8d9c_level0_row1" class="row_heading level0 row1" >30대</th>
      <td id="T_a8d9c_row1_col0" class="data row1 col0" >6.0</td>
      <td id="T_a8d9c_row1_col1" class="data row1 col1" >13.3</td>
    </tr>
    <tr>
      <th id="T_a8d9c_level0_row2" class="row_heading level0 row2" >40대</th>
      <td id="T_a8d9c_row2_col0" class="data row2 col0" >9.1</td>
      <td id="T_a8d9c_row2_col1" class="data row2 col1" >17.9</td>
    </tr>
    <tr>
      <th id="T_a8d9c_level0_row3" class="row_heading level0 row3" >50대</th>
      <td id="T_a8d9c_row3_col0" class="data row3 col0" >11.0</td>
      <td id="T_a8d9c_row3_col1" class="data row3 col1" >15.7</td>
    </tr>
    <tr>
      <th id="T_a8d9c_level0_row4" class="row_heading level0 row4" >60대</th>
      <td id="T_a8d9c_row4_col0" class="data row4 col0" >5.7</td>
      <td id="T_a8d9c_row4_col1" class="data row4 col1" >11.2</td>
    </tr>
    <tr>
      <th id="T_a8d9c_level0_row5" class="row_heading level0 row5" >70대 이상</th>
      <td id="T_a8d9c_row5_col0" class="data row5 col0" >0.7</td>
      <td id="T_a8d9c_row5_col1" class="data row5 col1" >1.0</td>
    </tr>
  </tbody>
</table>




카탈로그 구매는 확실히 함께 사는 사람들에게 두드러진 특징으로 나타난다. (20대 제외)  
40-50대에서 이용률이 많음  
이유가 무엇일까?  
Catalog는 함께 사는 사람들이 좋아할만한 품목들이 더 많은가?

## 3) 자녀가 있는/많은 사람들의 구매 경향은?



```python
sql = '''WITH
             users_with_age AS (
              SELECT
                *
                , 2015 - Year_Birth AS age
              FROM train
            )
            , users_with_category AS (
              SELECT
                *
                , CASE
                    WHEN Marital_Status="Married" THEN 'Together'
                    WHEN Marital_Status="Divorced" THEN 'Single'
                    WHEN Marital_Status="Widow" THEN 'Single'
                    ELSE Marital_Status
                  END
                    AS Marital
                , CASE
                    WHEN age BETWEEN 20 AND 30 THEN '20대'
                    WHEN age BETWEEN 30 AND 40 THEN '30대'
                    WHEN age BETWEEN 40 AND 50 THEN '40대'
                    WHEN age BETWEEN 50 AND 60 THEN '50대'
                    WHEN age BETWEEN 60 AND 70 THEN '60대'
                    WHEN age >=71 THEN '70대 이상'
                  END
                 AS category
              FROM
                users_with_age
            )


        SELECT 
            NumCatalogPurchases,
            NumWebPurchases,
            NumDealsPurchases,
            NumStorePurchases,
            NumWebVisitsMonth,
            Teenhome,
            Kidhome,
            target,
            Marital,
            category
        FROM users_with_category
        ; '''
```


```python
df = pd.read_sql(sql, db)
df
```



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NumCatalogPurchases</th>
      <th>NumWebPurchases</th>
      <th>NumDealsPurchases</th>
      <th>NumStorePurchases</th>
      <th>NumWebVisitsMonth</th>
      <th>Teenhome</th>
      <th>Kidhome</th>
      <th>target</th>
      <th>Marital</th>
      <th>category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>7</td>
      <td>10</td>
      <td>8</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>541</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10</td>
      <td>5</td>
      <td>1</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>899</td>
      <td>Single</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>6</td>
      <td>2</td>
      <td>9</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>901</td>
      <td>Together</td>
      <td>60대</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>3</td>
      <td>8</td>
      <td>0</td>
      <td>1</td>
      <td>50</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>8</td>
      <td>7</td>
      <td>5</td>
      <td>7</td>
      <td>1</td>
      <td>2</td>
      <td>444</td>
      <td>Together</td>
      <td>60대</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1103</th>
      <td>1</td>
      <td>3</td>
      <td>5</td>
      <td>6</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>241</td>
      <td>Together</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>1104</th>
      <td>0</td>
      <td>3</td>
      <td>3</td>
      <td>4</td>
      <td>8</td>
      <td>0</td>
      <td>1</td>
      <td>147</td>
      <td>Together</td>
      <td>20대</td>
    </tr>
    <tr>
      <th>1105</th>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>6</td>
      <td>0</td>
      <td>1</td>
      <td>30</td>
      <td>Together</td>
      <td>30대</td>
    </tr>
    <tr>
      <th>1106</th>
      <td>1</td>
      <td>6</td>
      <td>8</td>
      <td>7</td>
      <td>8</td>
      <td>1</td>
      <td>1</td>
      <td>447</td>
      <td>Single</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>1107</th>
      <td>1</td>
      <td>4</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>2</td>
      <td>0</td>
      <td>302</td>
      <td>Single</td>
      <td>60대</td>
    </tr>
  </tbody>
</table>
<p>1108 rows × 10 columns</p>
</div>




```python
df['child']=df['Teenhome']+df['Kidhome']
```


```python
from sklearn import preprocessing

train_corr = df[['child','NumCatalogPurchases','NumWebPurchases','NumDealsPurchases','NumStorePurchases','NumWebVisitsMonth']]
scaler= preprocessing.MinMaxScaler() 
train_corr[train_corr.columns] = scaler.fit_transform(train_corr[train_corr.columns])
corr28 = train_corr.corr(method= 'pearson')

corr28[['child']]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>child</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>child</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>NumCatalogPurchases</th>
      <td>-0.440880</td>
    </tr>
    <tr>
      <th>NumWebPurchases</th>
      <td>-0.133453</td>
    </tr>
    <tr>
      <th>NumDealsPurchases</th>
      <td>0.460249</td>
    </tr>
    <tr>
      <th>NumStorePurchases</th>
      <td>-0.315581</td>
    </tr>
    <tr>
      <th>NumWebVisitsMonth</th>
      <td>0.396654</td>
    </tr>
  </tbody>
</table>
</div>



child와 상관관계가 높은 것  
아이가 있을 수록 카탈로그 구매 감소, 할인된 구매 증가, 지난달 웹방문 증가  
<br>
-> 추론해보면, 아이가 있으니 카탈로그 볼 시간이 없나? 경제적인 것을 중요시해서 할인된 구매를 하고 주로 웹방문을 하나?


```python
df.groupby('child').mean()[['NumCatalogPurchases','NumWebPurchases','NumDealsPurchases','NumStorePurchases','NumWebVisitsMonth', 'target']]\
.style.background_gradient(cmap='Oranges').format(precision=1)
```




<style type="text/css">
#T_bd57d_row0_col0, #T_bd57d_row0_col1, #T_bd57d_row0_col3, #T_bd57d_row0_col5, #T_bd57d_row2_col2, #T_bd57d_row3_col4 {
  background-color: #7f2704;
  color: #f1f1f1;
}
#T_bd57d_row0_col2, #T_bd57d_row0_col4, #T_bd57d_row3_col0, #T_bd57d_row3_col1, #T_bd57d_row3_col3, #T_bd57d_row3_col5 {
  background-color: #fff5eb;
  color: #000000;
}
#T_bd57d_row1_col0 {
  background-color: #fdbf86;
  color: #000000;
}
#T_bd57d_row1_col1 {
  background-color: #ae3903;
  color: #f1f1f1;
}
#T_bd57d_row1_col2 {
  background-color: #fa8532;
  color: #f1f1f1;
}
#T_bd57d_row1_col3 {
  background-color: #fb8735;
  color: #f1f1f1;
}
#T_bd57d_row1_col4 {
  background-color: #c34002;
  color: #f1f1f1;
}
#T_bd57d_row1_col5 {
  background-color: #fdd0a2;
  color: #000000;
}
#T_bd57d_row2_col0 {
  background-color: #feeddb;
  color: #000000;
}
#T_bd57d_row2_col1 {
  background-color: #fdc590;
  color: #000000;
}
#T_bd57d_row2_col3 {
  background-color: #fedcb9;
  color: #000000;
}
#T_bd57d_row2_col4 {
  background-color: #912e04;
  color: #f1f1f1;
}
#T_bd57d_row2_col5 {
  background-color: #fff5ea;
  color: #000000;
}
#T_bd57d_row3_col2 {
  background-color: #8b2c04;
  color: #f1f1f1;
}
</style>
<table id="T_bd57d">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_bd57d_level0_col0" class="col_heading level0 col0" >NumCatalogPurchases</th>
      <th id="T_bd57d_level0_col1" class="col_heading level0 col1" >NumWebPurchases</th>
      <th id="T_bd57d_level0_col2" class="col_heading level0 col2" >NumDealsPurchases</th>
      <th id="T_bd57d_level0_col3" class="col_heading level0 col3" >NumStorePurchases</th>
      <th id="T_bd57d_level0_col4" class="col_heading level0 col4" >NumWebVisitsMonth</th>
      <th id="T_bd57d_level0_col5" class="col_heading level0 col5" >target</th>
    </tr>
    <tr>
      <th class="index_name level0" >child</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
      <th class="blank col2" >&nbsp;</th>
      <th class="blank col3" >&nbsp;</th>
      <th class="blank col4" >&nbsp;</th>
      <th class="blank col5" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_bd57d_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_bd57d_row0_col0" class="data row0 col0" >4.7</td>
      <td id="T_bd57d_row0_col1" class="data row0 col1" >4.5</td>
      <td id="T_bd57d_row0_col2" class="data row0 col2" >1.1</td>
      <td id="T_bd57d_row0_col3" class="data row0 col3" >7.4</td>
      <td id="T_bd57d_row0_col4" class="data row0 col4" >3.6</td>
      <td id="T_bd57d_row0_col5" class="data row0 col5" >1096.9</td>
    </tr>
    <tr>
      <th id="T_bd57d_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_bd57d_row1_col0" class="data row1 col0" >2.1</td>
      <td id="T_bd57d_row1_col1" class="data row1 col1" >4.3</td>
      <td id="T_bd57d_row1_col2" class="data row1 col2" >2.5</td>
      <td id="T_bd57d_row1_col3" class="data row1 col3" >5.7</td>
      <td id="T_bd57d_row1_col4" class="data row1 col4" >5.9</td>
      <td id="T_bd57d_row1_col5" class="data row1 col5" >482.0</td>
    </tr>
    <tr>
      <th id="T_bd57d_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_bd57d_row2_col0" class="data row2 col0" >1.2</td>
      <td id="T_bd57d_row2_col1" class="data row2 col1" >3.5</td>
      <td id="T_bd57d_row2_col2" class="data row2 col2" >3.8</td>
      <td id="T_bd57d_row2_col3" class="data row2 col3" >4.5</td>
      <td id="T_bd57d_row2_col4" class="data row2 col4" >6.3</td>
      <td id="T_bd57d_row2_col5" class="data row2 col5" >279.9</td>
    </tr>
    <tr>
      <th id="T_bd57d_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_bd57d_row3_col0" class="data row3 col0" >1.0</td>
      <td id="T_bd57d_row3_col1" class="data row3 col1" >3.0</td>
      <td id="T_bd57d_row3_col2" class="data row3 col2" >3.6</td>
      <td id="T_bd57d_row3_col3" class="data row3 col3" >3.8</td>
      <td id="T_bd57d_row3_col4" class="data row3 col4" >6.5</td>
      <td id="T_bd57d_row3_col5" class="data row3 col5" >275.5</td>
    </tr>
  </tbody>
</table>




아이가 많을수록 구매량이 적다.   
상대적으로 아이가 많을 수록 할인된 구매가 많고 웹 방문이 많은 편인데,  
경제적 문제를 생각해볼 수 있을 것 같다.

# 4. 웹 방문 빈도수가 오히려 target을 떨어뜨리는 이유?



```python
sql = '''WITH
             users_with_age AS (
              SELECT
                *
                , 2015 - Year_Birth AS age
              FROM train
            )
            , users_with_category AS (
              SELECT
                *
                , CASE
                    WHEN Marital_Status="Married" THEN 'Together'
                    WHEN Marital_Status="Divorced" THEN 'Single'
                    WHEN Marital_Status="Widow" THEN 'Single'
                    ELSE Marital_Status
                  END
                    AS Marital
                , CASE
                    WHEN age BETWEEN 20 AND 30 THEN '20대'
                    WHEN age BETWEEN 30 AND 40 THEN '30대'
                    WHEN age BETWEEN 40 AND 50 THEN '40대'
                    WHEN age BETWEEN 50 AND 60 THEN '50대'
                    WHEN age BETWEEN 60 AND 70 THEN '60대'
                    WHEN age >=71 THEN '70대 이상'
                  END
                 AS category
              FROM
                users_with_age
            )


        SELECT 
            NumCatalogPurchases,
            NumWebPurchases,
            NumDealsPurchases,
            NumStorePurchases,
            NumWebVisitsMonth,
            age,
            Teenhome,
            Kidhome,
            target,
            Marital,
            category,
            Education
        FROM users_with_category
        ; '''
```


```python
df = pd.read_sql(sql, db)
df
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NumCatalogPurchases</th>
      <th>NumWebPurchases</th>
      <th>NumDealsPurchases</th>
      <th>NumStorePurchases</th>
      <th>NumWebVisitsMonth</th>
      <th>age</th>
      <th>Teenhome</th>
      <th>Kidhome</th>
      <th>target</th>
      <th>Marital</th>
      <th>category</th>
      <th>Education</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>7</td>
      <td>10</td>
      <td>8</td>
      <td>7</td>
      <td>41</td>
      <td>1</td>
      <td>1</td>
      <td>541</td>
      <td>Together</td>
      <td>40대</td>
      <td>Master</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10</td>
      <td>5</td>
      <td>1</td>
      <td>7</td>
      <td>1</td>
      <td>53</td>
      <td>1</td>
      <td>0</td>
      <td>899</td>
      <td>Single</td>
      <td>50대</td>
      <td>Graduation</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>6</td>
      <td>2</td>
      <td>9</td>
      <td>3</td>
      <td>64</td>
      <td>1</td>
      <td>0</td>
      <td>901</td>
      <td>Together</td>
      <td>60대</td>
      <td>Graduation</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>3</td>
      <td>8</td>
      <td>41</td>
      <td>0</td>
      <td>1</td>
      <td>50</td>
      <td>Together</td>
      <td>40대</td>
      <td>Basic</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>8</td>
      <td>7</td>
      <td>5</td>
      <td>7</td>
      <td>69</td>
      <td>1</td>
      <td>2</td>
      <td>444</td>
      <td>Together</td>
      <td>60대</td>
      <td>PhD</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1103</th>
      <td>1</td>
      <td>3</td>
      <td>5</td>
      <td>6</td>
      <td>4</td>
      <td>59</td>
      <td>1</td>
      <td>0</td>
      <td>241</td>
      <td>Together</td>
      <td>50대</td>
      <td>Graduation</td>
    </tr>
    <tr>
      <th>1104</th>
      <td>0</td>
      <td>3</td>
      <td>3</td>
      <td>4</td>
      <td>8</td>
      <td>29</td>
      <td>0</td>
      <td>1</td>
      <td>147</td>
      <td>Together</td>
      <td>20대</td>
      <td>Graduation</td>
    </tr>
    <tr>
      <th>1105</th>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>6</td>
      <td>40</td>
      <td>0</td>
      <td>1</td>
      <td>30</td>
      <td>Together</td>
      <td>30대</td>
      <td>Master</td>
    </tr>
    <tr>
      <th>1106</th>
      <td>1</td>
      <td>6</td>
      <td>8</td>
      <td>7</td>
      <td>8</td>
      <td>41</td>
      <td>1</td>
      <td>1</td>
      <td>447</td>
      <td>Single</td>
      <td>40대</td>
      <td>Graduation</td>
    </tr>
    <tr>
      <th>1107</th>
      <td>1</td>
      <td>4</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>63</td>
      <td>2</td>
      <td>0</td>
      <td>302</td>
      <td>Single</td>
      <td>60대</td>
      <td>PhD</td>
    </tr>
  </tbody>
</table>
<p>1108 rows × 12 columns</p>
</div>



## 1) 평균 방문 경향


```python
df['NumWebVisitsMonth'].describe()
```




    count    1108.000000
    mean        5.348375
    std         2.405115
    min         0.000000
    25%         3.000000
    50%         6.000000
    75%         7.000000
    max        20.000000
    Name: NumWebVisitsMonth, dtype: float64



절반 이상은 지난달 6번 이상 웹에 방문했다.

## 2) web 방문 빈도가 높은 사람들의 성별+연령


```python
pd.pivot_table(df[df['NumWebVisitsMonth']>=6], 
               index = 'Marital',
               columns='category', 
               values='target', 
               aggfunc='count').style.background_gradient(cmap='Oranges').format(precision=1)
```




<style type="text/css">
#T_b2bc7_row0_col0, #T_b2bc7_row0_col1, #T_b2bc7_row0_col2, #T_b2bc7_row0_col3, #T_b2bc7_row0_col4, #T_b2bc7_row0_col5, #T_b2bc7_row1_col5 {
  background-color: #fff5eb;
  color: #000000;
}
#T_b2bc7_row1_col0, #T_b2bc7_row1_col1, #T_b2bc7_row1_col2, #T_b2bc7_row1_col3, #T_b2bc7_row1_col4 {
  background-color: #7f2704;
  color: #f1f1f1;
}
</style>
<table id="T_b2bc7">
  <thead>
    <tr>
      <th class="index_name level0" >category</th>
      <th id="T_b2bc7_level0_col0" class="col_heading level0 col0" >20대</th>
      <th id="T_b2bc7_level0_col1" class="col_heading level0 col1" >30대</th>
      <th id="T_b2bc7_level0_col2" class="col_heading level0 col2" >40대</th>
      <th id="T_b2bc7_level0_col3" class="col_heading level0 col3" >50대</th>
      <th id="T_b2bc7_level0_col4" class="col_heading level0 col4" >60대</th>
      <th id="T_b2bc7_level0_col5" class="col_heading level0 col5" >70대 이상</th>
    </tr>
    <tr>
      <th class="index_name level0" >Marital</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
      <th class="blank col2" >&nbsp;</th>
      <th class="blank col3" >&nbsp;</th>
      <th class="blank col4" >&nbsp;</th>
      <th class="blank col5" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_b2bc7_level0_row0" class="row_heading level0 row0" >Single</th>
      <td id="T_b2bc7_row0_col0" class="data row0 col0" >23.0</td>
      <td id="T_b2bc7_row0_col1" class="data row0 col1" >45.0</td>
      <td id="T_b2bc7_row0_col2" class="data row0 col2" >69.0</td>
      <td id="T_b2bc7_row0_col3" class="data row0 col3" >39.0</td>
      <td id="T_b2bc7_row0_col4" class="data row0 col4" >24.0</td>
      <td id="T_b2bc7_row0_col5" class="data row0 col5" >3.0</td>
    </tr>
    <tr>
      <th id="T_b2bc7_level0_row1" class="row_heading level0 row1" >Together</th>
      <td id="T_b2bc7_row1_col0" class="data row1 col0" >37.0</td>
      <td id="T_b2bc7_row1_col1" class="data row1 col1" >101.0</td>
      <td id="T_b2bc7_row1_col2" class="data row1 col2" >139.0</td>
      <td id="T_b2bc7_row1_col3" class="data row1 col3" >58.0</td>
      <td id="T_b2bc7_row1_col4" class="data row1 col4" >43.0</td>
      <td id="T_b2bc7_row1_col5" class="data row1 col5" >nan</td>
    </tr>
  </tbody>
</table>




할인된 구매와 유사하게,함께 사는 사람들이 확실히 웹방문도 많이 하는 편  
그 중에서도 30-40대의 방문 빈도가 높은편  


```python
pd.pivot_table(df[df['NumWebVisitsMonth']>=6], 
               index = 'Marital',
               columns='Education', 
               values='target', 
               aggfunc='count').style.background_gradient(cmap='Oranges').format(precision=1)
```




<style type="text/css">
#T_6ed12_row0_col0, #T_6ed12_row0_col1, #T_6ed12_row0_col2, #T_6ed12_row0_col3, #T_6ed12_row0_col4 {
  background-color: #fff5eb;
  color: #000000;
}
#T_6ed12_row1_col0, #T_6ed12_row1_col1, #T_6ed12_row1_col2, #T_6ed12_row1_col3, #T_6ed12_row1_col4 {
  background-color: #7f2704;
  color: #f1f1f1;
}
</style>
<table id="T_6ed12">
  <thead>
    <tr>
      <th class="index_name level0" >Education</th>
      <th id="T_6ed12_level0_col0" class="col_heading level0 col0" >2n Cycle</th>
      <th id="T_6ed12_level0_col1" class="col_heading level0 col1" >Basic</th>
      <th id="T_6ed12_level0_col2" class="col_heading level0 col2" >Graduation</th>
      <th id="T_6ed12_level0_col3" class="col_heading level0 col3" >Master</th>
      <th id="T_6ed12_level0_col4" class="col_heading level0 col4" >PhD</th>
    </tr>
    <tr>
      <th class="index_name level0" >Marital</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
      <th class="blank col2" >&nbsp;</th>
      <th class="blank col3" >&nbsp;</th>
      <th class="blank col4" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_6ed12_level0_row0" class="row_heading level0 row0" >Single</th>
      <td id="T_6ed12_row0_col0" class="data row0 col0" >14</td>
      <td id="T_6ed12_row0_col1" class="data row0 col1" >4</td>
      <td id="T_6ed12_row0_col2" class="data row0 col2" >110</td>
      <td id="T_6ed12_row0_col3" class="data row0 col3" >30</td>
      <td id="T_6ed12_row0_col4" class="data row0 col4" >45</td>
    </tr>
    <tr>
      <th id="T_6ed12_level0_row1" class="row_heading level0 row1" >Together</th>
      <td id="T_6ed12_row1_col0" class="data row1 col0" >32</td>
      <td id="T_6ed12_row1_col1" class="data row1 col1" >14</td>
      <td id="T_6ed12_row1_col2" class="data row1 col2" >199</td>
      <td id="T_6ed12_row1_col3" class="data row1 col3" >60</td>
      <td id="T_6ed12_row1_col4" class="data row1 col4" >74</td>
    </tr>
  </tbody>
</table>




학력보다는 나이의 문제일 것 같음

## 3) web 방문 빈도가 높은 사람들의 구매 패턴


```python

train_corr = df[['age','NumCatalogPurchases','NumWebPurchases','NumDealsPurchases','NumStorePurchases','NumWebVisitsMonth','target']]
scaler= preprocessing.MinMaxScaler() 
train_corr[train_corr.columns] = scaler.fit_transform(train_corr[train_corr.columns])
corr28 = train_corr.corr(method= 'pearson')

corr28[['NumWebVisitsMonth']]
```



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NumWebVisitsMonth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>age</th>
      <td>-0.094419</td>
    </tr>
    <tr>
      <th>NumCatalogPurchases</th>
      <td>-0.513101</td>
    </tr>
    <tr>
      <th>NumWebPurchases</th>
      <td>-0.085937</td>
    </tr>
    <tr>
      <th>NumDealsPurchases</th>
      <td>0.378053</td>
    </tr>
    <tr>
      <th>NumStorePurchases</th>
      <td>-0.429612</td>
    </tr>
    <tr>
      <th>NumWebVisitsMonth</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>target</th>
      <td>-0.488252</td>
    </tr>
  </tbody>
</table>
</div>



웹 방문은 카탈로그 구매횟수와 반비례 관계가 있는 편이다.-> 카탈로그와 웹의 반대되는 특성  
의외로 나이와는 선형관계가 없어보인다.  
할인된 구매는 유일한 양의 상관관계.  
전제 소비량과 음의 상관관계가 있는 것이 인상깊다.


```python
sns.scatterplot(data=df, x='NumWebVisitsMonth', y='target')
```

    
![png](/assets/img/post/Cus_Ana/output_106_1.png)
    



```python
sns.scatterplot(data=df, x='NumWebVisitsMonth', y='NumDealsPurchases')
```

    
![png](/assets/img/post/Cus_Ana/output_107_1.png)
    


```python
db.commit()
db.close()
```
