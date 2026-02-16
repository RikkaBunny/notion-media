# SplitWaterMesh 

裁剪水面分为两大类型，一种是以小块InstanceMesh来拼接成水面，(以前的Unity地形便是如此)

另一种是以地形划分的Cell为单位，切割为一块一块StaticMesh。这里我们通常使用第二种。

得到boudingbox有两种简单方式，一个就是bound节点，另一个就是Labs的multi_bounding_box节点

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这部分其实很简单主要是根据我们地形具体streaming的包围盒大小，作为我们裁剪使用的boundbox，遍历每个包围盒，然后嵌套遍历每条河流， 对每条河流做Boolean操作，取Intersect部分，这样就可以得到根据地形Cell划分的水面Mesh，最基础的功能这里就已经实现啦。

这时候按5键可以看到我们UV大概会只占据0-1之间的一部分。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

注意有时候按5键，界面是黑的什么都出不来，这时候我们可能需要刷新下界面，heightfield有时候显示出错，同理也可以使用刷新既可。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

当我们UV只占0-1之间一部分时候，有些精度就浪费了，这时候我们可以通过uvtransform来把uv自动移动与缩放。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

第一步，以uv的X最小值为轴点，向X的方向移动 负X最小值。

第二步，以uv的X最小值为轴点，缩放 1/X的最大值。

这里我们就实现了填充UV 0-1空间的功能。(这里我的Y轴为河流宽一直是UV.y 0 -1范围所以并不需要做uv.y方向的位移与缩放)

接下来我们便是准备重新输出我们的水片，这里我们需要把水片的名称Tag重新赋予水片，并且加上两个新Tag，一个为当前水片名称+切块Index的Tag，一个为当前地块名称的Tag，有了这几个Tag，我们之后的处理将会更方便。之后便是Pack起来，为Pack的时候可以随手将我们设置的Tag Transfer Attributes过去，方便之后更好操作。(注意如果是河流的话需要把引流线一起Pack)

