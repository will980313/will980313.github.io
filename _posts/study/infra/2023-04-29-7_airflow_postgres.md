---
layout: post
title: 7. Airflow postgres
category: study
tags: infra

sitemap: false
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}
## catch up
chatch up을 사용하여 start_date가 과거로 설정되어 있는 dag가 재실행 되었을 경우 과거의 작업을 실행할 지 현재 작업만 실행할 지 정할 수 있다.

## docker-compose.yaml 수정
postgres에 ports 추가

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
    ports:
      - 5432:5432
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "airflow"]
      interval: 10s
      retries: 5
      start_period: 5s
    restart: always
```
```
docker compose down
docker compose up airflow-init
docker compose up -d
```

## postgres에 database 추가
![](/assets/img/post/airflow_postgres/postgres1.png)
postgres 도커에 접속
```sql
psql -U airflow airflow
CREATE DATABASE test;
```
database 추가

## connections 등록
http://localhost:8080 접속 후 상당 탭의 Admin -> Connections 클릭
![](/assets/img/post/airflow_postgres/postgres2.png)
\+ 버튼을 눌러 connection 등록
![](/assets/img/post/airflow_postgres/postgres3.png)  
host는 서버에서 웹에 접속했을 경우는 localhost, 서버와 다른 곳에서 웹에 접속했을 경우는 서버의 아이피를 입력  
Test 버튼을 눌러 서버 연결 확인  
## dag 파일 작성


```py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.providers.postgres.operators.postgres import PostgresOperator


default_args = {
    'owner': 'jueon',
    'retries': 5,
    'retry_delay': timedelta(minutes=5)
}


with DAG(
    dag_id='dag_with_postgres_operator',
    default_args=default_args,
    start_date=datetime(2023, 1, 1),
    schedule_interval='0 0 * * *'
) as dag:
    # 테이블 생성
    task1 = PostgresOperator(
        task_id='create_postgres_table',
        postgres_conn_id='postgres_localhost',
        sql="""
            create table if not exists dag_runs (
                dt date,
                dag_id character varying,
                primary key (dt, dag_id)
            )
        """
    )
    # 테이블에 데이터 추가
    task2 = PostgresOperator(
        task_id='insert_into_table',
        postgres_conn_id='postgres_localhost',
        sql="""
            insert into dag_runs (dt, dag_id) values ('{{ ds }}', '{{ dag.dag_id }}')
        """
    )
    # 테이블 데이터 삭제
    # dag를 재 실행할 경우 해당 데이터가 존재하므로 삭제 함
    task3 = PostgresOperator(
        task_id='delete_data_from_table',
        postgres_conn_id='postgres_localhost',
        sql="""
            delete from dag_runs where dt = '{{ ds }}' and dag_id = '{{ dag.dag_id }}';
        """
    )
    task1 >> task3 >> task2
```

## 작동 확인
postgres 도커 접속 후 psql 접속
```sql
\c test
SELECT * FROM public.dag_runs
``` 
데이터 출력 확인
![](/assets/img/post/airflow_postgres/postgres4.png)

