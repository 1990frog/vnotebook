
# if语法
```
x=5

if [$x=5];then
    echo "x equal 5"
else
    echo "x does not equal 5"
    exit 1
fi
```

# 退出状态
```
$ true
$ echo $?
0

$ false
$ echo $?
1
```

# test命令
## 文件表达式
评估文件的状态
[-e "$file"] file存在

""为了防备参数为null
##