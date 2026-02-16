# 黑客帝国-Layout 

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

输入共4个 ：

城市的底座 、城市的主路 、城市的高度 、 道路阻挡块

输出2个：

一个可视化 、 一个综合数据Pack集

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

进去第一部分就是用线框出一个城市的底座，支持三种不同类型输入

接下来switch 有两种路线一种是用divide细分的方式，旋转细分。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

另一种是先获得一个简化后的形状，这一步先细分，提取边缘片删除，只保留内部规整的矩形，转为边缘线段，提取线段长度小于细分大小+10的线段，再遍历每一块内部线段，看看每一块由多少线段组成，如果是偶数便删除最后一个prim，奇数则不用，最后再将中间的点输出只保留头尾。将断的部分给连起来，可以选择用凸包来代替形状，这样就得到一个简易的形状(过程一点不简易好伐:)。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

最后在以这个形状每个点为中点 一个点切几刀，最后切出这个形状。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

接下来就通过这个形状来找主干道，先找到输入主干道线，与城市底座圈相交处最近的点，这两个点代表主干道的起点与终点，使用这两个点在切过的城市底座上寻找最近路径。这个路径便是主干道。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

然后使用主干道将一整块的底座使用主干道切分为多块prim。

接下来就是清理一下这些块数据 ，因为这些块是用Boolean切分的可能不感觉，并且提取大块小块，判断条件为面积大于50000，然后循环每一块找它们的最长边并算出最长边旋转到坐标水平轴的夹角。等下好按照这个夹角去旋转这个面，让最长边平行与Z轴，每隔一段切一刀，在让其平行与Z轴继续隔一段切一刀。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

切的时候就分配了 B道路 ，逻辑就是先把这个输入面对应切方向的轴 缩放为0，就变成一列点，这些点就是切B级道路的，然后这些点剔除大于一定范围的点，剩下点将fuse，这些就是保留的B级道路点，C级道路点便是根据B级的距离生成的。(这段需要自己细看内部实现，我只说大概)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

将ABC三种道路分别附上对应属性 名字 id 宽度，在根据宽度做相交剔除，离比自己高级的道路太近的道路线段将会被剔除，

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

先细分为点判断距离，在由点转到线段prim上判断是否剔除。

然后清理边缘重叠与计算道路线段与输入4道路阻挡块的相交，如果当前道路段包含在阻挡块中就剔除。

然后删除太小的路，还有删除离主干道太近的路，然后继续剔除在阻挡块中的路段，再清理线段

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

重新再根据路段name附上属性，接下来将距离比较近的路口融合起来，设置属性和再次继续剔除在阻挡块中的路段，再清理和修复线段属性，这时候基本就能拿到一个最终的道路曲线，有了道路曲线我们的输出1和输出2就基于这道路曲线。

输出1：将道路曲线按照自身的宽度属性与位置，生成一个新的prim道路片，再将道路片polyextrude挤出，就能得到基础的道路样子，再翻转顶点与重新计算法线 与 我们切分道路时产生的底座合并输出便可以。

输出2：输出2是由多个输出组合起来分别是 lot底座布局、sidewalk人行道、底座形状、road_network路网、city_metadata城市的元数据。

road_network路网数据不用说就是我们最后拿到的道路曲线给上一个Group便可以啦。

底座形状便是通过我们的road_networdk路网做一个凸包，取这个包围盒的正面既可。

lot底座可以理解为街道块挤出与底座做一个boolean操作得到。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

但是这个底座这边有一个要注意的地方，就是把边缘的点给标记出来啦，逻辑就是生成两个道路片 A 、B，A按照正常道路宽度生成，B按照正常(道路宽度-1)生成，然后A算 xyzdist(B,@P,prim,uvw);，如果重叠的地方就是0，如果边缘的地方就不重叠 距离为1。下面就可以根据距离将边缘点提取出来，每个边缘点按照N方向生成一条线(N就是街道的方向)，再取他们的相交点就是fuse的位置。就能缝合这种边缘。boolean得到的底座进行一个数据清理与法线计算便可以了。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

sidewalk人行道：人行道我们可以用polypath把lot底座的边缘提取出来，遍历road_network每一条道路，将道路属性转移到对应附近的人行道到，有了道路属性的人行道线段就可以输出啦。

city_metadata城市的元数据就是我们切分道路时产生的底座附上各种城市的元数据(高度呀等等)就可以啦。

