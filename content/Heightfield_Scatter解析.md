# Heightfield Scatter解析

    最近在重新改进我的Houdini生态撒点器，所以有了此篇前置记录，植被撒点其实就是对mask区域内进行一系列的点云分布。针对植被撒点大多数是直接用Heightfield Scatter来撒点，但是对Heightfield Scatter的内部原理并不太了解，Heightfield Scatter其实也是一个内部组合节点，进去就能看到内部细节，看内部节点这也是一个不错的学习途径。

    首先Heightfield Scatter为最大三个输入，最小一个输入，一个输出的节点，三个输入分别为1：需要撒点的地形、2：撒点所在的Mask或者预先撒好的点、3：需要在点上Instance的图元。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

首先一进来就看见 1号输入与2号输入同时都有这部分，也就是分离出没有tag的点(每个Heightfield Scatter撒出来的点都会带Tag)，

接下来看看有没有二号输入，如果有，检查一下有没有指定撒点mask，如果有将这个指定mask替换为height的mask(红色)，

在撒点前，我们再看看Heightfield Scatter一共有四种撒点类型，分别是By Coverage using Mask Layer、By Density using Mask Layer、Total Point Count using Mask Layer、Per Point Count using Source Point，每种都有三种撒点方法，分别是 Uniform Distribution 、 Normal Distribution 、Exact Scale， 这里我们主要看最常用的两种，使用覆盖率撒点和使用密度撒点

接下来我们将计算正太分布的期望值，等下添加pscale大小时候使用，之后我们需要计算撒点的密度，如果是By Density模式的话，density = density，也就是没平方米的密度，如果是By Coverage模式的话，覆盖率是1的情况下，根据Outer Radius和Falloff来动态的调整density，去除Falloff的影响大致就是半径Radius的圆内的覆盖密度。
圆的面积 = 3.1415 * pow((Outer Radius * Scale),2)

最终Coverage也需要转换为Density，因为Scatter直接上Density这个参数，也就是每平方米上的产生的点数，所以这里我不推荐使用其他撒点模式，只推荐 By Density using Mask Layer 因为直观一点，By Coverage using Mask Layer这种模式 一是最终还得转会Density，二是改变Outer Radius的值不但会改变density，还会改变之后的点与点之间的避让，同时改变两个不相干的逻辑。

然后我们直接使用scatter节点来为mask范围内撒点，密度为计算出的density。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

使用flatten来使用撒来处的点Y轴都为0，

接下来就是两个 swicth ，分别是 1：是否使用了Mask，如果有就在mask内撒点，没有就在高度上撒点也就是全部撒点。 2：是否使用了预先撒点，如果有就用预先撒的点，如果没有就用刚刚撒的点。

然后就是一系列操作，将点的高度附着到地面，为点创建属性添加内圈与外圈概念，内圈为面板Outer Radius * (1.0f - Falloff)，外圈为Outer Radius，最主要的还是为点云计算了pscale的大小，通过每个点的pos所在的quantize块来计算得到他自己的一个随机值。(当quantize等于10，那么10米内的点得到的随机值是一样的)

```python
 //randomize on point position to keep the changes localized
vector quant_P = quantize_pos(@P, quantization_amount);
float rand_val = rand(quant_P + global_seed);
```

一共三种模式类型

如果是Exact Scale，pscale就是固定值，如果是Uniform Distribution，pscale就是在最大与最小值之间随机的一个值，如果是Normal Distribution 也就是正太分布，Spread参数控制标准差，也就是方差的大小，方差越小那么就越接近平均值，整体分布满足正太分布。

这里我推荐用正太分布与随机分布这两种。

接下来就是重头戏啦，也就是relax部分，通过relax_iterations来指定次数的循环迭代。

迭代内容主要是 硬性条件约束、软条件约束、属性设置、点云剔除。

硬约束：通过relax_point自定义函数得到vector4返回值，@P.xyz = vector4.xyz,   remove_flag = vector.w, 会删除当前点。
软约束：通过relax_point自定义函数得到vector4返回值，@P.xyz = vector4.xyz ，不会删除点。

relax_point 自定义函数，通过pcfind_radius找到一个点附近所有带特定属性的点云，遍历点云中每一个点，拿到其他点相对与当前点的方向与距离，判断距离，如果两点距离特别近，就可以将最终的返回值vector4.w = 1，跳出点云循环。如果两点之间的距离小于 两点各自的内圈距离之和，那么就随机一个0到1之间的值，看看是否小于删除概率，如果小于那么将返回值vector4.w=1跳出点云循环，最后在设置一下返回值vector4.xyz=c_pos + (movement_factor * (avoiddist - diff_len) * (diff_vec / diff_len)); 也就是根据步进值向两点反方向步进。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

属性设置：将的地形的上属性与高度设置给对应xz平面的点云。

点云剔除：将点云当前所在位置mask值小于 MaskCutoff的值或者点云所在位置超出地形范围的点给remove_flag = 1。

最后循环完之后就没啥好介绍的啦，就是赋值一下地形属性，设置一下旋转，是向上还是向地形法线方向，最后在剔除掉 remove_flag = 1的点云基本就没啦。

这里再介绍一两个有意思的小技巧吧 Heightfield Scatter节点里面也有用到

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

使用Attribute Wrangle里面的Bindings，将自己想要的几何属性绑定上一个自己在VEX里面调用这个属性的名字，相当于指针指向AttributeName,指针的名字是VexParameter。   上面的Autobind by Name就是为每一个AttributeName自动创建一个对应名字的VexParameter，在Vex里面通过@VexParameter就可以调用这个Attribute。

还有个就是

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

在每个节点创建时候，自动给seed属性给一个随机的值，node.sessionId()是每个节点不同的一个ID值。这个我们还可以给上节点颜色、节点形状、配套节点等等一系列初始化操作而且不只初始化还有很多Event函数。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

