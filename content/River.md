# River 

![Water.drawio.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Water.drawio.png)

## 输入输出：

首先 ：
输入与输出有

![Image.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Image.png)

![wEf7nwu3RR4wQAAAABJRU5ErkJggg==](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/wEf7nwu3RR4wQAAAABJRU5ErkJggg==)

![D+H4a4cN0pAkQAAAABJRU5ErkJggg==](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/D+H4a4cN0pAkQAAAABJRU5ErkJggg==)

支持最小输入两个、最大输入3个分别为：河流原始流向曲线、地形、河流水面Mesh
支持3个输出分布为：最终结果预览、地形、河流水面Mesh

### 设置曲线：

首先去曲线进行resample细分，保证曲线密度均匀，使用jitter在xz平面抖动曲线，把曲线吸附地形与水面上，比较曲线0point与@numpt-1高度，确定曲线头尾，保证水流从高到底流向。如果有输入3其他水面，提取输入3水面中的引流线与曲线的头尾点做距离判断，如果距离过近认为两条河流相交将曲线自动吸附再其他河流曲线上。

![%E9%9A%8F%E6%9C%BA%E6%8A%96%E5%8A%A8.gif](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/%E9%9A%8F%E6%9C%BA%E6%8A%96%E5%8A%A8.gif)

为了避免曲线在太过陡峭的地形在生成河流，我们可以循环对河流曲线做沿着地形坡向滑动操作避开陡峭区域。

![CurverSlide.gif](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/CurverSlide.gif)

最后还需要贴合地形与水面一次，并且保证曲线从高到低，上一个点的高度必须大于或等于下一个点。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 生成瀑布：

进入生成瀑布节点可以先使用resample均匀曲线密度，然后根据曲线坡度(斜率)选择瀑布，直接用Group节点就能实现，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

通过指定瀑布的起点和终点得到两两配对的瀑布点

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

根据瀑布的起始点与终止点，我们可以得到当前瀑布端的长度和位置，我们可以根据需求删除过短和距离过近的瀑布，之后我们需要调节瀑布的坡度，就是再曲线上滑动瀑布终止点位置，这个对于生成的瀑布只能做到全局调节他们的坡度

![Curver.gif](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Curver.gif)

之后我们还需要支持人工设置瀑布，一个Multiparm Block(List)来存瀑布参照

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

使用循环遍历来应用上每一个瀑布，保证最后瀑布终止点不低于河流最低点，最后保证每个瀑布高度不低于一个阈值既可。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 设置河流属性：

我们需要模拟流水侵蚀的两种主要形式(溯源侵蚀可以不做):下蚀、侧蚀作用：

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

下蚀很容易，就只需要根据侵蚀力度加深河流深度既可，侧蚀的话需要先定义一个 -1到1 的noise，用来向左或者向右扭曲河道。需要注意的是侧蚀操作要避免河流的瀑布区域还有头和尾扭曲。
之后我们还要计算河流的切线方向、河道下挖的方向以及河流的斜率等你需要的信息，为了计算精度，我们可以先提高精度计算然后再转移到现有曲线上。

```c#
vector forward = normalize(v@tangentu);
vector right = normalize(cross(forward, v@up));
v@depthDir = normalize(cross(forward, right));
```

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

设置完这些之后，我们可以设置河流的宽度以及深度，这里我喜欢通过Ramp图参数以及一些额外的noise来控制整条河流的宽度与深度，注意河流的宽度还和河流的斜率有关，这里我们为了模拟水流越平缓越宽阔、越陡峭越细窄的特点还需要乘以一个河流的坡度。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这里ramp大小可以超过0到1.

### 转换为河道曲线：

这里要注意看看我们参数列表

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

河道我们可以理解为一个拥有八个顶点的线段，一共四个宽度控制，0、7点为河流的总宽度，1、6点为河岸宽度，2、5点为河面宽度，3、4点为河底宽度，线段通过copytopoints生成在河流曲线上，通过河流曲线顶点上的pscale参数控制线段大小(顶点处当前河流宽度)，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 曲线重叠修复：

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

思路主要是用找到曲线自身相交的点，提取相交打结的点，把点均匀分布在离相交点最近的两个点之间。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

使用intersectionanalysis节点就能得到相交的点，打开PrimitiveUVW就能得到相加uv范围。你得到相交点 uv之后，你就有一个 uv 区间。

```javascript
@maxu = max(v[]@sourceprimuv[0].x,v[]@sourceprimuv[1].x);
@minu = min(v[]@sourceprimuv[0].x,v[]@sourceprimuv[1].x);
```

你就可以将 这个 uv区间的点得到给他们上一个 fix的属性标记，之后就可以将顶点均匀分布在 fix-1 到 fix+1 顶点这个区间。

```javascript
int minpoint=0;
int maxpoint=0;
float fixNum = 0;
float curFixNum = 0;

for(int i = 0; i < npoints(0); i++){
    int pointfix = point(0,"fix",i);
    if(pointfix==1){
        minpoint = i-1;
        break;
    }
}

for(int i = npoints(0)-1; i >= 0; i--){
    int pointfix = point(0,"fix",i);
    if(pointfix==1){
        maxpoint = i+1;
        break;
    }
}

for(int i = 0; i < npoints(0); i++){
    int pointfix = point(0,"fix",i);
    if(pointfix==1){
        fixNum++;
    }
}

vector pA = point(0,"P",minpoint);
vector pB = point(0,"P",maxpoint);
vector dirATB = pB - pA;



for(int i = 0; i < npoints(0); i++){
    int pointfix = point(0,"fix",i);
    if(pointfix==1){
        curFixNum++;
        
        vector pos = pA + float(curFixNum/fixNum) * dirATB;
        setpointattrib(0,"P",i,pos);
    }
}
```

注意如果要计算线段UV，使用UVTexture节点需要把textuerType改为Rows&Columns.

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 设置河道曲线：

得到修正过后的河道曲线之后，我们可以开始做河道的扭曲(侧蚀)啦，我们提取河道中间的四根曲线，根据我们在河流属性设置哪里产生的侧蚀Noise来做扭曲，扭曲方向沿河流的切线方向(在河流属性设置步骤得到)，还得保证了河流只在河道内做扭曲不会超出河道。

```c#
vector B = normalize(cross(v@tangentu, {0,1,0}));
B.y = 0.0;

float factor = @meandering_factor;

@P += @pscale * B * factor * 0.45;
```

接下来我们调整整条河流的高度已到达自定义控制河流的高度，

```c#
float depth = chramp("floodplain_height",@curveu);
@P += {0,1,0} * depth;
```

然后我们使用当前河道与输入三的其他河流水片做一个高度匹配融合，在一定范围内当前水片匹配输入水片高度之后在慢慢lerp回原来高度。
我们需要遍历每条河道曲线，去设置他的宽度。注意我们需要把宽度控制在上一级宽度之下，0以上。河流宽度 > 河道宽度 > 河面宽度 > 河底宽度.

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 下挖河道：

设置完每条曲线的长度值之后，我们就要开始下挖河道辣，3、4点为河底，所以我们只需要把这两个点沿深度下降方向即可(属性在设置河流曲线属性存储)。顺便把曲线的0、7点设置权重为1(这个权重值主要是后面与地形融合的权重)。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

挖完河道之后我们可以使用Skin节点把曲线转换为我们需要的河道Mesh，给上Normal节点，现在的河道Mesh还过于简单啦，我们还需要使用remesh或者subdivide细分一下mesh，到达同等密度网格时，remesh的消耗要大于subdivide，但是remesh细分不规则网格效果比subdivide要好的多，这里我们结合一下，我使用的是 先remesh一次密度为2或者1都可，smoothing暂时为0(需要和水面高度保持，现在不能形变网格)，remesh之后接一次subdivide，depth为1或者2都可以，这样得到的网格颗粒度都比较均匀，注意在remesh之前我们需要使用clean节点保持网格的干净，剔除可能错误的值 比如：nan
接下来我们就要考虑网格的平滑和不规则的噪声，这里我使用一个attribblur和mountain来操作，但是，这两个操作使我们的网格不太可控，容易出现水面高出河道网格的现象，为此我们需要做一个权重来控制网格形变的大小，权重大小可以根据水片曲线边缘的两条曲线与地形mesh的距离得到(P.y等与0)，根据权重lerp原始未形变的河道和形变过后的河道，既可保证河道高于水片。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 下挖地形：

下挖地形我们前先，我们需要把下挖权重(下挖河道处设置)映射在地形上做为权重Mask，

```javascript
vector up = {0,1,0};
float maxdist = 10000.0;

vector hitpos;
vector hituvw;
int hitprim;
vector raydir = up * maxdist;

hitprim = intersect(1, @P-raydir*0.5, raydir, hitpos, hituvw);
if (hitprim >= 0)
{
    vector p = primuv(1, "P", hitprim, hituvw);
    float weight = 1.0 - primuv(1, "Weight", hitprim, hituvw);
    float h = weight > 0.99 ? p.y : @height;
    vector origin = hitpos;
    @mask = weight;
    @height = lerp(@height,p.y,@mask);
}
```

对于地形下挖我们常遇到的就是转角处高度错乱，我们通过提前处理Mesh可以避免，这里我们再加一次常用的处理操作，就是第一次hit打中后继续向上检测一次，如果还打中，那么说明这个地方mesh重叠，我们需要把当前位置Mask设为1，没有重叠的就是0。然后根据mask轻微blur之后进行heightblur操作。

### 设置地形属性：

这一步我们需要提前我们做地形地表时候需要的mask信息，我们可以得到我们想要的一些信息：比如河岸、河流区域等等。。这一步看个人啦。
在把做好的Mask信息与输入的地形原有Mask做Max叠加。输出既可。


### 设置引流线

我们将 转换为河面曲线 过程中由2，5号曲线得到的河面线段，通过Remesh固定数量2 再通过skin转换为3条河流曲线，我们提取其中的1号曲线就是我们的河流引流线啦，得到引流线我们resample为一条密度比较均匀的线条，删除其中冗余的数据，并且为自己打上Tag就可以啦。

### 转换为河面曲线：

在设置完河道线段之后，使用skin的columns后，提取线段的所有2号、5号点组成的曲线，也就是河面，再次skin得到河面线段，现在的河道线段只有两个端点生成的mesh太过稀疏，为了mesh的密度均匀(密度均匀shader做WPO效果比较好)，我们可以对河面每条线段做resample操作，这里的resample可以自动适配也可以固定数目。自动适配就用measure计算每条线段的长度，通过attribpromote可以得到平均长度，再通过参数取得我们之前resample河流曲线的间隔密度。有了这两个参数就可以算出resample的值啦。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

### 设置河面属性：

我们需要用skin把河面线段又转换为河面曲线，然后提取河面曲线两侧边缘曲线，0号与`nprims(0)-1`号点，使用ramp参数动态扩张两条曲线任意位置。

```c#
//int值最后会四舍五入
int inum = @ptnum / @numprim;

vector p = point(0, "P", inum * @numprim + (@numprim-1 - @primnum));
float water_depth = chramp("waterSurface_expansion",@curveu);
@P = p + (@P - p) * (1.0 + water_depth);
```

接下来需要与输入的水面做贴合，水面附近也需要渐变贴合。

### 设置河面UV

河面UV的核心算法就是把UV合理的分布在0-1空间里，也可以根据河流长度缩放河流的uv.x

```javascript
@uv.y=@ptnum % @numprim / (float)(@numprim-1.0);

@uv.x = (@ptnum/ @numprim)/(float)(@numpt / @numprim);
```

uv缩放可以使用uvtrnasform 或者 vex，配合measure节点，主要代码为
detail("../length/", "perimeter", 0) / detail("../width/","perimeter", 0) * uv.x;
我这里选择使用第一种0-1空间的uv，作为主uv，第二种放在uv1上。

### 转换为河面Mesh

通过skin变为河面mesh，给上normal节点，注意顶点顺序会导致河面法线可能向下，通过attribpromote节点把法线平均值传递给detail属性，判断N.y是否大于0，从而得知法线是向下还是向上，如果向下使用erverse反转顶点既可。

### 设置河面Mesh

这里我们需要把当前水面与输入的水面做一次粗略的剔除，并且也可以选择性的剔除超出地形的水面Mesh，做粗略剔除时，注意要把两边mesh的P.y都归到0的位置，这样可以去除Y轴导致的偏差，在后面boolean的时候也会用到这种方法.
使用xyzdist()判断每个顶点是否与输入水片相叠，如果相互重叠那么我们标记这个顶点，之后我们在遍历每个prim，判断这个片元的每一个顶点是否有这个重叠标记，如果当前片元里面所有顶点都满足重叠，那么我们可以删除这个片元。
为了保证水面贴合，这里我们可以再做一次贴合水面的操作，并且重新再计算一边法线。

### 输出水片

输出水片这一步我们需要清理水片Mesh的一些冗余信息，并且设置我们需要的信息标记当前Mesh，比如当前Mesh的name等，名字我们可以使用 $HIPNAME+`opname("../../../")` +`opname("../../")`获得当前唯一ID，这个可以作为Mesh的name，完成之后与河流引流线合并，并Pack起来设置Tag后输出。

最终输出的时候我们可以地形和水面merge起来输出到 output0来显示，实际数据用output1与output2传递，这里我把顺便把输出节点颜色改变一下，使用相同颜色对应相同颜色连接。就此我们的河流HDA完成。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

