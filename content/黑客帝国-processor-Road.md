# 黑客帝国-processor-Road 

到路这一部分啦，这一个HDA的主要输入就是城市Layout的复合输出，我们主要用到道路，就主要把道路线段blast出来处理，

拿到道路线段之后，我们先要计算与得到一些数据给之后的解算用，首先我们计算交叉路口每个点的属性，包括每个点法线与其他点法线的最大角度，最小角度，与水平轴的角度，用最大角度作为顺序属性等等。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

再把道路相近的点融合变成一个交叉路一个点，这个点取所以点的交叉路口数最大值，道路宽度最大值。不同宽度给上不同颜色。分离出大于三个分支的、边缘有其他点的、角落点的道路相交点给之后的交叉路口解算器用，  分离出的点 叫点 S 吧

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

道路交叉解算器这边先拿到原始的道路线段数据将每一段道路从中间分为两段，每段都和 分离出的点S 在同样位置width米内就保留否则剔除，遍历每条分割后的道路线段。(width为路的宽度属性)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

为每个分割后的道路线段的交叉路交点copytopoints一个box，box的
scale={@width,1,@perimeter}
@perimeter 就是与这条道路最近的相交道路宽度。

将copy的点云与copy得到的模型一起输出。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

可以看出有些地方的交叉路口没有贴合，我们需要修复一下他们，就是用他们没有相交的地方提取处理做个凸包就能得到 交叉路口修复块模型。

从交叉路口解算器，可以得到两个主要的东西 一个是 交叉路口点云与copy得到的模型(粗模 表范围)，两个都有作用 ，先说点云。

点云每一个点在当前位置向法线方向生成一块road长度为@perimeter-3宽度为@width，剩下3m的长度，点云会移动@P += @N*(@perimeter-3);后再生成（3m的宽度主要就是马路口的人行道）。移动前的点云和移动后的点云会合并为新的点云集合。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

点云copy得到的模型的作用主要就是标记路口的范围，之后就可以专心搞道路的块，道路线段为 整个道路线段 去掉在 交叉路口的块。

拿到道路线段之后分两路，一路是计算道路的各种属性(这就看各自需求啦)

另一路就是主要的生成路这一部分。首先删除太小或者太大的线段，大家法线都向内，然后向法线方向移动5m，判断移动后的法线是否与移动前的法线相同。不同就删除掉当前线段。根据长度分为20m、10m、5m的块大小，太小的用scale的缩放，正常为1，每个块生成一个对应的点云输出。

之后就组合我们得到的数据，拿到道路线段后计算完属性的道路线段点云 、 拿到道路线段后计算得到每一块道路类型与大小的点云 、 交叉路口移动前的点云和移动后的点云会合并为新的点云集合。

组合后的每一个点我们都会附上对应的属性blockCode Type，已经unreal里面路径等等。基本就是结果点云啦

循环每一个blockCode中的第一个点根据附上对应的属性 用road_block_maker产生出对应类型的路模型。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最后给一个可视化输出到input0，可视化就是直接用得到的点云与自己产生的道路模型，做撒点输出可视化(模拟ue里面撒点)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

输出0就是可视化   输出1就是点云   输出2车道线与人行道线(车道线和人行道线由道路产生器直接产生)   输出3是路口数据   输出4是产生模型集合  输出5是交叉路口修复块模型

