# Flowmap

![Flowmap.drawio.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Flowmap.drawio.png)

### 输入输出：

对于Flowmap的输入，其实只有必要的三个类型，首先是水面(含引流线)，然后是和水面相交互的地形信息和与水面相交互的碰撞信息。

对于输出，我们需要把处理过的水片输出，在flowmap循环处理阶段调用Python输出flowmap贴图到指定文件夹，以及保存一份点云信息的bgeo文件。

核心处理流程：

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

//////////////////////////////////////////////foreach_begin/////////////////////////////////////////////

### 水面预处理

当水面数据输入进来之后，我们需要做的就是数据的分类以及剔除，首先我们需要把水面Pack给Unpack了，将引流线数据与水面数据相互分离，可以根据我们预先给引流线打上的tag来区分引流线与水面，用blast来分离两类数据，这样我们就得到了引流线数据与水面数据，接下来我们会分别对这两种数据进行再次处理。

首先处理的是水面数据，对于水面，我们要分离出当前要处理的水面A与其余水面(1-A)，A水面我们需要检测它的Tag类型，判读它是河流还是湖泊，当A水面为不同类型水面的时候使用不同类型的参数，比如细分的程度、flowmap模糊强度等等。通过Tag切换不同的参数设置。(这里可以根据各自需求自行调整)，有了细分参数之后，我们就可以使用remesh来将我们的网格给细分到均匀密度上。注意当我们remesh之前需要进行一步clean节点清除错误的数据。细分之后为了让密度均匀一些，我们可以再使用一个subdivide节点进行depth为1的细分。操作完之后便可以准备与（1-A）水面进行merge操作。

（1-A）水面我们需要做一个粗略的剔除，只保留与A水面相接的一部分水面，这步操作的意义在于让水面交接的地方白沫信息继承过来，让相接处的白沫不会断裂。

```c#
float dist = xyzdist(1, @P);
if (dist > ch("MaxDist"))
{
    removeprim(0,@primnum,1);
}
```

我们计算白沫等信息只需要(1-A)水的网格信息，并不需要（1-A）水面的UV，如果（1-A）的UV信息带入计算，可能在计算flowmap结果会出错。这里我并不打算删除(1-A)的UV信息(删掉后续操作可能会出现奇怪错误)，而是将(1-A)的UV信息设为0。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

接下来就是使用remesh来重新控制网格密度，这里remesh的大小参数可以之间Link到水面A的remesh参数大小，让两边参数统一、密度统一，remesh之后还是使用一个subdivide细分一次既可。再使用一个Group节点标记当前Mesh为TransitionMesh就可以和水面A merge起来进行下一步操作了。（merge起来细分会使两边网格合并比较麻烦，所以分开细分）

引流线方面我们首先使用一个resample细分为密度均匀的曲线，之后我们需要找到影响水面流向的曲线点，我们可以先把两边高度归0，使用xyzdist()找到离水面一定距离内的点保留下来，把超出水面一定距离外的点删除，再把曲线高度还原，就得到了影响水面流向的引流线部分。

### FlowMap设置：

1. 生成flowmap
     为了快速实现flowmap相关一些东西，这里我主要使用SideFxLabs的一些东西(其实节点里面的逻辑并不难)，首先习惯性使用一个Flowmap先根据法线生成基础层的V向量。

再使用Flowamp_guide来根据引流线计算流向信息，flowmap_guide节点里面其实很简单就是计算引流线的切线属性v，通过attribtransfer转移到网格上。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这里值得注意的是需要注意以下attribtransfer的conditions下的参数，如流向线影响范围、强度等参数需要暴露在HDA面板。我这边是通一开始水面类型判断决定不同类型水面用不同参数。

接下来我们需要自己定义流动速度快慢，我们可以根据河流的Slope值来控制，对于Slope我们可以ramp到新的参数上自行控制范围、强度等。然后使用 v@v = normalize(v@v) * @slope;既可 (flowmap_guide计算出来的v就有大小变化，我们需要自定义控制，只需要归一化矢量方向既可)

1. 生成flowmap碰撞信息
生成河流碰撞信息我们通常使用flowmap_obstacle,里面的逻辑也很简单，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

就是先生成体素，因为体素比较容易拿到梯度向量，然后直接在vop里面采当前位置的梯度作为v的方向，blur之后与原向量v相加既可以。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![%E6%A2%AF%E5%BA%A6%E5%8F%AF%E8%A7%86%E5%8C%96.gif](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/%E6%A2%AF%E5%BA%A6%E5%8F%AF%E8%A7%86%E5%8C%96.gif)

flowmap_obstacle知道怎么算了，我们就要处理我们的输入物体了，

对于石块、外部导入碰撞等碰撞物体，都为HDA输入，我们可以直接拿到。

对于地形碰撞信息，我们还得做一步裁剪处理，首先我们使用水面投影在地形上获得水面投影Mask，对Mask进行一次maskexpand，使用heightfield_crop裁剪出mask区域，再convertheightfield把地形转换为Mesh，delete掉@mask<0.1的Mesh，就可以得到与水面做碰撞的地形信息啦。

1. 生成深度信息(可选)
对于某些情况，我们需要预先烘培出水体深度信息(这里的深度指垂直深度并不是真正深度 摄像机到水底)，我们可以使用上一步得到的地形Mesh做垂直射线检测便可以得到水体深度，这个深度可以拿来做水体颜色变化、边缘波浪、边缘白沫、焦散等等(也可以烘培SDF代替深度，好处就是均匀平滑)，

```c#
float maxDist = chf("maxdist");
vector hitpos, uvw;
float h1 = @P.y;

float dist = 0;

int hit = intersect(1, @P, set(0,-1,0) * maxDist * 10, hitpos, uvw);
if (hit > 0)
    dist =@P.y - hitpos.y;
    

@depth = fit(dist,0,maxDist,0,1);
```

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

1. 生成白沫
白沫的生成主要是三部分：与地形碰撞的白沫、与碰撞物碰撞的白沫、瀑布处的白沫，外加一步径向模糊操作。

得到地形碰撞的白沫与碰撞物的白沫首先需要我们提取地形与水面 、碰撞物与水面的相交线。使用intersectionanalysis节点既可获得。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

拿到相交线之后，我们便可以使用nearpoint找到最近的相交线上的点，根据distance(p, @P)得到与相交线距离，并通过 P-@P向量与当前流向v向量做点乘既可得到与水面方向做强度衰减。

```c#
float maxDist = chf("foam_size");
int num = nearpoint(1, @P);
vector p = point(1, "P", num);
vector v0 = normalize(p - @P);
float d = distance(p, @P);
vector v = point(2, "v", @ptnum);

float atten = clamp(1.0 - d/maxDist, 0.0, 1.0) * length(v);
float dir_atten = clamp(fit(dot(v0, normalize(v)), 0.4, 1.0, 0.0, 1.0), 0, 1);
@Foam += atten * pow(dir_atten, chf("foam_power"));
@Distance = d;
```

(这里point(2, "v", @ptnum);2号端口为第一步生成flowmap完得到的geo，这里v并没有被第二步干扰)

(这里也可以记录下distance(p, @P)的距离，存贮为河流的sdf信息，制作岸边波浪效果)

对于瀑布白沫，我们也只需要把相同瀑布部位的点提取出来连接为线，做同样操作既可。

得到相交处白沫大致位置之后，我们可以适当的模糊一下，就可以进入径向模糊阶段

```c#
float foam = 0;
vector p0 = @P;
float w = 0;
float stepsize = ch("Step_size");
int step_num = ch("Step_num");
float k = 5 / (float)step_num;
for(int i=0; i<step_num; i++)
{
    vector p;
    vector uvw;
    int prim;
    prim = intersect(0, p0+{0,10,0}, {0,-100,0}, p, uvw);
    vector v = primuv(0, "v", prim, uvw);
    p0 -= v * stepsize;
    prim = intersect(0, p0+{0,10,0}, {0,-100,0}, p, uvw);
    float f = primuv(0, "Foam", prim, uvw);
    float weight = exp(-(float)i * k);
    foam += f * weight;
    w += weight;
}
foam /= w;
foam = clamp(foam*2.0, 0.0, 1.0);
@Foam = foam;
@Cd.b = @Foam;
```

到这里，我们的白沫就处理完了。

### Flowmap输出：

对于flowmap输出部分，我们需要先blast掉我们一开始输出进来的TransitionModel 衔接水面。之后我们需要针对Unity与Unreal以不同的坐标系进行编码，大致代码如下

```c#
vector col = v@v * 0.5 + 0.5;
@Cd = set(col.x, col.z, @Cd.z);
```

通过swtich选择对应的输出引擎。

最后连上一个clean，就可以使用maps_baker输出flowmap贴图啦(数据贴图 Gamma通常为1)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

(输出路径使用引用，不要使用Python动态设置输出路径，这样需要这个Node设为Editor)

为了定制A通道数据我们需要进maps_baker里面更改一些东西

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

注意 需要图片输出的预乘Alpha改为Unpremulitiplied

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 碰撞水花设置：

得到碰撞水花这一步，我们需要使用生成白沫阶段获得的地形与水面 、碰撞物与水面的相交线，我们使用object_merge link到这些线段，使用attribtransfer把原来地形的法线、碰撞物的法线转移到相交线上。我们再通过vex判断水面流向方向与相交线法线的角度，当角度小于30度时，我们将生成水花粒子，为了更智能，还得判断生成点的水面流速大小，当流速大于阈值时候，我们使用大型水花粒子，当流速小于阈值时候，我们使用小型水花粒子，不同类型粒子的pscale缩放也可以根据流速大小进行一定范围内的缩放。

之后便是使用fuse融合相隔比较近的点云，得到最终点云后，我们还需要将点云位置向法线方向略微位移一些。因为我们设置了不同类型的水花粒子，最后我们还得使用blast将两种类型水花点云分开设置unreal_instance，设置完之后便可以准备合并输出。

### 瀑布水花设置：

得到瀑布我们需要用到河流输出的引流线，上面有我们一开始设置的 fallstart 与 fallend点，我们可以将瀑布起始点与终止点 blast 出来，将瀑布点云赋予上他们对应的unreal_instance属性，我们只需要设置好水花特效的旋转既可,设置完之后便可以准备合并输出。

### 设置反射探针(可选)：

无论是unreal 还是 unity 我们都可以选择性的开启或关闭这个功能。河流的探针比较好弄，就是将引流线 resample 成密度合适的线段，通过add节点的delete geometry but keep the points 保留下点云信息，将点云上冗余的数据删除之后我们边可以为点云赋上对应的unreal_instance属性了，湖泊的话，我们需要将湖泊mesh单独提取出来，foreach循环每个湖泊mesh，使用measure计算每个湖泊的mesh面积 ，measure节点能计算出每个prim的面积，用attribpromote节点将prim的area属性 以 sum的方式转移到detail 我们就可以得到湖面的总面积，再使用scatter节点通过湖泊面积除以一个自定义大小就可以得到一个合适的密度，注意我们需要调整scatter的relax 相关属性使点云分布更加均匀。得到点云后设置好unreal_instance，设置完之后便可以合并输出。

### 水面网格设置：

这一步主要是设置顶点色，以及法线平滑。顶点色需要根据各自项目不同材质 自行设置，这里我以我自己的顶点色信息设置为参考，R通道为混合信息：一个水材质可以支持两种不同类型的参数 通过 顶点色R通道做 lerp，G通道为透明度信息：可以通过手动刷顶点色或自动设置顶点色来控制透明区域，B通道为流速信息：可以通过B通道来控制波浪大小，当流速缓慢时产生的波浪较小，当流速快时产生的波浪较大。

设置水面R通道我们在CombineMesh HDA便设置好了，G通道 我们可以使用Group节点的Include by Edges功能选择水面边缘，如果水面边缘没有与其他水面相接那么我们将判断水面边缘是否在地形之上，如果水面边缘在地形之上，那么我们将边缘顶点色G通道设置为0。

```javascript
float dist = xyzdist(1, @P);
if (dist > 0.5)
{
    float h = volumesample(2, "height", @P);
    vector p0 = point(2, "P", 0);
    h += p0.y;
    if (@P.y - h > 0.05)// && @Cd.r < 0.5)
        @Cd.y = 0;
}
```

对于B通道，我们可以使用attribtransfer来将引流线上的slope属性转移到水面mesh网格上，注意设置conditions面板里的distance threshold属性(可以不勾选)，接下来使用attribute remap节点让我们可以更好的控制slope强度曲线，最后将slope属性设置到B通道既可。

最后就是水面mesh的输出啦，这个看个人流程，我这里就将水面mesh存为Fbx文件，最后通过点云来控制水面Fbx的生成位置。

