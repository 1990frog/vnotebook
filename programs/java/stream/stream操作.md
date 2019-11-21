# stream操作

stream包含中间（intermediate operations）和最终（terminal operation）两种形式的操作。  

+ 中间操作（intermediate operations）的返回值还是一个stream，因此可以通过链式调用将中间操作（intermediate operations）串联起来。  
+ 最终操作（terminal operation）只能返回void或者一个非stream的结果。在上述例子中：filter, map ，sorted是中间操作，而forEach是一个最终操作。  

