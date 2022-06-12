---
layout: post
title: 지역별 유치원/초등학교 수 관련 분석
description: >
  저출산이 심각한 사회문제가 된 요즘, 유치원과 초등학교는 어떨까? 아이들이 얼마나 줄고 있을까? 학교는? 교원 수는?
sitemap: false
hide_last_modified: true
hide_description: true
categories:
  - study
  - python
---

* toc
{:toc .large-only}

# 지역별 유치원/초등학교 수 관련 분석


## 1. 필요한 라이브러리 가져오기
```python
import pandas as pd
import numpy as np
import plotly.express as px
import matplotlib.pyplot as plt
```

## 1.1 그래프 한글폰트 설정(plt, seaborn에만 필요)
```python
def get_font_family():
    """
    시스템 환경에 따른 기본 폰트명을 반환하는 함수
    """
    import platform
    system_name = platform.system()

    if system_name == "Darwin" :
        font_family = "AppleGothic"
    elif system_name == "Windows":
        font_family = "Malgun Gothic"
    else:
        # Linux(colab)
        !apt-get install fonts-nanum -qq  > /dev/null
        !fc-cache -fv

        import matplotlib as mpl
        mpl.font_manager._rebuild()
        findfont = mpl.font_manager.fontManager.findfont
        mpl.font_manager.findfont = findfont
        mpl.backends.backend_agg.findfont = findfont
        
        font_family = "NanumBarunGothic"
    return font_family


# style 설정은 꼭 폰트설정 위에서 합니다.
# style 에 폰트 설정이 들어있으면 한글폰트가 초기화 되어 한글이 깨집니다.
plt.style.use("seaborn")
# 폰트설정
plt.rc("font", family=get_font_family())

# 마이너스폰트 설정
plt.rc("axes", unicode_minus=False)

# 그래프에 retina display 적용
%config InlineBackend.figure_format = 'retina'
```

## 2. 데이터 불러오기
```python
df = pd.read_csv('2012_2021_school_information_lite.csv', encoding='cp949')
```


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 152641 entries, 0 to 152640
    Data columns (total 15 columns):
     #   Column     Non-Null Count   Dtype  
    ---  ------     --------------   -----  
     0   학교명        152641 non-null  object 
     1   학교 코드      152641 non-null  object 
     2   조사기준일      152641 non-null  int64  
     3   시도         152641 non-null  object 
     4   행정구        152641 non-null  object 
     5   학교급        152641 non-null  object 
     6   개교일        152641 non-null  int64  
     7   상태         152641 non-null  object 
     8   주소         152641 non-null  object 
     9   학급수_계      152641 non-null  int64  
     10  학생수_총계_계   152641 non-null  int64  
     11  학급당 학생수    152641 non-null  float64
     12  교원수_총계_계   152641 non-null  int64  
     13  교원1인당 학생수  152641 non-null  float64
     14  학생1인당교지면적  152641 non-null  float64
    dtypes: float64(3), int64(5), object(7)
    memory usage: 17.5+ MB


## 3. 데이터 오류 정제


### 3.1) 날짜 컬럼 수정

```python
df.loc[df['개교일']<=10000000,['학교명','개교일']]
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
      <th>학교명</th>
      <th>개교일</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>80688</th>
      <td>메이플유치원</td>
      <td>130301</td>
    </tr>
    <tr>
      <th>140948</th>
      <td>문화유치원</td>
      <td>840302</td>
    </tr>
    <tr>
      <th>141147</th>
      <td>메이플유치원</td>
      <td>130301</td>
    </tr>
    <tr>
      <th>141969</th>
      <td>옥동예은유치원</td>
      <td>950301</td>
    </tr>
    <tr>
      <th>149971</th>
      <td>연일형산초등학교병설유치원</td>
      <td>2000301</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.loc[df['개교일']<=10000000,'개교일'] =[20130301, 19980302, 20130401, 19950301, 20040303]
```


컬럼을 날짜타입으로 바꾸기 위해 날짜 형식이 맞지 않은 값들을 수정해주었다.


```python
df['조사기준일']=pd.to_datetime(df['조사기준일'].astype('string')).dt.year
df['개교일']=pd.to_datetime(df['개교일'].astype('int').astype('string'))
```


```python
df.columns
```




    Index(['학교명', '학교 코드', '조사기준일', '시도', '행정구', '학교급', '개교일', '상태', '주소', '학급수_계',
           '학생수_총계_계', '학급당 학생수', '교원수_총계_계', '교원1인당 학생수', '학생1인당교지면적'],
          dtype='object')


하는 김에 날짜 컬럼 명도 바꿔줬다.

```python
df.columns = ['학교명','학교 코드', '연도', '시도', '행정구', '학교급', '개교일', '상태', '주소', '학급수 총계', '학생수 총계',
                      '학급당 학생수', '교원수 총계', '교원1인당 학생수', '학생 1인당 교지면적']
```

### 3.2) key 값 통일

학교코드, 학교명, 주소, 개교일은 모두 동일해야하는데  
띄어쓰기 등의 문제로 조금씩 차이가 있어서  
이것을 모두 학교코드를 기준으로 최빈값으로 통일하였다.  

사실 최빈값이 동률이 나오기도 하는데 크게 유의미한 값은 아니어서 그냥 진행했다.  


```python
### 데이터 통일성 확인 - 학쿄 코드가 동일할 경우 이름과 주소, 개교일을 동일하게 만들어주겠음
code_list = df['학교 코드'].unique()
for code in code_list : 
    data = df[df['학교 코드']==code]
    if data['학교명'].nunique()>1 :
        df.loc[df['학교 코드']==code,'학교명'] = data['학교명'].mode()[0]
    if data['주소'].nunique()>1 :
        df.loc[df['학교 코드']==code,'주소'] = data['주소'].mode()[0]
    if data['개교일'].nunique()>1 :
        if data['개교일'].value_counts()[0]==data['개교일'].value_counts()[1] :
            df.loc[df['학교 코드']==code,'개교일'] = np.NaN
        else : 
            df.loc[df['학교 코드']==code,'개교일'] = data['개교일'].mode()[0]
```

## 4. 데이터 분석

### 4.1) 지역별 학교 당 학생수


```python
# 지역별 학교 수 + 학생 수
def cal_Stu_per_Sch(df_name, st_col, year) :
    result_data = df[df['연도']==year].groupby([st_col]).agg({'학교 코드':'nunique', '학생수 총계':'sum'}).reset_index()
    result_data['학교당 학생수'] = round(result_data['학생수 총계']/result_data['학교 코드'],2)
    result_data.rename(columns={'학교 코드':'학교수 총계'}, inplace=True)
    return(result_data)
```


```python
graph = cal_Stu_per_Sch(df, '시도', 2021)
fig = px.bar(data_frame=graph, x='시도', y='학교당 학생수'
             , custom_data=['학교수 총계','학생수 총계']
             , color='학교당 학생수', color_continuous_scale ='Blues'
             , text="학교당 학생수"
             , title="2021년 지역별 학교당 학생수")
fig.update_xaxes(categoryorder='total descending')
fig.update_traces(marker_line_color='rgb(8,48,107)',
                  marker_line_width=1, opacity=0.6,
                  textfont = {'size':11},
                  texttemplate='%{text}', 
                  textposition='outside')

# 레이블, 배경, hover tooltip 설정 변경
fig.update_layout(annotations = [dict(
                                x=0.98,
                                y=0.98,    #Trying a negative number makes the caption disappear - I'd like the caption to be below the map
                                xref='paper',
                                yref='paper',
                                text='(단위 : 명)',
                                showarrow = False)],
                  margin=dict(pad=5),
                  paper_bgcolor='rgba(0,0,0,0)',
                  plot_bgcolor='rgba(0,0,0,0)',
                  hoverlabel=dict(
                      bgcolor="white",
                      font_size=11,
                      font_family="Open Sans"
                        ))

# hover : tooltip 도구설명 수정
fig.update_traces(
    hovertemplate="<br>".join([
        "지역 : %{x}",
        "학교당 학생수 : %{y}명",
        "학교수 총계 : %{customdata[0]:,.0f}개",
        "학생수 총계 : %{customdata[1]:,.0f}명" ])
    )

fig.update_xaxes(title='')
fig.update_layout(coloraxis_showscale=False,
                  yaxis=dict(showgrid=False))
fig.show()
```

![png](/assets/img/post/20220612_python_kids/newplot (1).png)

pyplot은 동적인 그래프인데, 편의상 이미지로 저장해서 올렸다.  
최종적으로 대시보드를 만드는 목적이 있어서 pyplot을 이용했다.






### 4.2) 지역별 학교당 학생수 증감율


```python
# 데이터프레임, 기준 컬럼, 시작연도, 끝연도를 넣으면 증감율을 계산해서 만들어주는 함수
def cal_Stu_per_Sch_ratio(df_name, st_col, start_year, end_year) : 
    
    #start/end year 데이터가 없으면 drop
    element = df_name[df_name['연도']==end_year][st_col].unique()
    mask = np.isin(df_name[df_name['연도']==end_year][st_col].unique(), df[df['연도']==start_year][st_col].unique())
    st_col_list = element[mask]
    
    st_col_mask = df_name[st_col].isin(st_col_list)
    year_mask = df_name['연도'].isin([start_year, end_year])
    df_name_drop = df_name.loc[st_col_mask & year_mask,:]
    
    #연도별 폐교, 휴원 제외하고 기존, 신설 학교만 포함.
    cal_df = df_name_drop.loc[df_name['상태'].isin(['기존(원)교','신설(원)교']),:]\
            .groupby([st_col,'연도']).agg({'학교 코드':'nunique', '학생수 총계':'sum'})\
            .reset_index()
    
    cal_df['학교당 학생수'] = cal_df['학생수 총계']/cal_df['학교 코드']
    
    cal_df['학교당 학생수 증감율'] = round(cal_df.sort_values([st_col,'연도'])['학교당 학생수'].pct_change()*100,2)
    
    result_data = cal_df[cal_df['연도']==end_year].copy()
    result_data.rename(columns={'학교 코드':'학교수 총계'}, inplace=True)
    return(result_data)
```


아직 함수를 정의해서 쓰는게 불안하다.  
데이터 확인은 필수!  
그래도 함수 짜서 쓰면 뭔가 뿌듯해🙉


```python
cal_Stu_per_Sch_ratio(df, '시도', 2012, 2021)
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
      <th>시도</th>
      <th>연도</th>
      <th>학교수 총계</th>
      <th>학생수 총계</th>
      <th>학교당 학생수</th>
      <th>학교당 학생수 증감율</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>강원</td>
      <td>2021</td>
      <td>731</td>
      <td>86057</td>
      <td>117.725034</td>
      <td>-11.73</td>
    </tr>
    <tr>
      <th>3</th>
      <td>경기</td>
      <td>2021</td>
      <td>3468</td>
      <td>925892</td>
      <td>266.981546</td>
      <td>-8.58</td>
    </tr>
    <tr>
      <th>5</th>
      <td>경남</td>
      <td>2021</td>
      <td>1191</td>
      <td>234320</td>
      <td>196.742233</td>
      <td>-6.84</td>
    </tr>
    <tr>
      <th>7</th>
      <td>경북</td>
      <td>2021</td>
      <td>1155</td>
      <td>161287</td>
      <td>139.642424</td>
      <td>-4.69</td>
    </tr>
    <tr>
      <th>9</th>
      <td>광주</td>
      <td>2021</td>
      <td>435</td>
      <td>107196</td>
      <td>246.427586</td>
      <td>-16.30</td>
    </tr>
    <tr>
      <th>11</th>
      <td>대구</td>
      <td>2021</td>
      <td>565</td>
      <td>155266</td>
      <td>274.807080</td>
      <td>-14.27</td>
    </tr>
    <tr>
      <th>13</th>
      <td>대전</td>
      <td>2021</td>
      <td>398</td>
      <td>99160</td>
      <td>249.145729</td>
      <td>-19.23</td>
    </tr>
    <tr>
      <th>15</th>
      <td>부산</td>
      <td>2021</td>
      <td>691</td>
      <td>192572</td>
      <td>278.685962</td>
      <td>-13.64</td>
    </tr>
    <tr>
      <th>17</th>
      <td>서울</td>
      <td>2021</td>
      <td>1383</td>
      <td>469393</td>
      <td>339.402025</td>
      <td>-16.64</td>
    </tr>
    <tr>
      <th>19</th>
      <td>울산</td>
      <td>2021</td>
      <td>312</td>
      <td>82790</td>
      <td>265.352564</td>
      <td>-9.86</td>
    </tr>
    <tr>
      <th>21</th>
      <td>인천</td>
      <td>2021</td>
      <td>661</td>
      <td>191441</td>
      <td>289.623298</td>
      <td>-10.16</td>
    </tr>
    <tr>
      <th>23</th>
      <td>전남</td>
      <td>2021</td>
      <td>952</td>
      <td>108505</td>
      <td>113.975840</td>
      <td>-6.48</td>
    </tr>
    <tr>
      <th>25</th>
      <td>전북</td>
      <td>2021</td>
      <td>907</td>
      <td>112936</td>
      <td>124.515987</td>
      <td>-14.52</td>
    </tr>
    <tr>
      <th>27</th>
      <td>제주</td>
      <td>2021</td>
      <td>241</td>
      <td>47605</td>
      <td>197.531120</td>
      <td>-0.43</td>
    </tr>
    <tr>
      <th>29</th>
      <td>충남</td>
      <td>2021</td>
      <td>912</td>
      <td>143327</td>
      <td>157.156798</td>
      <td>-0.59</td>
    </tr>
    <tr>
      <th>31</th>
      <td>충북</td>
      <td>2021</td>
      <td>583</td>
      <td>99839</td>
      <td>171.250429</td>
      <td>-6.14</td>
    </tr>
  </tbody>
</table>
</div>




```python
graph = cal_Stu_per_Sch_ratio(df, '시도', 2012, 2021)
fig = px.bar(data_frame=graph, x='시도', y='학교당 학생수 증감율'
             , custom_data=['학교수 총계','학생수 총계']
             , color='학교당 학생수 증감율', color_continuous_scale ='reds_r'
             ,text="학교당 학생수 증감율"
             ,title="10년 전 대비 지역별 학교당 학생수 증감율")

fig.update_xaxes(categoryorder='total descending')
fig.update_traces(marker_line_color='rgb(8,48,107)',
                  marker_line_width=1, opacity=0.6,
                  textfont = {'size':10},
                  texttemplate='%{text:}%', 
                  textposition='outside')

fig.update_layout(margin=dict(pad=5),
                  xaxis={'side': 'top'},
                  paper_bgcolor='rgba(0,0,0,0)',
                  plot_bgcolor='rgba(0,0,0,0)',
                  hoverlabel=dict(
                      bgcolor="white",
                      font_size=11,
                      font_family="Open Sans"
                        ))


# hover : tooltip 도구설명 수정
fig.update_traces(
    hovertemplate="<br>".join([
        "지역 : %{x}",
        "학교당 학생수 증감율 : %{y}%",
        "학교수 총계 : %{customdata[0]:,.0f}개",
        "학생수 총계 : %{customdata[1]:,.0f}명" ])
    )

fig.update_xaxes(title='')
fig.update_layout(coloraxis_showscale=False,
                yaxis=dict(showgrid=False))

fig.show()
```
![png](/assets/img/post/20220612_python_kids/newplot (2).png)


```python
graph = cal_Stu_per_Sch_ratio(df, '시도', 2016, 2021)
fig = px.bar(data_frame=graph, x='시도', y='학교당 학생수 증감율'
             , custom_data=['학교수 총계','학생수 총계']
             , color='학교당 학생수 증감율', color_continuous_scale ='reds_r'
             ,text="학교당 학생수 증감율"
             ,title="5년 전 대비 지역별 학교당 학생수 증감율")

fig.update_xaxes(categoryorder='total descending')
fig.update_traces(marker_line_color='rgb(8,48,107)',
                  marker_line_width=1, opacity=0.6,
                  textfont = {'size':10},
                  texttemplate='%{text:}%', 
                  textposition='outside')

fig.update_layout(margin=dict(pad=5),
                  xaxis={'side': 'top'},
                  paper_bgcolor='rgba(0,0,0,0)',
                  plot_bgcolor='rgba(0,0,0,0)',
                  hoverlabel=dict(
                      bgcolor="white",
                      font_size=11,
                      font_family="Open Sans"
                        ))


# hover : tooltip 도구설명 수정
fig.update_traces(
    hovertemplate="<br>".join([
        "지역 : %{x}",
        "학교당 학생수 증감율 : %{y}%",
        "학교수 총계 : %{customdata[0]:,.0f}개",
        "학생수 총계 : %{customdata[1]:,.0f}명" ])
    )

fig.update_xaxes(title='')
fig.update_layout(coloraxis_showscale=False,
                  yaxis=dict(showgrid=False))

fig.show()
```


![png](/assets/img/post/20220612_python_kids/newplot (3).png)



### 4.3) 지역별 교원 당 학생수


```python
# 지역별 교원 수 + 학생 수
def cal_Stu_per_Tch(df_name, st_col, year) :
    result_data = df[df['연도']==year].groupby([st_col]).agg({'교원수 총계':'sum', '학생수 총계':'sum'}).reset_index()
    result_data['교원당 학생수'] = round(result_data['학생수 총계']/result_data['교원수 총계'],2)
    return(result_data)
```


```python
graph = cal_Stu_per_Tch(df, '시도', 2021)
fig = px.bar(data_frame=graph, x='시도', y='교원당 학생수'
             , custom_data=['교원수 총계','학생수 총계']
             , color='교원당 학생수', color_continuous_scale ='Blues'
             , text="교원당 학생수"
             , title="2021년 지역별 교원당 학생수")
fig.update_xaxes(categoryorder='total descending')
fig.update_traces(marker_line_color='rgb(8,48,107)',
                  marker_line_width=1, opacity=0.6,
                  textfont = {'size':11},
                  texttemplate='%{text}', 
                  textposition='outside')

# 레이블, 배경, hover tooltip 설정 변경
fig.update_layout(annotations = [dict(
                                x=0.98,
                                y=0.98,    #Trying a negative number makes the caption disappear - I'd like the caption to be below the map
                                xref='paper',
                                yref='paper',
                                text='(단위 : 명)',
                                showarrow = False)],
                  margin=dict(pad=5),
                  paper_bgcolor='rgba(0,0,0,0)',
                  plot_bgcolor='rgba(0,0,0,0)',
                  hoverlabel=dict(
                      bgcolor="white",
                      font_size=11,
                      font_family="Open Sans"
                        ))

# hover : tooltip 도구설명 수정
fig.update_traces(
    hovertemplate="<br>".join([
        "지역 : %{x}",
        "교원당 학생수 : %{y}명",
        "교원수 총계 : %{customdata[0]:,.0f}개",
        "학생수 총계 : %{customdata[1]:,.0f}명" ])
    )

fig.update_xaxes(title='')
fig.update_layout(coloraxis_showscale=False,
                  yaxis=dict(showgrid=False))
fig.show()
```


![png](/assets/img/post/20220612_python_kids/newplot (4).png)



### 4.4) 교원당 학생수 증감율


```python
# 데이터프레임, 기준 컬럼, 시작연도, 끝연도를 넣으면 증감율을 계산해서 만들어주는 함수
def cal_Stu_per_Tch_ratio(df_name, st_col, start_year, end_year) : 
    
    #start/end year 데이터가 없으면 drop
    element = df_name[df_name['연도']==end_year][st_col].unique()
    mask = np.isin(df_name[df_name['연도']==end_year][st_col].unique(), df[df['연도']==start_year][st_col].unique())
    st_col_list = element[mask]
    
    st_col_mask = df_name[st_col].isin(st_col_list)
    year_mask = df_name['연도'].isin([start_year, end_year])
    df_name_drop = df_name.loc[st_col_mask & year_mask,:]
    
    #연도별 폐교, 휴원 제외하고 기존, 신설 학교만 포함.
    cal_df = df_name_drop.loc[df_name['상태'].isin(['기존(원)교','신설(원)교']),:]\
            .groupby([st_col,'연도']).agg({'교원수 총계':'sum', '학생수 총계':'sum'})\
            .reset_index()
    
    cal_df['교원당 학생수'] = cal_df['학생수 총계']/cal_df['교원수 총계']
    
    cal_df['교원당 학생수 증감율'] = round(cal_df.sort_values([st_col,'연도'])['교원당 학생수'].pct_change()*100,2)
    
    result_data = cal_df[cal_df['연도']==end_year].copy()
    
    return(result_data)
```


```python
graph = cal_Stu_per_Tch_ratio(df, '시도', 2012, 2021)
fig = px.bar(data_frame=graph, x='시도', y='교원당 학생수 증감율'
             , custom_data=['교원수 총계','학생수 총계']
             , color='교원당 학생수 증감율', color_continuous_scale ='reds_r'
             ,text="교원당 학생수 증감율"
             ,title="10년 전 대비 지역별 교원당 학생수 증감율")

fig.update_xaxes(categoryorder='total descending')
fig.update_traces(marker_line_color='rgb(8,48,107)',
                  marker_line_width=1, opacity=0.6,
                  textfont = {'size':10},
                  texttemplate='%{text:}%', 
                  textposition='outside')

fig.update_layout(margin=dict(pad=5),
                  xaxis={'side': 'top'},
                  paper_bgcolor='rgba(0,0,0,0)',
                  plot_bgcolor='rgba(0,0,0,0)',
                  hoverlabel=dict(
                      bgcolor="white",
                      font_size=11,
                      font_family="Open Sans"
                        ))


# hover : tooltip 도구설명 수정
fig.update_traces(
    hovertemplate="<br>".join([
        "지역 : %{x}",
        "교원당 학생수 증감율 : %{y}%",
        "교원수 총계 : %{customdata[0]:,.0f}개",
        "학생수 총계 : %{customdata[1]:,.0f}명" ])
    )

fig.update_xaxes(title='')
fig.update_layout(coloraxis_showscale=False,
                  yaxis=dict(showgrid=False))

fig.show()
```

![png](/assets/img/post/20220612_python_kids/newplot (5).png)


```python
graph = cal_Stu_per_Tch_ratio(df, '시도', 2016, 2021)
fig = px.bar(data_frame=graph, x='시도', y='교원당 학생수 증감율'
             , custom_data=['교원수 총계','학생수 총계']
             , color='교원당 학생수 증감율', color_continuous_scale ='reds_r'
             ,text="교원당 학생수 증감율"
             ,title="5년 전 대비 지역별 교원당 학생수 증감율")

fig.update_xaxes(categoryorder='total descending')
fig.update_traces(marker_line_color='rgb(8,48,107)',
                  marker_line_width=1, opacity=0.6,
                  textfont = {'size':10},
                  texttemplate='%{text:}%', 
                  textposition='outside')

fig.update_layout(margin=dict(pad=5),
                  xaxis={'side': 'top'},
                  paper_bgcolor='rgba(0,0,0,0)',
                  plot_bgcolor='rgba(0,0,0,0)',
                  hoverlabel=dict(
                      bgcolor="white",
                      font_size=11,
                      font_family="Open Sans"
                        ))


# hover : tooltip 도구설명 수정
fig.update_traces(
    hovertemplate="<br>".join([
        "지역 : %{x}",
        "교원당 학생수 증감율 : %{y}%",
        "교원수 총계 : %{customdata[0]:,.0f}개",
        "학생수 총계 : %{customdata[1]:,.0f}명" ])
    )

fig.update_xaxes(title='')
fig.update_layout(coloraxis_showscale=False,
                  yaxis=dict(showgrid=False))

fig.show()
```


![png](/assets/img/post/20220612_python_kids/newplot (6).png)



만든 것들을 모아 대시보드로 만들고  
streamlit으로 배포하는 것까지 도전해봤다.  

태블로로 간단한 대시보드를 만들 때 웹으로 공유할 수가 없어서 웹으로 배포하고자 하는 니즈가 있었는데  
잘 연습하면 유용하게 써먹을 수 있지 않을까 싶다.  

Dash 프로그램도 다음엔 한번 도전해보고 싶다.   
커스터마이징이 더 자유로운 대신, 하나하나 섬세하게 꾸며줘야 예쁘다고 한다.


[대시보드 웹으로 확인](https://share.streamlit.io/jhyew0n/heyvorite/main/DataScience/Kids-School/kids_school_analysis.py)


