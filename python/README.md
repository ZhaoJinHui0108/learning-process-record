# Python 学习笔记

## 目录
1. [热门常用库](#热门常用库)
2. [异步操作](#异步操作)
3. [高并发操作](#高并发操作)

---

## 热门常用库

### 常用标准库
- `os` - 操作系统接口
- `sys` - 系统参数和函数
- `json` - JSON处理
- `datetime` - 日期时间处理
- `collections` - 集合数据类型
- `itertools` - 迭代器工具
- `functools` - 函数式编程工具
- `pathlib` - 路径操作
- `logging` - 日志记录
- `argparse` - 命令行参数

### 常用第三方库
- `requests` - HTTP客户端
- `numpy` - 数值计算
- `pandas` - 数据分析
- `matplotlib` - 数据可视化
- `scikit-learn` - 机器学习
- `flask` / `fastapi` - Web框架
- `django` - 全栈Web框架
- `sqlalchemy` - ORM
- `celery` - 任务队列
- `pytest` - 测试框架
- `pydantic` - 数据验证

---

## 异步操作

### asyncio 基本概念
- `async` / `await` - 异步关键字
- `Coroutine` - 协程对象
- `Task` - 任务对象
- `EventLoop` - 事件循环

### 示例1: 基础异步操作
```python
import asyncio
import time

async def say_hello(name: str, delay: int):
    """异步函数：延迟打印问候语"""
    await asyncio.sleep(delay)
    print(f"Hello, {name}! (waited {delay}s)")

async def main():
    """主异步函数"""
    print("Start...")
    
    # 并发执行多个协程
    await asyncio.gather(
        say_hello("Alice", 2),
        say_hello("Bob", 1),
        say_hello("Charlie", 3)
    )
    
    print("All done!")

if __name__ == '__main__':
    start = time.time()
    asyncio.run(main())
    print(f"Total time: {time.time() - start:.2f}s")
```

### 示例2: 异步上下文管理器
```python
import asyncio
import aiohttp

class AsyncTimer:
    """异步上下文管理器，计时代码块执行时间"""
    
    def __init__(self, name: str):
        self.name = name
        self.start_time = None
    
    async def __aenter__(self):
        self.start_time = time.time()
        print(f"[{self.name}] Started")
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        elapsed = time.time() - self.start_time
        print(f"[{self.name}] Finished in {elapsed:.2f}s")
        return False

async def fetch_data(session: aiohttp.ClientSession, url: str):
    """异步获取数据"""
    async with session.get(url) as response:
        return await response.text()

async def main():
    """使用异步上下文管理器"""
    urls = [
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/2",
        "https://httpbin.org/delay/1"
    ]
    
    async with AsyncTimer("Fetch Data"):
        async with aiohttp.ClientSession() as session:
            tasks = [fetch_data(session, url) for url in urls]
            results = await asyncio.gather(*tasks)
    
    print(f"Fetched {len(results)} responses")

if __name__ == '__main__':
    import time
    asyncio.run(main())
```

### 示例3: 异步迭代器
```python
import asyncio
import aiofiles

class AsyncLineReader:
    """异步文件读取器"""
    
    def __init__(self, filepath: str):
        self.filepath = filepath
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if not hasattr(self, 'file'):
            self.file = await aiofiles.open(self.filepath, mode='r')
            self.line = None
        
        line = await self.file.readline()
        if line:
            self.line = line.strip()
            return self.line
        else:
            await self.file.close()
            raise StopAsyncIteration

async def process_file(filepath: str):
    """处理文件中的每一行"""
    async for line in AsyncLineReader(filepath):
        print(f"Processing: {line}")
        await asyncio.sleep(0.1)

if __name__ == '__main__':
    asyncio.run(process_file("example.txt"))
```

### 示例4: 异步队列
```python
import asyncio
import random

class AsyncProducer:
    """异步生产者"""
    
    def __init__(self, queue: asyncio.Queue, name: str, count: int):
        self.queue = queue
        self.name = name
        self.count = count
    
    async def produce(self):
        for i in range(self.count):
            item = f"{self.name}-item-{i}"
            await self.queue.put(item)
            print(f"[{self.name}] Produced: {item}")
            await asyncio.sleep(random.uniform(0.1, 0.5))

class AsyncConsumer:
    """异步消费者"""
    
    def __init__(self, queue: asyncio.Queue, name: str):
        self.queue = queue
        self.name = name
    
    async def consume(self):
        while True:
            try:
                item = await asyncio.wait_for(self.queue.get(), timeout=2.0)
                print(f"[{self.name}] Consumed: {item}")
                await asyncio.sleep(random.uniform(0.2, 0.8))
                self.queue.task_done()
            except asyncio.TimeoutError:
                print(f"[{self.name}] Queue empty, stopping...")
                break

async def main():
    """异步生产者-消费者示例"""
    queue = asyncio.Queue(maxsize=10)
    
    # 创建生产者和消费者
    producers = [
        AsyncProducer(queue, "Producer-A", 3),
        AsyncProducer(queue, "Producer-B", 3)
    ]
    consumers = [
        AsyncConsumer(queue, "Consumer-1"),
        AsyncConsumer(queue, "Consumer-2")
    ]
    
    # 并发运行所有任务
    await asyncio.gather(
        *[p.produce() for p in producers],
        *[c.consume() for c in consumers]
    )
    
    print("All producers and consumers finished!")

if __name__ == '__main__':
    asyncio.run(main())
```

### 示例5: 异步锁和信号量
```python
import asyncio

class AsyncResource:
    """使用信号量限制并发访问的资源"""
    
    def __init__(self, max_connections: int = 2):
        self.semaphore = asyncio.Semaphore(max_connections)
        self.active_connections = 0
    
    async def use(self, name: str):
        async with self.semaphore:
            self.active_connections += 1
            print(f"[{name}] Connected. Active: {self.active_connections}")
            
            # 模拟工作
            await asyncio.sleep(random.uniform(0.5, 1.5))
            
            self.active_connections -= 1
            print(f"[{name}] Disconnected. Active: {self.active_connections}")

async def main():
    """使用信号量限制并发"""
    import random
    resource = AsyncResource(max_connections=2)
    
    # 尝试创建5个并发连接，但只有2个可以同时活跃
    tasks = [resource.use(f"Task-{i}") for i in range(5)]
    await asyncio.gather(*tasks)

if __name__ == '__main__':
    asyncio.run(main())
```

### 示例6: 异步HTTP请求
```python
import asyncio
import aiohttp
import time

async def fetch_all(urls: list[str]) -> list[dict]:
    """并发获取多个URL"""
    async with aiohttp.ClientSession() as session:
        async def fetch(url: str) -> dict:
            start = time.time()
            async with session.get(url) as response:
                text = await response.text()
                return {
                    "url": url,
                    "status": response.status,
                    "content_length": len(text),
                    "time": time.time() - start
                }
        
        tasks = [fetch(url) for url in urls]
        return await asyncio.gather(*tasks)

async def post_data(url: str, data: dict) -> dict:
    """异步POST请求"""
    async with aiohttp.ClientSession() as session:
        async with session.post(url, json=data) as response:
            return await response.json()

async def main():
    urls = [
        "https://httpbin.org/get",
        "https://httpbin.org/ip",
        "https://httpbin.org/headers",
        "https://httpbin.org/uuid",
    ]
    
    start = time.time()
    results = await fetch_all(urls)
    print(f"Fetched {len(results)} URLs in {time.time() - start:.2f}s")
    
    for result in results:
        print(f"  {result['url']}: {result['status']} ({result['time']:.2f}s)")

if __name__ == '__main__':
    asyncio.run(main())
```

---

## 高并发操作

### 示例1: 多线程基础
```python
import threading
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

def download_file(file_id: int, delay: float) -> dict:
    """模拟文件下载"""
    thread_name = threading.current_thread().name
    print(f"[{thread_name}] Downloading file {file_id}...")
    time.sleep(delay)
    return {"file_id": file_id, "size": delay * 100}

def main():
    """多线程下载示例"""
    files = [
        (1, 1.0),
        (2, 0.5),
        (3, 2.0),
        (4, 0.8),
        (5, 1.5)
    ]
    
    print("Sequential downloads:")
    start = time.time()
    for file_id, delay in files:
        result = download_file(file_id, delay)
        print(f"  Downloaded file {file_id}, size: {result['size']}KB")
    print(f"Total time: {time.time() - start:.2f}s\n")
    
    print("Parallel downloads with ThreadPoolExecutor:")
    start = time.time()
    
    with ThreadPoolExecutor(max_workers=3) as executor:
        futures = {executor.submit(download_file, fid, delay): fid for fid, delay in files}
        
        for future in as_completed(futures):
            file_id = futures[future]
            try:
                result = future.result()
                print(f"  Downloaded file {result['file_id']}, size: {result['size']}KB")
            except Exception as e:
                print(f"  File {file_id} failed: {e}")
    
    print(f"Total time: {time.time() - start:.2f}s")

if __name__ == '__main__':
    main()
```

### 示例2: 多进程基础
```python
from multiprocessing import Process, Queue, cpu_count
import time

def worker(worker_id: int, task_queue: Queue, result_queue: Queue):
    """工作进程"""
    while True:
        task = task_queue.get()
        if task is None:  # 终止信号
            break
        
        task_id, delay = task
        time.sleep(delay)
        result = {"worker_id": worker_id, "task_id": task_id}
        result_queue.put(result)
        print(f"[Worker-{worker_id}] Processed task {task_id}")

def main():
    """多进程计算示例"""
    num_workers = min(cpu_count(), 4)
    print(f"Starting {num_workers} workers...\n")
    
    task_queue = Queue()
    result_queue = Queue()
    
    # 添加任务
    tasks = [(i, 1.0 - i*0.1) for i in range(1, 6)]
    for task in tasks:
        task_queue.put(task)
    
    # 添加终止信号
    for _ in range(num_workers):
        task_queue.put(None)
    
    # 启动工作进程
    workers = []
    for i in range(num_workers):
        worker_process = Process(target=worker, args=(i, task_queue, result_queue))
        worker_process.start()
        workers.append(worker_process)
    
    # 收集结果
    results = []
    for _ in range(len(tasks)):
        result = result_queue.get()
        results.append(result)
    
    # 等待所有工作进程结束
    for worker_process in workers:
        worker_process.join()
    
    print(f"\nProcessed {len(results)} tasks")

if __name__ == '__main__':
    main()
```

### 示例3: 线程间同步
```python
import threading
import time
from threading import Lock, Condition, Event

class Counter:
    """线程安全的计数器"""
    
    def __init__(self):
        self.value = 0
        self.lock = Lock()
    
    def increment(self, amount: int = 1):
        with self.lock:
            self.value += amount
            return self.value
    
    def get(self):
        with self.lock:
            return self.value

class TaskRunner:
    """带事件通知的任务运行器"""
    
    def __init__(self):
        self.tasks_done = 0
        self.lock = Lock()
        self.all_done = Condition(self.lock)
        self.expected_tasks = 0
    
    def set_expected(self, count: int):
        with self.lock:
            self.expected_tasks = count
    
    def complete_task(self):
        with self.lock:
            self.tasks_done += 1
            print(f"Task completed: {self.tasks_done}/{self.expected_tasks}")
            if self.tasks_done >= self.expected_tasks:
                self.all_done.notify_all()
    
    def wait_all_done(self, timeout: float = None):
        with self.lock:
            while self.tasks_done < self.expected_tasks:
                self.all_done.wait(timeout=timeout)
            print("All tasks completed!")

def worker(task_id: int, runner: TaskRunner):
    """模拟工作线程"""
    time.sleep(1 - task_id * 0.1)
    print(f"Worker {task_id} starting task...")
    time.sleep(0.5)
    runner.complete_task()

def main():
    """线程同步示例"""
    runner = TaskRunner()
    num_tasks = 5
    
    runner.set_expected(num_tasks)
    threads = []
    
    for i in range(num_tasks):
        t = threading.Thread(target=worker, args=(i, runner))
        threads.append(t)
        t.start()
    
    runner.wait_all_done(timeout=10)
    
    for t in threads:
        t.join()
    
    print("Main thread finished")

if __name__ == '__main__':
    main()
```

### 示例4: 生产者-消费者模式
```python
import threading
import time
from queue import Queue, Empty
from concurrent.futures import ThreadPoolExecutor

class Producer(threading.Thread):
    """生产者线程"""
    
    def __init__(self, queue: Queue, name: str, count: int, delay: float):
        super().__init__()
        self.queue = queue
        self.name = name
        self.count = count
        self.delay = delay
    
    def run(self):
        for i in range(self.count):
            item = f"{self.name}-{i}"
            self.queue.put(item)
            print(f"[{self.name}] Produced: {item}")
            time.sleep(self.delay)

class Consumer(threading.Thread):
    """消费者线程"""
    
    def __init__(self, queue: Queue, name: str, max_get: int = None):
        super().__init__()
        self.queue = queue
        self.name = name
        self.max_get = max_get
        self.consumed = 0
    
    def run(self):
        while self.max_get is None or self.consumed < self.max_get:
            try:
                item = self.queue.get(timeout=1)
                print(f"[{self.name}] Consumed: {item}")
                self.queue.task_done()
                self.consumed += 1
                time.sleep(0.5)
            except Empty:
                if self.queue.empty():
                    break

def main():
    """生产者-消费者示例"""
    queue = Queue(maxsize=10)
    
    # 启动生产者
    producers = [
        Producer(queue, "Producer-A", 3, 0.3),
        Producer(queue, "Producer-B", 3, 0.2)
    ]
    
    # 启动消费者
    consumers = [
        Consumer(queue, "Consumer-1", max_get=3),
        Consumer(queue, "Consumer-2", max_get=3)
    ]
    
    all_threads = producers + consumers
    
    for t in all_threads:
        t.start()
    
    for t in all_threads:
        t.join()
    
    print("\nAll threads finished!")

if __name__ == '__main__':
    main()
```

### 示例5: 进程池
```python
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor
import multiprocessing as mp
import time
import math

def is_prime(n: int) -> tuple[int, bool]:
    """判断素数"""
    if n < 2:
        return (n, False)
    if n == 2:
        return (n, True)
    if n % 2 == 0:
        return (n, False)
    
    for i in range(3, int(math.sqrt(n)) + 1, 2):
        if n % i == 0:
            return (n, False)
    return (n, True)

def cpu_bound_task(n: int) -> int:
    """CPU密集型任务"""
    total = 0
    for i in range(n):
        total += sum(math.sqrt(i) for i in range(100))
    return total

def main():
    numbers = [1000000 + i for i in range(20)]
    
    print("ProcessPoolExecutor:")
    start = time.time()
    with ProcessPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(is_prime, numbers))
    print(f"  Found {sum(1 for _, prime in results if prime)} primes")
    print(f"  Time: {time.time() - start:.2f}s\n")
    
    print("ThreadPoolExecutor (for CPU-bound):")
    start = time.time()
    with ThreadPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(cpu_bound_task, [1000]*10))
    print(f"  Completed {len(results)} tasks")
    print(f"  Time: {time.time() - start:.2f}s")

if __name__ == '__main__':
    main()
```

### 示例6: asyncio + ThreadPool 混合
```python
import asyncio
from concurrent.futures import ThreadPoolExecutor
import time

def blocking_io_task(task_id: int) -> dict:
    """阻塞型IO任务（如文件读写、数据库查询）"""
    time.sleep(1)  # 模拟IO等待
    return {"task_id": task_id, "status": "completed"}

def blocking_cpu_task(task_id: int) -> dict:
    """阻塞型CPU任务"""
    total = sum(i**2 for i in range(1000000))
    return {"task_id": task_id, "result": total}

async def run_in_executor(executor, func, *args):
    """在执行器中运行阻塞函数"""
    loop = asyncio.get_event_loop()
    return await loop.run_in_executor(executor, func, *args)

async def main():
    """混合异步和多线程示例"""
    io_executor = ThreadPoolExecutor(max_workers=10)
    cpu_executor = ThreadPoolExecutor(max_workers=4)
    
    print("Starting mixed async tasks...")
    
    # IO密集型任务使用线程池
    io_tasks = [
        run_in_executor(io_executor, blocking_io_task, i)
        for i in range(10)
    ]
    
    # CPU密集型任务使用进程池（通过线程池调用）
    cpu_tasks = [
        run_in_executor(cpu_executor, blocking_cpu_task, i)
        for i in range(4)
    ]
    
    start = time.time()
    
    # 并发执行所有任务
    all_results = await asyncio.gather(*io_tasks, *cpu_tasks)
    
    print(f"All tasks completed in {time.time() - start:.2f}s")
    print(f"Total results: {len(all_results)}")
    
    io_executor.shutdown(wait=True)
    cpu_executor.shutdown(wait=True)

if __name__ == '__main__':
    asyncio.run(main())
```

### 示例7: 限流器
```python
import time
import threading
from threading import Lock
from collections import deque

class RateLimiter:
    """滑动窗口限流器"""
    
    def __init__(self, max_calls: int, window_seconds: float):
        self.max_calls = max_calls
        self.window_seconds = window_seconds
        self.calls = deque()
        self.lock = Lock()
    
    def acquire(self) -> bool:
        """尝试获取令牌"""
        with self.lock:
            now = time.time()
            
            # 清理过期的请求记录
            while self.calls and self.calls[0] <= now - self.window_seconds:
                self.calls.popleft()
            
            if len(self.calls) < self.max_calls:
                self.calls.append(now)
                return True
            return False
    
    def wait_and_acquire(self, timeout: float = None) -> bool:
        """等待直到获取令牌或超时"""
        start = time.time()
        while True:
            if self.acquire():
                return True
            if timeout and (time.time() - start) >= timeout:
                return False
            time.sleep(0.01)

def worker(limiter: RateLimiter, worker_id: int, task_count: int):
    """使用限流器的工作线程"""
    for i in range(task_count):
        if limiter.wait_and_acquire(timeout=5):
            print(f"[Worker-{worker_id}] Task-{i} executed at {time.time():.2f}")
            time.sleep(0.1)
        else:
            print(f"[Worker-{worker_id}] Task-{i} timeout!")

def main():
    """限流器示例"""
    # 每秒最多处理5个请求
    limiter = RateLimiter(max_calls=5, window_seconds=1.0)
    
    threads = []
    for i in range(3):
        t = threading.Thread(target=worker, args=(limiter, i, 10))
        threads.append(t)
        t.start()
    
    for t in threads:
        t.join()
    
    print("All workers finished!")

if __name__ == '__main__':
    main()
```
