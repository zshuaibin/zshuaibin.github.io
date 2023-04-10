---
draft: true
---

# 模块(module)

任何py文件都可以作为一个模块。

## 好处

- 避免命名空间的冲突
- 代码工程化、结构化
- 暴露接口，隐藏细节

## 导入模块

```python
# 方式1 - 导入全部的成员（变量、函数、类等等），调用需要使用前缀
import 模块名1 [as 别名1], 模块名2 [as 别名2]

# 方式2 - 导入指定的成员，调用不需要使用前缀
from 模块名 import 成员名1 [as 别名1], 成员名2 [as 别名2]
```

例子：

```python
# File: a.py

foo = 1

def hello():
    print('hello')

class Hello:
    def hello(self):
        print('hello from class Hello')
```

```python
# File: b.py
import a

print(a.foo)
a.hello()
a.Hello().hello()
```

```python
# File: c.py
from a import foo, Hello, hello

print(foo)
hello()
Hello().hello()
```

## 特殊变量

### `__name__`

1. 当py文件被直接运行的时候，值为`__main__`

2. 当py文件被import的时候，值为py的文件名

```python
if __name__ == '__main__'
    # 想运行的代码在这里
```

### `__doc__`

```python
# File: a.py
'''
Hello, Everyone!
'''

# File: b.py
import a
print(a.__doc__)
```

## 寻找路径

1. 当前的目录

2. 环境变量PYTHONPATH中的每一个目录

3. python的安装目录

# 包(package)

包（package）将大量的模块组织在一起并管理它们。

## 常规包

封装成包是很简单的。在文件系统上组织代码，确保每个目录都定义了一个`__init__.py`文件：

```shell
graphics/
    __init__.py
    primitive/
        __init__.py
        line.py
        fill.py
        text.py
    formats/
        __init__.py
        png.py
        jpg.py
```

```python
import graphics.primitive.line
from graphics.primitive import line
import graphics.formats.jpg as jpg
```

例子：

```shell
abcd/
    __init__.py
    pack/
        __init__.py
    cat.py
b.py
```

```python
# File: abcd/cat.py
name = 'cat'
```

```python
# File: b.py
import abcd
# from abcd import cat

print(abcd)
print(abcd.cat.name)


# 执行 python b.py
# 报错了，错误如下：
# <module 'abcd' from 'learn-python/abcd/__init__.py'>
# AttributeError: module 'abcd' has no attribute 'cat'


# 如果解除第三行的注释，运行成功
# <module 'abcd' from 'learn-python/abcd/__init__.py'>
# <module 'abcd.cat' from 'learn-python/abcd/cat.py'>
# cat
```

## 命名空间包（python >= 3.3）

可以没有`__init__.py`文件

参考：[10.5 利用命名空间导入目录分散的代码 &mdash; python3-cookbook 3.0.0 文档](https://python3-cookbook.readthedocs.io/zh_CN/latest/c10/p05_separate_directories_import_by_namespace.html)

例如：

```shell
abcd/
    pack/
    cat.py
b.py
```

```python
import abcd
from abcd import cat

print(abcd)
print(cat)
print(abcd.cat.name)

# <module 'abcd' (namespace)>
# <module 'abcd.cat' from 'learn-python/abcd/cat.py'>
# cat
```

## 区别

- 常规包中的`__init__.py`可以编写一些初始代码
  
  ```python
  # File: abcd/pack/__init__.py
  from .a import foo
  
  # 使用方：
  # 替代 form abcd.pack.a import foo
  from abcd.pack import foo
  ```

- 命名空间包可以导入目录分散的代码
