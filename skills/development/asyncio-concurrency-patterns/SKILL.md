---
name: asyncio-concurrency-patterns
description: Complete guide for asyncio concurrency patterns including event loops, coroutines, tasks, futures, async context managers, and performance optimization
tags: [asyncio, concurrency, async, python, event-loop, coroutines, performance]
tier: tier-1
---

# Asyncio Concurrency Patterns

A comprehensive skill for mastering Python's asyncio library and concurrent programming patterns. This skill covers event loops, coroutines, tasks, futures, synchronization primitives, async context managers, and production-ready patterns for building high-performance asynchronous applications.

## When to Use This Skill

Use this skill when:

- Building I/O-bound applications that need to handle many concurrent operations
- Creating web servers, API clients, or websocket applications
- Implementing real-time systems with event-driven architecture
- Optimizing application performance with concurrent request handling
- Managing multiple async operations with proper coordination and error handling
- Building background task processors or job queues
- Implementing async database operations and connection pooling
- Creating chat applications, real-time dashboards, or notification systems
- Handling parallel HTTP requests efficiently
- Managing websocket connections with multiple event sources
- Building microservices with async communication patterns
- Optimizing resource utilization in network applications

## Core Concepts

### What is Asyncio?

Asyncio is Python's built-in library for writing concurrent code using the async/await syntax. It provides:

- **Event Loop**: The core of asyncio that schedules and runs asynchronous tasks
- **Coroutines**: Functions defined with `async def` that can be paused and resumed
- **Tasks**: Scheduled coroutines that run concurrently
- **Futures**: Low-level objects representing results of async operations
- **Synchronization Primitives**: Locks, semaphores, events for coordination

### Event Loop Fundamentals

The event loop is the central execution mechanism in asyncio:

```python
import asyncio

# Get or create an event loop
loop = asyncio.get_event_loop()

# Run a coroutine until complete
loop.run_until_complete(my_coroutine())

# Modern approach (Python 3.7+)
asyncio.run(my_coroutine())
```

**Key Event Loop Concepts:**

1. **Single-threaded concurrency**: One thread, many tasks
2. **Cooperative multitasking**: Tasks yield control voluntarily
3. **I/O multiplexing**: Efficient handling of many I/O operations
4. **Non-blocking operations**: Don't wait for I/O, do other work

### Coroutines vs Functions

**Regular Function:**
```python
def fetch_data():
    # Blocks until complete
    return requests.get('http://api.example.com')
```

**Coroutine:**
```python
async def fetch_data():
    # Yields control while waiting
    async with aiohttp.ClientSession() as session:
        async with session.get('http://api.example.com') as resp:
            return await resp.text()
```

### Tasks and Futures

**Tasks** wrap coroutines and schedule them on the event loop:

```python
# Create a task
task = asyncio.create_task(my_coroutine())

# Task runs in background
# ... do other work ...

# Wait for result
result = await task
```

**Futures** represent eventual results:

```python
# Low-level future (rarely used directly)
future = asyncio.Future()

# Set result
future.set_result(42)

# Get result
result = await future
```

### Async Context Managers

Manage resources with async setup/teardown:

```python
class AsyncResource:
    async def __aenter__(self):
        # Async setup
        await self.connect()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        # Async cleanup
        await self.disconnect()

# Usage
async with AsyncResource() as resource:
    await resource.do_work()
```

## Concurrency Patterns

### Pattern 1: Gather - Concurrent Execution

Run multiple coroutines concurrently and wait for all to complete:

```python
import asyncio
import aiohttp

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    async with aiohttp.ClientSession() as session:
        # Run all fetches concurrently
        results = await asyncio.gather(
            fetch(session, 'http://python.org'),
            fetch(session, 'http://docs.python.org'),
            fetch(session, 'http://pypi.org')
        )
        return results

# Results is a list in the same order as inputs
results = asyncio.run(main())
```

**When to use:**
- Need all results
- Order matters
- Want to fail fast on first exception (default)
- Can handle partial results with `return_exceptions=True`

### Pattern 2: Wait - Flexible Waiting

More control over how to wait for multiple tasks:

```python
import asyncio

async def task_a():
    await asyncio.sleep(2)
    return 'A'

async def task_b():
    await asyncio.sleep(1)
    return 'B'

async def main():
    tasks = [
        asyncio.create_task(task_a()),
        asyncio.create_task(task_b())
    ]

    # Wait for first to complete
    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.FIRST_COMPLETED
    )

    # Get first result
    first_result = done.pop().result()

    # Cancel remaining
    for task in pending:
        task.cancel()

    return first_result

result = asyncio.run(main())  # Returns 'B' after 1 second
```

**Wait strategies:**
- `FIRST_COMPLETED`: Return when first task finishes
- `FIRST_EXCEPTION`: Return when first task raises exception
- `ALL_COMPLETED`: Wait for all tasks (default)

### Pattern 3: Semaphore - Limit Concurrency

Control maximum number of concurrent operations:

```python
import asyncio
import aiohttp

async def fetch_with_limit(session, url, semaphore):
    async with semaphore:
        # Only N requests run concurrently
        async with session.get(url) as resp:
            return await resp.text()

async def main():
    # Limit to 5 concurrent requests
    semaphore = asyncio.Semaphore(5)

    urls = [f'http://api.example.com/item/{i}' for i in range(100)]

    async with aiohttp.ClientSession() as session:
        tasks = [
            fetch_with_limit(session, url, semaphore)
            for url in urls
        ]
        results = await asyncio.gather(*tasks)

    return results

asyncio.run(main())
```

**When to use:**
- Rate limiting API requests
- Controlling database connection usage
- Preventing resource exhaustion
- Respecting external service limits

### Pattern 4: Lock - Mutual Exclusion

Ensure only one coroutine accesses a resource at a time:

```python
import asyncio

class SharedCounter:
    def __init__(self):
        self.value = 0
        self.lock = asyncio.Lock()

    async def increment(self):
        async with self.lock:
            # Critical section - only one coroutine at a time
            current = self.value
            await asyncio.sleep(0)  # Simulate async work
            self.value = current + 1

async def worker(counter):
    for _ in range(100):
        await counter.increment()

async def main():
    counter = SharedCounter()

    # Run 10 workers concurrently
    await asyncio.gather(*[worker(counter) for _ in range(10)])

    print(f"Final count: {counter.value}")  # Always 1000

asyncio.run(main())
```

### Pattern 5: Event - Signaling

Coordinate multiple coroutines with events:

```python
import asyncio

async def waiter(event, name):
    print(f'{name} waiting for event')
    await event.wait()
    print(f'{name} received event')

async def setter(event):
    await asyncio.sleep(2)
    print('Setting event')
    event.set()

async def main():
    event = asyncio.Event()

    # Multiple waiters
    await asyncio.gather(
        waiter(event, 'Waiter 1'),
        waiter(event, 'Waiter 2'),
        waiter(event, 'Waiter 3'),
        setter(event)
    )

asyncio.run(main())
```

### Pattern 6: Queue - Producer/Consumer

Coordinate work between producers and consumers:

```python
import asyncio

async def producer(queue, n):
    for i in range(n):
        await asyncio.sleep(0.1)
        await queue.put(f'item-{i}')
        print(f'Produced item-{i}')

    # Signal completion
    await queue.put(None)

async def consumer(queue, name):
    while True:
        item = await queue.get()

        if item is None:
            # Propagate sentinel to other consumers
            await queue.put(None)
            break

        print(f'{name} processing {item}')
        await asyncio.sleep(0.2)
        queue.task_done()

async def main():
    queue = asyncio.Queue()

    # Start producer and consumers
    await asyncio.gather(
        producer(queue, 10),
        consumer(queue, 'Consumer-1'),
        consumer(queue, 'Consumer-2'),
        consumer(queue, 'Consumer-3')
    )

asyncio.run(main())
```

## Task Management

### Creating Tasks

**Basic Task Creation:**

```python
import asyncio

async def background_task():
    await asyncio.sleep(10)
    return 'Done'

async def main():
    # Create task - starts running immediately
    task = asyncio.create_task(background_task())

    # Do other work while task runs
    await asyncio.sleep(1)

    # Wait for result
    result = await task
    return result

asyncio.run(main())
```

**Named Tasks (Python 3.8+):**

```python
task = asyncio.create_task(
    background_task(),
    name='my-background-task'
)

print(task.get_name())  # 'my-background-task'
```

### Task Cancellation

**Graceful Cancellation:**

```python
import asyncio

async def long_running_task():
    try:
        while True:
            await asyncio.sleep(1)
            print('Working...')
    except asyncio.CancelledError:
        print('Task cancelled, cleaning up...')
        # Cleanup logic
        raise  # Re-raise to mark as cancelled

async def main():
    task = asyncio.create_task(long_running_task())

    # Let it run for 3 seconds
    await asyncio.sleep(3)

    # Request cancellation
    task.cancel()

    try:
        await task
    except asyncio.CancelledError:
        print('Task was cancelled')

asyncio.run(main())
```

**Cancellation with Context Manager:**

```python
import asyncio
from contextlib import suppress

async def run_with_timeout():
    task = asyncio.create_task(long_running_task())

    try:
        # Wait with timeout
        await asyncio.wait_for(task, timeout=5.0)
    except asyncio.TimeoutError:
        task.cancel()
        with suppress(asyncio.CancelledError):
            await task
```

### Exception Handling in Tasks

**Gather with Exception Handling:**

```python
import asyncio

async def failing_task(n):
    await asyncio.sleep(n)
    raise ValueError(f'Task {n} failed')

async def successful_task(n):
    await asyncio.sleep(n)
    return f'Task {n} succeeded'

async def main():
    # return_exceptions=True: Returns exceptions instead of raising
    results = await asyncio.gather(
        successful_task(1),
        failing_task(2),
        successful_task(3),
        return_exceptions=True
    )

    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f'Task {i} failed: {result}')
        else:
            print(f'Task {i} result: {result}')

asyncio.run(main())
```

**Task Exception Retrieval:**

```python
import asyncio

async def main():
    task = asyncio.create_task(failing_task(1))

    # Wait for task
    await asyncio.sleep(2)

    # Check if task failed
    if task.done() and task.exception():
        print(f'Task failed with: {task.exception()}')

asyncio.run(main())
```

## Event Loop Management

### Event Loop Policies

**Default Event Loop:**

```python
import asyncio

async def main():
    # Get running loop
    loop = asyncio.get_running_loop()
    print(f'Loop: {loop}')

asyncio.run(main())
```

**Custom Event Loop:**

```python
import asyncio

async def main():
    pass

# Create new event loop
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)

try:
    loop.run_until_complete(main())
finally:
    loop.close()
```

**Event Loop Best Practices:**

1. **Use `asyncio.run()`** for simple programs (Python 3.7+)
2. **Avoid creating ClientSession outside event loop**
3. **Always close loops when done**
4. **Don't call blocking functions in event loop**

### Running Blocking Code

**Using ThreadPoolExecutor:**

```python
import asyncio
import time
from concurrent.futures import ThreadPoolExecutor

def blocking_io():
    # Blocking operation
    time.sleep(2)
    return 'Done'

async def main():
    loop = asyncio.get_running_loop()

    # Run blocking code in thread pool
    result = await loop.run_in_executor(
        None,  # Use default executor
        blocking_io
    )

    return result

asyncio.run(main())
```

**Custom Executor:**

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

async def main():
    loop = asyncio.get_running_loop()

    # Custom executor with 4 threads
    with ThreadPoolExecutor(max_workers=4) as executor:
        results = await asyncio.gather(*[
            loop.run_in_executor(executor, blocking_io)
            for _ in range(10)
        ])

    return results

asyncio.run(main())
```

### Loop Callbacks

**Schedule Callback:**

```python
import asyncio

def callback(arg):
    print(f'Callback called with {arg}')

async def main():
    loop = asyncio.get_running_loop()

    # Schedule callback
    loop.call_soon(callback, 'immediate')

    # Schedule with delay
    loop.call_later(2, callback, 'delayed')

    # Schedule at specific time
    loop.call_at(loop.time() + 3, callback, 'scheduled')

    await asyncio.sleep(4)

asyncio.run(main())
```

## Async Context Managers

### Creating Async Context Managers

**Class-Based:**

```python
import asyncio

class AsyncDatabaseConnection:
    def __init__(self, host):
        self.host = host
        self.connection = None

    async def __aenter__(self):
        print(f'Connecting to {self.host}')
        await asyncio.sleep(0.1)  # Simulate connection
        self.connection = f'Connection to {self.host}'
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print(f'Closing connection to {self.host}')
        await asyncio.sleep(0.1)  # Simulate cleanup
        self.connection = None

    async def query(self, sql):
        if not self.connection:
            raise RuntimeError('Not connected')
        await asyncio.sleep(0.05)
        return f'Results for: {sql}'

async def main():
    async with AsyncDatabaseConnection('localhost') as db:
        result = await db.query('SELECT * FROM users')
        print(result)

asyncio.run(main())
```

**Decorator-Based:**

```python
import asyncio
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_resource(name):
    # Setup
    print(f'Acquiring {name}')
    await asyncio.sleep(0.1)

    try:
        yield name
    finally:
        # Cleanup
        print(f'Releasing {name}')
        await asyncio.sleep(0.1)

async def main():
    async with async_resource('database') as db:
        print(f'Using {db}')

asyncio.run(main())
```

### Real-World Example: aiohttp ClientSession

```python
import aiohttp
import asyncio

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    # ClientSession as async context manager
    async with aiohttp.ClientSession() as session:
        html = await fetch(session, 'http://python.org')
        print(f'Body: {html[:100]}...')

asyncio.run(main())
```

**Why use async context manager for ClientSession?**

1. Ensures proper cleanup of connections
2. Prevents resource leaks
3. Manages SSL connections correctly
4. Handles graceful shutdown

## Performance Optimization

### Profiling Async Code

**Basic Timing:**

```python
import asyncio
import time

async def slow_operation():
    await asyncio.sleep(1)

async def main():
    start = time.perf_counter()

    await slow_operation()

    elapsed = time.perf_counter() - start
    print(f'Took {elapsed:.2f} seconds')

asyncio.run(main())
```

**Profiling Multiple Operations:**

```python
import asyncio
import time

async def timed_task(name, duration):
    start = time.perf_counter()
    await asyncio.sleep(duration)
    elapsed = time.perf_counter() - start
    print(f'{name} took {elapsed:.2f}s')
    return name

async def main():
    await asyncio.gather(
        timed_task('Task 1', 1),
        timed_task('Task 2', 2),
        timed_task('Task 3', 0.5)
    )

asyncio.run(main())
```

### Optimizing Concurrency

**Bad - Sequential Execution:**

```python
async def slow_approach():
    results = []
    for i in range(10):
        result = await fetch_data(i)
        results.append(result)
    return results
# Takes 10 * fetch_time
```

**Good - Concurrent Execution:**

```python
async def fast_approach():
    tasks = [fetch_data(i) for i in range(10)]
    results = await asyncio.gather(*tasks)
    return results
# Takes ~fetch_time
```

**Better - Controlled Concurrency:**

```python
async def controlled_approach():
    semaphore = asyncio.Semaphore(5)  # Max 5 concurrent

    async def fetch_with_limit(i):
        async with semaphore:
            return await fetch_data(i)

    tasks = [fetch_with_limit(i) for i in range(10)]
    results = await asyncio.gather(*tasks)
    return results
# Takes ~2 * fetch_time, but respects limits
```

### Avoiding Common Performance Pitfalls

**1. Don't create sessions per request:**

```python
# BAD - Creates new session each time
async def bad_fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.text()

# GOOD - Reuse session
async def good_fetch():
    async with aiohttp.ClientSession() as session:
        results = await asyncio.gather(
            session.get('http://example.com/1'),
            session.get('http://example.com/2'),
            session.get('http://example.com/3')
        )
        return results
```

**2. Don't use blocking operations:**

```python
import asyncio
import requests  # Blocking library

# BAD - Blocks event loop
async def bad_request():
    response = requests.get('http://example.com')  # BLOCKS!
    return response.text

# GOOD - Use async library
async def good_request():
    async with aiohttp.ClientSession() as session:
        async with session.get('http://example.com') as resp:
            return await resp.text()

# ACCEPTABLE - If must use blocking, use executor
async def acceptable_request():
    loop = asyncio.get_running_loop()
    result = await loop.run_in_executor(
        None,
        lambda: requests.get('http://example.com').text
    )
    return result
```

**3. Proper cleanup with zero-sleep:**

```python
async def proper_cleanup():
    async with aiohttp.ClientSession() as session:
        async with session.get('http://example.org/') as resp:
            await resp.read()

    # Zero-sleep to allow underlying connections to close
    await asyncio.sleep(0)
```

## Common Pitfalls

### Pitfall 1: Creating ClientSession Outside Event Loop

**Problem:**

```python
import aiohttp

# BAD - Session created outside event loop
session = aiohttp.ClientSession()

async def fetch(url):
    async with session.get(url) as resp:
        return await resp.text()
```

**Why it's bad:**
- Session binds to event loop at creation time
- If loop changes (e.g., uvloop), session becomes invalid
- Can cause program to hang

**Solution:**

```python
import aiohttp
import asyncio

async def main():
    # Create session inside async function
    async with aiohttp.ClientSession() as session:
        async with session.get('http://python.org') as resp:
            print(await resp.text())

asyncio.run(main())
```

### Pitfall 2: Session as Class Variable

**Problem:**

```python
class API:
    session = aiohttp.ClientSession()  # BAD - global instance

    async def fetch(self, url):
        async with self.session.get(url) as resp:
            return await resp.text()
```

**Solution:**

```python
class API:
    def __init__(self):
        self.session = None

    async def __aenter__(self):
        self.session = aiohttp.ClientSession()
        return self

    async def __aexit__(self, *args):
        await self.session.close()

    async def fetch(self, url):
        async with self.session.get(url) as resp:
            return await resp.text()

# Usage
async def main():
    async with API() as api:
        result = await api.fetch('http://example.com')
```

### Pitfall 3: Forgetting await

**Problem:**

```python
async def process_data():
    # Forgot await - returns coroutine, doesn't execute!
    result = fetch_data()  # Missing await
    return result
```

**Solution:**

```python
async def process_data():
    result = await fetch_data()  # Proper await
    return result
```

### Pitfall 4: Blocking the Event Loop

**Problem:**

```python
import asyncio
import time

async def bad_sleep():
    time.sleep(5)  # BAD - Blocks entire event loop!

async def main():
    await asyncio.gather(
        bad_sleep(),
        another_task()  # Blocked for 5 seconds
    )
```

**Solution:**

```python
import asyncio

async def good_sleep():
    await asyncio.sleep(5)  # GOOD - Yields control

async def main():
    await asyncio.gather(
        good_sleep(),
        another_task()  # Runs concurrently
    )
```

### Pitfall 5: Not Handling Task Cancellation

**Problem:**

```python
async def bad_task():
    while True:
        await asyncio.sleep(1)
        process_data()
        # No cleanup on cancellation!
```

**Solution:**

```python
async def good_task():
    try:
        while True:
            await asyncio.sleep(1)
            process_data()
    except asyncio.CancelledError:
        # Cleanup resources
        cleanup()
        raise  # Re-raise to mark as cancelled
```

### Pitfall 6: Deadlocks with Locks

**Problem:**

```python
import asyncio

lock1 = asyncio.Lock()
lock2 = asyncio.Lock()

async def task_a():
    async with lock1:
        await asyncio.sleep(0.1)
        async with lock2:  # Deadlock potential
            pass

async def task_b():
    async with lock2:
        await asyncio.sleep(0.1)
        async with lock1:  # Deadlock potential
            pass
```

**Solution:**

```python
# Always acquire locks in same order
async def safe_task_a():
    async with lock1:
        async with lock2:
            pass

async def safe_task_b():
    async with lock1:  # Same order
        async with lock2:
            pass
```

## Production Patterns

### Pattern 1: Graceful Shutdown

**Complete Shutdown Example:**

```python
import asyncio
import signal
from contextlib import suppress

class Application:
    def __init__(self):
        self.should_exit = False
        self.tasks = []

    async def worker(self, name):
        try:
            while not self.should_exit:
                print(f'{name} working...')
                await asyncio.sleep(1)
        except asyncio.CancelledError:
            print(f'{name} cancelled, cleaning up...')
            raise

    def handle_signal(self, sig):
        print(f'Received signal {sig}, shutting down...')
        self.should_exit = True

    async def run(self):
        # Setup signal handlers
        loop = asyncio.get_running_loop()
        for sig in (signal.SIGTERM, signal.SIGINT):
            loop.add_signal_handler(
                sig,
                lambda s=sig: self.handle_signal(s)
            )

        # Start workers
        self.tasks = [
            asyncio.create_task(self.worker(f'Worker-{i}'))
            for i in range(3)
        ]

        # Wait for shutdown signal
        while not self.should_exit:
            await asyncio.sleep(0.1)

        # Cancel all tasks
        for task in self.tasks:
            task.cancel()

        # Wait for cancellation to complete
        await asyncio.gather(*self.tasks, return_exceptions=True)

        print('Shutdown complete')

# Run application
app = Application()
asyncio.run(app.run())
```

### Pattern 2: Background Tasks with Application Lifecycle

**aiohttp Application with Background Tasks:**

```python
import asyncio
from contextlib import suppress
from aiohttp import web

async def listen_to_redis(app):
    """Background task that listens to Redis"""
    # Simulated Redis listening
    try:
        while True:
            # Process messages
            await asyncio.sleep(1)
            print('Processing Redis message...')
    except asyncio.CancelledError:
        print('Redis listener stopped')
        raise

async def background_tasks(app):
    """Cleanup context for managing background tasks"""
    # Startup: Create background task
    app['redis_listener'] = asyncio.create_task(listen_to_redis(app))

    yield  # App is running

    # Cleanup: Cancel background task
    app['redis_listener'].cancel()
    with suppress(asyncio.CancelledError):
        await app['redis_listener']

# Setup application
app = web.Application()
app.cleanup_ctx.append(background_tasks)
```

### Pattern 3: Retry Logic with Exponential Backoff

```python
import asyncio
import aiohttp
from typing import Any, Callable

async def retry_with_backoff(
    coro_func: Callable,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    *args,
    **kwargs
) -> Any:
    """
    Retry async function with exponential backoff

    Args:
        coro_func: Async function to retry
        max_retries: Maximum number of retries
        base_delay: Initial delay between retries
        max_delay: Maximum delay between retries
    """
    for attempt in range(max_retries):
        try:
            return await coro_func(*args, **kwargs)
        except Exception as e:
            if attempt == max_retries - 1:
                # Last attempt failed
                raise

            # Calculate delay with exponential backoff
            delay = min(base_delay * (2 ** attempt), max_delay)

            print(f'Attempt {attempt + 1} failed: {e}')
            print(f'Retrying in {delay:.1f} seconds...')

            await asyncio.sleep(delay)

# Usage
async def unstable_api_call():
    async with aiohttp.ClientSession() as session:
        async with session.get('http://unstable-api.com') as resp:
            return await resp.json()

async def main():
    result = await retry_with_backoff(
        unstable_api_call,
        max_retries=5,
        base_delay=1.0
    )
    return result
```

### Pattern 4: Circuit Breaker

```python
import asyncio
from datetime import datetime, timedelta
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"      # Normal operation
    OPEN = "open"          # Failing, reject requests
    HALF_OPEN = "half_open"  # Testing if recovered

class CircuitBreaker:
    def __init__(
        self,
        failure_threshold: int = 5,
        recovery_timeout: float = 60.0,
        success_threshold: int = 2
    ):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.success_threshold = success_threshold

        self.failure_count = 0
        self.success_count = 0
        self.state = CircuitState.CLOSED
        self.opened_at = None

    async def call(self, coro_func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            # Check if should try recovery
            if datetime.now() - self.opened_at > timedelta(seconds=self.recovery_timeout):
                self.state = CircuitState.HALF_OPEN
                self.success_count = 0
            else:
                raise Exception('Circuit breaker is OPEN')

        try:
            result = await coro_func(*args, **kwargs)
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        self.failure_count = 0

        if self.state == CircuitState.HALF_OPEN:
            self.success_count += 1
            if self.success_count >= self.success_threshold:
                self.state = CircuitState.CLOSED
                self.success_count = 0

    def _on_failure(self):
        self.failure_count += 1

        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
            self.opened_at = datetime.now()

# Usage
async def flaky_service():
    # Simulated flaky service
    import random
    await asyncio.sleep(0.1)
    if random.random() < 0.5:
        raise Exception('Service error')
    return 'Success'

async def main():
    breaker = CircuitBreaker(failure_threshold=3, recovery_timeout=5.0)

    for i in range(20):
        try:
            result = await breaker.call(flaky_service)
            print(f'Request {i}: {result} - State: {breaker.state.value}')
        except Exception as e:
            print(f'Request {i}: Failed - State: {breaker.state.value}')

        await asyncio.sleep(0.5)
```

### Pattern 5: WebSocket with Multiple Event Sources

**Handling Parallel WebSocket and Background Events:**

```python
import asyncio
from aiohttp import web

async def read_subscription(ws, redis):
    """Background task reading from Redis and sending to WebSocket"""
    # Simulated Redis subscription
    channel = await redis.subscribe('channel:1')

    try:
        # Simulate receiving messages
        for i in range(10):
            await asyncio.sleep(1)
            message = f'Redis message {i}'
            await ws.send_str(message)
    finally:
        await redis.unsubscribe('channel:1')

async def websocket_handler(request):
    """WebSocket handler with parallel event sources"""
    ws = web.WebSocketResponse()
    await ws.prepare(request)

    # Create background task for Redis subscription
    redis = request.app['redis']
    task = asyncio.create_task(read_subscription(ws, redis))

    try:
        # Handle incoming WebSocket messages
        async for msg in ws:
            if msg.type == web.WSMsgType.TEXT:
                # Process incoming message
                await ws.send_str(f'Echo: {msg.data}')
            elif msg.type == web.WSMsgType.ERROR:
                print(f'WebSocket error: {ws.exception()}')
    finally:
        # Cleanup: Cancel background task
        task.cancel()

    return ws
```

## Best Practices

### Testing Async Code

**Using pytest-asyncio:**

```python
import pytest
import asyncio

@pytest.mark.asyncio
async def test_async_function():
    result = await async_operation()
    assert result == 'expected'

@pytest.mark.asyncio
async def test_with_fixture(aiohttp_client):
    client = await aiohttp_client(create_app())
    resp = await client.get('/')
    assert resp.status == 200
```

**Manual Event Loop Setup:**

```python
import asyncio
import unittest

class TestAsyncCode(unittest.TestCase):
    def setUp(self):
        self.loop = asyncio.new_event_loop()
        asyncio.set_event_loop(self.loop)

    def tearDown(self):
        self.loop.close()

    def test_coroutine(self):
        async def test_impl():
            result = await async_function()
            self.assertEqual(result, 'expected')

        self.loop.run_until_complete(test_impl())
```

### Debugging Async Code

**Enable Debug Mode:**

```python
import asyncio
import warnings

# Enable asyncio debug mode
asyncio.run(main(), debug=True)

# Or manually
loop = asyncio.get_event_loop()
loop.set_debug(True)
loop.run_until_complete(main())
```

**What debug mode detects:**
- Coroutines that were never awaited
- Callbacks taking too long
- Tasks destroyed while pending

**Logging Slow Callbacks:**

```python
import asyncio
import logging

logging.basicConfig(level=logging.DEBUG)

loop = asyncio.get_event_loop()
loop.slow_callback_duration = 0.1  # 100ms threshold
loop.set_debug(True)
```

### Documentation

**Documenting Async Functions:**

```python
async def fetch_user_data(user_id: int) -> dict:
    """
    Fetch user data from the database.

    Args:
        user_id: The unique identifier of the user

    Returns:
        Dictionary containing user data

    Raises:
        UserNotFoundError: If user doesn't exist
        DatabaseError: If database connection fails

    Example:
        >>> async def main():
        ...     user = await fetch_user_data(123)
        ...     print(user['name'])

    Note:
        This function must be called within an async context.
        Connection pooling is handled automatically.
    """
    async with get_db_connection() as conn:
        return await conn.fetch_one(
            'SELECT * FROM users WHERE id = $1',
            user_id
        )
```

## Complete Examples

### Example 1: Parallel HTTP Requests

```python
import asyncio
import aiohttp
import time

async def fetch(session, url):
    """Fetch a single URL"""
    async with session.get(url) as response:
        return {
            'url': url,
            'status': response.status,
            'length': len(await response.text())
        }

async def fetch_all(urls):
    """Fetch multiple URLs concurrently"""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

async def main():
    urls = [
        'http://python.org',
        'http://docs.python.org',
        'http://pypi.org',
        'http://github.com/python',
        'http://www.python.org/dev/peps/'
    ]

    start = time.perf_counter()
    results = await fetch_all(urls)
    elapsed = time.perf_counter() - start

    for result in results:
        print(f"{result['url']}: {result['status']} ({result['length']} bytes)")

    print(f"\nFetched {len(urls)} URLs in {elapsed:.2f} seconds")

asyncio.run(main())
```

### Example 2: Rate-Limited API Client

```python
import asyncio
import aiohttp
from typing import List, Dict, Any

class RateLimitedClient:
    def __init__(self, rate_limit: int = 10):
        """
        Args:
            rate_limit: Maximum concurrent requests
        """
        self.semaphore = asyncio.Semaphore(rate_limit)
        self.session = None

    async def __aenter__(self):
        self.session = aiohttp.ClientSession()
        return self

    async def __aexit__(self, *args):
        await self.session.close()
        # Allow connections to close
        await asyncio.sleep(0)

    async def fetch(self, url: str) -> Dict[str, Any]:
        """Fetch URL with rate limiting"""
        async with self.semaphore:
            print(f'Fetching {url}')
            async with self.session.get(url) as resp:
                return {
                    'url': url,
                    'status': resp.status,
                    'data': await resp.json()
                }

    async def fetch_all(self, urls: List[str]) -> List[Dict[str, Any]]:
        """Fetch all URLs with rate limiting"""
        tasks = [self.fetch(url) for url in urls]
        return await asyncio.gather(*tasks, return_exceptions=True)

async def main():
    urls = [f'https://api.github.com/users/{user}'
            for user in ['python', 'django', 'flask', 'requests', 'aiohttp']]

    async with RateLimitedClient(rate_limit=2) as client:
        results = await client.fetch_all(urls)

        for result in results:
            if isinstance(result, Exception):
                print(f'Error: {result}')
            else:
                print(f"User: {result['data'].get('login', 'unknown')}")

asyncio.run(main())
```

### Example 3: Database Connection Pool

```python
import asyncio
from typing import List, Any

class AsyncConnectionPool:
    def __init__(self, size: int = 10):
        self.pool = asyncio.Queue(maxsize=size)
        self.size = size

    async def init(self):
        """Initialize connection pool"""
        for i in range(self.size):
            conn = await self._create_connection(i)
            await self.pool.put(conn)

    async def _create_connection(self, conn_id: int):
        """Create a database connection (simulated)"""
        await asyncio.sleep(0.1)  # Simulate connection time
        return {'id': conn_id, 'connected': True}

    async def acquire(self):
        """Acquire connection from pool"""
        return await self.pool.get()

    async def release(self, conn):
        """Release connection back to pool"""
        await self.pool.put(conn)

    async def execute(self, query: str) -> Any:
        """Execute query using pooled connection"""
        conn = await self.acquire()
        try:
            # Simulate query execution
            await asyncio.sleep(0.05)
            return f"Query '{query}' executed on connection {conn['id']}"
        finally:
            await self.release(conn)

    async def close(self):
        """Close all connections"""
        while not self.pool.empty():
            conn = await self.pool.get()
            # Close connection (simulated)
            conn['connected'] = False

async def worker(pool: AsyncConnectionPool, worker_id: int):
    """Worker that executes queries"""
    for i in range(5):
        result = await pool.execute(f'SELECT * FROM table WHERE id={i}')
        print(f'Worker {worker_id}: {result}')

async def main():
    # Create and initialize pool
    pool = AsyncConnectionPool(size=5)
    await pool.init()

    # Run multiple workers concurrently
    await asyncio.gather(*[
        worker(pool, i) for i in range(10)
    ])

    # Cleanup
    await pool.close()

asyncio.run(main())
```

### Example 4: Real-Time Data Processor

```python
import asyncio
import random
from datetime import datetime

class DataProcessor:
    def __init__(self):
        self.queue = asyncio.Queue()
        self.processed = 0
        self.errors = 0

    async def producer(self, producer_id: int):
        """Produce data items"""
        for i in range(10):
            await asyncio.sleep(random.uniform(0.1, 0.5))
            item = {
                'producer_id': producer_id,
                'item_id': i,
                'timestamp': datetime.now(),
                'data': random.randint(1, 100)
            }
            await self.queue.put(item)
            print(f'Producer {producer_id} generated item {i}')

        # Signal completion
        await self.queue.put(None)

    async def consumer(self, consumer_id: int):
        """Consume and process data items"""
        while True:
            item = await self.queue.get()

            if item is None:
                # Propagate sentinel
                await self.queue.put(None)
                break

            try:
                # Simulate processing
                await asyncio.sleep(random.uniform(0.05, 0.2))

                # Process item
                result = item['data'] * 2
                print(f"Consumer {consumer_id} processed: {item['item_id']} -> {result}")

                self.processed += 1
            except Exception as e:
                print(f'Consumer {consumer_id} error: {e}')
                self.errors += 1
            finally:
                self.queue.task_done()

    async def monitor(self):
        """Monitor processing statistics"""
        while True:
            await asyncio.sleep(2)
            print(f'\n=== Stats: Processed={self.processed}, Errors={self.errors}, Queue={self.queue.qsize()} ===\n')

    async def run(self, num_producers: int = 3, num_consumers: int = 5):
        """Run the data processor"""
        # Start monitor
        monitor_task = asyncio.create_task(self.monitor())

        # Start producers and consumers
        await asyncio.gather(
            *[self.producer(i) for i in range(num_producers)],
            *[self.consumer(i) for i in range(num_consumers)]
        )

        # Cancel monitor
        monitor_task.cancel()

        print(f'\nFinal Stats: Processed={self.processed}, Errors={self.errors}')

async def main():
    processor = DataProcessor()
    await processor.run(num_producers=3, num_consumers=5)

asyncio.run(main())
```

### Example 5: Async File I/O with aiofiles

```python
import asyncio
import aiofiles
from pathlib import Path

async def write_file(path: str, content: str):
    """Write content to file asynchronously"""
    async with aiofiles.open(path, 'w') as f:
        await f.write(content)

async def read_file(path: str) -> str:
    """Read file content asynchronously"""
    async with aiofiles.open(path, 'r') as f:
        return await f.read()

async def process_files(file_paths: list):
    """Process multiple files concurrently"""
    tasks = [read_file(path) for path in file_paths]
    contents = await asyncio.gather(*tasks)

    # Process contents
    results = []
    for path, content in zip(file_paths, contents):
        result = {
            'path': path,
            'lines': len(content.split('\n')),
            'words': len(content.split()),
            'chars': len(content)
        }
        results.append(result)

    return results

async def main():
    # Create test files
    test_files = ['test1.txt', 'test2.txt', 'test3.txt']

    # Write files concurrently
    await asyncio.gather(*[
        write_file(f, f'Content of file {f}\n' * 10)
        for f in test_files
    ])

    # Process files
    results = await process_files(test_files)

    for result in results:
        print(f"{result['path']}: {result['lines']} lines, "
              f"{result['words']} words, {result['chars']} chars")

    # Cleanup
    for f in test_files:
        Path(f).unlink(missing_ok=True)

# asyncio.run(main())  # Uncomment to run (requires aiofiles)
```

## Resources

- **Python asyncio Documentation**: https://docs.python.org/3/library/asyncio.html
- **aiohttp Documentation**: https://docs.aiohttp.org/
- **Real Python asyncio Guide**: https://realpython.com/async-io-python/
- **PEP 492 - Coroutines with async and await syntax**: https://www.python.org/dev/peps/pep-0492/
- **asyncio Cheat Sheet**: https://www.pythonsheets.com/notes/python-asyncio.html
- **Effective Python: Item 60 - Consider asyncio**: https://effectivepython.com/

---

**Skill Version**: 1.0.0
**Last Updated**: October 2025
**Skill Category**: Concurrency, Performance, Async Programming
**Compatible With**: Python 3.7+, aiohttp, asyncio, uvloop
