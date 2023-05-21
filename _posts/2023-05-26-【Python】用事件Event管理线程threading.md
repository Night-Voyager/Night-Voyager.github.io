---
layout: post
title: 【Python】用事件Event管理线程threading
# author: 焜_8899
date: 2023-05-26 11:40:40 +0800
tags: [Python]
toc:  true
---

## 0. 背景

写了段涉及多线程的程序，但并不想要个别线程一直执行。希望能在有需要的时候唤醒它们，不需要的时候让它们阻塞。感觉`事件`比较满足想要的效果。

> ## [事件对象](https://docs.python.org/zh-cn/3/library/threading.html#event-objects "永久链接至标题")
> 这是线程之间通信的最简单机制之一：一个线程发出事件信号，而其他线程等待该信号。

## 1. 事件

[官方文档](https://docs.python.org/zh-cn/3/library/threading.html#event-objects)对事件对象的解释清晰明了：
> 一个事件对象管理一个内部标识，调用 [`set()`](https://docs.python.org/zh-cn/3/library/threading.html#threading.Event.set "threading.Event.set") 方法可将其设置为 true ，调用 [`clear()`](https://docs.python.org/zh-cn/3/library/threading.html#threading.Event.clear "threading.Event.clear") 方法可将其设置为 false ，调用 [`wait()`](https://docs.python.org/zh-cn/3/library/threading.html#threading.Event.wait "threading.Event.wait") 方法将进入阻塞直到标识为 true 。

更为详细的内容请移步[官方文档](https://docs.python.org/zh-cn/3/library/threading.html#event-objects)。

## 2. 用法

### 2.1 代码示例

以下代码演示了用事件管理线程的一种方式。其中`event.set()`和`event.clear()`可以被用在需要触发相关事件的地方，例如按钮。

```python
import threading
import time

event = threading.Event()  # 实例化事件对象


def print_1():  # 第一个线程的调用对象
    while True:
        event.wait()  # 阻塞线程直到被唤醒
        print(1)
        time.sleep(1)


def print_2():  # 第二个线程的调用对象
    while True:
        print(2)
        time.sleep(1)


if __name__ == '__main__':
    # 开始线程活动
    threading.Thread(target=print_1).start()
    threading.Thread(target=print_2).start()

    while True:
        print('event.set()')
        event.set()  # 唤醒被阻塞的线程
        time.sleep(5)

        print('event.clear()')
        event.clear()  # 阻塞线程
        time.sleep(5)

```

### 2.2 运行效果

以下为上述代码的运行效果。可以看到，每过5秒，输出`1`的线程会被阻塞或唤醒一次。

```
2
event.set()
1
12

12

12

12

event.clear()
2
2
2
2
2
event.set()
1
2
1
2
1
2
1
2
1
2
event.clear()
2
2
2
2
2
event.set()
1
2
1
2
1
2
1
2
1
2
event.clear()
2
2
2
2
2
event.set()
1
2
1
2
```
