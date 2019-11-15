[TOC]

![vim](_v_images/20190816153331073_6254123.png)

# 模式
+ 命令模式(command-mode)
+ 插入模式(insert-mode)
+ 可视模式(visual-mode)
+ 正常模式(normal-mode) 


# Normal模式：

## 移动
### 屏幕滚动
ctrl+f/b：向前翻一页/向后翻一页  
ctrl+d/u：向后翻半页/向前翻半页  
ctrl+e/y：向后一行/向前一行  

#### 行内搜索/移动：
##### 移动到检索字符：
f{char}：可以移动到char字符上，t移动到char的前一个字符，如果第一次没搜到，可以用（分号;）或者（逗号,）继续搜该行下一个/上一个  
F{char}：表示反过来搜前面的字符  

##### 光标移动：
h向左移动,[n]h向左移动n个字符  
h向左移动,[n]h向左移动n个字符  
j向下移动  
k向上移动  
l向右移动  

w/W：移到下一个word/WORD开头（WORD以空白符分割的单词）  
e/E：移到下一个word/WORD尾部  
b/B：移到上一个word/WORD开头  

##### 瞬间移动：
ctrl+o 快速返回  
gg：移动到文件开头  
zz把当前位置设置为屏幕中间  
H/M/L 跳转到屏幕的开头head，中间middle，结尾lower  
G：移动到文件结尾  
[n]G快速移动到指定行n上  
gi:快速跳转到你最后一次编辑的地方并进入插入模式  
0：移动到行首的第一个字符  
^：移动到第一个非空白字符  
$：移动到行尾，可搭配[n]使用  
g_：移动到行尾非空白字符，可搭配[n]使用  
%：移动到与与制匹配的括号上去(),{},[],<>等  

Space：光标右移一个字符  
Backspace：光标坐移一个字符  
Enter：光标下移一行  

(:光标移至句首  
):光标移至句尾  
{：光标移至段落开头  
}：光标移至段落结尾  

##### 操作：

i:在光标所在字符前开始插入  
I:在光标所在行的行首开始插入 如果行首有空格则在空格之后插入在空格之后插入  

a:在光标所在字符后开始插入  
A:在光标所在你行的行尾开始插入  

o:在光标所在行的下面另起一新行插入  
O:在光标所在行的上面另起一行开始插入  

s:删除光标所在的字符并开始插入  
S:删除光标所在行并开始插入  

r:替换当前字符  
R:替换当前字符及其后的字符，直至ESC键  

ncw/nCW：修改指定数目的word  
nCC：修改指定数目的行  

dd：删除当前行  
u：撤销上一步操作  
ctrl+r：恢复上一步被撤销的操作  

### 增删改查

数字+命令标示多次执行命令  

#### 删除：
vim在normal模式下使用x快速删除一个字符  
使用d(delete)  
配合文本对象快速删除一个单词daw（d around word）  
配合文本对象快速删除一个单词diw 不删除空格  
d+$当前位置删除到行尾  
d+0当前位置删除到行首  
2dd 删除两行  
4x删除4个字符  
v选择字符，后按d删除选中区域  
V行选  

#### 修改：
r（replace）R  
c（change）C  
    ct  
    cw  
s（substitute）S  
4s删除4个字符  
normal模式下使用r可以替换一个字符。s替换并进入插入模式，使用c配合文本对象，我们可以快速进行修改  

#### 查询：
使用/或者?进行向前或者反向搜索  
使用n/N跳转到下一个或者上一个匹配  
使用*或者#进行当前单词的向前和向后匹配  

#### 替换：
substritute命令允许我们查找并且替换文本，并且支持正则表达式  
:[range]s[ubstitute]/{pattern}/{string}/[flags]  
range表范围，比如:10,20表示10-20行，%表示全部  
pattern是要替换的模式，string是替换后文本  
flages有几个常用的标志：  
g(global)表示全局范围内执行  
c(confirm)表示确认，可以确认或者拒绝修改  
n(number)报告匹配到的次数而不替换，可以用来查询匹配次数  


Insert模式：
+ ctrl+h删除上一个字符
+ ctrl+w删除上一个单词
+ ctrl+u删除当前行

Command模式（一些常用的Vim配置，在~/.vimrc中）：
+ :w保存
+ :q退出
+ :wq!保存并退出
+ :vs竖分屏
+ :sp横分屏
+ :% s/foo/bar/g 全局替换
+ :syntax on 支持语法高亮
+ :set nu 显示行号
+ :set nonu 不显示行号
+ :set ai 设置自动缩进
+ :set shiftwidth=4 设置自动缩进 4 个空格, 当然要设自动缩进先.
+ :set sts=4 即设置 softtabstop 为 4. 输入 tab 后就跳了 4 格.
+ :set tabstop=4 实际的 tab 即为 4 个空格, 而不是缺省的 8 个.
+ :set expandtab 在输入 tab 后, vim 用恰当的空格来填充这个 tab.
+ :set hls 打开搜索高亮
+ :set incserach 边搜索边高亮
+ :set nohls 取消搜索高亮 
+ :set list ： 显示特殊字符
+ :set nolist 

Visual模式（可视化模式）：
+ Normal模式下使用v进入visual选择  
+ 使用V选择行
+ 使用ctrl+v进行方块选择
+ d:删除
+ y:复制

快速切换insert和normal模式：ctrl+c，ctrl+[,ESC  

Vim多文件操作（Buffer、Window、Tab）  
Buffer是指打开的一个文件的内存缓冲区  
窗口是Buffer可视化的分割区域  
Tab可以组织窗口为一个工作区  

Buffer-什么是缓冲区？  
Vim打开一个文件后会加载文件内容到缓冲区  
之后的修改都是针对内存中的缓冲区，并不会直接保存到文件  
直到我们执行:w(write)的时候才会把修改内容写入到文件里  

如何在Buffer之间切换？  
1.使用:ls会列举当前缓冲区，然后使用:b n跳转到第n个缓冲区  
2.:bpre :bnext :bfirst :blast  
3.或者用:b buffer_name 加上tab补全来跳转  
 :e filename 打开当前目录下其他文件  

Window窗口  
窗口是可视化的分割区域  
1.一个缓冲区可以分割成多个窗口，每个窗口也可以打开不同缓冲区。  
2.<Ctrl+w>s水平分割，<Ctrl+w>v垂直分割。或:sp和:vs  
3.每个窗口可以继续被无限分割  

切换窗口的命令都是使用Ctrl+w（window）作为前缀  
+ <C-w>w 在窗口间循环切换
+ <C-w>h 切换到左边的窗口
+ <C-w>j 切换到下边的窗口
+ <C-w>k 切换到上边的窗口
+ <C-w>l 切换到右边的窗口

+ <C-w>H 移动窗口到左边
+ <C-w>J 移动窗口到下边
+ <C-w>K 移动窗口到上边
+ <C-w>L 移动窗口到右边

+ <C-w>=使所有窗口等宽、等高
+ <C-w>_最大化活动窗口的高度
+ <C-w>|最大化活动窗口的宽度
+ [N]<C-w>_把活动窗口的高度设为[N]行
+ [N]<C-w>|把活动窗口的宽度设为[N]列

Tab:
+ :tabe[dit] {filename} 在新标签页中打开{filename}
+ <C-w>T 把当前窗口移动到一个新标签页
+ :tabc[lose] 关闭当前标签页及其中的索爷窗口
+ :tabo[nly] 只保留活动标签页，关闭所有其他标签页
+ :tabn[ext]{N} {N}gt 切换到编号为{N}的标签页
+ :tabn[ext] gt 切换到下一标签页
+ :tabp[revious] gT切换到上一标签页

Text Object(文本对象)
Vim里文本页游对象的概念，比如一个单词，一段句子，一个段落  
很多其他编辑器经常只能操作单个字符来修改文本，比较低效  
通过操作文本对象来修改要比只操作单个字符高效  

[number]<command>[text object]
number表示次数  
command是命令，d[elete]、c[hange]、y[ank]  
text object是要操作的文本对象，比如单词w，句子s，段落p  


ci', di', yi'：修改、剪切或复制'之间的内容。  
ca', da', ya'：修改、剪切或复制'之间的内容，包含'。  
ci", di", yi"：修改、剪切或复制"之间的内容。  
ca", da", ya"：修改、剪切或复制"之间的内容，包含"。  
ci(, di(, yi(：修改、剪切或复制()之间的内容。  
ca(, da(, ya(：修改、剪切或复制()之间的内容，包含()。  
ci[, di[, yi[：修改、剪切或复制[]之间的内容。  
ca[, da[, ya[：修改、剪切或复制[]之间的内容，包含[]。  
ci{, di{, yi{：修改、剪切或复制{}之间的内容。  
ca{, da{, ya{：修改、剪切或复制{}之间的内容，包含{}。  
ci<, di<, yi<：修改、剪切或复制<>之间的内容。  
ca<, da<, ya<：修改、剪切或复制<>之间的内容，包含<>。  

ci = change inner  

常见4个命令  
d删除，c改变，v选择,y复制  

Vim复制粘贴与寄存器的使用  

Vim Normal模式复制粘贴分别使用y(yank)和p(put)，剪贴d和p  
我们可以使用v(visual)命令选中所要复制的地方，然后使用p粘贴  
配合文本对象：比如使用yiw复制一个单词，yy复制一行  

os中剪贴(cut)复制(copy)粘贴(paste)  
vim中分别为delete/yank/put  

Insert模式下复制粘贴  
Ctrl+c   
Ctrl+v  
很多人在vimrc中设置了autoindent，粘贴python代码缩进错乱，这个时候需要使用:set paste和:set nopaste解决  


vim寄存器  
vim里操作的是寄存器而不是系统剪切板，这和其他编译器不同  
默认我们使用d删除或者y复制的内容都放到了“无名寄存器”  
小技巧：  
用x删除一个字符放到无名寄存器，然后p粘贴，可以调换俩字符  

深入寄存器（register）
vim不使用单一剪切板进行剪切、复制与粘贴，而是多组寄存器
1.通过"{register}前缀可以指定寄存器，不指定默认用无名寄存器
2.比如使用"ayiw复制一个单词到寄存器a中，"bdd删除当前行到寄存器b中
:reg a 查看a寄存器
:reg b 查看b寄存器
vim中 ""表示无名寄存器，缺省使用。"" p其实就等同p

除了有名寄存器a-z，vim中还有一些其他常见寄存器
复制专用寄存器"0使用y复制文本同时会被拷到复制寄存器0
系统剪切板“+可以在复制前加上 "+ 复制到系统剪切板
其他一些寄存器比如 "% 当前文件名，".上次插入的文本

:echo has('clipboard') 查看是否有系统剪切板
:set clipboard = unnamed 可以让你直接复制粘贴内容到系统剪切板


Vim宏（macro）
宏可以看成是一系列命令的集合
我们可以使用宏[录制]一系列操作，然后用于[回放]
宏可以非常方便的把一系列命令用在多行文本上

vim使用q来录制，同时也是q结束录制
使用q{register}回访寄存器中保存的一系列命令

visual全选 normal @a


vim中补全
<C-n>普通关键字
<C-x><C-n>当前缓冲区关键字
<C-x><C-i>包含文件关键字
<C-x><C-]>标签文件关键字
<C-x><C-k>字典查找
<C-x><C-l>整行补全ss
<C-x><C-f>文件名补全
<C-x><C-o>全能（Omni）补全

vim中常用补全
使用ctrl+n和ctrl+p补全单词
使用ctrl+x ctrl+f补全文件吗
使用ctrl+x ctrl+o补全代码，需要开启文件类型检查，安装插件


vim更换配色
使用:colorscheme显示当前的主题配色，默认是default
用:colorscheme <ctrl+d> 可以显示所有配色
用:colorscheme 配色名 就可以修改配色

vim配色仓库：https://github.com/flazz/vim-colorschemes
