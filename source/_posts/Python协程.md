---
title: Python 协程
tags:
  - Python
  - 协程
  - Coroutine
categories: Python
date: 2016-10-25 22:02:48
---
# 背景

第一次听同学提到协程 Coroutine，说是一个类似于多线程而又不是多线程的东西，听得云里雾里，不觉明厉。后来找了个机会好好看了一下相关的知识，发现协程是一个很有趣的东西。

[Wiki百科](https://zh.wikipedia.org/wiki/%E5%8D%8F%E7%A8%8B)给协程的定义是：“与子例程一样，协程也是一种程序组件。。。协程更适合于用来实现彼此熟悉的程序组件，如合作式多任务，迭代器，无限列表和管道。”

那什么是子程序呢？[Wiki百科](https://zh.wikipedia.org/wiki/%E5%AD%90%E7%A8%8B%E5%BA%8F)给出的定义是：“在计算机科学中，子程序（英语：Subroutine, procedure, function, routine, method, subprogram, callable unit），是一个大型程序中的某部分代码，由一个或多个语句块组成。它负责完成某项特定任务，而且相较于其他代码，具备相对的独立性。”
<!-- more -->


后来翻阅多方资料，相信我可以把协程讲清楚。

# 概念理解

假如程序中有两种函数，我们把它们命名为 **自私函数**和 **无私函数**。由于CPU资源非常宝贵，每个函数都希望自己能够被执行。

**自私函数**选择对自己最有利的方案，每次都有自己执行完毕(return)才把CPU资源交给别的函数，于是所有的函数按照调用顺序依次执行。

**无私函数**由于学过纳什均衡愿意牺牲自己一部分利益换取群里利益最大化，当然对方必须也是无私函数。因此无私函数可以在自己还没有执行完(not return)的时候把CPU资源交给别人。

比如有这个两个函数

```python
def A():
    print('1')
    print('2')
    print('3')

def B():
    print('x')
    print('y')
    print('z')

#调用函数
A()
B()
```

如果这两个函数都是自私函数，那么调用顺序决定输出结果：

```
1
2
3
x
y
z
```

而如果两个函数都是无私函数，那么在执行A的过程中，可以随时中断，去执行B，B也可能在执行过程中中断再去执行A。可能出现的结果是：

```
1
2
x
y
3
z
```

看起来好像两个函数同时执行了。但是一定要注意，这里只有一个线程。

无私函数之间相互协作完成任务，所以称为“协程”。

无私函数（协程）通过yield来调用其它无私函数（协程）。通过yield方式转移执行权的协程之间不是调用者与被调用者的关系，而是彼此对称、平等的。所以协程在执行过程中可以中断该子程序，去执行其他子程序。

## 协程与子程序

协程与子程序的根本区别是 **执行时期控制权能否转接**。


## 协程与多线程

协程只有一个线程在执行，由于 CPU 速度非常快，所以能达到（看起来）多个任务同时执行的效果。

用小时候非常喜欢看的《龙珠》做比喻就是，协程就是残像拳，悟空不断地在A和B两地移动，速度非常快，看起来就像两个悟空一样；而多线程就是沙鲁分泌出来的小沙鲁，每个小沙鲁都能独立作战。


# 协程的使用

## yield

创建一个斐波那契序列的生成器

```python
def fib(n):
    index = 0
    a, b = 0, 1
    while index < n:
        yield a
        a, b = b, a + b
        index += 1

#使用for in消费这些数据
for i in fib(20):
    print(i)
```

当一个函数中包含 yield 语句时，python 会自动将其识别为一个生成器。这时 fib() 并不会真正调用函数体，而是以函数体生成了一个生成器对象实例。

yield 在这里可以保留fib函数的计算现场（a, b的值），暂停fib的计算并将 b 返回。而将 fib 放入 for…in 循环中时，每次循环都会调用 next(fib())，唤醒生成器，执行到下一个 yield 语句处，直到抛出 StopIteration 异常。此异常会被 for 循环捕获，导致跳出循环。

执行的时候发现速度非常快，而且不会给内存带来很大的压力，因为每一次i的值都是动态生成的，而不需要把它们存储在列表中。更概括的说上面的例子中使用 yield 便可获得了一个协程，协程会消费掉发送给它的值。

## send

从上面的程序中可以看到，目前只有数据从 fib() 中通过 yield 流向外面的 for 循环；如果可以向 fib() 发送数据，那不是就可以在 Python 中实现协程了嘛。

平时写程序的时候总是会遇到一些比较耗时的操作，比如读写文件，读取网络等，所以我们给刚才的 fib() 函数加上一段休眠变成慢速 fib()

```python
import time
import random

def slow_fib(n):
    index = 0
    a = 0
    b = 1
    while index < n:
        sleep_cnt = yield b
        print('let me think %.2f secs' % sleep_cnt)
        time.sleep(sleep_cnt)
        a, b = b, a + b
        index += 1

sfib = slow_fib(20)
fib_res = next(sfib) #sfib.send(None)
while True:
    print(fib_res)
    try:
        fib_res = sfib.send(random.uniform(0, 0.5))
    except StopIteration:
        break
```

其中 next(sfib) 相当于 sfib.send(None)，可以使得sfib运行至第一个 yield 处返回。后续的 sfib.send(random.uniform(0, 0.5))则将一个随机的秒数发送给 sfib，作为当前中断的 yield 表达式的返回值。

于是，Python 中的生成器有了 send 函数，yield 表达式也拥有了返回值。


## grep

Python 实现的 grep 也是一个很好的协程的例子

```python
def grep(pattern):
    print("Searching for", pattern)
    while True:
        line = (yield)
        if pattern in line:
            print(line)
```

调用方式：使用 next() 启动一个协程，协程中包含的生成器并不是立刻执行，而是通过 next() 方法来响应 send() 方法。因此，你必须通过 next() 方法来执行 yield 表达式。

```python
search = grep('coroutine') 
next(search)  #Searching for coroutine
```

使用 send() 向 search 传值,当传入的值中包含 'coroutine' 时，输出传入的值

```python
search.send("I love you")
search.send("Don't you love me?")
search.send("I love coroutine instead!")  #I love coroutine instead!
```

通过 close() 方法来关闭一个协程

```python
search.close()
```

## 生产者消费者 

下面来看一个完整的生产者消费者的例子：

```python
def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'


def produce(c):
    c.send(None)
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()


c = consumer()
produce(c)
```


注意到 consumer 函数是一个 generator，把一个 consumer 传入 produce 后：
1. 首先调用 c.send(None) 启动生成器；
2. 然后，一旦生产了东西，通过 c.send(n) 切换到 consumer 执行；
3. consumer 通过 yield 拿到消息，处理，又通过 yield 把结果传回；
4. produce 拿到 consumer 处理的结果，继续生产下一条消息；
5. produce 决定不生产了，通过 c.close() 关闭 consumer，整个过程结束。

整个流程无锁，由一个线程执行，produce 和 consumer 协作完成任务，所以称为“协程”。

# 总结

每次使用协程都要依赖生成器是不是很麻烦呢？

Python3.5 引入 async/await 让协程表面上独立于生成器而存在，让 Python 写协程更加方便。

学习完成后会更新博客，敬请期待。