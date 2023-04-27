---
layout: post
title: Airflow xcom
category: study
tags: infra

sitemap: false
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}

## dag 파일 작성
```py
from datetime import datetime, timedelta

from airflow import DAG
from airflow.operators.python import PythonOperator


default_args = {
    'owner': 'jueon',
    'retries': 5,
    'retry_delay': timedelta(minutes=5)
}


## task에서 사용할 함수 선언
## xcom을 사용해 task들 끼리 통신
## xcom에 key, value를 사용해 딕셔너리로 push
## task_id와 key를 이용해 xcom에서 pull

def greet(some_dict, ti):
    print("some dict: ", some_dict)
    first_name = ti.xcom_pull(task_ids='get_name', key='first_name')
    last_name = ti.xcom_pull(task_ids='get_name', key='last_name')
    age = ti.xcom_pull(task_ids='get_age', key='age')
    print(f"Hello World! My name is {first_name} {last_name}, "
          f"and I am {age} years old!")


def get_name(ti):
    ti.xcom_push(key='first_name', value='Jerry')
    ti.xcom_push(key='last_name', value='Fridman')


def get_age(ti):
    ti.xcom_push(key='age', value=19)


with DAG(
    default_args=default_args,
    dag_id='our_dag_with_python_operator',
    description='Our first dag using python operator',
    start_date=datetime(2023, 4, 26),
    schedule_interval='@daily'
) as dag:
    task1 = PythonOperator(
        task_id='greet',
        python_callable=greet,

        # op_kwargs를 사용해 인자 전달도 가능
        op_kwargs={'some_dict': {'a': 1, 'b': 2}} 
    )

    task2 = PythonOperator(
        task_id='get_name',
        python_callable=get_name
    )

    task3 = PythonOperator(
        task_id='get_age',
        python_callable=get_age
    )

    [task2, task3] >> task1
```
## 작동 확인
#### http://localhost:8080 접속
#### 로그 확인
![](/assets/img/post/airflow_xcom/xcom1.png)


  
>출처: https://github.com/coder2j/airflow-docker