# ML Pipeline Orchestration with Apache Airflow

Tags: Airflow, Docker, ML Pipeline, MLOps
Created: October 31, 2024 9:46 PM

# Objectives:

### What is ML pipeline?

A Machine Learning (ML) pipeline is a structured workflow that automates and streamlines the process of developing and deploying a machine learning model. It encompasses several steps starting from data collection and preprocessing to train the model and evaluate it as well as deployment. 

![Screenshot 2024-11-01 at 5.01.53 AM.png](ML%20Pipeline%20Orchestration%20with%20Apache%20Airflow%20130d4718bce88002bd00d09a363f69d4/Screenshot_2024-11-01_at_5.01.53_AM.png)

# Table of contents:

1. Setting up the environment for Apache Airflow.
2. Creating DAG file to scheduling the python operation.
3. Creating ML pipeline.
4. Initializing the docker to monitor the DAGs it on Apache Airflow webserver.
5. Run and test the DAGs.

## Step 01: Setting up the environment for Apache Airflow.

1. Update system package.

```python
sudo apt update
sudo apt upgrade -y
```

1. Installing the python development tools.

```python
sudo apt install python3-pip python3-dev build-essential
```

1. Creating and activating the virtual environment.

```python
sudo pip3 install virtualenv
```

```python
python3 -m venv newapp
```

```python
source newapp/bin/activate
```

1. Setting airflow home directory.

```python
export AIRFLOW_HOME=~/airflow
```

1. Installing apache airflow using pip.
- Installing with constrain for version compatibility.

```python
AIRFLOW_VERSION=2.8.1
PYTHON_VERSION="$(python3 --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
```

- Installing Airflow with extras for common use cases.

```python
pip install "apache-airflow==${AIRFLOW_VERSION}" \
--constraint "[https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt](https://raw.githubusercontent.com/apache/airflow/constraints-$%7BAIRFLOW_VERSION%7D/constraints-$%7BPYTHON_VERSION%7D.txt)"
```

1. Initializing the Airflow database.

```python
airflow db init
```

1. Starting Airflow webserver and scheduler.

```python
airflow webserver --port 8080
```

```python
airflow scheduler
```

## Step 02: Creating the DAG file to schedule the python operations.

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime, timedelta
from pipeline import (ingest_data, preprocess_data, train_model,
                     evaluate_model, deploy_model)

default_args = {
    'owner': 'your_name',
    'start_date': datetime(2024, 10, 20),
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'ml_pipeline',
    default_args=default_args,
    description='ML pipeline using Airflow',
    schedule_interval=timedelta(days=1),
)

t1 = PythonOperator(
    task_id='ingest_data',
    python_callable=ingest_data,
    dag=dag,
)

t2 = PythonOperator(
    task_id='preprocess_data',
    python_callable=preprocess_data,
    dag=dag,
)

t3 = PythonOperator(
    task_id='train_model',
    python_callable=train_model,
    dag=dag,
)

t4 = PythonOperator(
    task_id='evaluate_model',
    python_callable=evaluate_model,
    dag=dag,
)

t5 = PythonOperator(
    task_id='deploy_model',
    python_callable=deploy_model,
    dag=dag,
)

t1 >> t2 >> t3 >> t4 >> t5
```

## Step 03: Creating a folder for Python Operators for doing the python operations.

```python
def ingest_data(**kwargs):
    # Your data ingestion code here
    pass

def preprocess_data(**kwargs):
    # Your data preprocessing code here
    pass

def train_model(**kwargs):
    # Your model training code here
    pass

def evaluate_model(**kwargs):
    # Your model evaluation code here
    pass

def deploy_model(**kwargs):
    # Your model deployment code here
    pass

```

## Step 04: Initializing the Docker.

1. Create a docker-compose.yaml file.

```
services:
  airflow-init:
    image: apache/airflow:2.10.2
    environment:
      - AIRFLOW__CORE__EXECUTOR=LocalExecutor
      - AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres/airflow
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
      - ./data:/opt/airflow/data
    entrypoint: airflow db init

  airflow-webserver:
    image: apache/airflow:2.10.2
    depends_on:
      - airflow-init
    environment:
      - AIRFLOW__CORE__EXECUTOR=LocalExecutor
      - AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres/airflow
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
      - ./data:/opt/airflow/data
    ports:
      - "8081:8080"
    command: webserver

  airflow-scheduler:
    image: apache/airflow:2.10.2
    depends_on:
      - airflow-webserver
    environment:
      - AIRFLOW__CORE__EXECUTOR=LocalExecutor
      - AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres/airflow
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
      - ./data:/opt/airflow/data
    command: scheduler

  postgres:
    image: postgres:13
    environment:
      - POSTGRES_USER=airflow
      - POSTGRES_PASSWORD=airflow
      - POSTGRES_DB=airflow
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

Here we have used this version of docker-compose.yaml file.

1. Create the required directories.

```python
mkdir -p ./dags ./logs ./plugins ./config
```

1. Setting the Airflow user id.

```python
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

1. Initializing the Airflow database.

```python
docker compose up airflow-init
```

1. Starting all devices.

```python
docker compose up -d
```

![Screenshot 2024-12-19 at 12.06.45.png](ML%20Pipeline%20Orchestration%20with%20Apache%20Airflow%20130d4718bce88002bd00d09a363f69d4/f2d91925-a702-47cc-864a-056bc71bd1cb.png)

1. To check running containers.

```python
docker ps
```

1. To stop all containers.

```python
docker-compose down
```

![Screenshot 2024-12-19 at 12.06.58.png](ML%20Pipeline%20Orchestration%20with%20Apache%20Airflow%20130d4718bce88002bd00d09a363f69d4/e601de61-fd6a-4dc2-a9e5-6b435da5fc97.png)

## Step 05: Run and test the file.

To run and test the file, we can expose the Airflow GUI by Poridhi Load Balancer. For that,

- Go to the load balancer.

![Screenshot 2024-11-20 at 8.47.51 PM.png](ML%20Pipeline%20Orchestration%20with%20Apache%20Airflow%20130d4718bce88002bd00d09a363f69d4/Screenshot_2024-11-20_at_8.47.51_PM.png)

- Use ifconfig to find your VM’s IP.

![Screenshot 2024-11-20 at 8.48.41 PM.png](ML%20Pipeline%20Orchestration%20with%20Apache%20Airflow%20130d4718bce88002bd00d09a363f69d4/Screenshot_2024-11-20_at_8.48.41_PM.png)

- Create the load balancer with your IP and Airflow exposing port which is 8081 here and launch it.

![Screenshot 2024-11-20 at 8.49.13 PM.png](ML%20Pipeline%20Orchestration%20with%20Apache%20Airflow%20130d4718bce88002bd00d09a363f69d4/Screenshot_2024-11-20_at_8.49.13_PM.png)

- After logging in to Airflow using credentials, it will provide visuals of your created DAGs.

![Screenshot 2024-12-19 at 12.03.11.png](ML%20Pipeline%20Orchestration%20with%20Apache%20Airflow%20130d4718bce88002bd00d09a363f69d4/Screenshot_2024-12-19_at_12.03.11.png)

![Screenshot 2024-12-19 at 12.03.29.png](ML%20Pipeline%20Orchestration%20with%20Apache%20Airflow%20130d4718bce88002bd00d09a363f69d4/Screenshot_2024-12-19_at_12.03.29.png)

![Screenshot 2024-12-19 at 12.03.51.png](ML%20Pipeline%20Orchestration%20with%20Apache%20Airflow%20130d4718bce88002bd00d09a363f69d4/Screenshot_2024-12-19_at_12.03.51.png)

# Conclusion:

This documentation involves the ML pipeline orchestrations with Apache Airflow.