# WaterMaterial

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这是我们通常得到水面到水底深度的公式，最终得到的结果是一个从0到1的渐变Mask，           

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最后在加一步控制调整 强度 与 对比度 ，最后clamp既可。

### 颜色

这个时候我们就得到我们最终一个从岸边到深水的从0到1的mask，通过这个mask，我们可以使用lerp函数，去插值两个颜色边可以得到水的一个颜色表现，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这一步也可以使用LUT的方式来采样颜色，已到达更好控制的效果。我们可以去PS里面做一张LUT的色彩控制表，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最终我们导出的LUT表，我用的是256*1像素，那么我们uv的U就是我们从0到1的深度渐变值，V就可以定值，主要LUT不要压缩，tiling为clamp，不然就会采样到错误位置。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

颜色这边最终还得和其他效果混合叠加输出 比如白沫、SSS散射。

### 透明度

透明度这边和颜色思路差不多，也是通过深度渐变mask去lerp两个不同的透明度大小，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

之后有个稍微特殊的地方，就可以可能特别近的岸边和特别远的远景，透明度还得单独处理一下，这里我让特别近的岸边水更加透明，显的水比较清澈，很远处的水不透明，显的更深邃一点。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最后还加上 菲涅尔 和 顶点色对透明度影响输出既可。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 法线

法线这边走的是四向偏移法线 与 flowmap法线 混合的方案。(这里还有距离场等等方案暂不介绍了)

首先是四向偏移,uv使用的 世界空间三向投影(低配版)，在uv阶段就插值，结果就是会有分界区域会有一像素偏差 不过也能接受。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

四向偏移用的和ue内置基本一样，只不过为了优化指令数，我自己封装了一个MF，把里面的贴图采样器设为wrap了而已。

接下来就是平滑法线，也就是与法线 (0,0,1) 做lerp。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

还有一个便是与flowmap 产生的法线做 混合，用的RNM方法具体如下。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最后归一化便可以输出。

### WPO

这里我用的和神海有点相似的方案，就是WPO加flowmap。WPO我选的是正弦波，并没有用Gerstner波

![water.gif](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/water.gif)

大体是这样

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

顶点色R是控制混合区域，顶点色B是流速控制波浪的大小，流速缓慢的地方波浪小，湍急的地方波浪大。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

波形状就是这段

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最后根据深度加一个波强度的淡入淡出既可。

### 焦散

焦散其实挺简单的，和四向偏移法线差不多，我们使用两个方向来对焦散做偏移既可，只不过需要多注意注意 UV，我们需要采到水底的uv。具体如下

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这样就能达到焦散在水底的感觉啦

![waterCaustics.gif](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/waterCaustics.gif)

在用DepthFade控制焦散的淡入淡出既可。

我们还可以使用SceneColor，也就是用屏幕亮度来判断阴影处与光亮处，来控制焦散的亮度。

### 假高光

这一步主要是在UE里面做，在Unity里面还好，UE里面因为shading mode固定，没法调整光照模型，这个主要是帮助水面有波光粼粼的效果

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这个就是自己算一层高光加在自发光里面充当一层假的高光，麻烦的地方在于需要用蓝图传递一个光向量(延迟渲染拿不到光方向)，然后将光向量存储在MPC中，材质再通过MPC得到光的方向就可以计算高光了。为了波光粼粼的效果我们也可以直接用采样对应需要的光斑贴图。

### FlowMap

对应flowmap贴图的生产在houdini里面制作，RG通道存储的是速度，B是白沫，A存深度(可选)。UE里面其实有对应的FlowMaps_Simple，我们只需要传对应的参数即可，这里我简单优化过，如果用单通道的白沫图，我们就可以把白沫与法线两张图Pack起来一起传，算完之后再Unpack就好。

UV的选择这里需要注意Flowmap贴图的采样用模型自身UV采样，得到RG通道之后将 0到1区间的颜色值转换到-1到1区间的向量值传入。法线与白沫贴图的UV需要传入世界空间UV，这样不同的水面法线与白沫也可以连续起来。(不同Flowmap贴图烘出来已经接缝处已经是连续的啦)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

FlowmapSimple 里面逻辑也很简单，主要就两张贴图循环交替的移动，造就视觉上的一个移动假象，波形大概是这样，交替迭代，然后动态lerp两张图片的结果既可。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这里注意一下，因为坐标系的缘故，需要将X轴翻转一下。贴图采样器也可以改为 Warp。

最后还有一点需要注意 

折射我们需要改为 Pixel Normal Offset，否则水面边缘会有撕裂。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

