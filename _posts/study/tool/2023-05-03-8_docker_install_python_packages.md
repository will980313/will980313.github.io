---
layout: post
title: 8. Airflow docker install python packages
category: study
tags: infra

sitemap: false
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}




## Dockerfile 작성
```
FROM apache/airflow:2.5.3
COPY requirements.txt /requirements.txt
RUN pip install --user --upgrade pip
RUN pip install --no-cache-dir --user -r /requirements.txt
```
## requirments.txt 작성
```
scikit-learn == 1.0.2
matplotlib == 3.5.3
```

## dag 파일 작성

```py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator


default_args = {
    'owner': 'jueon',
    'retry': 5,
    'retry_delay': timedelta(minutes=5)
}


def get_sklearn():
    import sklearn
    print(f"sklearn with version: {sklearn.__version__} ")


def get_matplotlib():
    import matplotlib
    print(f"matplotlib with version: {matplotlib.__version__}")


with DAG(
    default_args=default_args,
    dag_id="dag_with_python_dependencies",
    start_date=datetime(2023, 5, 1),
    schedule_interval='@daily'
) as dag:
    task1 = PythonOperator(
        task_id='get_sklearn',
        python_callable=get_sklearn
    )
    
    task2 = PythonOperator(
        task_id='get_matplotlib',
        python_callable=get_matplotlib
    )

    task1 >> task2

```
## docker-compose.yaml 수정
53번째 줄 인 image를 다음과 같이 수정
```yaml
version: '3.8'
x-airflow-common:
  &airflow-common
  # In order to add custom dependencies or upgrade provider packages you can use your extended image.
  # Comment the image line, place your Dockerfile in the directory where you placed the docker-compose.yaml
  # and uncomment the "build" line below, Then run `docker-compose build` to build the images.

  #image: ${AIRFLOW_IMAGE_NAME:-apache/airflow:2.5.3} 
  image: ${AIRFLOW_IMAGE_NAME:-extending_airflow:latest} 
  # build: .
  environment:
    &airflow-common-env
```
## docker 빌드
```
docker build . --tag extending_airflow:latest
```
## docker_compose 실행
```
docker compose up -d
``` 
## 실행 확인
  
![](/assets/img/post/airflow_docker_install/docker1.png)  

![](/assets/img/post/airflow_docker_install/docker2.png)

>출처: <https://github.com/coder2j/airflow-docker>