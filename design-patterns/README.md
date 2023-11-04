并发设计模式是用于处理多线程和多进程编程中的常见问题的通用解决方案。以下是一些常见的并发设计模式，以及与每个模式相关的经典案例：
生产者-消费者模式：
经典案例：一个生产者不断生成数据并将其放入队列，同时一个或多个消费者从队列中获取数据并进行处理。这种模式可用于解决数据生成和处理的异步问题，例如日志记录系统。
简易代码：
```pycon
import threading
import time
import queue

# 共享数据的队列
shared_queue = queue.Queue(5)  # 设置队列的最大容量为5

# 生产者函数
def producer():
    for i in range(1, 11):
        item = f"Item {i}"
        print(f"Producing {item}")
        shared_queue.put(item)
        time.sleep(1)

# 消费者函数
def consumer():
    while True:
        item = shared_queue.get()
        if item is None:
            break
        print(f"Consuming {item}")
        shared_queue.task_done()

if __name__ == "__main":
    # 创建生产者和消费者线程
    producer_thread = threading.Thread(target=producer)
    consumer_thread = threading.Thread(target=consumer)

    # 启动线程
    producer_thread.start()
    consumer_thread.start()

    # 等待生产者线程完成
    producer_thread.join()

    # 向队列发送结束信号以停止消费者线程
    shared_queue.put(None)
    consumer_thread.join()
```

稍微探讨一下，多线程/进程/单进程消费有什么区别，有个非常有趣的实验：
```pycon
import threading
import time
import queue
import multiprocessing
import concurrent.futures
import random

# 共享数据的队列
shared_queue = multiprocessing.Queue(5)  # 设置队列的最大容量为5

# 存放处理结果的队列
resultQueue = multiprocessing.Queue()

# 定义一个装饰器函数，用于测量函数执行时间
def timing_decorator(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        execution_time = end_time - start_time
        print(f"{func.__name__} 执行时间: {execution_time} 秒")
        return result
    return wrapper

def print_thread_and_process():
    current_thread = threading.current_thread()
    current_process = multiprocessing.current_process()
    process_name = current_process.name
    print(f"####Executing in thread: {current_thread.ident} and in process: {current_process.ident}")

# 生产者函数
def producer():
    print_thread_and_process()
    for i in range(1, 10):
        item = f"Item {i}"
        print(f"生产 Item {i}")
        shared_queue.put(item)
        # time.sleep(1)

# 处理函数
def process_item(item):
    print_thread_and_process()
    print(f"》》》消费 {item}")
    result = f"》》》消费 {item}"
    # 生成一个1到5秒之间的随机休眠时间
    # random_sleep_time = random.uniform(0.2, 2)
    # time.sleep(random_sleep_time)
    time.sleep(0.5)
    resultQueue.put(result)

# 消费者函数
@timing_decorator
def consumer():
    # 多进程消费
    # consumer 执行时间: 1.776261329650879 秒
    with concurrent.futures.ProcessPoolExecutor(max_workers=3) as executor:
        while True:
            item = shared_queue.get()
            if item is None:
                break
            executor.submit(process_item, item)

    # 多线程消费
    # consumer 执行时间: 1.508080005645752 秒
    # with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    #     while True:
    #         item = shared_queue.get()
    #         if item is None:
    #             break
    #         executor.submit(process_item, item)

    # 单线程消费
    # consumer 执行时间: 4.5048298835754395 秒
    # while True:
    #     print_thread_and_process()
    #     item = shared_queue.get()
    #     if item is None:
    #         break
    #     result = f"Processed {item}"
    #     print(f"》》》消费 {item}")
    #     time.sleep(0.5)
    #     resultQueue.put(result)

if __name__ == "__main__":
    producer_thread = threading.Thread(target=producer)
    consumer_thread = threading.Thread(target=consumer)

    # 启动线程
    producer_thread.start()
    time.sleep(5)
    consumer_thread.start()

    # 等待生产者线程完成
    producer_thread.join()

    # 向队列发送结束信号以停止消费者线程
    shared_queue.put(None)
    consumer_thread.join()

    # 获取处理结果
    while not resultQueue.empty():
        result = resultQueue.get()
        print(result)
```
-------------------------------------------------------------
观察者模式：
经典案例：多个观察者监视一个主题对象，当主题对象状态发生变化时，观察者会收到通知并执行相应操作。这种模式用于实现事件驱动编程，例如 GUI 应用程序中的事件处理。
-------------------------------------------------------------
互斥锁模式：
经典案例：多个线程需要竞争访问共享资源，使用互斥锁来确保在任何给定时刻只有一个线程可以访问共享资源，以防止竞争条件。这是在多线程编程中非常常见的模式。
-------------------------------------------------------------
线程池模式：
经典案例：为了减少线程创建和销毁的开销，使用线程池来维护一组可重复使用的线程，以便执行任务队列中的任务。这提高了性能和资源利用率。
-------------------------------------------------------------
工作队列模式：
经典案例：将任务放入队列中，然后多个工作者线程从队列中获取任务并执行。这用于实现任务的异步执行，例如 Web 服务器中的请求处理。
-------------------------------------------------------------
信号量模式：
经典案例：使用信号量来控制并发线程的数量，以确保系统资源得到合理分配。这在限制并发连接数或资源池管理中很有用。
-------------------------------------------------------------
多路复用模式：
经典案例：使用多路复用技术，例如 epoll 或 select，来管理多个 I/O 通道的事件和状态，以实现高效的非阻塞 I/O 操作。