# 高级水面白沫涟漪OR海浪

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这是我们传统的通过水面当前像素深度、世界坐标位置、场景深度、相机位置求得一个稳定的水面深度，深度有两种，一种是从摄像机出发的深度，另一种则是垂直深度。这里我们求的为垂直深度。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

（这里我用的SingleLayerWater，得到场景深度的节点为SceneDepthWithoutWater，正常半透明用SceneDepth）

注意：SingleLayerWater的深度图存在降采样与半精度问题

r.Water.SingleLayer.RefractionDownsampleFactor 1

r.Water.SingleLayer.RefractionFullPrecision 1

实际使用需要在代码里面更改一下默认值。我这里已经改过了。

得到水的深度这个线性值之后，便是捏水波或者说白沫的形状了，这部分全靠自己调整一下波形的相位、周期、振幅这些，而且也不局限于Sin波，只需要是连续可导的就行，有点像自己捏一个噪声。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

为了展示这种方法会出现的问题，这里我简单给个sin做示范。

可以看见在这种深度渐变比较均匀的地形上，这个方法是比较不错的，而且视角的变动也不会影响水波。

但是，如果在深度渐变不均匀的地形或者场景中，就特别容易发生白沫大面积出现的问题。

出现这个问题的原因就是大面积的斜率过小，导致大面积的深度值一样，同时出现白沫。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

就如此图一样。

如何避免这类问题，以下为个人思考和经验，既然确定这个问题是由于斜率过小不合适导致的，那我们应该屏蔽这类斜率过小的区域。但是仅仅根据已知的这几个信息无法知道斜率，所以大部分人都会选择烘焙一张垂直的拍摄的信息图，像SDF或者法线这些。

然而我像做一个通用的效果，并不想去烘焙这些信息，在这个前提下，该怎么拿到水面下的场景深度斜率，这里就会想到用DDX与DDY去拿到当前深度与周围的深度的变化率，就可以算得当前点的法线，归一化的法线的Y轴便是当前点的斜率大小。

![Untitled.jpeg](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.jpeg)

这一步，我们便是使用当前水底点的世界空间坐标去和周围水底点世界空间坐标去做DDX与DDY得到当前点的水底世界空间法线。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

法线为-1到1大小，所以有部分为黑色。因为地形本身就是格子，一个地形面片内的法线是固定的，所以看起来结果是这样的，目前可以确认是正确的。

现在我们只需要屏蔽掉不合适的区域既可以。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这里我就屏蔽了斜率小于一定阈值的区域。最后就可以得到正确的效果啦。

剩下还没有有调整白沫形状函数的细节，我就不过多介绍啦，很多地方都有。具体可以看https://zhuanlan.zhihu.com/p/63722738

