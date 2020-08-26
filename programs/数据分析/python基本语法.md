[TOC]

当然除了 Python 自身的特点，Python 还有强大的开发者工具。在数据科学领域，Python 有许多非常著名的工具库：比如科学计算工具 NumPy 和 Pandas 库，深度学习工具 Keras 和 TensorFlow，以及机器学习工具 Scikit-learn，使用率都非常高。

# 输入输出
```python
name = raw_input("What's your name?")
sum = 100+100
print ('hello,%s' %name)
print ('sum = %d' %sum)
```
raw_input 是 Python2.7 的输入函数，在 python3.x 里可以直接使用 input，赋值给变量 name，print 是输出函数，%name 代表变量的数值，因为是字符串类型，所以在前面用的 %s 作为代替。
# 判断语句
```python

if score>= 90:
       print 'Excellent'
else:
       if score < 60:
           print 'Fail'
       else:
           print 'Good Job'
```
if … else … 是经典的判断语句，需要注意的是在 if expression 后面有个冒号，同样在 else 后面也存在冒号。另外需要注意的是，Python 不像其他语言一样使用{}或者 begin…end 来分隔代码块，而是采用代码缩进和冒号的方式来区分代码之间的层次关系。所以代码缩进在 Python 中是一种语法，如果代码缩进不统一，比如有的是 tab 有的是空格，会怎样呢？会产生错误或者异常。相同层次的代码一定要采用相同层次的缩进。
# 循环
## for
```python
sum = 0
for number in range(11):
    sum = sum + number
print sum
```
for 循环是一种迭代循环机制，迭代即重复相同的逻辑操作。如果规定循环的次数，我们可以使用 range 函数，它在 for 循环中比较常用。range(11) 代表从 0 到 10，不包括 11，也相当于 range(0,11)，range 里面还可以增加步长，比如 range(1,11,2) 代表的是[1,3,5,7,9]。

## while
```python
sum = 0
for number in range(11):
    sum = sum + number
print sum
```
1 到 10 的求和也可以用 while 循环来写，这里 while 控制了循环的次数。while 循环是条件循环，在 while 循环中对于变量的计算方式更加灵活。因此 while 循环适合循环次数不确定的循环，而 for 循环的条件相对确定，适合固定次数的循环。

# 数据类型：列表、元组、字典、集合
## 列表：[]
```python
lists = ['a','b','c']
lists.append('d')
print lists
print len(lists)
lists.insert(0,'mm')
lists.pop()
print lists
```
列表是 Python 中常用的数据结构，相当于数组，具有增删改查的功能，我们可以使用 len() 函数获得 lists 中元素的个数；使用 append() 在尾部添加元素，使用 insert() 在列表中插入元素，使用 pop() 删除尾部的元素。

## 元组 (tuple)
```python
tuples = ('tupleA','tupleB')
print tuples[0]
```
元组 tuple 和 list 非常类似，但是 tuple 一旦初始化就不能修改。因为不能修改所以没有 append(), insert() 这样的方法，可以像访问数组一样进行访问，比如 tuples[0]，但不能赋值。

## 字典 {dictionary}
```python
# -*- coding: utf-8 -*
#定义一个dictionary
score = {'guanyu':95,'zhangfei':96}
#添加一个元素
score['zhaoyun'] = 98
print score
#删除一个元素
score.pop('zhangfei')
#查看key是否存在
print 'guanyu' in score
#查看一个key对应的值
print score.get('guanyu')
print score.get('yase',99)
```
字典其实就是{key, value}，多次对同一个 key 放入 value，后面的值会把前面的值冲掉，同样字典也有增删改查。增加字典的元素相当于赋值，比如 score[‘zhaoyun’] = 98，删除一个元素使用 pop，查询使用 get，如果查询的值不存在，我们也可以给一个默认值，比如 score.get(‘yase’,99)。
## 集合：set
```python
s = set(['a', 'b', 'c'])
s.add('d')
s.remove('b')
print s
print 'c' in s
```
集合 set 和字典 dictory 类似，不过它只是 key 的集合，不存储 value。同样可以增删查，增加使用 add，删除使用 remove，查询看某个元素是否在这个集合里，使用 in。

# 注释：#
注释在 python 中使用 #，如果注释中有中文，一般会在代码前添加 # -- coding: utf-8 -。如果是多行注释，使用三个单引号，或者三个双引号，比如：
```python
# -*- coding: utf-8 -*
'''
这是多行注释，用三个单引号
这是多行注释，用三个单引号 
这是多行注释，用三个单引号
'''
```

# 引用模块 / 包：import
```python
# 导入一个模块
import model_name
# 导入多个模块
import module_name1,module_name2
# 导入包中指定模块 
from package_name import moudule_name
# 导入包中所有模块 
from package_name import *
```
Python 语言中 import 的使用很简单，直接使用 import module_name 语句导入即可。这里 import 的本质是什么呢？import 的本质是路径搜索。import 引用可以是模块 module，或者包 package。针对 module，实际上是引用一个.py 文件。而针对 package，可以采用 from … import …的方式，这里实际上是从一个目录中引用模块，这时目录结构中必须带有一个 __init__.py 文件。

# 函数：def
```python
def addone(score):
   return score + 1
print addone(99)
```
函数代码块以 def 关键词开头，后接函数标识符名称和圆括号，在圆括号里是传进来的参数，然后通过 return 进行函数结果得反馈。

