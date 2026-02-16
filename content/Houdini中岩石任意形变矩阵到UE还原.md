# Houdini中岩石任意形变矩阵到UE还原

在houdini中我们可以通过extracttransform节点来得到两个模型的最佳形变拟合矩阵。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

当模型的变换符合矩阵变换的规则，https://chartreuse-room-39e.notion.site/e47e1c2cf20b43c0b1ff38fa42a0db35那么模型的拟合误差则为0，为完全重合。通过extracttransform节点我们就可以得到两模型的变换矩阵。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

当我们在houdini得到一个任意的形变矩阵，我们都可以将其在拆分为最简单的几个矩阵的乘积， 分别为 位移矩阵、旋转矩阵、缩放矩阵、剪切矩阵。通过矩阵之间合理的拆分与组合，我们可以在任何地方(DCC 或者 引擎中)进行形变的拆解与组装。https://chartreuse-room-39e.notion.site/dd8c976117b243ab879e304d015bf4ac

在houdini中 ，我们便可以把这一个模型形变矩阵 拆解为 RHST 的矩阵组合。

```plain text
//得到 FullMatrix 4x4的形变矩阵
matrix FMatrix = 4@transform;
// 将4x4的形变矩阵分解为 CT 矩阵 C为复合矩阵 T为位移矩阵
4@T = ident();
4@T.wx = FMatrix.wx;
4@T.wy = FMatrix.wy;
4@T.wz = FMatrix.wz;
FMatrix.wx = 0;
FMatrix.wy = 0;
FMatrix.wz = 0;
//得到C矩阵的行列式和C转置矩阵，行列式不能为0，
matrix CMatrix = FMatrix;
4@C = CMatrix;
float i = determinant(4@C);
@i = i;
matrix CTMatrix = transpose(CMatrix);

//根据推导公式将C矩阵分解为 R旋转矩阵 与 D三角矩阵
matrix C_Matrix = CTMatrix * CMatrix;
4@C_M = C_Matrix;
float dxx = sqrt(C_Matrix.xx);
float dxy = C_Matrix.xy / float(dxx);
float dxz = C_Matrix.xz / float(dxx);
float dyy = sqrt(C_Matrix.yy - dxy*dxy);
float dyz = (C_Matrix.yz - dxy*dxz) / float(dyy);
float dzz = sqrt(C_Matrix.zz - dxz*dxz - dyz*dyz);

matrix DMatrix = set(set(dxx,dxy,dxz),set(0,dyy,dyz),set(0,0,dzz));
4@D = DMatrix;
matrix DTMatrix = invert(DMatrix);

matrix RMatrix = CMatrix * DTMatrix ;

4@R = RMatrix;

//在将D三角矩阵分解为 S缩放矩阵 与 H剪切矩阵
matrix HMatrix = set(set(1,dxy/float(dxx),dxz/float(dxx)),set(0,1,dyz/float(dyy)),set(0,0,1));
4@H = HMatrix;
matrix SMatrix = set(set(dxx,0,0),set(0,dyy,0),set(0,0,dzz));
4@S = SMatrix;

```

上面为 Houdini中使用Vex分解 4x4矩阵的代码，我们可以通过上面这段代码将 一个 4x4的形变矩阵分解为 RHST 矩阵组合(因为这里是T为行矩阵所以RHST，如果T为列矩阵则为TRHS)。

我们可以知道各个矩阵的数据占比，T位移矩阵 为 3x1float，R旋转矩阵为3x3float，H剪切矩阵为3x1float，S缩放矩阵为3x1float。

这里我们可以将 H 矩阵拆出来，将RST预先存入模型中，也就是模型的Local To World矩阵中。

我们知道矩阵的乘法是不可逆的 RHS  != RSH，所以我们应该怎么办勒。

通过上面分析各个矩阵的数据占比，我们可以将 R矩阵直接给与模型变换，我们将得到一个模型只有Rotation有值的变化，position将为（0，0，0），scale将为（1，1，1）。

如果我们要在UE中还原模型的形变，让一个模型可以变换为多种形态和样子。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

那么 恰好 T矩阵为 3个float，S矩阵也为 3个float，刚刚可以存在 Location 和 Scale里面，之后我们在去UE里面将 Location和Scale里面的 S和T抽出来 作为矩阵参与运算既可。

对于单个静态模型的形变来说，H剪切矩阵的三个数据我们只需要手动输入即可。而我们重点讨论的是对于多个模型instance的H剪切矩阵来说，理论上我们只需要将H剪切矩阵的三个数据加入到Instance Buffer中，在每个instance顶点的VertexShader阶段取得当前instance的H剪切矩阵 外加 当前Instance的Location数据为T矩阵，Scale数据为S矩阵，便可以在顶点阶段 应用 HST 三个矩阵乘积 就能变换到形变之后的结果啦 (R矩阵我们已经提前应用到顶点上了)。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

对于多个instance模型形变支持，我们可以将每个H矩阵的三个数据存储在 CSV文件中，导入UE当做 DataTable类型数据使用。通过蓝图的SetCustomDataValue函数，我们就可以将DataTable中的每个数据都设置进当前 ISM 或者 HISM 组件的Custom data中。（这一步如果Houdini Engine插件为19以上可以直接在houdini设置Instance的Custom data数据）

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

当我们把数据设置进Instance Buffer的Custom data里面之后，我们在材质里面就可以取到啦。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

通过指定对应的 DataIndex索引，也就是 0 1 2 这三个索引，我们就可以取到当前Instance Buffer对应Instance N号索引上的数据啦。

所以我们在材质里面可以完全得到H矩阵啦，那么我们只需要得到：每个顶点位置(R旋转矩阵应用过的顶点数据)、每个instance的轴点位置(T矩阵数据)、每个instance的缩放大小(S矩阵数据)

对应Instance的每个顶点LocalPosition，我们使用LocalPosition节点是拿不到的(这问题耗了我一下午还是得看代码才能解决)，这时候我们可以打开当前材质对应的HLSL代码看看，我们可以在

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

VertexParameters参数这里找到，当我们使用的Instance的话，顶点工厂传过来的参数里面就有我们需要的 InstanceLocalPosition 和 InstanceLocalToWorld。

使用Custom节点，直接通过代码，我们便可以取到我们想要的变量。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

但是，这里的InstanceLocalPosition得到的顶点并不直接是我们要的R旋转应用之后的顶点位置，因为我们将S缩放矩阵存储在Scale上，所以我们还得 除于 Scale缩放值，我们才能拿到正确的值。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

每个instance的缩放大小(S矩阵数据) ，我们直接用 (1，0，0)、（0，1，0）、（0，0，1）去应用Instance的InstanceLocalToWorld，在计算一下长度便可以得到，各个轴的缩放数据。

对于每个instance的轴点位置(T矩阵数据)，我们只需要用ObjectPosition节点便可以取得每个Instance的轴点。

这样我们需要的数据就都齐啦，只需要使用Custom节点将得到的数据输入进去计算便可以还原变换啦。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这里计算了 HS = D 矩阵的结果，最后的T矩阵运算就是直接 加上 ObjectPosition便可以啦，

这里 transform position节点中 ，Pos为 顶点应用 R矩阵之后的结果.

```plain text
Pos.xyz = Pos.xzy;

float3 P = Pos;

P.x = Pos.x*m_xx + Pos.y*0+ Pos.z*0;
P.y = Pos.x*m_xy + Pos.y*m_yy+ Pos.z*0;
P.z = Pos.x*m_xz + Pos.y*m_yz + Pos.z*m_zz;
P.xyz = P.xzy;

return P;

```

因为houdini和UE的 坐标系不同 运算同一在左手坐标系，最后变换到UE的坐标系中。

我们可以将这个顶点变换连入 World Position Offset 口中，注意 因为我们的WPO是在当前顶点基础上加上变换值，而我们计算得到值为最终顶点位置值，所以我们需要在加上一个 WorldPosition * -1，让顶点位置归零。这样我们就可以真正得到模型形变之后的结果啦。

当我们的模型形变完之后，我们会发现光照并不正确，那是因为我们只形变了模型顶点，没有将模型的法线也做相应的形变。

这里就直接给出结论了吧，当我们对模型顶点应用了形变矩阵M之后，顶点法线就需要应用形变矩阵M的逆转置 或者 伴随转置

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

根据 伴随转置矩阵计算器，便可以计算得出M的伴随转置矩阵

一步步解的计算器 - Symbolab

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

```plain text
Pos.xyz = Pos.xzy;

float3 P = Pos;
float I = -(i*z);
float J = (i*k - j*y);
float K = -k*x;

P.x = Pos.x*(y*z) + Pos.y*I+ Pos.z*J;
P.y = Pos.x*0 + Pos.y*(x*z)+ Pos.z*K;
P.z = Pos.x*0  + Pos.y*0  + Pos.z*(x*y);
P.xyz = P.xzy;
P=normalize(P);
return P;

```

这只是顶点法线，顶点法线只需应用形变矩阵M的伴随转置矩阵。

对于顶点切线于顶点副切线，我们仍然应用形变矩阵M，执行与顶点相同的变换即可。需要注意运算需要在Vertex阶段，所以后面我们需要加上 VertexInterpolator节点，用于VS插值到PS计算法线贴图的变换。

拿到变换后的顶点 法线 、切线、 副切线。我们便可以构建法线的 TBN To World 来变换法线贴图。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这样 模型的顶点形变于法线形变都应用成功啦。

