# Houdini VEX(六)组(Group)的相关操作

一、@group_name方法

- 等于1是建组，等于0是移除出去
- 这里我们对顶点的y值大于0的点打组到了up组，并对顶点序号是4的点移除出了up组
- 可以在spreadsheet中看到group分组
- 如果你在spreadsheet中看不到group，勾选group attributes
- 我们也可以对组进行判断操作：
二、Dedicated Function：专门为group设计的函数

1. setpointgroup函数：把点添加到组中，或从组中移除
1. setpointgroup函数其他用法：多了最后一个参数--模式
1. inpointgroup函数：判断点在不在组里面
- 判断0号输入端中的顶点是否在up组，在组内，则返回1；否则返回0
1. npointsgroup函数：返回该组中的多少点
- 0号输入端中有多少顶点是up组中的
1. expandpointgroup函数：把组中的元素都列出来
