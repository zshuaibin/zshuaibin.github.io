---
author: zshuaibin
title: ruby3.0尝鲜记
date: 2021-06-01T16:11:37.390Z
description: Guide to emoji usage in Hugo
draft: true
hideToc: false
enableToc: true
enableTocContent: false
tags:
  - ruby
---

# 回顾下ruby2.7的一些特性

## 模式匹配（Pattern matching）

函数式编程中，模式匹配非常常见，可以帮助我们快速的进行数据结构检查以及变量绑定。

基本语法：
```
# 多行语法
case <expression>
in <pattern1>
  ...
in <pattern2>
  ...
in <pattern3>
  ...
else
  ...
end

# 单行语法
# 返回true or false
<expression> in <pattern>

# 单行语法ruby3.0+
# match and unpack
<expression> => <pattern>
```

简单的例子：
```ruby
# 给定如下的json数据
# 用户名显示规则如下：
# 
# * 如果username存在，返回username
# * 如果nickname，fullname等于字符串，返回nickname（fullname）
# * 如果nickname不存在，fullname等于字符串，返回fullname
# * 如果nickname，firstname，lastname存在，返回nickname（firstname.lastname）
# * 如果firstname，lastname存在，但是nickname不存在，返回firstname lastname
# * 都不满足，返回字符串'Guest'
data = {
  username: 'alex.zang',
  nickname: 'alex',
  fullname: {
    firstname: 'alex',
    lastname: 'zang'
  }
}

# 传统方式
def func1(data)
  fullname = data[:fullname].is_a?(String) ? data[:fullname] : [data[:fullname][:firstname], data[:fullname][:lastname]].join('.') # 这里存在类型风险
  if data[:username]
    data[:username]
  elsif data[:nickname] && fullname.present?
    "#{data[:nickname]}（#{fullname}）"
  elsif data[:fullname].is_a?(Hash) && data[:fullname][:firstname].present? && data[:fullname][:lastname].present?
    "#{data[:fullname][:firstname]} #{data[:fullname][:lastname]}"
  else
    'Guest'
  end
end

# Pattern matching
def func2(data)
  case data
    in {username: }
      username
    in {nickname: , fullname: String => fullname}
      "#{nickname}（#{fullname}）"
    in {fullname: String => fullname}
      fullname
    in {nickname: , fullname: {firstname: , lastname: }}
      "#{nickname}（#{firstname}.#{lastname}）"
    in {fullname: {firstname: , lastname: }}
      "#{firstname} #{lastname}"
    else
      'Guest'
  end
end

```

### Value pattern

```ruby
# 类似右侧赋值
'foo' => a

# 类似===
2 in 1..3
```

### Array pattern

形如：`[<subpattern>, <subpattern>, <subpattern>, ...]`

数组必须完全匹配

```ruby
[1,2,3] in [Integer, Integer]
# => false

[1,2,3] in [Integer, Integer, Integer]
# => true

[1,2,3] in [a, b, c]
# => true
# a => 1
# b => 2
# c => 3

[1,2,3] in [a,b,String => c]
# => false

[1,2,3] in [1,2,a]
# => true
[1,2,3] in [1,3,a]
# => false
[1,2,3] in [1,_,a]
# => true

[1,2,3] in [1,*a]
# => true
# a => [2,3]

[1, [2, 3, 4]] in [a, [b, *c]]
# => true
# a => 1
# b => 2
# c => [3,4]

```

### Hash pattern

形如：`{key: <subpattern>, key: <subpattern>, ...}`

不需要完全匹配

```ruby
{a: 1, b: 2, c: 3} => {a: }
# a => 1

{a: 1, b: 2, c: 3} => {a: Integer => v}
# v => 1

{a: 1, b: 2, c: 3} => {a: Integer => v, **nil}
# raise NoMatchingPatternError
{a: 1} => {a: Integer => v, **nil}
# => true

{a: 1, b: 2, c: 3} => {a: , **opts}
# opts => {:b=>2, :c=>3}

{a: 1, b: 2, c: [{d: 3}, {e: 4}]} => {a: , c: [{d:}, *]}
# a => 1
# d => 3

```

### Find pattern

形如：`[*variable, <subpattern>, <subpattern>, <subpattern>, ..., *variable]`

```ruby
users = [
  {name: 'John', role: 'user'},
  {name: 'Jane', role: 'manager'},
  {name: 'Barb', role: 'admin'},
  {name: 'Dave', role: 'manager'}
]

# 找出其中admin的名称
users => [*, {name: name, role: 'admin'}, *]
# name => Barb
```

### 参考资料

* https://docs.ruby-lang.org/en/3.0.0/doc/syntax/pattern_matching_rdoc.html
* https://rubyreferences.github.io/rubychanges/2.7.html
* https://rubyreferences.github.io/rubychanges/3.0.html

## 参数数字化（Numbered block parameters）

```ruby
filenames.each { |f| File.read(f) }

# 更加DRY
filenames.each { File.read(_1) }

[10, 20, 30].map { _1**2 }
# => [100, 400, 900]
```

## Argument Forwarding

```ruby
def perform(*args, **kws, &block)
  block.call(args, kws)
end

# 不需要alias的情况下
def call(...)
  perform(...)
end

# ruby3.0支持部分Forwarding
def request(method, url, headers: {})
  puts "#{method.upcase} #{url} (headers=#{headers})"
end

def get(...)
  request(:get, ...)
end
get('https://example.com', headers: {content_type: 'json'})
```

## Enumerator.produce

```ruby
# Before
require 'date'

d = Date.today
while !d.tuesday
  d += 1
end

# After
Enumerator.produce(Date.today, &:next).find(&:tuesday?)

```

# ruby3.0

* Performance Faster 3x3
* Pattern matching
* Type declarations
* Ractors
* Non-blocking Fiber

## 类型声明（Type declarations）

> 2010s were an age of statically typed programming languages. Ruby seeks the future with static type checking, without type declaration, using abstract interpretation. RBS & TypeProf are the first step to the future. More steps to come. — Matz

简单了解下背景：http://kamiiyu.github.io/blog/2016/11/30/ruby3-typing/

借助 https://github.com/ruby/rbs 实现

> RBS is a language to describe the structure of Ruby programs.

简单的例子：

```ruby
class Alice
  attr_accessor :name, :age, :positions

  def initialize(name, age, positions)
    @name = name
    @age = age
    @positions = positions
  end

  def eats(foods = [])
    "Alice eats foods: #{foods.join(',')}"
  end
end
```

查看prototype：`bundle exec rbs prototype rb alice.rb`

```
class Alice
  attr_accessor name: untyped

  attr_accessor age: untyped

  attr_accessor positions: untyped

  def initialize: (untyped name, untyped age, untyped positions) -> untyped

  def eats: (?untyped foods) -> ::String
end
```

生成rbs文件：`bundle exec typeprof alice.rb > sig/alice.rbs`

```
# TypeProf 0.14.1

# Classes
class Alice
  attr_accessor name: String
  attr_accessor age: Integer
  attr_accessor positions: [String, String]
  def initialize: (String name, Integer age, [String, String] positions) -> [String, String]
  def eats: (?Array[String] foods) -> String
end
```

使用steep来做类型check

```
# 初始化steep
bundle exec steep init

bundle exec steep stats

bundle exec steep check
```

### 参考资料

* https://sorbet.org/
* https://www.ruby-lang.org/en/news/2020/12/25/ruby-3-0-0-released/
* https://blog.appsignal.com/2021/01/27/rbs-the-new-ruby-3-typing-language-in-action.html
* https://github.com/ruby/typeprof
* https://github.com/soutaro/steep
* https://evilmartians.com/chronicles/climbing-steep-hills-or-adopting-ruby-types

## “Endless” method definition

ruby中的方法都是通过`do...end`来声明的，但是某些比较简单的方法（大多数只有一行），end就占用一行看起来比较*重*了。

```ruby
# Before
def available?
  !@internal.empty?
end

# After
# 变得更加函数式
def available? = !@internal.empty?

def square(x) = x**2
square(3)

```



















