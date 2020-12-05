7.Awk循环语句
while循环: while(条件表达式)动作
[root@oldxu ~]# awk 'BEGIN{ i=1; while(i<=10){print i; i++} }'
[root@oldxu ~]# awk -F: '{i=1; while(i<=NF){print $i; i++}}' /etc/passwd
[root@oldxu ~]# awk -F: '{i=1; while(i<=10) {print $e; i++}}' /etc/passwd
[root@oldxu ~]#cat b. txt
111 222
333444 555
666 777 888 999
[root@oldxu ~]# awk '{i=1; while(i<=NF){print $i; i++}}' b. txt
for循环: for(初始化计数器;计数器测试;计数器变更动作



1.Awk数组概述
1.什么是awk数组
数组其实也算是变量，传统的变量只能存储一个值 ,但数组可以存储多个值。
2.awk数组应用场景
通常用来统计、比如:统计网站访问TOP10、网站Url访问TOP10等等等
3.awk数组统计技巧
1.在awk中,使用数组时,不仅可以使用1 2 3.n作为数组下标,也可以使用字符串作为数组下
标。
N
2.要统计某个字段的值,就将该字段作为数组的索引,然后对索引进行遍历。
4.awk数组的语法
array_ name[index]=value
5.awk数组示例
1.统计letc/passwd中各种类型shell的数量。