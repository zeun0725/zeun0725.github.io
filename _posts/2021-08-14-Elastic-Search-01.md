---
title: "ES-01 검색 시스템 이해하기"
date: 2021-08-14 18:10:00
categories: [elasticsearch]
---
# 0. 설치 버전 및 패키지, 의존성 등등 정리

## elasticsearch version: 7.14.0

## kibana version: 7.14.0

# 1. 검색 시스템 이해하기

![basic_es](https://user-images.githubusercontent.com/26589907/129473272-9f720432-843a-4c33-a24d-8973616d7ce3.png)

cluster_name : 엘라스틱 서치 클러스터를 구분하는 중요 속성이므로 유니크한 값으로 설정을 하는게 좋음

## 커스텀 플러그인 설치하기

### 엘라스틱 서치 설정정보

config 디렉터리 아래 elasticsearch.yml 파일을 수정해 변경함

클러스터 이름, 노드 이름, 로그 경로, 데이터 경로 등 다양한 설정을 지정할 수 있음

- 주요 설정 항목

    cluster.name : 클러스터 네임 (클러스터로 여러 노드를 하나로 묶을 수 있음)

    node.name: 노드명 설정

    path.repo: 인덱스를 백업하기 위한 스냅숏의 경로 지정

    network.host: 특정 ip만 접근하도록 허용

설치 참고문서: [https://kugancity.tistory.com/entry/elasticsearch-632-설치하기](https://kugancity.tistory.com/entry/elasticsearch-632-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)

### 스냅샷

ELasticsearch에서는 버전별로 데이터를 저장하여 이전상태를 유지 할 수 있는 Snapshot이라는 기능을 제공합니다. 이 모듈은 인덱스의 스탭샷 또는 클러스터 전체의 스냅샷을 만들 수 있습니다. 또한 리포트로 데이터의 저장을 지원 합니다.

### 엘라스틱 서치 실행

bin 폴더 아래에 elasticsearch 실행

```sql
./elasticsearch
```

### 엘라스틱 서치 설정 정보 변경

설정 정보는 설치 디렉터리의 config 폴더 아래에 elasticsearch.yml 파일을 수정해 변경할 수 있음

**elasticsearch.yml : 클러스터 이름, 노드 이름, 로그 경로, 데이터 경로 등 다양한 설정을 지정할 수 있음**

```yaml
# elasticsearch.yml
cluster.name: zeun-cluster
node.name: zeun-node1
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
node.master: true
node.data: true
path.repo: ["/Users/we/book_backup/search_example", "/Users/we/book_backup/agg_example"]
discovery.seed_hosts: ["127.0.0.1", "[::1]"]
cluster.initial_master_nodes: ["127.0.0.1"]

# 7.14.0 으로 바꾸고 나서 cluster.uuid 가 _na_로 인식되어서 아래와 같이 변경함
cluster.name: zeun-cluster
node.name: zeun-node1
network.host: 127.0.0.1
http.port: 9200
transport.tcp.port: 9300
node.master: true
node.data: true
path.repo: ["/Users/we/book_backup/search_example", "/Users/we/book_backup/agg_example"]
#discovery.seed_hosts: ["127.0.0.1", "[::1]"]
#cluster.initial_master_nodes: ["127.0.0.1"]
```

### 스냅숏 데이터 엘라스틱 서치로 인식 시키기

path.repo 에 설정한 두 디렉터리 중 먼저 search_example 디렉터리의 데이터를 활성화 시켜보자

```bash
# search_example 데이터가 zeun 이라는 이름의 논리적인 스냅숏으로 생성됨
curl -H "Content-Type: application/json" -XPUT 'http://localhost:9200/_snapshot/zeun' -d '{
"type": "fs",
"settings": {
"location": "/Users/we/book_backup/search_example",
"compress": true
}
}'

# 생성된 논리적인 스냅숏 확인
curl -XGET 'http://localhost:9200/_snapshot/zeun/_all'

# agg_example 디렉터리의 데이터 활성화
curl -H "Content-Type: application/json" -XPUT 'http://localhost:9200/_snapshot/apache-web-log' -d '{
"type": "fs",
"settings": {
"location": "/Users/we/book_backup/agg_example",
"compress": true
}
}'

curl -XGET 'http://localhost:9200/_snapshot/apache-web-log/_all'

```

## 키바나 설치하기

키바나 설치 후 config 폴더 밑 kibana.yml 설정 하기

```bash
elasticsearch.hosts: ["http:// 127.0.0.1:9200"]
```

키바나 실행 후

[localhost:5601](http://localhost:5601) 로 이동

## 키바나 사용방법

[http://localhost:5601/app/dev_tools#/console](http://localhost:5601/app/dev_tools#/console)

```bash
(1)GET (2)_search
{ (3)
  "query": {
    "match_all": {}
  }
}
```

(1) 요청 전달 방식: GET, POST, PUT, DELETE 지정 가능
GET: 결과 반환
POST: 변경
PUT: 삽입
DELETE: 삭제

_search는 검색 쿼리를 의미.
_search 앞부분에 인덱스를 명시해 해당 인덱스로만 범위를 한정해서 검색을 수행할 수 있음

쿼리 본문에 해당하며 여기서는 모든 문서를 검색함

## 키바나 쿼리 모음

```bash
PUT movie_kibana_execute/_doc/1
{
  "message": "hello world"
}

GET movie_kibana_execute/_search
{
  "query": {
    "match_all": {}
  }
}
```
