# Lake 

![Lake.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Lake.png)

支持最小输入两个、最大输入3个分别为：湖泊闭合曲线、地形、水面Mesh
支持3个输出分布为：最终结果预览、地形、水面Mesh
湖泊这里比较简单，首先我们把输入的闭合曲线@P.y=0，然后使用resample细分这个闭合曲线，我们采样曲线上的每一个点对应地形的高度值。

vector p = @P;
p.y = 0.0;
vector ori = point(1, "P", 0);
float h = volumesample(1, "height", p) + ori.y;
@height = h;
p.y = @height;

通过attribpromote直接把最小值转移到detail，这个最小高度就是作为我们湖泊的最小湖面高度。
对于湖泊我们还可以做一步是否裁剪超出地形外的操作，使用地形的boundbox与输入的闭合片做boolean裁剪，得到我们真正需要的水面部分。

对于地面部分：

我们可以使用heightfield_maskbyobject节点把水面投影一个mask在地形上，最简单的就是直接一个blur，然后根据参数的深度对地形做一个@height下降既可以，但是这样的地形特别容易与水面脱离，需要手动对水面做高度调节。第二种就是先把地形拍平@height=0;然后同样把水面投影在地形上，使用ConvertVolume提取地形mask的边缘，使用xyzdist获得距离，通过距离做高度下降mask。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

使用这个拍平过后并高度下降过的地形与原地形做一个height add操作，便可以将下降高度传给原地形。(其实使用原地形下降也可以，这里使用拍平过后的地形主要是可以对地形高度做blur等额外操作)
接下来就可以对地形投影mask范围内做noise、blur等操作(参数暴露可自行选择控制)，使用生成的水面做mask投影做好湖的layer输出既可。

### 对面水面部分：

我们需要把与地形裁剪过的Mesh作为初步的水面，水面的整体高度为我们存在detail里面的最小高度，对于水面我们需要控制他的一些参数。
水面高度：这里我们直接用@P.y += chf(”waterSurfaceOffset”)就可以控制。
水面的宽度我们可以使用polyexpand2d节点通过调节Offset来控制。
之后再通过remesh到我们需要的密度，remesh这里我们可以使用Adaptive功能(旧的可以生成自适应密度的Mesh，新的好像不行了），我们可以均匀remesh之后，通过polyreduce剔除一些重要性较低的顶点。计算向上的法线(判断法线朝向，防止法线向下)，给上UV，打上Tag，就可以pack起来与，输入的水片一起输出啦。同样输入输出口颜色对应改一下养成好习惯。

