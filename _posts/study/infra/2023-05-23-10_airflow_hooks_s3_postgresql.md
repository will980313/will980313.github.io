---
layout: post
title: 10. Airflow Hooks S3 PostgreSQL
category: study
tags: infra

sitemap: false
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}



## postgres 설정

[postgres post](https://will980313.github.io/study/2023-04-29-7_airflow_postgres)

위 포스트를 참고하여 postgres 설정

docker-compose.yaml 변경  


```yaml
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    volumes:
      - postgres-db-volume:/var/lib/postgresql/data
      - ${AIRFLOW_PROJ_DIR:-.}/data:/data
    ports:
      - 5432:5432
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "airflow"]
      interval: 10s
      retries: 5
      start_period: 5s
    restart: always
```




docker_compose 실행  
```
docker compose up airflow-init
docker compose up -d
``` 



Orders.csv 다운  
https://github.com/coder2j/airflow-docker/blob/main/data/Orders.csv  
data 디렉토리로 이동
  
postgres 도커에 접속 후 postgres에 접속  
```
psql -U airflow airflow
```
postgres 테이블 생성  
```sql
\c test

CREATE TABLE if NOT EXISTS public.orders(
    order_id character varying,
    date DATE,
    product_name character varying,
    quantity integer,
    PRIMARY KEY (order_id)
);
```
데이터베이스에 csv import 
```sql
COPY public.orders(order_id, date, product_name,quantity)
FROM '/data/Orders.csv' WITH delimiter ',' csv header;
```
import 확인
```sql
SELECT * FROM public.orders LIMIT 100;
```
![](/assets/img/post/airflow_hook/hook_1.png)  

## s3 설정

[s3 post](https://will980313.github.io/study/2023-05-17-9_aws_s3_key_sensor_operator)  
위 포스트를 참고해 s3 연결  

## dag 작성

```py
import csv
import logging
from datetime import datetime, timedelta
from tempfile import NamedTemporaryFile

from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.providers.amazon.aws.hooks.s3 import S3Hook


default_args = {
    'owner': 'jueon',
    'retries': 5,
    'retry_delay': timedelta(minutes=10)
}

def postgres_to_s3(ds_nodash, next_ds_nodash):
    # step 1: query data from postgresql db and save into text file
    hook = PostgresHook(postgres_conn_id="postgres_localhost")
    conn = hook.get_conn()
    cursor = conn.cursor()
    cursor.execute("select * from orders where date >= %s and date < %s",
                   (ds_nodash, next_ds_nodash))
    with NamedTemporaryFile(mode='w', suffix=f"{ds_nodash}") as f:
        csv_writer = csv.writer(f)
        csv_writer.writerow([i[0] for i in cursor.description])
        csv_writer.writerows(cursor)
        f.flush()
        cursor.close()
        conn.close()
        logging.info("Saved orders data in text file: %s", f"dags/get_orders_{ds_nodash}.txt")
    # step 2: upload text file into S3
        s3_hook = S3Hook(aws_conn_id="minio_conn")
        s3_hook.load_file(
            filename=f.name,
            key=f"orders/{ds_nodash}.txt",
            bucket_name="airflow",
            replace=True
        )
        logging.info("Orders file %s has been pushed to S3!", f.name)


with DAG(
    dag_id="dag_with_postgres_hooks",
    default_args=default_args,
    start_date=datetime(2022, 4, 30),
    schedule_interval='@daily'
) as dag:
    task1 = PythonOperator(
        task_id="postgres_to_s3",
        python_callable=postgres_to_s3
    )
    task1
```



## 실행 확인
airflow 브라우저  
![](/assets/img/post/airflow_hook/hook_2.png)    
![](/assets/img/post/airflow_hook/hook_3.png)    
minio > Object Browser > airflow > orders       
![](/assets/img/post/airflow_hook/hook_4.png)    
저장된 txt파일  
![](/assets/img/post/airflow_hook/hook_5.png)    
  
>출처: <https://github.com/coder2j/airflow-docker>