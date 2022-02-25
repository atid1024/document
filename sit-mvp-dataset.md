## MVP Sampel Data 구조
### 기준정보 (Aurora Mysql RDS)
|pvname|constName|const|
|------|---|---|
|AI028|forTae|498.0|
|AI028|forSirs|0.0|

### TSDB 정보 (InfluxDB 2.0)
- Measurement(발전소 ID)
|date|pdevice|cdevice|parameter|value|
|------|---|---|---|---|
|2022-02-24T14:52:00Z|TRN0001|ACB0001|TAE|562538.625|
|2022-02-24T15:02:00Z|TRN0001|ACB0001|TAE|562538.625|
