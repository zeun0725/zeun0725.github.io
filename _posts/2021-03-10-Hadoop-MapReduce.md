---
title: "Hadoop - MapReduce"
date: 2021-03-10 22:30:00 -0400
categories: [hadoop]
---
# 하둡 완벽 가이드: 2장 맵리듀스

## 맵리듀스란?

디스크에서 데이터를 읽고 쓰는 문제를 키-값 쌍의 계산으로 변환한 추상화 된 프로그래밍 모델

대표적인 대용량 데이터 처리를 위한 병렬 처리 기법

![mapreduce](https://user-images.githubusercontent.com/26589907/110638072-75e55f80-81f1-11eb-805b-e183ddce63c0.png)

## 맵리듀스를 활용하여 기상 데이터 셋 분석하기

### 목적 및 데이터 포맷

연도별 전 세계 최고 기온 구하기

gzip 압축 파일, 구분자 없이 필드가 한 행으로 붙어 있음

## 유닉스 도구로 데이터 분석 하기

```bash
for year in all/*
do
	echo -ne `basename $year .gz`"\t"
gunzip -c $year | \
	awk '{ temp = substr($0, 88, 5) + 0;
				 q = substr($0, 93, 1);
				 if (temp !=9999 && q ~ /[01459]/ && temp > max) max = temp }
			END { print max }'
done
```

전체 데이터 EC2 고성능 인스턴스에서 실행해 본 결과 42분이 걸림

## 하둡으로 데이터 분석하기

1) 맵 단계

입력: 원본 데이터

맵 함수 역할: 각 행에서 연도와 기온을 추출, 잘못된 레코드를 걸러주는 작업

출력: 연도와 기온 (ex) (1950, 0) (1950, 22) (1951, 27) ...

key : 연도

value : 기온

2) 리듀스 단계

입력: 연도별로 측정된 모든 기온 값(리스트 형식) (ex) (1949, [111,78,22,24]) (1950, [0,22,-5]) ...

리듀스 함수 역할: 리스트 전체를 반복해 최고 기온 값을 추출

출력: 연도와 최고 기온 (ex) (1949, 111) ( 1950, 22) ...

### 자바 맵리듀스

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;

public class MaxTemperatureMapper extends Mapper<T> {
	//Mapper : 4개의 정규 타입 매개변수를 가짐(입력키, 입력값, 출력키, 출력값)
	@Override
	public void map(LongWritable key, Text value, Context context) {
		...
		//출력을 위한 context의 인스턴스 제공
		context.write(new Text(year), new IntWritable(airTemperature));
	}
}

public class MaxTemperatureReducer extends Reducer<T> {
	@Override
	public void reduce(Text key, Iterable<IntWritable> values, Context context) {
		//리듀스의 입력타입은 맵함수의 출력타입인 Text와 IntWritable
		...
		context.write(key, new IntWritable(maxValue));
	}
}

```

### 자바 맵리듀스 잡을 구동하는 코드

```java
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MaxTemperature {
	public static void main(String[] args) throws Exception {

		//Job 객체는 잡 명세서를 작성 함
		//JAR 파일로 묶으면 하둡은 클러스터의 해당 머신에 JAR 파일을 배포 함
		Job job = new Job();
		job.setJarByClass(MaxTemperature.class);
		job.setJobName("Max Temperature");

		//입출력 경로 지정
		...

		//맵, 리듀스 입출력 데이터 타입 지정
		...

		// 잡이 끝날 때까지 기다리며 진척 상황을 콘솔로 보고
		System.exit(job.waitForCompletion(true) ? 0 : 1);

	}
}
```

### 실행 로그

```bash
export HADOOP_CLASSPATH=hadoop-examples.jar
hadoop MaxTemperature *.txt output
```

m_0000_0 ⇒ map job // r_0000_0 ⇒ reduce job

매퍼에 있어 정상적인 입력 레코드는 한번에 하나의 출력 레코드를 생성함

리듀스 당 하나의 출력 파일이 생성 됨

## 분산형으로 확장하기

### 데이터 흐름

맵리듀스 잡: 클라이언트가 수행하는 작업의 기본 단위

하둡은 잡을 맵 태스크, 리듀스 태스크로 나누어 실행함

전체 데이터를 hdfs에 저장

각 태스크는 YARN을 이용하여 스케줄링되고 클러스터의 여러 노드에서 실행 됨.

Q) 실행중인 특정 노드의 태스크가 실패된다면?

A) 자동으로 다른 노드를 재할당하여 다시 실행 됨.

### 스플릿

잡의 입력을 고정크기 조각으로 분리하는 것

**스플릿을 하는 이유**

전체 입력을 통으로 처리하는 것 보다 스플릿으로 분리된 조각을 각각 처리하는 것이 훨씬 빠름 ⇒ 너무 작으면 스플릿 관리, 맵태스크 생성을 위한 오버헤드가 발생함 ⇒ 적절한 크기는 HDFS 블록의 기본 크기(클러스터의 설정에 따라 다름)

### 데이터 지역성 최적화

- 입력데이터가 있는 노드에서 맵 태스크를 실행할 때 가장 빠르게 작동 함
- 맵 태스크의 입력 스플릿에 해당하는 HDFS 블록 복제본이 저장된 세 개의 노드 모두 다른 맵 태스크를 실행하여 여유가 없을 경우 블록 복제본이 저장된 동일 랙에 속한 다른 노드에서 가용한 맵 슬롯을 찾음
- 데이터 복제본이 저장된 노드가 없는 외부 랙의 노드가 선택될 경우 랙 간에 네트워크 전송이 불가피하게 일어남

**리듀스태스크는 모든 맵의 결과를 입력으로 받으므로 데이터 지역성의 장점이 없다**

![rack](https://user-images.githubusercontent.com/26589907/110638087-78e05000-81f1-11eb-9e35-cd3cedcd45c0.png)

### 태스크 결과

Map

- 로컬에 저장함
- 중간결과물로서 잡이 완료되면 삭제 됨

Reduce

- Hdfs에 저장함
- hdfs블록의 첫번째 복제본은 로컬노드에 저장, 나머지 복제본은 외부 랙에 저장

**리듀스 태스크가 하나일 경우**

![reduce](https://user-images.githubusercontent.com/26589907/110638100-7bdb4080-81f1-11eb-9fe3-0d17ffe5b315.png)

**리듀스 태스크가 여럿일 경우**

![reduce](https://user-images.githubusercontent.com/26589907/110638110-7e3d9a80-81f1-11eb-9013-5a792ec011bd.png)

리듀스가 여럿이면 맵 태스크는 리듀스 수만큼 파티션을 생성하고 맵의 결과를 각 파티션에 분배.

셔플이 일어남 ⇒ 리듀스의 수를 선택하는 것은 잡의 실행 시간에 영향을 미치므로 튜닝이 필요

### 분산 맵리듀스 잡

EC2 10개 노드 ⇒ 6분

## 하둡 스트리밍

하둡은 자바 외에 다른 언어로 맵과 리듀스 함수를 작성할 수 있는 맵리듀스 API를 제공함

### 파이썬 코드 (맵 함수)

```python
import re
import sys

for line in sys.stdin:
	val = line.strip()
	(year, temp, q) = (val[15:19], val[87:92], val[92,93])
	if (temp != "+9999" and re.match("[01459]", q):
		print "%s\t%s" % (year, temp)
```

### 파이썬 코드 (리듀스 함수)

```python
import sys

(last_key, max_val) = (None, -sys.maxint)
for line in sys.stdin:
	(key, val) = line.strip().split("\t")
	if last_key and last_key != key:
		print "%s\t%s" % (last_key, max_val)
		(last_key, max_val) = (key, int(val))
	else:
		(last_key, max_val) = (key, max(max_val, int(val)))

if last_key:
	print "%s\t%s" % (last_key, max_val)
```
