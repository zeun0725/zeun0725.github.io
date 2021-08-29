---
title: "ES-02 엘라스틱서치 살펴보기"
date: 2021-08-28 15:39:00
categories: [elasticsearch]
---
# 2. 엘라스틱서치 살펴보기

## 엘라스틱서치를 구성하는 개념

### 기본 용어

**엘라스틱서치 데이터**

![struct](https://user-images.githubusercontent.com/26589907/131241206-3c5a70c8-fc0c-484f-9f08-dd12494d8464.png)

**인덱스**

데이터 저장 공간

- 하나의 인덱스는 하나의 타입만 가지며 하나의 물리적인 노드에 여러 개의 논리적인 인덱스를 생성할 수 있음
- 검색 시 인덱스 이름으로 문서 데이터를 검색, 여러 인덱스를 동시에 검색하는 것도 가능
- ES가 분산환경일 경우 하나의 인덱스가 여러 노드에 분산 저장되어 관리됨
- 인덱스 생성 시 기본적으로 5개의 프라이머리 샤드와 1개의 레플리카 샤드 세트를 생성함
- 각각의 샤드 수는 인덱스를 생성할 때 옵션을 이용해 변경할 수 있음
- **인덱스의 이름은 모두 소문자** 여야 하고 추가, 수정, 삭제, 검색은 RESTful API로 수행할 수 있음
- 만약 인덱스가 없는 상태에서 데이터가 추가된다면 데이터를 이용해 인덱스가 자동으로 생성됨

**샤드**

색인된 문서는 하나의 인덱스에 담긴다.

- 인덱스 내부에 색인된 데이터는 물리적인 공간에 여러 개의 파티션으로 나뉘어 구성되는데, 이 파티션을 엘라스틱서치에서는 샤드라고 부름
- 다수의 샤드로 문서를 분산 저장하고 있어 데이터 손실 위험을 최소화 할 수 있음

**타입**

인덱스의 논리적 구조를 의미

- 인덱스 속성에 따라 분류하기도 함
- ES 6.1 이상 버전부터는 인덱스당 하나의 타입만 사용할 수 있음
- 옛버전에서는 특정 카테고리를 분류하는 목적으로 타입이 많이 사용되었다, 하지만 현재는 그렇게 타입을 사용하는 것을 권장하지 않기에 장르별로 별도의 인덱스를 생성해 사용해야 함

**문서(Document)**

데이터가 저장되는 최소 단위

- 기본적으로 json 포맷으로 데이터가 저장됨
- DB와 비교하면 테이블의 행이 엘라스틱서치의 다큐먼트라고 볼 수 있음
- 하나의 문서는 다수의 필드로 구성되어 있는데 각 필드는 데이터 형태에 따라 용도에 맞는 데이터 타입을 정의해야 함
- 문서는 중첩 구조를 지원하기에 이를 이용해 문서 안에 문서를 지정하는 것도 가능

**필드**

문서를 구성하기 위한 속성

- DB의 column과 비교할 수 있으나, 칼럼은 정적인 데이터 타입인 데 반해 필드는 좀 더 동적인 데이터 타입이라고 볼 수 있음
- 하나의 필드는 목적에 따라 다수의 데이터 타입을 가질 수 있음

**매핑**

문서의 필드와 필드의 속성을 정의하고 그에 따른 색인 방법을 정의하는 프로세스

- 인덱스의 매핑 정보에는 여러 가지 데이터 타입을 지정할 수 있지만 필드명은 중복해서 사용할 수 없음

### 노드의 종류

클러스터: 물리적인 노드 인스턴스들의 모임, 모든 노드의 검색과 색인 작업을 관장하는 논리적인 개념

분산처리를 위해서는 다양한 형태의 노드들을 조합해서 클러스터를 구성해야 함

기본적으로 마스터노드가 전체적인 클러스터를 관리하고, 데이터 노드가 실제 데이터를 관리.

엘라스틱 서치는 각 설정에 따라 4가지 유형의 노드를 제공함

- 마스터 노드(Master Node)

    클러스터를 관리함

    노드 추가와 제거같은 클러스터의 전반적인 관리를 담당함

- 데이터 노드(Data Node)

    실질적인 데이터를 저장함

    검색과 통계같은 데이터 관련 작업을 수행

- 코디네이팅 노드(Coordinating Node)

    사용자의 요청만 받아서 처리

    클러스터 관련 요청은 마스터 노드에 전달하고, 데이터 관련 요청은 데이터 노드에 전달함

- 인제스트 노드(Ingest Node)

    문서의 전처리 작업을 담당함

    인덱스 생성 전 문서의 형식을 다양하게 변경할 수 있음

설정에 따라 각 노드는 한가지 유형으로 동작할 수도 있고, 여러 개의 유형을 겸해서 동작할 수도 있음

**마스터 노드**
인덱스를 생성, 삭제하는 등 클러스터와 관련된 전반적인 작업을 담당함
⇒ 네트워크 속도가 빠르고 지연이 없는 노드를 마스터 노드로 선정해야 함
다수의 노드를 마스터 노드로 설정할 수 있지만 결과적으로 하나의 노드만이 마스터 노드로 선출되어 동작한다.

```yaml
# 만약 노드를 마스터 노드 전용으로 설정하고자 한다면 엘라스틱서치 서버의 conf 폴더 안의
# elasticsearch.yml 파일을 열고 다음과 같이 설정해야 함

node.master: true
node.data: false
node.ingest: false
search.remote.connect: false
```

**데이터 노드**
문서가 실제로 저장되는 노드
데이터가 실제로 분산 저장되는 물리적 공간인 샤드가 배치되는 노드이기도 함
색인 작업은 cpu, 메모리, 스토리지 같은 컴퓨팅 리소스를 많이 소모하기 때문에 리소스 모니터링이 필요하다.
⇒ 데이터 노드는 가능한 한 마스터 노드와 분리해서 구성하는 것이 좋음

```yaml
# 데이터 노드 전용 설정
node.master: false
node.data: true
node.ingest: false
search.remote.connect: false
```

**코디네이팅 노드**
데이터 노드, 마스터 노드, 인제스트 노드의 역할을 하지 않고, 들어온 요청을 단순히 라운드로빈 방식으로 분산시켜 주는 노드

```yaml
# 코디네이팅 노드 전용 설정
node.master: false
node.data: false
node.ingest: false
search.remote.connect: false
```

**인제스트 노드**
색인에 앞서 데이터를 전처리하기 위한 노드
데이터의 포맷을 변경하기 위해 스크립트로 전처리 파이프라인을 구성하고 실행할 수 있다.

```yaml
# 인제스트 노드 전용 설정
node.matser: false
node.data: false
node.ingest: true
search.remote.connect: false
```

### 클러스터, 노드, 샤드

![cluster](https://user-images.githubusercontent.com/26589907/131241221-ceeec912-6a6f-493e-9823-09a8c04d23f1.png)

하나의 엘라스틱서치 클러스터에 노드1, 노드2로 총 2개의 물리적 노드가 존재함

엘라스틱서치 클러스터는 인덱스의 문서를 조회할 때 마스터 노드를 통해 2개의 노드를 모두 조회해서 각 데이터를 취합한 후 결과를 하나로 합쳐서 제공함

현재는 하나의 클러스터만 만들어져 있지만 여러 개의 클러스터를 연결해서 구성할 수도 있으며, 이때는 클러스터의 이름으로 각각을 구분한다. 만약 클러스터의 이름이 명시적으로 설정되지 않았다면 엘라스틱서치는 클러스터의 이름을 임의의 문자열로 지정함.

또한 클러스터에 있는 노드는 실시간으로 추가, 제거가 가능하기 때문에 가용성이나 확장성 측면에서 매우 유연함.

**클러스터의 동작방식 테스트**

3개의 클러스터를 구성했다고 가정, 프라이머리 샤드와 레플리카 샤드의 수를 조정해 가며 인덱스 생성 시 클러스터가 어떻게 동작하는 지 확인

일반적으로 프라이머리 샤드는 안정성을 위해 하나의 노드에 하나씩 분산 저장됨

**프라이머리 샤드 3개, 레플리카 샤드 0세트 구성**

data-node-01 | 1

data-node-02 | 2

data-node-03 | 3

**프라이머리 샤드 6개, 레플리카 샤드 0세트 구성**

data-node-01 | 1, 4

data-node-02 | 2, 5

data-node-03 | 3, 6

앞선 구성보다 샤드의 수가 2배 많기에 데이터가 더 잘게 쪼개져서 저장됨, 색인 시 6개의 샤드에 데이터가 분산이 됨

**프라이머리 샤드 3개, 레플리카 샤드 1세트 구성**

레플리카 샤드(프라이머리 샤드의 복제본)

data-node-01 | 1, (3)

data-node-02 | 2, (1)

data-node-03 | 3, (2)

장애 시에 레플리카 샤드를 이용해 샤드를 복구함.

프라이머리 샤드의 복제본이 존재하기에 물리적인 노드 하나가 죽더라도 나머지 2개의 노드로 전체 데이터를 복구할 수가 있음

장애가 발생하면 마스터 노드는 데이터를 재분배하거나 레플리카 샤드를 프라이머리 샤드로 승격시켜 서비스 중단 없는 복구가 가능해짐 ⇒ 따라서 장애극복 상황을 염두에 두고 노드와 샤드의 수를 적절히 구성해야 함

## 엘라스틱서치에서 제공하는 주요 API

### API의 종류

엘라스틱서치는 RESTful 방식의 API를 제공하며, 이를 통해 JSON기반으로 통신 함

기본적으로 http 통신을 위해 9200포트 사용

다음과 같은 주요 API를 제공함

- 인덱스 관리 API(Indices API): 인덱스 관리
- 문서 관리 API(Document API) : 문서의 추가, 수정, 삭제
- 검색 API(Search API): 문서 조회
- 집계 API(Aggregation API): 문서 통계

문서를 색인하기 위해서는 기본적으로 인덱스라는 그릇을 생성해야 함

인덱스를 통해 입력되는 문서의 필드를 정의하고 각 필드에 알맞은 데이터 타입을 지정할 수 있음

- Index vs Indices

    색인: 데이터가 토큰화되어 저장된 자료구조를 의미

    index: 색인 데이터

    indexing: 색인하는 과정

    indices: 매핑 정보를 저장하는 논리적인 데이터 공간

    엘라스틱서치에서는 용어에 따른 혼란을 방지하기 위해 색인을 의미할 경우 index를, 매핑 정의 공간을 의미할 경우 indices 라는 단어로 표현함.

엘라스틱서치는 사용 편의성을 위해 스키마리스 기능을 제공함.

문서를 색인하기 위해서는 기본적으로 인덱스를 생성하는 과정이 필요한데, 인덱스를 생성하는 과정 없이 문서를 추가하더라도 문서가 색인되도록 지원하는 일종의 편의 기능임

스키마리스는 검색 품질이 떨어지거나 성능상 문제가 발생할 가능성이 커지기에 쓰지 않는 것이 좋음!

### 인덱스 관리 API

인덱스를 관리하기 위한 API로 인덱스를 추가하거나 삭제할 수 있음

**인덱스 생성**

엘라스틱서치 7.x부터 인덱스에 여러 타입을 생성할 수 없기에 _doc 부분을 제거하고 스키마 매핑 값을 요청했음

```json
PUT /movie
{
	"settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
	},
  "mappings": {
    "properties": {
      "movieCd": {"type": "integer"},
      "movieNm": {"type": "text"},
      "movieNmEn": {"type": "text"},
      "prdtYear": {"type": "integer"},
      "openDt": {"type": "date"},
      "typeNm": {"type": "keyword"},
      "prdtStatNm": {"type": "keyword"},
      "nationAlt": {"type": "keyword"},
      "genreAlt": {"type": "keyword"},
      "repNationNm": {"type": "keyword"},
      "repGenreNm": {"type": "keyword"}
    }
  }
}

//실행결과
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "movie"
}
```

엘라스틱서치는 다양한 형태의 타입을 제공함.

단순히 문자열로 저장하고 싶을 경우 keyword 타입을 사용하면 되고, 형태소 분석을 원할 경우 text 타입을 사용함

**인덱스 삭제**

```json
DELETE /movie

//실행결과
{
  "acknowledged" : true
}
```

인덱스를 한번 삭제하면 복구할 수 없으므로 인덱스 삭제는 신중히 하기

### 문서 관리 API

실제 문서를 색인하고 조회, 수정, 삭제를 지원하는 API → 문서를 색인하고 내용을 수정하거나 삭제할 수 있음

문서관리 API의 세부 기능 제공 API

- Index API: 한 건의 문서를 색인함
- Get API: 한 건의 문서를 조회함
- Delete API: 한 건의 문서를 삭제함
- Update API: 한 건의 문서를 업데이트함

문서관리 API는 기본적으로 한 건의 문서를 처리하기 위한 기능을 제공하며 Single document API라고도 부름.

클러스터를 운영하다보면 다수의 문서를 처리해야 하는 경우가 발생하므로 이런 경우에 대비해 Multi-document API도 제공함

- Mulit Get API: 다수의 문서를 조회함
- Bulk API: 대량의 문서를 색인함
- Delete By Query API: 다수의 문서를 삭제함
- Update By Query API: 다수의 문서를 업데이트함
- Reindex API: 인덱스의 문서를 다시 색인함

**문서 생성**

```json
POST /movie/_doc/1
{
  "movieCd": "1",
  "movieNm": "살아남은 아이",
  "movieNmEn": "Last Child",
  "prdtYear": "2021",
  "openDt": "2021-08-27",
  "typeNm": "장편",
  "prdtStatNm": "기타",
  "nationAlt": "한국",
  "genreAlt": "드라마,가족",
  "repNationNm": "한국",
  "repGenreNm": "드라마"
}

//실행 결과
{
  "_index" : "movie",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

**문서 조회**

```json
GET /movie/_doc/1

//실행 결과
{
  "_index" : "movie",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "movieCd" : "1",
    "movieNm" : "살아남은 아이",
    "movieNmEn" : "Last Child",
    "prdtYear" : "2021",
    "openDt" : "2021-08-27",
    "typeNm" : "장편",
    "prdtStatNm" : "기타",
    "nationAlt" : "한국",
    "genreAlt" : "드라마,가족",
    "repNationNm" : "한국",
    "repGenreNm" : "드라마"
  }
}
```

**문서 삭제**

```json
DELETE /movie/_doc/1

//실행 결과
{
  "_index" : "movie",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

**id를 지정하지 않고 문서 생성**

```json
POST /movie/_doc
```

id 지정안하고 생성하면 _id값은 UUID를 통해 무작위로 생성이 됨 → 관리하기가 쉽지 않아 애로사항이 생길 수 있음

### 검색 API

검색 API 사용 방식은 크게 두가지로 나뉨

1. HTTP URI 형태의 파라미터를 URI에 추가해 검색하는 방법
2. RESTful API 방식인 QueryDSL 을 사용해 요청 본문에 질의 내용을 추가해 검색하는 방법

QueryDSL을 사용하면 가독성이 높고, json형식으로 다양한 표현이 가능해짐

**URI 방식의 검색 질의**

URI 에 파라미터를 붙여 질의하는 식

```json
//id 가 1인 키를 가진 문서 질의
GET /movie/_doc/1?pretty=true

//실행 결과
{
  "_index" : "movie",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "movieCd" : "1",
    "movieNm" : "살아남은 아이",
    "movieNmEn" : "Last Child",
    "prdtYear" : "2021",
    "openDt" : "2021-08-27",
    "typeNm" : "장편",
    "prdtStatNm" : "기타",
    "nationAlt" : "한국",
    "genreAlt" : "드라마,가족",
    "repNationNm" : "한국",
    "repGenreNm" : "드라마"
  }
}

//q 파라미터를 사용해 해당 용어와 일치하는 문서만 조회
POST /movie/_search?q=장편

//실행 결과
{
  "took" : 17,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "movieCd" : "1",
          "movieNm" : "살아남은 아이",
          "movieNmEn" : "Last Child",
          "prdtYear" : "2021",
          "openDt" : "2021-08-27",
          "typeNm" : "장편",
          "prdtStatNm" : "기타",
          "nationAlt" : "한국",
          "genreAlt" : "드라마,가족",
          "repNationNm" : "한국",
          "repGenreNm" : "드라마"
        }
      }
    ]
  }
}

// q파라미터에 별도의 필드명을 지정하지 않으면 존재하는 모든 필드를 대상으로 검색을 수행함

// q파라미터로 특정 필드 조회
POST /movie/_search?q=typeNm:장편

//실행 결과
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "movieCd" : "1",
          "movieNm" : "살아남은 아이",
          "movieNmEn" : "Last Child",
          "prdtYear" : "2021",
          "openDt" : "2021-08-27",
          "typeNm" : "장편",
          "prdtStatNm" : "기타",
          "nationAlt" : "한국",
          "genreAlt" : "드라마,가족",
          "repNationNm" : "한국",
          "repGenreNm" : "드라마"
        }
      }
    ]
  }
}
```

**Request Body 방식의 검색 질의**

URI 검색 질의로는 여러 필드를 각기 다른 검색어로 질의하기는 어려움 → 이럴 때 json 방식으로 질의 하는 것이 좋음

기본 구문

```json
POST /{index명}/_search
{
JSON 쿼리 구문
}
```

```json
//movie 인덱스의 typeNm 필드 검색
POST /movie/_search
{
	"query": {
		"term": {"typeNm": "장편"}
	}
}

//실행 결과
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "movieCd" : "1",
          "movieNm" : "살아남은 아이",
          "movieNmEn" : "Last Child",
          "prdtYear" : "2021",
          "openDt" : "2021-08-27",
          "typeNm" : "장편",
          "prdtStatNm" : "기타",
          "nationAlt" : "한국",
          "genreAlt" : "드라마,가족",
          "repNationNm" : "한국",
          "repGenreNm" : "드라마"
        }
      }
    ]
  }
}
```

쿼리구문은 다음과 같이 여러 개의 키를 조합해 객체의 키 값으로 설정할 수 있음

```json
{
	size: 몇개의 결과를 반환할 지 결정(기본 값은 10)
	from: 어느 위치부터 반환할 지 결정 (기본 값은 0) -> 0부터 시작하면 상위 0-10건의 데이터 반환
	_source: 특정 필드만 결과롤 반환하고 싶을 때 사용
	sort: 특정 필드를 기준으로 정렬 (asc, desc로 오름차순, 내림차순 정렬 지정)
	query: { 검색될 조건을 정의 }
	filter: { 검색 결과 중 특정 값을 다시 보여줌. 결과 내에서 재 검색할 때 사용하는 기능,
필터를 사용하면 자동으로 score값이 정렬되지는 않음 }
}
```

### 집계 API

메모리 기반으로 동작하기에 대용량의 데이터 통계 작업이 가능해짐

엘라스틱서치의 집계 API는 각종 통계 데이터를 실시간으로 제공할 수 있는 강력한 기능임

**데이터 집계**

```json
//movie 인덱스의 문서를 장르별로 집계
POST /movie/_search?size=0
{
	"aggs": {
		"genre": {
			"terms": {
				"field": "genreAlt"
			}
		}
	}
}

//실행 결과
{
  "took" : 350,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "genre" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "코미디",
          "doc_count" : 3
        },
        {
          "key" : "드라마",
          "doc_count" : 1
        },
        {
          "key" : "드라마,가족",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

집계 결과를 보면 버킷이라는 구조 안에 그룹화된 데이터가 포함돼 있음.

엘라스틱서치는 버킷 안에 다른 버킷의 결과를 추가할 수 있음 ⇒ 다양한 집계 유형을 결합하거나 중첩, 조합하는 것이 가능함

```json
//장르별 국가 형태를 중첩해서 보여주는 집계
POST /movie/_search?size=0
{
	"aggs": {
		"genre": {
			"terms": {
				"field": "genreAlt"
			},
			"aggs": {
    		"nation": {
    			"terms": {
    				"field": "nationAlt"
    			}
    		}
    	}
		}
	}
}

//실행 결과
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 6,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "genre" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "코미디",
          "doc_count" : 3,
          "nation" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "한국",
                "doc_count" : 3
              }
            ]
          }
        },
        {
          "key" : "드라마,가족",
          "doc_count" : 2,
          "nation" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "미국",
                "doc_count" : 1
              },
              {
                "key" : "한국",
                "doc_count" : 1
              }
            ]
          }
        },
        {
          "key" : "드라마",
          "doc_count" : 1,
          "nation" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "한국",
                "doc_count" : 1
              }
            ]
          }
        }
      ]
    }
  }
}
```

**데이터 집계 타입**

현재 4가지 API로 제공됨, 이들을 서로 조합해 사용할 수 있으며 이를 조합해 매우 강력한 기능을 제공할 수 있음

- 버킷 집계 : 집계 중 가장 많이 사용됨, 문서의 필드를 기준으로 버킷을 집계함
- 메트릭 집계 : 문서에서  추출된 값을 가지고 sum, max, min, avg 를 계산함
- 매트릭스 집계 : 행렬의 값을 합하거나 곱함
- 파이프라인 집계 : 버킷에서 도출된 결과 문서를 다른 필드 값으로 재분류함, 즉. 다른 집계에 의해 생성된 출력 결과를 다시 한번 집계함 (집계api가 패싯api보다 강력한 이유)
