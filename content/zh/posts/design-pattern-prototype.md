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

# 原型模式

复制已有对象, 而又无需使代码依赖它们所属的类。

## 使用场景

![174frg](https://static-1255384104.cos.ap-shanghai.myqcloud.com/uPic/174frg.jpg)

## 具体应用

### 模拟场景

```ruby
#
# 游戏中有许多的怪物
# 有些行为是一致的
# 有些又不一致，例如外观，血量，速度等等
#
# 
class Monster
  def run; end
  def eat; end
  def beat; end
  # ...
end

#########################################################
# 不使用原型模式
#########################################################
class Ghost < Monster
end

class Demon < Monster
end

class Sorcerer < Monster
end

class MonsterFactory
  #
  # 一个工厂方法，随机生成大量的怪物
  #
  def self.generate
    arr = []
    rand(999).times do
      arr << [Ghost, Demon, Sorcerer].sample.new
    end
    arr
  end
end

#########################################################
# 使用原型模式
#########################################################
class MonsterPrototype
  def initialize
  end
end

class Monster
  def initialize
    @type = 'Monster'
    @prototype = MonsterPrototype.new
  end

  # 省略已有方法......

  # 
  # 避免和ruby自带的 +clone+ 重复
  def _clone(type)
    @type = type
    @prototype = @prototype.clone
    # 可以针对 @prototype 做一些处理
    self.clone
  end
end

class MonsterFactory
  #
  # 一个工厂方法，随机生成大量的怪物
  #
  def self.generate
    @monster = Monster.new
    arr = []
    rand(999).times do
      arr << @monster._clone(['Ghost', 'Demon', 'Sorcerer'].sample)
    end
    arr
  end
end
```

## 优缺点

### 优点

* 克隆对象，无需与它们所属的具体类相耦合
* 性能相比实例化对象要高
* 复杂对象生成简单

### 缺点

* 内部的细节被隐藏，容易产生Bug

## 参考链接

* https://refactoringguru.cn/design-patterns/prototype

* https://gpp.tkchu.me/prototype.html

## `dup` vs `clone`

```
###
# singleton class
###
a = Object.new
def a.foo
  :foo
end

b = a.dup # => undefined method error
c = a.clone # => :foo

###
# frozen state
###
class Foo
  attr_accessor :bar
end
o = Foo.new
o.freeze

o.dup.bar = 10 # => 10
o.clone.bar = 10 # => raises runtimeError

###
# share the same reference
###
class Foo
  attr_accessor :bar
  def initialize
    self.bar = [1,2,3]
  end
end

a = Foo.new
b = a.clone
a.bar.clear
b.bar # => []
```


