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
