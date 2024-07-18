[学习Redis的CSDN博客](https://blog.csdn.net/m0_55993923/article/details/129718974)









### 介绍




### Redis集群




这时候加入一个
``` shell 
# mget mset 在集群中使用的话，由于存储位置 可能 不在同一个redis 会导致 这个命令执行失败
mset k1{name} v1 k2{name} v2 //这种情况下执行会成功 会把他们存储在同一个槽位对应的redis下
mget k1{name} k2{name}


```




### BigKEY

禁用一些危险指令

>redis.conf

``` shell

rename-command keys ""   // 禁用keys
rename-command flushdb   //
rename-command flushall  //

```



``` java

 scan  cursor [mathch pattern] [COUNT count]   //  遍历数据
 
 redis-cli --bigkeys   // 查看计算当前有没有大key


memry usage  // 计算每个key占用的字节数


 

```
















