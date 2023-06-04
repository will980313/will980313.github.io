---
layout: post
title: 2. Airflow dag 만들기
category: study
tags: infra

sitemap: false
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}

## dag 파일 생성
#### project 디렉토리 내 dags 디렉토리에 my_first_dag.py 파일 생성   
![](/assets/img/post/airflow_dag_만들기/dag1.png)

## dag 파일 작성
```py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator

# Task 공통 속성값 정의
default_args = {
    'owner': 'jueon',
    'retries': 5,
    'retry_delay': timedelta(minutes=2)
}

# Dag 정의
with DAG(
    dag_id='my_first_dag',
    default_args=default_args,
    description='This is our first dag that we write',
    start_date=datetime(2023, 4, 25),
    schedule_interval='@daily'
) as dag:
    
    # Task 정의
    task1 = BashOperator(
        task_id='first_task',
        bash_command="echo hello world, this is the first task!"
    )

    task2 = BashOperator(
        task_id='second_task',
        bash_command="echo hey, I am task2 and will be running after task1!"
    )

    task3 = BashOperator(
        task_id='thrid_task',
        bash_command="echo hey, I am task3 and will be running after task1 at the same time as task2!"
    )

    # Task 종속성
    # task는 다른 task들과 종속성을 가진다


    # Task 종속성을 나타내는 방법들

    # Task dependency method 1
    # task1.set_downstream(task2)
    # task1.set_downstream(task3)

    # Task dependency method 2
    # task1 >> task2
    # task1 >> task3

    # Task dependency method 3
    task1 >> [task2, task3]
```
## 작동 확인
#### http://localhost:8080 접속
#### dag unpause 후 클릭  
![](/assets/img/post/airflow_dag_만들기/dag2.png)  
  
### 좌측 Task들의 초록색 박스 클릭
![](/assets/img/post/airflow_dag_만들기/dag3.png)
  
### first_task 로그확인
![](/assets/img/post/airflow_dag_만들기/dag4.png)

### second_task 로그확인
![](/assets/img/post/airflow_dag_만들기/dag5.png)

### thrid_task 로그확인
![](/assets/img/post/airflow_dag_만들기/dag6.png)

### task 그래프 확인
![](/assets/img/post/airflow_dag_만들기/dag7.png)


>출처: <https://github.com/coder2j/airflow-docker>