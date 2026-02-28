---
name: airflow-3x-migration
description: Comprehensive guide and patterns for migrating Apache Airflow 2.x workflows to Airflow 3.x, covering import changes, deprecated features, and new paradigms like Asset scheduling and TaskFlow API.
version: 1.0.0
---

# Airflow 3.x Skills

## Import Path Changes

### Operators
```python
# Airflow 2.x
from airflow.operators.python import PythonOperator

# Airflow 3.x
from airflow.providers.standard.operators.python import PythonOperator
from airflow.providers.standard.operators.bash import BashOperator
from airflow.providers.standard.operators.empty import EmptyOperator
```

### Sensors
```python
# Airflow 3.x
from airflow.providers.standard.sensors.filesystem import FileSensor
from airflow.providers.standard.sensors.time import TimeSensor
```

## Removed Features

| Removed | Replacement |
|---------|-------------|
| `SubDagOperator` | `TaskGroup` |
| `packaged_dag_processor` | Use standard DAG loading |
| `airflow.contrib.*` | Provider packages |
| `schedule_interval` param | `schedule` param |

## DAG Definition Changes

```python
# Airflow 3.x preferred
from airflow import DAG
from datetime import datetime

with DAG(
    dag_id="my_dag",
    schedule="@daily",  # Not schedule_interval
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=["betting", "sports"],
) as dag:
    ...
```

## TaskFlow API (Preferred)

```python
from airflow.decorators import dag, task

@dag(schedule="@daily", start_date=datetime(2024, 1, 1), catchup=False)
def betting_workflow():

    @task
    def download_games(sport: str) -> list:
        # Returns are automatically passed via XCom
        return fetch_games(sport)

    @task
    def update_elo(games: list) -> dict:
        return calculate_elo(games)

    # Chain tasks
    games = download_games("nba")
    ratings = update_elo(games)

betting_dag = betting_workflow()
```

## Asset-Based Scheduling (Replaces Dataset)

```python
from airflow.sdk import Asset

# Define assets
games_data = Asset("games_data")
elo_ratings = Asset("elo_ratings")

# Producer DAG
@dag(schedule="@daily")
def download_dag():
    @task(outlets=[games_data])
    def download():
        ...

# Consumer DAG - triggers when asset updates
@dag(schedule=[games_data])
def process_dag():
    @task
    def process():
        ...
```

## Setup/Teardown Tasks

```python
@task
def setup_db_connection():
    return create_connection()

@task
def cleanup_connection(conn):
    conn.close()

@task
def process_data(conn):
    ...

# Define setup/teardown relationship
with dag:
    conn = setup_db_connection()
    process_data(conn) >> cleanup_connection(conn)

    # Or use context manager style
    conn.as_setup() >> process_data(conn) >> conn.as_teardown()
```

## DAG Versioning

```python
from airflow import DAG

with DAG(
    dag_id="betting_workflow",
    version="2.0.0",  # New in 3.x
    schedule="@daily",
) as dag:
    ...
```

## Backfill Changes

```bash
# Airflow 3.x - use REST API
curl -X POST "http://localhost:8080/api/v1/dags/my_dag/dagRuns" \
  -H "Content-Type: application/json" \
  -d '{"logical_date": "2024-01-15T00:00:00Z"}'

# Or use new backfill command
airflow dags backfill my_dag --start-date 2024-01-01 --end-date 2024-01-15
```

## New REST API Endpoints

```python
import requests

# Get DAG runs
response = requests.get(
    "http://localhost:8080/api/v1/dags/betting_workflow/dagRuns",
    auth=("admin", "admin")
)

# Trigger DAG
response = requests.post(
    "http://localhost:8080/api/v1/dags/betting_workflow/dagRuns",
    json={"conf": {"sport": "nba"}},
    auth=("admin", "admin")
)
```

## Edge Labels

```python
from airflow.utils.edgemodifier import Label

download >> Label("success") >> process
download >> Label("failure") >> alert
```

## Migration Checklist

- [ ] Update all operator imports to provider packages
- [ ] Replace `schedule_interval` with `schedule`
- [ ] Convert SubDags to TaskGroups
- [ ] Replace Dataset with Asset
- [ ] Test DAG parsing with `python dags/my_dag.py`
- [ ] Update docker-compose to Airflow 3.x image

## Files to Reference

- `dags/multi_sport_betting_workflow.py` - Already uses 3.x imports
- [Airflow 3.0 Migration Guide](https://airflow.apache.org/docs/apache-airflow/stable/migrations-ref.html)


## Airflow 3.x CLI

 - Has changed significantly since Airflow 2.
 - Please look at latest docs before running CLI commands
