# UE-使用Deitr做一次采样的三项映射

## 简介：

    三项映射的做法其实是利用世界空间坐标，作为UV空间的坐标去采样一张贴图。最简单最直观的就是地形的UV就是使用世界空间的 X与Y来采样UV空间的贴图来做，但是因为没有考虑到Z轴的影响所有导致在垂直的地方，地形贴图拉伸会很严重。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

## 三项映射：

    三项映射的做法是根据当前世界空间顶点法线的朝向来判断，自己当前是属于哪个投影空间的，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

蓝色就说明自己在 属于 XY平面UV，绿色就属于 XZ平面UV，红色就属于 YZ平面UV。

使用一点简单的数学运算就可以得到当前位置属于哪个平面UV空间。( Pow可改为cheapContrast，pow值越大，过渡越窄 )

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

使用每个世界空间中的三个平面，用每个平面UV都去采样一次贴图。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

顺序分别为 前后面，左右面，上下面。

通过世界空间法线得到的三色方向Mask来确定当前像素应该使用哪个面。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最终就得到了一个比较标准的三项映射。

好处就是适用于任何模型的连续贴图。坏处就是太费了。

### 优化为一次采样：

每个平面采样一次贴图实在是太费了，那么我们可以确定下每个像素所在平面的UV，最后统一去采样一次贴图嘛?

当然可以，只不过有适用条件。当你面与面是硬切的时候就可以使用这种方式，但是绝大多数情况下不适用.（我做破碎的时候内部切面基本都是硬切，所以用的这种方法）

这种方法UV如果插值之后，就会产生这种扭曲的效果，插值过后的UV采样不到正确的位置，所以出现了这种错误。而用先采样三张图再插值，那是采样结果图淡入淡出，原理是不一样的。

### 优化uv扭曲：

    通过世界空间法线得到的三色方向Mask来确定当前像素应该使用哪个面，这个Mask的值本身就是非零既一的两种可能，在这个采样平面上和不在这个采样平面上两种可能。

而我们现在得到的结果是渐变的Mask。结果就是 用A平面UV混合用B平面UV去采样贴图，那么结果肯定就错误了。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

那么，我们就需要确定下来到底用哪个平面去采样。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

结果就是 过渡不行，是硬过渡有很明显的接缝问题。那么想要解决过渡问题，就可以用到Dither这项技术。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

加上Dither这步操作之后看起来确实就好多了。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

那么，正常的步骤应该到这就结束了， 但是 你会发现过渡的地方还是有莫名其妙的接缝问题。

这该怎么解释了。其实把这个问题是因为 UV跳变太大导致 DDX DDY算错了，所以导致贴图的Mipmap也错了，过渡区域的Mipmap跳变导致过渡的地方有莫名其妙的接缝。一张图可以看懂

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

使用frac,和Fmod就会有这种问题。

解决方式也很简单，就是传一个连续的UV当作DDX 和 DDY运算的uv就好拉

将贴图的采样器改为 Derivative 模式

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

传入我们一开始用淡入淡出方向Mask得到的插值过后的UV进行DDX和DDY计算就可以拉。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

完整的Shader这样就可以啦

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

一次采样。

