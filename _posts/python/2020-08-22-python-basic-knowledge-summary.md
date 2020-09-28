---
layout: post
title: Python 基础知识
categories: [Python]
description: some word here
keywords: Python
---

# Python 基础知识

## 变量和数据类型

### 变量

python 中的变量不用显示声明类型，变量的类型由接收的值的类型来决定，每个变量都会存储一个值

+ 变量名只能包含**字母**、**数字**、**下划线**，变量名可以使用字母、下划线开头，但是不能以数字打头
+ 变量名不能包含空格，一般使用下划线来分隔其中的单词
+ 不能将python中的关键字和函数名用作变量名
+ 变量名应该**简短**且具有**描述性**

#### 关键字

| 关键字       | 描述                                                |
| ------------ | --------------------------------------------------- |
| **and**      | 逻辑运算符**与**                                    |
| **or**       | 逻辑运算符**或**                                    |
| **not**      | 逻辑运算符**非**                                    |
| **is**       | 判断两个值是否相等，与 == 的区别                    |
| **in**       | 判断元素是否在列表、元组等集合中                    |
| **True**     | 布尔值**真**，比较运算的结果                        |
| **False**    | 布尔值**假**，比较运算的结果                        |
| **if**       | 条件语句                                            |
| **else**     | 条件语句                                            |
| **elif**     | 在条件语句中使用，等同于 else if                    |
| **for**      | 创建 for 循环                                       |
| **while**    | 创建 while 循环                                     |
| **break**    | 跳出循环                                            |
| **continue** | 继续循环的下一个迭代                                |
| **None**     | 表示 null 值                                        |
| **try**      | try...except 语句                                   |
| **except**   | 处理异常，发生异常时执行的流程                      |
| **finally**  | 处理异常，无论是否存在异常，都将执行 finally 代码块 |
| **raise**    | 手动产生异常                                        |
| **with**     | 用于简化异常处理                                    |
| **from**     | 导入模块的特定部分                                  |
| **import**   | 导入模块                                            |
| **as**       | 创建别名                                            |
| **assert**   | 用于调试                                            |
| **class**    | 定义类                                              |
| **def**      | 定义函数                                            |
| **lambda**   | 创建匿名函数                                        |
| **return**   | 结束并返回函数结果                                  |
| **yield**    | 结束函数，返回生成器                                |
| **del**      | 删除对象                                            |
| **pass**     | null 语句，一条什么都不做的语句                     |
| **global**   | 声明全局变量                                        |
| **nonlocal** | 声明非局部变量                                      |



#### 内置函数

| 函数名               | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| **abs()**            | 返回数字的绝对值                                             |
| **divmod()**         | 把除数和余数运算结果结合起来，返回一个包含商和余数的元组     |
| **input()**          | 接受一个标准输入数据，返回为 string 类型                     |
| **open()**           | 函数用于打开一个文件，创建一个 **file** 对象，相关的方法才可以调用它进行读写 |
| **staticmethod()**   | 返回函数的静态方法                                           |
| **all()**            | 用于判断给定的可迭代参数 iterable 中的所有元素是否都为 True，如果是返回 True，否则返回 False |
| **enumerate()**      | 函数用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中 |
| **int()**            | 用于将一个字符串或数字转换为整型                             |
| **ord()**            | 函数以一个字符（长度为1的字符串）作为参数，返回对应的 ASCII 数值，或者 Unicode 数值 |
| **str()**            | 函数将对象转化为适于人阅读的形式（字符串）                   |
| **any()**            | 函数用于判断给定的可迭代参数 iterable 是否全部为 False，则返回 False，如果有一个为 True，则返回 True |
| **eval()**           | 函数用来执行一个字符串表达式，并返回表达式的值               |
| **isinstance()**     | 函数来判断一个对象是否是一个已知的类型，类似 type()          |
| **pow()**            | 计算 x 的 y 次方的值                                         |
| **sum()**            | 对系列进行求和计算                                           |
| **ascii()**          | 返回一个对象可打印的字符串                                   |
| **bin()**            | 将一个数字转换为`0b`开头的二进制数                           |
| **bool()**           | 返回一个布尔值 True 或者 False                               |
| **bytearray()**      | 返回一个新的 bytes 数组，bytearray 是一个可变序列            |
| **bytes()**          | 返回一个新的 bytes 对象，是一个不可变序列                    |
| **callable(object)** | 如果 object 是一个可调用的返回 True，否则返回 False          |
| **chr()**            | 返回 unicode ，ord() 的逆向操作                              |
| **@classmethod**     | 把一个方法封装成类方法                                       |
| **complex()**        | 返回一个复数值                                               |
| **delattr()**        | 删除对象指定属性                                             |
| **dict()**           | 创建一个新字典                                               |
| **help()**           | 用于查看函数或模块用途的详细说明                             |
| **min()**            | 返回给定参数的最小值                                         |
| **setattr()**        | 用于设置属性值，该属性不一定是存在的                         |
| **getattr()**        | 用于返回一个对象属性值                                       |
| **dir()**            | 不带参数时，返回当前范围内的变量、方法和定义的类型列表；带参数时，返回参数的属性、方法列表 |
| **hex()**            | 用于将一个指定数字转换为 16 进制数                           |
| **next()**           | 返回迭代器的下一个项目；要和生成迭代器的 iter() 函数一起使用 |
| **slice()**          | 实现切片对象，主要用在切片操作函数里的参数传递               |
| **id()**             | 函数返回对象的唯一标识符，标识符是一个整数（内存地址）       |
| **object()**         |                                                              |
| **sorted()**         | 对所有可迭代的对象进行排序操作                               |
| **oct()**            | 将一个整数转换成 8 进制字符串                                |
| **exec()**           | 执行储存在字符串或文件中的 Python 语句                       |
| **filter()**         | 用于过滤序列，过滤掉不符合条件的元素，返回一个迭代器对象，如果要转换为列表，可以使用 **list()** 来转换 |
| **issubclass()**     | 判断参数 class 是否是类型参数 classinfo 的子类               |
| **super()**          | 调用父类(超类)的一个方法                                     |
| **float()**          | 将整数和字符串转换成浮点数                                   |
| **iter()**           | 用来生成迭代器                                               |
| **print()**          | 打印输出                                                     |
| **tuple()**          | 将可迭代系列（如列表）转换为元组                             |
| **format()**         | 格式化字符串                                                 |
| **len()**            | 返回对象（字符、列表、元组等）长度或项目个数                 |
| **property()**       | 是在新式类中返回属性值                                       |
| **type()**           |                                                              |
| **frozenset()**      | 返回一个冻结的集合，冻结后集合不能再添加或删除任何元素       |
| **list()**           | 将元组或字符串转换为列表                                     |
| **range()**          | 返回一个可迭代对象（类型是对象），而不是列表类型             |
| **vars()**           | 返回对象object的属性和属性值的字典对象                       |
| **locals()**         | 以字典类型返回当前位置的全部局部变量                         |
| **repr()**           | 将对象转化为供解释器读取的形式                               |
| **zip()**            | 将可迭代的对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的对象 |
| **compile()**        | 将一个字符串编译为字节代码                                   |
| **globals()**        | 会以字典类型返回当前位置的全部全局变量                       |
| **map()**            | 会根据提供的函数对指定序列做映射                             |
| **reversed()**       | 返回一个反转的迭代器                                         |
| **\_\_import\_\_()** | 用于动态加载类和函数                                         |
| **hasattr()**        | 用于判断对象是否包含对应的属性                               |
| **max()**            | 返回给定参数的最大值，参数可以为序列                         |
| **round()**          | 返回四舍五入的值                                             |
| **hash()**           | 获取取一个对象（字符串或者数值等）的哈希值                   |
| **memoryview()**     | 返回给定参数的内存查看对象                                   |
| **set()**            | 创建一个无序不重复元素集，可进行关系测试，删除重复数据，还可以计算交集、差集、并集 |

> 1. **isinstance()** 与 **type()** 区别：
>- type() 不会认为子类是一种父类类型，不考虑继承关系
>    - isinstance() 会认为子类是一种父类类型，考虑继承关系
> 
>如果要判断两个类型是否相同推荐使用 isinstance()

all() ：

```python
def all(iterable):
    for element in iterable:
        if not element:
            return False
    return True
```

 any() ：

```python
def any(iterable):
    for element in iterable:
        if element:
            return True
    return False
```



#### 运算符

1. 算数运算符

| 运算符 | 描述   |
| ------ | ------ |
| +      | 加     |
| -      | 减     |
| *      | 乘     |
| /      | 除     |
| %      | 取余   |
| **     | 幂运算 |
| //     | 取整数 |

2. 比较运算符

| 运算符 | 描述     |
| ------ | -------- |
| **==** | 等于     |
| **!=** | 不等于   |
| **>**  | 大于     |
| **<**  | 小于     |
| **>=** | 大于等于 |
| **<=** | 小于等于 |

3. 赋值运算符

| 运算符   | 描述       |
| -------- | ---------- |
| **=**    | 赋值       |
| **+=**   | 加赋值     |
| **-=**   | 减赋值     |
| ***=**   | 乘赋值     |
| **/=**   | 除赋值     |
| **\**=** | 幂赋值     |
| **%=**   | 取余赋值   |
| **//=**  | 取整赋值   |
| **:=**   | 海象运算符 |

4. 位运算符

| 运算符 | 描述     |
| ------ | -------- |
| **&**  | 按位与   |
| **\|** | 按位或   |
| **^**  | 按位异或 |
| **~**  | 按位取反 |
| **<<** | 左移     |
| **>>** | 右移     |

5. 逻辑运算符

| 运算符  | 描述 |
| ------- | ---- |
| **and** | 与   |
| **or**  | 或   |
| **not** | 非   |

6. 成员运算符

| 运算符     | 描述         |
| ---------- | ------------ |
| **in**     | 在指定序列   |
| **not in** | 不在指定序列 |

7. 身份运算符

| 运算符     | 描述                               |
| ---------- | ---------------------------------- |
| **is**     | 判断两个标识符是不是引用自一个对象 |
| **not is** | 判断两个标识符是不是引用自不同对象 |

> 运算符优先级



### 数据类型

+ **Number**: 数字
+ **String:** z字符串
+ **List:** 列表
+ **Tuple:** 元组
+ **Set:** 集合
+ **Dictionary:** 字典

> 不可变数据： Number（数字）、String（字符串）、Tuple（元组） 
>
> 可变数据： List（列表）、Dictionary（字典）、Set（集合） 



####  Number

Python 支持的类型：**int**、**float**、**bool**、**complex** 

> Python 支持同时为多个变量赋值 a,b=1,2



#### String

Python 中字符串使用单引号 `''` 和双引号 `""` 括起来都叫字符串，使用`\`转义特殊字符

字符串截取：

```python
变量[头下标:尾下标] # 索引值 0 表示开头，-1 表示结尾 ; 使用 + 连接字符串，使用 * 复制字符串
```



#### List

Python 中列表是放在 `[]` 中的数据，元素之间使用`,`分隔

列表截取：

```python
变量[头下标:尾下标]  # 索引值 0 表示开头，-1 表示结尾 ; 使用 + 连接操作，使用 * 复制操作
```

> 常用处理列表函数：
>
> 1. **count()** ： 统计元素出现次数
> 2. **len()** ：  统计列表长度
> 3.  **append()** ：  往列表的最后一个位置插入一个数据（入栈）操作 
> 4.  **extend()** ：  添加指定列表的所有元素来扩充列表  
> 5.  **insert()**：  将元素插入到列表的指定位置 
> 6.  **pop()** :   删除指定位置元素 
> 7.  **remove()**：   删除列表中值为 x 的第一个元素 ， 如果没有指定索引，pop() 返回最后一个元素 ， 元素随即从列表中被移除 
> 8.   **index()** ：从列表中找出与某个元素匹配的第一个匹配项的位置 
> 9.  **reverse()**：  翻转列表 
> 10. **sort()** ： 对原列表进行排序 
> 11. **set()**：  列出列表中不重复的元素（去重）集合 
> 12. **clear()：**  移除列表中的所有项，等于del a[:] 

 

#### Tuple

元组与列表类似，但是元组中的元素不能修改，使用`()`来容纳数据，使用 `,` 对元素进行分隔

元组(tuple) 可是使用 `+` 进行运算，元组里的元素可以是可变对象



#### Set

 集合（set）是由一个或数个形态各异的大小整体组成的，构成集合的事物或对象称作元素或是成员

使用`{}` 表示数据，使用`set()` 创建一个空的集合，不能使用`{}`来创建一个空集合，`{}`创建一个空字典



#### Dictionary

 字典（dictionary） 是无序的对象集合 ， 列表是有序的对象集合， 两者之间的区别在于：字典当中的元素是通过**键**来存取的，而不是通过偏移存取 ；字典使用 `{}` 标识，是一个无序的 `key:value` 集合

**键(key)** 的值必须是唯一的，不可变的值

遍历技巧：

```python
knights = {'gallahad': 'the pure', 'robin': 'the brave'}
for k, v in knights.items():
    print(k, v)
```



 

#### 数据类型转换

| 函数             | 描述                                                |
| ---------------- | --------------------------------------------------- |
| **int(x)**       | 将x转换为一个整数                                   |
| **float(x)**     | 将x转换到一个浮点数                                 |
| **complex()**    | 创建一个复数                                        |
| **str(x)**       | 将对象 x 转换为字符串                               |
| **repr(x)**      | 将对象 x 转换为表达式字符串                         |
| **eval(str)**    | 用来计算在字符串中的有效Python表达式,并返回一个对象 |
| **tuple(s)**     | 将序列 s 转换为一个元组                             |
| **list(s)**      | 将序列 s 转换为一个列表                             |
| **set(s)**       | 转换为可变集合                                      |
| **dict(d)**      | 创建一个字典，d 必须是一个 (key, value)元组序列     |
| **frozenset(s)** | 转换为不可变集合                                    |
| **chr(x)**       | 将一个整数转换为一个字符                            |
| **ord(x)**       | 将一个字符转换为它的整数值                          |
| **hex(x)**       | 将一个整数转换为一个十六进制字符串                  |
| **oct(x)**       | 将一个整数转换为一个八进制字符串                    |



## 控制流

### 条件语句

#### if 语句 

Python 中使用 `elif` 代替了 `else if` ，所以 Python 中的 `if` 语句关键字为 `if-elif-else`

> 1. 每个条件后面使用 `:` ，表示满足条件将要执行的代码块
> 2. 使用缩进来划分语句块，相同缩进表示一个语句块

```python
number = 7
guest = -1
while guest != number:
    guest = int(input("input a number:"))

    if guest == number:
        print ("right")
    elif guest < number:
        print ("smaller")
    elif guest > number:
        print ("bigger")
```

```python
# coding=utf-8
num = int(input("输入一个数字："))
if num % 2 == 0:
    if num % 3 == 0:
        print ("你输入的数字可以整除 2 和 3")
    else:
        print ("你输入的数字可以整除 2，但不能整除 3")
else:
    if num % 3 == 0:
        print ("你输入的数字可以整除 3，但不能整除 2")
    else:
        print  ("你输入的数字不能整除 2 和 3")
```



### 循环语句

#### while 循环

```python
#!/usr/bin/env python3
 
n = 100
sum = 0
counter = 1
while counter <= n:
    sum = sum + counter
    counter += 1
 
print("1 到 %d 之和为: %d" % (n,sum))
```



#### for循环

```python
#!/usr/bin/python3
 
sites = ["Baidu", "Google","Str","Taobao"]
for site in sites:
    if site == "Str":
        print("Str!")
        break
    print("循环数据 " + site)
else:
    print("没有循环数据!")
print("完成循环!")
```



#### range()函数 

遍历列表

```python
a = ['Google', 'Baidu', 'Str', 'Taobao', 'QQ']
for i in range(len(a)):
    print(i, a[i])
```

> break 语句
>
> continue 语句
>
> pass 语句



### 迭代器&生成器

迭代是访问集合元素的一种方式，迭代器是一个可以记住遍历的位置的对象，迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束，迭代器只能往前不会后退，迭代器有两个基本的方法：**iter()** 和 **next()**



#### iter()

```python
#!/usr/bin/python3
 
list=[1,2,3,4]
it = iter(list)    # 创建迭代器对象
for x in it:	# for 循环遍历
    print (x, end=" ")
```



#### next()

```python
#!/usr/bin/python3
 
import sys         # 引入 sys 模块
 
list=[1,2,3,4]
it = iter(list)    # 创建迭代器对象
 
while True:
    try:
        print (next(it))
    except StopIteration:
        sys.exit()
```



#### 迭代器

把一个类作为一个迭代器使用需要在类中实现两个方法 \_\_iter\_\_() 与 \_\_next\_\_() 

\_\_iter\_\_() 方法返回一个特殊的迭代器对象， 这个迭代器对象实现了 \_\_next\_\_() 方法并通过 StopIteration 异常标识迭代的完成，防止出现无限循环的情况， \_\_next\_\_() 方法会返回下一个迭代器对象。

创建一个返回数字的迭代器，初始值为 1，逐步递增 1：

```python
class MyNumbers:
  def __iter__(self):
    self.a = 1
    return self
 
  def __next__(self):
    if self.a <= 20:
      x = self.a
      self.a += 1
      return x
    else:
      raise StopIteration
 
myclass = MyNumbers()
myiter = iter(myclass)
 
for x in myiter:
  print(x)
```



#### 生成器

在 Python 中，使用了 yield 的函数被称为生成器（generator），跟普通函数不同的是，生成器是一个返回迭代器的函数，只能用于迭代操作，更简单点理解生成器就是一个迭代器，在调用生成器运行的过程中，每次遇到 yield 时函数会暂停并保存当前所有的运行信息，返回 yield 的值, 并在下一次执行 next() 方法时从当前位置继续运行，调用一个生成器函数，返回的是一个迭代器对象。

以下实例使用 yield 实现斐波那契数列：

```python
#!/usr/bin/python3
 
import sys
 
def fibonacci(n): # 生成器函数 - 斐波那契
    a, b, counter = 0, 1, 0
    while True:
        if (counter > n): 
            return
        yield a
        a, b = b, a + b
        counter += 1
f = fibonacci(10) # f 是一个迭代器，由生成器返回生成
 
while True:
    try:
        print (next(f), end=" ")
    except StopIteration:
        sys.exit()
```



## 函数

- 函数代码块以 **def** 关键词开头，后接函数标识符名称和圆括号 **()**
- 任何传入参数和自变量必须放在圆括号中间，圆括号之间可以用于定义参数
- 函数的第一行语句可以选择性地使用文档字符串—用于存放函数说明
- 函数内容以冒号起始，并且缩进
- **return [表达式]** 结束函数，选择性地返回一个值给调用方，不带表达式的return相当于返回 None

```python
def 函数名（参数列表）:
    函数体
```

```python
#!/usr/bin/python3
 
# 定义函数
def printme( str ):
   # 打印任何传入的字符串
   print (str)
   return
 
# 调用函数
printme("我要调用用户自定义函数!")
printme("再次调用同一函数")
```



### 参数传递

**可更改(mutable)**与**不可更改(immutable)**对象

在 python 中，strings，tuples 和 numbers 是不可更改的对象，而 list，dict 等则是可以修改的对象

- **不可变类型：**变量赋值 **a=5** 后再赋值 **a=10**，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5 被丢弃，不是改变 a 的值，相当于新生成了a
- **可变类型：**变量赋值 **list=[1,2,3,4]** 后再赋值 **list[2]=5** 则是将 list 的第三个元素值更改，本身 list 没有变，只是其内部的一部分值被修改了

python 函数的参数传递：

- **不可变类型：**类似 c++ 的值传递，如 整数、字符串、元组。如fun（a），传递的只是 a 的值，没有影响 a 对象本身。比如在 fun（a）内部修改 a 的值，只是修改另一个复制的对象，不会影响 a 本身。
- **可变类型：**类似 c++ 的引用传递，如 列表，字典。如 fun（list），则是将 list 真正的传过去，修改后fun外部的 list 也会受影响

python 中一切都是对象，严格意义我们不能说值传递还是引用传递，我们应该说传**不可变对象**和传**可变对象**

```python
#!/usr/bin/python3
 
def ChangeInt( a ):
    a = 10
 
b = 2
ChangeInt(b)
print( b ) # 结果是 2
```

```python
#!/usr/bin/python3
 
# 可写函数说明
def changeme( mylist ):
   "修改传入的列表"
   mylist.append([1,2,3,4])
   print ("函数内取值: ", mylist)
   return
 
# 调用changeme函数
mylist = [10,20,30]
changeme( mylist )
print ("函数外取值: ", mylist)
```



### 参数

- 必需参数
- 关键字参数
- 默认参数
- 不定长参数

**必须参数**

必需参数须以正确的顺序传入函数。调用时的数量必须和声明时的一样 

```python
#!/usr/bin/python3
 
#可写函数说明
def printme( str ):
   "打印任何传入的字符串"
   print (str)
   return
 
# 调用 printme 函数，不加参数会报错
printme()
```

**关键字参数**

关键字参数和函数调用关系紧密，函数调用使用关键字参数来确定传入的参数值

使用关键字参数允许函数调用时参数的顺序与声明时不一致，因为 Python 解释器能够用参数名匹配参数值

```python
#!/usr/bin/python3
 
#可写函数说明
def printme( str ):
   "打印任何传入的字符串"
   print (str)
   return
 
#调用printme函数
printme( str = "关键字参数")
```

```python
#!/usr/bin/python3
 
#可写函数说明
def printinfo( name, age ):
   "打印任何传入的字符串"
   print ("名字: ", name)
   print ("年龄: ", age)
   return
 
#调用printinfo函数
printinfo( age=50, name="Tom" )
```

**默认参数**

 调用函数时，如果没有传递参数，则会使用默认参数 

```python
#!/usr/bin/python3
 
#可写函数说明
def printinfo( name, age = 35 ):
   "打印任何传入的字符串"
   print ("名字: ", name)
   print ("年龄: ", age)
   return
 
#调用printinfo函数
printinfo( age=50, name="Tom" )
print ("------------------------")
printinfo( name="Tom" )
```

**不定长参数**

 函数能够处理比声明时更多的参数，这些参数叫做不定长参数 ，加了一个星号 ***** 的参数会以元组(tuple)的形式导入，存放所有未命名的变量参数 ， 加了两个星号 ***\*** 的参数会以字典的形式导入 ，星号 ***** 也可以单独出现， 如果单独出现，那么星号 ***** 后的参数必须用关键字传入 

基本语法:

```python
def functionname([formal_args,] *var_args_tuple ):
   "函数_文档字符串"
   function_suite
   return [expression]
```

```python
#!/usr/bin/python3
  
# 可写函数说明
def printinfo( arg1, *vartuple ):
   "打印任何传入的参数"
   print ("输出: ")
   print (arg1)
   print (vartuple)
 
# 调用printinfo 函数
printinfo( 70, 60, 50 )
```

```python
输出: 
70
(60, 50)
```

基本语法：

```python
def functionname([formal_args,] **var_args_dict ):
   "函数_文档字符串"
   function_suite
   return [expression]
```

```python
#!/usr/bin/python3
  
# 可写函数说明
def printinfo( arg1, **vardict ):
   "打印任何传入的参数"
   print ("输出: ")
   print (arg1)
   print (vardict)
 
# 调用printinfo 函数
printinfo(1, a=2,b=3)
```

```
输出: 
1
{'a': 2, 'b': 3}
```

基本语法：

```python
def functionname(a,b,*,c):
     "函数_文档字符串"
   function_suite
   return [expression]
```

```python
def f(a,b,*,c):
     return a+b+c
    
f(1,2,3)   # 错误调用
f(1,2,c=3) # 正确调用
```





### 匿名函数

python 使用 lambda 来创建匿名函数，所谓匿名，即不再使用 def 语句这样标准的形式定义一个函数

- lambda 只是一个表达式，函数体比 def 简单很多
- lambda 的主体是一个表达式，而不是一个代码块，仅仅能在 lambda 表达式中封装有限的逻辑进去
- lambda 函数拥有自己的命名空间，且不能访问自己参数列表之外或全局命名空间里的参数
- 虽然 lambda 函数看起来只能写一行，却不等同于 C 或 C++ 的内联函数，后者的目的是调用小函数时不占用栈内存从而增加运行效率

基本语法：

```python
lambda [arg1 [,arg2,.....argn]]:expression
```

```python
#!/usr/bin/python3
 
# 可写函数说明
sum = lambda arg1, arg2: arg1 + arg2
 
# 调用sum函数
print ("相加后的值为 : ", sum( 10, 20 ))
print ("相加后的值为 : ", sum( 20, 20 ))
```



### return 语句

 **return [表达式]** 语句用于退出函数，选择性地向调用方返回一个表达式，不带参数值的return语句返回None 

```python
#!/usr/bin/python3
 
# 可写函数说明
def sum( arg1, arg2 ):
   # 返回2个参数的和."
   total = arg1 + arg2
   print ("函数内 : ", total)
   return total
 
# 调用sum函数
total = sum( 10, 20 )
print ("函数外 : ", total)
```



## 模块

**模块**是一个包含了所有你定义的函数和变量的文件，其后缀为`.py`，模块可以被其他程序引入，以使用该模块中的函数等功能，这也是使用 python 标准库的方法

**模块**可以让我们有逻辑的组织 python 代码，将相关的代码分配到一个模块，让代码更易用，更易懂

**模块** 能够定义函数、类、变量，也可以包含可执行代码



### 模块使用

#### 导入模块

```python
# 从模块中导入部分/全部部件
from moduel import * 
import module
```

> 使用模块中函数的方法 `module_name.function_name`

#### 使用模块中的数据 

`mymodule.py`

```python
person = {
  "name": "Tom",
  "age": 10,
  "country": "China"
}
```

```python
import mymodule
a = mymodule.person["name"]
print(a)
```

#### 重命名

```python
from oslo_log import log as logging
```

#### dir() 函数

该函数可以列出模块中所有的函数名（或变量名）

```python
import platform

x = dir(platform)
print(x)
```

### 包

**包** 是 python 用于管理 module 命名空间，使用`package.mpdule` 形式使用，采用package + module 的形式可以避免出现重名导致无法区分的问题；导入一个包的时候，Python会根据`sys.path`中的目录来寻找包的子目录，只有目录下面有 `__init__.py` 文件才会认为是一个包



## 面向对象编程

### 基本概念

**类(Class)** 用来描述具有相同的属性和方法的集合，它定义了每个对象所共有的属性方法，对象是类的实例

**方法** 类中定义的函数

**实例变量** 类中定义的一些变量，每个实例都可以使用，一般属性使用变量来表示

**类变量** 是所有对象所共有的，可理解为静态变量

**对象** 类的实例化，具有类的属性（数据成员）和方法

**继承** 派生类（子类）会继承基类（父类）的字段和方法，继承允许把派生类的对象当作基类的对象对待

**重写** 继承基类的方法，如果不能满足派生类的需求，可以进行重写覆盖（override）



### 基本使用

#### 创建类

```python
class Person:
    def __init__(self):
        pass
    age = 10
    name = "Tom"
```

#### 创建对象

```python
person = Person()
person = Person
print(str(person.age) + person.name)
```

####  ` __init__(self):` 函数

该函数，类似于构造函数，用于在实例化对象时传入需要的参数为属性赋值

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age


person = Person("Tom", 10)

print(person.name)
print(person.age)
```

#### 对象方法

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def my_func(self):
        print("Hello my name is " + self.name)


person = Person("Tom", 10)

person.my_func()
```

> **self** 参数表示的是当前类的实例，并非是类；类的方法和普通方法的区别就是第一个额外的参数 **self**
>
> 可是使用 pass 来实现无内容的类的定义，也可以实现接口的功能

#### 继承

子类会继承父类的所有属性和方法

##### 单继承

```python
class DerivedClassName(BaseClassName1):
```

```python
# 类定义
class People:
    # 定义基本属性
    name = ''
    age = 0
    # 定义私有属性,私有属性在类外部无法直接进行访问
    __weight = 0

    # 定义构造方法
    def __init__(self, n, a, w):
        self.name = n
        self.age = a
        self.__weight = w

    def speak(self):
        print("%s say: i am %d years old" % (self.name, self.age))


# 单继承示例
class Student(People):
    # 成员私有属性
    grade = ''

    def __init__(self, n, a, w, g):
        # 调用父类的构函
        People.__init__(self, n, a, w)
        self.grade = g

    # 覆写父类的方法
    def speak(self):
        print("%s say: i am %d years old,i am  %d class" % (
            self.name, self.age, self.grade))


s = Student('Tom', 10, 60, 3)
s.speak()
```

##### 多继承

```python
class DerivedClassName(Base1, Base2, Base3):
```

```python
#!/usr/bin/python3

# 类定义
class People:
    # 定义基本属性
    name = ''
    age = 0
    # 定义私有属性,私有属性在类外部无法直接进行访问
    __weight = 0

    # 定义构造方法
    def __init__(self, name, age, weight):
        self.name = name
        self.age = age
        self.__weight = weight

    def speak(self):
        print("%s say: i am %d years old." % (self.name, self.age))


# 单继承示例
class Student(People):
    grade = ''

    def __init__(self, name, age, weight, grade):
        # 调用父类的构函
        People.__init__(self, name, age, weight)
        self.grade = grade

    # 覆写父类的方法
    def speak(self):
        print("%s say: i am %d years old,i am %d class." % (
            self.name, self.age, self.grade))


class Speaker():
    topic = ''
    name = ''

    def __init__(self, name, topic):
        self.name = name
        self.topic = topic

    def speak(self):
        print("my name is %s,i am a speaker,the topic is %s" % (
            self.name, self.topic))


# 多重继承
class Sample(Speaker, Student):
    a = ''

    def __init__(self, name, age, weight, grade, topic):
        Student.__init__(self, name, age, weight, grade)
        Speaker.__init__(self, name, topic)


test = Sample("Tom", 25, 80, 4, "Python")
test.speak()  # 方法名同,默认调用的是在括号中排前地父类的方法
```

> 当父类的方法无法满足子类的需求时，我们可以重写覆盖父类的方法
>
> 可以使用 `super()` 函数用于调用父类的方法

#### 类的属性与方法

##### 私有属性与方法

类的私有属性和方法使用两个下划线 `__` 开头表示，私有属性和方法只能在类的内部访问，不能在类的外部访问，内部使用 `self.__attr` 方式访问

类的方法与普通方法是有区别的，类的方法也是使用 `def` 进行标识，但是会使用`self`这个必要的参数，且必须为函数的第一个参数

##### 公有属性与方法

公有属性和方法可以在任何地方进行访问，不需要使用特殊的标识来标识



## 命名空间与作用域

### 命名空间

**内置** Python 语言内置的名称，例如内置函数

**全局** 在模块中定义的数据

**局部** 函数中定义的数据

> 查找顺序是从小作用域到大作用域进行查找

### 作用域

**局部作用域** 在函数内部定义的变量，只能在作用域内使用

**全局作用域** 在 python 主体代码内部创建的变量，全局变量在任何范围都可以使用

> 可以使用 `global ` 标识符在任何位置创建一个全局变量

**内建作用域**  通过一个名为 `builtin` 的标准模块来实现的 

```python
import builtins
dir(builtins)
```

>  Python 中只有模块（module），类（class）以及函数（def、lambda）才会引入新的作用域，其它的代码块（if/elif/else/、try/except、for/while等）是不会引入新的作用域的，也就是说这些语句内定义的变量，外部也可以访问



## 文件操作

### 文件打开

使用 `open()` 函数打开一个文件，`open()` 函数有两个参数：文件名和模式， 一般有以下几种不同的打开模式：

+ `r`  读取，默认值。打开文件进行读取，如果文件不存在则会报错
+ `a`  追加，打开文件并追加内容，如果文件不存在则会创建一个新文件
+ `w`  写入，打开文件写入内容（会进行覆盖），如果文件不存在会创建文件
+ `x`  创建，创建指定文件，如果指定文件已经存在则会报错

还可以指定文件的模式

+ `t`  文本文件，默认值
+ `b`  二进制文件

```python
f = open("demofile.txt")
f = open("demofile.txt", "rt")
```

### 文件读取

使用 `open()` 打开文件会返回一个 `file` 对象，我们可以使用这个对象进行对文件进行读取，使用 `read()` 可以读取文件的内容

```python
f = open("demofile.txt", "r")
# read 默认读取整个文件
print(f.read())
# 读取文件一行
f = open("demofile.txt", "r")
print(f.readline())
# 循环读取整个文件
f = open("demofile.txt", "r")
for x in f:
  print(x)
# 关闭文件
f.close()
```

### 文件删除

删除文件需要使用 `os` 模块，使用 `os.remove()` 

```python
import os
if os.path.exists("demofile.txt"):
  os.remove("demofile.txt")
else:
  print("The file does not exist")
```

使用 `os.rmdir()` 删除整个文件夹

```python
import os
os.rmdir("myfolder")
```



## Python 标准库

+ `os`  库提供了与操作系统

```python
import os
```

+ `glob`  该模块提供了用于处理文件的函数

```python
import glob
glob.glob('*.py')
```

+ `sys` 库用于处理常用的命令行操作

```python
import sys
print(sys.argv)
```

+ `re` 用于处理复杂的正则匹配

```python
import re
re.findall(r'\bf[a-z]*', 'which foot or hand fell fastest')
```

+ `math` 用于处理数学中的各种需求

```python
import math
math.cos(math.pi / 4)

math.log(1024, 2)

import random
```

+ `urlopen` `smtplib` 用于处理网络通信协议

```python
from urllib.request import urlopen
import smtplib
```

+ `datetime ` 该模块提供了对于时间日期的处理能力

```python
from datetime import date
```

+ `zlib` 该模块可用于处理各种压缩格式数据

```python
>>> import zlib
>>> s = b'witch which has which witches wrist watch'
>>> len(s)
41
>>> t = zlib.compress(s)
>>> len(t)
37
>>> zlib.decompress(t)
b'witch which has which witches wrist watch'
>>> zlib.crc32(s)
```

+ `unittest` 该模块可用于单元测试

```python
import unittest

class TestStatisticalFunctions(unittest.TestCase):

    def test_average(self):
        self.assertEqual(average([20, 30, 70]), 40.0)
        self.assertEqual(round(average([1, 5, 7]), 1), 4.3)
        self.assertRaises(ZeroDivisionError, average, [])
        self.assertRaises(TypeError, average, 20, 30, 70)

unittest.main() # Calling from the command line invokes all tests
```



## 数据库使用

### 创建连接

> 使用 MySQL 数据库

```python
import mysql.connector

mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="root"
)

print(mydb)
```

### 使用数据库

创建数据库，数据表

```python
import mysql.connector

mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="root"
)

mycursor = mydb.cursor()

# 创建数据库
mycursor.execute("CREATE DATABASE test")

# 查询数据库是否存在
mycursor.execute("SHOW DATABASES")
for x in mycursor:
  print(x)

# 创建数据表
mycursor.execute("CREATE TABLE customers (id INT AUTO_INCREMENT PRIMARY KEY, "
"name VARCHAR(255), address VARCHAR(255))")

# 查询数据表
mycursor.execute("SHOW TABLES")
for x in mycursor:
  print(x)
```

插入数据

```python
import mysql.connector

mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="root"
)

mycursor = mydb.cursor()

# 插入数据
sql = "INSERT INTO customers (name, address) VALUES (%s, %s)"
val = ("Tom", "China")
mycursor.execute(sql, val)
# 对插入操作进行提交,提交之后数据表才会发生变化
mydb.commit()
print(mycursor.rowcount, "record inserted.")
print("1 record inserted, ID:", mycursor.lastrowid)

# 插入多行数据
sql = "INSERT INTO customers (name, address) VALUES (%s, %s)"
val = [
  ('Peter', 'Lowstreet 4'),
  ('Amy', 'Apple st 652'),
  ('Hannah', 'Mountain 21'),
  ('Michael', 'Valley 345'),
  ('Sandy', 'Ocean blvd 2'),
  ('Betty', 'Green Grass 1'),
  ('Richard', 'Sky st 331'),
  ('Susan', 'One way 98'),
  ('Vicky', 'Yellow Garden 2'),
  ('Ben', 'Park Lane 38'),
  ('William', 'Central st 954'),
  ('Chuck', 'Main Road 989'),
  ('Viola', 'Sideway 1633')
]
mycursor.executemany(sql, val)
mydb.commit()
print(mycursor.rowcount, "was inserted.")
```

查询数据

```python
import mysql.connector

mydb = mysql.connector.connect(
  host="localhost",
  user="yourusername",
  passwd="yourpassword",
  database="mydatabase"
)
mycursor = mydb.cursor()
mycursor.execute("SELECT * FROM customers")
# fetchall 从最后执行的语句中获取所有行
myresult = mycursor.fetchall()
for x in myresult:
  print(x)

# 查询指定列的数据
mycursor.execute("SELECT name, address FROM customers")
myresult = mycursor.fetchall()
for x in myresult:
  print(x)

# 返回查询结果的第一行
mycursor.execute("SELECT * FROM customers")
myresult = mycursor.fetchone()
print(myresult)
```

删除数据

```python
import mysql.connector

mydb = mysql.connector.connect(
  host="localhost",
  user="yourusername",
  passwd="yourpassword",
  database="mydatabase"
)

mycursor = mydb.cursor()
sql = "DELETE FROM customers WHERE address = 'Mountain 21'"
mycursor.execute(sql)
# commit 提交修改
mydb.commit()
print(mycursor.rowcount, "record(s) deleted")

# 使用占位符 %s
sql = "DELETE FROM customers WHERE address = %s"
adr = ("Yellow Garden 2", )
mycursor.execute(sql, adr)
mydb.commit()
print(mycursor.rowcount, "record(s) deleted")
```

删除表

```python
import mysql.connector

mydb = mysql.connector.connect(
  host="localhost",
  user="yourusername",
  passwd="yourpassword",
  database="mydatabase"
)

mycursor = mydb.cursor()
sql = "DROP TABLE customers"
mycursor.execute(sql)
```

更新表

```python
import mysql.connector

mydb = mysql.connector.connect(
  host="localhost",
  user="yourusername",
  passwd="yourpassword",
  database="mydatabase"
)

mycursor = mydb.cursor()
sql = "UPDATE customers SET address = 'Canyon 123' WHERE address = 'Valley 345'"
mycursor.execute(sql)
mydb.commit()
print(mycursor.rowcount, "record(s) affected")

# 使用占位符 %s
sql = "UPDATE customers SET address = %s WHERE address = %s"
val = ("Valley 345", "Canyon 123")
mycursor.execute(sql, val)
mydb.commit()
print(mycursor.rowcount, "record(s) affected")
```



-----------------------------------------------------

[Reference >> PEP8]( https://pep8.org/ )

[Reference >> ]( https://www.runoob.com/python/python-built-in-functions.html )

[Reference >>]( https://www.w3school.com.cn/python/python_scope.asp )

