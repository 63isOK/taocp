# vol1, 1.3.2 习题包2.10

## 题目

有一个9x8的矩阵

    a11 a12 ... a17 a18
    ...             ...
    a91 a92 ... a97 a98

用mixal存,可以这么存: aij = 1000 + 8i + j

    1009 1010 ... 1015 1016
    ...                ...
    1073 1074     1079 1080

saddle point,叫鞍点,在数学的很多方面都有应用,
在矩阵中,特指行中最小列中最大的某个元素.
鞍点满足以下条件:

- aij = min(aik) k取值为1到8,行中最小
- aij = max(akj) k取值为1到9,列中最大

那么问题是:写一个程序来查找矩阵的鞍点,
如果没有鞍点,返回0,如果有鞍点,返回第一个鞍点的位置,
返回值放在rI1中.

## 分析

常规思路,行中最小列中最大,可以遍历行,
找到最小值,判断这个值在列中是不是最大.
这种思路,遍历列也是一样的.

习题中给出提示,使用索引寄存器rIi来实现二维数组.

对于解题,考虑到矩阵的输入,用一个生产函数来代替读设备,
用查找函数来作为题目的解,用主函数来串联两函数,
起到调试的作用.

在编写查找函数的过程中,发现动手写代码,真不如在纸上画,
至少要先画出整个流程,毕竟各种跳转还是很容易出错的.
那么有没有一种方法,让写出的代码有较强的可读性?

这点在后面的学习中,会重点关注.

当我在纸上写出整个计算流程时,将其翻译成mixal时还是花了很长时间.
可能是应为1.3.1的习题没有刷,导致很多常用的写法没有很深的认识,
写完之后,对mixal的指令的理解也更加深入了.

编译过程:

- 函数不能放在主函数后面,会包符号找不到
- 函数的如何使用和限制,还需要进一步学习,已将代码改为单函数模式
- ORIG 前面不能放标签,START标签应该放在mixal指令上
- MUL等指令,操作对象是V,如果要传立即数,要用字面量 =N=
- 调试时,不能下断在START,在START的下一条是可以下断的
- 要将rAX存到内存单元怎么办
  - 问题是内存单元的编号是变化的
  - 可以先将内存单元编号放到index寄存器
  - STX 0,1 再利用这种方式来修改内存单元的值
  - w-exp表达式是否可以解决这种问题?值得探索

修改了诸多错误之后,发现运行结果是错的,
应该是某些基础逻辑有问题,下一步,手动创建数据,将矩阵变小,再来单步调试.

现在终于调试通过了,通过这个,对mixal了解了更多.
后续需要补上总结部分.

## 解答

具体编码看mixal源码文件

## 总结

这是对这次解决问题的总结.

现在列出M不是内存地址,而是当成数字的指令类型:

- 地址转移操作,ENTA/INCA/DECA
- 输入输出操作,M是跟着指令走的,不是内存地址,也不是随意的数字
- 位移操作,M表示的位移的位数
- MOVE指令中的M表示要拷贝的字数

注意跳转操作,不涉及M.转换操作不涉及M.

## 书本答案理解

问题的核心:矩阵的行列中,最大最小值是潜在的鞍点.

## 解法1

解法1的思路和我们自己写的思路很类似:

遍历每一行,找出行中最小元素,看看在其列上是否是最大.

"ENT3 0", "INC3 1" 连续的这种写法,非常适合循环中,
这样跳转放在INC3 1这行.

这个题目中鞍点是指极值点,需要注意的是一行可能存在多个极值.
如果某一行所有元素相等,那么这行所有元素都是潜在的鞍点.

解法1中,将每一行的所有最小数全部找出来,rI3,也就是代码中的k,
k表示最小数的个数,而且还用内存来存储每个最小数的列索引,
也就是存储c.

最巧的是,利用124H来对应大于/等于/小于.设计的非常巧妙.

rowmin,找行中最小,还是非常优雅的.找列中最大,写法非常简洁,
这些和后面的比起来就稍显逊色.

mixal的写法和c++/go的写法思路完全不一样,
不需要子程序的概念,直接使用跳转.
行中最小,是一个代码片段,只有一个跳出出口,就是J2Z COLMAX,
列中最大,有两个出口:要么是鞍点,要么不是鞍点,
是鞍点跳到YES,不是鞍点跳到NO,在NO中决定是继续跳到处理下一个最小值,
还是跳到行中最小来处理下一行.

这是第一次摆脱高级语言的"函数"特征,体会到机器语言的美丽和优雅.

而且这个写法的代码量,只是我写出来的一半.
这个写法避免了除法,每个小模块的写法都有套路:
前面是初始化,紧跟迭代点,最后面是出口点.

下面是代码:

    * find saddle point of matrix
    *
    * convention:
    * rX = current min
    * rI1 = address of check item ([72...0])
    * rI2 = column index of rI1
    * rI3 = size of list of minima
    *
    * terminating condition: rIx <= 0
    *
    * label   ins   operand     comment
    A10       EQU   1008        address of a10
    LIST      EQU   1000

    START     ENT1  9*8         rI1(m) m=72,"A10,1" is address of item
    ROWMIN    ENT2  8           rI2(c) c=8, column index of m
    2H        LDX   A10,1       rX=content(m)
              ENT3  0           rI3(k) k=0
    4H        INC3  1           k++
              ST2   LIST,3      store c to 1001-1008, content(1001-1008) is c
    1H        DEC1  1           m--
              DEC2  1           c--
              J2Z   COLMAX      a row is finshed
    3H        CMPX  A10,1       compare m and last m
              JL    1B          not found min
              JG    2B          found min
              JMP   4B          found same min
    COLMAX    LD2   LIST,3      rI2(p) p = lastest checking min (row)
              INC2  9*8-8       p = lastest item of the column
    1H        CMPX  A10,2       compare rX and every item in column
              JL    NO          it is not a saddle point
              DEC2  8           p-=8, p = item above of column
              J2P   1B
    YES       INC1  A10+8,2     find a saddle point
              HLT
    NO        DEC3  1           check the next min item of a row
              J3P   COLMAX
              J1P   ROWMIN      find next row
              HLT

## 解法2

这是另外一种角度的解法,数学的角度.令人期待.

这个非常有意思,当然是在动手画了几个矩阵之后.
可以画六七个矩阵,2x2 3x4 4x4,多画几个,越复杂的越要多安排几个鞍点.

是否存在鞍点,存在于不同行不同列?

如果存在两个鞍点,不同行不同列,apq,amn,
如果aqp在amn的右上角,那么存在apn,amq (m大于p,q大于n),
根据鞍点的特性apn大于apq,qpq大于amq,amq大于amn,
这可以得出`apn大于amn`,此时amn是鞍点,说明`amn大于apn`,
完全是相反的结论(上面做标记的两处).
所以如果存在多鞍点,要么在同一行,要么在同一列.

上面唯一特殊的情况是什么,等于情况,矩阵所有元素都相等,
那么每个元素都是鞍点.正是等于情况,才会出现多鞍点.

进一步讨论,只看列,鞍点会出现在哪些列?

列中最大,是鞍点的出现的条件之一.每列都有一个最大,
比较会发现(通过实际创建矩阵),如果将每列的最大值提取出来,
做成一个集合,这个集合的最小值才是鞍点分布的列,
其实这隐含了鞍点的另一个特性:行中最小.
如果这个集合中有多个元素等于最小值,那么这几列都是潜在的鞍点列.

进一步讨论,只看行,鞍点会出现在哪些行?

和列的思考类似,先找每行的最小值,做成一个集合,
找出和最大值相等的元素,就找到了哪些行是潜在的鞍点行.

最后,我们只需要潜在行和潜在列合并,找到交集,就知道解了.

接下来,给解法的源码添加注释

在源码中的第一阶段,用rX来存放每列的最大值,
每次存放列最大值时,会和rA比较,rA中方法的最小的列最大值.

非常优雅的就是rA存的是潜在最小的"列最大值",
每次计算完一列,rA都会指向最新的最小列最大值.
rA存的是具体的值,而不是矩阵的行列,
好处是:就算出现多个鞍点,他们在一行上,或一列上,值是相等的.

所以第一阶段就很好理解了,就是查找列最大值,并存放在1001-1008上,
共8个,rA指向最小的列最大值.

分析第二阶段,依然是遍历所有元素,源码中是从1080到1009,倒序.
遍历的过程中会遇到以下情况:

- 如果某个元素比rA还小,则说明这行不是鞍点的潜在行
  - 因为这行如果存在鞍点,鞍点的值要小于等于上面说的元素
  - 在列上,这个鞍点的值要大于等于rA,显然是不可能的
- 如果某个元素和rA相等
  - 且这列的最大值就是rA,那么这个元素是潜在的鞍点
  - 如果遍历到这行的最后一个元素,和rA相等,且列最大值和rA相等,那么就是鞍点
- 如果某个元素比rA大

存不存在某行,都比rA大,还存在鞍点?
不存在.因为每列最大值都不比这行元素小,rA取至列最大值,
所以不存在rA比行中所有元素都小的情况,至少有一个元素
等于rA或小于rA.

当一行不是潜在行时,先前标记的潜在鞍点,也会被清除掉.

第二阶段,也是基于数学的视角,不是复制第一阶段的处理过程.
而是基于第一阶段,用数学再次分析了的.

    * find saddle point of matrix
    *
    * convention:
    *
    * terminating condition: rIx <= 0
    * * label   ins   operand     comment
    CMAX        EQU   1000
    A10         EQU   CMAX+8
    PHASE1      ENT1  8           rI1=c, c = current column
    3H          ENT2  9*8-8,1     rI2=m, A10+m:address of the item
                JMP   2F
    1H          CMPX  A10,2       compare m and above
                JGE   *+2         if m > above, find next above,otherwise rX=above
    2H          LDX   A10,2       rX=content(m)
                DEC2  8           m-=8
                J2P   1B          until now, we found the max item of column
                STX   CMAX+8,2    1001-1008,store the max value of column
                J2Z   1F
                CMPA  CMAX+8,2    rA=min(1001-1008),rA=min item of max(every colume)
                JLE   *+2
    1H          LDA   CMAX+8,2    rA=content(m)
                DEC1  1           c--
                J1P   3B          now, 1001-1008,stroe the max of (column1...9)
    PHASE2      ENT3  9*8-8
    3H          ENT2  8,3
                ENT4  8
    1H          CMPA  A10,2
                JG    NO
                JL    2F
                CMPA  CMAX,4
                JNE   2F
                ENT1  A10,2
    2H          DEC4  1
                DEC2  1
                J4P   1B
                HLT
    NO          DEC3  8
                ENT1  0
                J3P   3B
                HLT

这部分也有很多新的写法.不过解法2最重要的是:不理清数学分析,是看不懂代码的.
