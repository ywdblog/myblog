在《【意译】Python3中的线程，GIL，线程安全（上）》文章中描述了一些概念，本文从实践的角度去理解 Python 多线程编程。

在使用例子讲解前，再一次提醒如何让你的代码线程安全：

- 一些线程在读数据的时候，没有其他线程在写。
- 一个线程在写数据的时候，其他线程没有读。

### 线程安全实践 

1：没有共享易变的状态，安全

```
import threading

def print_number():
    number = 42
    print(threading.current_thread().name, number)

t1 = threading.Thread(target=print_number)
t2 = threading.Thread(target=print_number)
t1.start()
t2.start()
```

每个线程中，number 都是局部变量，不会共享数据，所以代码是安全的。

2：共享不可变状态，安全

```
import threading

number = 42

def print_number():
    print(threading.current_thread().name, number)
```

在这个代码片段中，所有线程都读取同一个数据，但代码是安全的，因为没有线程会修改 number 变量。

3：共享可变状态，非常不安全

```
import threading

number = 42

def print_number():
    global number
    number += 1
    print(threading.current_thread().name, number) 
```

在这个例子中，number 变量被所有线程在同一时刻更新，这个操作是不安全的。

如果你在 CPython 中运行这段代码，可能永远不会得到正确的结果。

该代码反映了一个问题：线程会竞争。

4：避免竞争

threading 模块有很多特性用于保护数据竞争，最常见的就是对数据加锁，锁能够确保在同一时间仅有一个线程读写数据（锁不关心读或写）。

```
data_lock = threading.Lock()
data = b'Hello' * 1000

def hash_data():
    global data
    with data_lock:
        data = sha512(data).hexdigest().encode() 
```

data_lock 是一个锁实例，通过 with 上下文获取锁（也会自动释放），确保同一时刻只会运行一个 sha512 函数。

这个例子其实没有什么用，相比单线程代码没有任何的优势，反而因为锁导致性能出现下降。

### 多线程通用解决方案

1：APIs

前面例子避免了数据竞争的情况，虽然可以工作，但不能作为 APIs 中的一部分，对于 API 来说，让程序员在获取更新数据之前先加锁是非常别扭的。（潜台词就是想优雅使用 threading 库并不容易，我们期待有更好的方式编写多线程代码，让开发者根本意识不到锁）。

一个解决方案就是封装锁操作，让锁对程序员不可见，Python 标准模块 logging 就是其中一个典型的例子。

```
class DataHasher:

    def __init__(self, data):
        self._data = data
        self._lock = threading.Lock()

    def hash(self):
        with self._lock:
            self._data = sha512(self._data).hexdigest().encode()

    def get(self):
        with self._lock:
            return self._data 

data = DataHasher(b'Hello' * 1000)
for i in range(10):
    threading.Thread(target=data.hash).start()
```

扩展一下，如果 get 函数处理的是一个可变对象，API 和程序员必须自行处理，要么是不去修改这个可变对象，或者进行一个深度拷贝，比如：

```
def get(self):
    from copy import deepcopy
    with self._lock:
        return deepcopy(self._data) 
```

Python 中对象面向引用的，建议不要使用深度拷贝，因为某些对象（比如套接字）是不可拷贝的，最重要的是会带来性能损耗。

2：生产和消费模式

这种模式在 Python 中非常常见，Python 标准库 queue 模块就是这种模式。

```
import threading
import queue
import time

q = queue.Queue()

def append_to_file():
    while True:
        to_write = q.get()
        with open('/tmp/log', 'a') as f:
            f.write(to_write + '\n')

def produce_something():
    for i in range(5):
        q.put(threading.current_thread().name)
        time.sleep(1)

threading.Thread(target=append_to_file, daemon=True).start()
threading.Thread(target=produce_something).start()
threading.Thread(target=produce_something).start()
```

daemon=True 可以确保消费者完成任务后，主程序就自动退出（想想会不会有什么问题）。另外有了这个模块，你是不是完全接触不到锁了？这就是 API 的作用。

3：Poison pills 

当非 daemon 线程退出的时候，daemon 线程会立刻终止（参考上个例子），这意味着 daemon 线程在退出之前不会进行一些清理工作，从而导致问题（比如没有写到 /tmp/file 文件中）。

有两种方法解决这个问题：

- 让消费者定期检查有没有特定事件发生。
- 发送一个 poison pill 给队列，当消费者发现 poison pill 就停止。

```
import threading
import queue
import time

q = queue.Queue()
must_stop = threading.Event()

def append_to_file():
    with open('/tmp/log', 'a') as f:
        while True:
            data = q.get()
            if data is must_stop:
                print('Consumer received the poison pill, stopping')
                return
            f.write(data + '\n')
            f.flush()

def produce_something():
    for i in range(5):
        q.put(threading.current_thread().name)
        time.sleep(1)

threading.Thread(target=append_to_file).start()
p1 = threading.Thread(target=produce_something)
p1.start()
p1.join()
q.put(must_stop)
```

4：线程池 

当一个程序有很多小任务的时候，频繁创建一个线程也非常昂贵，所以会使用一些技术来重用线程，依次节省 CPU 切换，这种工作机制就是线程池。

一般情况下不是让你手动管理线程，Python 标准库提供了一个很容易使用的实现，这个概念非常简单，线程池启动 N 个线程，每个线程处理接收一个任务，然后处理它，不断的重复。

--- 

欢迎关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳）和书《深入浅出HTTPS：从原理到实战》。