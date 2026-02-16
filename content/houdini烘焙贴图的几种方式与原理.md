# houdini烘焙贴图的几种方式与原理

在houdini里面烘焙贴图，说白了就是将模型UV空间上值按照你输出的像素数一个一个的取出来，一个重要的函数便是 uvsample(<geometry>geometry, string attr_name, string uv_attr_name, vector uvw) ,

第一个是你要采样的几何体，

第二个是你需要采样的属性，比如法线、位置等，

第三个是你需要采样的是那套UV(如果有多套uv，uv、uv1、uv2，自行选一套采样)，

第四是你采样的位置，就是在uv 0到1中，你当前的uv值。通常第三个值 Z为0就行。

这个函数的意思是拿到你的uv值，通过找对应的片元顶点上的属性值，做重心坐标的插值就可以得到当前位置像素上的值。这一点和光栅化是相似的。

### 第一种 SOP 到 COP：

我们可以在sop中先开一个 uv空间大小的 体素。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

我们开了一个size为（1，1，1）的体素大小 ， 注意体素的左下角需要对应在 (0，0)，对应体素的分辨率，我们给一个我们需要输出的图片像素大小，比如 1024x1024 ，那么我们体素大小便对应像素大小为（1024，1024，1），这样我们使用 VolumeWrangle便可以通过当前体素的v@P去uvsample，我们需要的模型属性将它转移到体素上存储。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

注意我们开volume要存储的单位 float还是vector

之后再开一个COP，使用sopimport开一个像素大小一样的图片就可以用rop输出啦。

### 第二种 COP采样SOP：

这种方式的思路大概就是 我们需要把要烘焙的物体展开到UV平面，@P=@UV；，然后在COP里面开一个1024(我们需要的尺寸)大小像素图片，通过VOP COP2 Generator节点来 使用每一个像素去采样我们展开的UV平面得到我们需要的属性输出。

首先我们需要把我们要烘焙的物体拆开，通过Point Split节点，拆出共享顶点，使每个点独立出来。(我通过UV属性去拆分，并把属性转移到point上)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

使用vex将每个点的世界空间位置转换为UV空间的位置。

```javascript
@P = @uv;
```

就可以输出一个空节点 OUT_UV ， 原始需要烘焙的几何体也需要输出一个空节点 OUT_GEO等下会用。

我们就可以创一个COP Network啦。

在COP里面我们使用Color创建一个新的贴图，贴图大小为我们需要的大小。

接下来使用vopcop2generater来进行属性的读取。

打开vop，可以看见一个 global节点和一个 output节点，global节点上有很多属性 我们可以通过F1去一个一个查看(SOP中的vop也是同理)，这里我们主要用到 x与 y两个属性，代表uv的0到1.

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

将uv合并起来为一个vector通过xyzdist找到OUT_UV上最近的一个点UV值，通过这个UV值去找OUT_GEO上对应位置UV的属性，我们就可以拿到我们需要的属性啦。然后output即可。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

主要一个 xyzdist节点与primuv节点需要的GeometryFIle ，如果是引用的SOP的节点的话，我们需要在路径前面加上一个op：。

最后我们还可以用 Channel Copy来自由组合通道，使用rop file output来输出既可，注意 如果要输出透明通道，需要把预乘Alpha关闭。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 第三种 ： 使用BakeTexture

第三种比较简单，我就简单记录一下流程，我们先在geo中直接输出一个物体，这里我直接用一个内置test模型

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

然后我们转到Shop层级新建一个材质 material shader builder(也可以在mat层级建一个 随意啦)。我们就简单输出一个法线N吧，里面还可以自己Bind 需要的属性输出。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

有了材质之后，我们给我们的geo的Render上 上我们的材质。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

再转到out层级，新建一个baketexture节点，讲我们要烘焙的物体路径给上 UV Objector(我们也可以高模烘焙到低模，将对应模型拖到对应栏即可)，调整好输出路径。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

接下来就是我们要输出什么信息，渲染那些通道。这里我只要法线N就好啦，下面那个是我们可以渲染额外的通道。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

然后直接点击 Render to Disk就好啦，渲染前我们还可以 点击 Render to MPlay 预览一下结果，记得要选择一下你要预览的通道。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

剩下还有个 mantra 可以烘焙出图。但是常用于给 云呀 烟雾呀这些拍明暗、法线这些渲染出图，这里暂时先不讲了=。

