# Houdini VEX(十六)类型转换

一、类型转换

- 转换变量类型
方法通常是
变量类型( 要转化的变量 )
如 float(100) -> 100.0
- 示例：
@P.x = int(@P.x);//将x坐标从浮点转为整数 ，
 //重新赋予给@P.x后又会 自动 转成浮点
 
print_once( sprintf('%s\n',int(@P.x)) );

- int(@P.x)部分是手动转换，将float类型用int函数强制转换为int类型
- P.x = int(@P.x);部分是先手动转换再自动转化，将int类型自动转化为float类型
- print_once函数：只打印一次
- sprintf函数：将打印的内容转为字符串
- 特别注意除法
- 示例：
//@Cd = @ptnum/(@numpt-1); //整数除以整数，结果仍是整数
@Cd = float(@ptnum)/(@numpt-1);
//所以需要至少一个是浮点

二、类型指定

- 指定函数返回类型
比如要求某函数返回浮点(前提是该函数能返回浮点)
方法通常是
变量类型( 函数() )
如 float( rand(@ptnum) )
- 实例：
//@P += noise(@P)*chf('amp'); 
//这里'+='后面的代码求出的是浮点，在加法运算时自动被转成向量了

@P += vector(noise(@P))*chf('amp');

//noise() 可以返回浮点也可以返回向量，
//vector(noise())明确让它返回向量

//@Cd = length(point(1,'P',@ptnum));
@Cd = 0.2*length(vector(point(1,'P',@ptnum)));

//length() 返回浮点，但参数可以有多种类型，如vector2 vector vector4
//point() 返回点属性，点属性有多种类型
//length()不知道point()会返回什么类型，所以报错 Ambiguous call

