# Houdini VEX(十二)Intrinsic属性

一、Intrinsic属性和普通属性基本相似，区别在于普通属性存储在几何体上，Intrinsic属性仅在需要的时候计算出来
二、只有prim和detail有intrinsic属性

![9148742-797da39ae5b0ce6a.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-797da39ae5b0ce6a.webp)

三、solidembed节点：将模型变成四边形网格

![9148742-0bb5bce3b9cfc797.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-0bb5bce3b9cfc797.webp)

![9148742-41b08a8fb85b5e93.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-41b08a8fb85b5e93.webp)

四、读取的方法一：prim函数

- 代码：
//读取0号输入端的prim序号测量出来的体积
if(prim(0,'intrinsic:measuredvolume',@primnum) > chf('threshold'))//threshold是定义的一个阈值，可用通过调整来控制显示的面
    removeprim(0,@primnum,1);//移除这个面，1代表并删除顶点
/*   
measuredarea 测量出来的面积
measuredperimeter 测量出来的周长
measuredvolume 测量出来的体积
*/

五、读取的方法二：primintrinsic函数（最常用）

- 代码：
if(primintrinsic(0,'measuredarea',@primnum)>chf('threshold'))
    removeprim(0,@primnum,1);
/*   
measuredarea 测量出来的面积
measuredperimeter 测量出来的周长
measuredvolume 测量出来的体积
*/

六、读取的方法三：在组里写，满足这个组的才会alpha变成0

![9148742-e07276eb26b55cea.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-e07276eb26b55cea.webp)

- ch('threshold')是用esc键下面的波浪线那个按键括起来的
七、写的方法一：setprimintrinsic函数

- 代码：
matrix3 trans = 4; //意味着{4,0,0, 0,4,0, 0,0,4}
setprimintrinsic(0,'transform',0,trans);//该属性负责旋转和缩放

八、写的方法二：先pack节点打包，然后会多出来一些intrinsic属性，再通过setprimintrinsic函数控制这些intrinsic属性

![9148742-04a59845d1743319.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-04a59845d1743319.webp)

- 这些intrinsic属性都可以控制：
- 代码：
setprimintrinsic(0,'viewportlod',0,'box');//pack geo在视口的显示模式
setprimintrinsic(0,'pivot',0, set(0,chf('height'),0) );//轴心点

九、写的方法三：通过isooffset节点先转化成体积，然后通过setprimintrinsic函数控制体积独有的intrinsic属性

![9148742-5331993cfbfee537.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-5331993cfbfee537.webp)

- 读写同理，都可以找到这些intrinsic属性
- 不同种的prim有不同的intrinsic属性
十、写的方法四：先convertvdb节点转化，再跟上面同理
- 代码：
setprimintrinsic(0,'vdb_class',0,'sdf volume');//vdb体积类型

十一、还有很多不同的intrinsic属性可以写，需要去探索
十二、Spreadsheet中，灰色的Intrinsic属性是不可更改的

