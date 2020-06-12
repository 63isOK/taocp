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

## 解答
