---
date: "2019-07-08T18:21:48+08:00"
draft: false
title: "python基础梳理"
tags: ["python"]
categories: ["python"]
toc: true
---

## python简介
### 定位
python的定位：优雅、明确、简单。

### python解释器
`Cpython`：C语言开发。通常在命令行下运行python就是启动CPython解释器，一般是以`>>>`作提示符。      
IPython：基于CPython之上的一个交互式解释器，只是在交互方式上有所增强。Python用`In [序号]:`作为提示符。    
PyPy：P采用JIT技术，对Python代码进行动态编译（注意不是解释）。所以可以显著提高python代码执行速度。     
Jython：运行在Java平台上的Python解释器，直接把Python代码编译成Java字节码执行。     
IronPython：类似JPython，把Python代码编译成.Net的字节码。

### 输入输出
输入：input()     
输出：print()

### 注释
用`#`标记注释。

### 缩进
当语句以冒号`:`结尾时，缩进的语句视为代码块。

### 大小写敏感
Python程序是大小写敏感的，如果写错了大小写，程序会报错。

## python基础
### 数据类型和变量
整数     
浮点数（值得注意的是：科学计数法可以将1.23x10<sup>9</sup>表示为1.23e9）     
字段串     
布尔值：True/False    
`空值：None`

### 条件判断
`if`语句：
```python
age = 3
if age >= 18:
    print('adult')
elif age >= 6:
    print('teenager')
else:
    print('kid')
```

### 循环
Python的循环有两种，一种是for...in循环，一种循环是while循环。     
ffor循环常用的几个场景：
```python
names = ['Michael', 'Bob', 'Tracy']
for name in names:
    print(name)
```
```python
sum = 0
for x in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:
    sum = sum + x
print(sum)
```
```python
sum = 0
for x in range(101):
    sum = sum + x
print(sum)

```
比较高级的用法，在列表生成式中的应用：
```python
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]

```
```python
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

### list与tuple
`list`：列表。是一种有序的集合，可随时添加删除其中元素。
```python
l = [1,2,3] 
l = list(range(1, 11))
```
`tuple`：元组。是一种有序列表，初始化后不能修改。
```python
t = (1,2,3)
```
这里要注意的是：如果要定义只有一个元素的tuple。`(1)`python会解释成整数1，所以为了区分整数与tuple，一般只有
一个元素的tuple写成`(1,)`。要多加一个逗号。

### dict与set
`dict`：字典。其他语言也称map，key - value模型，极快速查询。
```python
d = {'a':1,'b':2}
```
在dict中，key必须是不可变对象。字符串、整数不可变，可key。list可变，不能做key。

`set`：与dict类似，也是一组key的集合，但不存储value。在set中，没有重复的key。
```python
s = set([1,2,3])
```

### 可变对象与不可变对象
`不可变对象`：调用自身任意方法。不改变对象自身内容。相反，会创建新的对象并返回。
