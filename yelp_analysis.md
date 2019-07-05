# Yelp Restaurants Big Data Analysis Project Tutorial

## Step 1: Data Preparation


```
tar -zxvf yelp_dataset.tar
```
* will generate 6 jsons and some pdf fiels: ./dataset/business.json, checkin.json, photos.json, review.json, tip.json, user.json


## 파일 업로드
hue -> upload -> dataset 파일 선택



## upload data to hdfs
```
# make directories

hdfs dfs -mkdir /user/training/yelp
hdfs dfs -mkdir /user/training/yelp/business
hdfs dfs -mkdir /user/training/yelp/review
hdfs dfs -mkdir /user/training/yelp/users
hdfs dfs -mkdir /user/training/yelp/tip
hdfs dfs -mkdir /user/training/yelp/checkin


#put files
hdfs dfs -put business.json /user/training/yelp/business
hdfs dfs -put review.json /user/training/yelp/review
hdfs dfs -put users.json /user/training/yelp/users
hdfs dfs -put tip.json /user/training/yelp/tip
hdfs dfs -put checkin.json /user/training/yelp/checkin
```

* Use Hue or the HDFS command line to “put” the json files in their respective directories


## Adding RCONGUI JSON SerDe
1. Download two JAR files from http://www.congiu.net/hive-json-serde/ into your HDFS Cluster
```
wget -O json-serde-1.3.8-jar-with-dependencies.jar \ www.congiu.net/hive-json-serde/1.3.8/cdh5/json-serde-1.3.8-jar-with-dependencies.jar
wget -O json-udf-1.3.8-jar-with-dependencies.jar \ www.congiu.net/hive-json-serde/1.3.8/cdh5/json-udf-1.3.8-jar-with-dependencies.jar
```

2. Add the following at the beginning of each HIVE session:
```
ADD JAR json-serde-1.3.8-jar-with-dependencies.jar; ADD JAR json-udf-1.3.8-jar-with-dependencies.jar;
```


## json 파일을 이용하여 hive, Impala table 생성


```
#pyspark접속
pyspark2

bizDF = spark.read.json("/user/training/yelp/business/business.json")
bizDF.printSchema()
bizDF.write.parquet("/user/training/yelp/busines_parquet")
```
위와 같이 할 경우 에러가 발생함. 컬럼명에 "-"가 있으면 안됨  
아래와 같이 "-"를 "_"로 변경하는 작업을 수행함


```
text = sc.textFile("/user/training/yelp/business/business.json")

# 문자 변환
changedText = text.map(lambda s: s.replace("dairy-free","dairy_free")).map(lambda s: s.replace("gluten-free","gluten_free")).map(lambda s: s.replace("soy-free","soy_free"))


bizDF1 = spark.read.json(changedText)
bizDF1.printSchema()

bizDF1.write.parquet("/user/training/yelp/busines2_parquet")
```

## Impala에 테이블 생성
hue에서 Impala 쿼리창에서 아래 쿼리 실행
```
CREATE EXTERNAL TABLE mybiz like PARQUET '/user/training/yelp/busines2_parquet/part-00000-2f343655-88e0-4a1e-b9b9-710816be2a2b-c000.snappy.parquet'
STORED AS PARQUET
LOCATION '/user/training/yelp/biz_parquet'
```
