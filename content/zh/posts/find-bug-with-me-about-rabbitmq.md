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

# 一起找Bug

## 代码展示

```ruby
# RabbitMQ消息发送
class Api::MessagePublisher

  def self.publish(message = {}, retry_count = 0)
    begin
      return if retry_count >= 2

      exchange = channel.topic("bid_info.special_solr", :durable => true, :auto_delete => false)
      message[:timestamp] = Time.now.strftime("%Y-%m-%d %H:%M:%S")
      exchange.publish(message.to_json, :persistent=>true, :routing_key => "bid_info.special_solr_key")
      write_log ["Success Send", message.inspect].join(' ')
    rescue => e
      @connection = nil
      write_log "#{e.message}"
      write_log ["Need retry", message.inspect].join(' ')
      # 对于重试的放入到Redis中，避免数据丢失
      Api::RmqRedisClient.queue_push(:sync_solr_bid_ids,{"ids"=>message["ids"]})
      # 增加重试机制
      self.publish(message, retry_count + 1) 
    end
  end

  def self.channel
    @channel ||= connection.create_channel
  end

  def self.connection
    @connection ||= Bunny.new.tap do |c|
      c.start
    end
  end

  def self.write_log text
    logger = Logger.new Rails.root.to_s + '/log/api_message_publish.log'
    logger.info text
  end

end
```

存在什么问题呢？

* 重试机制

线上的问题

![screenshot-2](https://static-1255384104.cos.ap-shanghai.myqcloud.com/uPic/screenshot-2.png)

## 思路

怀疑是`RabbitMQ`重启导致的

### 先复现一下Bug

```ruby
require 'bunny'

module Api
  class MessagePublisher
    def self.publish
      msg = "Hello, everybody! #{rand(9999)}"
      puts msg
      q = channel.queue('test1')
      q.publish(msg)
    end

    def self.channel
      @channel ||= connection.create_channel
    end

    def self.connection
      @connection ||= Bunny.new.tap do |c|
        c.start
      end
    end
  end
end

loop do
  sleep 0.5
  Api::MessagePublisher.publish
end

```

反馈的错误

```
Connection-level error: CONNECTION_FORCED - broker forced connection closure with reason 'shutdown' (Bunny::ConnectionForced)
```

## 解决它!

思路：

* 建立连接池
* 增加重试机制



##### 
