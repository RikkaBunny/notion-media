# UE-高级地形材质

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这次的目标是做一个比较3A级别的地形，那么地形的效果上有几点的非常重要的，一个是地表重复度，一个是地表置换高度，一个是地表阴影，一个是地表光感，这也是大多数人做地形常常遇到的问题。

这次准备用上UE比较新的技术，比如Virtual heightfieldMesh这些，从资源流程规范到地形制作技术还有场景的光阴都会有介绍，为什么要介绍资源的规范和场景的光照了，是因为我们在制作的时候地形的时候不止要关心地形制作本身，资源与场景光影同样影响着我们地形的制作，所以这次会全面的介绍一遍，如果对某个点感兴趣可针对性查找。

### 贴图：

地形的最重要底子之一，这里我们去Quixel里面去找我们需要的地表贴图，这里贴图的导出格式我们需要自定义。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这里我用的格式为 BaseColor + Roughness 为一张，Normal + Specular + Displacement 为一张，总共2张4K贴图，(这里贴图我导出后在SD中再次处理过  主要为两次 翻转法线Y ，因为我发现Quixel里面选择翻转法线Y并没有给我翻转，还有一个处理就是将置换图的色阶自动拉伸一下 ，注意需要在SD的线性空间下处理不然会给你弄成Gamma空间，下方箭头处 )

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

下面就是贴图在引擎里面的设置啦，我们可以右键所有贴图Asset Actions → Bulk Editvia Property Matrix  使用批处理来修改贴图设置。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

我们需要把C_R开头的贴图sRGB勾上，N_S_H开头的sRGB去掉。然后所有压缩格式用BC7 ，Mipmap设置为 Sharpen10  锐化度为10(可以自行调整)，这样能让我们在远处也有比较清晰的贴图。

为什么用BC7压缩了？（BC7与 Direct X 11 一起发布，是一种更高级的 S3 压缩格式，它使用一系列方法对 4 个纹理数据通道进行编码。与DXT1/5（BC1/3）等旧格式相比，它将提供更高质量的压缩和更少的伪像，并支持 sRGB，但它永远会保留 Alpha 通道。）

简单一句话就是 在 PC 上，使用 BC7 (modern) 或 DXTC (old) 格式更好，因为BC7有永远保留Alpha通道的特性，所以我们每一张图都得用上Alpha，而且Alpha有不压缩的特点所以适合保存一些重要数据(所以一开始贴图配比为 C_R，N_S_H)。

### VirtualTextuer:

这里VT的基础配置教程，我们就不讲了网上一搜一大堆，这里讲一讲VT设置里面需要注意的地方，有些同学设置完Virtual Heightfield Mesh之后发现地形还存在，这时就会有两个地形同时存在，我们需要找到地形的Draw in Main Pass这个选项，选择FromVirtualTexture，代表我们走VT渲染，本身不渲染。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

当我们Build SVT的时候有时候会直接Build卡死，这时候我们需要调整一下Streaming Levels，0 代表最大，1代表最小 往后 越来越大。

如果发现VT的BaseColor，得到结果有条纹，可以尝试使用YCoCg空间的BaseColor得到更好质量的。

### 材质：

可以先看一下材质样子，比较简洁

LandScape  Material 

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

Virtual Height field Mesh Material

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

注意两边的法线都为世界空间法线。

我们进入一个NFLayer 内部看看 一步一步的拆解来看

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这个部分主要分为两个，近处的石头 与 远处的石头，通过一个远近的淡入淡出Mask进行一个过渡，这样做有两个原因，一个是减弱贴图Tile情况，另一个是因为贴图拉远之后 纹素比越来越大，导致看不清细节等情况，对于远处的石头我们可以调节它的UVTile 大小 与 置换高度。对于VT来说 VT其实是独立于Camera的 ，所以远近淡入淡出的Mask，不可以用WorldPosition 与 CameraPositon的距离来控制，所以这里我们需要用另一种思路，那就是Mipmap。因为VT这种技术会根据当前贴图纹素比自动调整Mipmap大小，从而保持场景一个稳定的纹素比。那么我们只需要当Mimap小于某个值就使用近处Rock VT，大于某个值就使用远处Rock VT 中间淡入淡出过渡。

View Property节点便可以为我们提供当前VT的Mipamp级别。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

通过Mipmap，我们就能达到远近淡入淡出的效果啦。

我们再看看RockLayer里面实际逻辑是什么。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最前面是UV，我们把UVSacle暴露出去。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

先看UseCellBomb为false的状态，C_R这里很简单就是将RGB给一个Color Tint 之后直接输入到BaseColor既可，A通道直接给Rougness就可以啦。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

N_S_H这边也是比较简单，我们只需要把B通道给Specular，RG通道这边通过叉乘得到法线RGB，连入Normal，A通道代表高度，我们需要将A通道单独输出 OUT_Height这个参数控制 地形Layer Blend的高度混合，另一边高度也要连接到Opacity上，这个代表着我们置换的高度。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

Displacement 这里主要控制地形高度的起伏，我们需要让地表的石头有真正的凹凸，所以我们需要控制这个凹凸的强度，这个我们减去了凸起的1/3，这是一个Trick，因为地形如果凸起太高了人物陷入地面太多，这样会使凸起之后整体往下移动一小段距离。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

对于解析法线 ，我们通常需要将法线从 0 到 1 空间转换到 -1 到 1的空间，然后叉乘计算出Z，我们可以顺带法线增强也一起写了，对于法线增强其实就是增强X 和 Y两个轴，最后将结果归一化基本就可以啦。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

但是后面发现法线出现不规则黑斑，经过测试发现着部分法线的长度等于任何值，所以我在后面加上了一个法线错误检查，如果法线出现错误，那么输出法线 (0,0,1)。

这时候其实效果以及差不多啦。

如果要极限的去Tile的话就，可以打开UseCellBomb这个开关。对比一下

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这个技术的原理就是通过一张贴图来 缩放 、 平移 、 旋转 UV，通过这个旋转过后的UV去采样贴图，就能保证采样的贴图的每一块UV都是不同的，那么每一块不同的采样结果怎么融合在一起了？这时候我们还得用原来的UV在采样一次贴图，当不同采样块的结果将要突变的时候，我们便可以过渡到用原UV采样的结果，这样便可以连续起来啦，控制贴图的Alpha通道便记录了一个淡入淡出的Mask。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这里每一块都代表一块UV的缩放、平移、旋转。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

具体方法可以去黑客帝国demo 或者 Quixel出的村庄demo里面搜索 CellBomb  函数方法即可。

### 光与影：

对于大世界来说，光影的方案一直是一个问题 不仅需要考虑性能，也要考虑效果，对于阴影我所采用的方案是 近处20M左右(级联阴影 2到3级 既可)  + 中远距离 (距离场阴影) + 超远距离(Far 级联阴影 这个是比较特殊的级联阴影 可选) + 补阴影(屏幕空间阴影) 。

这套方案算是大世界阴影里面不错的一套，无论远近皆可达到不错的效果，这里我要挨个说明一下，对于近处我们需要高质量的阴影，所以级联阴影是比较好的选择，对于说近处具体的是多少米，那就需要你自己用肉眼去评估啦，这里的级联我们只需要给到两到三级就可以啦。对于中远距离阴影，那就是阴影最多的地方，这时候我们就需要空间换时间啦，用速度最快的距离场阴影来做中远距离就是一个不错的选择或者预烘焙 都是可选的(这里我还是喜欢大世界全动态方案)，对于UE来说距离场阴影就是一个比较好的用的功能，需要注意的是 Foliage要开启距离场阴影需要在Foliage面板下开启Affect Distance Field Lighting

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

对于超远距离 Far级联我通常不太用的上，对于需要的同学可以去Light下面的Cascaded Shadow Maps 面板展开 自行测试。

对于屏幕空间阴影(Contact Shadow)(主要起到补充阴影的功能 )，屏幕空间阴影有一个好处就是基于深度图来实现，在屏幕上Track固定距离，这个特性就代表 屏幕空间阴影 针对的物体可大 可小，当你屏幕上大部分是细节时候，那么它会提供不错的细节阴影 (比级联更加细致)能大大提高真实度。(物体的自阴影)

当你屏幕上大部分是远景时候，能提供大物体之间的自阴影

开启方法：

注意地形上也需要开启Contact Shadow。

对于物体使用Contact Shadow并不一定需要真正的顶点偏移置换，因为屏幕空间阴影是基于深度图来实现的，我们只需要偏移深度既可，也就是输出Pixel Depth Offset即可，具体示例可以看 RuralAustralia Demo。

### 最后：

因为用了VT，所以我们可以使用海量贴花来丰富地形细节，我们如果有Height的话也可以投射高度到地形，花样很多，效果也不错。

只需要将物体输出连接到Runtime Virtual Texture Output即可，只是VT暂时没有Metallic，不支持金属哦。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

但是VT的缺点也暴露出了一点，那就是只有 XY，当遇到垂直高度的时候会拉伸。但是整体来说瑕不掩瑜，以上便是新流程地形制作总结。

上面大多数只是针对效果向的东西，对于性能来说，VT已经比较成熟优缺点大家有总结，可自行判断，对于VirtualHeightfieldMesh来说，虚拟高度的场的性能开销还是小于曲面细分的，因为不用走细分的pass，Mesh的生成已经Lod都是在GPU自己进行控制，精度、网格生成算法、LOD优化都要比曲面细分要好，但是确实是实实在在生成一个高精度网格。

其他像是用 Tex2DArray[] 、地表混合采样算法这些 扩展性的东西就需要与引擎程序的配合啦。剩下就自行扩展。

