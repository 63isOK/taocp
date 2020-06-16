# vol1, 1.3.2 习题包1.3

## 题目

书中例子有个查找最后最大数的子程序,
结合下面的代码,整个程序的效果是什么?

    START     IN    X+1(0)
              JBUS  *(0)
              ENT1  100
    1H        JMP   MAXMUM
              LDX   X,1
              STA   X,1
              STX   X,2
              DEC1  1
              J1P   1B
              OUT   X+1(1)
              HLT
              END   START

## 分析

任何一个非闭环的问题,学习起来都是非常麻烦的.
单独看查找最后最大数的算法过程,很清晰;
看mixal的翻译,就有很多疑问,特别是对初学者.

如果将算法的子程序和主程序(带END的程序)一起看,
就很清晰了.

下面先看子程序,反复研究多次,解决掉每一个未知点.

    * max.mixal : find lastest max number fo a array
    *
    * algorithm M
    * 1.[Initialize.] k=n,j=n,m=X[n],k=n-1.
    * 2.[All tested?] return; if k>0, To M3.
    * 3.[Compare.] if m >= X[k], To M4. others, To M5.
    * 4.[Change m.] j=k,m=X[k],To M5.
    * 5.[Decrease k.] k=k-1. To M2.
    *
    * convention:
    * rI1 = n
    * rI2 = j
    * rI3 = k
    * rA = m, (X[k])
    *
    * label   ins   operand     comment
    X         EQU   1000        address of array
              ORIG  3000
    MAXNUM    STJ   EXIT        call sub function
    INIT      ENT3  0,1         k=n.(n is input param of MAXNUM)
              JMP   CHANGEM
    LOOP      CMPA  X,3         compare X[k] and X[k-1]
              JGE   *+3         if X[k]>X[k-1],compare X[k] and X[k-2]
    CHANGEM   ENT2  0,3         j=k, or find a new biggest number
              LDA   X,3         m=X[k]
              DEC3  1           k=k-1
              J3P   LOOP        if k > 0, start compare,find the biggest
    EXIT      JMP   *

先要弄明白以下几点:

- 这是一段子程序
  - 会有入参,数组长度n.ENT3 0,1 就是rI3=rI1,rI1就是在主程序中设置的
- 输出
  - 最大数存在rA中,索引放在rI2中

回到题目,给源码添加上注释.

    * call MAXNUM

    START     IN    X+1(0)      u0, word block is 100words
              JBUS  *(0)        wait until u0 is ready
              ENT1  100         set n=100
    1H        JMP   MAXNUM      call MAXNUM
              LDX   X,1         rX=X[n]
              STA   X,1         X[n]=rA
              STX   X,2         X[j]=rX
              DEC1  1           n--
              J1P   1B          loop,change every item
              OUT   X+1(1)      output: X[1...n]
              HLT               end
              END   START

知道了MAXNUM再读题目就比较简单了.

依次查找MAXNUM(n)中的最大数,然后和`X[n]`进行交换,
n从起初的100,到最后的1,这是一个典型的排序,正序.

可惜没有设备0,无法真正运行程序.

## 解答

最后的效果是:按正序输出X数组.
