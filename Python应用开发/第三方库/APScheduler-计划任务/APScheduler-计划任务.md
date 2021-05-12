# APScheduler 计划任务

实现一个类似定时执行脚本的功能，最简单的方式是使用操作系统提供的计划任务，比如`crontab`。但如果计划任务较多，任务之间的关系复杂，很显然使用这种方式会非常难以维护。

Python中，`APScheduler`这个模块能够更容易的实现该功能。

[https://apscheduler.readthedocs.io/en/stable/](https://apscheduler.readthedocs.io/en/stable/)

## 安装

```
pip3 install apscheduler
```

## 基本概念

1. 触发器：用于根据调度规则触发计划任务，支持间隔触发、按日期时间触发、cron表达式触发
2. 任务存储器：储存任务执行情况的组件，支持储存在内存或数据库中，默认未内存
3. 执行器：安排任务到线程池或进程池的组件，默认为线程池
4. 调度器：根据执行计划，协调上述组件执行任务

实际开发中，我们直接实例化一个调度器，然后将配置好任务，调度器就能自动按照我们的计划执行了。

## 定时任务例子

下面代码中，我们每隔2秒打印一条信息，信息中包含当前的线程ID。这里任务存储器和执行器都选择了默认值，调度器使用的是`BlockingScheduler`，它会阻塞主线程阻止程序返回（另一个常用的调度器是不会阻塞的`BackgroundScheduler`）。

```python
import threading
from apscheduler.schedulers.blocking import BlockingScheduler


def foo():
    print('计划任务执行 线程ID %d' % threading.current_thread().ident)


if __name__ == '__main__':
    print('计划任务开始 主线程ID %d' % threading.current_thread().ident)

    # 创建调度器
    scheduler = BlockingScheduler()
    # 添加任务
    scheduler.add_job(foo, 'interval', seconds=2)
    # 开启计划任务
    scheduler.start()
```

`scheduler.add_job()`方法中，第一个参数是要调用的函数；第二个参数是触发方式，这里我们采用间隔触发；第三个参数指定间隔2秒触发。

输出：
```
计划任务开始 主线程ID 12668
计划任务执行 线程ID 21936
计划任务执行 线程ID 21936
计划任务执行 线程ID 21536
计划任务执行 线程ID 21936
计划任务执行 线程ID 21536
...
```

## 任务触发方式

我们可以使用三种方式触发任务：间隔触发、按日期时间触发、cron表达式触发。

`interval`间隔触发：

```
scheduler.add_job(foo, 'interval', seconds=2)
```

`date`按日期时间触发：

```
scheduler.add_job(foo, 'date', run_date='2020-01-01 12:00:00')
```

`cron`按Linux风格的cron表达式触发：

```
scheduler.add_job(foo, CronTrigger.from_crontab('* * * * *'))
```

三种方式都比较简单，我们选择一种最适合的就可以了。

## 任务存储器配置

APScheduler支持以下几种任务保存方式：

* 内存
* MongoDB
* Redis
* ZooKeeper
* （基于SQLAlchemy）关系型数据库

默认情况下，任务是存储在内存中的，这适合比较简单的功能，我们的任务在每次程序启动时，都需要通过`add_job`方法添加任务。但如果情况更复杂一些，比如需要将任务保存在文件或数据库中，我们的程序能对其进行增删改查、配置、启动或停止，这种情况存储在内存中就不合适了。

下面例子配置使用Redis作为任务存储数据库：

```python
jobstores = {
    'default': RedisJobStore(db=0, host='127.0.0.1', port=6379)
}
scheduler = BlockingScheduler(jobstores=jobstores)
```

## 任务数据操作

前面我们只使用了一次性的添加任务，这里我们基于非内存的任务存储，介绍下任务的添加、删除、查询等操作。

### 添加任务

```python
scheduler.add_job(foo, 'interval', seconds=1, id='testjob', replace_existing=True)
```

这里我们指定了两个新参数：

* `id`：任务名，后续删除等操作需要用到
* `replace_existing`：覆盖同名已存在的任务，这里建议指定上，id的唯一性则由我们程序逻辑保证

### 删除任务

```python
scheduler.remove_job('testjob')
```

删除任务比较简单，直接使用任务id即可删除。

### 列出任务

```
jobs = scheduler.get_jobs()
```

`scheduler.get_jobs()`返回当前存储器中已经部署的任务。

### 暂停任务

```python
scheduler.pause_job('testjob')
```

### 恢复任务

```python
scheduler.resume_job('testjob')
```

## 在Django中的使用

因为Django这种类似CGI模式运行程序的特殊性，不建议试图将APScheduler整合进Django。建议单独部署APScheduler作为一个服务，并通过RPC方式与应用程序进行互操作。
