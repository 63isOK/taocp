# vol1, 1.3.2 习题包2.9

## 题目

要判断内存单元INST中,是否是一个有效的mixal指令.
如果是一个有效指令,跳转到GOOD处;否则跳到BAD处.

提示:有效指令是指address/index/mod/opcode都是有效的.
address如果在opcode中是一个地址,而且index=0,
此时address如果不是一个有效地址(0-3999),那么这个指令是无效的.
对于address的其他场景,都认为是有效的.

## 分析

书上有提示,要使用"路由表switching table"来进行"多路决策multiway decisions".

书上还推荐,使用一张路由表,64个入口(opcode是1byte,正好是64),
使用类似"LD1 C; LD1 TABLE,1; JMP 0,1"的写法.
实在是太棒了.

我们需要做的无非是对opcode细分,对mod,对address/index的一步步细分.
是不是可以用多个表来完成.

## 解答

这里只提思路,不写具体的代码,因为标准答案也不是一个安全标准的代码,
这题的考点是使用路由表来实现多路选择,说白了就是mixal中,
代替 if,else if, else if, 的实现.

针对这题:我自己的思路是:

- 提供一个路由表A,里面有64个元素
  - 这里面对应64种opcode
- 提供一个路由表B,里面有10个元素
  - 这里对应mod的10种取值范围
- index只有两种选择
  - 要么是0
  - 要么是1-6
  - 根据opcode和mod,可以判断index是否正确
- address,只判断index=0且address表示内存单元时
  - 判断M是否是有效地址

针对mixal的指令,还有一些其他的约束并没有表现出来,
eg:M在某些指令下,是可以忽略的.

