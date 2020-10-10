# Bitmap（位图）

![](https://gitee.com/caijingquan/imagebed/raw/master/1602320126_20191126215616213_15912252.png)

setbit key offset value
给位图指定索引设置值

getbit key offset
获取位图指定索引位置

bitcount key [start end]
获取位图指定范围（start到end，单位为字节，如果不指定就是获取全部）位值为1的个数

位图的简单使用：
1.使用set和bitmap
2.1亿用户，5千万独立

![](https://gitee.com/caijingquan/imagebed/raw/master/1602320128_20191126220526666_497178824.png)

![](https://gitee.com/caijingquan/imagebed/raw/master/1602320129_20191126220750184_1272126802.png)

使用经验
type=string，最大512mb
注意setbit时的偏移量，可能有较大耗时
redis单线程