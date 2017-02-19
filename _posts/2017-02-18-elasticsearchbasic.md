---
layout: post
title: ElasticSearch 기초
date: 2017-02-18 10:04
categories: ElasticSearch
---

# ElasticSearch 기초

### 설치 (ubuntu)

```bash

#fileName : ElasticSearchInstall.sh
#!/bin/bash

### USAGE
###
### ./ElasticSearch.sh 1.7 will install Elasticsearch 1.7
### ./ElasticSearch.sh will fail because no version was specified (exit code 1)
###
### CLI options Contributed by @janpieper
### Check http://www.elasticsearch.org/download/ for latest version of ElasticSearch

### ElasticSearch version
if [ -z "$1" ]; then
  echo ""
  echo "  Please specify the Elasticsearch version you want to install!"
  echo ""
  echo "    $ $0 1.7"
  echo ""
  exit 1
fi
 
ELASTICSEARCH_VERSION=$1
 
if [[ ! "${ELASTICSEARCH_VERSION}" =~ ^[0-9]+\.[0-9]+ ]]; then
  echo ""
  echo "  The specified Elasticsearch version isn't valid!"
  echo ""
  echo "    $ $0 1.7"
  echo ""
  exit 2
fi
 
### Install Java 8
cd ~
sudo apt-get install python-software-properties -y
sleep 1
sudo add-apt-repository ppa:webupd8team/java -y
sleep 1
sudo apt-get update
sleep 1
sudo apt-get install oracle-java8-installer -y

### Download and install the Public Signing Key
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

### Setup Repository
echo "deb http://packages.elastic.co/elasticsearch/${ELASTICSEARCH_VERSION}/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elk.list

### Install Elasticsearch
sudo apt-get update && sudo apt-get install elasticsearch -y

### Start ElasticSearch 
sudo service elasticsearch start

### Lets wait a little while ElasticSearch starts
sleep 5

### Make sure service is running
curl http://localhost:9200

### Should return something like this:
# {
#  "status" : 200,
#  "name" : "Storm",
#  "version" : {
#    "number" : "1.3.1",
#    "build_hash" : "2de6dc5268c32fb49b205233c138d93aaf772015",
#    "build_timestamp" : "2014-07-28T14:45:15Z",
#    "build_snapshot" : false,
#    "lucene_version" : "4.9"
#  },
#  "tagline" : "You Know, for Search"
# }
```

위 스크립트를 `./ElasticSearchInstall.sh [버전]`으로 실행 실행하면 된다.


### 기본 개념


| Elastic Search | Relational DB|
|----------------|--------------|
|Index|Databases|
|Type|Table|
|Document|Row|
|Field|Column|
|Mapping|Schema|

RESTfull 방식으로 사용된다.

|Elastic Search | Relational DB | CRUD
|----|----|----|
|GET| Select|Read|
|PUT|Update|Update
|Post|Insert|Create|
|DELETE|Delete|Delete|

<br>
**1) Index 조회** 

```
curl -XGET '[elasticsearch 주소]/[Index name]

```

**결과**

```
[root@bigdataStudy]# curl -XGET localhost:9200/basketball?pretty
{
  "basketball" : {
    "aliases" : { },
    "mappings" : {
      "record" : {
        "properties" : {
          "assists" : {
            "type" : "long"
          },
          "blocks" : {
            "type" : "long"
          },
          "name" : {
            "type" : "string"
          },
          "points" : {
            "type" : "long"
          },
          "rebounds" : {
            "type" : "long"
          },
          "submit_date" : {
            "type" : "date",
            "format" : "yyyy-MM-dd"
          },
          "team" : {
            "type" : "string"
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1487474066966",
        "uuid" : "uFna9rNjR5S3vRKEZtEHvg",
        "number_of_replicas" : "1",
        "number_of_shards" : "5",
        "version" : {
          "created" : "2030399"
        }
      }
    },
    "warmers" : { }
  }
}
```

`basketball` Index정보를 볼 수 있다. `record`라는 Type의 정보도 함께 나오는 것을 확인 할 수 있다.


**2) `_update`**

`_update`옵션은 Type의 Field를 추가할 때 사용한다.

다음과 같은 Document가 있다

```
[root@bigdataStudy]# curl -XGET localhost:9200/classes/class/1/?pretty
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "title" : "Bigdata",
    "professor" : "Jo"
  }
}
```

다음과 같이 `point` Field를 추가하였다. 

```
[root@bigdataStudy]# curl -XPOST localhost:9200/classes/class/1/_update?pretty -d '
> {"doc":{"point":1}}'
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 2,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  }
}
```

결과를 확인하면 다음과 같다.

```
[root@ip-172-31-12-34 ch04]# curl -XGET localhost:9200/classes/class/1/?pretty
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 2,
  "found" : true,
  "_source" : {
    "title" : "Bigdata",
    "professor" : "Jo",
    "point" : 1
  }
}
```
추가 된 것을 확인 할 수 있다.

<br>

**3) `_bulk`**

여러개의 Document를 입력할 때 사용한다. 

다음은 파일에서 값을 가져오는 방법이다.

```
culr -XPOST [elasticsearch 주소]/_bulk --data-binary @[파일명]
```
<br>
**4) Mapping**

 Index를 생성하면 mapping 정보가 빠져 있다.
 
```
 [root@bigdataStudy]# curl -XGET localhost:9200/classes/?pretty
{
  "classes" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "creation_date" : "1487477362931",
        "uuid" : "Z3HY5liZToOyUiEgRL9J4g",
        "number_of_replicas" : "1",
        "number_of_shards" : "5",
        "version" : {
          "created" : "2030399"
        }
      }
    },
    "warmers" : { }
  }
}
```

class라는 `Type`의 mapping 값을 다음과 같이 입력하겠다.

```json
classesRating_mapping.json
{
	"class" : {
		"properties" : {
			"title" : {
				"type" : "string"
			},
			"professor" : {
				"type" : "string"
			},
			"major" : {
				"type" : "string"
			},
			"semester" : {
				"type" : "string"
			},
			"student_count" : {
				"type" : "integer"
			},
			"unit" : {
				"type" : "integer"
			},
			"rating" : {
				"type" : "integer"
			},
			"submit_date" : {
				"type" : "date",
				"format" : "yyyy-MM-dd"
			},
			"school_location" : {
				"type" : "geo_point"
			}
		}
	}
}
```

입력을 다음과 같이 한다.

```
[root@bigdataStudy]# curl -XPUT localhost:9200/classes/class/_mapping?pretty -d @classesRating_mapping.json
{
  "acknowledged" : true
}
```

`Type`의 mapping값이 입력된 것을 확인 할 수 있다.

```
[root@bigdataStudy]# curl -XGET localhost:9200/classes/?pretty
{
  "classes" : {
    "aliases" : { },
    "mappings" : {
      "class" : {
        "properties" : {
          "major" : {
            "type" : "string"
          },
          "professor" : {
            "type" : "string"
          },
          "rating" : {
            "type" : "integer"
          },
          "school_location" : {
            "type" : "geo_point"
          },
          "semester" : {
            "type" : "string"
          },
          "student_count" : {
            "type" : "integer"
          },
          "submit_date" : {
            "type" : "date",
            "format" : "yyyy-MM-dd"
          },
          "title" : {
            "type" : "string"
          },
          "unit" : {
            "type" : "integer"
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1487477362931",
        "uuid" : "Z3HY5liZToOyUiEgRL9J4g",
        "number_of_replicas" : "1",
        "number_of_shards" : "5",
        "version" : {
          "created" : "2030399"
        }
      }
    },
    "warmers" : { }
  }
}
```

**5) `_search`**

RDBMS의 `where`조건 같은 느낌도 있다. 

```
curl -XGET localhost:9200/basketball/record/_search?q=points:30&pretty
```
위는 record `type`에서 points가 30인 `Document`를 출력하게 된다.

더 많은 옵션은 [ElasticSearch API Doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html)에서 확인 할 수 있다.

<br>
**6) Aggregation**

* Metrics Aggregations - sum, min, max, avg을 구할 때 사용한다.

* Bucket Aggregations - RDBMS의 `Group By`와 같은 기능을 제공한다.

**사용법**

`_search`옵션과 함께 사용한다.

```
{
	"size" : 0,
	"aggs" : {
		"avg_score" : {
			"avg" : {
				"field" : "points"
			}
		}
	}
}
```

위와 같이 Metrics Aggregation 설정 json파일을 만든다.

평균을 구하는 aggregation이다. 

- `aggs`은 aggregation을 사용하겠다는 선언이다. 
- `avg_score`은 alias같은 느낌이다. 검색 결과의 field이름 이다.
- `avg`는 Metrics Aggregation에서 평균을 구하는 옵션이다.
- `field`는 `avg`에 사용되는  `field`값이다.

검색 결과는 다음과 같다.

```
[root@bigdataStudy]# curl -XGET localhost:9200/_search?pretty --data-binary @avg_points_aggs.json
{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "avg_score" : {
      "value" : 25.0
    }
  }
}
```

평균값이 나온 것을 확인 할 수 있다. 

Metrics Aggregation에서 `stats`옵션을 사용하면 count, min, max, avg, sum을 한번에 출력한다.


Bucket Aggregation예도 한번 보자

```

	"size" : 0,
	"aggs" : {
		"team_stats" : {
			"terms" : {
				"field" : "team"
			},
			"aggs" : {
				"stats_score" : {
					"stats" : {
						"field" : "points"
					}
				}
			}
		}
	}
}
```

이렇게 aggregation json파일을 만들었다.

- `aggs`를 사용한다고 선언했다.
- `terms`라는 bucket aggregation의 옵션을 사용한다. 이는 `team`이라는 필드로 `group by`하겠다라는 뜻이다. 
- `sub aggs`를 선언하고 point로 Metrics정보를 출력한다.

다음과 같이 4개의 Documents를 입력하였다.
```
[root@bigdataStudy]# curl -XGET localhost:9200/_search?pretty
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 4,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "basketball",
      "_type" : "record",
      "_id" : "2",
      "_score" : 1.0,
      "_source" : {
        "team" : "Chicago",
        "name" : "Michael Jordan",
        "points" : 20,
        "rebounds" : 5,
        "assists" : 8,
        "blocks" : 4,
        "submit_date" : "1996-10-13"
      }
    }, {
      "_index" : "basketball",
      "_type" : "record",
      "_id" : "4",
      "_score" : 1.0,
      "_source" : {
        "team" : "LA",
        "name" : "Kobe Bryant",
        "points" : 40,
        "rebounds" : 4,
        "assists" : 8,
        "blocks" : 6,
        "submit_date" : "2014-11-13"
      }
    }, {
      "_index" : "basketball",
      "_type" : "record",
      "_id" : "1",
      "_score" : 1.0,
      "_source" : {
        "team" : "Chicago",
        "name" : "Michael Jordan",
        "points" : 30,
        "rebounds" : 3,
        "assists" : 4,
        "blocks" : 3,
        "submit_date" : "1996-10-11"
      }
    }, {
      "_index" : "basketball",
      "_type" : "record",
      "_id" : "3",
      "_score" : 1.0,
      "_source" : {
        "team" : "LA",
        "name" : "Kobe Bryant",
        "points" : 30,
        "rebounds" : 2,
        "assists" : 8,
        "blocks" : 5,
        "submit_date" : "2014-10-13"
      }
    } ]
  }
}
```

그리고 위의 aggregation을 사용하여 검색을 해보았다.

```
[root@bigdataStudy]# curl -XGET localhost:9200/_search?pretty --data-binary @stats_by_team.json
{
  "took" : 13,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 4,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "team_stats" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [ {
        "key" : "chicago",
        "doc_count" : 2,
        "stats_score" : {
          "count" : 2,
          "min" : 20.0,
          "max" : 30.0,
          "avg" : 25.0,
          "sum" : 50.0
        }
      }, {
        "key" : "la",
        "doc_count" : 2,
        "stats_score" : {
          "count" : 2,
          "min" : 30.0,
          "max" : 40.0,
          "avg" : 35.0,
          "sum" : 70.0
        }
      } ]
    }
  }
}
```
위와 같이 `chicago`팀의 stats와 `la`팀의 stats가 출력되는 것을 확인 할 수 있다.


### Reference

* [Inflearn강의](https://www.inflearn.com/course/elk-%EC%8A%A4%ED%83%9D-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%B6%84%EC%84%9D/)
* [ricardo-rossi-GitRepo](https://gist.github.com/ricardo-rossi/8265589463915837429d)
* [ElasticSearch Doc](https://www.elastic.co/guide/index.html)


