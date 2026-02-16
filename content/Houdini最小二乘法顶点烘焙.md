# Houdini最小二乘法顶点烘焙

https://users.cs.utah.edu/~ladislav/kavan11least/kavan11least.pdf

https://www.twblogs.net/a/5ef0c2afc3cd5d0a4793eb37   可能需要科学上网

做这个东西的起因是 Houdini原本的attribtransfer节点出来的效果并不能满足我的需要，我的底模的效果过渡始终没有那么平滑，与高模烘焙出来的数据还是有一定差异。所以去找了下资料发现确实有更好的方法。那么接下来就是在Houdini中实现啦。

原理说明

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

出发点是这篇论文，d图是高模的烘焙结果，a 、b 、c ，分别是 (a)点采样方法、(b)平均三角形采样点、(c) 最小二乘法。

步骤大概是houdini里面由输入低模细分为高模，离线烘焙需要的数据，这种数据一般为大范围且低频的AO或者GI数据，再将这类数据由高模传递给低模，这里我将AO烘焙数据记录给顶点颜色。

这篇文章比较简洁的思路说明便是 使用采样点得到高模与低模在同一位置的颜色，用最小二乘法残差平方和越小 ，两曲线越拟合的思路去通过梯度下降法一步一步的去迭代低模与高模两者之间颜色的残差平方和。

首先我们要了解一下 点在三角形中的重心坐标概念，简单来说 一个三角形内的任意一点的颜色值，都是由三角形的三个顶点颜色值通过重心坐标插值而来的。

所以想知道任意采样点对所在三角形的三个顶点的影响，我们需要求得当前采样点的重心坐标。我们需要存储记录每个采样点 P 所烘焙的信息（AO or GI），这里我记录的烘焙的Cd(AO值)，还有P点所在点的位置pos，P点位于低模哪个三角形上，记录下P点所在低模三角形的prim 与 uvw。

```javascript
//得到离高模点最近的低模prim与uvw。
int prim;
vector uvw;
float dist = xyzdist(1,@P,prim,uvw);
//得到采样点对应低模上的Pos
vector pos = primuv(1,"P",prim,uvw);

i@L_prim = prim;
v@L_pos = pos;
v@L_uvw = uvw;
```

我们先使用divide将低模三角化，默认每个Prim都只有三个顶点。

遍历低模上的每一个Primitives。

```javascript
v[]@barycentric;
//遍历高模每个采样点。
for(int i = 0; i < npoints(1); i++){
    int prim = point(1,"L_prim",i);
    if(prim == i@primnum){
        // This is the point we want to find barycentric coordinates of
        vector p = point(1, "L_pos", i);
        vector pColorGap = point(1, "Cd", i);
        vector pUVW = point(1, "L_uvw", i);
        
        // These are the vertices of the main triangle we want to find coordinates for
        i[]@pts = primpoints(0,i@primnum);
        vector v1 = point(0,"P",@pts[0]),
               v2 = point(0,"P",@pts[1]),
               v3 = point(0,"P",@pts[2]);
        
        // Edge Vectors of the main triangle
        vector e1 = v3 - v2,
               e2 = v1 - v3,
               e3 = v2 - v1;
        
        // Sub Triangle Edge Vectors created using p
        vector d1 = p - v1,
               d2 = p - v2,
               d3 = p - v3;
        
        // We need a normal vector so generate one from the edge vectors of the main triangle
        vector n = cross(e1, e2) / length(cross(e1, e2));
        
        // Now find the area of each triangle using the triple product: (a × b) ⋅ n
        float  AT = dot(cross(e1, e2), n) / 2,
              AT1 = dot(cross(e1, d3), n) / 2,
              AT2 = dot(cross(e2, d1), n) / 2,
              AT3 = dot(cross(e3, d2), n) / 2;
        
        // Now we divide the area of each subtriangle against the area of the main triangle
        float u = AT1/AT,
              v = AT2/AT,
              w = AT3/AT;
        
        // Congrats these are our barycentric coordinates!
        vector b = set(u, v, w);
        append(@barycentric, b);
        
        float au[];
        append(au, u);
        setpointattrib(0,"b",@pts[0],au,"append");
         float av[];
        append(av, v);
        setpointattrib(0,"b",@pts[1],av,"append");
         float aw[];
        append(aw, w);
        setpointattrib(0,"b",@pts[2],aw,"append");
        
        vector gap1[] ,
               gap2[] ,
               gap3[] ;
            
        append(gap1, pColorGap);
        append(gap2, pColorGap);
        append(gap3, pColorGap);
        
        setpointattrib(0,"H_Color",@pts[0],gap1,"append");
        setpointattrib(0,"H_Color",@pts[1],gap2,"append");
        setpointattrib(0,"H_Color",@pts[2],gap3,"append");
        
        vector uvw[];    
        append(uvw, pUVW);
        setpointattrib(0,"H_UVW",@pts[0],uvw,"append");
        setpointattrib(0,"H_UVW",@pts[1],uvw,"append");
        setpointattrib(0,"H_UVW",@pts[2],uvw,"append");

        int LPrim[];
        append(LPrim, prim);
        setpointattrib(0,"prim",@pts[0],LPrim,"append");
        setpointattrib(0,"prim",@pts[1],LPrim,"append");
        setpointattrib(0,"prim",@pts[2],LPrim,"append");
    }
}
```

这里我在低模的每个点中记录了影响当前点的采样点的颜色、采样点UV等。

![QianJianTec1704635736839.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/QianJianTec1704635736839.png)

l(p_{i} )为采样点在低模的值，h(p_{i} )则为采样点在高模的值

这个公式即为低模与高模的最小二乘关系的式子，也就是他们的平方差之和。

最小二乘法有好几种解法，工程上我们常用梯度下降法，也是我用的最多的拟合算法，还有退火算法等。

![QianJianTec1704183240465.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/QianJianTec1704183240465.png)

梯度下降的核心便是这个公式。eta是所谓的学习率，也就是步进值，决定我们每次步进多少。是一个经验值。    \bigtriangledown f(x)是函数f(x)梯度。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

现在我们只需要求得函数梯度，推导可得以下公式： （推导过程链接有）

![QianJianTec1704645620734.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/QianJianTec1704645620734.png)

p_{A}为低模上三角形上的点的，B(i,A)则为当前采样点到p_{A}的重心坐标。

通过上面公式，我们在VEX中去实现，求得当前顶点的梯度。

```javascript
vector colorGapTotal = set(0,0,0);
//foreach n sample points
for(int i = 0; i<len(f[]@b); i++){
    int prim = i[]@prim[i];
    vector uvw = v[]@H_UVW[i];
    //get L(pi)
    vector L_Color = primuv(0,"Cd",prim,uvw);
    vector LC[];    
    append(LC, L_Color);
    setpointattrib(0,"L_Color",@ptnum,LC,"append");
    // get (L(pi) - H(pi))
    vector ColorGap = L_Color - v[]@H_Color[i];
    
    vector Gap[];    
    append(Gap, ColorGap);
    setpointattrib(0,"H_ColorGap",@ptnum,Gap,"append");
		// (L(pi) - H(pi)) * B(i,A)
    colorGapTotal += ColorGap*f[]@b[i];
    //vector D[];    
    //append(D, ColorGap*f[]@b[i]);
    //setpointattrib(0,"D",@ptnum,D,"append");
}

v@preGradient = v@Gradient;
// get current point gradient
v@Gradient = colorGapTotal*2;
```

得到了当前顶点关于最小二乘公式的梯度之后，我们只需要带入梯度下降公式即可。通过一步一步的迭代，我们还可以compile起来多线程来提升迭代速度，使其更快的稳定下来，得到一个最接近高模的结果。

