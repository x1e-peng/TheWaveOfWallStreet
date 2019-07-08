---
date: "2019-07-07T18:11:48+08:00"
draft: false
title: "python关键字"
tags: ["python"]
categories: ["python"]
toc: true
---

### 关键字获取
Python自带了一个`keyword`模块，用于检测关键字。
```python
>>> import keyword
>>> print(keyword.kwlist)

['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await', 'break', 'class', 
'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 
'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 
'return', 'try', 'while', 'with', 'yield']

```

### 关键字判断
关键字判断也是用的keyword模块。
```python
>>> keyword.iskeyword('class')
True
>>> keyword.iskeyword('hello')
False
```

### 关键字详解
#### and、or、not
分别对应逻辑判断 `【与】`、`【或】`、`【非】`。

#### with、as
with、as单独没有实际意思，常与with组合使用，`with...as`。例如我生成一个txt文件，在里面写一点内容：
```python
with open('./world.txt', 'w') as f:
    f.write('人生苦短, 我用python')
```

#### try、raise、except、finally
这几个也是配合着使用，来抛出异常和捕获异常：
```python
try:
    raise EOFError
except EOFError:
    print('捕获到一个异常。')
finally:
    print('始终都会执行')
```

#### from、import
`from...import`一般配合使用，用作导入相应的模块：
```python
from world import my_abs
    print(my_abs(-99))
```

#### is
Python中的对象包含三要素：`id`、`type`、`value`
其中id用来唯一标识一个对象(id 就相当于内存地址 指针)，type标识对象的类型，value是对象的值: 
   
 > `is`判断的是a对象是否就是b对象，是通过id来判断的, `is not`用于判断两个对象不相等。        
 `==`判断的是a对象的值是否和b对象的值相等，是通过value来判断的。

```python
list1 = [1, 2, 3, 4, 5]
list2 = [1, 2, 3, 4, 5]
list3 = list2
print(id(list1))
print(id(list2))
print(id(list3))
```

#### lambda
表示匿名函数。
```python
f = lambda : 'Hello World !'
print(f())

max_value = lambda x, y=2: max(x, y)
print(max_value(5))
```

#### global、nonlocal
`global`：定义全局变量。    
`nonlocal`：关键字nonlocal的作用与关键字global类似，使用nonlocal关键字可以在一个嵌套的函数中修改嵌套作用域中的变量。
```python
#定义全局变量：变量名全部大写
NAME = 'test'

def get_NAME():
    return NAME

def set_NAME(name_value):
    global NAME
    NAME = name_value

print(get_NAME())
set_NAME('testtest')
print(get_NAME())
```

```python
def foo1():
    num = 2
    def foo2():
        nonlocal num
        num = 5
    foo2()
    print('num = ', num)

foo1() # num = 5
```

#### pass
pass的意思就是什么都不做。相当于占位：当我们写一个软件的框架时，具体方法和类来不及实现，
就在方法和类里面加上pass，这样编译起来就不会报错。
```python
def test():
    pass #  不加pass 会报错
```

#### yield
`yield`一般用来定义生成器函数。具体生成数函数是什么，这里就不解释了，在后续python基础章节再详解吧。
```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'


f = fib(3)

print(next(f))
print(next(f))
```