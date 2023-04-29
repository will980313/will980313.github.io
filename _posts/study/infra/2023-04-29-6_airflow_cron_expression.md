---
layout: post
title: 6. Airflow cron expression
category: study
tags: infra

sitemap: false
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}





## catch up
chatch up을 사용하여 start_date가 과거로 설정되어 있는 dag가 재실행 되었을 경우 과거의 작업을 실행할 지 현재 작업만 실행할 지 정할 수 있다.

## Cron Expression Preset
airflow는 cron을 사용하여 자세하게 스케줄링을 할 수 있다.  
cron은 맨 앞부터 분 시 일 월 요일로 나타낸다.
예를 들면 5 12 * * SAT는 매주 토요일 12시 5분이라는 것이다.
요일은 00\**MON,WED,FRI 처럼 여러개 선택 가능하고 00\**MON-FRI 처럼 연속된 요일을 한번에 나타낼 수 있다.
|preset|meaning|cron|  
|:-:|:-:|:-:|
|None|Don't schedule, use for exclusively "externally triggered" DAGs||
|@once|Schedule once and only once||
|@hourly|Run once an hour at the beginning of the hour|0****|
|@daily|Run once a day at midnight|00***|
|@weekly|Run once a week at midnight on Sunday morning|00**0|
|@monthly|Run once a month at midnight of the first day of the month|001**|
|@yearly|Run once a year at midnight of January 1|0011*|



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
    default_args=default_args,
    dag_id="dag_with_cron_expression",
    start_date=datetime(2023, 1, 1),
    schedule_interval='0 3 * * MON,WED,FRI'
) as dag:
    task1 = BashOperator(
        task_id='task1',
        bash_command="echo dag with cron expression!"
    )
    task1
```
## 작동 확인
#### http://localhost:8080 접속 
![](/assets/img/post/airflow_cron/cron1.png)

