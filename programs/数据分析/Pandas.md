[TOC]

在数据分析工作中，Pandas 的使用频率是很高的，一方面是因为 Pandas 提供的基础数据结构 DataFrame 与 json 的契合度很高，转换起来就很方便。另一方面，如果我们日常的数据清理工作不是很复杂的话，你通常用几句 Pandas 代码就可以对数据进行规整。Pandas 可以说是基于 NumPy 构建的含有更高级数据结构和分析能力的工具包。在 NumPy 中数据结构是围绕 ndarray 展开的，那么在 Pandas 中的核心数据结构是什么呢？

下面主要给你讲下 Series 和 DataFrame 这两个核心数据结构，他们分别代表着一维的序列和二维的表结构。基于这两种数据结构，Pandas 可以对数据进行导入、清洗、处理、统计和输出。

# 数据结构：Series 和 DataFrame
Series 是个定长的字典序列。说是定长是因为在存储的时候，相当于两个 ndarray，这也是和字典结构最大的不同。因为在字典的结构里，元素的个数是不固定的。

Series 有两个基本属性：index 和 values。在 Series 结构中，index 默认是 0,1,2,……递增的整数序列，当然我们也可以自己来指定索引，比如 index=[‘a’, ‘b’, ‘c’, ‘d’]。

```python
import pandas as pd
from pandas import Series, DataFrame
x1 = Series([1,2,3,4])
x2 = Series(data=[1,2,3,4], index=['a', 'b', 'c', 'd'])
print x1
print x2

0    1
1    2
2    3
3    4
dtype: int64
a    1
b    2
c    3
d    4
dtype: int64
```

这个例子中，x1 中的 index 采用的是默认值，x2 中 index 进行了指定。我们也可以采用字典的方式来创建 Series，比如：
```python
d = {'a':1, 'b':2, 'c':3, 'd':4}
x3 = Series(d)
print x3

a    1
b    2
c    3
d    4
dtype: int64
```
DataFrame 类型数据结构类似数据库表。

它包括了行索引和列索引，我们可以将 DataFrame 看成是由相同索引的 Series 组成的字典类型。

我们虚构一个王者荣耀考试的场景，想要输出几位英雄的考试成绩：
```python
import pandas as pd
from pandas import Series, DataFrame
data = {'Chinese': [66, 95, 93, 90,80],'English': [65, 85, 92, 88, 90],'Math': [30, 98, 96, 77, 90]}
df1= DataFrame(data)
df2 = DataFrame(data, index=['ZhangFei', 'GuanYu', 'ZhaoYun', 'HuangZhong', 'DianWei'], columns=['English', 'Math', 'Chinese'])
print df1
print df2
```

在后面的案例中，我一般会用 df, df1, df2 这些作为 DataFrame 数据类型的变量名，我们以例子中的 df2 为例，列索引是[‘English’, ‘Math’, ‘Chinese’]，行索引是[‘ZhangFei’, ‘GuanYu’, ‘ZhaoYun’, ‘HuangZhong’, ‘DianWei’]，所以 df2 的输出是：
```python
            English  Math  Chinese
ZhangFei         65    30       66
GuanYu           85    98       95
ZhaoYun          92    96       93
HuangZhong       88    77       90
DianWei          90    90       80
```

# 数据导入和输出

Pandas 允许直接从 xlsx，csv 等文件中导入数据，也可以输出到 xlsx, csv 等文件，非常方便。
```python
import pandas as pd
from pandas import Series, DataFrame
score = DataFrame(pd.read_excel('data.xlsx'))
score.to_excel('data1.xlsx')
print score
```

需要说明的是，在运行的过程可能会存在缺少 xlrd 和 openpyxl 包的情况，到时候如果缺少了，可以在命令行模式下使用“pip install”命令来进行安装。

# 数据清洗
例子
```python
data = {'Chinese': [66, 95, 93, 90,80],'English': [65, 85, 92, 88, 90],'Math': [30, 98, 96, 77, 90]}
df2 = DataFrame(data, index=['ZhangFei', 'GuanYu', 'ZhaoYun', 'HuangZhong', 'DianWei'], columns=['English', 'Math', 'Chinese'])
```

## 删除DataFrame中不必要的行或列
Pandas 提供了一个便捷的方法 drop() 函数来删除我们不想要的列或行。
```python
df2 = df2.drop(columns=['Chinese'])
df2 = df2.drop(index=['ZhangFei'])
```

## 重命名列名columns
如果你想对 DataFrame 中的 columns 进行重命名，可以直接使用 rename(columns=new_names, inplace=True) 函数，比如我把列名 Chinese 改成 YuWen，English 改成 YingYu。
```python
df2.rename(columns={'Chinese': 'YuWen', 'English': 'Yingyu'}, inplace = True)
```

## 去重复的值
```python
df = df.drop_duplicates() #去除重复行
```

## 格式问题
更改数据格式
```python
df2['Chinese'].astype('str') 
df2['Chinese'].astype(np.int64) 
```
数据间的空格
有时候我们先把格式转成了 str 类型，是为了方便对数据进行操作，这时想要删除数据间的空格，我们就可以使用 strip 函数：
```python
#删除左右两边空格
df2['Chinese']=df2['Chinese'].map(str.strip)
#删除左边空格
df2['Chinese']=df2['Chinese'].map(str.lstrip)
#删除右边空格
df2['Chinese']=df2['Chinese'].map(str.rstrip)
```
如果数据里有某个特殊的符号，我们想要删除怎么办？同样可以使用 strip 函数，比如 Chinese 字段里有美元符号，我们想把这个删掉，可以这么写：
```python
df2['Chinese']=df2['Chinese'].str.strip('$')
```
大小写转换
```python
#全部大写
df2.columns = df2.columns.str.upper()
#全部小写
df2.columns = df2.columns.str.lower()
#首字母大写
df2.columns = df2.columns.str.title()
```
查找空值
数据量大的情况下，有些字段存在空值 NaN 的可能，这时就需要使用 Pandas 中的 isnull 函数进行查找。比如，我们输入一个数据表如下：
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321762_20200830230520049_984966220.png)
如果我们想看下哪个地方存在空值 NaN，可以针对数据表 df 进行 df.isnull()，结果如下：
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321762_20200830230534270_16461040.png)
如果我想知道哪列存在空值，可以使用 df.isnull().any()，结果如下：
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321763_20200830230547012_21912450.png)

# 使用apply函数对数据进行清洗

apply 函数是 Pandas 中自由度非常高的函数，使用频率也非常高。比如我们想对 name 列的数值都进行大写转化可以用：

```python
df['name'] = df['name'].apply(str.upper)
```

我们也可以定义个函数，在 apply 中进行使用。比如定义 double_df 函数是将原来的数值 *2 进行返回。然后对 df1 中的“语文”列的数值进行 *2 处理，可以写成：
```python
def double_df(x):
           return 2*x
df1[u'语文'] = df1[u'语文'].apply(double_df)
```

我们也可以定义更复杂的函数，比如对于 DataFrame，我们新增两列，其中’new1’列是“语文”和“英语”成绩之和的 m 倍，'new2’列是“语文”和“英语”成绩之和的 n 倍，我们可以这样写：
```python
def plus(df,n,m):
    df['new1'] = (df[u'语文']+df[u'英语']) * m
    df['new2'] = (df[u'语文']+df[u'英语']) * n
    return df
df1 = df1.apply(plus,axis=1,args=(2,3,))
```
其中 axis=1 代表按照列为轴进行操作，axis=0 代表按照行为轴进行操作，args 是传递的两个参数，即 n=2, m=3，在 plus 函数中使用到了 n 和 m，从而生成新的 df。

# 数据统计
常用函数
表格中有一个 describe() 函数，统计函数千千万，describe() 函数最简便。它是个统计大礼包，可以快速让我们对数据有个全面的了解。下面我直接使用 df1.descirbe() 输出结果为：
```python
df1 = DataFrame({'name':['ZhangFei', 'GuanYu', 'a', 'b', 'c'], 'data1':range(5)})
print df1.describe()
```
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321764_20200901222429938_1561259650.png)

# 数据表合并
有时候我们需要将多个渠道源的多个数据表进行合并，一个 DataFrame 相当于一个数据库的数据表，那么多个 DataFrame 数据表的合并就相当于多个数据库的表合并。

比如我要创建两个 DataFrame：
```python
df1 = DataFrame({'name':['ZhangFei', 'GuanYu', 'a', 'b', 'c'], 'data1':range(5)})
df2 = DataFrame({'name':['ZhangFei', 'GuanYu', 'A', 'B', 'C'], 'data2':range(5)})
```
两个 DataFrame 数据表的合并使用的是 merge() 函数，有下面 5 种形式：
1. 基于指定列进行连接

比如我们可以基于 name 这列进行连接。
```python
df3 = pd.merge(df1, df2, on='name')
```
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321764_20200901222655097_1920940979.png)

2. inner 内连接
inner 内链接是 merge 合并的默认情况，inner 内连接其实也就是键的交集，在这里 df1, df2 相同的键是 name，所以是基于 name 字段做的连接：
```python
df3 = pd.merge(df1, df2, how='inner')
```
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321765_20200901222736585_1693087143.png)

3. left 左连接
左连接是以第一个 DataFrame 为主进行的连接，第二个 DataFrame 作为补充。
```python
df3 = pd.merge(df1, df2, how='left')
```
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321765_20200901222827541_224325874.png)

4. right 右连接
右连接是以第二个 DataFrame 为主进行的连接，第一个 DataFrame 作为补充。
```python
df3 = pd.merge(df1, df2, how='right')
```
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321765_20200901222910064_821715680.png)

5. outer 外连接
外连接相当于求两个 DataFrame 的并集。
```python
df3 = pd.merge(df1, df2, how='outer')
```
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321766_20200901222954128_1014629190.png)

# 如何用 SQL 方式打开 Pandas
Pandas 的 DataFrame 数据类型可以让我们像处理数据表一样进行操作，比如数据表的增删改查，都可以用 Pandas 工具来完成。不过也会有很多人记不住这些 Pandas 的命令，相比之下还是用 SQL 语句更熟练，用 SQL 对数据表进行操作是最方便的，它的语句描述形式更接近我们的自然语言。事实上，在 Python 里可以直接使用 SQL 语句来操作 Pandas。这里给你介绍个工具：pandasql。

```python
import pandas as pd
from pandas import DataFrame
from pandasql import sqldf, load_meat, load_births
df1 = DataFrame({'name':['ZhangFei', 'GuanYu', 'a', 'b', 'c'], 'data1':range(5)})
pysqldf = lambda sql: sqldf(sql, globals())
sql = "select * from df1 where name ='ZhangFei'"
print pysqldf(sql)
```

上面这个例子中，我们是对“name='ZhangFei”“的行进行了输出。当然你会看到我们用到了 lambda，lambda 在 python 中算是使用频率很高的，那 lambda 是用来做什么的呢？它实际上是用来定义一个匿名函数的，具体的使用形式为：

```python
 lambda argument_list: expression
```

这里 argument_list 是参数列表，expression 是关于参数的表达式，会根据 expression 表达式计算结果进行输出返回。


pysqldf = lambda sql: sqldf(sql, globals())
在这个例子里，输入的参数是 sql，返回的结果是 sqldf 对 sql 的运行结果，当然 sqldf 中也输入了 globals 全局参数，因为在 sql 中有对全局参数 df1 的使用。总结

# 总结
mPy 一样，Pandas 有两个非常重要的数据结构：Series 和 DataFrame。使用 Pandas 可以直接从 csv 或 xlsx 等文件中导入数据，以及最终输出到 excel 表中。我重点介绍了数据清洗中的操作，当然 Pandas 中同样提供了多种数据统计的函数。最后我们介绍了如何将数据表进行合并，以及在 Pandas 中使用 SQL 对数据表更方便地进行操作。Pandas 包与 NumPy 工具库配合使用可以发挥巨大的威力，正是有了 Pandas 工具，Python 做数据挖掘才具有优势。

---

# 生成数据
# 处理数据
# pivot 透视
以统计学生成绩信息为例。
在做学生成绩信息统计的时候，我们从学生各科考试成绩文件（.csv或.xls等）中把数据抽取上来。样本模拟数据（data_df）如下：

|     | userNum | score | subjectCode | subjectName | userName |
| --- | ------- | ----- | ----------- | ----------- | -------- |
| 0   | 001     | 90    | 01          | 语文        | 张三     |
| 1   | 002     | 90    | 01          | 语文        | 李四     |
| 2   | 003     | 90    | 01          | 语文        | 王五     |
| 3   | 004     | 90    | 02          | 数学        |   张三       |
| 4   | 005     | 90    | 02          | 数学        |   李四       |
| 5   | 006     | 90    | 02          | 数学        |    王五      |

要把上面二维表转换为每个人各科的成绩信息。就像咱们中学时期的成绩单一样。类似于下表的一张二维表。

| 学籍号 | 姓名 | 班级 | 语文成绩 | 语文排名 | 数学成绩 | 数学排名 |
| ----- | ---- | ---- | ------- | ------- | -------- | -------- |
| ...   | ...  | ...  | ...     | ...     | ...     | ...     |

我之前的传统统计方式，给data_df根据学籍号进行groupby，再循环遍历该分组得到每个人的各科成绩信息，再统计到一张新表中，然后循环append每一张新表，可生成以上的样表。如果我们需要统计全年级的学生呢？可能一个年级有500个学生，那就是循环500次。此时我们需要统计一个市区内多校联考的学生呢？岂不是要循环成百上千次？实际情况，这样的做法使得我们的脚本跑的非常的慢。

DataFrame.pivot(index=None, columns=None, values=None)
Parameters
+ index:str or object or a list of str,optional
+ columns:str or object or a list of str
+ values:str,object or a list of the previous,optional

第一个index是重塑的新表的索引名称是什么，
第二个columns是重塑的新表的列名称是什么，一般来说就是被统计列的分组，
第三个values就是生成新列的值应该是多少，如果没有，则会对data_df剩下未统计的列进行重新排列放到columns的上层。

Returns: DataFrame

```python
>>> pivot_df = data_df.pivot(index='userNum', columns='subjectCode', values='score')
>>> print pivot_df
subjectCode  01  02
userNum
001          90  87
002          96  82
003          93  80
```
这就生成了我们大致想要的样子了，之后可以再给pivot_df的列名进行调整，还有其整体样式的调整。
```python
# 这只是其中一个方式，如有更好的方式，不吝赐教~

# 列名称置空
pivot_df.columns.name = None
# 遍历每个学科对新表列名进行修改
data_df_G = data_df.groupby(["subjectCode"], as_index=False)
temp_count = 1
for index, subject_df in data_df_G:
    # 把成绩排名添加到各科成绩之后
    pivot_df.insert(temp_count, "rank_" + str(index), pivot_df[index].rank(ascending=False, method='min'))
    # 重命名各科成绩
    pivot_df.rename(columns={index: ("score_" + str(index))}, inplace=True)
    temp_count += 2
# 把userNum添加的列中
pivot_df['userNum'] = pivot_df.index
# 索引名称置空
pivot_df.index.name = None

temp_df = data_df.loc[:, ["userNum", "userName"]]
temp_df.drop_duplicates(inplace=True)
# 剩余列拼接
pivot_df = temp_df.merge(pivot_df, on="userNum", how="left")

print(pivot_df)
  userNum userName  score_01  rank_01  score_02  rank_02
0   001       张三      90        3        87        1
1   002       李四      96        1        82        2
2   003       王五      93        2        80        3
```



Examples
```python
>>> df = pd.DataFrame({'foo': ['one', 'one', 'one', 'two', 'two',
                           'two'],
                   'bar': ['A', 'B', 'C', 'A', 'B', 'C'],
                   'baz': [1, 2, 3, 4, 5, 6],
                   'zoo': ['x', 'y', 'z', 'q', 'w', 't']})
>>> df
    foo   bar  baz  zoo
0   one   A    1    x
1   one   B    2    y
2   one   C    3    z
3   two   A    4    q
4   two   B    5    w
5   two   C    6    t

>>> df.pivot(index='foo', columns='bar', values='baz')
      baz       zoo
bar   A  B  C   A  B  C
foo
one   1  2  3   x  y  z
two   4  5  6   q  w  t
```

# pivot_table 透视表


---

pandas中，这三种方法都是用来对表格进行重排的，其中stack()是unstack()的逆操作。某种意义上，unstack()方法和pivot()方法是很像的，主要的不同在于，unstack()方法是针对索引或者标签的，即将列索引转成最内层的行索引；而pivot()方法则是针对列的值，即指定某列的值作为行索引，指定某列的值作为列索引，然后再指定哪些列作为索引对应的值。因此，总结起来一句话就是：unstack()针对索引进行操作，pivot()针对值进行操作。但实际上，两者在功能往往可以互相实现。

unstack(self, level=-1, fill_value=None)、pivot(self, index=None, columns=None, values=None，对比这两个方法的参数，这里要注意的是，对于pivot()，如果参数values指定了不止一列作为值的话，那么生成的DataFrame的列索引就会出现层次索引，最外层的索引为原来的列标签；unstack()没有指定值的参数，会把剩下的列都作为值，即把剩下的列标签都作为最外层的索引，每个索引对应一个子表。

pivot()方法其实比较容易理解，就是指定相应的列分别作为行、列索引以及值。下面我们通过几张原理图详细说明stack()和unstack()，最后再通过一个具体的例子来对比stack()、unstack()和pivot()这三种方法。

先看stack()，如图。stack()是将原来的列索引转成了最内层的行索引，这里是多层次索引，其中AB索引对应第三层，即最内层索引。


---

![](_v_images/20201015142026466_11271.png)

# stack 堆叠

# unstack

# columns 属性
重新设置列信息

---

# 基础
# DataFrame
DataFrame是一种表格型数据结构，它含有一组有序的列，每列可以是不同的值。DataFrame既有行索引，也有列索引，它可以看作是由Series组成的字典，不过这些Series公用一个索引。DataFrame的创建有多种方式，不过最重要的还是根据dict进行创建，以及读取csv或者txt文件来创建

## 根据字段创建
```python
data = {
    'state':['Ohio','Ohio','Ohio','Nevada','Nevada'],
    'year':[2000,2001,2002,2001,2002],
    'pop':[1.5,1.7,3.6,2.4,2.9]
}
frame = pd.DataFrame(data)
frame

#输出
    pop state   year
0   1.5 Ohio    2000
1   1.7 Ohio    2001
2   3.6 Ohio    2002
3   2.4 Nevada  2001
4   2.9 Nevada  2002
```

DataFrame的行索引是index，列索引是columns，我们可以在创建DataFrame时指定索引的值

```python
frame2 = pd.DataFrame(data,index=['one','two','three','four','five'],columns=['year','state','pop','debt'])
frame2

#输出
    year    state   pop debt
one 2000    Ohio    1.5 NaN
two 2001    Ohio    1.7 NaN
three   2002    Ohio    3.6 NaN
four    2001    Nevada  2.4 NaN
five    2002    Nevada  2.9 NaN
```
使用嵌套字典也可以创建DataFrame，此时外层字典的键作为列，内层键则作为索引:
```python
pop = {'Nevada':{2001:2.4,2002:2.9},'Ohio':{2000:1.5,2001:1.7,2002:3.6}}
frame3 = pd.DataFrame(pop)
frame3
#输出
    Nevada  Ohio
2000    NaN 1.5
2001    2.4 1.7
2002    2.9 3.6
```

我们可以用index，columns，values来访问DataFrame的行索引，列索引以及数据值，数据值返回的是一个二维的ndarra

```python
frame2.values
#输出
array([[2000, 'Ohio', 1.5, 0],
       [2001, 'Ohio', 1.7, 1],
       [2002, 'Ohio', 3.6, 2],
       [2001, 'Nevada', 2.4, 3],
       [2002, 'Nevada', 2.9, 4]], dtype=object)
```

# 读取文件
读取文件生成DataFrame最常用的是read_csv,read_table方法

# DataFrame轴
在DataFrame的处理中经常会遇到轴的概念，这里先给大家一个直观的印象，我们所说的axis=0即表示沿着每一列或行标签\索引值向下执行方法，axis=1即表示沿着每一行或者列标签模向执行对应的方法。
![](_v_images/20201015145623123_491.png =720x)

# DataFrame一些性质
