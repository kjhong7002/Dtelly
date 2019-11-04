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
