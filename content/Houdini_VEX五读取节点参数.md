# Houdini VEX(五)读取节点参数

一、节点参数的定义：能够在属性面板中看到的参数

![9148742-fc3e30bed348ce2a.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-fc3e30bed348ce2a.webp)

二、既叫做节点参数，也叫做通道
三、用ch*()读取各种类型的参数(通道)

1. ch也就是channel(通道)的缩写
1. 写完vex代码，点击右侧的按钮，在下侧就会生成刚刚写的通道：
1. 测测前面写的：打印代码的值，看看是不是和通道中的值相同
四、chramp()的用法以及注意事项

1. 以@P.x的值作为输入，经过下面的映射处理，再输出给@P.y的值
- ramp需要自变量的范围在0-1之间，超出范围会循环
1. fit函数可以把一个范围映射到另一个范围：这里我们把@P.x的值映射到0到1之间，最小值用min通道表示，最大值用max通道表示：
1. chf函数的其他用法：
- 可以用来找绝对路径的参数值
- 可以用来找相对路径的参数值（../表示当前节点的上一级）
五、可以通过鼠标拖动通道到场景视图中，切换到选择工具或者移动工具可以改变滑条的值：

![9148742-e878cc0b992f9ecd.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-e878cc0b992f9ecd.webp)

- 点这里调节手柄大小：
- 如果滑块没效果，点击这里设置手柄的参数：
- 如果不想看到滑条，可以在移动工具处右键隐藏：
- chramp函数默认返回的是浮点类型，用vector函数框起来后，返回的就是矢量类型：
- 把chramp也限定到0-1之间：
