## MVP Sampel Data 구조
### 기준정보 (Aurora Mysql RDS)
|pvname|constName|const|
|------|---|---|
|AI028|forTae|498.0|
|AI028|forSirs|0.0|

### TSDB 정보 (InfluxDB 2.0)
RAW DATA
  
|date|pdevice|cdevice|parameter|value|
|------|---|---|---|---|
|2022-02-24T14:52:00Z|TRN0001|ACB0001|TAE|562538.625|
|2022-02-24T15:02:00Z|TRN0001|ACB0001|TAE|562538.625|
  
DownSampling DATA : 1Hour
  
|date|pdevice|cdevice|parameter|value|
|------|---|---|---|---|
|2022-02-24T15:00:00Z|TRN0001|ACB0001|TAE|0.0|
|2022-02-24T16:00:00Z|TRN0001|ACB0001|TAE|0.0|

### 통계 Service 정보 (Aurora Mysql RDS)
service DATA

|date|pvname|pdevice|cdevice|stae|ssrs|PR|
|------|---|---|---|---|---|---|
|2022-02-24T14:52:00Z|AI028|TRN0001|ACB0001|0.0|92.0|0.0|
|2022-02-24T15:02:00Z|AI028|TRN0001|ACB0001|0.0|93.0|0.0|
