---
layout: post
title: 9. Airflow AWS S3 Key Sensor Operator
category: study
tags: tool

sitemap: false
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}




## Docker에서 MINIO 실행
```
docker run \
    -p 9000:9000 \
    -p 9001:9001 \
    -e "MINIO_ROOT_USER=AKIAIOSFODNN7EXAMPLE" \
    -e "MINIO_ROOT_PASSWORD=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
    quay.io/minio/minio server /data --console-address ":9001"
```
## MINIO 접속
http://127.0.0.1:9001/
```
ID: AKIAIOSFODNN7EXAMPLE
PW: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```
![](/assets/img/post/airflow_s3/s3_1.png)  
## Bucket 생성
Create a Bucket 클릭
![](/assets/img/post/airflow_s3/s3_2.png)  
Bucket Name: airflow 설정 후 Creat Bucket 클릭 
![](/assets/img/post/airflow_s3/s3_3.png)  
## Access Key 생성
Access Keys > Create access key 클릭  
id와 key 복사
![](/assets/img/post/airflow_s3/s3_6.png)
## data.csv 작성
data/data.csv 생성
```
product_id, delivery_dt
10005, 2023-01-01
10001, 2022-12-30
```
## data.csv 업로드
Object Browser 클릭  

![](/assets/img/post/airflow_s3/s3_4.png)

생성한 data.csv 업로드  
![](/assets/img/post/airflow_s3/s3_5.png)  

## dag 작성
```py
from datetime import datetime, timedelta
from airflow import DAG
from aairflow.providers.amazon.aws.sensors.s3 import S3KeySensor


default_args = {
    'owner': 'jueon',
    'retries': 5,
    'retry_delay': timedelta(minutes=10)
}


with DAG(
    dag_id='dag_with_minio_s3',
    start_date=datetime(2023, 4, 1),
    schedule_interval='@daily',
    default_args=default_args
) as dag:
    task1 = S3KeySensor(
        task_id='sensor_minio_s3',
        bucket_name='airflow',
        bucket_key='data.csv',
        aws_conn_id='minio_conn',
        mode='poke',
        poke_interval=5,
        timeout=30
    )
```
## docker_compose 실행
```
docker compose up airflow-init
docker compose up -d
``` 
## MINIO 연결
airflow 웹에 접속해 Admin > Connections 클릭  
\+ 클릭
  
![](/assets/img/post/airflow_s3/s3_7.png)
아래와 같이 config 작성    
![](/assets/img/post/airflow_s3/s3_8.png)
test 버튼을 누르면 클라이언트 오류가 발생하지만 정상적으로 연결됨  

## 실행 확인
![](/assets/img/post/airflow_s3/s3_9.png)  
![](/assets/img/post/airflow_s3/s3_10.png)


>출처: <https://github.com/coder2j/airflow-docker>