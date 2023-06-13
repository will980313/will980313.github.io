---
layout: post
title: 4. Airflow decorateor
category: study
tags: tool

sitemap: false
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}

## dag 파일 작성
```py
from datetime import datetime, timedelta

# 데코레이터 import
from airflow.decorators import dag, task


default_args = {
    'owner': 'jueon',
    'retries': 5,
    'retry_delay': timedelta(minutes=5)
}

# dag 선언
@dag(dag_id='dag_with_taskflow_api', 
     default_args=default_args, 
     start_date=datetime(2023, 4, 27), 
     schedule_interval='@daily')
def hello_world_etl():

    # task 선언
    @task(multiple_outputs=True)
    def get_name():
        return {
            'first_name': 'Jerry',
            'last_name': 'Fridman'
        }

    @task()
    def get_age():
        return 19

    @task()
    def greet(first_name, last_name, age):
        print(f"Hello World! My name is {first_name} {last_name} "
              f"and I am {age} years old!")
    
    # 함수 기반으로 flow 생성
    name_dict = get_name()
    age = get_age()
    greet(first_name=name_dict['first_name'], 
          last_name=name_dict['last_name'],
          age=age)

# dag 함수 실행
greet_dag = hello_world_etl()
```
## 작동 확인
#### http://localhost:8080 접속
#### 로그 확인
![](/assets/img/post/airflow_decorator/decorator1.png)


  
>출처: <https://github.com/coder2j/airflow-docker>