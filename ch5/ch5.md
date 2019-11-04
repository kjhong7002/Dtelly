## 5.3.1 범위 집계

+ 사용자가 지정한 범위 내 집계 수행.

아파치 웹 로그에서 데이터 크기가 1000바이트에서 2000바이트 사이의 데이터 집계
```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "bytes_range" : {
            "range": {
              "field": "bytes",
              "ranges": [
                {
                  "from": 1000,
                  "to": 2000
                }
              ]
            }
        }
    }
}
```

+ 여러 범위 지정 (range 속성에 추가)

```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "bytes_range" : {
            "range": {
              "field": "bytes",
              "ranges": [
                {
                  "to": 1000
                },
                {
                  "from": 1000,
                  "to": 2000
                },
                {
                  "from": 2000,
                  "to": 3000
                }
              ]
            }
        }
    }
}
```

+ key에 제공되는 범위를 좀 더 명확하게 표현

```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "bytes_range" : {
            "range": {
              "field": "bytes",
              "ranges": [
                {
                  "key": "small",
                  "to": 1000
                },
                {
                  "key": "medium",
                  "from": 1000,
                  "to": 2000
                },
                {
                  "key": "large",
                  "from": 2000,
                  "to": 3000
                }
              ]
            }
        }
    }
}
```

## 5.3.2 날짜 범위 집계

+ 날짜 값을 범위로 집계 수행

엘라스틱서치에서 지원하는 형식만 사용해야 함 (https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html)

특정 기간동안 서버로 전달된 요청 수

```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "request count with date range" : {
            "date_range": {
              "field": "timestamp",
              "ranges": [
                {
                  "from": "2015-01-04T05:14:00.000Z",
                  "to": "2015-01-04T05:16:00.000Z"
                }
              ]
            }
        }
    }
}

```

## 5.3.3 히스토그램 집계

+ 지정한 간격 범위 내에서 집계를 수행

서버로 유입되는 데이터의 크리를 10000 바이트 간격으로 집계

```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "bytes_histogram" : {
            "histogram" : {
                "field" : "bytes",
                "interval": 10000
            }
        }
    }
}
```

문서 개수가 0인 간격 제외하기 위해 최소 문서 수 설정(min_doc_count)

```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "bytes_histogram" : {
            "histogram" : {
                "field" : "bytes",
                "interval": 10000,
                "min_doc_count": 1
            }
        }
    }
}
```

## 5.3.4 날짜 히스토그램 집계

+ 분, 시간, 월, 연도를 구간으로 집계

분 단위로 얼마만큼의 사용자 유입이 있었는 지 확인

```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "daily_request_count" : {
            "date_histogram": {
              "field": "timestamp",
              "interval": "minute"
            }
        }
    }
}
```

조금 더 간단한 날짜 형식("yyyy-MM-dd")으로 변환한 일자별 집계

```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "daily_request_count" : {
            "date_histogram": {
              "field": "timestamp",
              "interval": "day",
              "format": "yyyy-MM-dd"
            }
        }
    }
}
```

타임존을 한국시로 설정 ("+09:00")

```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "daily_request_count" : {
            "date_histogram": {
              "field": "timestamp",
              "interval": "day",
              "time_zone": "+09:00"
            }
        }
    }
}
```
