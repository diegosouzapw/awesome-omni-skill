---
name: dapr-skill
description: Comprehensive Dapr (Distributed Application Runtime) skill for building distributed microservices from hello world to professional production systems. Use when developing event-driven applications, implementing service-to-service communication, managing distributed state, building actor-based systems, or deploying microservices with resilience and observability.
---

# Dapr (Distributed Application Runtime) Skill

This skill provides comprehensive support for building distributed applications with Dapr, from simple hello world examples to professional production-ready systems.

## When to Use This Skill

Use this skill when you need to:
- Build distributed microservices with Dapr building blocks
- Implement service-to-service communication using service invocation
- Manage distributed state with various state store providers
- Create event-driven architectures with publish/subscribe patterns
- Build applications using the virtual actor pattern
- Handle secrets securely in distributed applications
- Implement workflow orchestration across microservices
- Add observability (metrics, tracing, logging) to Dapr applications
- Deploy and operate Dapr in production environments

## Prerequisites

- Dapr CLI installed on your development machine
- Understanding of distributed systems concepts
- Appropriate SDK for your preferred language (Go, .NET, Java, JavaScript, Python, etc.)

## Pre-Implementation Context Gathering

Before providing specific recommendations, this skill will gather important context about your requirements:

### Team Experience Assessment
- What is your team's current experience with Dapr? (beginner, intermediate, advanced)
- Have you worked with distributed systems before?
- Do you have experience with container orchestration (Docker, Kubernetes)?

### Architecture Requirements
- What distributed system patterns do you need? (service invocation, state management, pub/sub, actors)
- What state store providers do you plan to use? (Redis, SQL, NoSQL, etc.)
- Do you need actor-based microservices or traditional services?

### Scale Requirements
- What is your expected service-to-service call frequency?
- What is your expected message throughput for pub/sub?
- How much state storage do you anticipate?
- Do you anticipate rapid growth in service count?

### Latency Requirements
- What are your service invocation latency requirements?
- Are you processing real-time or near-real-time events?
- Do you need exactly-once, at-least-once, or at-most-once delivery semantics?

### Infrastructure Constraints
- What infrastructure do you have available? (cloud, on-premises, hybrid)
- What is your preferred deployment method? (self-hosted, Kubernetes, serverless)
- Do you have specific compliance or security requirements?

### Operational Considerations
- What monitoring and observability tools do you currently use?
- Do you have dedicated DevOps/SRE resources for Dapr operations?
- What are your disaster recovery and backup requirements?

## Quick Start: Hello World

### Install Dapr

```bash
# On Windows
winget install dapr

# On macOS
brew install dapr/tap/dapr-cli

# On Linux
wget -q https://raw.githubusercontent.com/dapr/cli/master/install/install.sh -O - | /bin/bash

# Or download manually from GitHub releases
curl -fsSL https://raw.githubusercontent.com/dapr/cli/master/install/install.sh | /bin/bash
```

### Initialize Dapr Locally

```bash
# Initialize Dapr with default settings (uses Docker)
dapr init

# Verify installation
dapr --version
dapr status -k  # Shows system services status
```

### Hello World Example with Python

1. Create a simple Python application:

```python
# app.py
import flask
from flask import Flask, request, jsonify
import time
import logging

app = Flask(__name__)

@app.route('/dapr/subscribe', methods=['GET'])
def subscribe():
    subscriptions = [
        {
            'pubsubname': 'pubsub',
            'topic': 'messages',
            'route': 'messages'
        }
    ]
    return jsonify(subscriptions)

@app.route('/messages', methods=['POST'])
def messages_handler():
    req_data = request.get_json()
    print(f'Received message: {req_data}', flush=True)
    return {'success': True}

@app.route('/hello', methods=['GET'])
def say_hello():
    return {'message': 'Hello from Dapr!'}, 200

@app.route('/state', methods=['POST'])
def save_state():
    # This will use Dapr's state management
    state_req = request.get_json()
    # In a real app, you would call Dapr sidecar to save state
    return {'success': True}, 200

if __name__ == "__main__":
    app.run(port=5000)
```

2. Create a Dapr component configuration for state management:

```yaml
# components/statestore.yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.redis
  version: v1
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: ""
  - name: actorStateStore
    value: "true"
```

3. Create a Dapr component configuration for pub/sub:

```yaml
# components/pubsub.yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
spec:
  type: pubsub.redis
  version: v1
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: ""
```

4. Run your application with Dapr:

```bash
# Run with Dapr sidecar
dapr run --app-id hello-dapr --app-port 5000 --dapr-http-port 3500 python app.py

# Call your service
curl http://localhost:3500/v1.0/invoke/hello-dapr/method/hello
```

## Dapr Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Application   │    │   Application   │    │   Application   │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          ▼                      ▼                      ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Dapr Sidecar  │    │   Dapr Sidecar  │    │   Dapr Sidecar  │
│    (daprd)      │    │    (daprd)      │    │    (daprd)      │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │   Dapr Control Plane    │
                    │  (Operator, Placement,  │
                    │    Sentry, Scheduler)   │
                    └─────────────────────────┘
```

### Sidecar Architecture Fundamentals

The Dapr sidecar pattern works like a **translator** between your application and infrastructure concerns. Your application speaks a standard HTTP/gRPC API to Dapr, and Dapr translates that to whatever infrastructure you're using (Redis, Kafka, Azure, AWS, etc.).

Just like a human translator allows you to communicate without knowing the other person's language, Dapr allows your application to communicate with various infrastructure services without being coupled to specific implementations.

### Dapr Sidecar (daprd) Ports

The Dapr sidecar exposes several ports for different communication patterns:

| Port | Protocol | Purpose |
|------|----------|---------|
| **3500** | HTTP | Primary HTTP API for Dapr building blocks |
| **50001** | gRPC | Primary gRPC API for Dapr building blocks |
| **9090** | HTTP | Metrics endpoint (Prometheus) |
| **9900** | HTTP | Health check endpoint |
| **50002** | gRPC | Internal control plane communication |

### Core Components

| Component | Purpose |
|-----------|---------|
| **Sidecar (daprd)** | Runtime that runs alongside your application, exposing Dapr APIs |
| **App ID** | Unique identifier for your application in Dapr |
| **Building Blocks** | Standardized APIs for common distributed system concerns |
| **Components** | Pluggable parts connecting to external systems (databases, queues, etc.) |
| **Control Plane** | System services managing security, actor placement, etc. |
| **SDK** | Language-specific libraries for interacting with Dapr APIs |

### Kubernetes Integration

When deploying Dapr applications to Kubernetes, three essential annotations are required for service invocation:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    metadata:
      annotations:
        dapr.io/enabled: "true"           # Enable Dapr sidecar injection
        dapr.io/app-id: "myapp"          # Unique application identifier for service invocation
        dapr.io/app-port: "8080"         # Application port for Dapr to route to
      # Optional annotations for enhanced service invocation:
      # dapr.io/config: "app-config"     # Configuration for service invocation policies
      # dapr.io/components: "component-1,component-2"  # Specific components to load
      # dapr.io/log-as-json: "true"      # Format Dapr logs as JSON
      # dapr.io/enable-api-logging: "true"  # Enable detailed API logging for debugging
      # dapr.io/sidecar-cpu-limit: "1.0" # Resource limits for sidecar
      # dapr.io/sidecar-memory-limit: "512Mi"
      # dapr.io/log-level: "debug"       # Set log level for troubleshooting
</template>
```

### Deployment Modes

Dapr supports two primary deployment modes:

#### Container Mode (Default)
- Dapr runs as a sidecar container in the same pod as your application
- Each application container gets its own Dapr sidecar container
- Provides complete isolation between application and Dapr runtime
- Recommended for production deployments

#### Process Mode (Self-Hosted)
- Dapr runs as a process on the same host as your application
- Uses localhost communication between application and Dapr
- Suitable for development and testing
- Achieved with `dapr run` command

## Dapr Building Blocks vs Components

Understanding the relationship between building blocks and components is crucial to Dapr:

### Building Blocks = Portable APIs
Building blocks are **standardized, portable APIs** that provide consistent interfaces for common distributed system patterns:
- **Service Invocation** - Portable service-to-service communication API
- **State Management** - Portable state storage and retrieval API
- **Publish & Subscribe** - Portable event messaging API
- **Bindings** - Portable input/output binding API
- **Actors** - Portable virtual actor pattern API
- **Secrets Management** - Portable secret access API

### Components = Pluggable Implementations
Components are **pluggable implementations** that connect building blocks to specific infrastructure:
- **state.redis** - Redis implementation for state management
- **pubsub.kafka** - Kafka implementation for pub/sub
- **bindings.rabbitmq** - RabbitMQ implementation for bindings
- **secretstores.azure.keyvault** - Azure Key Vault implementation for secrets

### Same Code, Different Backends
The magic happens when you can use the **same application code** with **different infrastructure** by changing only the component configuration:

**Application code (stays the same):**
```python
with DaprClient() as client:
    # This API call works with ANY state store provider
    client.save_state(store_name='statestore', key='mykey', value='myvalue')
```

**Component configuration (changes to switch infrastructure):**
```yaml
# To use Redis: change type to state.redis
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.redis
  # ... Redis-specific config

# To use PostgreSQL: change type to state.postgresql
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.postgresql
  # ... PostgreSQL-specific config

# To use Azure CosmosDB: change type to state.azure.cosmosdb
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.azure.cosmosdb
  # ... CosmosDB-specific config
```

Your application code remains unchanged while you can switch between different infrastructure providers!

## Dapr Building Blocks

Dapr provides several building blocks that address common distributed application challenges:

### 1. Service Invocation

Enables secure, reliable service-to-service communication with built-in service discovery, load balancing, and resilience features.

```
Service A (app-id: service-a)
    ↓ (calls Dapr API)
Dapr Sidecar → Service Discovery → Service B's Dapr Sidecar → Service B (app-id: service-b)
```

```bash
# Invoke another service
curl -X POST http://localhost:3500/v1.0/invoke/service-b/method/api/endpoint \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello"}'
```

### 2. State Management

Provides a consistent API for storing and retrieving state with support for various state store providers.

```bash
# Save state
curl -X POST http://localhost:3500/v1.0/state/statestore \
  -H "Content-Type: application/json" \
  -d '[{ "key": "myKey", "value": "myValue" }]'

# Get state
curl -X GET http://localhost:3500/v1.0/state/statestore/myKey
```

### 3. Publish & Subscribe

Enables event-driven architectures with support for various message brokers.

```bash
# Publish a message
curl -X POST http://localhost:3500/v1.0/publish/pubsub/messages \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello World!"}'
```

### 4. Bindings

Connects applications to external systems for input/output operations.

### 5. Actors

Implements the virtual actor pattern for stateful microservices.

### 6. Secrets Management

Provides secure access to secrets from various secret stores.

## Component Configuration Examples

### State Store Component (Redis)

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: mystatestore
spec:
  type: state.redis
  version: v1
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: ""
  - name: actorStateStore
    value: "true"
  - name: concurrency
    value: parallel
  - name: dialTimeout
    value: "5s"
  - name: readTimeout
    value: "3s"
```

### State Store Component Structure (Generic Template)

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore-name
  namespace: production  # Optional: specify namespace for scoping
spec:
  type: state.<provider>  # Examples: state.redis, state.azure.cosmosdb, state.postgresql
  version: v1
  metadata:
  # Connection settings
  - name: <connection-param>
    value: "<value>"
  # Authentication settings
  - name: <auth-param>
    secretKeyRef:
      name: <secret-name>
      key: <secret-key>
  # Configuration settings
  - name: <config-param>
    value: "<value>"
  # Metadata for state operations
  - name: <metadata-param>
    value: "<value>"
# Optional: scope this component to specific applications
scopes:
- app1
- app2
```

### State Store Component (Azure Cosmos DB)

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: cosmosdb-statestore
spec:
  type: state.azure.cosmosdb
  version: v1
  metadata:
  - name: url
    value: "https://<account>.documents.azure.com:443/"
  - name: masterKey
    secretKeyRef:
      name: cosmosdb-secrets
      key: masterKey
  - name: database
    value: "mydatabase"
  - name: collection
    value: "mycollection"
  - name: actorStateStore
    value: "true"
scopes:
- myapp
```

### State Store Component (PostgreSQL)

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: postgresql-statestore
spec:
  type: state.postgresql
  version: v1
  metadata:
  - name: connectionString
    secretKeyRef:
      name: postgres-secrets
      key: connectionString
  - name: versioning
    value: "true"
  - name: cleanupIntervalInMinutes
    value: "5"
scopes:
- myapp
```

### Pub/Sub Component (Redis)

```yaml
apiVolume: dapr.io/v1alpha1
kind: Component
metadata:
  name: mypubsub
spec:
  type: pubsub.redis
  version: v1
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: ""
  - name: consumerID
    value: "{{.AppID}}"
```

### Secret Store Component (Local File)

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: local-secret-store
spec:
  type: secretstores.local.file
  version: v1
  metadata:
  - name: secretsFile
    value: "/path/to/secrets.json"
  - name: nestedSeparator
    value: ":"
```

## Dapr Programming Models

### With SDKs (Recommended)

Dapr provides SDKs for multiple languages that make it easier to work with Dapr building blocks.

#### Python Example

```python
import dapr.clients
from dapr.clients import DaprClient

# Initialize Dapr client
with DaprClient() as dapr_client:
    # Invoke another service
    response = dapr_client.invoke_method(
        app_id='target-service',
        method_name='method-name',
        data={'message': 'Hello Dapr'},
        http_verb='POST'
    )

    # Save state
    dapr_client.save_state(
        store_name='statestore',
        key='myKey',
        value='myValue'
    )

    # Get state
    state_response = dapr_client.get_state(
        store_name='statestore',
        key='myKey'
    )
    value = state_response.data

    # Publish event
    dapr_client.publish_event(
        pubsub_name='pubsub',
        topic_name='messages',
        data={'message': 'Hello from Python'}
    )
```

### DaprClient Context Manager Pattern for State Operations

```python
from dapr.clients import DaprClient
from dapr.clients.exceptions import DaprInternalError
import json
from datetime import datetime

def perform_state_operations():
    """
    Demonstrates the DaprClient context manager pattern for state operations
    with proper error handling and resource management.
    """
    with DaprClient() as client:
        try:
            # Basic state operations
            # Save single state
            client.save_state(
                store_name='statestore',
                key='user:123',
                value=json.dumps({'name': 'John Doe', 'email': 'john@example.com'}),
                etag=None  # Use None for unconditional save
            )

            # Get state with metadata
            state_response = client.get_state(
                store_name='statestore',
                key='user:123',
                metadata={'partitionKey': 'users'}  # Optional metadata for partitioned stores
            )
            user_data = json.loads(state_response.data.decode('utf-8')) if state_response.data else None
            etag = state_response.etag

            # Delete state
            client.delete_state(
                store_name='statestore',
                key='user:123',
                etag=etag,  # Conditional delete with ETag
                options={
                    'consistency': 'strong'  # Options for consistency level
                }
            )

            # Bulk state operations
            states = [
                {
                    'key': 'product:1',
                    'value': json.dumps({'id': 1, 'name': 'Product A', 'price': 10.99}),
                },
                {
                    'key': 'product:2',
                    'value': json.dumps({'id': 2, 'name': 'Product B', 'price': 19.99}),
                }
            ]

            client.save_bulk_state(
                store_name='statestore',
                states=states
            )

            # Get bulk state
            bulk_response = client.get_bulk_state(
                store_name='statestore',
                keys=['product:1', 'product:2'],
                metadata={'partitionKey': 'products'}
            )

            for item in bulk_response.items:
                print(f"Key: {item.key}, Value: {item.data}, ETag: {item.etag}")

        except DaprInternalError as e:
            print(f"Dapr error occurred: {e}")
            raise
        except Exception as e:
            print(f"General error occurred: {e}")
            raise

def conditional_state_update(user_id: str, new_email: str):
    """
    Example of conditional state update using ETags to prevent conflicts
    """
    with DaprClient() as client:
        try:
            # Get current state and ETag
            state_response = client.get_state(
                store_name='statestore',
                key=f'user:{user_id}'
            )

            if not state_response.data:
                print(f"No user found with ID: {user_id}")
                return False

            current_user = json.loads(state_response.data.decode('utf-8'))
            current_etag = state_response.etag  # Get the current ETag

            # Update the user data
            current_user['email'] = new_email
            current_user['last_updated'] = str(datetime.utcnow().isoformat())

            # Attempt conditional save using ETag to prevent race conditions
            client.save_state(
                store_name='statestore',
                key=f'user:{user_id}',
                value=json.dumps(current_user),
                etag=current_etag  # Conditional save - will fail if ETag doesn't match
            )

            print(f"Successfully updated user {user_id}")
            return True

        except DaprInternalError as e:
            if "ETag mismatch" in str(e) or "precondition failed" in str(e).lower():
                print(f"ETag mismatch for user {user_id} - state was modified by another process")
                return False
            else:
                print(f"Dapr error occurred: {e}")
                raise
        except Exception as e:
            print(f"General error occurred: {e}")
            raise
```

### ETag-Based Optimistic Concurrency Example

ETags (entity tags) in Dapr provide a way to implement optimistic concurrency control, preventing lost updates when multiple clients try to modify the same state simultaneously. Here's an example of how to use ETags for safe concurrent state updates:

```python
from dapr.clients import DaprClient
from dapr.clients.exceptions import DaprInternalError
import json
from datetime import datetime
import threading
import time

def etag_concurrent_update_example():
    """
    Demonstrates ETag-based optimistic concurrency to prevent lost updates
    when multiple processes attempt to modify the same state.
    """
    with DaprClient() as client:
        # Initialize state
        initial_value = {
            'counter': 0,
            'last_modified_by': 'initializer',
            'timestamp': datetime.now().isoformat()
        }

        # Save initial state
        client.save_state(
            store_name='statestore',
            key='concurrent-counter',
            value=json.dumps(initial_value)
        )

        def increment_counter(thread_id: str):
            """Function to increment counter with ETag checking"""
            max_retries = 5

            for attempt in range(max_retries):
                try:
                    # Get current state and ETag
                    state_response = client.get_state(
                        store_name='statestore',
                        key='concurrent-counter'
                    )

                    if not state_response.data:
                        print(f"Thread {thread_id}: No state found")
                        return

                    current_data = json.loads(state_response.data.decode('utf-8'))
                    current_etag = state_response.etag

                    # Update the counter
                    current_data['counter'] += 1
                    current_data['last_modified_by'] = f'thread-{thread_id}'
                    current_data['timestamp'] = datetime.now().isoformat()

                    # Attempt to save with ETag - will fail if state changed since we read it
                    client.save_state(
                        store_name='statestore',
                        key='concurrent-counter',
                        value=json.dumps(current_data),
                        etag=current_etag  # Conditional save with ETag
                    )

                    print(f"Thread {thread_id}: Successfully incremented counter to {current_data['counter']} (attempt {attempt + 1})")
                    return  # Success, exit retry loop

                except DaprInternalError as e:
                    if "ETag mismatch" in str(e) or "precondition failed" in str(e).lower():
                        print(f"Thread {thread_id}: ETag mismatch on attempt {attempt + 1}, retrying...")
                        time.sleep(0.1)  # Brief delay before retry
                        continue  # Retry the operation
                    else:
                        print(f"Thread {thread_id}: Dapr error: {e}")
                        raise
                except Exception as e:
                    print(f"Thread {thread_id}: Unexpected error: {e}")
                    raise

            print(f"Thread {thread_id}: Failed to update after {max_retries} attempts")

        # Simulate concurrent updates from multiple threads
        threads = []
        for i in range(3):  # 3 concurrent updates
            thread = threading.Thread(target=increment_counter, args=(str(i),))
            threads.append(thread)
            thread.start()

        # Wait for all threads to complete
        for thread in threads:
            thread.join()

        # Read final state
        final_response = client.get_state(
            store_name='statestore',
            key='concurrent-counter'
        )

        if final_response.data:
            final_data = json.loads(final_response.data.decode('utf-8'))
            print(f"Final counter value: {final_data['counter']}")
            print(f"Last modified by: {final_data['last_modified_by']}")
        else:
            print("No final state found")

# Example of implementing a distributed lock using ETags
def distributed_lock_example():
    """
    Example of using ETags to implement a simple distributed lock mechanism
    """
    with DaprClient() as client:
        lock_key = 'distributed-lock:resource-1'

        # Try to acquire lock
        def try_acquire_lock(client, lock_holder_id: str, ttl_seconds: int = 30):
            lock_data = {
                'holder': lock_holder_id,
                'acquired_at': datetime.now().isoformat(),
                'ttl': ttl_seconds
            }

            try:
                # Attempt to create the lock with an initial save
                # If key already exists, this will fail
                client.save_state(
                    store_name='statestore',
                    key=lock_key,
                    value=json.dumps(lock_data),
                    etag=None  # No ETag means create only if doesn't exist
                )

                print(f"Lock acquired by {lock_holder_id}")
                return True

            except DaprInternalError:
                # Lock already exists, try to update with ETag verification
                state_response = client.get_state(
                    store_name='statestore',
                    key=lock_key
                )

                if state_response.data:
                    current_data = json.loads(state_response.data.decode('utf-8'))
                    current_etag = state_response.etag

                    # Check if lock is expired
                    acquired_time = datetime.fromisoformat(current_data['acquired_at'])
                    if (datetime.now() - acquired_time).seconds > current_data['ttl']:
                        # Lock is expired, try to acquire it
                        new_lock_data = {
                            'holder': lock_holder_id,
                            'acquired_at': datetime.now().isoformat(),
                            'ttl': ttl_seconds
                        }

                        try:
                            client.save_state(
                                store_name='statestore',
                                key=lock_key,
                                value=json.dumps(new_lock_data),
                                etag=current_etag  # Conditional update
                            )
                            print(f"Expired lock acquired by {lock_holder_id}")
                            return True
                        except DaprInternalError:
                            print(f"Failed to acquire expired lock, someone else got it")
                            return False
                    else:
                        print(f"Lock held by {current_data['holder']}, cannot acquire")
                        return False
                else:
                    # Unexpected state, try initial save again
                    try:
                        client.save_state(
                            store_name='statestore',
                            key=lock_key,
                            value=json.dumps(lock_data),
                            etag=None
                        )
                        print(f"Lock acquired by {lock_holder_id}")
                        return True
                    except DaprInternalError:
                        return False

        def release_lock(client, lock_holder_id: str):
            """Release the lock if we're the holder"""
            state_response = client.get_state(
                store_name='statestore',
                key=lock_key
            )

            if state_response.data:
                current_data = json.loads(state_response.data.decode('utf-8'))
                current_etag = state_response.etag

                if current_data['holder'] == lock_holder_id:
                    # Delete the lock
                    client.delete_state(
                        store_name='statestore',
                        key=lock_key,
                        etag=current_etag
                    )
                    print(f"Lock released by {lock_holder_id}")
                    return True
                else:
                    print(f"Cannot release lock - held by {current_data['holder']}")
                    return False
            return False

        # Example usage
        if try_acquire_lock(client, "process-1"):
            # Do work while holding lock
            time.sleep(2)
            release_lock(client, "process-1")
```

### Service Invocation Patterns

Dapr service invocation enables secure, reliable service-to-service communication with built-in service discovery, load balancing, and resilience features.

#### Python SDK Service Invocation (invoke_method)

```python
from dapr.clients import DaprClient
import json

def service_invocation_examples():
    with DaprClient() as client:
        # Basic service invocation
        response = client.invoke_method(
            app_id='target-service',
            method_name='api/users',
            data={'userId': '123', 'action': 'get'},
            http_verb='GET',
            http_query={'format': 'json', 'include_metadata': 'true'}
        )

        # Parse response
        result_data = response.text()
        print(f"Response: {result_data}")

        # POST request with JSON payload
        response = client.invoke_method(
            app_id='order-service',
            method_name='create-order',
            data=json.dumps({
                'customerId': '456',
                'items': ['item1', 'item2'],
                'total': 99.99
            }),
            http_verb='POST',
            content_type='application/json'
        )

        # Handling binary data
        binary_data = b'some binary content'
        response = client.invoke_method(
            app_id='image-processor',
            method_name='resize',
            data=binary_data,
            http_verb='POST',
            content_type='application/octet-stream'
        )

        # With custom headers
        response = client.invoke_method(
            app_id='auth-service',
            method_name='validate-token',
            data={'token': 'abc123'},
            http_verb='POST',
            metadata={
                'Authorization': 'Bearer token123',
                'X-Custom-Header': 'custom-value'
            }
        )
```

#### Kubernetes Service Invocation Annotations

When deploying Dapr-enabled applications to Kubernetes, specific annotations are required for service invocation:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    metadata:
      annotations:
        # Essential annotations for service invocation
        dapr.io/enabled: "true"              # Enable Dapr sidecar injection
        dapr.io/app-id: "myapp"             # Unique app identifier for service invocation
        dapr.io/app-port: "8080"            # Port where your app listens

        # Optional annotations for enhanced service invocation
        dapr.io/config: "app-config"        # Configuration for service invocation policies
        dapr.io/log-as-json: "true"         # Better logging for debugging invocations
        dapr.io/enable-api-logging: "true"  # Log all API calls for debugging
        dapr.io/sidecar-listen-addresses: "0.0.0.0"  # Address for sidecar listening
```

#### Alternative: HTTP Header Pattern for Service Invocation

Instead of using the app-id in the URL path, you can use the `dapr-app-id` HTTP header:

```bash
# Traditional approach: app-id in URL path
curl -X POST http://localhost:3500/v1.0/invoke/target-service/method/api/endpoint \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello"}'

# Alternative approach: using dapr-app-id header
curl -X POST http://localhost:3500/v1.0/invoke/method/api/endpoint \
  -H "Content-Type: application/json" \
  -H "dapr-app-id: target-service" \
  -d '{"message": "Hello"}'

# This is particularly useful for programmatic invocations where the target service
# is determined at runtime
```

#### Common Service Invocation Errors and Solutions

##### ERR_DIRECT_INVOKE Error
This error occurs when Dapr cannot reach the target service:

```python
from dapr.clients.exceptions import DaprInternalError

try:
    response = client.invoke_method(
        app_id='non-existent-service',
        method_name='api/endpoint',
        data={'test': 'data'}
    )
except DaprInternalError as e:
    if "ERR_DIRECT_INVOKE" in str(e):
        print(f"Service invocation failed: {e}")
        # Solutions:
        # 1. Verify target service is running and registered with Dapr
        # 2. Check that dapr.io/app-id annotation matches the invoked app_id
        # 3. Ensure both services are in the same namespace or accessible cross-namespace
        # 4. Check network connectivity between services
```

##### Connection Refused Error
This occurs when the Dapr sidecar cannot establish connection:

```python
# Troubleshooting connection refused errors:
# 1. Verify Dapr sidecar is running alongside your application
dapr list  # Should show both services

# 2. Check if the target app-port is correct
# 3. Verify the application is listening on the specified port
# 4. Check firewall/network policies restricting communication
# 5. Ensure sufficient resources for both application and sidecar
```

#### Service Invocation Debugging

Enable API logging to troubleshoot service invocation issues:

```yaml
# In your Kubernetes deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: debuggable-service
spec:
  template:
    metadata:
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "debuggable-service"
        dapr.io/app-port: "8080"
        # Enable detailed API logging for debugging service invocation
        dapr.io/enable-api-logging: "true"
        # Additional debugging annotations
        dapr.io/log-level: "debug"
        dapr.io/sidecar-readiness-probe-delay: "10"
        dapr.io/sidecar-readiness-probe-timeout: "5"
```

```bash
# View Dapr sidecar logs for service invocation debugging
dapr logs --app-id myapp --tail

# Check service invocation metrics
kubectl port-forward -n dapr-system svc/dapr-metrics 9900:9900
# Then visit http://localhost:9900 to see metrics including service invocation stats

# Verify service registration
kubectl exec -it <pod-name> -- dapr service-invocation --verbose
```

### Direct HTTP/gRPC API Calls

You can also interact with Dapr directly using HTTP or gRPC APIs without SDKs.

```bash
# Service invocation
curl -X POST http://localhost:3500/v1.0/invoke/target-service/method/api/endpoint \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello"}'

# State management
curl -X POST http://localhost:3500/v1.0/state/statestore \
  -H "Content-Type: application/json" \
  -d '[{ "key": "myKey", "value": "myValue" }]'

# Publish event
curl -X POST http://localhost:3500/v1.0/publish/pubsub/messages \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello World!"}'
```

## Security in Dapr

### mTLS (Mutual TLS)

Dapr automatically handles mTLS between sidecars for secure communication:

```yaml
# Configuration for enabling mTLS in Kubernetes
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: dapr-config
spec:
  mtls:
    enabled: true
    workloadCertTTL: 24h
    allowedClockSkew: 15m
```

### API Authentication

Restrict which Dapr APIs applications can access:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: api-allowlist-config
spec:
  httpPipeline:
    handlers:
    - name: oauth2
      type: middleware.http.oauth2
  apiAllowlist:
    enabled: true
    verbs:
    - GET
    - POST
```

### Component Scoping

Limit which applications can use specific components:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: sensitive-component
spec:
  # ... component spec ...
scopes:
- app1
- app2
```

## Observability in Dapr

### Tracing

Configure distributed tracing with various backends:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: tracing
spec:
  tracing:
    samplingRate: "1"
    zipkin:
      endpointAddress: "http://localhost:9411/api/v2/spans"
```

### Metrics

Dapr exposes Prometheus-compatible metrics:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: metrics
spec:
  metrics:
    enabled: true
    port: 9090
```

## Production Deployment Patterns

### Kubernetes Deployment

```yaml
# Sample deployment with Dapr sidecar injection
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "myapp"
        dapr.io/app-port: "8080"
        dapr.io/config: "app-config"
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
```

### Self-Hosted Production

For self-hosted production scenarios:

```bash
# Run Dapr with specific configurations
dapr run \
  --app-id myapp \
  --app-port 8080 \
  --dapr-http-port 3500 \
  --dapr-grpc-port 50001 \
  --config ./config/config.yaml \
  --components-path ./components \
  --log-level warn \
  -- python app.py
```

## Common Commands

```bash
# Initialize Dapr
dapr init

# Run an application with Dapr sidecar
dapr run --app-id myapp --app-port 8080 python app.py

# List running Dapr applications
dapr list

# Stop a Dapr application
dapr stop --app-id myapp

# Check Dapr runtime status
dapr status

# Get Dapr sidecar logs
dapr logs --app-id myapp

# Dashboard for monitoring Dapr applications
dapr dashboard

# Remove Dapr runtime
dapr uninstall
```

## Helm Deployment Commands

### Installing Dapr with Helm

```bash
# Add the Dapr Helm repository
helm repo add dapr https://dapr.github.io/helm-charts/
helm repo update

# Create the dapr-system namespace
kubectl create namespace dapr-system

# Install Dapr with --wait flag for verification
helm install dapr dapr/dapr \
  --namespace dapr-system \
  --set-string global.tag=1.11.0 \
  --wait \
  --timeout 10m

# Verify installation
helm status dapr --namespace dapr-system
kubectl get pods --namespace dapr-system
dapr status -k
```

### Upgrading Dapr with Helm

```bash
# Upgrade Dapr with verification
helm upgrade dapr dapr/dapr \
  --namespace dapr-system \
  --set-string global.tag=1.12.0 \
  --wait \
  --timeout 10m \
  --atomic

# Verify the upgrade
kubectl rollout status deployment/dapr-operator --namespace dapr-system
kubectl rollout status deployment/dapr-placement --namespace dapr-system
kubectl rollout status deployment/dapr-sentry --namespace dapr-system
kubectl rollout status deployment/dapr-sidecar-injector --namespace dapr-system
```

### Uninstalling Dapr with Helm

```bash
# Uninstall Dapr with verification
helm uninstall dapr --namespace dapr-system
kubectl delete namespace dapr-system
```

## Production Checklist

Before deploying to production:

### Infrastructure
- [ ] Dapr control plane services deployed and monitored
- [ ] High availability configuration for control plane
- [ ] Proper resource allocation for Dapr sidecars
- [ ] Network policies configured for Dapr communication

### Security
- [ ] mTLS enabled for all communication
- [ ] API allow-lists configured appropriately
- [ ] Component scoping applied to limit access
- [ ] Secrets management configured securely
- [ ] Authentication and authorization implemented

### Configuration
- [ ] Appropriate resiliency policies defined
- [ ] Proper state store configuration for production
- [ ] Production pub/sub broker configured
- [ ] Monitoring and alerting configured

### Monitoring
- [ ] Dapr metrics collected and visualized
- [ ] Distributed tracing enabled
- [ ] Health checks implemented
- [ ] Log aggregation configured
- [ ] Performance baselines established

### Operations
- [ ] Backup and recovery procedures for state
- [ ] Rollback procedures defined
- [ ] Scaling strategies implemented
- [ ] Regular security updates planned

## Reference Documentation

| Reference | Description |
|-----------|-------------|
| [CORE_CONCEPTS.md](references/CORE_CONCEPTS.md) | Dapr's core concepts, sidecar pattern, building blocks, and component architecture |
| [SECURITY.md](references/SECURITY.md) | Security features including mTLS, authentication, component scoping, and security best practices |
| [DEPLOYMENT.md](references/DEPLOYMENT.md) | Deployment patterns for self-hosted and Kubernetes, component management, and production best practices |
| [SIDECAR_ARCHITECTURE.md](references/SIDECAR_ARCHITECTURE.md) | Sidecar architecture fundamentals, translator analogy, port configurations, Kubernetes annotations, and deployment modes |

## Official Documentation

- [Dapr Documentation](https://docs.dapr.io/)
- [Dapr GitHub Repository](https://github.com/dapr/dapr)
- [Dapr SDKs](https://docs.dapr.io/developing-applications/sdks/)
- [Component Reference](https://docs.dapr.io/reference/components-reference/)
- [Configuration Reference](https://docs.dapr.io/reference/components-and-scopes/)