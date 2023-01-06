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

# 核心需求点checklist

- 简单的网页通过配置参数可实现抓取（产品可以使用，甚至业务方可以使用）
- 复杂的网页可以自己编写脚本实现，并且后续维护方便
- 支持模拟浏览器抓取，解决javascript的问题
- 支持代理
- 支持数据插入到MQ中
- 支持爬取数据预加工 => 不支持
- 数据去重
- 定时任务
- 数据监控，成功了多少，爬了多少，失败了多少等等
- 尽量易于扩展，二次开发 => 开源


