# Python - multiprocessing


## Introduction
以往需要用到 multi-processing 時大多都是使用 `concurrent.futures` 來處理  
最近有機會使用到其他的 Library，把常見的幾種方式紀錄下來  

### Task
常見的模式，有個需要重複 N 次的 task， task 內的邏輯都一樣，唯一的差異在於輸入的參數，task 執行時也不相依於其他 task

* example：
    * Input： 長度 100 的 list
    * 使用 10 個 workers， task 處理的部份
    * Output： 預期同樣是長度 100 的 list; 順序可能會變
```graph
   ┌───────────┐                                       ┌───────────┐ ─┐
   │ input 1   │              ┌───────────┐            │ output 3  │  │
   ├───────────┤              │ worker 1  │            ├───────────┤  │
   │ input 2   │              └───────────┘            │ output 1  │  │
   ├───────────┤                                       ├───────────┤  │
   │ input 3   │                                       │ output 4  │  │
   ├───────────┤              ┌───────────┐            ├───────────┤  │
   │ input 4   │              │ worker 2  │            │ output 2  │  │ 100
   ├───────────┤ ──────────► └───────────┘ ─────────►├───────────┤  │
   │ input 5   │                   .                   │ output 10 │  │
   ├───────────┤                   .                   ├───────────┤  │
   │     .     │                   .                   │     .     │  │
   │     .     │              ┌───────────┐            │     .     │  │
   │     .     │              │ worker 10 │            │     .     │  │
   ├───────────┤              └───────────┘            ├───────────┤  │
   │ input 100 │                                       │ output ?  │  │
   └───────────┘                                       └───────────┘ ─┘
```
* sample code:
    * https://gist.github.com/fltermare/d277f6d85d8e0bc09fb792fd3e87d19e

## Preliminary work
```python
from multiprocessing import Process, Queue, Pool
from concurrent import futures
from functools import wraps
import time
import datetime

class ProcTarget():
    @classmethod
    def proc1(cls, idx, input_q, output_q):
        print("Child process #{} started".format(idx))
        while True:
            while input_q.empty():
                time.sleep(0.1)

            i = input_q.get(True)
            if i is None:
                print("Child process #{} ending".format(idx))
                return
            res = cls.proc2(i, i)
            output_q.put(res, True)

    @classmethod
    def proc2(cls, i1, i2):
        time.sleep(5)
        return i1 * i2


def timer(func):
    @wraps(func)
    def wrap(*args, **kwargs):
        time_start = datetime.datetime.now()
        x = func(*args, **kwargs)
        time_end = datetime.datetime.now()

        print("[{}] cost time: {}".format(func.__name__, time_end - time_start))
        print("[{}] return len: {}".format(func.__name__, len(x)))

        return x

    return wrap
```
* `ProcTarget()`: worker 要執行的 function
    * 邏輯：是先睡 5 秒，再把 input 相乘並回傳
    * `ProcTarget.func1`： For `multiprocessing.Process`， 須自行處理取出 input 與回傳 output
    * `ProcTarget.func2`： For `multiprocessing.Pool` & `concurrent.futures`
* `timer()`： python decorator; 計時用

## multiprocessing.Process
```python
@timer
def use_process_queue(inputs, num_proc):
    res = []

    qsize = 100
    input_q = Queue(maxsize=qsize)
    output_q = Queue(maxsize=qsize)

    # create and start child processes
    proc_queue = []
    for i in range(num_proc):
        proc = Process(target=ProcTarget.proc1, args=(i, input_q, output_q))
        proc.start()
        proc_queue.append(proc)

    # put input into input_q; get output and the same time
    for i in inputs:
        while True:
            while True:
                try:
                    out = output_q.get(False)
                    res.append(out)
                except Exception:
                    break

            if not input_q.full():
                input_q.put(i, True)
                break

    # put stop sign (None) into input_q; to terminate child process
    for i in range(num_proc):
        input_q.put(None, True)

    # get remaining output
    while True:
        try:
            out = output_q.get(timeout=30)
            res.append(out)
        except Exception:
            break

    # make sure all child processes have stopped
    for proc in proc_queue:
        proc.join()

    return res
```
* 需要自行管理
    * input queue & output queue
    * put data into input queue & get the result from output queue
    * child processes lifecycle

## multiprocessing.Pool
```python
@timer
def use_mp_pool_sync(inputs, num_proc):
    with Pool(num_proc) as pool:
        res = pool.starmap(ProcTarget.proc2, [(i, i) for i in inputs])

    return res


@timer
def use_mp_pool_async(inputs, num_proc):
    with Pool(num_proc) as pool:
        tmp = pool.starmap_async(ProcTarget.proc2, [(i, i) for i in inputs])
        res = tmp.get()

    return res
```



## concurrent.futures
```python
@timer
def use_future_process(inputs, num_proc):
    res = []
    with futures.ProcessPoolExecutor(max_workers=num_proc) as executor:
        fs = []
        for i in inputs:
            future = executor.submit(ProcTarget.proc2, i, i)
            fs.append(future)

        for future in futures.as_completed(fs):
            tmp = future.result()
            res.append(tmp)

    return res


@timer
def use_future_thread(inputs, num_proc):
    res = []
    with futures.ThreadPoolExecutor(max_workers=num_proc) as executor:
        fs = []
        for i in inputs:
            future = executor.submit(ProcTarget.proc2, i, i)
            fs.append(future)

        for future in futures.as_completed(fs):
            tmp = future.result()
            res.append(tmp)

    return res
```

## Experiment
```python
def main():
    inputs = list(range(100))
    num_proc = 10

    use_process_queue(inputs, num_proc)
    use_mp_pool_sync(inputs, num_proc)
    use_mp_pool_async(inputs, num_proc)
    use_future_process(inputs, num_proc)
    use_future_thread(inputs, num_proc)


if __name__ == "__main__":
    main()
```

```
❯ python3 multi.py
Child process #0 started
Child process #1 started
Child process #2 started
Child process #3 started
Child process #4 started
Child process #5 started
Child process #6 started
Child process #7 started
Child process #8 started
Child process #9 started
Child process #5 ending
Child process #0 ending
Child process #7 ending
Child process #1 ending
Child process #8 ending
Child process #9 ending
Child process #4 ending
Child process #6 ending
Child process #3 ending
Child process #2 ending
[use_process_queue] cost time: 0:01:20.191917
[use_process_queue] return len: 100
[use_mp_pool_sync] cost time: 0:01:00.059515
[use_mp_pool_sync] return len: 100
[use_mp_pool_async] cost time: 0:01:00.060099
[use_mp_pool_async] return len: 100
[use_future_process] cost time: 0:00:50.064544
[use_future_process] return len: 100
[use_future_thread] cost time: 0:00:50.050698
[use_future_thread] return len: 100
```
* 基本上，未來能用 `concurrent.futures` 我還是會優先使用
* `multiprocessing.Process` 相對有彈性，但我不確定我是否有正確使用


## Ref
* https://docs.python.org/3/library/concurrent.futures.html
* https://docs.python.org/3/library/multiprocessing.html
