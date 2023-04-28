---
layout: post
title: 5. Airflow catch up & back fill
category: study
tags: infra

sitemap: false
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}


## catch up
chatch up을 사용하여 start_date가 과거로 설정되어 있는 dag가 재실행 되었을 경우 과거의 작업을 실행할 지 현재 작업만 실행할 지 정할 수 있다.

## dag 파일 작성

```py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator


default_args = {
    'owner': 'jueon',
    'retries': 5,
    'retry_delay': timedelta(minutes=5)
}

with DAG(
    dag_id='dag_with_catchup_backfill',
    default_args=default_args,
    start_date=datetime(2023, 1, 28),
    schedule_interval='@daily',
    # catchup=False
    catchup=True
) as dag:
    task1 = BashOperator(
        task_id='task1',
        bash_command='echo This is a simple bash command!'
    )
```
## 작동 확인
#### http://localhost:8080 접속
#### catch up True  
![](/assets/img/post/airflow_catch_up/catch_up1.png)

#### catch up False  
![](/assets/img/post/airflow_catch_up/catch_up2.png)

## back fill
back fill은 dag 실행중 과거에 처리되지 않은 작업을 수행하거나 start_date 이전 작업까지 수행할 때 사용한다.


docker ps 사용해 airflow-scheduler 도커를 찾아 docker exec를 사용해 접속한다. 

![](/assets/img/post/airflow_catch_up/back_fill1.png)

접속 후 backfill 명령어를 사용해 2022년 1월 28일 부터 23년 1월 28일까지 dag를 실행해 주었다. 
```
airflow dags backfill -s 2022-01-28 -e 2023-01-28 dag_with_catchup_backfill
```
## 작동 확인
http://localhost:8080 접속

![](/assets/img/post/airflow_catch_up/back_fill2.png)

>출처: https://github.com/coder2j/airflow-docker