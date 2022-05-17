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
```


```python
import seaborn as sns
```


```python
import matplotlib.pyplot as plt
```


```python
pw = 'bemore42'
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




    5




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


      Input In [47]
        COALESCE( SUM(CASE WHEN category='20대' THEN 1 END), 0) as 20s,
                                                                   ^
    SyntaxError: invalid decimal literal




```python
df = pd.read_sql(sql, db)
print(df)
```

      Marital_Status   20s    30s    40s   50s   60s  70s
    0       Together  25.0   72.0   88.0  68.0  43.0  0.0
    1         Single  53.0   55.0   57.0  49.0  21.0  3.0
    2        Married  34.0  108.0  138.0  74.0  54.0  6.0
    3          Widow   0.0    3.0    9.0   9.0  17.0  1.0
    4       Divorced   4.0   26.0   44.0  31.0  12.0  3.0


    /Users/heyvorite/opt/anaconda3/lib/python3.9/site-packages/pandas/io/sql.py:761: UserWarning: pandas only support SQLAlchemy connectable(engine/connection) ordatabase string URI or sqlite3 DBAPI2 connectionother DBAPI2 objects are not tested, please consider using SQLAlchemy
      warnings.warn(



```python
df['Marital_Status']=df['Marital_Status'].replace('Married','Together').replace('Widow','Single').replace('Divorced','Single')
```


```python
custom = df.groupby('Marital_Status').sum().reset_index()
```


```python
custom.index = custom['Marital_Status']
```


```python
del custom['Marital_Status']
```


```python
for i in custom.columns :
    custom[i]=custom[i].astype(int)
```


```python
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
```


```python
round(custom*100/total,2).style.background_gradient(cmap='Oranges').set_precision(1)
```

    /var/folders/24/90v1mz7x535_f85xz_jjgbnh0000gn/T/ipykernel_8405/3479166196.py:1: FutureWarning: this method is deprecated in favour of `Styler.format(precision=..)`
      round(custom*100/total,2).style.background_gradient(cmap='Oranges').set_precision(1)





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

    /var/folders/24/90v1mz7x535_f85xz_jjgbnh0000gn/T/ipykernel_8405/720542787.py:1: FutureWarning: this method is deprecated in favour of `Styler.format(precision=..)`
      pd.DataFrame(custom.sum(axis=0)*100/total).style.background_gradient(cmap='Oranges').set_precision(1)





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

        Marital      20s      30s       40s      50s      60s     70s
    0  Together  31490.0  94966.0  123795.0  93759.0  75285.0  8337.0
    1    Single  32901.0  44112.0   63077.0  72645.0  38659.0  4623.0


    /Users/heyvorite/opt/anaconda3/lib/python3.9/site-packages/pandas/io/sql.py:761: UserWarning: pandas only support SQLAlchemy connectable(engine/connection) ordatabase string URI or sqlite3 DBAPI2 connectionother DBAPI2 objects are not tested, please consider using SQLAlchemy
      warnings.warn(



```python
df.index = df['Marital']
```


```python
del df['Marital']
```


```python
total2 = sum(list(df.sum()))
```


```python
# 소비량 비율
(df*100/total2).style.background_gradient(cmap='Oranges').set_precision(1)
```

    /var/folders/24/90v1mz7x535_f85xz_jjgbnh0000gn/T/ipykernel_8405/535086250.py:1: FutureWarning: this method is deprecated in favour of `Styler.format(precision=..)`
      (df*100/total2).style.background_gradient(cmap='Oranges').set_precision(1)





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

    /var/folders/24/90v1mz7x535_f85xz_jjgbnh0000gn/T/ipykernel_8405/3479166196.py:1: FutureWarning: this method is deprecated in favour of `Styler.format(precision=..)`
      round(custom*100/total,2).style.background_gradient(cmap='Oranges').set_precision(1)





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
print(df)
```

          NumWebPurchases  NumCatalogPurchases  NumStorePurchases  age  target  \
    0                   7                    1                  8   41     541   
    1                   5                   10                  7   53     899   
    2                   6                    6                  9   64     901   
    3                   3                    0                  3   41      50   
    4                   8                    2                  5   69     444   
    ...               ...                  ...                ...  ...     ...   
    1103                3                    1                  6   59     241   
    1104                3                    0                  4   29     147   
    1105                1                    0                  2   40      30   
    1106                6                    1                  7   41     447   
    1107                4                    1                  6   63     302   
    
           Marital category  
    0     Together      40대  
    1       Single      50대  
    2     Together      60대  
    3     Together      40대  
    4     Together      60대  
    ...        ...      ...  
    1103  Together      50대  
    1104  Together      20대  
    1105  Together      30대  
    1106    Single      40대  
    1107    Single      60대  
    
    [1108 rows x 7 columns]


    /Users/heyvorite/opt/anaconda3/lib/python3.9/site-packages/pandas/io/sql.py:761: UserWarning: pandas only support SQLAlchemy connectable(engine/connection) ordatabase string URI or sqlite3 DBAPI2 connectionother DBAPI2 objects are not tested, please consider using SQLAlchemy
      warnings.warn(



```python
df['count']= df['NumWebPurchases']+df['NumCatalogPurchases']+df['NumStorePurchases']
```


```python
df =df.iloc[:,3:]
```


```python
CPP = df.groupby(['category','Marital']).agg({'age':'count', 'count':'sum'}).reset_index()
```


```python
CPP['cpp']=CPP['count']/CPP['age']
```


```python
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
```


```python
df = df[df['count']>0]
```


```python
# 결혼 상태, 연령별 1회당 소비량
sns.barplot(data=df, x='category', y='tpc', hue='Marital', ci=None)
```




    <AxesSubplot:xlabel='category', ylabel='tpc'>



    /Users/heyvorite/opt/anaconda3/lib/python3.9/site-packages/IPython/core/pylabtools.py:151: UserWarning: Glyph 45824 (\N{HANGUL SYLLABLE DAE}) missing from current font.
      fig.canvas.print_figure(bytes_io, **kw)
    /Users/heyvorite/opt/anaconda3/lib/python3.9/site-packages/IPython/core/pylabtools.py:151: UserWarning: Glyph 51060 (\N{HANGUL SYLLABLE I}) missing from current font.
      fig.canvas.print_figure(bytes_io, **kw)
    /Users/heyvorite/opt/anaconda3/lib/python3.9/site-packages/IPython/core/pylabtools.py:151: UserWarning: Glyph 49345 (\N{HANGUL SYLLABLE SANG}) missing from current font.
      fig.canvas.print_figure(bytes_io, **kw)



    
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




    <AxesSubplot:xlabel='category', ylabel='count'>



    /Users/heyvorite/opt/anaconda3/lib/python3.9/site-packages/IPython/core/pylabtools.py:151: UserWarning: Glyph 45824 (\N{HANGUL SYLLABLE DAE}) missing from current font.
      fig.canvas.print_figure(bytes_io, **kw)



    
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
print(df)
```

          AcceptedCmp1  AcceptedCmp2  AcceptedCmp3  AcceptedCmp4  AcceptedCmp5  \
    0                0             0             0             0             0   
    1                0             0             1             0             0   
    2                0             0             0             0             0   
    3                0             0             0             0             0   
    4                1             0             0             0             0   
    ...            ...           ...           ...           ...           ...   
    1103             0             0             0             0             0   
    1104             0             0             0             0             0   
    1105             0             0             1             0             0   
    1106             0             0             0             0             0   
    1107             0             0             0             0             0   
    
           Education  age  target   Marital category  
    0         Master   41     541  Together      40대  
    1     Graduation   53     899    Single      50대  
    2     Graduation   64     901  Together      60대  
    3          Basic   41      50  Together      40대  
    4            PhD   69     444  Together      60대  
    ...          ...  ...     ...       ...      ...  
    1103  Graduation   59     241  Together      50대  
    1104  Graduation   29     147  Together      20대  
    1105      Master   40      30  Together      30대  
    1106  Graduation   41     447    Single      40대  
    1107         PhD   63     302    Single      60대  
    
    [1108 rows x 10 columns]


    /Users/heyvorite/opt/anaconda3/lib/python3.9/site-packages/pandas/io/sql.py:761: UserWarning: pandas only support SQLAlchemy connectable(engine/connection) ordatabase string URI or sqlite3 DBAPI2 connectionother DBAPI2 objects are not tested, please consider using SQLAlchemy
      warnings.warn(



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
```


```python
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

          NumDealsPurchases  age   Education  target   Marital category
    0                    10   41      Master     541  Together      40대
    1                     1   53  Graduation     899    Single      50대
    2                     2   64  Graduation     901  Together      60대
    3                     2   41       Basic      50  Together      40대
    4                     7   69         PhD     444  Together      60대
    ...                 ...  ...         ...     ...       ...      ...
    1103                  5   59  Graduation     241  Together      50대
    1104                  3   29  Graduation     147  Together      20대
    1105                  1   40      Master      30  Together      30대
    1106                  8   41  Graduation     447    Single      40대
    1107                  6   63         PhD     302    Single      60대
    
    [1108 rows x 6 columns]


    /Users/heyvorite/opt/anaconda3/lib/python3.9/site-packages/pandas/io/sql.py:761: UserWarning: pandas only support SQLAlchemy connectable(engine/connection) ordatabase string URI or sqlite3 DBAPI2 connectionother DBAPI2 objects are not tested, please consider using SQLAlchemy
      warnings.warn(



```python
df.sort_values('NumDealsPurchases', ascending=False).head(30)
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
      <th>NumDealsPurchases</th>
      <th>age</th>
      <th>Education</th>
      <th>target</th>
      <th>Marital</th>
      <th>category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>91</th>
      <td>15</td>
      <td>52</td>
      <td>PhD</td>
      <td>9</td>
      <td>Together</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>182</th>
      <td>15</td>
      <td>44</td>
      <td>Graduation</td>
      <td>8</td>
      <td>Single</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>778</th>
      <td>15</td>
      <td>48</td>
      <td>2n Cycle</td>
      <td>1082</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>946</th>
      <td>13</td>
      <td>42</td>
      <td>Master</td>
      <td>747</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>736</th>
      <td>12</td>
      <td>59</td>
      <td>Graduation</td>
      <td>684</td>
      <td>Together</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>12</th>
      <td>11</td>
      <td>66</td>
      <td>Master</td>
      <td>1178</td>
      <td>Together</td>
      <td>60대</td>
    </tr>
    <tr>
      <th>510</th>
      <td>11</td>
      <td>66</td>
      <td>Master</td>
      <td>1178</td>
      <td>Together</td>
      <td>60대</td>
    </tr>
    <tr>
      <th>1038</th>
      <td>11</td>
      <td>58</td>
      <td>PhD</td>
      <td>1019</td>
      <td>Together</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>0</th>
      <td>10</td>
      <td>41</td>
      <td>Master</td>
      <td>541</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>157</th>
      <td>10</td>
      <td>45</td>
      <td>Graduation</td>
      <td>1392</td>
      <td>Single</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>414</th>
      <td>10</td>
      <td>39</td>
      <td>Graduation</td>
      <td>515</td>
      <td>Together</td>
      <td>30대</td>
    </tr>
    <tr>
      <th>690</th>
      <td>10</td>
      <td>44</td>
      <td>PhD</td>
      <td>736</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>1067</th>
      <td>10</td>
      <td>52</td>
      <td>PhD</td>
      <td>793</td>
      <td>Together</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>205</th>
      <td>9</td>
      <td>43</td>
      <td>Master</td>
      <td>642</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>211</th>
      <td>9</td>
      <td>49</td>
      <td>Graduation</td>
      <td>433</td>
      <td>Single</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>864</th>
      <td>9</td>
      <td>57</td>
      <td>PhD</td>
      <td>496</td>
      <td>Single</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>1002</th>
      <td>8</td>
      <td>44</td>
      <td>Graduation</td>
      <td>467</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>791</th>
      <td>8</td>
      <td>49</td>
      <td>Master</td>
      <td>610</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>1106</th>
      <td>8</td>
      <td>41</td>
      <td>Graduation</td>
      <td>447</td>
      <td>Single</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>155</th>
      <td>8</td>
      <td>40</td>
      <td>Graduation</td>
      <td>415</td>
      <td>Single</td>
      <td>30대</td>
    </tr>
    <tr>
      <th>701</th>
      <td>8</td>
      <td>63</td>
      <td>2n Cycle</td>
      <td>1120</td>
      <td>Single</td>
      <td>60대</td>
    </tr>
    <tr>
      <th>1074</th>
      <td>8</td>
      <td>36</td>
      <td>Graduation</td>
      <td>653</td>
      <td>Together</td>
      <td>30대</td>
    </tr>
    <tr>
      <th>898</th>
      <td>8</td>
      <td>50</td>
      <td>2n Cycle</td>
      <td>1826</td>
      <td>Single</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>361</th>
      <td>8</td>
      <td>53</td>
      <td>PhD</td>
      <td>929</td>
      <td>Together</td>
      <td>50대</td>
    </tr>
    <tr>
      <th>344</th>
      <td>8</td>
      <td>62</td>
      <td>PhD</td>
      <td>480</td>
      <td>Together</td>
      <td>60대</td>
    </tr>
    <tr>
      <th>44</th>
      <td>7</td>
      <td>62</td>
      <td>PhD</td>
      <td>1314</td>
      <td>Together</td>
      <td>60대</td>
    </tr>
    <tr>
      <th>367</th>
      <td>7</td>
      <td>34</td>
      <td>PhD</td>
      <td>906</td>
      <td>Together</td>
      <td>30대</td>
    </tr>
    <tr>
      <th>541</th>
      <td>7</td>
      <td>44</td>
      <td>PhD</td>
      <td>235</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
    <tr>
      <th>306</th>
      <td>7</td>
      <td>61</td>
      <td>Graduation</td>
      <td>500</td>
      <td>Together</td>
      <td>60대</td>
    </tr>
    <tr>
      <th>236</th>
      <td>7</td>
      <td>43</td>
      <td>Graduation</td>
      <td>1027</td>
      <td>Together</td>
      <td>40대</td>
    </tr>
  </tbody>
</table>
</div>




```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

#fetch 메서드(조회결과 콘솔창에서 보기 위함)
rows = curs.fetchall()
print(rows)
 

```


```python

```


```python

```


```python

```


```python

```


```python
db.commit()
db.close()
```
