# Houdini VEX(十)点、线、面互访

一、点、线、面互访

![9148742-3ec8965bdcc75c2b.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-3ec8965bdcc75c2b.webp)

- 线性顶点序号：针对的是整个几何体的顶点序号
1）点击spreadsheet面板中的，勾选map offset（线性顶点序号）
- 顶点序号：针对的是任一面的顶点序号
- setvertexattrib函数：设置顶点位置属性
1）一般情况，第三个参数是写面序号，此时第四个参数写顶点序号，但当第三个参数为-1时，表示第四个参数写线性顶点序号
