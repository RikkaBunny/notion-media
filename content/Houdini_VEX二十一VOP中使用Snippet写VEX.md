# Houdini VEX(二十一)VOP中使用Snippet写VEX

![9148742-69b15c23ef2cf20f.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-69b15c23ef2cf20f.webp)

![9148742-8a9d26c46479cc56.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-8a9d26c46479cc56.webp)

![9148742-6cbea0e9e8522bf5.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-6cbea0e9e8522bf5.webp)

![9148742-d9d866d9ed3e9514.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-d9d866d9ed3e9514.webp)

![9148742-a14c35e73d1baaac.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-a14c35e73d1baaac.webp)

![9148742-6be5d40ce3731974.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-6be5d40ce3731974.webp)

![9148742-fbdbc6f129c5fc58.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-fbdbc6f129c5fc58.webp)

一、VOP中使用Snippet写VEX

1. VEX本身就是snippet：点进wrangle里面，就是snippet
1. 自己尝试用snippet写成和wrangle里一样的效果
- 先创建一个attribvop节点
- 进入attribvop节点，创建一个snippet节点
- 删除剩下的两个节点
- 写上代码，并将Bindings to Export写上星号（这样带@的属性就是可读可写的，不打星号是只读的）
1. 在vop中额外的使用snippet做些其他的事情
- vop和wrangle有他们各自的好处
- vop中用noise很方便
创建一个turbnoise节点

![webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/webp)

创建一个displace along normal节点，并连线

![webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/webp)

我们希望某个方向上强，然后逐渐朝某个方向上衰减，我们用点乘Dot Product节点，再用fit range节点映射范围，修改属性值，并连线

![webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/webp)

- 再在以上基础上，加入snippet
![webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/webp)

如果要在snippet里获取noise属性，需要将noise节点连接到snippet节点
如果想要P可写，需要在Bindings to Export上写上P

![webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/webp)

再加个法线

![webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/webp)

- 比较效果
打开snippet节点

![webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/webp)

打开vop节点

![webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/webp)

这两个效果是一样的，都是置换效果

- 给snippet添加和vop一样的dot效果
给snippet添加个用户控制，中键点击snippet上的next属性，点击Promote Parameter
显示
调整属性值
继续添加点乘，映射
再添加一个可以用户控制的值
修改代码，让用户控制的maxAngle从0-180映射到1到-1
