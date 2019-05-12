title: Python 函数的参数规则
date: 2015-10-13 01:26:38
categories:
tags:
- python
---
## 位置参数
```python
def fun(a):
    pass
```

<!--more-->
## 任意多的位置参数
```python
def fun(*args):
    pass
fun(1, '2', 3.4) # 调用
```
args 会被映射成一个tuple，可以使用 *args 进行解包


## 默认参数
```python
def fun(a, defalut=1):
    pass
```
默认参数必须出现在位置参数后，否则报错 SyntaxError
可以用以下三种方式调用：
```python
fun(1)
fun(1, 2)
fun(1, default=2)
```

## 关键字参数
```python
def fun(*, kw=1): # 1
    pass
def fun(*args, kw=1): # 2
    pass
```
这两种方式可以定义关键字参数，即在 * 后定义参数，这样的参数就不再是默认参数了
也可以不带默认值：
```python
def fun(*, kw):
    pass
```
它只能以下面这钟方式调用：
```python
fun(kw=1)
```
如果像下面这样：
```python
def fun(*, kw=1): 
    pass
    
fun(2)
```
则会报错 TypeError: fun() takes 0 positional arguments but 1 was given
或者这样：
```python
def fun(*, kw):
    pass

fun()
```
则会报错 TypeError: kwfun() missing 1 required keyword-only argument: 'kw'

## 任意多的关键字参数
```python
def fun(**kwargs):
    pass
```
kwargs会被映射成dict，可以使用 **kwargs 进行解包

## 参数顺序
位置参数 < 默认参数 < 任意多位置参数 < 关键字参数 < 任意多关键字参数
**注意**有默认参数的关键字参数要在无默认参数的后面
