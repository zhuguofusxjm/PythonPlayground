多进程是一种并发计算的方式，它允许在同一时间内运行多个独立的进程，每个进程都有自己的内存空间和执行上下文。多进程编程通常用于以下情况：
* 利用多核 CPU：多进程可以充分利用多核 CPU，提高计算性能。
* 并行计算：多进程可以同时执行多个计算密集型任务，从而加速计算过程。
* 隔离性：每个进程都有自己的内存空间，因此它们相互隔离，不会相互干扰。
* 可靠性：如果一个进程崩溃，其他进程不受影响，因为它们是独立的。
注：多线程也可以充分利用多核 CPU，但与多进程相比，多线程在 Python 中受到全局解释器锁（Global Interpreter Lock，GIL）的限制。全局解释器锁是 CPython 解释器的特性，它限制了同一时间只能有一个线程执行 Python 字节码。这意味着在多线程的情况下，多个线程无法并行执行 Python 代码。
对于 CPU 密集型任务，由于 GIL 的存在，多线程通常无法显著提高性能，因为多个线程无法并行执行 CPU 密集型任务。在这种情况下，多进程通常更为有效，因为每个进程都有自己的独立解释器和内存空间，不受 GIL 的限制。

Python 提供了多进程编程的支持，其中最常用的是 multiprocessing 模块。以下是一个经典的多进程示例，用于并行计算一组数字的平方：
```pycon
import multiprocessing

def square(number):
    result = number * number
    print(f"The square of {number} is {result}")

if __name__ == "__main__":
    numbers = [1, 2, 3, 4, 5]
    processes = []

    for number in numbers:
        process = multiprocessing.Process(target=square, args=(number,))
        processes.append(process)
        process.start()

    for process in processes:
        process.join()

    print("All processes have finished.")

```

下面给出一个更加接近实用的例子, concurrent.futures.ProcessPoolExecutor 来执行多进程任务
```pycon
import concurrent.futures
from functools import partial
import multiprocessing
import threading
import random
import time

# 示例任务函数

def print_thread_and_process():
    # 生成一个1到5秒之间的随机休眠时间
    random_sleep_time = random.uniform(3, 5)
    time.sleep(random_sleep_time)
    current_thread = threading.current_thread()
    thread_name = current_thread.name
    current_process = multiprocessing.current_process()
    process_name = current_process.name
    print(f"####Executing in thread: {current_thread.ident} and in process: {current_process.ident}")


def example_task1(a, b):
    print_thread_and_process()
    return f"Task executed with parameters a={a} and b={b}, sum={a+b}"
def example_task2(*args, **kwargs):
    '''
    :param args:*args：它用于传递不定数量的非关键字参数（位置参数）。这意味着您可以向函数传递任意数量的参数，它们将被打包成一个元组，供函数内部使用。
    :param kwargs:**kwargs：它用于传递不定数量的关键字参数（键值对参数）。这意味着您可以向函数传递任意数量的关键字参数，它们将被打包成一个字典，供函数内部使用。
    如：
    def example_function(arg1, *args, kwarg1="default", **kwargs):
    print("arg1:", arg1)        --arg1: 1
    print("args:", args)        --args: (2, 3)
    print("kwarg1:", kwarg1)    --kwarg1: custom
    print("kwargs:", kwargs)    --kwargs: {'param1': 'value1', 'param2': 'value2'}
    # 调用函数
    example_function(1, 2, 3, kwarg1="custom", param1="value1", param2="value2")
    :return:
    '''
    print_thread_and_process()
    result = 0
    for arg in args:
        print(f"arg: {arg}")
        result += arg
    for key, value in kwargs.items():
        print(f"Key: {key}, Value: {value}")
        result += value
    return f"Task executed with parameters sum={result}"

class ProcessPoolManager:
    '''
    多进程处理器
    '''
    def __init__(self, max_workers=3):
        self.max_workers = max_workers

    def execute_tasks_in_processpool(self, tasks, timeout=None):
        '''
        通过进程池来处理任务
        :param tasks:
        :param timeout:
        :return:
        '''
        with concurrent.futures.ProcessPoolExecutor(max_workers=self.max_workers) as executor:
            results = [self.execute_task(executor, task, timeout=timeout) for task in tasks]
            return results


    def execute_task(self, executor, task, timeout=None):
        try:
            if timeout:
                future = executor.submit(task)
                result = future.result(timeout=timeout)
            else:
                result = task()
            print(f"Task completed successfully: {result}")
            return result
        except concurrent.futures.TimeoutError:
            print("Task timed out.")
        except Exception as e:
            print(f"Task encountered an exception: {e}")
        return None



if __name__ == "__main__":
    '''
    # functools.partial创建一个带有部分参数绑定的函数，它将 example_task1 函数的前两个参数绑定为 3 和 4
    # def add(a, b):
    #     return a + b
    # # 创建一个偏函数，将参数绑定到任务函数中
    # add_task = partial(add, 1, 2)
    # print(add_task())     ---3
    '''
    work_manager = ProcessPoolManager(max_workers=3)
    tasks = [partial(example_task1, 3, 4), partial(example_task2, 5, 6, 7, param1=100, param2=200)]
    results = work_manager.execute_tasks_in_processpool(tasks, timeout=3)
    for result in results:
        if result is not None:
            print(f"Result: {result}")
```

