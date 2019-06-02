# map() 和 foreach() 的区别

foreach(func) 是一个 action 算子，会把函数作用在 rdd 的每个数据上，但是它不会修改 rdd，也不会生成一个新的 RDD。

map(func) 是一个transaction 算子，会把函数作用在 rdd 的每个数据上，并且会返回一个新的 rdd。

