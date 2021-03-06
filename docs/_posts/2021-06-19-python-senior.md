---
title: Python高级用法汇总
date: 2021-03-24 23:29:53
description: 整理了一下py的一些高级用法，目前主流的web框架，类的魔法方法使用，三大器(生成器、迭代器、装饰器)，列表字典推导式，元编程，多任务编程
categories:
- Python
---
####  1. python主流web框架

##### 1.1 同步框架

​	django, flask, hug

​	实现原理：　wsgi ,  handler(env, start_response)

​    部署方式：　nginx + web服务器(uwsgi/gunicorn) + web app(django, flask, hug)

##### 1.2 异步框架

​	tornado, sanic, fastapi

​    实现原理：　event_loop(asyncio/gevent) + 协程

​	部署方式：　nginx + web服务器(tornado, sanic, fastapi)

#### 2. 魔术方法的使用

##### 2.1  \_\_new_\_,  \_\_init\_\_

```python
class Person(object):
    __instance = None

    def __new__(cls, *args, **kwargs):
        if Person.__instance is None: 
            obj = object.__new__(cls) 
            Person.__instance = obj
        return Person.__instance   # __new__返回对象实例

    def __init__(self):
        print("创建对象后，进行初始化")　　　# __init__ 是对象创建后的初始化操作
```

##### 2.2  \_\_call\_\_   : 可调用对象

```python
class Add(object):
    def __init__(self, a):
        self.a = a
    
    def __call__(b):
        return self.a + b
    
add = Add(1)
add(2)    #   3
```

##### 2.3 \_\_slots\_\_ : [限制实例的属性](https://www.cnblogs.com/zmc940317/p/10284560.html)

```python
# 提升访问速度（member decriptor），　降低内存使用（__dict__）
class A(object): __slots__ = (x, y)
    
a = A()
a.x, a.y = 1, 2
```

##### 2.4  \_\_enter\_\_   \_\_exit\_\_ :上下文管理

```python
class LockType(object):
    def acquire():
        pass
    
    def release():
        pass
    
    def __enter__():
        self.acquire()
    
    def __exit__():
        self.release()
        
with LockType() as lock:
    print(1)
```

##### 2.5  \_\_getitem\_\_,  \_\_setitem\_\_

```python
class Member(object):
    def __init__(self):
        self._mydict = {}
        
    def __getitem__(self, key):
        try:
            return self._mydict[key]
        except:
            return None
        
    def __setitem__(self, key, value):
        self._mydict[key] = value
        
m = Member()
m["name"] = "ming"  # 触发__setitem__
name = m["name"]    # 触发__getitem__
```

####　3. 装饰器

##### 3.0 装饰器实现原理

> 函数作为参数： 高级语言可以将函数作为参数使用

> 闭包：在一个外函数中定义了一个内函数，内函数里运用了外函数的临时变量，并且外函数的返回值是内函数的引用

##### 3.1 不带参数的装饰器

```python
import time

def cal_time(func):
    def wraper(*args, **kwords):
        start = time.time()
        res = func(*args, **kwords)
        print("func cost time:", time.time() - start)
    return wraper

@cal_time
def test():
    print(1111)
```

##### 3.2 带参数的装饰器

```python
def auth(user_id):
    def decrator(func):
        def wraper(*args, **kwords):
            if user_id:
                return func(*args, kwords)
            else:
                return 403
        return wraper
    return decrator
```

##### 3.3 类装饰器

```python
class logger(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print("the function {func}() is running..."\
            .format(func=self.func.__name__))
        return self.func(*args, **kwargs)

@logger
def say(something):
    print("say {}!".format(something))

say("hello")
```

##### 3.4 装饰类的装饰器

```python
def singleton(cls):
    _instance_dict = {}
    def _singleton(*args, **kwords):
        if cls not in _instance_dict:
            _instance_dict[cls] = cls(*args, **kwords):
        return _instance_dict[cls]
    return _singleton

@singleton
class Test(object):
    pass
```

##### 3.5 装饰器的执行时间，执行顺序

```python
@decorator_b
@decorator_a
def test():
    pass
# 装饰器函数在被装饰函数定义好后立即执行从下往上执行 f=decorator_b(decorator_a(f))
# 函数调用时从上到下执行
```

##### 3.6 python内置的常见装饰器

```python
from functools import lru_cache

class A(object):
    
    @classmethod              # 类方法，参数为当前类， 类和实例都可以调用
    def a(cls):
        print("aaa")
        
    @staticmethod             # 静态方法，和当前类和实例都没有关系
    def b():
        print("bbbb")
        
    @property                 # 将方法当做属性来使用
    def name(self):
        return "ming"
    
    @lru_cache                # 内存缓存
    def fab(n):
        return n**2
        
a = A()
A.a()
a.a()
A.b()
a.b()
print(a.name)
```

#### 4. 列表推导式，字典推导式

```python
# 列表推导式
a = [i for i in range(10)]
b = [j for j in a if j > 3]

# 字典推导式
c = {str(index): item for index, item in enumerate(a)}
print("a:", a)    # a: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print("b:", b)    # b: [4, 5, 6, 7, 8, 9]
print("c:", c)    # c: {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}
```

#### 5. 多继承

```python
# 旧式类： 深度优先
# 新式类： 广度优先
class AA(object):
    def toString(s):
        return str(s) * 2
   
class A(AA):
    pass

class B(object):
    def toString(s):
        return str(s) * 3
    
class C(object):
    pass

class D(A, B, C):
    pass

d = D()
d.toString("extend")
# 旧式类调用顺序： D -> A -> AA
# 新式类调用顺序： D -> A -> B
```

#### 6. 获取属性和设置属性

```python
class A(object):
    
    def toString(self, s):
        return str(s)
        
class B(object):
    pass
        
def trans(obj, s):
    if hasattr(obj, "toString"):
        res = getattr(obj, "toString")(s)             # 类似的用法:  eval("{'a': 1, 'b': 2}")  返回传入字符串的表达式的结果
    else:                                             # eval的危害: eval("os.system('rm -rf *')")
        setattr(obj, "toString", lambda s: str(s)*2)
        toString = getattr(obj, "toString")
        res = toString(s)
        
a = A()
b = B()
trans(a, "aa")
trans(b, "bb")
```

#### 7. 元编程

##### 7.1 元类

定义： 创建类的类

使用\_\_class\_\_查看对象是由哪个类创建的

```python
class A(object):
    pass
print("元类：", A.__class__)
# 元类：type
```

type： 创建类的类

type创建类的语法

```python
# type(类名, 由⽗类名称组成的元组（针对继承的情况，可以为空）， 包含属性的字典)
def upper(self):
    return self.name.upper()

B = type("B",(object,),{
    "name": "ming",
    "name_upper": upper
})
b = B()
b.name_upper()
```

##### 7.2  创建一个元类

```python
# metaclass 
class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]


class Application(metaclass=Singleton):  # python3 用法
    pass

#class Application(object):  # python2 用法
#    __metaclass__=Singleton

# 调用顺序： singleton.__new__, singleton.__init__ 创建Application类， 创建Application实例时调用 singleton.__call__
```

#### 8. 多线程编程

##### 8.1 线程的使用场景

使用场景： io密集型任务

原        因： 由于GIL锁的存在，所以同一时间同一个进程内只有一个线程在cpu上执行，所以严格说python多线程只能实现并发，而不是并行

##### 8.2 线程的使用方法

```python
# 线程的基本使用
import time
from threading import Thread, Lock

def test(n):
    time.sleep(n)
    
td = Thread(target=test, args=(5,))
td.start()
td.join()          # 主线程会阻塞，直到td线程执行结束

# 线程常见用法
class Heartbeat(Thread):
    def __init__(self, connection):
        super().__init__()
        self.lock = Lock()
        self.connection = connection
        self.heartbeat_interval = int(getattr(connection, '_impl').params.heartbeat / 2)
        self.setDaemon(True)  # 主进程结束了但该守护线程没有运行完，守护进程就会被强制结束

    def run(self):
        while True:
            time.sleep(self.heartbeat_interval)
            self.lock.acquire()
            try:
                self.connection.process_data_events()
            # 心跳检测连接关闭不记录错误日志，cmsk等客户环境流程是正常的出现很多错误日志
            except ConnectionClosed:
                pass
            except BaseException as ex:
                logging.exception(ex)
            finally:
                self.lock.release()
                
# 线程池用法
from concurrent.futures import ThreadPoolExecutor
import time

def spider(page):
    time.sleep(page)
    print(f"crawl task{page} finished")
    return page

with ThreadPoolExecutor(max_workers=5) as t:  # 创建一个最大容纳数量为5的线程池
    obj_list = []
    for page in range(1, 5):
        obj = t.submit(spider, page)     # 通过submit提交执行的函数到线程池中
        obj_list.append(obj)

    for future in as_completed(obj_list):  # as_completed 当子线程中的任务执行完后，直接用 result() 获取返回结果
        data = future.result()
        print(f"main: {data}")
```

#### 9. 多进程编程

##### 9.1  调用外部程序

```python
# 在cmd中调起外部程序， 阻塞的方式
os.system(command="python test.py")

# 在cmd中调起外部程序， 非阻塞的方式
import subprocess
process = subprocess.Popen(command="python test.py", stderr=subprocess.PIPE)
# 终止进程
# process.kill()	

# communicate： 当前线程阻塞，直到子进程执行结束
stdout, stderr = process.communicate(timeout=172800)
```

##### 9.2  开启子进程执行函数

```python
# 使用场景
# cpu密集型程序

# 和threading使用方式相同
import multiprocessing
import time

def worker(n):
    time.sleep(n)

for i in range(5):
   process = multiprocessing.Process(target=worker, args=(5,))
   process.start()
   record.append(process)
    
for process in record:
   process.join()
```

##### 9.3 进程池

```python
from multiprocessing.pool import ThreadPool

def worker(n):
    time.sleep(n)

# 设置最大并行数
pool = ThreadPool(processes=3)
# apply_async(func,args) 从进程池中取出一个进程执行func，args为func的参数。它将返回一个AsyncResult的对象，你可以对该对象调用get()方法以获得结果。
result = pool.apply_async(worker, args=(5,))

pool.close()       # 进程池不再创建新的进程

pool.join()        # wait进程池中的全部进程。必须对Pool先调用close()方法才能join。
```

##### 9.4 进程间通讯

```python
# pipe  只支持两个进程间传递
import multiprocessing as mul

def proc1(pipe):
    pipe.send('hello')
    print('proc1 rec:', pipe.recv())

def proc2(pipe):
    print('proc2 rec:', pipe.recv())
    pipe.send('hello, too')

# Pipe对象返回一个含有两个元素的表，每个元素代表Pipe的一端。我们对Pipe的某一端调用send()方法来传送对象，在另一端使用recv()来接收。
pipe = mul.Pipe()
if __name__ == '__main__':
    p1 = mul.Process(target=proc1, args=(pipe[0],))
    p2 = mul.Process(target=proc2, args=(pipe[1],))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
    
    
    
# Queue 支持多个进程间传递
import os
import multiprocessing
import time

def inputQ(queue):
    info = str(os.getpid()) + '(put):' + str(time.time())
    queue.put(info)

def outputQ(queue):
    info = queue.get()
    print (str(os.getpid()) + ' get: ' + info)

record1 = [] 
record2 = [] 

queue = multiprocessing.Queue(3)  # 设置队列大小

if __name__ == '__main__':
    for i in range(5):
        process = multiprocessing.Process(target=inputQ,args=(queue,))
        process.start()
        record1.append(process)
    
    for i in range(5):
        process = multiprocessing.Process(target=outputQ,args=(queue,))
        process.start()
        record2.append(process)
    
    for p in record1:
        p.join()
    
    queue.close()  # 不再接受新的数据
    
    for p in record2:
        p.join()
```

#### 10. 生成器和协程

##### 10.1 创建生成器

```python
b = (i for i in range(10))   
print(b)                      # <class 'generator'>

def test(a):
    yield a
print(test(3))                # <generator object test at 0x00000283083CD8E0>
```

##### 10.2 生成器的迭代

````python
# 同其他可迭代对象一样迭代
b = (i for i in range(10)) 
for i in b:
    print(i)
    
# next()方法
def test():
    for i in range(3):
        yield i                # yield返回一个值
b = test()
print(next(b))  # 0
print(next(b))  # 1
print(next(b))  # 2
# print(next(b))  StopIteration error
````

##### 10.3 生成器的控制----协程

```python
# 生成器的send方法
def consumer():
    r = ''
    while True:
        n = yield r   # send(x)  由n接收x的值
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    c.send(None)     # 启动生成器
    # next(c)        # 也可以用next启动生成器
    n = 0
    while n < 2:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()        # 关闭携程

c = consumer()
produce(c)

# [PRODUCER] Producing 1...
# n: 1
# [CONSUMER] Consuming 1...
# [PRODUCER] Consumer return: 200 OK
# [PRODUCER] Producing 2...
# n: 2
# [CONSUMER] Consuming 2...
# [PRODUCER] Consumer return: 200 OK
```

#### 11. 异步编程

##### 11.1 asyncio

###### 11.1.0 asyncio用法

```python
import asyncio

# @asyncio.coroutine 协程预激装饰器
async def hello():               # python3.5以后的新语法 async代替协程预激装饰器， await代替yield from, 以同步的方式写异步程序
    print("Hello world!")
    r = await asyncio.sleep(1)   # 异步调用asyncio.sleep(1):
    print("Hello again!")

# 获取EventLoop:
loop = asyncio.get_event_loop()
# 执行coroutine
loop.run_until_complete(hello())
loop.close()
```

###### 11.1.1 用协程实现单进程并发

```python
# 和线程实现并发的区别

import asyncio
  
import time
  
async def do_some_work(x):
    print('Waiting: ', x)
  
    await asyncio.sleep(x)
    print('Done after {}s'.format(x))
  
start = time.time()
  
coroutine1 = do_some_work(3)
coroutine2 = do_some_work(5)
  
tasks = [
    asyncio.ensure_future(coroutine1),    # ensure_future 可以将 coroutine 封装成 Task
    asyncio.ensure_future(coroutine2),
]
  
loop = asyncio.get_event_loop()
# asyncio.wait方法则返回一个 coroutine。
# run_until_complete 既可以接收 Future 对象，也可以是 coroutine 对象，如果是coroutine,则先把他转化为future
loop.run_until_complete(asyncio.wait(tasks))   # loop.run_until_complete(asyncio.gather(coroutine1, coroutine1))
  
print('TIME: ', time.time() - start)
```

###### 11.1.2 event_loop原理

io多路复用：

​       windows:   select

​       linux       ： epoll

###### 11.1.3 asyncio异步调度原理

```python
# Future 表示未来要完成的事情.
class Future:
    def __init__(self):
        self.result = None
        self._callbacks = []

    def add_done_callback(self, fn):
        self._callbacks.append(fn)

    def set_result(self, result):
        self.result = result
        for fn in self._callbacks:   # set_result会调用call_back
            fn(self)
            
# Task是用来驱动future的
class Task:
    def __init__(self, coro):
        self.coro = coro
        f = Future()
        f.set_result(None)  # 激活协程
        self.step(f)

    def step(self, future):
        try:
            next_future = self.coro.send(future.result)    # 通过send方法来控制协程
        except StopIteration:
            return

        next_future.add_done_callback(self.step)          # callback中就是step方法
```

##### 11.2 gevent

###### 11.2.0 gevent实现并发(基于IO切换的协程)

```python
# 由于切换是在IO操作时自动完成，所以gevent需要修改Python自带的一些标准库，这一过程在启动时通过monkey patch完成：
from gevent import monkey; monkey.patch_all()
import gevent
import urllib2

def f(url):
    print('GET: %s' % url)
    resp = urllib2.urlopen(url)
    data = resp.read()
    print('%d bytes received from %s.' % (len(data), url))

gevent.joinall([                                       # 将协程任务添加到事件循环，接收一个任务列表
        gevent.spawn(f, 'https://www.python.org/'),    # 创建一个普通的Greenlet对象并切换
        gevent.spawn(f, 'https://www.yahoo.com/'),
        gevent.spawn(f, 'https://github.com/'),
])
```

###### 11.2.1 原理

event_loop:   基于libev的事件循环

协程           ： 当处理一个socket链接时，就创建一个协程greenlet(c拓展库实现)去处理，遇到io操作自动切换

##### 11. 3 asyncio和gevent的区别

gevent:  

* python2 需要通过yield + 回调 的方式去实现异步编程，gevent可以使web app以同步的开发方式实现异步编程
* gevent 会在遇到io操作时自动切换，适合无脑开发

asyncio:

* asyncio是在python3.4的时候被加入标准库
* python3.5时支持async  await 语法， 让python可以以同步的方式开发异步程序
* asyncio是由用户自己控制阻塞的时机， 所以需要使用异步的io库才能实现真正的异步
* 目前异步io库还不全，成熟的有：aiohttp, aioredis, aiopika等， 没有成熟且好用的orm(mysql)框架