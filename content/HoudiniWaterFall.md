# HoudiniWaterFall 

![waterfall1.gif](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/waterfall1.gif)

### 设置瀑布粒子：

因为我们所输入的Mesh密度、大小等是不确定的，所以对于瀑布粒子我们的策略是先粗略的选择我们需要产生的地方，之后再精细的选择我们所产生的粒子。这里我们可以使用Sctter节点来选择我们需要的地方产生瀑布粒子。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

注意选择的时候需要开启视窗 只选择可见 那个小眼睛。

选择完之后我们就可以继续精细选择我们需要的瀑布粒子啦。我们只需使用Blast选择我们需要的Points既可。得到我们瀑布粒子之后，我们便需要选择一个点作为发射力的原点。这样我们就可以通过 @v = normalize(curPos - firingPos);得到我们的发射力的方向啦。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最后我们还可以连上一个Add节点用来打破所有geo只保留点云。为了保证粒子点的高度还得更具Mesh高度重新调整一下高度。

```javascript
vector hitpos;
vector uvw;

int hit = intersect(1,@P,set(0,3,0),hitpos,uvw);

if(hit < 0){
    hit = intersect(1,@P,set(0,-5,0),hitpos,uvw);
    if(hit < 0){
        removepoint(0,@ptnum);
    }
}

@P.y = hitpos.y+0.1;
```

### 模拟与缓存：

这里我们使用的并不是真正的Fluid模拟，我们可以使用Houdini的Vellum里面的专门模拟沙子的vellumconstraints_grain用来模拟水流轨迹，主要的结构便是这样。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

Grain节点一个0输入点云、2输入碰撞体，模拟的具体参数我就不讲了，没有特殊需求默认就可以，

![FallSim.gif](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/FallSim.gif)

接下来给每个点打上Tag，好让接下的Add节点通过Tag连接成线。

这里我们通过Solver来获得上一帧点（也可使用trail节点），并且可以通过自定义的规则来剔除不需要的点。这里我剔除速度为0的点并且通过自定义参数来控制线段密度。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这样我们就可以得到线段轨迹啦。

通过TimeShift就可以得到稳定的一帧状态，因为数据量大每次的计算和内存Cache不稳定我们可以使用FileCache节点把结果缓存在硬盘中方便下次读取。推荐使用唯一名称来存储路径+/$HIPNAME/`opname("..")`.bgeo，

### 曲线剔除与选择：

对于瀑布曲线的选择我们可以使用分簇剔除 - 程序剔除 - 手工剔除的步骤来获得我们需要的瀑布曲线。首先对于得到的曲线数据，我们需要遍历我们每一根曲线，我们可以发现我们的曲线是由一根根的线段组装起来的，我们就需要曲线变换为一根线段（一根曲线由多个prim变为一个prim），我们使用我们留在点上的Tag来进行分组遍历，通过Add打散每个线段，再使用Add把所有点连接为一个新的线段。

接下来使用measure节点来测量每一根线段的长度，然后attribpromote把线段的平均长度记录再Detail属性里面，再通过判断当前每根曲线的长度与平均长度的差距值来剔除一些短小错误的曲线。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

为了加速我们的处理，我们可以使用重要性采样的概念，通过曲线位置进行一个分簇处理，然后通过随机百分比删除来剔除每一簇中的一部分的曲线，这样剔除能使分布更加均匀。

我们需要将每条曲线Pack起来，这样就可以使用一个点来代替一条曲线，将曲线替代点的Y轴存贮在prim属性里面，再将Y轴归零，这样我们就仅仅在XZ平面上做基于位置的曲线分簇操作。使用cluster节点我们就可以得到分好簇的曲线。将Y轴属性恢复并把cluster属性带入color节点可视化显示：

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

接下来便可以Unpack，将cluster属性转移到prim，使用循环遍历prim以cluster作为piece Attribute，循环内使用vex按照一定百分比去剔除当前簇的曲线。

```javascript
float threshold = chf("threshold");
int r = chi("random");
float rand = random(@num*r);
if(threshold < rand){
    removepoint(0,@ptnum);
}
```

下面我们就要遍历每一条曲线，然后剔除隔得太近的瀑布曲线，这里我使用的是先给resample过后的线段每个点copy一个球当它的检测范围，然后我们就可以控制这个球的大小从而控制检测范围，然后通过Group的Keep in Bounding Regions这个功能来标记附件有哪些曲线在这个范围内。然后把标记数据转移到整体线段上，最后剔除掉当前线段。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这里我还额外控制了当速度小于一定阈值的时候不不生成检测范围球。最终我们就可以得到符合一定条件的瀑布曲线啦。这一步的目的是控制瀑布之间距离，防止进入引擎之后半透重叠部分太多导致drawcall。(美术效果与性能的自行权衡)

最后我们还需要留一个人工选择删除的操作。如果有不想要的曲线方便我们自己可以选择性删除。

### 设置曲线：

这里主要还是设置曲线的宽度属性，其他属性和操作可自行选择。

设置瀑布宽度时，我们需要给出瀑布大小表现的一些对比度，所以我们需要两种不同宽度大小的瀑布，而区分这两者我们可以通过速度给一个阈值，调节这个阈值我们便可以控制大瀑布片与小瀑布片的数量比例啦。

首先我们resample一下瀑布曲线，均匀一下曲线密度，记得勾选下面的Curve U Attribute这个复选框，它会为我们产出一个Curveu的属性，代表着当前点在当前曲线上所占的比例。使用attribpromote将曲线点的v属性平均值转移到prim里面，通过这个平均速度，我们便可以使用一个阈值区分大瀑布与小瀑布，并使用Noise来扰动当前瀑布宽度。

```c#
vector v = prim(0,"v",@primnum);
v = set(v.x,0,v.z);
float vlength = length(v);

float noise = rand(@primnum+chi("rand"));

float widthL = chramp("widthL",@curveu);
@pscale = widthL*fit01(noise,1-chf("noiseRange"),1+chf("noiseRange"));

float widthS = chramp("widthS",@curveu);

if(vlength < chf("v_Threshold")){
    @pscale = widthS*fit01(noise,1-chf("noiseRange"),1+chf("noiseRange"));
}
```

为了接下来计算的UV密度均匀，我们还需要使用measure来计算当前曲线的长度(用作V方向的缩放)

### 转换为Mesh：

foreach遍历每条曲线，用Polyframe使法线为切线方向，设置@up={0,1,0};顺便可以把曲线的斜率一起保存@slpoe=@N.y；之后还会更加斜率做一些额外的操作。

接下来就是把线段Copytopoints到曲线上，这里有个小的注意点，我们copy的线段不一定是直的，我们可以将线段做成有一定弧度，这影响着瀑布的表面是曲面还是直面。接下来就是Skin转换为网格在用normal给上法线，注意这个法线方向可能会向下，我们需要得到法线平均值，判断N.y是否小于0，如果小于0，我们需要reverse一下在重新计算法线。

因为我们是通过线段Copytopoints生成的Mesh所以我们可以找出UV分布的规律。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

瀑布UV的核心算法就是把UV合理的分布在0-1空间里，然后根据瀑布的长度与宽度缩放瀑布的uv

```javascript
@uv.x=@ptnum % @numline / (float)(@numline -1.0) * @width;

@uv.y = (@ptnum/ @numline )/(float)(@numpt / @numline) * @length;
//@numline为存储Copy线段的点数
```

我们还可以为这个UV缩放加上一些noise与offset ，确保让每片瀑布表现都各不相同。

接下来还得完成一个表现效果，就是当水流的快慢表现，就是在水流垂直下落的时候流速会加快，在触碰石头的时候水流速度会变慢，这个现象我们可以通过控制UV的拉伸来实现，水流的速度，我们可以通存贮的斜率属性来得到，使用@slope就可以控制我们UV的拉伸。

![waterfall111.gif](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/waterfall111.gif)

```c#
@addValueSum = 0;
int curType = (@ptnum % @numline );

for(int i = 0; i < @ptnum-(@numline -1.0); i++){
    int type = i % @numline;
    if(type==curType){
        float slope = point(0,"slope",i);
        float uvAddValue = (1-slope);
        @addValueSum += pow(uvAddValue*ch("intersity"),ch("contrast"));
    }
}

@uv.y+=@addValueSum;
```

到这，我们瀑布的主要功能就以及做完。

### 网格设置：

得到瀑布Mesh之后，我们可能还会面临一些材质上的问题，这里我们需要把一些信息写入到顶点色中，以便我们可能会在shader中使用，这里我存了瀑布的Slope信息以及瀑布尾端的一个Alpha渐变信息以便我们可以在材质表现上做出瀑布尾部淡出的效果。我们还可以进行一些容错处理，比如瀑布的顶端部分需要贴合地形等等，这些都需要各个根据自己情况做可选处理，我就不过多讲解。

最后给上一个平滑的法线并clean一下便可以输出。

### HDA设置注意：

因为我们有timeshift和缓存机制，所以我们在每次更改瀑布模拟帧数时候都需要重新设置timeshift和缓存，这里我使用HDA参数的callback机制调用我写在Scripts里面的方法，这样就可以优雅的更新啦。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

当我们把所有东西封装为HDA的时候，当HDALock的时候，在我们选择需要的points、prims等，会发现不对，我们选择的物体与当前显示的物体无关，当我们把HDA解锁后便可以正常工作，后面观察了一下，发现当我们点击Selection角标时候 我们的RenderFlag便会在我们需要选择的节点geo上设置为Display。

后面我在Button函数里面强行设置它的RenderFlag，并且把需要的节点设置Editable，当HDA锁上之后发现我们的强行设置RenderFlag的代码并不起作用，当HDA解锁之后又恢复正常。

最终我选择一个取巧的方法，使用一个Switch节点来设置HDALock状态下不同节点的RenderFlag。下面为代码与节点操作。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

