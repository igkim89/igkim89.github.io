---
layout: post
author: author1
title: Airflow task context
categories: [airflow]
description: >
  
---

# Airflow task context

## 1. Task context

작업 실행시 값을 할당하기 위한 변수

예를 들면, 다음과 같은 코드는  
```
{{ ds_nodash }}
``` 

작업 실행시 다음과 같이 값이 치환된다.  
```
20220807
```

## 2. Task context list

Python operator를 사용하여 Task context를 출력할 수 있다.

```python
import pendulum
from airflow import DAG
from airflow.operators.python import PythonOperator

dag = DAG(
    dag_id="13_task_context_list",
    start_date=pendulum.today('Asia/Seoul').add(days=-3),
    schedule_interval="@daily",
    catchup=True,
    tags=['igkim', 'test'],
)


def _print_context(**kwargs):
    print(kwargs)


print_context = PythonOperator(
    task_id="print_context",
    python_callable=_print_context,
    dag=dag,
)

print_context
```

Task context list는 다음과 같다.

|Key|Desc|Value e.g.|
|:---|:---|:---|
|conf|Airflow 구성 정보|airflow.configuration.AirflowConfigParser object|
|dag|DAG Object|DAG: 13_task_context_list|
|dag_run|DagRun Object|DagRun 13_task_context_list @ 2022-08-07 15:00:00+00:00: scheduled__2022-08-07T15:00:00+00:00, externally triggered: False|
|ds|%Y-%m-%d 형식의 execution_date|"2022-08-07"|
|ds_nodash|%Y%m%d 형식의 execution_date|"20220807"|
|execution_date|Task Schedule 간격의 시작 날짜/시간|pendulum.datetime.DateTime(2022, 8, 7, 15, 0, 0, tzinfo=Timezone('UTC'))|
|next_ds|%Y-%m-%d 형식의 다음 Schedule execution_date|"2022-08-08"|
|next_ds_nodash|%Y%m%d 형식의 다음 Schedule execution_date|"20220808"|
|next_execution_date|Task의 다음 Schedule 시간|pendulum.datetime.DateTime(2022, 8, 8, 15, 0, 0, tzinfo=Timezone('UTC'))|
|prev_ds|%Y-%m-%d 형식의 이전 Schedule execution_date|"2022-08-06"|
|prev_ds_nodash|%Y%m%d 형식의 이전 Schedule execution_date|"20220806"|
|prev_execution_date|Task의 이전 Schedule 시간|pendulum.datetime.DateTime(2022, 8, 6, 15, 0, 0, tzinfo=Timezone('UTC'))|
|prev_execution_date_success|동일한 Task중 성공적으로 완료된 가장 최근 execution_date|pendulum.datetime.DateTime(2022, 8, 6, 15, 0, 0, tzinfo=Timezone('UTC'))|
|prev_start_date_success|동일한 Task중 성공적으로 시작된 가장 최근 실제 시작 시간|pendulum.datetime.DateTime(2022, 8, 9, 4, 53, 50, 778836, tzinfo=Timezone('UTC'))|
|run_id|DagRun의 run_id|"scheduled__2022-08-07T15:00:00+00:00"|
|task|현재 Operator|Task(PythonOperator): print_context|
|task_instance|현재 Task instance|TaskInstance: 13_task_context_list.print_context scheduled__2022-08-07T15:00:00+00:00 [running]|
|task_instance_key_str|현재 Task instance의 고유 식별자|13_task_context_list__print_context__20220807|
|ti|task_instance와 동일한 현재 Task instance 객체|TaskInstance: 13_task_context_list.print_context scheduled__2022-08-07T15:00:00+00:00 [running]|
|tomorrow_ds|ds + 1 days|"2022-08-08"|
|tomorrow_ds_nodash|ds_nodash + 1 days|"20220808"|
|ts|ISO8601 포맷의 execution_date|"2022-08-07T15:00:00+00:00"|
|ts_nodash|%Y%m%dT%H%M%S 형식의 execution_date|"20220807T150000"|
|ts_nodash_with_tz|시간 정보가 있는 ts_nodash|"20220807T150000+0000"|
|yesterday_ds|ds - 1 days|"2022-08-06"|
|yesterday_ds_nodash|ds_nodash - 1 days|"20220806|