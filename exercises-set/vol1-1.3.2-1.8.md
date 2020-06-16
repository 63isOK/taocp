# vol1, 1.3.2 习题包1.8

## 题目

手动分析下列代码的意思

    BUF   ORIG    *+3000
    1H    ENT1    1                 m=1
          ENT2    0                 n=0
          LDX     4F                rX=AAAAA
    2H    ENT3    0,1               k=m
    3H    STZ     BUF,2             BUF+n,set zero
          INC2    1                 n++
          DEC3    1                 k--
          J3P     3B                if k>0, go to 3B
          STX     BUF,2             BUF+n,set rX
          INC2    1                 n++
          INC1    1                 m++
          CMP1    =75=              compare m and 75
          JL      2B                if m < 75, go to 2B
          ENN2    2400              n = -2400
          OUT     BUF+2400,2(18)    BUF+2400+n,output to u18
          INC2    24                n+=24
          J2N     *-2               if n<0, go to OUT again
          HLT
    4H    ALF     AAAAA
          END     1B

## 分析

- 这是一个程序,包含了END
- 这里用到了3个索引寄存器,rI1到rI3,我们用mnk来表示
  - m=rI1
  - n=rI2
  - k=rI3
- 给所有指令添加注释

即使给每条指令添加上注释,但还是不能一眼看出其功能.
列出每一个疑问点:

- J3P 3B及之前的指令是什么意思
  - 分析发现,k的初始值是m(初始值为1),在J3P之前,有一个k--
  - 所以J3P永远不会被触发
  - J3P之后,状态是 m=1,n=1,k=0
  - 这是这部分源码执行一次之后的结果
  - STZ BUF,2  这个执行之后,之前的旧指令变为NOP了
- 观察JL 2B之前,将m改为了75,n和k的值在后面都没有用上
  - 而且JL 2B,实际上没做什么操作,STZ BUF,2 会将很多指令变为NOP
- 最后只剩一个打印
  - u18,一次OUT是24字,打印的就是代码的汇编数据
  - 打印迭代的次数是100次,此时早就超过了3999
  - 猜测OUT指令如遇到超出3999的场景,应该是不处理

这样,最后的结果应该是打印汇编,从程序开始处,打印到3999

## 解答

最后的结果应该是打印汇编,从程序开始处,打印到3999.
汇编中,有几行是被设置为NOP,有几行是被设置为AAAAA.
