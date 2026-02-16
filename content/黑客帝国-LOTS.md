# 黑客帝国-LOTS 

LOTS 建筑布局HDA节点的主要输入有4个：

loyout的复合输出中的lots group   、 高速公路 立交桥 、标志性建筑 、HEIGHT OVERRIDE(高度上的艺术指导)

我们会先将高速公路提取处理，取它向上的面挤出一个几何体，将这个几何体体素化之后 Y轴中心为0，与体素化之后的标志性建筑合并转为 阻挡Mesh 后输出，我们再提取出城市布局底座 离 阻挡Mesh 比较近的块，与阻挡Mesh 进行boolean操作，最后未取出部分合并。去除一些比较小的块，并重新整理一下数据。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这下基本得到最基础的布局块，接下来有4种布局方式，我介绍3种

1： 就是不布局直接用。。。

2： 45degrees    遍历每个块，先计算得出最长边，再得出最长边与Z轴的角度，将这个块transform到原点并且最长边与Z轴平行，细分当前块，把每个点都判读附近相交两个点到自己的向量角度，如果角度不是 @angle%45 == 0 就标记这个点所在的块，然后删除这个块，完事后在拿到当前线框边，如果哪条边太短了，小于细分的块的长度都给它提取出来。然后在循环遍历提取出的边，如果这个边只有两个点最后出来结果的就两个点，如果边有3个点最后保留距离最远的两个点形成一条新边。

3：subdivison   遍历每个块，先计算得出最长边，再得出最长边与Z轴的角度，将这些属性先存储在这个块上，计算每个块上的每一个点是不是内凹角(两条边构成的角为锐角 ，这个用calculate_occlusion节点快速计算)，循环遍历每个块，如果这个块包含内凹角的点，那么我们会先把模型旋转，对齐XZ，提取出每个内凹点，在每个点位置横着一刀竖着一刀，就是X方向和Z方向切割，将切割完的模型输出。如果没有包含内凹点，那么直接输出。

得到切分过后的块之后我们还需要删除比较小的面积，利用周长面积比提取出比较混乱的面，将相邻的混乱面合成一大块在，再利用Lot_subdivision细分，再把分开的拆为一个一个单独的面，再计算周长面积比，再次剔除混乱的面。（lot_subdivision：找到最长边，垂直与最长边切割当前块）

（周长面积比：
多边形的周长与面积的比值，用来度量多边形的紧凑度。圆的周长面积比在所有的几何图形中是最小的。面积是缺陷特征的一个度量。面积只与缺陷的边界有关，而与其内部灰度级的变化无关。缺陷的周长在区别具有简单和复杂形状物体时特别有用。一个形状简单的物体用相对较短的周长来包围它所占有的面积。周长与面积比是用来描述缺陷形状的参数，当形状为圆时，周长与面积比最小，越呈长条状，周长与面积比越大）

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

布局拿到之后我们可以将loyout的复合输出中的city_metadata上的城市高度数据转递到新的布局块上。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

之后找到沿着街边的布局块，我们将这些块根据对应道路类型人行道宽度做出对应的收缩，主要是A道路与B道路 C道路的人行道是默认宽度，将道路与人行道生成出来与对应街边的块做一个boolean操作。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

将boolean之后的块整理过后，与输入HEIGHT OVERRIDE 做一个高度映射，就是如果当前建筑高度小于HEIGHT OVERRIDE的高度，那么当前建筑高度就等于HEIGHT OVERRIDE的高度。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

之后便是给上布局块不同的建筑风格，这里有两种，一种纽约风格，一种传统风格。

剔除不是正方形的面，判断条件1： 等于四个点，条件2： 角度余90为0.

之后再根据建筑高度(一开始拉的那两个凸的包代表建筑的高度)高度小于一定值，

之后再剔除太小的 与 太大的块，剩下的就用纽约风格来布局

被剔除的都再一起用传统方式来布局

纽约风格：

先找最长边，最长边与相邻边平行 x 和 z轴的角度值，旋转到与X轴与Z轴平行，
头尾各隔25米切一刀，split出来头尾的25米的条，再x轴细分为10米一格10米一格，格子的大小3米内随机浮动。
split出来的中间的条，再按照头尾25米切一刀再同样split出来头尾的25米的条，再x轴细分为10米一格10米一格，格子的大小3米内随机浮动。

找到比较小的块面积小于220大小，提取1/7合并为大块，

再把中间的边(内边)，向内方向随机offset3米大小

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

传统风格：

通过 面片的面积与建筑高度来决定lot_subdivision的切割迭代次数，最后如果面积还是大于某个值我们还需要再切割一次。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

将两种风格的切割完的块合并输出，做一些数据处理，剔除比较小的块、找到内部块(没有一个边接触空地)等数据操作。

现在有些小块的形状还是太过不规则，我们将分离出不是4个点构成的块。使用提取做好的17中块用UV填充的方式布局在这些不规则的块中，合并4个点的块输出，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

我们又要分离出沿着街边的布局块，垂直与道路的法线做一个随机的offset。让街边有些不规则的感觉。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

用现在的块做一次与输入HEIGHT OVERRIDE 做一个高度映射。然后在做一次数据的清理，包括剔除一些比较狭长的块等。

之后我们需要引入 BDF 文件数据，我们将bdf信息赋予到每一个地块上，地块根据城市元数据分住宅与其他两种房子，如果是住宅在 属性带NYA的bdffile里面选择，如果是其他则在不带NYA的dbffile里面选择，但是其他类型的房子需要建筑的高与宽在当前选择的dbf所定义的高宽范围内，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

得到带有BDF信息的地块后，分离出没有UV布局的地块，不是纽约风格的块，不是超级细长的块，不是正方形的地块，面积大于629的地块，这些地块我们将赋予它们我们制作的预制形状。

有两种选择方法 一种是 根据BDF的最小宽度信息移除宽度不满足的预制片，在根据每个预制片的出现概率来选择，另一种是 直接根据自己定义的概率来选择。选择完之后与我们分离出的块合并输出。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

拿到最终的地块之后，我们会分两大类五种方式

一类是帝国大厦类型的和传统类型

帝国大厦类型区分是根据只能特定预制地块类型，地块面积不能太细长，不能太小,按照比例与预制地块类型分配两种不同方式的建筑， 其他为传统类型 。

帝国大厦的两种方式分别为 1 类型大厦是挤出一个楼体基础，再在楼体顶部挤出为一个锥体，体素化，在体素内撒均匀点，每个点copy一个正方体，再把超出体素的正方体删除既可

2类型是在地块面上根据ramp图，做出一阶一阶的矩形，每阶矩形在根据ramp做不同高度的挤出。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

传统类型 3种分别是 一级挤出 、二级挤出 、三级挤出

一级与 二级、三级的区别在与是否为复杂形状，就是一开始不是4个点构成矩形 用uv布局的地块

二级与三级的区别在与自己分配的比例参数。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这样基本就得到了最基本的BDF建筑体，如果我们需要那种下层是一种BDF类型，上层是另一种BDF类型，我们需要把一些BDF的最底层就是第一级的挤出剔除出来，换一种新的建筑风格就可以。

然后我们就要判断建筑的正面和背面，主要是细分后用法线打射线到包围盒上，打中为正，自己@P+(v@N*0.001)打自己，打中为背面。得到的正面背面数据再传递回BDF建筑上。之后就是为BDF建筑计算一些之后要用的属性，根据BDF类型给上不同颜色即可输出。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

