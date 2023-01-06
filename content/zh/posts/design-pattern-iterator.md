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
# 迭代器

## 不暴露复杂数据结构内部细节的情况下，遍历其中所有的元素

## 模式结构

![YM81He](https://static-1255384104.cos.ap-shanghai.myqcloud.com/uPic/YM81He.jpg)

## 核心要素

* getNext()
* hasMore(): bool

## Code

```golang
package main

type user struct {
  name string
  age  int
}

type collection interface {
  createIterator() iterator
}

type iterator interface {
  hasNext() bool
  getNext() *user
}

type userCollection struct {
  users []*user
}

func (u *userCollection) createIterator() iterator {
  return &userIterator{
    users: u.users,
  }
}

type userIterator struct {
  index int
  users []*user
}

func (u *userIterator) hasNext() bool {
  if u.index < len(u.users) {
    return true
  }
  return false
}

func (u *userIterator) getNext() *user {
  if u.hasNext() {
    user := u.users[u.index]
    u.index++
    return user
  }
  return nil
}

// 使用
func main() {

  user1 := &user{
    name: "a",
    age:  30,
  }
  user2 := &user{
    name: "b",
    age:  20,
  }

  userCollection := &userCollection{
    users: []*user{user1, user2},
  }

  iterator := userCollection.createIterator()

  for iterator.hasNext() {
    user := iterator.getNext()
    fmt.Printf("User is %+v\n", user)
  }
}
```

```ruby
iterator = [
  {name: 'a', age: 30},
  {name: 'b', age: 20}
].to_enum

iterator.each { |user| puts user }

# We need more!

iterator.next
iterator.peek

while iterator.peek
  user = iterator.next
  puts user
end

class SomeIterator
  include Enumerable

  # The class must provide a method each, which yields successive members of the collection.
  def each(&block)
    # yield
    # yield
    # yield
  end
end

```

