---
layout: post
title: 코로나19 데이터 분석 연습
description: >
    코로나19 확진자/사망자 데이터, 백신 데이터를 이용한 데이터 분석 스터디 연습 과제
sitemap: false
hide_last_modified: false
hide_description: true
categories:
  - favorite
---

# 코로나 확진자 패턴 분석

* toc
{:toc .large-only}



### 1. 사용 데이터 확인

[데이터 출처](https://covid19.who.int/data)


```python
import os
file_list = os.listdir()
```


```python
data_list = [x for x in file_list if "csv" in x]
print(data_list)
```

    ['vaccination-metadata.csv', 'World Country Population.csv', 'vaccination-data.csv', 'WHO-COVID-19-global-table-data.csv', 'WHO-COVID-19-global-data.csv']



```python
# !pip3 install pandas
import pandas as pd
```

### 1) 국가별 확진자, 사망자 데이터

|Field name|Type|Description|
|:----:|:----:|:----|
|Date_reported|Date|Date of reporting to WHO|
|Country_code|String|ISO Alpha-2 country code|
|Country|String|Country, territory, area|
|WHO_region|String|WHO regional offices: WHO Member States are grouped into six WHO regions -- Regional Office for Africa (AFRO), Regional Office for the Americas (AMRO), Regional Office for South-East Asia (SEARO), Regional Office for Europe (EURO), Regional Office for the Eastern Mediterranean (EMRO), and Regional Office for the Western Pacific (WPRO).|
|New_cases|Integer|New confirmed cases. Calculated by subtracting previous cumulative case count from current cumulative cases count.|
|Cumulative_cases|Integer|Cumulative confirmed cases reported to WHO to date.|
|New_deaths|Integer|New confirmed deaths. Calculated by subtracting previous cumulative deaths from current cumulative deaths.|
|Cumulative_deaths|Integer|Cumulative confirmed deaths reported to WHO to date.|


```python
covid = pd.read_csv('WHO-COVID-19-global-data.csv')
covid.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 198843 entries, 0 to 198842
    Data columns (total 8 columns):
     #   Column             Non-Null Count   Dtype 
    ---  ------             --------------   ----- 
     0   Date_reported      198843 non-null  object
     1   Country_code       198004 non-null  object
     2   Country            198843 non-null  object
     3   WHO_region         198843 non-null  object
     4   New_cases          198843 non-null  int64 
     5   Cumulative_cases   198843 non-null  int64 
     6   New_deaths         198843 non-null  int64 
     7   Cumulative_deaths  198843 non-null  int64 
    dtypes: int64(4), object(4)
    memory usage: 12.1+ MB


### 2) COVID19 확진자 일반화 통계

Skip data description


```python
covid_sum = pd.read_csv('WHO-COVID-19-global-table-data.csv')
covid_sum.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 238 entries, Global to Tuvalu
    Data columns (total 12 columns):
     #   Column                                                        Non-Null Count  Dtype  
    ---  ------                                                        --------------  -----  
     0   Name                                                          237 non-null    object 
     1   WHO Region                                                    238 non-null    int64  
     2   Cases - cumulative total                                      237 non-null    float64
     3   Cases - cumulative total per 100000 population                238 non-null    int64  
     4   Cases - newly reported in last 7 days                         237 non-null    float64
     5   Cases - newly reported in last 7 days per 100000 population   238 non-null    int64  
     6   Cases - newly reported in last 24 hours                       238 non-null    int64  
     7   Deaths - cumulative total                                     237 non-null    float64
     8   Deaths - cumulative total per 100000 population               238 non-null    int64  
     9   Deaths - newly reported in last 7 days                        237 non-null    float64
     10  Deaths - newly reported in last 7 days per 100000 population  238 non-null    int64  
     11  Deaths - newly reported in last 24 hours                      0 non-null      float64
    dtypes: float64(5), int64(6), object(1)
    memory usage: 24.2+ KB


### 3) 백신 데이터

|Field name|Type|Description|
|:----|:----:|:----|
|COUNTRY|String|Country, territory, area|
|ISO3|String|ISO Alpha-3 country code|
|WHO_REGION|String|	WHO regional offices: WHO Member States are grouped into six WHO regions: Regional Office for Africa (AFRO), Regional Office for the Americas (AMRO), Regional Office for South-East Asia (SEARO), Regional Office for Europe (EURO), Regional Office for the Eastern Mediterranean (EMRO), and Regional Office for the Western Pacific (WPRO).|
|DATA_SOURCE|String|Indicates data source: - REPORTING: Data reported by Member States, or sourced from official reports - OWID: Data sourced from Our World in Data: https://ourworldindata.org/covid-vaccinations |
|DATE_UPDATED	|Date|	Date of last update|
|TOTAL_VACCINATIONS|Integer|Cumulative total vaccine doses administered|
|PERSONS_VACCINATED_1PLUS_DOSE	|Decimal|Cumulative number of persons vaccinated with at least one dose|
|TOTAL_VACCINATIONS_PER100	|Integer|	Cumulative total vaccine doses administered per 100 population|
|PERSONS_VACCINATED_1PLUS_DOSE_PER100	|Decimal|	Cumulative persons vaccinated with at least one dose per 100 population|
|PERSONS_FULLY_VACCINATED	|Integer|	Cumulative number of persons fully vaccinated|
|PERSONS_FULLY_VACCINATED_PER100	|Decimal|	Cumulative number of persons fully vaccinated per 100 population|
|VACCINES_USED	|String|	Combined short name of vaccine: “Company - Product name” (see below)|
|FIRST_VACCINE_DATE	|Date|	Date of first vaccinations. Equivalent to start/launch date of the first vaccine administered in a country.|
|NUMBER_VACCINES_TYPES_USED|	Integer|	Number of vaccine types used per country, territory, area|
|PERSONS_BOOSTER_ADD_DOSE	|Integer|	Persons received booster or additional dose|
|PERSONS_BOOSTER_ADD_DOSE_PER100	|Decimal|	Persons received booster or additional dose per 100 population|


```python
vac = pd.read_csv('vaccination-data.csv')
vac.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 228 entries, 0 to 227
    Data columns (total 16 columns):
     #   Column                                Non-Null Count  Dtype  
    ---  ------                                --------------  -----  
     0   COUNTRY                               228 non-null    object 
     1   ISO3                                  228 non-null    object 
     2   WHO_REGION                            228 non-null    object 
     3   DATA_SOURCE                           228 non-null    object 
     4   DATE_UPDATED                          228 non-null    object 
     5   TOTAL_VACCINATIONS                    222 non-null    float64
     6   PERSONS_VACCINATED_1PLUS_DOSE         227 non-null    float64
     7   TOTAL_VACCINATIONS_PER100             222 non-null    float64
     8   PERSONS_VACCINATED_1PLUS_DOSE_PER100  227 non-null    float64
     9   PERSONS_FULLY_VACCINATED              226 non-null    float64
     10  PERSONS_FULLY_VACCINATED_PER100       226 non-null    float64
     11  VACCINES_USED                         225 non-null    object 
     12  FIRST_VACCINE_DATE                    209 non-null    object 
     13  NUMBER_VACCINES_TYPES_USED            225 non-null    float64
     14  PERSONS_BOOSTER_ADD_DOSE              183 non-null    float64
     15  PERSONS_BOOSTER_ADD_DOSE_PER100       183 non-null    float64
    dtypes: float64(9), object(7)
    memory usage: 28.6+ KB


### 4) 백신 메타데이터

|Field name|Type|Description|
|:----:|:----:|:----|
|ISO3	|String|	ISO Alpha-3 country code|
|VACCINE_NAME	|String|Combined short name of vaccine: “Company - Product name” (see below)|
|PRODUCT_NAME	|String|	Name or label of vaccine product, or type of vaccine (if unnamed).|
|COMPANY_NAME	|String|	Marketing authorization holder of vaccine product.|
|FIRST_VACCINE_DATE	|Date|	Date of first vaccinations. Equivalent to start/launch date of the first vaccine administered in a country.|
|AUTHORIZATION_DATE	|Date|	Date vaccine product was authorised for use in the country, territory, area.|
|START_DATE	|Date|	Start/launch date of vaccination with vaccine type (excludes vaccinations during clinical trials).|
|END_DATE	|Date|	End date of vaccine rollout|
|COMMENT	|String|	Comments related to vaccine rollout|
|DATA_SOURCE	|String|	Indicates data source - REPORTING: Data reported by Member States, or sourced from official reports - OWID: Data sourced from Our World in Data: https://ourworldindata.org/covid-vaccinations|



```python
vac_meta = pd.read_csv('vaccination-metadata.csv')
vac_meta.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 887 entries, 0 to 886
    Data columns (total 9 columns):
     #   Column              Non-Null Count  Dtype  
    ---  ------              --------------  -----  
     0   ISO3                887 non-null    object 
     1   VACCINE_NAME        887 non-null    object 
     2   PRODUCT_NAME        886 non-null    object 
     3   COMPANY_NAME        882 non-null    object 
     4   AUTHORIZATION_DATE  390 non-null    object 
     5   START_DATE          624 non-null    object 
     6   END_DATE            0 non-null      float64
     7   COMMENT             0 non-null      float64
     8   DATA_SOURCE         887 non-null    object 
    dtypes: float64(2), object(7)
    memory usage: 62.5+ KB



```python
vac_meta
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
      <th>ISO3</th>
      <th>VACCINE_NAME</th>
      <th>PRODUCT_NAME</th>
      <th>COMPANY_NAME</th>
      <th>AUTHORIZATION_DATE</th>
      <th>START_DATE</th>
      <th>END_DATE</th>
      <th>COMMENT</th>
      <th>DATA_SOURCE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>JEY</td>
      <td>Moderna - mRNA-1273</td>
      <td>mRNA-1273</td>
      <td>Moderna</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>OWID</td>
    </tr>
    <tr>
      <th>1</th>
      <td>JEY</td>
      <td>AstraZeneca - AZD1222</td>
      <td>AZD1222</td>
      <td>AstraZeneca</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>OWID</td>
    </tr>
    <tr>
      <th>2</th>
      <td>JEY</td>
      <td>Pfizer BioNTech - Comirnaty</td>
      <td>Comirnaty</td>
      <td>Pfizer BioNTech</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>OWID</td>
    </tr>
    <tr>
      <th>3</th>
      <td>GGY</td>
      <td>Moderna - mRNA-1273</td>
      <td>mRNA-1273</td>
      <td>Moderna</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>OWID</td>
    </tr>
    <tr>
      <th>4</th>
      <td>GGY</td>
      <td>AstraZeneca - AZD1222</td>
      <td>AZD1222</td>
      <td>AstraZeneca</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>OWID</td>
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
    </tr>
    <tr>
      <th>882</th>
      <td>SYR</td>
      <td>Gamaleya - Sputnik-Light</td>
      <td>Sputnik-Light</td>
      <td>Gamaleya Research Institute</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>REPORTING</td>
    </tr>
    <tr>
      <th>883</th>
      <td>PHL</td>
      <td>Julphar - Hayat-Vax</td>
      <td>Hayat-Vax</td>
      <td>NaN</td>
      <td>2021-08-11</td>
      <td>2021-08-25</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>REPORTING</td>
    </tr>
    <tr>
      <th>884</th>
      <td>SYC</td>
      <td>Julphar - Hayat-Vax</td>
      <td>Hayat-Vax</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>REPORTING</td>
    </tr>
    <tr>
      <th>885</th>
      <td>PRY</td>
      <td>Julphar - Hayat-Vax</td>
      <td>Hayat-Vax</td>
      <td>NaN</td>
      <td>2020-12-30</td>
      <td>2021-05-24</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>REPORTING</td>
    </tr>
    <tr>
      <th>886</th>
      <td>IRN</td>
      <td>Shifa - COVIran Barakat</td>
      <td>COVIran Barakat</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>REPORTING</td>
    </tr>
  </tbody>
</table>
<p>887 rows × 9 columns</p>
</div>



### 5) 국가별 인구, 면적 데이터
https://www.worldometers.info/world-population/population-by-country/

|Field name|Type|Description|
|:----:|:----:|:----|
|Country|String|
|Population|Integer|
|Yearly Change|Decimal|
|Net Change|Integer|
|Density|Integer|
|Land Area|Integer|
|Migrants|Integer|
|Fert Rate|Decimal|Fertility Rate;the average number of children that would be born to a woman over her lifetime|
|Med Age|Decimal|
|Urban Pop|Integer|Urban population|
|World Share|Decimal|


```python
pop = pd.read_csv('World Country Population.csv')
```


```python
pop.info()
#데이터에 결측치가 'N.A.' 텍스트로 입력되어 있어 공백으로 수정하고 불러옴. +%제거
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 235 entries, 0 to 234
    Data columns (total 11 columns):
     #   Column         Non-Null Count  Dtype  
    ---  ------         --------------  -----  
     0   Country        235 non-null    object 
     1   Population     235 non-null    int64  
     2   Yearly Change  235 non-null    float64
     3   Net Change     235 non-null    int64  
     4   Density        235 non-null    int64  
     5   Land Area      235 non-null    int64  
     6   Migrants       201 non-null    float64
     7   Fert Rate      201 non-null    float64
     8   Med Age        201 non-null    float64
     9   Urban Pop      222 non-null    float64
     10  World Share    235 non-null    float64
    dtypes: float64(6), int64(4), object(1)
    memory usage: 20.3+ KB


## 2. 데이터 분석 시나리오

- 국가별 코로나 확진자 변화
    - 일간
    - 주간/월간 합계
    - 평일/주말
    - 인구당 확진자 수(정규화)
- 백신 비율과 확진자 수 관계
- 누적확진자 통계
    - 누적확진자 비율 -> 코로나 확진자 비율이 적은 국가는?
    - 면적당 누적확진자 -> 밀집된 지역일수록 감염자가 많을까?
    - 확진자 대비 사망률


```python
# !pip3 install matplotlib
```

## 3. 데이터 정제 (수정중)
- 데이터 출처가 달라 pop 데이터의 국가명이 covid 데이터의 국가명과 다름.
- 확인해보니 일부 국가명 특수기호, 괄호 등의 문제로 차이가 있음을 확인. -> 하나하나 수정이 필요할 것으로 보임 
    - sorted()
    - drop_duplicates()


```python
covid['Country'].unique()
```




    array(['Afghanistan', 'Albania', 'Algeria', 'American Samoa', 'Andorra',
           'Angola', 'Anguilla', 'Antigua and Barbuda', 'Argentina',
           'Armenia', 'Aruba', 'Australia', 'Austria', 'Azerbaijan',
           'Bahamas', 'Bahrain', 'Bangladesh', 'Barbados', 'Belarus',
           'Belgium', 'Belize', 'Benin', 'Bermuda', 'Bhutan',
           'Bolivia (Plurinational State of)', 'Bonaire',
           'Bosnia and Herzegovina', 'Botswana', 'Brazil',
           'British Virgin Islands', 'Brunei Darussalam', 'Bulgaria',
           'Burkina Faso', 'Burundi', 'Cabo Verde', 'Cambodia', 'Cameroon',
           'Canada', 'Cayman Islands', 'Central African Republic', 'Chad',
           'Chile', 'China', 'Colombia', 'Comoros', 'Congo', 'Cook Islands',
           'Costa Rica', 'Côte d’Ivoire', 'Croatia', 'Cuba', 'Curaçao',
           'Cyprus', 'Czechia', "Democratic People's Republic of Korea",
           'Democratic Republic of the Congo', 'Denmark', 'Djibouti',
           'Dominica', 'Dominican Republic', 'Ecuador', 'Egypt',
           'El Salvador', 'Equatorial Guinea', 'Eritrea', 'Estonia',
           'Eswatini', 'Ethiopia', 'Falkland Islands (Malvinas)',
           'Faroe Islands', 'Fiji', 'Finland', 'France', 'French Guiana',
           'French Polynesia', 'Gabon', 'Gambia', 'Georgia', 'Germany',
           'Ghana', 'Gibraltar', 'Greece', 'Greenland', 'Grenada',
           'Guadeloupe', 'Guam', 'Guatemala', 'Guernsey', 'Guinea',
           'Guinea-Bissau', 'Guyana', 'Haiti', 'Holy See', 'Honduras',
           'Hungary', 'Iceland', 'India', 'Indonesia',
           'Iran (Islamic Republic of)', 'Iraq', 'Ireland', 'Isle of Man',
           'Israel', 'Italy', 'Jamaica', 'Japan', 'Jersey', 'Jordan',
           'Kazakhstan', 'Kenya', 'Kiribati', 'Kosovo[1]', 'Kuwait',
           'Kyrgyzstan', "Lao People's Democratic Republic", 'Latvia',
           'Lebanon', 'Lesotho', 'Liberia', 'Libya', 'Liechtenstein',
           'Lithuania', 'Luxembourg', 'Madagascar', 'Malawi', 'Malaysia',
           'Maldives', 'Mali', 'Malta', 'Marshall Islands', 'Martinique',
           'Mauritania', 'Mauritius', 'Mayotte', 'Mexico',
           'Micronesia (Federated States of)', 'Monaco', 'Mongolia',
           'Montenegro', 'Montserrat', 'Morocco', 'Mozambique', 'Myanmar',
           'Namibia', 'Nauru', 'Nepal', 'Netherlands', 'New Caledonia',
           'New Zealand', 'Nicaragua', 'Niger', 'Nigeria', 'Niue',
           'North Macedonia',
           'Northern Mariana Islands (Commonwealth of the)', 'Norway',
           'occupied Palestinian territory, including east Jerusalem', 'Oman',
           'Other', 'Pakistan', 'Palau', 'Panama', 'Papua New Guinea',
           'Paraguay', 'Peru', 'Philippines', 'Pitcairn Islands', 'Poland',
           'Portugal', 'Puerto Rico', 'Qatar', 'Republic of Korea',
           'Republic of Moldova', 'Réunion', 'Romania', 'Russian Federation',
           'Rwanda', 'Saba', 'Saint Barthélemy', 'Saint Helena',
           'Saint Kitts and Nevis', 'Saint Lucia', 'Saint Martin',
           'Saint Pierre and Miquelon', 'Saint Vincent and the Grenadines',
           'Samoa', 'San Marino', 'Sao Tome and Principe', 'Saudi Arabia',
           'Senegal', 'Serbia', 'Seychelles', 'Sierra Leone', 'Singapore',
           'Sint Eustatius', 'Sint Maarten', 'Slovakia', 'Slovenia',
           'Solomon Islands', 'Somalia', 'South Africa', 'South Sudan',
           'Spain', 'Sri Lanka', 'Sudan', 'Suriname', 'Sweden', 'Switzerland',
           'Syrian Arab Republic', 'Tajikistan', 'Thailand',
           'The United Kingdom', 'Timor-Leste', 'Togo', 'Tokelau', 'Tonga',
           'Trinidad and Tobago', 'Tunisia', 'Turkey', 'Turkmenistan',
           'Turks and Caicos Islands', 'Tuvalu', 'Uganda', 'Ukraine',
           'United Arab Emirates', 'United Republic of Tanzania',
           'United States of America', 'United States Virgin Islands',
           'Uruguay', 'Uzbekistan', 'Vanuatu',
           'Venezuela (Bolivarian Republic of)', 'Viet Nam',
           'Wallis and Futuna', 'Yemen', 'Zambia', 'Zimbabwe'], dtype=object)




```python
sorted(pop['Country'])
```




    ['Afghanistan',
     'Albania',
     'Algeria',
     'American Samoa',
     'Andorra',
     'Angola',
     'Anguilla',
     'Antigua and Barbuda',
     'Argentina',
     'Armenia',
     'Aruba',
     'Australia',
     'Austria',
     'Azerbaijan',
     'Bahamas',
     'Bahrain',
     'Bangladesh',
     'Barbados',
     'Belarus',
     'Belgium',
     'Belize',
     'Benin',
     'Bermuda',
     'Bhutan',
     'Bolivia',
     'Bosnia and Herzegovina',
     'Botswana',
     'Brazil',
     'British Virgin Islands',
     'Brunei',
     'Bulgaria',
     'Burkina Faso',
     'Burundi',
     "C_te d'Ivoire",
     'Cabo Verde',
     'Cambodia',
     'Cameroon',
     'Canada',
     'Caribbean Netherlands',
     'Cayman Islands',
     'Central African Republic',
     'Chad',
     'Channel Islands',
     'Chile',
     'China',
     'Colombia',
     'Comoros',
     'Congo',
     'Cook Islands',
     'Costa Rica',
     'Croatia',
     'Cuba',
     'Cura_ao',
     'Cyprus',
     'Czech Republic (Czechia)',
     'DR Congo',
     'Denmark',
     'Djibouti',
     'Dominica',
     'Dominican Republic',
     'Ecuador',
     'Egypt',
     'El Salvador',
     'Equatorial Guinea',
     'Eritrea',
     'Estonia',
     'Eswatini',
     'Ethiopia',
     'Faeroe Islands',
     'Falkland Islands',
     'Fiji',
     'Finland',
     'France',
     'French Guiana',
     'French Polynesia',
     'Gabon',
     'Gambia',
     'Georgia',
     'Germany',
     'Ghana',
     'Gibraltar',
     'Greece',
     'Greenland',
     'Grenada',
     'Guadeloupe',
     'Guam',
     'Guatemala',
     'Guinea',
     'Guinea-Bissau',
     'Guyana',
     'Haiti',
     'Holy See',
     'Honduras',
     'Hong Kong',
     'Hungary',
     'Iceland',
     'India',
     'Indonesia',
     'Iran',
     'Iraq',
     'Ireland',
     'Isle of Man',
     'Israel',
     'Italy',
     'Jamaica',
     'Japan',
     'Jordan',
     'Kazakhstan',
     'Kenya',
     'Kiribati',
     'Kuwait',
     'Kyrgyzstan',
     'Laos',
     'Latvia',
     'Lebanon',
     'Lesotho',
     'Liberia',
     'Libya',
     'Liechtenstein',
     'Lithuania',
     'Luxembourg',
     'Macao',
     'Madagascar',
     'Malawi',
     'Malaysia',
     'Maldives',
     'Mali',
     'Malta',
     'Marshall Islands',
     'Martinique',
     'Mauritania',
     'Mauritius',
     'Mayotte',
     'Mexico',
     'Micronesia',
     'Moldova',
     'Monaco',
     'Mongolia',
     'Montenegro',
     'Montserrat',
     'Morocco',
     'Mozambique',
     'Myanmar',
     'Namibia',
     'Nauru',
     'Nepal',
     'Netherlands',
     'New Caledonia',
     'New Zealand',
     'Nicaragua',
     'Niger',
     'Nigeria',
     'Niue',
     'North Korea',
     'North Macedonia',
     'Northern Mariana Islands',
     'Norway',
     'Oman',
     'Pakistan',
     'Palau',
     'Panama',
     'Papua New Guinea',
     'Paraguay',
     'Peru',
     'Philippines',
     'Poland',
     'Portugal',
     'Puerto Rico',
     'Qatar',
     'R_union',
     'Romania',
     'Russia',
     'Rwanda',
     'Saint Barthelemy',
     'Saint Helena',
     'Saint Kitts & Nevis',
     'Saint Lucia',
     'Saint Martin',
     'Saint Pierre & Miquelon',
     'Samoa',
     'San Marino',
     'Sao Tome & Principe',
     'Saudi Arabia',
     'Senegal',
     'Serbia',
     'Seychelles',
     'Sierra Leone',
     'Singapore',
     'Sint Maarten',
     'Slovakia',
     'Slovenia',
     'Solomon Islands',
     'Somalia',
     'South Africa',
     'South Korea',
     'South Sudan',
     'Spain',
     'Sri Lanka',
     'St. Vincent & Grenadines',
     'State of Palestine',
     'Sudan',
     'Suriname',
     'Sweden',
     'Switzerland',
     'Syria',
     'Taiwan',
     'Tajikistan',
     'Tanzania',
     'Thailand',
     'Timor-Leste',
     'Togo',
     'Tokelau',
     'Tonga',
     'Trinidad and Tobago',
     'Tunisia',
     'Turkey',
     'Turkmenistan',
     'Turks and Caicos',
     'Tuvalu',
     'U.S. Virgin Islands',
     'Uganda',
     'Ukraine',
     'United Arab Emirates',
     'United Kingdom',
     'United States',
     'Uruguay',
     'Uzbekistan',
     'Vanuatu',
     'Venezuela',
     'Vietnam',
     'Wallis & Futuna',
     'Western Sahara',
     'Yemen',
     'Zambia',
     'Zimbabwe']




```python
print(len(covid['Country'].drop_duplicates()),len(pop['Country']))
# 2개 차이가 있긴 한데 제거 후 사용할 예정
```

    237 235


## 4. 데이터 만들기

### 4-1) 국가별 코로나 확진자 변화
    - 일간
    - 주간/월간 합계
    - 평일/주말
    - 인구당 확진자 수


```python
covid
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
      <th>Date_reported</th>
      <th>Country_code</th>
      <th>Country</th>
      <th>WHO_region</th>
      <th>New_cases</th>
      <th>Cumulative_cases</th>
      <th>New_deaths</th>
      <th>Cumulative_deaths</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-01-03</td>
      <td>AF</td>
      <td>Afghanistan</td>
      <td>EMRO</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-01-04</td>
      <td>AF</td>
      <td>Afghanistan</td>
      <td>EMRO</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-01-05</td>
      <td>AF</td>
      <td>Afghanistan</td>
      <td>EMRO</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-01-06</td>
      <td>AF</td>
      <td>Afghanistan</td>
      <td>EMRO</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-01-07</td>
      <td>AF</td>
      <td>Afghanistan</td>
      <td>EMRO</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
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
    </tr>
    <tr>
      <th>198838</th>
      <td>2022-04-16</td>
      <td>ZW</td>
      <td>Zimbabwe</td>
      <td>AFRO</td>
      <td>29</td>
      <td>247237</td>
      <td>0</td>
      <td>5462</td>
    </tr>
    <tr>
      <th>198839</th>
      <td>2022-04-17</td>
      <td>ZW</td>
      <td>Zimbabwe</td>
      <td>AFRO</td>
      <td>24</td>
      <td>247261</td>
      <td>0</td>
      <td>5462</td>
    </tr>
    <tr>
      <th>198840</th>
      <td>2022-04-18</td>
      <td>ZW</td>
      <td>Zimbabwe</td>
      <td>AFRO</td>
      <td>17</td>
      <td>247278</td>
      <td>1</td>
      <td>5463</td>
    </tr>
    <tr>
      <th>198841</th>
      <td>2022-04-19</td>
      <td>ZW</td>
      <td>Zimbabwe</td>
      <td>AFRO</td>
      <td>19</td>
      <td>247297</td>
      <td>1</td>
      <td>5464</td>
    </tr>
    <tr>
      <th>198842</th>
      <td>2022-04-20</td>
      <td>ZW</td>
      <td>Zimbabwe</td>
      <td>AFRO</td>
      <td>39</td>
      <td>247336</td>
      <td>2</td>
      <td>5466</td>
    </tr>
  </tbody>
</table>
<p>198843 rows × 8 columns</p>
</div>




```python
# 날짜 컬럼을 날짜 데이터형으로 만들기
covid['Date_reported']=pd.to_datetime(covid['Date_reported'])
```


```python
# 년 만들기
covid['Year'] = covid['Date_reported'].apply(lambda x : x.year)
```


```python
# 월 컬럼 만들기
covid['Month'] = covid['Date_reported'].apply(lambda x : x.month)
```


```python
# 주 번호 컬럼 만들기
covid['Week'] = covid['Date_reported'].apply(lambda x : x.week)
```


```python
# 요일 컬럼과 주말, 평일 구분 만들기
# 0이 월요일, 6이 일요일
covid['Weekday']=covid['Date_reported'].apply(lambda x : x.weekday())
```


```python
#평일 0, 주말 1
covid['Weekend']=covid['Weekday'].apply(lambda x : 0 if x<=4 else 1)
```

### 일간 확진자수


```python
import matplotlib.pyplot as plt
```


```python
covid[['Date_reported','Country','New_cases','Cumulative_cases', 'New_deaths','Cumulative_deaths']]
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
      <th>Date_reported</th>
      <th>Country</th>
      <th>New_cases</th>
      <th>Cumulative_cases</th>
      <th>New_deaths</th>
      <th>Cumulative_deaths</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-01-03</td>
      <td>Afghanistan</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-01-04</td>
      <td>Afghanistan</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-01-05</td>
      <td>Afghanistan</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-01-06</td>
      <td>Afghanistan</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-01-07</td>
      <td>Afghanistan</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>198838</th>
      <td>2022-04-16</td>
      <td>Zimbabwe</td>
      <td>29</td>
      <td>247237</td>
      <td>0</td>
      <td>5462</td>
    </tr>
    <tr>
      <th>198839</th>
      <td>2022-04-17</td>
      <td>Zimbabwe</td>
      <td>24</td>
      <td>247261</td>
      <td>0</td>
      <td>5462</td>
    </tr>
    <tr>
      <th>198840</th>
      <td>2022-04-18</td>
      <td>Zimbabwe</td>
      <td>17</td>
      <td>247278</td>
      <td>1</td>
      <td>5463</td>
    </tr>
    <tr>
      <th>198841</th>
      <td>2022-04-19</td>
      <td>Zimbabwe</td>
      <td>19</td>
      <td>247297</td>
      <td>1</td>
      <td>5464</td>
    </tr>
    <tr>
      <th>198842</th>
      <td>2022-04-20</td>
      <td>Zimbabwe</td>
      <td>39</td>
      <td>247336</td>
      <td>2</td>
      <td>5466</td>
    </tr>
  </tbody>
</table>
<p>198843 rows × 6 columns</p>
</div>



### 확진자/사망자 발생 추이


```python
case_var=covid[['Date_reported','New_cases','New_deaths']].groupby('Date_reported').sum().reset_index()
```


```python
# 그래프 그리기 - 한꺼번에 보기

fig, ax1 = plt.subplots()

ax1.plot(case_var['Date_reported'],case_var['New_cases'],'-', 
         color='green', linewidth=1, alpha=0.7, label='Cases')
ax1.set_xlabel('Date')
ax1.set_ylabel('Cases')
ax1.tick_params(axis='both', direction='in')

ax2 = ax1.twinx()
ax2.bar(case_var['Date_reported'],case_var['New_deaths'], 
        color='deeppink', label='Deaths', alpha=0.7, width=0.5)
ax2.set_ylabel('Deaths')
ax2.tick_params(axis='y', direction='in')

# cases 그래프를 위에 그려지게 만들기
ax1.set_zorder(ax2.get_zorder() + 10)
ax1.patch.set_visible(False)

plt.show()
```


    
![graph](assets/img/post/GlobalCOVID19/output_44_0.png'
    


결론 : 확진자가 늘어도 사망률은 오히려 더 낮아지고 있음

### 특정 일 최대 확진자 발생국


```python
#2022년 4월 22일 최다 확진자 발생국 TOP 10
covid[covid['Date_reported']=="2022-04-20"].sort_values('New_cases', ascending=False)\
.head(10)[['Country','New_cases']]
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
      <th>Country</th>
      <th>New_cases</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>144307</th>
      <td>Republic of Korea</td>
      <td>111319</td>
    </tr>
    <tr>
      <th>66280</th>
      <td>Germany</td>
      <td>107510</td>
    </tr>
    <tr>
      <th>190452</th>
      <td>United States of America</td>
      <td>48510</td>
    </tr>
    <tr>
      <th>10067</th>
      <td>Australia</td>
      <td>38925</td>
    </tr>
    <tr>
      <th>88933</th>
      <td>Japan</td>
      <td>37631</td>
    </tr>
    <tr>
      <th>87255</th>
      <td>Italy</td>
      <td>27326</td>
    </tr>
    <tr>
      <th>31881</th>
      <td>Canada</td>
      <td>22747</td>
    </tr>
    <tr>
      <th>177028</th>
      <td>Thailand</td>
      <td>20455</td>
    </tr>
    <tr>
      <th>177867</th>
      <td>The United Kingdom</td>
      <td>15822</td>
    </tr>
    <tr>
      <th>195486</th>
      <td>Viet Nam</td>
      <td>13500</td>
    </tr>
  </tbody>
</table>
</div>



### 주간 확진자수


```python
Weekcase=covid[['Date_reported','Country','New_cases', 'New_deaths','Year','Week']].groupby(['Country','Year','Week']).sum()
Weekcum=Weekcase.groupby(level=1).cumsum().reset_index()
```


```python
Weekcum.columns=['Country','Year',"Week","cum_cases","cum_deaths"]
```


```python
Weekdata=pd.merge(Weekcase.reset_index(), Weekcum, how="inner")
Weekdata
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
      <th>Country</th>
      <th>Year</th>
      <th>Week</th>
      <th>New_cases</th>
      <th>New_deaths</th>
      <th>cum_cases</th>
      <th>cum_deaths</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
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
    </tr>
    <tr>
      <th>29146</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>13</td>
      <td>282</td>
      <td>6</td>
      <td>217005128</td>
      <td>759521</td>
    </tr>
    <tr>
      <th>29147</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>14</td>
      <td>537</td>
      <td>0</td>
      <td>217005665</td>
      <td>759521</td>
    </tr>
    <tr>
      <th>29148</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>15</td>
      <td>330</td>
      <td>1</td>
      <td>217005995</td>
      <td>759522</td>
    </tr>
    <tr>
      <th>29149</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>16</td>
      <td>259</td>
      <td>1</td>
      <td>217006254</td>
      <td>759523</td>
    </tr>
    <tr>
      <th>29150</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>52</td>
      <td>355</td>
      <td>11</td>
      <td>217006609</td>
      <td>759534</td>
    </tr>
  </tbody>
</table>
<p>29151 rows × 7 columns</p>
</div>



### 월간 확진자수


```python
Monthcase=covid[['Date_reported','Country','New_cases', 'New_deaths','Year','Month']].groupby(['Country','Year','Month']).sum()
Monthcum=Monthcase.groupby(level=1).cumsum().reset_index()
```


```python
Monthcum.columns=['Country','Year',"Month","cum_cases","cum_deaths"]
```


```python
Monthdata=pd.merge(Monthcase.reset_index(), Monthcum, how="inner")
Monthdata
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
      <th>Country</th>
      <th>Year</th>
      <th>Month</th>
      <th>New_cases</th>
      <th>New_deaths</th>
      <th>cum_cases</th>
      <th>cum_deaths</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>2</td>
      <td>5</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>3</td>
      <td>161</td>
      <td>4</td>
      <td>166</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>4</td>
      <td>1661</td>
      <td>56</td>
      <td>1827</td>
      <td>60</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Afghanistan</td>
      <td>2020</td>
      <td>5</td>
      <td>13353</td>
      <td>194</td>
      <td>15180</td>
      <td>254</td>
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
    </tr>
    <tr>
      <th>6631</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2021</td>
      <td>12</td>
      <td>9864</td>
      <td>132</td>
      <td>204113841</td>
      <td>3518055</td>
    </tr>
    <tr>
      <th>6632</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>1</td>
      <td>54413</td>
      <td>167</td>
      <td>216874449</td>
      <td>758975</td>
    </tr>
    <tr>
      <th>6633</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>2</td>
      <td>121231</td>
      <td>403</td>
      <td>216995680</td>
      <td>759378</td>
    </tr>
    <tr>
      <th>6634</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>3</td>
      <td>9803</td>
      <td>154</td>
      <td>217005483</td>
      <td>759532</td>
    </tr>
    <tr>
      <th>6635</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>4</td>
      <td>1126</td>
      <td>2</td>
      <td>217006609</td>
      <td>759534</td>
    </tr>
  </tbody>
</table>
<p>6636 rows × 7 columns</p>
</div>



### 국가별 최대 유행기


```python
# 국가별 확진자 최대인 행 추출
max_cases = Monthdata.iloc[Monthdata.groupby(['Country'])['New_cases'].idxmax()][['Country','Year','Month']]
max_cases
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
      <th>Country</th>
      <th>Year</th>
      <th>Month</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>17</th>
      <td>Afghanistan</td>
      <td>2021</td>
      <td>6</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Albania</td>
      <td>2022</td>
      <td>1</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Algeria</td>
      <td>2022</td>
      <td>1</td>
    </tr>
    <tr>
      <th>110</th>
      <td>American Samoa</td>
      <td>2022</td>
      <td>3</td>
    </tr>
    <tr>
      <th>136</th>
      <td>Andorra</td>
      <td>2022</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>6510</th>
      <td>Wallis and Futuna</td>
      <td>2021</td>
      <td>3</td>
    </tr>
    <tr>
      <th>6539</th>
      <td>Yemen</td>
      <td>2021</td>
      <td>4</td>
    </tr>
    <tr>
      <th>6569</th>
      <td>Zambia</td>
      <td>2021</td>
      <td>6</td>
    </tr>
    <tr>
      <th>6603</th>
      <td>Zimbabwe</td>
      <td>2021</td>
      <td>12</td>
    </tr>
    <tr>
      <th>6633</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>2022</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
<p>237 rows × 3 columns</p>
</div>




```python
max_count=max_cases.groupby(['Year','Month']).count().reset_index()
max_count['Year']=max_count['Year'].astype('string')
max_count['Month']=max_count['Month'].astype('string')
max_count
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
      <th>Year</th>
      <th>Month</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020</td>
      <td>1</td>
      <td>7</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020</td>
      <td>5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020</td>
      <td>6</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020</td>
      <td>7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2020</td>
      <td>8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2020</td>
      <td>10</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2020</td>
      <td>12</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2021</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2021</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2021</td>
      <td>3</td>
      <td>3</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2021</td>
      <td>4</td>
      <td>6</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2021</td>
      <td>5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2021</td>
      <td>6</td>
      <td>9</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2021</td>
      <td>7</td>
      <td>10</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2021</td>
      <td>8</td>
      <td>11</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2021</td>
      <td>9</td>
      <td>3</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2021</td>
      <td>10</td>
      <td>6</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2021</td>
      <td>12</td>
      <td>10</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2022</td>
      <td>1</td>
      <td>94</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2022</td>
      <td>2</td>
      <td>39</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2022</td>
      <td>3</td>
      <td>17</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2022</td>
      <td>4</td>
      <td>7</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(20,5))
plt.plot(max_count['Year']+max_count['Month'],
         max_count['Country'], '')
plt.title('Monthly Max Cases Country count')
plt.xlabel('Date')
plt.ylabel('Count')
plt.show()

# 2022년 초에 확진자가 최대인 것은, 일부 국가에서 크게 나온 것이 아니라 전세계적 트렌드인 것으로 추정됨
```


    
![graph](assets/img/post/GlobalCOVID19/output_59_0.png'
    


### 주별 평일/주말 확진자 수


```python
weekday = covid[['Country','Year','Week','Weekend',
                 'New_cases','New_deaths']].groupby(['Country','Year','Week','Weekend',]).mean().reset_index()
```


```python
weekday = weekday.pivot(index=['Country','Year','Week'], columns='Weekend',
                        values=['New_cases','New_deaths']).reset_index()
```


```python
weekday.columns = ['Country','Year','Week',
                   'Weekday_New_cases','Weekend_New_cases','Weekday_New_deaths','Weekend_New_deaths']
```


```python
# 주말 대비 평일엔 n배 확진자가 나온다가 원하는 결론.
# 확진자 수 0 문제로 인해 답이 안나옴
weekday=weekday.fillna(0)
weekday['New_cases_var']=weekday['Weekday_New_cases']/weekday['Weekend_New_cases']
weekday['New_deaths_var']=weekday['Weekday_New_deaths']/weekday['Weekend_New_deaths']
```


```python
weekday=weekday.fillna(0)
weekday=weekday.replace(float("inf"),0)
```


```python
dayend=weekday[['Country','New_cases_var','New_deaths_var']]\
        [(weekday['New_cases_var']>0) & (weekday['New_deaths_var']>0)].groupby('Country').mean().reset_index()
```


```python
dayend
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
      <th>Country</th>
      <th>New_cases_var</th>
      <th>New_deaths_var</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>1.607277</td>
      <td>1.487356</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>0.993542</td>
      <td>1.146033</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Algeria</td>
      <td>1.014935</td>
      <td>1.166696</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Andorra</td>
      <td>1.218785</td>
      <td>0.757895</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Angola</td>
      <td>1.060907</td>
      <td>1.249839</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>208</th>
      <td>Wallis and Futuna</td>
      <td>1.333333</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>209</th>
      <td>Yemen</td>
      <td>1.630817</td>
      <td>1.465935</td>
    </tr>
    <tr>
      <th>210</th>
      <td>Zambia</td>
      <td>0.892779</td>
      <td>1.104105</td>
    </tr>
    <tr>
      <th>211</th>
      <td>Zimbabwe</td>
      <td>1.222184</td>
      <td>1.666615</td>
    </tr>
    <tr>
      <th>212</th>
      <td>occupied Palestinian territory, including east...</td>
      <td>1.230396</td>
      <td>1.192464</td>
    </tr>
  </tbody>
</table>
<p>213 rows × 3 columns</p>
</div>




```python
plt.scatter(dayend['New_cases_var'],dayend['New_deaths_var'],alpha=0.5)
plt.xlabel('cases-variation')
plt.ylabel('deaths-variation')
plt.show()

# 1보다 넘는게 몇이나 있는지 잘파악이 안되네.... 
# 대충 눈으로는 주말이 더 많이 나오는 편인 것 같긴 하다
# 어떻게 봐야 더 정확하게 알 수 있는지는 고민이 필요함.
```


    
![graph](assets/img/post/GlobalCOVID19/output_68_0.png)
    


### 인구당 확진자수


```python
covid_sum['Country']=covid_sum.index
```


```python
# 10만명 당 확진자수 TOP 10
covid_sum[['Name','Cases - cumulative total per 100000 population']]\
.sort_values('Cases - cumulative total per 100000 population', ascending=False).head(10)

#인구당 확진자 수는 확실히 선진국이 높음 -> Why? 
#저개발국가는 검사 자체를 적게 한다...? 경제 활동이 활발한 선진국에서 더 잘 퍼졌다?
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
      <th>Name</th>
      <th>Cases - cumulative total per 100000 population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Global</th>
      <td>NaN</td>
      <td>4760069</td>
    </tr>
    <tr>
      <th>Republic of Korea</th>
      <td>Western Pacific</td>
      <td>752602</td>
    </tr>
    <tr>
      <th>France</th>
      <td>Europe</td>
      <td>624536</td>
    </tr>
    <tr>
      <th>Germany</th>
      <td>Europe</td>
      <td>572348</td>
    </tr>
    <tr>
      <th>Italy</th>
      <td>Europe</td>
      <td>353193</td>
    </tr>
    <tr>
      <th>Japan</th>
      <td>Western Pacific</td>
      <td>316134</td>
    </tr>
    <tr>
      <th>Australia</th>
      <td>Western Pacific</td>
      <td>312941</td>
    </tr>
    <tr>
      <th>United States of America</th>
      <td>Americas</td>
      <td>255634</td>
    </tr>
    <tr>
      <th>Viet Nam</th>
      <td>Western Pacific</td>
      <td>216355</td>
    </tr>
    <tr>
      <th>The United Kingdom</th>
      <td>Europe</td>
      <td>166033</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 대륙별 top 5
head5 = covid_sum[['Name','Cases - cumulative total per 100000 population']].groupby('Name').head(5)
head5.sort_values(by=['Name','Cases - cumulative total per 100000 population'],ascending=False)
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
      <th>Name</th>
      <th>Cases - cumulative total per 100000 population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Republic of Korea</th>
      <td>Western Pacific</td>
      <td>752602</td>
    </tr>
    <tr>
      <th>Japan</th>
      <td>Western Pacific</td>
      <td>316134</td>
    </tr>
    <tr>
      <th>Australia</th>
      <td>Western Pacific</td>
      <td>312941</td>
    </tr>
    <tr>
      <th>Viet Nam</th>
      <td>Western Pacific</td>
      <td>216355</td>
    </tr>
    <tr>
      <th>Malaysia</th>
      <td>Western Pacific</td>
      <td>59675</td>
    </tr>
    <tr>
      <th>Thailand</th>
      <td>South-East Asia</td>
      <td>135430</td>
    </tr>
    <tr>
      <th>India</th>
      <td>South-East Asia</td>
      <td>9578</td>
    </tr>
    <tr>
      <th>Indonesia</th>
      <td>South-East Asia</td>
      <td>5101</td>
    </tr>
    <tr>
      <th>Bangladesh</th>
      <td>South-East Asia</td>
      <td>278</td>
    </tr>
    <tr>
      <th>Nepal</th>
      <td>South-East Asia</td>
      <td>62</td>
    </tr>
    <tr>
      <th>Other</th>
      <td>Other</td>
      <td>0</td>
    </tr>
    <tr>
      <th>France</th>
      <td>Europe</td>
      <td>624536</td>
    </tr>
    <tr>
      <th>Germany</th>
      <td>Europe</td>
      <td>572348</td>
    </tr>
    <tr>
      <th>Italy</th>
      <td>Europe</td>
      <td>353193</td>
    </tr>
    <tr>
      <th>The United Kingdom</th>
      <td>Europe</td>
      <td>166033</td>
    </tr>
    <tr>
      <th>Russian Federation</th>
      <td>Europe</td>
      <td>71407</td>
    </tr>
    <tr>
      <th>Iran (Islamic Republic of)</th>
      <td>Eastern Mediterranean</td>
      <td>13283</td>
    </tr>
    <tr>
      <th>Iraq</th>
      <td>Eastern Mediterranean</td>
      <td>1205</td>
    </tr>
    <tr>
      <th>Pakistan</th>
      <td>Eastern Mediterranean</td>
      <td>683</td>
    </tr>
    <tr>
      <th>Jordan</th>
      <td>Eastern Mediterranean</td>
      <td>475</td>
    </tr>
    <tr>
      <th>Morocco</th>
      <td>Eastern Mediterranean</td>
      <td>332</td>
    </tr>
    <tr>
      <th>United States of America</th>
      <td>Americas</td>
      <td>255634</td>
    </tr>
    <tr>
      <th>Brazil</th>
      <td>Americas</td>
      <td>99883</td>
    </tr>
    <tr>
      <th>Argentina</th>
      <td>Americas</td>
      <td>15554</td>
    </tr>
    <tr>
      <th>Colombia</th>
      <td>Americas</td>
      <td>1456</td>
    </tr>
    <tr>
      <th>Mexico</th>
      <td>Americas</td>
      <td>1280</td>
    </tr>
    <tr>
      <th>South Africa</th>
      <td>Africa</td>
      <td>9663</td>
    </tr>
    <tr>
      <th>Zambia</th>
      <td>Africa</td>
      <td>836</td>
    </tr>
    <tr>
      <th>Ethiopia</th>
      <td>Africa</td>
      <td>197</td>
    </tr>
    <tr>
      <th>Kenya</th>
      <td>Africa</td>
      <td>47</td>
    </tr>
    <tr>
      <th>Réunion</th>
      <td>Africa</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



### 확진자 수 대비 사망률


```python
covid_sum['deaths/cases']=covid_sum['Deaths - cumulative total']*100/covid_sum['Cases - cumulative total']
```


```python
covid_sum[['Name','deaths/cases']].sort_values('deaths/cases',ascending=False).head(10)
# 확실해 저개발국가가 사망률이 높다.
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
      <th>Name</th>
      <th>deaths/cases</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Yemen</th>
      <td>Eastern Mediterranean</td>
      <td>18.176312</td>
    </tr>
    <tr>
      <th>Sudan</th>
      <td>Eastern Mediterranean</td>
      <td>7.940017</td>
    </tr>
    <tr>
      <th>Peru</th>
      <td>Americas</td>
      <td>5.979499</td>
    </tr>
    <tr>
      <th>Mexico</th>
      <td>Americas</td>
      <td>5.655694</td>
    </tr>
    <tr>
      <th>Syrian Arab Republic</th>
      <td>Eastern Mediterranean</td>
      <td>5.646887</td>
    </tr>
    <tr>
      <th>Somalia</th>
      <td>Eastern Mediterranean</td>
      <td>5.138530</td>
    </tr>
    <tr>
      <th>Egypt</th>
      <td>Eastern Mediterranean</td>
      <td>4.773340</td>
    </tr>
    <tr>
      <th>Afghanistan</th>
      <td>Eastern Mediterranean</td>
      <td>4.302231</td>
    </tr>
    <tr>
      <th>Bosnia and Herzegovina</th>
      <td>Europe</td>
      <td>4.183319</td>
    </tr>
    <tr>
      <th>Ecuador</th>
      <td>Americas</td>
      <td>4.098162</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 인구당 사망자 수를 보면 선진국이 많은 것과 상반된 모습
covid_sum[['Name','Deaths - cumulative total per 100000 population']]\
.sort_values('Deaths - cumulative total per 100000 population',ascending=False).head(11)
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
      <th>Name</th>
      <th>Deaths - cumulative total per 100000 population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Global</th>
      <td>NaN</td>
      <td>15727</td>
    </tr>
    <tr>
      <th>United States of America</th>
      <td>Americas</td>
      <td>3030</td>
    </tr>
    <tr>
      <th>Russian Federation</th>
      <td>Europe</td>
      <td>1629</td>
    </tr>
    <tr>
      <th>Republic of Korea</th>
      <td>Western Pacific</td>
      <td>1486</td>
    </tr>
    <tr>
      <th>Thailand</th>
      <td>South-East Asia</td>
      <td>868</td>
    </tr>
    <tr>
      <th>Italy</th>
      <td>Europe</td>
      <td>861</td>
    </tr>
    <tr>
      <th>Brazil</th>
      <td>Americas</td>
      <td>699</td>
    </tr>
    <tr>
      <th>France</th>
      <td>Europe</td>
      <td>579</td>
    </tr>
    <tr>
      <th>The United Kingdom</th>
      <td>Europe</td>
      <td>534</td>
    </tr>
    <tr>
      <th>Greece</th>
      <td>Europe</td>
      <td>427</td>
    </tr>
    <tr>
      <th>Japan</th>
      <td>Western Pacific</td>
      <td>337</td>
    </tr>
  </tbody>
</table>
</div>



### 4-2) 백신 접종 비율과 확진자 수 관계

- 백신 접종 비율(fully/booster)과 확진자 비율
- 백신 시작 날짜와 확진자/사망자 비율 -> 백신을 적극적으로 도입한 국가가 방역을 더 잘했는가?
- 백신 종류과 확진자/사망자 비율

#### 백신이란? 
 인체에 코로나19 바이러스가 침입했을 때 활성화되어 바이러스나 바이러스에 감염된 세포를 제거합니다. 이러한 기전으로 코로나19 예방접종은 코로나19 감염의 위험을 줄여주고, 중증 환자발생이나 사망을 예방합니다. -질병관리청 <br>
 그렇다면 백신 접종자 비율이 높은 국가는 확진자 수 및 사망자 수가 적은가?


```python
vac.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 228 entries, 0 to 227
    Data columns (total 16 columns):
     #   Column                                Non-Null Count  Dtype  
    ---  ------                                --------------  -----  
     0   COUNTRY                               228 non-null    object 
     1   ISO3                                  228 non-null    object 
     2   WHO_REGION                            228 non-null    object 
     3   DATA_SOURCE                           228 non-null    object 
     4   DATE_UPDATED                          228 non-null    object 
     5   TOTAL_VACCINATIONS                    222 non-null    float64
     6   PERSONS_VACCINATED_1PLUS_DOSE         227 non-null    float64
     7   TOTAL_VACCINATIONS_PER100             222 non-null    float64
     8   PERSONS_VACCINATED_1PLUS_DOSE_PER100  227 non-null    float64
     9   PERSONS_FULLY_VACCINATED              226 non-null    float64
     10  PERSONS_FULLY_VACCINATED_PER100       226 non-null    float64
     11  VACCINES_USED                         225 non-null    object 
     12  FIRST_VACCINE_DATE                    209 non-null    object 
     13  NUMBER_VACCINES_TYPES_USED            225 non-null    float64
     14  PERSONS_BOOSTER_ADD_DOSE              183 non-null    float64
     15  PERSONS_BOOSTER_ADD_DOSE_PER100       183 non-null    float64
    dtypes: float64(9), object(7)
    memory usage: 28.6+ KB


### 백신 접종자 비율과 확진자-사망자 수의 관계


```python
## fully/booster vaccined ratio
vac_ratio = vac[['COUNTRY','PERSONS_FULLY_VACCINATED_PER100','PERSONS_BOOSTER_ADD_DOSE_PER100']]\
.sort_values('PERSONS_FULLY_VACCINATED_PER100', ascending=False)
```


```python
# 백신 접종자 비율과 확진자/사망자 비율
vac_case = pd.merge(covid_sum[['Country',
                    'Cases - cumulative total per 100000 population',
                    'Deaths - cumulative total per 100000 population']],
        vac_ratio, left_on='Country', right_on='COUNTRY')
```


```python
plt.plot(vac_case['Cases - cumulative total per 100000 population']/10000,
         vac_case['PERSONS_FULLY_VACCINATED_PER100'], 'ro',alpha=0.5)
plt.xlabel('Cases')
plt.ylabel('Vaccinated')
plt.show()

# 관련이 없어보이는데, 다르게 말하면 확진자 많은 국가들이 백신을 적극적으로 도입했을 수도 있음.
```


    
![graph](assets/img/post/GlobalCOVID19/output_84_0.png)
    


### 백신 도입 날짜와 확진자-사망자 수의 관계


```python
# 백신 시작 날짜와 확진자/사망자 비율
vac_start=pd.merge(vac_case, vac[['COUNTRY','FIRST_VACCINE_DATE']]).sort_values('FIRST_VACCINE_DATE')
```


```python
plt.figure(figsize=(20,5))
plt.plot(vac_start['FIRST_VACCINE_DATE'].apply(lambda x : str(x)),
         vac_start['Cases - cumulative total per 100000 population']/10000, 'ro')
plt.xlabel('Date')
plt.ylabel('Cumulative Cases')
plt.xticks([0,10,20,30,40,50,60,70,80,90,100])
plt.show()

# 이것도 딱히 늦게 도입했다고 해서 더 확진자가 많다고는 말할 수 없다.
```


    
![graph](assets/img/post/GlobalCOVID19/output_87_0.png)
    


### 대륙별 백신 첫 도입국가


```python
vac
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
      <th>COUNTRY</th>
      <th>ISO3</th>
      <th>WHO_REGION</th>
      <th>DATA_SOURCE</th>
      <th>DATE_UPDATED</th>
      <th>TOTAL_VACCINATIONS</th>
      <th>PERSONS_VACCINATED_1PLUS_DOSE</th>
      <th>TOTAL_VACCINATIONS_PER100</th>
      <th>PERSONS_VACCINATED_1PLUS_DOSE_PER100</th>
      <th>PERSONS_FULLY_VACCINATED</th>
      <th>PERSONS_FULLY_VACCINATED_PER100</th>
      <th>VACCINES_USED</th>
      <th>FIRST_VACCINE_DATE</th>
      <th>NUMBER_VACCINES_TYPES_USED</th>
      <th>PERSONS_BOOSTER_ADD_DOSE</th>
      <th>PERSONS_BOOSTER_ADD_DOSE_PER100</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>EMRO</td>
      <td>REPORTING</td>
      <td>2022-04-18</td>
      <td>5948889.0</td>
      <td>5253929.0</td>
      <td>15.282</td>
      <td>13.496</td>
      <td>4603187.0</td>
      <td>11.825</td>
      <td>Beijing CNBG - BBIBP-CorV,Janssen - Ad26.COV 2...</td>
      <td>2021-02-22</td>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>ALB</td>
      <td>EURO</td>
      <td>REPORTING</td>
      <td>2022-04-10</td>
      <td>2815381.0</td>
      <td>1306364.0</td>
      <td>97.800</td>
      <td>45.902</td>
      <td>1229560.0</td>
      <td>43.204</td>
      <td>AstraZeneca - Vaxzevria,Gamaleya - Gam-Covid-V...</td>
      <td>2021-01-13</td>
      <td>5.0</td>
      <td>279457.0</td>
      <td>9.819</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>AFRO</td>
      <td>REPORTING</td>
      <td>2022-03-23</td>
      <td>NaN</td>
      <td>7481277.0</td>
      <td>NaN</td>
      <td>17.061</td>
      <td>6147178.0</td>
      <td>14.018</td>
      <td>Beijing CNBG - BBIBP-CorV,Gamaleya - Gam-Covid...</td>
      <td>2021-01-30</td>
      <td>4.0</td>
      <td>514063.0</td>
      <td>1.172</td>
    </tr>
    <tr>
      <th>3</th>
      <td>American Samoa</td>
      <td>ASM</td>
      <td>WPRO</td>
      <td>REPORTING</td>
      <td>2022-03-17</td>
      <td>97346.0</td>
      <td>43637.0</td>
      <td>176.361</td>
      <td>79.057</td>
      <td>39137.0</td>
      <td>70.904</td>
      <td>Janssen - Ad26.COV 2-S,Moderna - Spikevax,Pfiz...</td>
      <td>2020-12-21</td>
      <td>3.0</td>
      <td>15135.0</td>
      <td>27.420</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Andorra</td>
      <td>AND</td>
      <td>EURO</td>
      <td>REPORTING</td>
      <td>2022-04-10</td>
      <td>152301.0</td>
      <td>57850.0</td>
      <td>197.100</td>
      <td>75.942</td>
      <td>53406.0</td>
      <td>70.108</td>
      <td>AstraZeneca - Vaxzevria,Moderna - Spikevax,Pfi...</td>
      <td>2021-01-20</td>
      <td>3.0</td>
      <td>41045.0</td>
      <td>53.881</td>
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
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>223</th>
      <td>Viet Nam</td>
      <td>VNM</td>
      <td>WPRO</td>
      <td>REPORTING</td>
      <td>2022-04-07</td>
      <td>208244568.0</td>
      <td>80195986.0</td>
      <td>213.938</td>
      <td>82.389</td>
      <td>76872167.0</td>
      <td>78.974</td>
      <td>AstraZeneca - Vaxzevria,Beijing CNBG - BBIBP-C...</td>
      <td>2021-03-08</td>
      <td>6.0</td>
      <td>15001180.0</td>
      <td>15.411</td>
    </tr>
    <tr>
      <th>224</th>
      <td>Wallis and Futuna</td>
      <td>WLF</td>
      <td>WPRO</td>
      <td>REPORTING</td>
      <td>2022-04-01</td>
      <td>16287.0</td>
      <td>6576.0</td>
      <td>144.825</td>
      <td>58.474</td>
      <td>6608.0</td>
      <td>58.759</td>
      <td>Moderna - Spikevax</td>
      <td>2021-03-19</td>
      <td>1.0</td>
      <td>3103.0</td>
      <td>27.592</td>
    </tr>
    <tr>
      <th>225</th>
      <td>Yemen</td>
      <td>YEM</td>
      <td>EMRO</td>
      <td>REPORTING</td>
      <td>2022-04-06</td>
      <td>814569.0</td>
      <td>649673.0</td>
      <td>2.731</td>
      <td>2.178</td>
      <td>411501.0</td>
      <td>1.380</td>
      <td>Janssen - Ad26.COV 2-S,SII - Covishield,Sinova...</td>
      <td>2021-04-20</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>226</th>
      <td>Zambia</td>
      <td>ZMB</td>
      <td>AFRO</td>
      <td>REPORTING</td>
      <td>2022-03-29</td>
      <td>3383947.0</td>
      <td>2919895.0</td>
      <td>18.407</td>
      <td>15.883</td>
      <td>2139138.0</td>
      <td>11.636</td>
      <td>Beijing CNBG - BBIBP-CorV,Janssen - Ad26.COV 2...</td>
      <td>2021-04-14</td>
      <td>3.0</td>
      <td>58881.0</td>
      <td>0.320</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Zimbabwe</td>
      <td>ZWE</td>
      <td>AFRO</td>
      <td>REPORTING</td>
      <td>2022-04-03</td>
      <td>9518765.0</td>
      <td>5513887.0</td>
      <td>64.044</td>
      <td>37.098</td>
      <td>3558338.0</td>
      <td>23.941</td>
      <td>Beijing CNBG - BBIBP-CorV,Bharat - Covaxin,Gam...</td>
      <td>2021-02-18</td>
      <td>4.0</td>
      <td>446540.0</td>
      <td>3.004</td>
    </tr>
  </tbody>
</table>
<p>228 rows × 16 columns</p>
</div>




```python
vac[['WHO_REGION','COUNTRY','FIRST_VACCINE_DATE']].sort_values(by='FIRST_VACCINE_DATE')\
.groupby('WHO_REGION').head(2).sort_values(['WHO_REGION','FIRST_VACCINE_DATE'])
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
      <th>WHO_REGION</th>
      <th>COUNTRY</th>
      <th>FIRST_VACCINE_DATE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>183</th>
      <td>AFRO</td>
      <td>Seychelles</td>
      <td>2021-01-10</td>
    </tr>
    <tr>
      <th>130</th>
      <td>AFRO</td>
      <td>Mauritius</td>
      <td>2021-01-26</td>
    </tr>
    <tr>
      <th>38</th>
      <td>AMRO</td>
      <td>Canada</td>
      <td>2020-12-14</td>
    </tr>
    <tr>
      <th>218</th>
      <td>AMRO</td>
      <td>United States of America</td>
      <td>2020-12-14</td>
    </tr>
    <tr>
      <th>15</th>
      <td>EMRO</td>
      <td>Bahrain</td>
      <td>2020-11-04</td>
    </tr>
    <tr>
      <th>216</th>
      <td>EMRO</td>
      <td>United Arab Emirates</td>
      <td>2020-12-14</td>
    </tr>
    <tr>
      <th>100</th>
      <td>EURO</td>
      <td>Israel</td>
      <td>2020-12-16</td>
    </tr>
    <tr>
      <th>56</th>
      <td>EURO</td>
      <td>Denmark</td>
      <td>2020-12-16</td>
    </tr>
    <tr>
      <th>118</th>
      <td>OTHER</td>
      <td>Liechtenstein</td>
      <td>2021-01-06</td>
    </tr>
    <tr>
      <th>95</th>
      <td>SEARO</td>
      <td>Indonesia</td>
      <td>2021-01-13</td>
    </tr>
    <tr>
      <th>94</th>
      <td>SEARO</td>
      <td>India</td>
      <td>2021-01-16</td>
    </tr>
    <tr>
      <th>43</th>
      <td>WPRO</td>
      <td>China</td>
      <td>2020-07-22</td>
    </tr>
    <tr>
      <th>112</th>
      <td>WPRO</td>
      <td>Lao People's Democratic Republic</td>
      <td>2020-11-25</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 우리나라의 첫 백신 도입일은?
vac[vac['COUNTRY']=="Republic of Korea"]['FIRST_VACCINE_DATE']
```




    167    2021-02-26
    Name: FIRST_VACCINE_DATE, dtype: object




```python
# 누적 확진자 비율이 30%가 넘는 국가
vac_start[vac_start['Cases - cumulative total per 100000 population']/1000>=30] \
.sort_values('Cases - cumulative total per 100000 population', ascending=False)
# 뭔가 값이 이상한데...;
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
      <th>Country</th>
      <th>Cases - cumulative total per 100000 population</th>
      <th>Deaths - cumulative total per 100000 population</th>
      <th>COUNTRY</th>
      <th>PERSONS_FULLY_VACCINATED_PER100</th>
      <th>PERSONS_BOOSTER_ADD_DOSE_PER100</th>
      <th>FIRST_VACCINE_DATE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7</th>
      <td>Republic of Korea</td>
      <td>752602</td>
      <td>1486</td>
      <td>Republic of Korea</td>
      <td>86.643</td>
      <td>63.976</td>
      <td>2021-02-26</td>
    </tr>
    <tr>
      <th>3</th>
      <td>France</td>
      <td>624536</td>
      <td>579</td>
      <td>France</td>
      <td>80.182</td>
      <td>70.667</td>
      <td>2020-12-30</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Germany</td>
      <td>572348</td>
      <td>85</td>
      <td>Germany</td>
      <td>76.040</td>
      <td>58.986</td>
      <td>2020-12-23</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Italy</td>
      <td>353193</td>
      <td>861</td>
      <td>Italy</td>
      <td>79.472</td>
      <td>65.048</td>
      <td>2020-12-23</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Japan</td>
      <td>316134</td>
      <td>337</td>
      <td>Japan</td>
      <td>79.726</td>
      <td>44.399</td>
      <td>2021-02-17</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Australia</td>
      <td>312941</td>
      <td>198</td>
      <td>Australia</td>
      <td>83.453</td>
      <td>50.840</td>
      <td>2021-02-21</td>
    </tr>
    <tr>
      <th>0</th>
      <td>United States of America</td>
      <td>255634</td>
      <td>3030</td>
      <td>United States of America</td>
      <td>64.478</td>
      <td>28.990</td>
      <td>2020-12-14</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Viet Nam</td>
      <td>216355</td>
      <td>117</td>
      <td>Viet Nam</td>
      <td>78.974</td>
      <td>15.411</td>
      <td>2021-03-08</td>
    </tr>
    <tr>
      <th>5</th>
      <td>The United Kingdom</td>
      <td>166033</td>
      <td>534</td>
      <td>The United Kingdom</td>
      <td>72.813</td>
      <td>NaN</td>
      <td>2020-12-21</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Thailand</td>
      <td>135430</td>
      <td>868</td>
      <td>Thailand</td>
      <td>72.330</td>
      <td>35.391</td>
      <td>2021-02-28</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Brazil</td>
      <td>99883</td>
      <td>699</td>
      <td>Brazil</td>
      <td>74.193</td>
      <td>35.706</td>
      <td>2021-01-17</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Russian Federation</td>
      <td>71407</td>
      <td>1629</td>
      <td>Russian Federation</td>
      <td>50.030</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Austria</td>
      <td>62602</td>
      <td>85</td>
      <td>Austria</td>
      <td>74.144</td>
      <td>57.842</td>
      <td>2020-12-30</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Spain</td>
      <td>61379</td>
      <td>329</td>
      <td>Spain</td>
      <td>78.645</td>
      <td>52.067</td>
      <td>2020-12-30</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Portugal</td>
      <td>60083</td>
      <td>139</td>
      <td>Portugal</td>
      <td>85.641</td>
      <td>61.648</td>
      <td>2020-12-23</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Malaysia</td>
      <td>59675</td>
      <td>108</td>
      <td>Malaysia</td>
      <td>79.739</td>
      <td>48.919</td>
      <td>2021-02-24</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Greece</td>
      <td>56361</td>
      <td>427</td>
      <td>Greece</td>
      <td>71.746</td>
      <td>53.248</td>
      <td>2020-12-23</td>
    </tr>
    <tr>
      <th>68</th>
      <td>New Zealand</td>
      <td>54726</td>
      <td>85</td>
      <td>New Zealand</td>
      <td>84.069</td>
      <td>0.707</td>
      <td>2021-02-19</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Canada</td>
      <td>43848</td>
      <td>224</td>
      <td>Canada</td>
      <td>82.365</td>
      <td>47.658</td>
      <td>2020-12-14</td>
    </tr>
    <tr>
      <th>59</th>
      <td>China</td>
      <td>35115</td>
      <td>309</td>
      <td>China</td>
      <td>84.687</td>
      <td>44.028</td>
      <td>2020-07-22</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Belgium</td>
      <td>31895</td>
      <td>122</td>
      <td>Belgium</td>
      <td>79.204</td>
      <td>61.744</td>
      <td>2020-12-30</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Turkey</td>
      <td>31194</td>
      <td>148</td>
      <td>Turkey</td>
      <td>63.736</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



### 백신 종류와 확진자 수의 상관관계

#### 1) 백신 종류 확인


```python
#백신 전체 종류 확인
vac_type=[]
for i in vac['VACCINES_USED'] :
    try :
        for j in i.split(',') :
            vac_type.append(j.replace(' ',''))
    except :
        pass

    
# 중복 제거
vac_list=sorted(list(set(vac_type)))
print(vac_list)

# 이렇게 백신 추출도 가능
print(vac_meta['VACCINE_NAME'].unique())
print(len(vac_meta['VACCINE_NAME'].unique()))
```

    ['AnhuiZL-Zifivax', 'AstraZeneca-AZD1222', 'AstraZeneca-Vaxzevria', 'BeijingCNBG-BBIBP-CorV', 'Bharat-Covaxin', 'CIGB-CIGB-66', 'CanSino-Convidecia', 'Finlay-Soberana-02', 'Finlay-SoberanaPlus', 'Gamaleya-Gam-Covid-Vac', 'Gamaleya-Sputnik-Light', 'Gamaleya-SputnikV', 'IMB-Covidful', 'Janssen-Ad26.COV2-S', 'Julphar-Hayat-Vax', 'Moderna-Spikevax', 'Moderna-mRNA-1273', 'Novavax-NUVAXOVID', 'PfizerBioNTech-Comirnaty', 'RIBSP-QazVac', 'SII-Covishield', 'SRCVB-EpiVacCorona', 'Shenzhen-LV-SMENP-DC', 'Shifa-COVIranBarakat', 'Sinovac-CoronaVac', 'Turkovac', 'WuhanCNBG-Inactivated', 'Zydus-ZyCov-D']
    ['Moderna - mRNA-1273' 'AstraZeneca - AZD1222'
     'Pfizer BioNTech - Comirnaty' 'SRCVB - EpiVacCorona'
     'Sinovac - CoronaVac' 'Turkovac' 'Anhui ZL - Zifivax'
     'Shenzhen - LV-SMENP-DC' 'Finlay - Soberana-02' 'IMB - Covidful'
     'Gamaleya - Gam-Covid-Vac' 'RIBSP - QazVac' 'Beijing CNBG - BBIBP-CorV'
     'Zydus - ZyCov-D' 'CIGB - CIGB-66' 'AstraZeneca - Vaxzevria'
     'CanSino - Convidecia' 'Wuhan CNBG - Inactivated' 'Moderna - Spikevax'
     'Bharat - Covaxin' 'Janssen - Ad26.COV 2-S' 'Novavax-NUVAXOVID'
     'SII - Covishield' 'Finlay - Soberana Plus' 'Gamaleya - Sputnik-Light'
     'Julphar - Hayat-Vax' 'Shifa - COVIran Barakat']
    27



```python
# 백신 이름이 중복이 있음 - 임시 승인을 위한 예비 이름 때문
vac_type=list(set(map(lambda x : x.split('-')[0],vac_list)))
vac_type
```




    ['AnhuiZL',
     'SII',
     'Novavax',
     'BeijingCNBG',
     'Shenzhen',
     'PfizerBioNTech',
     'Moderna',
     'AstraZeneca',
     'Shifa',
     'CanSino',
     'Bharat',
     'SRCVB',
     'Gamaleya',
     'Sinovac',
     'RIBSP',
     'IMB',
     'Zydus',
     'Finlay',
     'Turkovac',
     'CIGB',
     'Julphar',
     'WuhanCNBG',
     'Janssen']




```python
vac_re=[]
for i in vac_type :
    data=i.replace('PfizerBioNTech','Pfizer BioNTech')
    data=data.replace('BeijingCNBG','Beijing CNBG')
    data=data.replace('WuhanCNBG','Wuhan CNBG')
    vac_re.append(data)
vac_type=vac_re
```

#### 2) 백신 사용 여부 라벨링


```python
vac_c=vac

#백신 종류별 사용 여부 라벨링
for i in vac_type :
    vac_c[i]=vac_c['VACCINES_USED'].str.contains(i).astype(float)
```


```python
vac_c.keys()
```




    Index(['COUNTRY', 'ISO3', 'WHO_REGION', 'DATA_SOURCE', 'DATE_UPDATED',
           'TOTAL_VACCINATIONS', 'PERSONS_VACCINATED_1PLUS_DOSE',
           'TOTAL_VACCINATIONS_PER100', 'PERSONS_VACCINATED_1PLUS_DOSE_PER100',
           'PERSONS_FULLY_VACCINATED', 'PERSONS_FULLY_VACCINATED_PER100',
           'VACCINES_USED', 'FIRST_VACCINE_DATE', 'NUMBER_VACCINES_TYPES_USED',
           'PERSONS_BOOSTER_ADD_DOSE', 'PERSONS_BOOSTER_ADD_DOSE_PER100',
           'AnhuiZL', 'SII', 'Novavax', 'Beijing CNBG', 'Shenzhen',
           'Pfizer BioNTech', 'Moderna', 'AstraZeneca', 'Shifa', 'CanSino',
           'Bharat', 'SRCVB', 'Gamaleya', 'Sinovac', 'RIBSP', 'IMB', 'Zydus',
           'Finlay', 'Turkovac', 'CIGB', 'Julphar', 'WuhanCNBG', 'Janssen',
           'Wuhan CNBG'],
          dtype='object')




```python
vac_c =vac_c[['COUNTRY', 'Janssen', 'Beijing CNBG', 'Turkovac', 'Shifa', 'RIBSP', 'AnhuiZL',
       'CanSino', 'Pfizer BioNTech', 'Gamaleya', 'Wuhan CNBG', 'AstraZeneca',
       'SRCVB', 'Finlay', 'Moderna', 'SII', 'Shenzhen', 'Sinovac', 'IMB',
       'Novavax', 'Bharat', 'Zydus', 'CIGB', 'Julphar']]
```


```python
vac_cd=pd.merge(vac_c, covid_sum, left_on='COUNTRY', right_on='Country')\
[['Janssen', 'Beijing CNBG', 'Turkovac', 'Shifa', 'RIBSP',
       'AnhuiZL', 'CanSino', 'Pfizer BioNTech', 'Gamaleya', 'Wuhan CNBG',
       'AstraZeneca', 'SRCVB', 'Finlay', 'Moderna', 'SII', 'Shenzhen',
       'Sinovac', 'IMB', 'Novavax', 'Bharat', 'Zydus', 'CIGB', 'Julphar',
'Cases - cumulative total per 100000 population','Deaths - cumulative total per 100000 population','deaths/cases']]
```


```python
vac_cd
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
      <th>Janssen</th>
      <th>Beijing CNBG</th>
      <th>Turkovac</th>
      <th>Shifa</th>
      <th>RIBSP</th>
      <th>AnhuiZL</th>
      <th>CanSino</th>
      <th>Pfizer BioNTech</th>
      <th>Gamaleya</th>
      <th>Wuhan CNBG</th>
      <th>...</th>
      <th>Sinovac</th>
      <th>IMB</th>
      <th>Novavax</th>
      <th>Bharat</th>
      <th>Zydus</th>
      <th>CIGB</th>
      <th>Julphar</th>
      <th>Cases - cumulative total per 100000 population</th>
      <th>Deaths - cumulative total per 100000 population</th>
      <th>deaths/cases</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>221</td>
      <td>4</td>
      <td>4.302231</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>263</td>
      <td>2</td>
      <td>1.273429</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>16</td>
      <td>0</td>
      <td>2.586713</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>219</td>
      <td>4</td>
      <td>0.370833</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>381</td>
      <td>0</td>
      <td>0.375839</td>
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
      <th>220</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>216355</td>
      <td>117</td>
      <td>0.409702</td>
    </tr>
    <tr>
      <th>221</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>1.541841</td>
    </tr>
    <tr>
      <th>222</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4</td>
      <td>1</td>
      <td>18.176312</td>
    </tr>
    <tr>
      <th>223</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>836</td>
      <td>5</td>
      <td>1.246848</td>
    </tr>
    <tr>
      <th>224</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>242</td>
      <td>6</td>
      <td>2.209945</td>
    </tr>
  </tbody>
</table>
<p>225 rows × 26 columns</p>
</div>



#### 3) 백신과 확진자, 사망자 수의 상관관계 with corr, regression


```python
# 백신과 확진자, 사망자 수의 상관관계

print(vac_cd.corr())
#Novavax가 상대적으로 확진자, 사망자 수와 연관성이 그나마 있음 - 근데 유의미하진 않음.
#Sinovac의 경우 확진자 수 대비 사망자 수에 상대적으로 높은편
```

                                                      Janssen  Beijing CNBG  \
    Janssen                                          1.000000     -0.004930   
    Beijing CNBG                                    -0.004930      1.000000   
    Turkovac                                        -0.069111     -0.057645   
    Shifa                                           -0.069111      0.078496   
    RIBSP                                           -0.120248      0.136578   
    AnhuiZL                                               NaN           NaN   
    CanSino                                         -0.082238      0.054049   
    Pfizer BioNTech                                  0.262248     -0.133669   
    Gamaleya                                        -0.097182      0.401278   
    Wuhan CNBG                                      -0.120248      0.136578   
    AstraZeneca                                      0.067858     -0.096403   
    SRCVB                                           -0.097959      0.014777   
    Finlay                                          -0.071416      0.020993   
    Moderna                                          0.316970     -0.162127   
    SII                                              0.041602      0.305268   
    Shenzhen                                        -0.069111      0.078496   
    Sinovac                                          0.027606      0.214869   
    IMB                                             -0.069111      0.078496   
    Novavax                                          0.106646     -0.030389   
    Bharat                                          -0.046319      0.157997   
    Zydus                                            0.065473     -0.057645   
    CIGB                                            -0.071416      0.020993   
    Julphar                                         -0.042192      0.136578   
    Cases - cumulative total per 100000 population   0.136149     -0.143780   
    Deaths - cumulative total per 100000 population  0.120793     -0.124798   
    deaths/cases                                     0.113724      0.163341   
    
                                                     Turkovac     Shifa     RIBSP  \
    Janssen                                         -0.069111 -0.069111 -0.120248   
    Beijing CNBG                                    -0.057645  0.078496  0.136578   
    Turkovac                                         1.000000 -0.004525 -0.007873   
    Shifa                                           -0.004525  1.000000 -0.007873   
    RIBSP                                           -0.007873 -0.007873  1.000000   
    AnhuiZL                                               NaN       NaN       NaN   
    CanSino                                         -0.012138 -0.012138 -0.021119   
    Pfizer BioNTech                                  0.040003 -0.113113 -0.019201   
    Gamaleya                                        -0.041874  0.108061  0.188019   
    Wuhan CNBG                                      -0.007873 -0.007873 -0.013699   
    AstraZeneca                                     -0.083794  0.054000  0.014040   
    SRCVB                                           -0.006414 -0.006414  0.401740   
    Finlay                                          -0.009112  0.496595 -0.015854   
    Moderna                                         -0.064302 -0.064302 -0.033776   
    SII                                             -0.056065 -0.056065 -0.097549   
    Shenzhen                                        -0.004525 -0.004525 -0.007873   
    Sinovac                                          0.121626 -0.037203  0.027386   
    IMB                                             -0.004525 -0.004525 -0.007873   
    Novavax                                         -0.011211 -0.011211 -0.019507   
    Bharat                                          -0.016080  0.281399 -0.027978   
    Zydus                                           -0.004525 -0.004525 -0.007873   
    CIGB                                            -0.009112 -0.009112 -0.015854   
    Julphar                                         -0.007873 -0.007873 -0.013699   
    Cases - cumulative total per 100000 population   0.007483 -0.006251 -0.028526   
    Deaths - cumulative total per 100000 population  0.018957  0.031486 -0.030277   
    deaths/cases                                    -0.031249  0.022292  0.000703   
    
                                                     AnhuiZL   CanSino  \
    Janssen                                              NaN -0.082238   
    Beijing CNBG                                         NaN  0.054049   
    Turkovac                                             NaN -0.012138   
    Shifa                                                NaN -0.012138   
    RIBSP                                                NaN -0.021119   
    AnhuiZL                                              NaN       NaN   
    CanSino                                              NaN  1.000000   
    Pfizer BioNTech                                      NaN  0.107305   
    Gamaleya                                             NaN  0.117498   
    Wuhan CNBG                                           NaN  0.202137   
    AstraZeneca                                          NaN  0.039245   
    SRCVB                                                NaN -0.017204   
    Finlay                                               NaN -0.024442   
    Moderna                                              NaN  0.033939   
    SII                                                  NaN -0.045565   
    Shenzhen                                             NaN  0.372798   
    Sinovac                                              NaN  0.265388   
    IMB                                                  NaN  0.372798   
    Novavax                                              NaN -0.030073   
    Bharat                                               NaN  0.070862   
    Zydus                                                NaN -0.012138   
    CIGB                                                 NaN -0.024442   
    Julphar                                              NaN -0.021119   
    Cases - cumulative total per 100000 population       NaN -0.005718   
    Deaths - cumulative total per 100000 population      NaN  0.013195   
    deaths/cases                                         NaN  0.113648   
    
                                                     Pfizer BioNTech  Gamaleya  \
    Janssen                                                 0.262248 -0.097182   
    Beijing CNBG                                           -0.133669  0.401278   
    Turkovac                                                0.040003 -0.041874   
    Shifa                                                  -0.113113  0.108061   
    RIBSP                                                  -0.019201  0.188019   
    AnhuiZL                                                      NaN       NaN   
    CanSino                                                 0.107305  0.117498   
    Pfizer BioNTech                                         1.000000  0.095945   
    Gamaleya                                                0.095945  1.000000   
    Wuhan CNBG                                              0.069603  0.101060   
    AstraZeneca                                             0.425746  0.088382   
    SRCVB                                                  -0.160329  0.153168   
    Finlay                                                 -0.150694  0.142122   
    Moderna                                                 0.445316  0.028069   
    SII                                                    -0.275717  0.114027   
    Shenzhen                                                0.040003 -0.041874   
    Sinovac                                                 0.135226  0.343213   
    IMB                                                     0.040003 -0.041874   
    Novavax                                                 0.099115 -0.041834   
    Bharat                                                 -0.175247  0.162005   
    Zydus                                                  -0.113113  0.108061   
    CIGB                                                   -0.073611  0.142122   
    Julphar                                                -0.019201  0.188019   
    Cases - cumulative total per 100000 population          0.132887 -0.101060   
    Deaths - cumulative total per 100000 population         0.071833 -0.040432   
    deaths/cases                                           -0.193936  0.079649   
    
                                                     Wuhan CNBG  ...   Sinovac  \
    Janssen                                           -0.120248  ...  0.027606   
    Beijing CNBG                                       0.136578  ...  0.214869   
    Turkovac                                          -0.007873  ...  0.121626   
    Shifa                                             -0.007873  ... -0.037203   
    RIBSP                                             -0.013699  ...  0.027386   
    AnhuiZL                                                 NaN  ...       NaN   
    CanSino                                            0.202137  ...  0.265388   
    Pfizer BioNTech                                    0.069603  ...  0.135226   
    Gamaleya                                           0.101060  ...  0.343213   
    Wuhan CNBG                                         1.000000  ...  0.211622   
    AstraZeneca                                        0.014040  ...  0.051818   
    SRCVB                                             -0.011159  ... -0.052733   
    Finlay                                            -0.015854  ...  0.005042   
    Moderna                                           -0.033776  ... -0.017649   
    SII                                               -0.097549  ...  0.101310   
    Shenzhen                                           0.574732  ...  0.121626   
    Sinovac                                            0.211622  ...  1.000000   
    IMB                                                0.574732  ...  0.121626   
    Novavax                                           -0.019507  ...  0.038998   
    Bharat                                            -0.027978  ...  0.102970   
    Zydus                                             -0.007873  ... -0.037203   
    CIGB                                              -0.015854  ...  0.005042   
    Julphar                                           -0.013699  ...  0.119504   
    Cases - cumulative total per 100000 population    -0.012592  ... -0.076751   
    Deaths - cumulative total per 100000 population    0.014889  ... -0.007859   
    deaths/cases                                       0.054827  ...  0.314479   
    
                                                          IMB   Novavax    Bharat  \
    Janssen                                         -0.069111  0.106646 -0.046319   
    Beijing CNBG                                     0.078496 -0.030389  0.157997   
    Turkovac                                        -0.004525 -0.011211 -0.016080   
    Shifa                                           -0.004525 -0.011211  0.281399   
    RIBSP                                           -0.007873 -0.019507 -0.027978   
    AnhuiZL                                               NaN       NaN       NaN   
    CanSino                                          0.372798 -0.030073  0.070862   
    Pfizer BioNTech                                  0.040003  0.099115 -0.175247   
    Gamaleya                                        -0.041874 -0.041834  0.162005   
    Wuhan CNBG                                       0.574732 -0.019507 -0.027978   
    AstraZeneca                                     -0.083794  0.133795 -0.134550   
    SRCVB                                           -0.006414 -0.015891 -0.022792   
    Finlay                                          -0.009112 -0.022576  0.117379   
    Moderna                                         -0.064302  0.118739  0.010779   
    SII                                             -0.056065 -0.138910  0.124796   
    Shenzhen                                         1.000000 -0.011211 -0.016080   
    Sinovac                                          0.121626  0.038998  0.102970   
    IMB                                              1.000000 -0.011211 -0.016080   
    Novavax                                         -0.011211  1.000000  0.083002   
    Bharat                                          -0.016080  0.083002  1.000000   
    Zydus                                           -0.004525 -0.011211  0.281399   
    CIGB                                            -0.009112 -0.022576 -0.032380   
    Julphar                                         -0.007873  0.221078  0.317086   
    Cases - cumulative total per 100000 population   0.010489  0.416187 -0.052251   
    Deaths - cumulative total per 100000 population  0.058509  0.214356 -0.008512   
    deaths/cases                                     0.003185 -0.060647  0.022393   
    
                                                        Zydus      CIGB   Julphar  \
    Janssen                                          0.065473 -0.071416 -0.042192   
    Beijing CNBG                                    -0.057645  0.020993  0.136578   
    Turkovac                                        -0.004525 -0.009112 -0.007873   
    Shifa                                           -0.004525 -0.009112 -0.007873   
    RIBSP                                           -0.007873 -0.015854 -0.013699   
    AnhuiZL                                               NaN       NaN       NaN   
    CanSino                                         -0.012138 -0.024442 -0.021119   
    Pfizer BioNTech                                 -0.113113 -0.073611 -0.019201   
    Gamaleya                                         0.108061  0.142122  0.188019   
    Wuhan CNBG                                      -0.007873 -0.015854 -0.013699   
    AstraZeneca                                     -0.083794 -0.029998  0.014040   
    SRCVB                                           -0.006414 -0.012915 -0.011159   
    Finlay                                          -0.009112  0.745413 -0.015854   
    Moderna                                          0.070369 -0.061689  0.044331   
    SII                                              0.080708 -0.044043 -0.018223   
    Shenzhen                                        -0.004525 -0.009112 -0.007873   
    Sinovac                                         -0.037203  0.005042  0.119504   
    IMB                                             -0.004525 -0.009112 -0.007873   
    Novavax                                         -0.011211 -0.022576  0.221078   
    Bharat                                           0.281399 -0.032380  0.317086   
    Zydus                                            1.000000 -0.009112 -0.007873   
    CIGB                                            -0.009112  1.000000 -0.015854   
    Julphar                                         -0.007873 -0.015854  1.000000   
    Cases - cumulative total per 100000 population  -0.009092  0.051643 -0.027783   
    Deaths - cumulative total per 100000 population  0.048928 -0.019334  0.006198   
    deaths/cases                                    -0.008313 -0.036557  0.016152   
    
                                                     Cases - cumulative total per 100000 population  \
    Janssen                                                                                0.136149   
    Beijing CNBG                                                                          -0.143780   
    Turkovac                                                                               0.007483   
    Shifa                                                                                 -0.006251   
    RIBSP                                                                                 -0.028526   
    AnhuiZL                                                                                     NaN   
    CanSino                                                                               -0.005718   
    Pfizer BioNTech                                                                        0.132887   
    Gamaleya                                                                              -0.101060   
    Wuhan CNBG                                                                            -0.012592   
    AstraZeneca                                                                            0.147896   
    SRCVB                                                                                  0.015507   
    Finlay                                                                                -0.026745   
    Moderna                                                                                0.216483   
    SII                                                                                   -0.167745   
    Shenzhen                                                                               0.010489   
    Sinovac                                                                               -0.076751   
    IMB                                                                                    0.010489   
    Novavax                                                                                0.416187   
    Bharat                                                                                -0.052251   
    Zydus                                                                                 -0.009092   
    CIGB                                                                                   0.051643   
    Julphar                                                                               -0.027783   
    Cases - cumulative total per 100000 population                                         1.000000   
    Deaths - cumulative total per 100000 population                                        0.558913   
    deaths/cases                                                                          -0.113439   
    
                                                     Deaths - cumulative total per 100000 population  \
    Janssen                                                                                 0.120793   
    Beijing CNBG                                                                           -0.124798   
    Turkovac                                                                                0.018957   
    Shifa                                                                                   0.031486   
    RIBSP                                                                                  -0.030277   
    AnhuiZL                                                                                      NaN   
    CanSino                                                                                 0.013195   
    Pfizer BioNTech                                                                         0.071833   
    Gamaleya                                                                               -0.040432   
    Wuhan CNBG                                                                              0.014889   
    AstraZeneca                                                                             0.018046   
    SRCVB                                                                                   0.258953   
    Finlay                                                                                 -0.009193   
    Moderna                                                                                 0.142927   
    SII                                                                                    -0.136700   
    Shenzhen                                                                                0.058509   
    Sinovac                                                                                -0.007859   
    IMB                                                                                     0.058509   
    Novavax                                                                                 0.214356   
    Bharat                                                                                 -0.008512   
    Zydus                                                                                   0.048928   
    CIGB                                                                                   -0.019334   
    Julphar                                                                                 0.006198   
    Cases - cumulative total per 100000 population                                          0.558913   
    Deaths - cumulative total per 100000 population                                         1.000000   
    deaths/cases                                                                           -0.025299   
    
                                                     deaths/cases  
    Janssen                                              0.113724  
    Beijing CNBG                                         0.163341  
    Turkovac                                            -0.031249  
    Shifa                                                0.022292  
    RIBSP                                                0.000703  
    AnhuiZL                                                   NaN  
    CanSino                                              0.113648  
    Pfizer BioNTech                                     -0.193936  
    Gamaleya                                             0.079649  
    Wuhan CNBG                                           0.054827  
    AstraZeneca                                         -0.159977  
    SRCVB                                                0.026965  
    Finlay                                              -0.004450  
    Moderna                                             -0.197371  
    SII                                                  0.238719  
    Shenzhen                                             0.003185  
    Sinovac                                              0.314479  
    IMB                                                  0.003185  
    Novavax                                             -0.060647  
    Bharat                                               0.022393  
    Zydus                                               -0.008313  
    CIGB                                                -0.036557  
    Julphar                                              0.016152  
    Cases - cumulative total per 100000 population      -0.113439  
    Deaths - cumulative total per 100000 population     -0.025299  
    deaths/cases                                         1.000000  
    
    [26 rows x 26 columns]



```python
# !pip3 install statsmodels
```


```python
import statsmodels.api as sm
```


```python
vac_cd=vac_cd.dropna()
```


```python
# 데이터 불러오기

# crim, rm, lstat을 통한 다중 선형회귀분석
x_data = vac_cd.iloc[:,:23] #변수 여러개
target = vac_cd['Cases - cumulative total per 100000 population']

# for b0, 상수항 추가
x_data1 = sm.add_constant(x_data, has_constant = "add")

# OLS 검정
multi_model = sm.OLS(target, x_data1)
fitted_multi_model = multi_model.fit()
fitted_multi_model.summary()

# novavax이 좀 유의미한 변수로 추정.
```




<table class="simpletable">
<caption>OLS Regression Results</caption>
<tr>
  <th>Dep. Variable:</th>    <td>Cases - cumulative total per 100000 population</td> <th>  R-squared:         </th> <td>   0.264</td>
</tr>
<tr>
  <th>Model:</th>                                  <td>OLS</td>                      <th>  Adj. R-squared:    </th> <td>   0.184</td>
</tr>
<tr>
  <th>Method:</th>                            <td>Least Squares</td>                 <th>  F-statistic:       </th> <td>   3.311</td>
</tr>
<tr>
  <th>Date:</th>                            <td>Sun, 01 May 2022</td>                <th>  Prob (F-statistic):</th> <td>5.84e-06</td>
</tr>
<tr>
  <th>Time:</th>                                <td>19:20:28</td>                    <th>  Log-Likelihood:    </th> <td> -2734.7</td>
</tr>
<tr>
  <th>No. Observations:</th>                     <td>   216</td>                     <th>  AIC:               </th> <td>   5513.</td>
</tr>
<tr>
  <th>Df Residuals:</th>                         <td>   194</td>                     <th>  BIC:               </th> <td>   5588.</td>
</tr>
<tr>
  <th>Df Model:</th>                             <td>    21</td>                     <th>                     </th>     <td> </td>   
</tr>
<tr>
  <th>Covariance Type:</th>                     <td>nonrobust</td>                   <th>                     </th>     <td> </td>   
</tr>
</table>
<table class="simpletable">
<tr>
         <td></td>            <th>coef</th>     <th>std err</th>      <th>t</th>      <th>P>|t|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>const</th>           <td> 8924.0845</td> <td> 1.59e+04</td> <td>    0.560</td> <td> 0.576</td> <td>-2.25e+04</td> <td> 4.03e+04</td>
</tr>
<tr>
  <th>Janssen</th>         <td> 1.309e+04</td> <td> 1.24e+04</td> <td>    1.056</td> <td> 0.292</td> <td>-1.14e+04</td> <td> 3.75e+04</td>
</tr>
<tr>
  <th>Beijing CNBG</th>    <td>-1.377e+04</td> <td> 1.34e+04</td> <td>   -1.027</td> <td> 0.306</td> <td>-4.02e+04</td> <td> 1.27e+04</td>
</tr>
<tr>
  <th>Turkovac</th>        <td> 3.863e+04</td> <td> 8.33e+04</td> <td>    0.464</td> <td> 0.643</td> <td>-1.26e+05</td> <td> 2.03e+05</td>
</tr>
<tr>
  <th>Shifa</th>           <td> 2.286e+05</td> <td>  1.3e+05</td> <td>    1.755</td> <td> 0.081</td> <td>-2.83e+04</td> <td> 4.86e+05</td>
</tr>
<tr>
  <th>RIBSP</th>           <td> 7711.1682</td> <td> 5.94e+04</td> <td>    0.130</td> <td> 0.897</td> <td>-1.09e+05</td> <td> 1.25e+05</td>
</tr>
<tr>
  <th>AnhuiZL</th>         <td>-1.686e-10</td> <td> 8.86e-11</td> <td>   -1.902</td> <td> 0.059</td> <td>-3.43e-10</td> <td> 6.25e-12</td>
</tr>
<tr>
  <th>CanSino</th>         <td> 1.059e+04</td> <td> 3.52e+04</td> <td>    0.301</td> <td> 0.764</td> <td>-5.89e+04</td> <td>    8e+04</td>
</tr>
<tr>
  <th>Pfizer BioNTech</th> <td>-4598.0409</td> <td> 1.69e+04</td> <td>   -0.272</td> <td> 0.786</td> <td> -3.8e+04</td> <td> 2.88e+04</td>
</tr>
<tr>
  <th>Gamaleya</th>        <td>-7476.8542</td> <td> 1.56e+04</td> <td>   -0.478</td> <td> 0.633</td> <td>-3.83e+04</td> <td> 2.34e+04</td>
</tr>
<tr>
  <th>Wuhan CNBG</th>      <td> 9367.0208</td> <td>    6e+04</td> <td>    0.156</td> <td> 0.876</td> <td>-1.09e+05</td> <td> 1.28e+05</td>
</tr>
<tr>
  <th>AstraZeneca</th>     <td> 9162.6973</td> <td> 1.35e+04</td> <td>    0.681</td> <td> 0.497</td> <td>-1.74e+04</td> <td> 3.57e+04</td>
</tr>
<tr>
  <th>SRCVB</th>           <td> 6.996e+04</td> <td> 8.38e+04</td> <td>    0.835</td> <td> 0.405</td> <td>-9.54e+04</td> <td> 2.35e+05</td>
</tr>
<tr>
  <th>Finlay</th>          <td>-2.015e+05</td> <td>  9.5e+04</td> <td>   -2.121</td> <td> 0.035</td> <td>-3.89e+05</td> <td>-1.41e+04</td>
</tr>
<tr>
  <th>Moderna</th>         <td> 2.118e+04</td> <td> 1.33e+04</td> <td>    1.588</td> <td> 0.114</td> <td>-5123.604</td> <td> 4.75e+04</td>
</tr>
<tr>
  <th>SII</th>             <td>-5020.3658</td> <td> 1.31e+04</td> <td>   -0.384</td> <td> 0.701</td> <td>-3.08e+04</td> <td> 2.08e+04</td>
</tr>
<tr>
  <th>Shenzhen</th>        <td> 1.818e+04</td> <td> 5.36e+04</td> <td>    0.339</td> <td> 0.735</td> <td>-8.75e+04</td> <td> 1.24e+05</td>
</tr>
<tr>
  <th>Sinovac</th>         <td>-1.176e+04</td> <td> 1.49e+04</td> <td>   -0.790</td> <td> 0.431</td> <td>-4.11e+04</td> <td> 1.76e+04</td>
</tr>
<tr>
  <th>IMB</th>             <td> 1.818e+04</td> <td> 5.36e+04</td> <td>    0.339</td> <td> 0.735</td> <td>-8.75e+04</td> <td> 1.24e+05</td>
</tr>
<tr>
  <th>Novavax</th>         <td> 2.209e+05</td> <td> 3.52e+04</td> <td>    6.279</td> <td> 0.000</td> <td> 1.52e+05</td> <td>  2.9e+05</td>
</tr>
<tr>
  <th>Bharat</th>          <td>-1.065e+04</td> <td>  2.9e+04</td> <td>   -0.367</td> <td> 0.714</td> <td>-6.79e+04</td> <td> 4.66e+04</td>
</tr>
<tr>
  <th>Zydus</th>           <td>-1.046e+04</td> <td> 8.84e+04</td> <td>   -0.118</td> <td> 0.906</td> <td>-1.85e+05</td> <td> 1.64e+05</td>
</tr>
<tr>
  <th>CIGB</th>            <td> 2.029e+05</td> <td> 8.25e+04</td> <td>    2.459</td> <td> 0.015</td> <td> 4.01e+04</td> <td> 3.66e+05</td>
</tr>
<tr>
  <th>Julphar</th>         <td>-6.562e+04</td> <td> 5.25e+04</td> <td>   -1.249</td> <td> 0.213</td> <td>-1.69e+05</td> <td>  3.8e+04</td>
</tr>
</table>
<table class="simpletable">
<tr>
  <th>Omnibus:</th>       <td>252.614</td> <th>  Durbin-Watson:     </th> <td>   2.035</td> 
</tr>
<tr>
  <th>Prob(Omnibus):</th> <td> 0.000</td>  <th>  Jarque-Bera (JB):  </th> <td>10283.622</td>
</tr>
<tr>
  <th>Skew:</th>          <td> 4.876</td>  <th>  Prob(JB):          </th> <td>    0.00</td> 
</tr>
<tr>
  <th>Kurtosis:</th>      <td>35.365</td>  <th>  Cond. No.          </th> <td>8.35e+17</td> 
</tr>
</table><br/><br/>Notes:<br/>[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.<br/>[2] The smallest eigenvalue is 9.86e-34. This might indicate that there are<br/>strong multicollinearity problems or that the design matrix is singular.



### 백신 종류별 사용 국가 수


```python
vac.keys()
```




    Index(['COUNTRY', 'ISO3', 'WHO_REGION', 'DATA_SOURCE', 'DATE_UPDATED',
           'TOTAL_VACCINATIONS', 'PERSONS_VACCINATED_1PLUS_DOSE',
           'TOTAL_VACCINATIONS_PER100', 'PERSONS_VACCINATED_1PLUS_DOSE_PER100',
           'PERSONS_FULLY_VACCINATED', 'PERSONS_FULLY_VACCINATED_PER100',
           'VACCINES_USED', 'FIRST_VACCINE_DATE', 'NUMBER_VACCINES_TYPES_USED',
           'PERSONS_BOOSTER_ADD_DOSE', 'PERSONS_BOOSTER_ADD_DOSE_PER100',
           'AnhuiZL', 'SII', 'Novavax', 'Beijing CNBG', 'Shenzhen',
           'Pfizer BioNTech', 'Moderna', 'AstraZeneca', 'Shifa', 'CanSino',
           'Bharat', 'SRCVB', 'Gamaleya', 'Sinovac', 'RIBSP', 'IMB', 'Zydus',
           'Finlay', 'Turkovac', 'CIGB', 'Julphar', 'WuhanCNBG', 'Janssen',
           'Wuhan CNBG'],
          dtype='object')




```python
vac[['Janssen', 'Beijing CNBG', 'Turkovac', 'Shifa', 'RIBSP', 'AnhuiZL',
       'CanSino', 'Pfizer BioNTech', 'Gamaleya', 'Wuhan CNBG', 'AstraZeneca',
       'SRCVB', 'Finlay', 'Moderna', 'SII', 'Shenzhen', 'Sinovac', 'IMB',
       'Novavax', 'Bharat', 'Zydus', 'CIGB', 'Julphar']].sum()
# 화이자, 아스트라제네카, 얀센, 모더나, 베이징 CNBG, SII 많이 쓴 것 같음
```




    Janssen            115.0
    Beijing CNBG        95.0
    Turkovac             1.0
    Shifa                1.0
    RIBSP                3.0
    AnhuiZL              0.0
    CanSino              7.0
    Pfizer BioNTech    167.0
    Gamaleya            63.0
    Wuhan CNBG           3.0
    AstraZeneca        137.0
    SRCVB                2.0
    Finlay               4.0
    Moderna            108.0
    SII                 91.0
    Shenzhen             1.0
    Sinovac             53.0
    IMB                  1.0
    Novavax              6.0
    Bharat              12.0
    Zydus                1.0
    CIGB                 4.0
    Julphar              3.0
    dtype: float64




```python
print(vac[vac['COUNTRY']=='Republic of Korea']['VACCINES_USED'])
```

    167    AstraZeneca - Vaxzevria,Janssen - Ad26.COV 2-S...
    Name: VACCINES_USED, dtype: object



```python
# 전체 백신접종 수 150만회 이상 국가(작은 나라 제외) 중 대륙별 백신 접종률 제일 낮은 국가

vac[['COUNTRY','WHO_REGION','PERSONS_FULLY_VACCINATED_PER100']]\
[vac['TOTAL_VACCINATIONS']>1500000].sort_values('PERSONS_FULLY_VACCINATED_PER100').\
groupby('WHO_REGION').head(3).sort_values('WHO_REGION')
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
      <th>COUNTRY</th>
      <th>WHO_REGION</th>
      <th>PERSONS_FULLY_VACCINATED_PER100</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>122</th>
      <td>Malawi</td>
      <td>AFRO</td>
      <td>4.505</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Cameroon</td>
      <td>AFRO</td>
      <td>4.527</td>
    </tr>
    <tr>
      <th>125</th>
      <td>Mali</td>
      <td>AFRO</td>
      <td>5.038</td>
    </tr>
    <tr>
      <th>91</th>
      <td>Honduras</td>
      <td>AMRO</td>
      <td>48.822</td>
    </tr>
    <tr>
      <th>159</th>
      <td>Paraguay</td>
      <td>AMRO</td>
      <td>47.023</td>
    </tr>
    <tr>
      <th>85</th>
      <td>Guatemala</td>
      <td>AMRO</td>
      <td>33.894</td>
    </tr>
    <tr>
      <th>196</th>
      <td>Sudan</td>
      <td>EMRO</td>
      <td>8.150</td>
    </tr>
    <tr>
      <th>200</th>
      <td>Syrian Arab Republic</td>
      <td>EMRO</td>
      <td>8.385</td>
    </tr>
    <tr>
      <th>191</th>
      <td>Somalia</td>
      <td>EMRO</td>
      <td>8.703</td>
    </tr>
    <tr>
      <th>168</th>
      <td>Republic of Moldova</td>
      <td>EURO</td>
      <td>26.174</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Bosnia and Herzegovina</td>
      <td>EURO</td>
      <td>25.789</td>
    </tr>
    <tr>
      <th>111</th>
      <td>Kyrgyzstan</td>
      <td>EURO</td>
      <td>18.646</td>
    </tr>
    <tr>
      <th>139</th>
      <td>Myanmar</td>
      <td>SEARO</td>
      <td>40.873</td>
    </tr>
    <tr>
      <th>95</th>
      <td>Indonesia</td>
      <td>SEARO</td>
      <td>59.143</td>
    </tr>
    <tr>
      <th>94</th>
      <td>India</td>
      <td>SEARO</td>
      <td>60.960</td>
    </tr>
    <tr>
      <th>161</th>
      <td>Philippines</td>
      <td>WPRO</td>
      <td>61.231</td>
    </tr>
    <tr>
      <th>112</th>
      <td>Lao People's Democratic Republic</td>
      <td>WPRO</td>
      <td>63.429</td>
    </tr>
    <tr>
      <th>134</th>
      <td>Mongolia</td>
      <td>WPRO</td>
      <td>66.341</td>
    </tr>
  </tbody>
</table>
</div>


