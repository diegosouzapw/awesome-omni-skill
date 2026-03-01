---
name: apache-airflow-orchestration
description: Complete guide for Apache Airflow orchestration including DAGs, operators, sensors, XComs, task dependencies, dynamic workflows, and production deployment
tags: [airflow, workflow, orchestration, dags, operators, sensors, data-pipelines, scheduling]
tier: tier-1
---

# Apache Airflow Orchestration

A comprehensive skill for mastering Apache Airflow workflow orchestration. This skill covers DAG development, operators, sensors, task dependencies, dynamic workflows, XCom communication, scheduling patterns, and production deployment strategies.

## When to Use This Skill

Use this skill when:

- Building and managing complex data pipelines with task dependencies
- Orchestrating ETL/ELT workflows across multiple systems
- Scheduling and monitoring batch processing jobs
- Coordinating multi-step data transformations
- Managing workflows with conditional execution and branching
- Implementing event-driven or asset-based workflows
- Deploying production-grade workflow automation
- Creating dynamic workflows that generate tasks programmatically
- Coordinating distributed task execution across clusters
- Building data engineering platforms with workflow orchestration

## Core Concepts

### What is Apache Airflow?

Apache Airflow is an open-source platform for programmatically authoring, scheduling, and monitoring workflows. It allows you to define workflows as Directed Acyclic Graphs (DAGs) using Python code, making complex workflow orchestration maintainable and version-controlled.

**Key Principles:**
- **Dynamic**: Workflows are defined in Python, enabling dynamic generation
- **Extensible**: Rich ecosystem of operators, sensors, and hooks
- **Scalable**: Can scale from single machine to large clusters
- **Observable**: Comprehensive UI for monitoring and troubleshooting

### DAGs (Directed Acyclic Graphs)

A DAG is a collection of tasks organized to reflect their relationships and dependencies.

**DAG Properties:**
- **dag_id**: Unique identifier for the DAG
- **start_date**: When the DAG should start being scheduled
- **schedule**: How often to run (cron, timedelta, or asset-based)
- **catchup**: Whether to run missed intervals on DAG activation
- **tags**: Labels for organization and filtering
- **default_args**: Default parameters for all tasks in the DAG

**DAG Definition Example:**
```python
from datetime import datetime
from airflow.sdk import DAG

with DAG(
    dag_id="example_dag",
    start_date=datetime(2022, 1, 1),
    schedule="0 0 * * *",  # Daily at midnight
    catchup=False,
    tags=["example", "tutorial"],
) as dag:
    # Tasks defined here
    pass
```

### Tasks and Operators

**Tasks** are the basic units of execution in Airflow. **Operators** are templates for creating tasks.

**Common Operator Types:**

1. **BashOperator**: Execute bash commands
2. **PythonOperator**: Execute Python functions
3. **EmailOperator**: Send emails
4. **EmptyOperator**: Placeholder/dummy tasks
5. **Custom Operators**: User-defined operators for specific needs

**Operator vs. Task:**
- Operator: Template/class definition
- Task: Instantiation of an operator with specific parameters

### Task Dependencies

Task dependencies define the execution order and workflow structure.

**Dependency Operators:**
- `>>`: Sets downstream dependency (task1 >> task2)
- `<<`: Sets upstream dependency (task2 << task1)
- `chain()`: Sequential dependencies for multiple tasks
- `cross_downstream()`: Many-to-many relationships

**Dependency Examples:**
```python
# Simple linear flow
task1 >> task2 >> task3

# Fan-out pattern
task1 >> [task2, task3, task4]

# Fan-in pattern
[task1, task2, task3] >> task4

# Complex dependencies
first_task >> [second_task, third_task]
third_task << fourth_task
```

### Executors

Executors determine how and where tasks run.

**Executor Types:**
- **SequentialExecutor**: Single-threaded, local (default, not for production)
- **LocalExecutor**: Multi-threaded, single machine
- **CeleryExecutor**: Distributed execution using Celery
- **KubernetesExecutor**: Each task runs in a separate Kubernetes pod
- **DaskExecutor**: Distributed execution using Dask

### Scheduler

The Airflow scheduler:
- Monitors all DAGs and their tasks
- Triggers task instances based on dependencies and schedules
- Submits tasks to executors for execution
- Handles retries and task state management

**Starting the Scheduler:**
```bash
airflow scheduler
```

## DAG Development Patterns

### Basic DAG Structure

Every DAG follows this structure:

```python
from datetime import datetime
from airflow.sdk import DAG
from airflow.providers.standard.operators.bash import BashOperator

with DAG(
    dag_id="basic_dag",
    start_date=datetime(2022, 1, 1),
    schedule="0 0 * * *",
    catchup=False,
) as dag:

    task1 = BashOperator(
        task_id="task1",
        bash_command="echo 'Task 1 executed'"
    )

    task2 = BashOperator(
        task_id="task2",
        bash_command="echo 'Task 2 executed'"
    )

    task1 >> task2
```

### Task Dependencies and Chains

**Linear Chain:**
```python
from airflow.sdk import chain

# These are equivalent:
task1 >> task2 >> task3 >> task4
chain(task1, task2, task3, task4)
```

**Dynamic Chain:**
```python
from airflow.sdk import chain
from airflow.operators.empty import EmptyOperator

# Dynamically generate and chain tasks
chain(*[EmptyOperator(task_id=f"task_{i}") for i in range(1, 6)])
```

**Pairwise Chain:**
```python
from airflow.sdk import chain

# Creates paired dependencies:
# op1 >> op2 >> op4 >> op6
# op1 >> op3 >> op5 >> op6
chain(op1, [op2, op3], [op4, op5], op6)
```

**Cross Downstream:**
```python
from airflow.sdk import cross_downstream

# Both op1 and op2 feed into both op3 and op4
cross_downstream([op1, op2], [op3, op4])
```

### Branching and Conditional Execution

**BranchPythonOperator:**
```python
from airflow.operators.python import BranchPythonOperator

def choose_branch(**context):
    if context['data_interval_start'].day == 1:
        return 'monthly_task'
    return 'daily_task'

branch = BranchPythonOperator(
    task_id='branch_task',
    python_callable=choose_branch
)

daily_task = BashOperator(task_id='daily_task', bash_command='echo daily')
monthly_task = BashOperator(task_id='monthly_task', bash_command='echo monthly')

branch >> [daily_task, monthly_task]
```

**Custom Branch Operator:**
```python
from airflow.operators.branch import BaseBranchOperator

class MyBranchOperator(BaseBranchOperator):
    def choose_branch(self, context):
        """
        Run extra branch on first day of month
        """
        if context['data_interval_start'].day == 1:
            return ['daily_task_id', 'monthly_task_id']
        elif context['data_interval_start'].day == 2:
            return 'daily_task_id'
        else:
            return None  # Skip all downstream tasks
```

### TaskGroups for Organization

TaskGroups help organize related tasks hierarchically:

```python
from airflow.sdk import task_group
from airflow.operators.empty import EmptyOperator

@task_group()
def data_processing_group():
    extract = EmptyOperator(task_id="extract")
    transform = EmptyOperator(task_id="transform")
    load = EmptyOperator(task_id="load")

    extract >> transform >> load

@task_group()
def validation_group():
    validate_schema = EmptyOperator(task_id="validate_schema")
    validate_data = EmptyOperator(task_id="validate_data")

    validate_schema >> validate_data

start = EmptyOperator(task_id="start")
end = EmptyOperator(task_id="end")

start >> data_processing_group() >> validation_group() >> end
```

### Edge Labeling

Add labels to dependency edges for clarity:

```python
from airflow.sdk import Label

# Inline labeling
my_task >> Label("When empty") >> other_task

# Method-based labeling
my_task.set_downstream(other_task, Label("When empty"))
```

### LatestOnlyOperator

Skip tasks if not the latest DAG run:

```python
from airflow.operators.latest_only import LatestOnlyOperator
from airflow.operators.empty import EmptyOperator
import pendulum

with DAG(
    dag_id='latest_only_example',
    start_date=pendulum.datetime(2023, 1, 1, tz="UTC"),
    catchup=True,
    schedule="@daily",
) as dag:
    latest_only = LatestOnlyOperator(task_id='latest_only')
    task1 = EmptyOperator(task_id='task1')
    task2 = EmptyOperator(task_id='task2')
    task3 = EmptyOperator(task_id='task3')
    task4 = EmptyOperator(task_id='task4', trigger_rule='all_done')

    latest_only >> task1 >> task3
    latest_only >> task4
    task2 >> task3
    task2 >> task4
```

## Operators Deep Dive

### BashOperator

Execute bash commands:

```python
from airflow.providers.standard.operators.bash import BashOperator

bash_task = BashOperator(
    task_id="bash_example",
    bash_command="echo 'Hello from Bash'; date",
    env={'MY_VAR': 'value'},  # Environment variables
    append_env=True,  # Append to existing env vars
    output_encoding='utf-8'
)
```

**Complex Bash Command:**
```python
bash_complex = BashOperator(
    task_id="complex_bash",
    bash_command="""
        cd /path/to/dir
        python process_data.py --input {{ ds }} --output {{ tomorrow_ds }}
        if [ $? -eq 0 ]; then
            echo "Success"
        else
            echo "Failed" && exit 1
        fi
    """,
)
```

### PythonOperator

Execute Python functions:

```python
from airflow.providers.standard.operators.python import PythonOperator

def my_python_function(name, **context):
    print(f"Hello {name}!")
    print(f"Execution date: {context['ds']}")
    return "Success"

python_task = PythonOperator(
    task_id="python_example",
    python_callable=my_python_function,
    op_kwargs={'name': 'Airflow'},
    provide_context=True
)
```

**Traditional ETL with PythonOperator:**
```python
import json
import pendulum
from airflow.sdk import DAG
from airflow.providers.standard.operators.python import PythonOperator

def extract():
    data_string = '{"1001": 301.27, "1002": 433.21, "1003": 502.22}'
    return json.loads(data_string)

def transform(ti):
    # Pull from XCom
    order_data_dict = ti.xcom_pull(task_ids="extract")
    total_order_value = sum(order_data_dict.values())
    return {"total_order_value": total_order_value}

def load(ti):
    # Pull from XCom
    total = ti.xcom_pull(task_ids="transform")["total_order_value"]
    print(f"Total order value is: {total:.2f}")

with DAG(
    dag_id="legacy_etl_pipeline",
    schedule=None,
    start_date=pendulum.datetime(2021, 1, 1, tz="UTC"),
    catchup=False,
) as dag:
    extract_task = PythonOperator(task_id="extract", python_callable=extract)
    transform_task = PythonOperator(task_id="transform", python_callable=transform)
    load_task = PythonOperator(task_id="load", python_callable=load)

    extract_task >> transform_task >> load_task
```

### EmailOperator

Send email notifications:

```python
from airflow.providers.smtp.operators.smtp import EmailOperator

email_task = EmailOperator(
    task_id='send_email',
    to='recipient@example.com',
    subject='Airflow Notification',
    html_content='<h3>Task completed successfully!</h3>',
    cc=['cc@example.com'],
    bcc=['bcc@example.com']
)
```

### EmptyOperator

Placeholder for workflow structure:

```python
from airflow.operators.empty import EmptyOperator

start = EmptyOperator(task_id='start')
end = EmptyOperator(task_id='end')

# Useful for organizing complex DAGs
start >> [task1, task2, task3] >> end
```

### Custom Operators

Create reusable custom operators:

```python
from airflow.models import BaseOperator
from airflow.utils.decorators import apply_defaults

class MyCustomOperator(BaseOperator):
    @apply_defaults
    def __init__(self, my_param, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.my_param = my_param

    def execute(self, context):
        self.log.info(f"Executing with param: {self.my_param}")
        # Custom logic here
        return "Result"

# Usage
custom_task = MyCustomOperator(
    task_id="custom",
    my_param="value"
)
```

## Sensors Deep Dive

Sensors are a special type of operator that wait for a certain condition to be met before proceeding.

### ExternalTaskSensor

Wait for tasks in other DAGs:

```python
from airflow.providers.standard.sensors.external_task import ExternalTaskSensor
import pendulum

with DAG(
    dag_id="example_external_task_sensor",
    start_date=pendulum.datetime(2021, 10, 20, tz="UTC"),
    catchup=False,
    schedule=None,
) as dag:

    wait_for_task = ExternalTaskSensor(
        task_id="wait_for_task",
        external_dag_id="upstream_dag",
        external_task_id="upstream_task",
        allowed_states=["success"],
        failed_states=["failed"],
        execution_delta=None,  # Same execution_date
        timeout=600,  # 10 minutes
        poke_interval=60,  # Check every 60 seconds
    )
```

**Deferrable ExternalTaskSensor:**
```python
# More efficient - releases worker slot while waiting
wait_for_task_async = ExternalTaskSensor(
    task_id="wait_for_task_async",
    external_dag_id="upstream_dag",
    external_task_id="upstream_task",
    allowed_states=["success"],
    failed_states=["failed"],
    deferrable=True,  # Use async mode
)
```

### FileSensor

Wait for files to appear:

```python
from airflow.sensors.filesystem import FileSensor

wait_for_file = FileSensor(
    task_id="wait_for_file",
    filepath="/path/to/file.csv",
    poke_interval=30,
    timeout=600,
    mode='poke'  # or 'reschedule' for long waits
)
```

### TimeDeltaSensor

Wait for a specific time period:

```python
from datetime import timedelta
from airflow.sensors.time_delta import TimeDeltaSensor

wait_one_hour = TimeDeltaSensor(
    task_id="wait_one_hour",
    delta=timedelta(hours=1)
)
```

### BigQuery Table Sensor

Wait for BigQuery table to exist:

```python
from airflow.providers.google.cloud.sensors.bigquery import BigQueryTableExistenceSensor
import pendulum

with DAG(
    dag_id="bigquery_sensor_example",
    start_date=pendulum.datetime(2023, 10, 26, tz="UTC"),
) as dag:

    wait_for_table = BigQueryTableExistenceSensor(
        task_id="wait_for_table",
        project_id="your-project-id",
        dataset_id="your_dataset",
        table_id="your_table",
        bigquery_conn_id="google_cloud_default",
        location="US",
        poke_interval=60,
        timeout=3600,
    )
```

### Custom Sensors

Create custom sensors for specific conditions:

```python
from airflow.sensors.base import BaseSensorOperator
from airflow.utils.decorators import apply_defaults

class MyCustomSensor(BaseSensorOperator):
    @apply_defaults
    def __init__(self, my_condition, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.my_condition = my_condition

    def poke(self, context):
        # Return True when condition is met
        self.log.info(f"Checking condition: {self.my_condition}")
        # Custom logic to check condition
        return check_condition(self.my_condition)
```

### Deferrable Sensors

Deferrable sensors release worker slots while waiting:

```python
from datetime import timedelta
from airflow.sdk import BaseSensorOperator, StartTriggerArgs

class WaitHoursSensor(BaseSensorOperator):
    start_trigger_args = StartTriggerArgs(
        trigger_cls="airflow.providers.standard.triggers.temporal.TimeDeltaTrigger",
        trigger_kwargs={"moment": timedelta(hours=1)},
        next_method="execute_complete",
        next_kwargs=None,
        timeout=None,
    )
    start_from_trigger = True

    def __init__(self, *args, trigger_kwargs=None, start_from_trigger=True, **kwargs):
        super().__init__(*args, **kwargs)
        if trigger_kwargs:
            self.start_trigger_args.trigger_kwargs = trigger_kwargs
        self.start_from_trigger = start_from_trigger

    def execute_complete(self, context, event=None):
        return  # Task complete
```

## XComs (Cross-Communication)

XComs enable task-to-task communication by storing and retrieving data.

### Basic XCom Usage

**Pushing to XCom:**
```python
def push_function(**context):
    value = "Important data"
    context['ti'].xcom_push(key='my_key', value=value)
    # Or simply return (uses 'return_value' key)
    return value

push_task = PythonOperator(
    task_id='push',
    python_callable=push_function,
    provide_context=True
)
```

**Pulling from XCom:**
```python
def pull_function(**context):
    # Pull by task_id (uses 'return_value' key)
    value = context['ti'].xcom_pull(task_ids='push')

    # Pull with specific key
    value = context['ti'].xcom_pull(task_ids='push', key='my_key')

    print(f"Pulled value: {value}")

pull_task = PythonOperator(
    task_id='pull',
    python_callable=pull_function,
    provide_context=True
)
```

### XCom with TaskFlow API

TaskFlow API automatically manages XComs:

```python
from airflow.decorators import task

@task
def extract():
    return {"data": [1, 2, 3, 4, 5]}

@task
def transform(data_dict):
    # Automatically receives XCom from extract
    total = sum(data_dict['data'])
    return {"total": total}

@task
def load(summary):
    print(f"Total: {summary['total']}")

# Automatic XCom handling
data = extract()
summary = transform(data)
load(summary)
```

### XCom Best Practices

**Size Limitations:**
- XComs are stored in the metadata database
- Keep XCom data small (< 1MB recommended)
- For large data, store in external systems and pass references

**Example with External Storage:**
```python
@task
def process_large_data():
    # Process data
    large_result = compute_large_dataset()

    # Store in S3/GCS
    file_path = save_to_s3(large_result, "s3://bucket/result.parquet")

    # Return only the path
    return {"result_path": file_path}

@task
def consume_large_data(metadata):
    # Load from S3/GCS
    data = load_from_s3(metadata['result_path'])
    process(data)
```

### XCom with Operators

**Reading XCom in Templates:**
```python
from airflow.providers.standard.operators.bash import BashOperator

process_file = BashOperator(
    task_id="process",
    bash_command="python process.py {{ ti.xcom_pull(task_ids='extract') }}",
)
```

**XCom with EmailOperator:**
```python
from airflow.sdk import task
from airflow.providers.smtp.operators.smtp import EmailOperator

@task
def get_ip():
    return "192.168.1.1"

@task(multiple_outputs=True)
def compose_email(external_ip):
    return {
        'subject': f'Server connected from {external_ip}',
        'body': f'Your server is connected from {external_ip}<br>'
    }

email_info = compose_email(get_ip())

EmailOperator(
    task_id='send_email',
    to='example@example.com',
    subject=email_info['subject'],
    html_content=email_info['body']
)
```

## Dynamic Workflows

Create tasks dynamically based on runtime conditions or external data.

### Dynamic Task Generation with Loops

```python
from airflow.sdk import DAG
from airflow.operators.empty import EmptyOperator

with DAG("dynamic_loop_example", ...) as dag:
    start = EmptyOperator(task_id="start")
    end = EmptyOperator(task_id="end")

    # Dynamically create tasks
    options = ["branch_a", "branch_b", "branch_c", "branch_d"]
    for option in options:
        task = EmptyOperator(task_id=option)
        start >> task >> end
```

### Dynamic Task Mapping

Map over task outputs to create dynamic parallel tasks:

```python
from airflow.decorators import task

@task
def extract():
    # Returns list of items to process
    return [1, 2, 3, 4, 5]

@task
def transform(item):
    # Processes single item
    return item * 2

@task
def load(items):
    # Receives all transformed items
    print(f"Loaded {len(items)} items: {items}")

# Dynamic mapping
data = extract()
transformed = transform.expand(item=data)  # Creates 5 parallel tasks
load(transformed)
```

**Mapping with Classic Operators:**
```python
from airflow.operators.bash import BashOperator

class ExtractOperator(BaseOperator):
    def execute(self, context):
        return ["file1.csv", "file2.csv", "file3.csv"]

class TransformOperator(BaseOperator):
    def __init__(self, input, **kwargs):
        super().__init__(**kwargs)
        self.input = input

    def execute(self, context):
        # Process single file
        return f"processed_{self.input}"

extract = ExtractOperator(task_id="extract")
transform = TransformOperator.partial(task_id="transform").expand(input=extract.output)
```

### Task Group Mapping

Map over entire task groups:

```python
from airflow.decorators import task, task_group

@task
def add_one(value):
    return value + 1

@task
def double(value):
    return value * 2

@task_group
def process_group(value):
    incremented = add_one(value)
    return double(incremented)

@task
def aggregate(results):
    print(f"Results: {results}")

# Map task group over values
results = process_group.expand(value=[1, 2, 3, 4, 5])
aggregate(results)
```

### Partial Parameters with Mapping

Mix static and dynamic parameters:

```python
@task
def process(base_path, filename):
    full_path = f"{base_path}/{filename}"
    return f"Processed {full_path}"

# Static parameter 'base_path', dynamic 'filename'
results = process.partial(base_path="/data").expand(
    filename=["file1.csv", "file2.csv", "file3.csv"]
)
```

## TaskFlow API

The modern way to write Airflow DAGs with automatic XCom handling and cleaner syntax.

### Basic TaskFlow Example

```python
from airflow.decorators import dag, task
import pendulum

@dag(
    dag_id="taskflow_example",
    start_date=pendulum.datetime(2023, 10, 26, tz="UTC"),
    schedule=None,
    catchup=False,
)
def my_taskflow_dag():

    @task
    def extract():
        data_string = '{"1001": 301.27, "1002": 433.21, "1003": 502.22}'
        import json
        return json.loads(data_string)

    @task
    def transform(order_data_dict):
        total = sum(order_data_dict.values())
        return {"total_order_value": total}

    @task
    def load(summary):
        print(f"Total order value: {summary['total_order_value']:.2f}")

    # Function calls create task dependencies automatically
    order_data = extract()
    summary = transform(order_data)
    load(summary)

# Instantiate the DAG
my_taskflow_dag()
```

### Multiple Outputs

Return multiple values from tasks:

```python
@task(multiple_outputs=True)
def extract_data():
    return {
        'orders': [1, 2, 3],
        'customers': ['A', 'B', 'C'],
        'revenue': 1000.50
    }

@task
def process_orders(orders):
    print(f"Processing {len(orders)} orders")

@task
def process_customers(customers):
    print(f"Processing {len(customers)} customers")

# Access individual outputs
data = extract_data()
process_orders(data['orders'])
process_customers(data['customers'])
```

### Mixing TaskFlow with Traditional Operators

```python
from airflow.decorators import dag, task
from airflow.providers.standard.operators.bash import BashOperator
import pendulum

@dag(
    start_date=pendulum.datetime(2023, 1, 1, tz="UTC"),
    schedule=None,
)
def mixed_dag():

    @task
    def get_date():
        from datetime import datetime
        return datetime.now().strftime("%Y%m%d")

    # Traditional operator
    bash_task = BashOperator(
        task_id="print_date",
        bash_command="echo Processing data for {{ ti.xcom_pull(task_ids='get_date') }}"
    )

    @task
    def process_results():
        print("Processing complete")

    # Mix task types
    date = get_date()
    date >> bash_task >> process_results()

mixed_dag()
```

### Virtual Environment for Tasks

Isolate task dependencies:

```python
from airflow.decorators import dag, task
import pendulum

@dag(
    dag_id="virtualenv_example",
    start_date=pendulum.datetime(2023, 10, 26, tz="UTC"),
)
def virtualenv_dag():

    @task.virtualenv(
        requirements=["pandas==1.5.0", "numpy==1.23.0"],
        system_site_packages=False
    )
    def analyze_data():
        import pandas as pd
        import numpy as np

        df = pd.DataFrame({'col1': [1, 2, 3], 'col2': [4, 5, 6]})
        result = np.mean(df['col1'])
        return float(result)

    @task
    def report_results(mean_value):
        print(f"Mean value: {mean_value}")

    result = analyze_data()
    report_results(result)

virtualenv_dag()
```

## Asset-Based Scheduling

Schedule DAGs based on data assets (formerly datasets) rather than time.

### Producer-Consumer Pattern

```python
from airflow.sdk import DAG, Asset
from airflow.operators.bash import BashOperator
from datetime import datetime

# Define asset
customer_data = Asset("s3://my-bucket/customers.parquet")

# Producer DAG
with DAG(
    dag_id="producer_dag",
    start_date=datetime(2023, 1, 1),
    schedule="@daily",
) as producer:

    BashOperator(
        task_id="generate_data",
        bash_command="python generate_customers.py",
        outlets=[customer_data]  # Marks asset as updated
    )

# Consumer DAG - triggered when asset updates
with DAG(
    dag_id="consumer_dag",
    schedule=[customer_data],  # Triggered by asset
    start_date=datetime(2023, 1, 1),
    catchup=False,
) as consumer:

    BashOperator(
        task_id="process_data",
        bash_command="python process_customers.py"
    )
```

### Multiple Asset Dependencies

**AND Logic (all assets must update):**
```python
from airflow.datasets import Dataset

asset_1 = Dataset("s3://bucket/file1.csv")
asset_2 = Dataset("s3://bucket/file2.csv")

with DAG(
    dag_id="wait_for_both",
    schedule=[asset_1 & asset_2],  # Both must update
    start_date=datetime(2023, 1, 1),
):
    pass
```

**OR Logic (any asset update triggers):**
```python
asset_1 = Dataset("s3://bucket/file1.csv")
asset_2 = Dataset("s3://bucket/file2.csv")

with DAG(
    dag_id="triggered_by_either",
    schedule=[asset_1 | asset_2],  # Either can trigger
    start_date=datetime(2023, 1, 1),
):
    pass
```

**Complex Logic:**
```python
asset_1 = Dataset("s3://bucket/file1.csv")
asset_2 = Dataset("s3://bucket/file2.csv")
asset_3 = Dataset("s3://bucket/file3.csv")

with DAG(
    dag_id="complex_condition",
    schedule=(asset_1 | (asset_2 & asset_3)),  # asset_1 OR (asset_2 AND asset_3)
    start_date=datetime(2023, 1, 1),
):
    pass
```

### Asset Aliases

Use aliases for flexible asset references:

```python
from airflow.datasets import Dataset, AssetAlias
from airflow.decorators import task

# Producer with alias
with DAG(dag_id="alias_producer", start_date=datetime(2023, 1, 1)):
    @task(outlets=[AssetAlias("my-alias")])
    def produce_data(*, outlet_events):
        # Dynamically add actual asset
        outlet_events[AssetAlias("my-alias")].add(
            Dataset("s3://bucket/my-file.csv")
        )

# Consumer depending on alias
with DAG(
    dag_id="alias_consumer",
    schedule=AssetAlias("my-alias"),
    start_date=datetime(2023, 1, 1),
):
    pass
```

### Accessing Asset Event Information

```python
@task
def process_asset_data(*, triggering_asset_events):
    for event in triggering_asset_events:
        print(f"Asset: {event.asset.uri}")
        print(f"Timestamp: {event.timestamp}")
        print(f"Extra: {event.extra}")
```

## Scheduling Patterns

### Cron Expressions

```python
# Every day at midnight
schedule="0 0 * * *"

# Every Monday at 9 AM
schedule="0 9 * * 1"

# Every 15 minutes
schedule="*/15 * * * *"

# First day of month at noon
schedule="0 12 1 * *"

# Weekdays at 6 PM
schedule="0 18 * * 1-5"
```

### Timedelta Scheduling

```python
from datetime import timedelta

with DAG(
    dag_id="timedelta_schedule",
    start_date=datetime(2023, 1, 1),
    schedule=timedelta(hours=6),  # Every 6 hours
):
    pass
```

### Preset Schedules

```python
# Common presets
schedule="@once"      # Run once
schedule="@hourly"    # Every hour
schedule="@daily"     # Daily at midnight
schedule="@weekly"    # Every Sunday at midnight
schedule="@monthly"   # First day of month at midnight
schedule="@yearly"    # January 1st at midnight
schedule=None         # Manual trigger only
```

### Catchup and Backfilling

**Catchup:**
```python
with DAG(
    dag_id="catchup_example",
    start_date=datetime(2023, 1, 1),
    schedule="@daily",
    catchup=True,  # Run all missed intervals
):
    pass

with DAG(
    dag_id="no_catchup",
    start_date=datetime(2023, 1, 1),
    schedule="@daily",
    catchup=False,  # Only run latest interval
):
    pass
```

**Manual Backfilling:**
```bash
# Backfill specific date range
airflow dags backfill \
    --start-date 2023-01-01 \
    --end-date 2023-01-31 \
    my_dag_id

# Backfill with marking success (no execution)
airflow dags backfill \
    --start-date 2023-01-01 \
    --end-date 2023-01-31 \
    --mark-success \
    my_dag_id
```

## Production Patterns

### Error Handling and Retries

**Task-Level Retries:**
```python
from airflow.operators.bash import BashOperator
from datetime import timedelta

task_with_retry = BashOperator(
    task_id="retry_task",
    bash_command="python might_fail.py",
    retries=3,
    retry_delay=timedelta(minutes=5),
    retry_exponential_backoff=True,
    max_retry_delay=timedelta(minutes=30),
)
```

**DAG-Level Default Args:**
```python
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-team',
    'depends_on_past': False,
    'email': ['alerts@company.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
}

with DAG(
    dag_id="production_dag",
    default_args=default_args,
    start_date=datetime(2023, 1, 1),
    schedule="@daily",
):
    pass
```

### Task Concurrency Control

**Per-Task Concurrency:**
```python
from airflow.operators.bash import BashOperator
from datetime import timedelta

# Limit concurrent instances of this task
limited_task = BashOperator(
    task_id="limited_task",
    bash_command="echo 'Processing'",
    max_active_tis_per_dag=3  # Max 3 instances running
)
```

**DAG-Level Concurrency:**
```python
with DAG(
    dag_id="concurrent_dag",
    start_date=datetime(2023, 1, 1),
    schedule="@daily",
    max_active_runs=5,  # Max 5 DAG runs simultaneously
    concurrency=10,     # Max 10 task instances across all runs
):
    pass
```

### Idempotency

Make tasks idempotent for safe retries:

```python
@task
def idempotent_load(**context):
    execution_date = context['ds']

    # Delete existing data for this date first
    delete_query = f"""
        DELETE FROM target_table
        WHERE date = '{execution_date}'
    """
    execute_sql(delete_query)

    # Insert new data
    insert_query = f"""
        INSERT INTO target_table
        SELECT * FROM source
        WHERE date = '{execution_date}'
    """
    execute_sql(insert_query)
```

### SLAs and Alerts

```python
from datetime import timedelta

def sla_miss_callback(dag, task_list, blocking_task_list, slas, blocking_tis):
    print(f"SLA missed for {task_list}")
    # Send alert to monitoring system

with DAG(
    dag_id="sla_dag",
    start_date=datetime(2023, 1, 1),
    schedule="@daily",
    default_args={
        'sla': timedelta(hours=2),  # Task should complete in 2 hours
    },
    sla_miss_callback=sla_miss_callback,
):
    pass
```

### Task Callbacks

```python
def on_failure_callback(context):
    print(f"Task {context['task_instance'].task_id} failed")
    # Send to Slack, PagerDuty, etc.

def on_success_callback(context):
    print(f"Task {context['task_instance'].task_id} succeeded")

def on_retry_callback(context):
    print(f"Task {context['task_instance'].task_id} retrying")

task_with_callbacks = BashOperator(
    task_id="monitored_task",
    bash_command="python my_script.py",
    on_failure_callback=on_failure_callback,
    on_success_callback=on_success_callback,
    on_retry_callback=on_retry_callback,
)
```

### Docker Deployment

**Docker Compose for Local Development:**
```yaml
version: '3'
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow

  webserver:
    image: apache/airflow:2.7.0
    depends_on:
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    ports:
      - "8080:8080"
    command: webserver

  scheduler:
    image: apache/airflow:2.7.0
    depends_on:
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    command: scheduler
```

### Kubernetes Executor

**KubernetesExecutor Configuration:**
```python
# In airflow.cfg
[kubernetes]
namespace = airflow
worker_container_repository = my-registry/airflow
worker_container_tag = 2.7.0
delete_worker_pods = True
delete_worker_pods_on_failure = False

[core]
executor = KubernetesExecutor
```

**Pod Override for Specific Task:**
```python
from kubernetes.client import models as k8s

task_with_gpu = BashOperator(
    task_id="gpu_task",
    bash_command="python train_model.py",
    executor_config={
        "pod_override": k8s.V1Pod(
            spec=k8s.V1PodSpec(
                containers=[
                    k8s.V1Container(
                        name="base",
                        resources=k8s.V1ResourceRequirements(
                            limits={"nvidia.com/gpu": "1"}
                        )
                    )
                ]
            )
        )
    }
)
```

### Monitoring and Logging

**Structured Logging:**
```python
from airflow.decorators import task
import logging

@task
def monitored_task():
    logger = logging.getLogger(__name__)

    logger.info("Starting data processing", extra={
        'process_id': 'abc123',
        'record_count': 1000
    })

    try:
        process_data()
        logger.info("Processing complete")
    except Exception as e:
        logger.error(f"Processing failed: {str(e)}", extra={
            'error_type': type(e).__name__
        })
        raise
```

**StatsD Metrics:**
```python
from airflow.stats import Stats

@task
def task_with_metrics():
    Stats.incr('my_dag.task_started')

    start_time = time.time()
    process_data()
    duration = time.time() - start_time

    Stats.timing('my_dag.task_duration', duration)
    Stats.incr('my_dag.task_completed')
```

## Best Practices

### DAG Design

1. **Keep DAGs Simple**: Break complex workflows into multiple DAGs
2. **Use Descriptive Names**: dag_id and task_id should be self-explanatory
3. **Idempotent Tasks**: Tasks should produce same result when re-run
4. **Small XComs**: Keep XCom data under 1MB
5. **External Storage**: Use S3/GCS for large data, pass references
6. **Proper Dependencies**: Model true dependencies, avoid unnecessary ones
7. **Error Handling**: Use retries, callbacks, and proper error logging
8. **Resource Management**: Set appropriate task concurrency limits

### Code Organization

```
dags/
├── common/
│   ├── __init__.py
│   ├── operators.py      # Custom operators
│   ├── sensors.py        # Custom sensors
│   └── utils.py          # Utility functions
├── etl/
│   ├── customer_pipeline.py
│   ├── order_pipeline.py
│   └── product_pipeline.py
├── ml/
│   ├── training_dag.py
│   └── inference_dag.py
└── maintenance/
    ├── cleanup_dag.py
    └── backup_dag.py
```

### Testing DAGs

**Unit Testing:**
```python
import pytest
from airflow.models import DagBag

def test_dag_loaded():
    dagbag = DagBag(dag_folder='dags/', include_examples=False)
    assert len(dagbag.import_errors) == 0

def test_task_count():
    dagbag = DagBag(dag_folder='dags/')
    dag = dagbag.get_dag('my_dag')
    assert len(dag.tasks) == 5

def test_task_dependencies():
    dagbag = DagBag(dag_folder='dags/')
    dag = dagbag.get_dag('my_dag')

    extract = dag.get_task('extract')
    transform = dag.get_task('transform')

    assert transform in extract.downstream_list
```

**Integration Testing:**
```python
from airflow.models import DagBag
from airflow.utils.state import State

def test_dag_runs():
    dagbag = DagBag(dag_folder='dags/')
    dag = dagbag.get_dag('my_dag')

    # Test DAG run
    dag_run = dag.create_dagrun(
        state=State.RUNNING,
        execution_date=datetime(2023, 1, 1),
        run_type='manual'
    )

    # Run specific task
    task_instance = dag_run.get_task_instance('extract')
    task_instance.run()

    assert task_instance.state == State.SUCCESS
```

### Performance Optimization

1. **Use Deferrable Operators**: For sensors and long-running waits
2. **Dynamic Task Mapping**: For parallel processing
3. **Appropriate Executor**: Choose based on scale (Local, Celery, Kubernetes)
4. **Connection Pooling**: Reuse database connections
5. **Task Parallelism**: Set max_active_runs and concurrency appropriately
6. **Lazy Loading**: Don't execute heavy logic at DAG parse time
7. **External Storage**: Keep metadata database light

### Security

1. **Secrets Management**: Use Airflow Secrets Backend (not hardcoded)
2. **Connection Encryption**: Use encrypted connections for databases
3. **RBAC**: Enable role-based access control
4. **Audit Logging**: Enable audit logs for compliance
5. **Network Isolation**: Restrict worker network access
6. **Credential Rotation**: Regularly rotate credentials

### Configuration Management

```python
# Use Variables for configuration
from airflow.models import Variable

config = Variable.get("my_config", deserialize_json=True)
api_key = Variable.get("api_key")

# Use Connections for external services
from airflow.hooks.base import BaseHook

conn = BaseHook.get_connection('my_postgres')
db_url = f"postgresql://{conn.login}:{conn.password}@{conn.host}:{conn.port}/{conn.schema}"
```

## Common Patterns and Examples

See EXAMPLES.md for 18+ detailed real-world examples including:
- ETL pipelines
- Machine learning workflows
- Data quality checks
- Multi-cloud orchestration
- Event-driven architectures
- Complex branching logic
- Dynamic task generation
- Asset-based scheduling
- Sensor patterns
- Error handling strategies

## Troubleshooting

### DAG Not Appearing in UI

1. Check for Python syntax errors in DAG file
2. Verify DAG file is in correct directory
3. Check dag_id is unique
4. Ensure schedule is not None if you expect it to run
5. Check scheduler logs for import errors

### Tasks Not Running

1. Check task dependencies are correct
2. Verify upstream tasks succeeded
3. Check task concurrency limits
4. Ensure executor has available slots
5. Review task logs for errors

### Performance Issues

1. Reduce DAG complexity (break into multiple DAGs)
2. Optimize SQL queries in tasks
3. Use appropriate executor for scale
4. Enable task parallelism
5. Check for slow sensors (use deferrable mode)
6. Monitor metadata database performance

### Common Errors

**Import Errors:**
```python
# Bad - imports at DAG level slow parsing
from heavy_library import process

with DAG(...):
    pass

# Good - imports inside tasks
with DAG(...):
    @task
    def my_task():
        from heavy_library import process
        process()
```

**Circular Dependencies:**
```python
# This will fail
task1 >> task2 >> task3 >> task1  # Circular!

# Must be acyclic
task1 >> task2 >> task3
```

**Large XComs:**
```python
# Bad - storing large data in XCom
@task
def process():
    large_df = pd.read_csv('big_file.csv')
    return large_df  # Too large!

# Good - store reference
@task
def process():
    large_df = pd.read_csv('big_file.csv')
    path = save_to_s3(large_df)
    return path  # Just the path
```

## Resources

- **Official Documentation**: https://airflow.apache.org/docs/
- **Airflow GitHub**: https://github.com/apache/airflow
- **Astronomer Guides**: https://docs.astronomer.io/learn
- **Community Slack**: https://apache-airflow.slack.com
- **Stack Overflow**: Tag `apache-airflow`
- **Awesome Airflow**: https://github.com/jghoman/awesome-apache-airflow

---

**Skill Version**: 1.0.0
**Last Updated**: January 2025
**Apache Airflow Version**: 2.7+
**Skill Category**: Data Engineering, Workflow Orchestration, Pipeline Management
