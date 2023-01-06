---
author: zshuaibin
title: crawlab爬虫框架预研
date: 2021-06-01T16:11:37.390Z
description: Guide to emoji usage in Hugo
draft: true
hideToc: false
enableToc: true
enableTocContent: false
tags:
  - spider
  - crawlab
---

# 起因

* 查看Sidekiq的任务日志，发现有许多任务有开始的输出，没有结束的输出，但是在Busy/Enqueued/Retries/Dead中都找不到
* 线上定时任务不能稳定运行，经常漏跑或者跑一半就结束

# Sidekiq存在的问题

## 做个小实验

```ruby
class LongRunJob < ApplicationJob
  def perform(num)
    loop do
      sleep 5
      puts "#{rand(999)}"
    end
  end
end

LongRunJob.perform_later(rand(9999))
```

来看一下在redis中存储的信息

```redis-cli
> llen 'queue:default'
(integer) 1

> lindex 'queue:default' 0
"{\"retry\":true,\"queue\":\"default\",\"class\":\"ActiveJob::QueueAdapters::SidekiqAdapter::JobWrapper\",\"wrapped\":\"LongRunJob\",\"args\":[{\"job_class\":\"LongRunJob\",\"job_id\":\"e07a1872-5f88-4a27-922a-76714fef5e81\",\"provider_job_id\":null,\"queue_name\":\"default\",\"priority\":null,\"arguments\":[3721],\"executions\":0,\"exception_executions\":{},\"locale\":\"en\",\"timezone\":\"UTC\",\"enqueued_at\":\"2021-06-28T16:30:04Z\"}],\"jid\":\"d28767c0407f997c5d8072cb\",\"created_at\":1624897804.59089,\"enqueued_at\":1624897804.591159}"
```

可以看出我们的一个任务在sidekiq中是存储在redis中的key`queue:default`，并且存储的类型是`list`

ok，接下来我们开始运行这个job。

```bash
bundle exec sidekiq
```

再来看看redis中的任务：

```
127.0.0.1:6379> lindex 'queue:default' 0
(nil)
```

如果观察redis中的key，我们发现多了一个：
```
127.0.0.1:6379> hgetall 'rccdeMBP-4:28169:03388a8b7747:workers'
1) "ovzcttgt9"
2) "{\"queue\":\"default\",\"payload\":\"{\\\"retry\\\":true,\\\"queue\\\":\\\"default\\\",\\\"class\\\":\\\"ActiveJob::QueueAdapters::SidekiqAdapter::JobWrapper\\\",\\\"wrapped\\\":\\\"LongRunJob\\\",\\\"args\\\":[{\\\"job_class\\\":\\\"LongRunJob\\\",\\\"job_id\\\":\\\"e07a1872-5f88-4a27-922a-76714fef5e81\\\",\\\"provider_job_id\\\":null,\\\"queue_name\\\":\\\"default\\\",\\\"priority\\\":null,\\\"arguments\\\":[3721],\\\"executions\\\":0,\\\"exception_executions\\\":{},\\\"locale\\\":\\\"en\\\",\\\"timezone\\\":\\\"UTC\\\",\\\"enqueued_at\\\":\\\"2021-06-28T16:30:04Z\\\"}],\\\"jid\\\":\\\"d28767c0407f997c5d8072cb\\\",\\\"created_at\\\":1624897804.59089,\\\"enqueued_at\\\":1624897804.591159}\",\"run_at\":1624898683}"
```

如果这时候，我们终止sidekiq会如何？我们按下`CTRL-C`，观察结果，我们发现正在运行的任务会放回到队列中，并且任务的参数不会改变。

符合预期！

接下来，如果我们强制杀死进程会怎么样？

```
kill -9 $pid
```

我们发现，正在运行的任务，不会放回队列中，并且当前的状态依旧是`Busy`：

![[DEVELOPMENT]Sidekiq2021-06-2900-58-39](https://static-1255384104.cos.ap-shanghai.myqcloud.com/uPic/[DEVELOPMENT] Sidekiq 2021-06-29 00-58-39.png)

再次启动sidekiq，显示`Busy`的任务被清空，因为`进程ID`已经被改变，sidekiq清除了所有不是当前`进程ID`的任务。

![[DEVELOPMENT]Sidekiq2021-06-2900-59-31](https://static-1255384104.cos.ap-shanghai.myqcloud.com/uPic/[DEVELOPMENT] Sidekiq 2021-06-29 00-59-31.png)

## 结论

* 不同的Server连接同一个redis的同一个namespace，会共享同一个任务队列
* 不同的Client启动，会生成独立的Process
* 如果队列中的任务多，并且没有消费的话，会吃掉许多的redis内存
* 抛开Redis本身的数据一致性问题，Sidekiq进程如果死掉，有可能丢失任务

# 使用RabbitMQ来代替Redis，Sneakers来代替Sidekiq

```ruby
require 'bunny'

conn = Bunny.new
conn.start

ch = conn.create_channel
ch.default_exchange.publish(rand(9999), routing_key: 'default')
```

```ruby
class LongRunSneakerJob
  include Sneakers::Worker
  from_queue :default
  
  def work(num)
    loop do
      sleep 5
      puts "#{rand(999)}"
    end
    # 最关键的地方
    ack!
  end
end

bundle exec sneakers work LongRunSneakerJob --require app/jobs/long_run_sneaker_job.rb
```

![nkWrmD](https://static-1255384104.cos.ap-shanghai.myqcloud.com/uPic/nkWrmD.jpg)

## 距离生产环境还需要解决

* 定时任务
* 兼容`ApplicationJob`
* 更方便的启动

## 我们还可以获得更多的优势！

![keqgor](https://static-1255384104.cos.ap-shanghai.myqcloud.com/uPic/keqgor.jpg)

定时任务统一通过配置中心来管理

### 参考资料

https://blog.stanko.io/rabbitmq-is-more-than-a-sidekiq-replacement-b730d8176fb
https://redis.io/topics/persistence



