# 5.3 버킷 집계

## 5.3.1 범위 집계

+ 사용자가 지정한 범위 내 집계 수행.
    + 아파치 웹 로그에서 데이터 크기가 1000바이트에서 2000바이트 사이의 데이터 집계

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
    + 엘라스틱서치에서 지원하는 형식만 사용해야 함 (https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html)
    + 특정 기간동안 서버로 전달된 요청 수

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
    + 서버로 유입되는 데이터의 크리를 10000 바이트 간격으로 집계
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

+ 문서 개수가 0인 간격 제외하기 위해 최소 문서 수 설정(min_doc_count)
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
    + 분 단위로 얼마만큼의 사용자 유입이 있었는 지 확인
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

+ 조금 더 간단한 날짜 형식("yyyy-MM-dd")으로 변환한 일자별 집계

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

+ 타임존을 한국시로 설정 ("+09:00")
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

+ 기존 날짜 값에 3시간 더해서 집계
+ 일자별 집계일 경우 3시부터 집계가 시작되도록 변경
```
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "daily_request_count" : {
            "date_histogram": {
              "field": "timestamp",
              "interval": "day",
              "offset": "+3h"
            }
        }
    }
}
```

## 5.3.3 텀즈 집계

+ 지정한 필드에 대해 빈도가 높은 Term의 순위 반환 (근사치를 계산)
    + 아파치 서버 요청량 국가별 순위 집계
    + doc_count_error_upper_bound은 오류의 상한선
    + sum_other_doc_count는 반환된 결과에 포함되지 않은 집계결과 (기본값 size 10, 10등까지 반환)
```
GET /apache-web-log/_search?size=0
{
  "aggs" : {
    "request count by country" : {
      "terms" : {
        "field" : "geoip.country_name.keyword"
      }
    }
  }
}
```

+ 각 샤드별로 상위 size개 만큼 반환 후 병합하므로 정확하지 않은 결과 반환할 수 있음
    + size 값을 100으로 변경 후 질의
```
GET /apache-web-log/_search?size=0
{
  "aggs" : {
    "request count by country" : {
      "terms" : {
        "field" : "geoip.country_name.keyword",
        "size": 100
      }
    }
  }
}
```

* 집계와 샤드 크기
텀즈 집계 시 정확히 size의 크기만큼이 아닌 경험적인 방법(샤드 크기 * 1.5 + 10)을 사용해 내부적으로 집계를 수행하는데, shard_size 속성을 사용해 각 샤드에서 집계할 크기를 직접 지정해서 불필요한 연산 줄이는 것도 가능함

# 5.4 파이프라인 집계

다른 집계로 생성된 버킷을 참조해서 집계 수행
집계 또는 중첩된 집계를 통해 생성된 버킷을 사용해 추가적인 계산 수행

## 5.4.1 형제 집계

+ 동일 선상의 위치에서 수행되는 새 집계
    + 아파치 웹 로그에서 분 단위로 합산된 데이터 중 가장 큰 데이터량

```
GET /apache-web-log/_search?size=0
{
  "aggs": {
    "histo": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "minute"
      },
      "aggs": {
        "bytes_sum": {
          "sum": {
            "field": "bytes"
          }
        }
      }
    },
    "max_bytes": {
      "max_bucket": {
        "buckets_path": "histo>bytes_sum"
      }
    }
  }
}
```
histo 버킷의 bytes_sum 버킷을 참조 (histo>bytes_sum)


+ 최대/최소/평균/통계/확장통계/백분위수/이동평균(안됨) 버킷 집계
```
GET /apache-web-log/_search?size=0
{
  "aggs": {
    "histo": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "minute"
      },
      "aggs": {
        "bytes_sum": {
          "sum": {
            "field": "bytes"
          }
        }
      }
    },
    "max_bytes": {
      "max_bucket": {
        "buckets_path": "histo>bytes_sum"
      }
    },
    "min_bytes": {
      "min_bucket": {
        "buckets_path": "histo>bytes_sum"
      }
    },
    "avg_bytes": {
      "avg_bucket": {
        "buckets_path": "histo>bytes_sum"
      }
    },
    "bytes_stats": {
      "stats_bucket": {
        "buckets_path": "histo>bytes_sum"
      }
    },
    "extend_stats": {
      "extended_stats_bucket": {
        "buckets_path": "histo>bytes_sum"
      }
    },
    "percentiles_bytes": {
      "percentiles_bucket": {
        "buckets_path": "histo>bytes_sum"
      }
    }
  }
}
```

## 5.4.2 부모 집계

+ 집계를 통해 생성된 버킷을 사용해 계산을 수행하고, 그 결과를 기존 집계에 반영

데이터가 존재하지 않는 부분을 갭 gap 이라 부름
+ 갭 발생 이유
    + 어느 하나의 버킷 안으로 포함되는 문서들에 요청된 필드가 포함되지 않은 경우
    + 하나 이상의 버킷에 대한 쿼리와 일치하는 문서가 존재하지 않는 경우
    + 다른 종속된 버킷에 값이 누락되어 계산된 메트릭이 값을 생성할 수 없는 경우
    
+ 갭 정책(gap_policy)
    + skip: 누락된 데이터를 버킷이 존재하지 않는 것으로 간주
    + insert_zero: 누락된 값을 0으로 대체하며 파이프라인 집계 계산은 정상적으로 진행
    
+ 아파치 웹 로그를 통해 수집된 데이터가 시간이 지남에 따른 변경폭 추이 확인
```
GET /apache-web-log/_search?size=0
{
  "aggs": {
    "histo": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "day"
      },
      "aggs": {
        "bytes_sum": {
          "sum": {
            "field": "bytes"
          }
        },
        "sum_deriv": {
          "derivative": {
            "buckets_path": "bytes_sum"
          }
        }
      }
    }
  }
}
```
버킷의 현재 집계 값에서 이전 집계 값을 뺌 
788636158 - 414259902 = 374376256 

+ 파생/누적/버킷스크립트/버킷셀렉터/시계열 차분 집계
```
GET /apache-web-log/_search?size=0
{
  "aggs": {
    "histo": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "day"
      },
      "aggs": {
        "bytes_sum": {
          "sum": {
            "field": "bytes"
          }
        },
        "total_count": {
          "value_count": {
            "field": "bytes"
          }
        },
        "sum_deriv": {
          "derivative": {
            "buckets_path": "bytes_sum"
          }
        },
        "cum_deriv": {
          "cumulative_sum": {
            "buckets_path": "bytes_sum"
          }
        },
        "b_script": {
          "bucket_script": {
            "buckets_path": { 
              "my_var1": "bytes_sum",
              "my_var2": "total_count"
            },
            "script":"params.my_var1 / params.my_var2"
          }
        },
        "b_selector": {
          "bucket_selector": {
            "buckets_path": { 
              "my_var1": "bytes_sum",
              "my_var2": "total_count"
            },
            "script":"params.my_var1 > params.my_var2"
          }
        },
        "s_diff": {
          "serial_diff": {
            "buckets_path": "bytes_sum",
            "lag": 1
          }
        }
      }
    }
  }
}
```

# 5.5 근사값으로 제공되는 집계 연산

## 5.5.1 집계 연산과 정확도

기능별 집계 분류
- 버킷 집계
    - 특정 기준을 충족하는 문서들을 버킷으로 분류하는 집계
- 메트릭 집계
    - 버킷에 존재하는 문서들을 이용한 각종 통계 지표를 생성하는 집계
- 파이프라인 집계
    - 서로 다른 메트릭 집계의 출력을 연결하는 집계 패밀리
- 행렬 집계
    - 추출한 값을 기반으로 결과를 행렬로 생성하는 집계 패밀리

## 5.5.2 분산 환경에서 집계 연산의 어려움

+ 루씬의 Facet API를 사용하지 않고 독자적으로 구현한 Aggregation API로 집계
+ 요청이 들어올 경우 각 샤드내에서 1차적으로 집계 수행 후 중간 결과 취학해서 최종 결과 제공

+ 10개 샤드에서 평균 집계
    1. 코디네이터 노드가 Avg 집계 요청 받는다.
    2. 코디네이터 노드는 클러스터에 존재하는 모든 샤드로 Avg 집계 요청을 보낸다.
    3. 각 샤드는 자신이 가지고 있는 데이터를 기준으로 평균값을 계산한다.
    4. 코디네이터 노드는 모든 샤드의 결과를 받을 때까지 기다린다.
    5. 모든 결과가 도착하면 도착한 10개의 값을 가지고 다시 평균값을 계산한다.
    6. 사용자에게 정확한 평균값을 결과로 제공한다.

+ 카디널리티 집계의 경우는 중복을 제가한 데이터 리스트를 모두 코디네이터 노드로 전송

+ 엘라스틱 서치는 정확도를 포기하는 방식으로 복잡한 연산을 수행하는 경우임
    + 카디널리티 집계와 같은 특수한 연산에 한해 제한적으로 근삿값 제공
