
---

It’s designed to handle I/O-bound tasks (like web requests or database queries) efficiently by allowing the program to do other things while waiting for data to return.

We  can start multiple i/o processes and run them and simultaneiusly get the results using asyncio package. 

```python
import asyncio
import time

# Simulated async database clients
async def fetch_from_vector_db(query: str) -> dict:
    await asyncio.sleep(1.2)  # Simulate network latency
    return {"source": "vector_db", "text": "Company Q3 revenue grew by 15%.", "score": 0.92}

async def fetch_from_web_search(query: str) -> dict:
    await asyncio.sleep(0.8)  # Simulate API latency
    return {"source": "web", "text": "Recent news shows a 15% bump in Q3.", "score": 0.88}

async def fetch_from_internal_wiki(query: str) -> dict:
    await asyncio.sleep(1.0)  # Simulate database latency
    return {"source": "wiki", "text": "Internal targets for Q3 were exceeded.", "score": 0.75}

async def high_throughput_retrieval(query: str) -> list[dict]:
    print(f"Starting concurrent retrieval for: '{query}'...")
    start_time = time.time()
    
    # Schedule all I/O bound tasks to run concurrently
    tasks = [
        fetch_from_vector_db(query),
        fetch_from_web_search(query),
        fetch_from_internal_wiki(query)
    ]
    
    # await asyncio.gather executes them in parallel and waits for all to finish
    results = await asyncio.gather(*tasks)
    
    print(f"Retrieval completed in {time.time() - start_time:.2f} seconds.")
    return results

# To run the async function in a standard Python script:
# results = asyncio.run(high_throughput_retrieval("Q3 revenue"))
```

### Basics

**Coroutines:** Functions defined with `async def`. They don't run immediately when called; they return a coroutine object that needs to be scheduled to run.

**The Event Loop:** The "manager" that schedules and runs your asynchronous tasks.

**Await:** The keyword used to pause a coroutine until the awaited task is finished, yielding control back to the event loop.

```python
import asyncio

async def say_hello():
    print("Starting...")
    await asyncio.sleep(1) # Simulates an I/O operation
    print("...Hello!")

# The entry point to run the top-level coroutine
asyncio.run(say_hello())
```

We can also run multiple tasks at the same time. 

```python
import asyncio

async def fetch_data(id, delay):
    await asyncio.sleep(delay)
    return f"Data {id}"

async def main():

    # Schedule multiple tasks simultaneously
    results = await asyncio.gather(
        fetch_data(1, 3),
        fetch_data(2, 1),
        fetch_data(3, 2)
    )

asyncio.run(main())
```

The results will be returned in the finish first order. The operation to finish first's will have the result. Usually result is a list of outputs. 

Notes - 

- Calling the `asyncio.run()` stops the current run untill inside function does not stops.
- `asyncio.fetch()` is strictly await reason being it is strcitly scheduled using event loop. 

### Details - 

- `Asyncio.run()` creates the event loop. Without this call event loop does not  exists. It takes one top-level function (usually called `main()`) and runs it until it finishes. Once that main function is done, `run()` shuts down the loop, closes any remaining "background" connections. 
- `gather` call on the other hand is used to run and orchestrate multiple tasks. It will run all the background tasks and wait for the results. 
- await - It is used to handle the async tasks in the event loop. Once statement prefixed with await is reached the call is scheduled in the event loop. However event loop will not pause the evecution entirely. Although the current function will stop event loop can schedule some other tasks in the mean time. 
