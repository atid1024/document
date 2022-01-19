# InfluxDB v2 : Flux language, quick reference guide and cheat sheet
## 데이터 쿼리
- 데이터 소스 정의(버킷 - 데이터베이스) : from
```SQL
from(bucket: "netdatatsdb/autogen")
```
- 시간 범위, 절대 또는 상대: range
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)

from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h, stop: -10m)

from(bucket: "netdatatsdb/autogen")
  |> range(start: 2021-01-25T00:00:00Z, stop: 2021-01-29T23:59:00Z)
```
- 측정 기준 필터링: filter
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r["_measurement"] == "vpsmetrics")
```
- 태그 키로 필터링:
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r["_measurement"] == "vpsmetrics")
  |> filter(fn: (r) => r["host"] == "vpsfrsqlpac1")
```
- 필드 및 필드 값으로 필터링:
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r["_measurement"] == "vpsmetrics")
  |> filter(fn: (r) => r["host"] == "vpsfrsqlpac1")
  |> filter(fn: (r) => r["_field"] == "pcpu")
  |> filter(fn: (r) => r["_value"] > 80)
```
- 필터는 / 연산자 filter를 사용하여 하나의 절 에서 결합할 수 있습니다.and/or
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r["_measurement"] == "vpsmetrics" and
                       r["host"] == "vpsfrsqlpac1" and
                       r["_field"] == "pcpu" and r["_value"] > 80
  )
```
- 기본 설정에 따라 점 표기법이 허용됩니다.
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "vpsmetrics" and
                       r.host == "vpsfrsqlpac1" and
                       r._field == "pcpu" and r._value > 80
  )
```
- 데이터 표시: yield
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "vpsmetrics" and
                       r.host == "vpsfrsqlpac1" and
                       r._field == "pcpu" and r._value > 80 )
  |> yield()
```
- 정규 표현식 사용:
```SQL
|> filter(fn: (r) => r.host =~ /^vpsfrsqlpac[1-8]$/ )

|> filter(fn: (r) => r.host !~ /^vpsensqlpac[1-8]$/ )
```
- 최고 n기록:limit
```SQL
|> limit(n:10)
```
- 마지막 n기록:tail
```SQL
|> tail(n:10)
```
- 데이터 정렬: sort
```SQL
|> sort(columns: ["_value"], desc: true)
```
- 열 이름 바꾸기: rename
```SQL
|> rename(columns: {_value: "average", _time: "when"})
```
- 출력 열 제거: drop
```SQL
|> drop(fn: (column) => column =~ /^_(start|stop|measurement)/)
```
- 출력 열 선택: keep
```SQL
|> keep(columns: ["_value", "_time"])
```
- 더 진행하기 전에 간단한 첫 번째 Flux 쿼리:
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "vpsmetrics" and
                       r.host == "vpsfrsqlpac1" and
                       r._field == "pcpu" and
                       r._value > 80)
  |> sort(columns: ["_value"], desc: true)
  |> rename(columns: {_value: "average"})
  |> drop(fn: (column) => column =~ /^_(start|stop|measurement)/)
  |> limit(n:10)
  |> yield()
```

## 윈도우 데이터
- aggregateWindow
```SQL
|> aggregateWindow(
      every: 1m,
      fn: mean,
      column: "_value",
      timeSrc: "_stop",
      timeDst: "_time",
      createEmpty: true
)
```
```
|> aggregateWindow(every: 10m, fn: mean)
```
-- _value기본 열과 다른 열에서 집계하려면 다음을 수행합니다.
```SQL
|> aggregateWindow(every: 10m, fn: mean, column: "colname")
```
-- 값 을 제거하려면 NULL다음으로 설정 createEmpty하십시오 False.
```
|> aggregateWindow(every: 10m, fn: mean, createEmpty: false)
```
- 윈도우
-- aggregateWindow실제로 기능을 사용하는 바로 가기 기능 window입니다.
```SQL
|> window(every: 10m)
|> mean()
```
-- createEmptytruenull 값을 표시하도록 설정됨
```
|> window(every: 10m, createEmpty: true)
|> mean()
```
-- 그러면 열 _time이 출력 테이블에 집계되지 않고 _time추가 처리를 위해 열 을 다시 추가하기 위해 duplicate함수가 호출되어 복제 _start하거나 _stop열을 새 _time열로 지정합니다.
```
|> window(every: 10m, createEmpty: true)
|> mean()
|> duplicate(column: "_stop", as: "_time")
```
-- 일반 형식을 복구하기 위해 데이터는 마침내 "창 해제"됩니다.
```
|> window(every: 10m, createEmpty: true)
|> mean()
|> duplicate(column: "_stop", as: "_time")
|> window(every: inf)
```
- fill
선택적으로 이 함수를 사용하여 데이터를 윈도우화할 때 가 정의된 fill경우 빈 값을 처리 합니다(createEmptytrue)
```SQL
|> fill(column: "_value", value: 0.0)
```
```SQL
|> fill(column: "_value", usePrevious: true)
```
- 달력 월 및 연도별 창
Flux는 달력 월 및 연도별로 윈도우 데이터를 지원합니다: 1mo, 1yr. 이 기능은 InfluxQL에서는 불가능했습니다.
```SQL
|> aggregateWindow(every: 1mo, fn: mean)
```

## 시간 선택
- hourSelection
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "vpsmetrics" and
                       r.host == "vpsfrsqlpac1" and
                       r._field == "pcpu"
  )
  |> hourSelection(start: 8, stop: 18)
```

## JOIN
- join
```SQL
datapcpu = from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "vps_pcpu" and
                       r.host == "vpsfrsqlpac1" and
                       r._field == "pcpu" )
  |> aggregateWindow(every: 10s, fn: mean, createEmpty: false)

datapmem = from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "vps_pmem" and
                       r.host == "vpsfrsqlpac1" and
                       r._field == "pmem" )
  |> aggregateWindow(every: 10s, fn: mean, createEmpty: false)

join(
    tables: {pcpu:datapcpu, pmem:datapmem},
    on: ["_time", "host"],
    method: "inner"
  )
```
방법 만 inner현재 허용됩니다(v 2.0.4). 향후 릴리스에서는 다른 조인 방법이 점진적으로 구현됩니다. cross, left, right, full.
-- _time초, 마이크로초, 나노초...를 저장하는 열에 조인을 수행 하는 것은 모든 값을 지정된 단위로 truncateTimeColumn자르는 기능을 사용하여 불규칙한 타임스탬프를 정규화하여 해결할 수 있습니다. (_time)
  ```SQL
  |> truncateTimeColumn(unit: 1m)
  ```
## PIVOT
- pivot
```SQL
from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "vpsmetrics" and
                       r.host == "vpsfrsqlpac1" )
  |> pivot(
       rowKey:["_time"],
       columnKey: ["_field"],
       valueColumn: "_value"
    )
```
조인과 마찬가지로 truncateTimeColumn함수를 사용하여 타임스탬프를 정규화합니다.
-- / schema.fieldsAsCols()에서만 데이터를 피벗할 때 바로 가기 기능 사용 :time_field
```SQL
import "influxdata/influxdb/schema"

from(bucket: "netdatatsdb/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "vpsmetrics" and
                       r.host == "vpsfrsqlpac1" )
  |> schema.fieldsAsCols()
```
- asdf
```SQL

```
