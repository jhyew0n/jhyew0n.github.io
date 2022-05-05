---
layout: post
title: 소비자 데이터를 이용한 소비 예측 with python
description: >
  데이터 분석 스터디 실습, DACON
sitemap: false
hide_last_modified: true
hide_description: true
categories:
  - project
---

## 소비자 데이터를 이용한 소비 예측 with python

* toc
{:toc .large-only}


<style>
table{float:left}
</style>



# 소비자 데이터 기반 소비 예측 경진대회 EDA

### 탐색적 데이터 분석(EDA, Exploratory Data Analysis)이란?

벨연구소의 수학자 ‘존 튜키’가 개발한 데이터분석 과정에 대한 개념으로,   
데이터를 분석하고 결과를 내는 과정에 있어서 지속적으로 해당 데이터에 대한 ‘탐색과 이해’를 기본으로 가져야 한다는 것을 의미한다.

데이터 분석의 목적이 정해지면 
- 필요한 데이터가 얼마나 있고 어떤 구조로 되어 있는지 파악 
- 파악한 데이터에서 어떤 의미 있는 결과를 찾을 수 있을지 탐색  <br>

***=> 분석 인사이트를 확보하고 전체적인 데이터 분석의 방향을 잡아 가는 과정***

기본적으로 데이터 자체에 대한 해석이 잘못되면,  
열심히 한줄한줄 코드를 짜며 고생해서 만든 그 데이터 프레임과 시각화한 그래프들이 그냥 휴지조각이 되고 만다

### 1) 데이터 전처리
**데이터를 정규화하거나 표준화하는 작업**
- 데이터가 없을 때 외부 조인
- 정보가 없을 때 ‘정보 없음’으로 출력
- 수치 값을 소수점 둘째 자리로 맞춘다
=> 데이터를 분석할 때 틀린 값이나 오류 값이 나타나지 않음. 

### 2) 데이터 마이닝
**의미 있는 결과를 도출하거나 패턴을 분석**
- 기초 통계 분석
- 특징 분석
- 선호도 분석 등

### 3) 해석과 평가
**분석 결과의 의미를 도출하고 반영**
- 이 과정에서는 근거를 통한 분석가의 주관이 포함될 수 있음
    - 온라인에서도 사업성이 충분히 높을 것이다
    - 직장인들이 타깃 고객이 될 수 있다
    - 스테이크의 상품 특성을 고려한 신 메뉴 상품을 만들어야 한다 등
- 보고자에게 의미를 쉽게 전달하기 위해 데이터 시각화 작업을 진행하기도 함


### 정보 시각화 방법

|     **시간**     |     **분포**     |  **관계**  |     **비교**    |  **공간** |
|:----------------:|:----------------:|:----------:|:---------------:|:---------:|
|    막대그래프    |     파이 차트    |   산점도   |      히트맵     | 지도 매핑 |
| 누적 막대 그래프 |     도넛 차트    |  버블차트  | 체르노프 페이스 |           |
|     점 그래프    |      트리맵      | 히스토그램 |    별 그래프    |           |
|                  | 누적 연속 그래프 |            |   평행 좌표계   |           |
|                  |                  |            |  다차원 척도법  |           |
{:.smaller}



[참고 링크](https://eda-ai-lab.tistory.com/13)




## 데이터 설명

### 1. train.csv : 학습 데이터



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



### 2. test.csv : 테스트 데이터
위와 동일



### 3. sample_submission.csv : 제출 양식 데이터

| col_names |      description      |
|:---------:|:---------------------:|
| id        | 샘플 아이디           |
| target    | 고객의 제품 총 소비량 |
{:.smaller}




## 1. 데이터 불러오기
먼저 주어진 데이터를 불러오고 확인합니다.


```python
# !pip3 install scikit-learn
# !pip3 install seaborn
```


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.preprocessing import MinMaxScaler

# error, warning 무시 --> 경고 끔
pd.set_option('mode.chained_assignment',  None)
```



```python
train = pd.read_csv('소비예측_train.csv')  # 다운받은 csv를 pandas의 DataFrame 형식으로 불러옵니다.
train.head()
```




<div style="width:100%; overflow:auto">
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
<table border="1" class="dataframe" width="100%" border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>Year_Birth</th>
      <th>Education</th>
      <th>Marital_Status</th>
      <th>Income</th>
      <th>Kidhome</th>
      <th>Teenhome</th>
      <th>Dt_Customer</th>
      <th>Recency</th>
      <th>NumDealsPurchases</th>
      <th>...</th>
      <th>NumStorePurchases</th>
      <th>NumWebVisitsMonth</th>
      <th>AcceptedCmp3</th>
      <th>AcceptedCmp4</th>
      <th>AcceptedCmp5</th>
      <th>AcceptedCmp1</th>
      <th>AcceptedCmp2</th>
      <th>Complain</th>
      <th>Response</th>
      <th>target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1974</td>
      <td>Master</td>
      <td>Together</td>
      <td>46014.0</td>
      <td>1</td>
      <td>1</td>
      <td>21-01-2013</td>
      <td>21</td>
      <td>10</td>
      <td>...</td>
      <td>8</td>
      <td>7</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>541</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1962</td>
      <td>Graduation</td>
      <td>Single</td>
      <td>76624.0</td>
      <td>0</td>
      <td>1</td>
      <td>24-05-2014</td>
      <td>68</td>
      <td>1</td>
      <td>...</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>899</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>1951</td>
      <td>Graduation</td>
      <td>Married</td>
      <td>75903.0</td>
      <td>0</td>
      <td>1</td>
      <td>08-04-2013</td>
      <td>50</td>
      <td>2</td>
      <td>...</td>
      <td>9</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>901</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>1974</td>
      <td>Basic</td>
      <td>Married</td>
      <td>18393.0</td>
      <td>1</td>
      <td>0</td>
      <td>29-03-2014</td>
      <td>2</td>
      <td>2</td>
      <td>...</td>
      <td>3</td>
      <td>8</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>50</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>1946</td>
      <td>PhD</td>
      <td>Together</td>
      <td>64014.0</td>
      <td>2</td>
      <td>1</td>
      <td>10-06-2014</td>
      <td>56</td>
      <td>7</td>
      <td>...</td>
      <td>5</td>
      <td>7</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>444</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>




```python
train.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1108 entries, 0 to 1107
    Data columns (total 22 columns):
     #   Column               Non-Null Count  Dtype  
    ---  ------               --------------  -----  
     0   id                   1108 non-null   int64  
     1   Year_Birth           1108 non-null   int64  
     2   Education            1108 non-null   object 
     3   Marital_Status       1108 non-null   object 
     4   Income               1108 non-null   float64
     5   Kidhome              1108 non-null   int64  
     6   Teenhome             1108 non-null   int64  
     7   Dt_Customer          1108 non-null   object 
     8   Recency              1108 non-null   int64  
     9   NumDealsPurchases    1108 non-null   int64  
     10  NumWebPurchases      1108 non-null   int64  
     11  NumCatalogPurchases  1108 non-null   int64  
     12  NumStorePurchases    1108 non-null   int64  
     13  NumWebVisitsMonth    1108 non-null   int64  
     14  AcceptedCmp3         1108 non-null   int64  
     15  AcceptedCmp4         1108 non-null   int64  
     16  AcceptedCmp5         1108 non-null   int64  
     17  AcceptedCmp1         1108 non-null   int64  
     18  AcceptedCmp2         1108 non-null   int64  
     19  Complain             1108 non-null   int64  
     20  Response             1108 non-null   int64  
     21  target               1108 non-null   int64  
    dtypes: float64(1), int64(18), object(3)
    memory usage: 190.6+ KB


## 2. 결측치 확인


```python
train.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1108 entries, 0 to 1107
    Data columns (total 22 columns):
     #   Column               Non-Null Count  Dtype  
    ---  ------               --------------  -----  
     0   id                   1108 non-null   int64  
     1   Year_Birth           1108 non-null   int64  
     2   Education            1108 non-null   object 
     3   Marital_Status       1108 non-null   object 
     4   Income               1108 non-null   float64
     5   Kidhome              1108 non-null   int64  
     6   Teenhome             1108 non-null   int64  
     7   Dt_Customer          1108 non-null   object 
     8   Recency              1108 non-null   int64  
     9   NumDealsPurchases    1108 non-null   int64  
     10  NumWebPurchases      1108 non-null   int64  
     11  NumCatalogPurchases  1108 non-null   int64  
     12  NumStorePurchases    1108 non-null   int64  
     13  NumWebVisitsMonth    1108 non-null   int64  
     14  AcceptedCmp3         1108 non-null   int64  
     15  AcceptedCmp4         1108 non-null   int64  
     16  AcceptedCmp5         1108 non-null   int64  
     17  AcceptedCmp1         1108 non-null   int64  
     18  AcceptedCmp2         1108 non-null   int64  
     19  Complain             1108 non-null   int64  
     20  Response             1108 non-null   int64  
     21  target               1108 non-null   int64  
    dtypes: float64(1), int64(18), object(3)
    memory usage: 190.6+ KB


= 결측치 없음

## 3. 데이터 전처리

### 1) 일자 분리


```python
train.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1108 entries, 0 to 1107
    Data columns (total 22 columns):
     #   Column               Non-Null Count  Dtype  
    ---  ------               --------------  -----  
     0   id                   1108 non-null   int64  
     1   Year_Birth           1108 non-null   int64  
     2   Education            1108 non-null   object 
     3   Marital_Status       1108 non-null   object 
     4   Income               1108 non-null   float64
     5   Kidhome              1108 non-null   int64  
     6   Teenhome             1108 non-null   int64  
     7   Dt_Customer          1108 non-null   object 
     8   Recency              1108 non-null   int64  
     9   NumDealsPurchases    1108 non-null   int64  
     10  NumWebPurchases      1108 non-null   int64  
     11  NumCatalogPurchases  1108 non-null   int64  
     12  NumStorePurchases    1108 non-null   int64  
     13  NumWebVisitsMonth    1108 non-null   int64  
     14  AcceptedCmp3         1108 non-null   int64  
     15  AcceptedCmp4         1108 non-null   int64  
     16  AcceptedCmp5         1108 non-null   int64  
     17  AcceptedCmp1         1108 non-null   int64  
     18  AcceptedCmp2         1108 non-null   int64  
     19  Complain             1108 non-null   int64  
     20  Response             1108 non-null   int64  
     21  target               1108 non-null   int64  
    dtypes: float64(1), int64(18), object(3)
    memory usage: 190.6+ KB



```python
train['Year'] = train['Dt_Customer'].apply(lambda x : x.split('-')[2]).astype('int')
train['Month'] = train['Dt_Customer'].apply(lambda x : x.split('-')[1]).astype('int')
train['Day'] = train['Dt_Customer'].apply(lambda x : x.split('-')[0]).astype('int')
```


```python
# 중복되어 필요없는 것 버리기
train = train.drop('Dt_Customer', axis = 1)
```

### 피쳐 타입 분리


```python
categorical = ['Education', 'Marital_Status', 'Kidhome', 'Teenhome', 
               'AcceptedCmp3', 'AcceptedCmp4', 'AcceptedCmp5', 'AcceptedCmp1', 'AcceptedCmp2', 
               'Complain', 'Response']

quantitative = ['Year_Birth', 'Income', 'Year', 'Month', 'Day', 'Recency', 'NumDealsPurchases', 'NumWebPurchases',
       'NumCatalogPurchases', 'NumStorePurchases', 'NumWebVisitsMonth']

print('카테고리형(정성적) columns : {0} 개'.format(len(categorical)))
print('수치형(정량적) columns : {0} 개'.format(len(quantitative)))
```

    카테고리형(정성적) columns : 11 개
    수치형(정량적) columns : 11 개


## 3. 범주형 데이터 탐색


```python
print(categorical)
```

    ['Education', 'Marital_Status', 'Kidhome', 'Teenhome', 'AcceptedCmp3', 'AcceptedCmp4', 'AcceptedCmp5', 'AcceptedCmp1', 'AcceptedCmp2', 'Complain', 'Response']



```python
# 데이터 별로 값이 들어가 있는지 확인
data={}

for i in categorical :
    data[i] = list(train[i].unique())

print(data)
```

    {'Education': ['Master', 'Graduation', 'Basic', 'PhD', '2n Cycle'], 
    'Marital_Status': ['Together', 'Single', 'Married', 'Widow', 'Divorced', 'Alone', 'YOLO', 'Absurd'], 
    'Kidhome': [1, 0, 2], 
    'Teenhome': [1, 0, 2], 
    'AcceptedCmp3': [0, 1], 
    'AcceptedCmp4': [0, 1], 
    'AcceptedCmp5': [0, 1], 
    'AcceptedCmp1': [0, 1], 
    'AcceptedCmp2': [0, 1], 
    'Complain': [0, 1], 
    'Response': [0, 1]}


***모르는 것 찾아보기***  

- Education : Basic(중등 졸업), Graduation(학사), Master(석사), PhD(박사), 2n Cycle(?)  
- Marital_Status : Together(동거), Widow(과부), Divorced(이혼), YOLO, Absurd(having no rational or orderly relationship to hyman life)  



```python
fig, axes = plt.subplots(4, 3, figsize=(20,15))
fig.suptitle('Distribution of customers per category', fontsize=40)

for ax, feature in zip(axes.flatten(), categorical):
    
    result = train[['id',feature]].groupby(feature).count().reset_index()
    
    # 비율 구하기
    result['ratio']=result['id'].apply(lambda x : round(x*100/sum(data['id']),1) )
    sns.barplot(x=feature, y='ratio', data=result, ax=ax).set(ylim=(0,100))
    
    # 축에 비율 라벨 붙이기
    for p in ax.patches:
        left, bottom, width, height = p.get_bbox().bounds
        ax.annotate("%.1f"%(height)+"%", (left+width/2, height*1.01), ha='center')

plt.show()
```


    
![png](/assets/img/post/EDA/output_33_0.png)
    



```python
axes.flatten()
```




    array([<AxesSubplot:xlabel='Education', ylabel='count'>,
           <AxesSubplot:xlabel='Marital_Status', ylabel='count'>,
           <AxesSubplot:xlabel='Kidhome', ylabel='count'>,
           <AxesSubplot:xlabel='Teenhome', ylabel='count'>,
           <AxesSubplot:xlabel='AcceptedCmp3', ylabel='count'>,
           <AxesSubplot:xlabel='AcceptedCmp4', ylabel='count'>,
           <AxesSubplot:xlabel='AcceptedCmp5', ylabel='count'>,
           <AxesSubplot:xlabel='AcceptedCmp1', ylabel='count'>,
           <AxesSubplot:xlabel='AcceptedCmp2', ylabel='count'>,
           <AxesSubplot:xlabel='Complain', ylabel='count'>,
           <AxesSubplot:xlabel='Response', ylabel='count'>, <AxesSubplot:>],
          dtype=object)



그래프를 보면, 다음과 같이 인사이트를 도출 할 수 있습니다.

1. Marital_Status에서 'Alone', 'YOLO', 'Absurd'는 데이터가 극히 작고 Single에 범주에 포함되므로 single로 통합시켜 분석할 수도 있습니다.
2. Kidhome와 Teenhome에서 자녀 및 청소년을 2명둔 사람은 소수라고 볼 수 있습니다.
3. AcceptedCmp1~5에서 보면, 1의 데이터가 적으므로 켐페인에 참여한 사람은 소수라고 볼 수 있습니다.



## 4. 수치형 데이터 탐색


```python
fig, axes = plt.subplots(4, 3, figsize=(20,20))
fig.suptitle('Distribution of quantitative features', fontsize=40)
#plt.tight_layout()

for ax,feature in zip(axes.flatten(),quantitative):
    sns.histplot(data = train, x = feature, ax=ax, color='#f55354', edgecolor='#f15354')
plt.show()
```


    
![png](/assets/img/post/EDA/output_37_0.png)
    


그래프를 보면, 다음과 같이 인사이트를 도출 할 수 있습니다.

Year_Birth에서 1970 ~ 1980년생의 거래건수가 가장 많이 발생했다고 볼 수 있습니다.     
Income에서 30000~70000 사이 가구 소득의 거래건수가 가장 많았다고 볼 수 있습니다.     
year에서 13년도에 가장 거래건수가 많았다고 볼 수 있습니다.    
Year_Birth, income, NumDealsPurchases, NumWebPurchases, NumWebVisitsMonth에서 이상치 발생 가능성을 볼 수 있습니다.  

## 5. 이상치 찾기
 
boxenplot을 이용해 시각화하여 이해를 돕겠습니다.



```python
fig, axes = plt.subplots(4, 3, figsize=(20,20))
fig.suptitle('Distribution of quantitative features', fontsize=40)
#plt.tight_layout()

for ax,feature in zip(axes.flatten(),quantitative):
    sns.boxenplot(data = train, x = feature, ax=ax, color='#f15354')
plt.show()
```


    
![png](/assets/img/post/EDA/output_40_0.png)
    


IQR을 이용하여 이상치의 최대 제한선을 구하고 그 사이 범위를 구해보겠습니다.

IQR 은 Interquartile range의 약자로써 사분위수의 상위 75% 지점의 값과 하위 25% 지점의 값 차이 (Q3 - Q1)를 의미합니다.


```python
from scipy import stats
def IQR(column):
    Q1 = column.quantile(0.25)
    Q3 = column.quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 -(1.5 * IQR)
    upper_bound = Q3 +(1.5 * IQR)
    return lower_bound, upper_bound

outliers = train[quantitative].apply(lambda column: IQR(column))
print('Range of outliers by method')
outliers
```

    Range of outliers by method





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
        font-size=12px;
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
      <th>Year_Birth</th>
      <th>Income</th>
      <th>Year</th>
      <th>Month</th>
      <th>Day</th>
      <th>Recency</th>
      <th>NumDealsPurchases</th>
      <th>NumWebPurchases</th>
      <th>NumCatalogPurchases</th>
      <th>NumStorePurchases</th>
      <th>NumWebVisitsMonth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1932.0</td>
      <td>-13066.25</td>
      <td>2013.0</td>
      <td>-7.5</td>
      <td>-14.5</td>
      <td>-51.5</td>
      <td>-2.0</td>
      <td>-4.0</td>
      <td>-6.0</td>
      <td>-4.5</td>
      <td>-3.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2004.0</td>
      <td>117159.75</td>
      <td>2013.0</td>
      <td>20.5</td>
      <td>45.5</td>
      <td>152.5</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>10.0</td>
      <td>15.5</td>
      <td>13.0</td>
    </tr>
  </tbody>
</table>
</div>



그럼 이 데이터에서 날짜를 제외한 이상치 데이터가 몇개나 되는지 구해볼까요?


```python
def IQRsum(column):
    Q1 = column.quantile(0.25)
    Q3 = column.quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 -(1.5 * IQR)
    upper_bound = Q3 +(1.5 * IQR)
    return ((column < lower_bound) | (column > upper_bound)).sum()

outliers = train[quantitative].drop(['Year', 'Month', 'Day'], axis = 1).apply(lambda column: IQRsum(column))
print('Number of outliers by method')
outliers
```

    Number of outliers by method





    Year_Birth              2
    Income                  3
    Recency                 0
    NumDealsPurchases      46
    NumWebPurchases         2
    NumCatalogPurchases     9
    NumStorePurchases       0
    NumWebVisitsMonth       4
    dtype: int64



할인된 구매 횟수인 'NumDealsPurchases' 에서 이상치가 46개로 많이 발생했다는 것을 알 수 있습니다.

## 6. 상관관계 확인하기

피어슨 상관관계 분석 방법을 이용하여,

feature들 간의 상관관계를 히트맵을 그려 어떤 컬럼들이 높은 상관관계를 갖는지 알아보겠습니다.


```python
quantitative.append('target')
```


```python
# 수치형 데이터 상관관계 히트맵 시각화
train_corr = train[quantitative]
scaler= MinMaxScaler() 
train_corr[train_corr.columns] = scaler.fit_transform(train_corr[train_corr.columns])
corr28 = train_corr.corr(method= 'pearson')

plt.figure(figsize=(12,10))
sns.heatmap(data = corr28, annot=True, fmt = '.2f', linewidths=.5, cmap='Blues')
plt.title('Correlation between features', fontsize=30)
```




    Text(0.5, 1.0, 'Correlation between features')




    
![png](/assets/img/post/EDA/output_48_1.png)
    


feature가 많아서 헷갈리니 Target과의 관계만을 살펴보겠습니다.


```python
# Target과 피쳐들의 상관관계
s28 = corr28.unstack()
df_temp28 = pd.DataFrame(s28['target'].sort_values(ascending=False), columns=['target'])
df_temp28.style.background_gradient(cmap='viridis')
```




<style type="text/css">
#T_de8de_row0_col0 {
  background-color: #fde725;
  color: #000000;
}
#T_de8de_row1_col0 {
  background-color: #a5db36;
  color: #000000;
}
#T_de8de_row2_col0 {
  background-color: #9dd93b;
  color: #000000;
}
#T_de8de_row3_col0 {
  background-color: #70cf57;
  color: #000000;
}
#T_de8de_row4_col0 {
  background-color: #40bd72;
  color: #f1f1f1;
}
#T_de8de_row5_col0 {
  background-color: #2e6f8e;
  color: #f1f1f1;
}
#T_de8de_row6_col0 {
  background-color: #2e6d8e;
  color: #f1f1f1;
}
#T_de8de_row7_col0 {
  background-color: #306a8e;
  color: #f1f1f1;
}
#T_de8de_row8_col0 {
  background-color: #375a8c;
  color: #f1f1f1;
}
#T_de8de_row9_col0 {
  background-color: #3d4e8a;
  color: #f1f1f1;
}
#T_de8de_row10_col0 {
  background-color: #3e4989;
  color: #f1f1f1;
}
#T_de8de_row11_col0 {
  background-color: #440154;
  color: #f1f1f1;
}

</style>
<table id="T_de8de">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_de8de_level0_col0" class="col_heading level0 col0" >target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_de8de_level0_row0" class="row_heading level0 row0" >target</th>
      <td id="T_de8de_row0_col0" class="data row0 col0" >1.000000</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row1" class="row_heading level0 row1" >NumCatalogPurchases</th>
      <td id="T_de8de_row1_col0" class="data row1 col0" >0.798065</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row2" class="row_heading level0 row2" >Income</th>
      <td id="T_de8de_row2_col0" class="data row2 col0" >0.784084</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row3" class="row_heading level0 row3" >NumStorePurchases</th>
      <td id="T_de8de_row3_col0" class="data row3 col0" >0.677785</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row4" class="row_heading level0 row4" >NumWebPurchases</th>
      <td id="T_de8de_row4_col0" class="data row4 col0" >0.546082</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row5" class="row_heading level0 row5" >Recency</th>
      <td id="T_de8de_row5_col0" class="data row5 col0" >0.050873</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row6" class="row_heading level0 row6" >Month</th>
      <td id="T_de8de_row6_col0" class="data row6 col0" >0.037649</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row7" class="row_heading level0 row7" >Day</th>
      <td id="T_de8de_row7_col0" class="data row7 col0" >0.018917</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row8" class="row_heading level0 row8" >NumDealsPurchases</th>
      <td id="T_de8de_row8_col0" class="data row8 col0" >-0.072802</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row9" class="row_heading level0 row9" >Year_Birth</th>
      <td id="T_de8de_row9_col0" class="data row9 col0" >-0.136035</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row10" class="row_heading level0 row10" >Year</th>
      <td id="T_de8de_row10_col0" class="data row10 col0" >-0.159404</td>
    </tr>
    <tr>
      <th id="T_de8de_level0_row11" class="row_heading level0 row11" >NumWebVisitsMonth</th>
      <td id="T_de8de_row11_col0" class="data row11 col0" >-0.488252</td>
    </tr>
  </tbody>
</table>
{:.scoll-table}



우리가 가장 관심있는 것은 총 제품 소비량와 어떤 컬럼이 가장 상관관계가 높은가 입니다.

그래프를 보니 NumCatalogPurchasest가 가장 높은 양의 상관관계가 있는것으로 나타났네요!

이는 카탈로그를 사용한 구매 수 많을수록 제품 소비량이 늘어나는 것으로 해석할 수 있겠습니다.

또한 상관계수가 0.3 이상이면 유의미한 양의 상관관계를 가진다고 해석할 수 있습니다.

그러므로 Income, NumStorePurchases, NumWebPurchases피쳐들도 상관계수가 0.3 이상이기 때문에 고객의 수입,  매장에서 직접 구매한 횟수, 웹사이트를 통한 구매 건수가 많을 수록 총 제품 소비량이 늘어난다는 결론을 도출 할 수 있겠습니다.

