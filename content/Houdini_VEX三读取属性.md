# Houdini VEX(三)读取属性

一、使用@符号

1. 例子：
float y = @P.y;//x,y,z
等价于
float y = @P.g;//r,g,b
等价于
float y = @P[1];//0,1,2

二、使用@opinput?_

1. 例子：
float r = @opinput1_Cd.y;//获取第二个输入端 相同序号的点 的Cd属性，常用属性不用标明属性类型

float f = v@opinput1_foo.y;//Houdini不认识的属性必须标明属性类型

三、使用函数

1. 例子：
vector color = point(1,"Cd",0);
color = prim(1,"Cd",0);
color = vertex(1,"Cd",0);
color = detail(1,"Cd");
//具体用法参考帮助文档VEX Functions

1. 查看一个节点存在哪些属性，可以通过attribute vop节点查看：这些就是存在的全局属性，可以通过@+红色框选的这些全局属性获取这些本身存在的属性
四、特别的：体积，不同于位置点、顶点、面的读取属性

1. 读取方式：@+体积名称，来读取体素值
1）例子：
1. 测试：
1）新建一个box节点，选中按i进入，连上isooffset节点，修改属性：name改为density，uniform sampling divs改为5：
3）连上volum wrangle节点进行vex代码

![9148742-dffaa9d4841dfe96.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-dffaa9d4841dfe96.webp)

五、注意事项

![9148742-472630fdae761279.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-472630fdae761279.webp)

