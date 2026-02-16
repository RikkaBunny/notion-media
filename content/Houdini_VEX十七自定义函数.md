# Houdini VEX(十七)自定义函数

一、自定义函数的目的：把我们常用的一些代码打包起来

- 示例：
/*
function 关键词 可加可不加
float 是该函数的返回类型,如果返回类型是void,代码中不需要return
mydot 是函数名字
a,b 是函数参数(由用户提供) 
vector 是参数类型，相同类型可用逗号隔开;
                 不同类型必须用分号，比如(vector a;float b)
const意味着只读 不可写
*/
function float mydot(const vector a,b)
{
    float c = a.x*b.x + a.y*b.y + a.z*b.z;
    return c;//必须有return关键词,返回浮点c
    //如果返回类型是void,不需要return
}
//函数重载，相同函数名下，可以有不同的参数类型和不同的返回类型
function float mydot(const vector2 a,b)
{
    return a.x*b.x + a.y*b.y;//return一个式子，式子计算结果是浮点
}
//@Cd = mydot({0,1,0},@N);
//@Cd = dot({0,1,0},@N);

vector2 pos = set(@P);//set函数将三维变量转为二维
@Cd = mydot({1,0},pos);

//可以有多个return关键词
function int mymax( const int a,b)
{
    //a*=2; //这样做会报错,因为const int b只能读不能写
    if(a>=b)
        return a;
    else
        return b;
}
int c = 10;
printf('%s\n',mymax(c,2));
printf('%s\n',c);

//参数always passed by reference，总是引用，不是拷贝
//意味着如果在函数中修改了参数变量，这个变量就改变了
//为了避免意外地改变参数，可以加上const关键词
function void acrossb(vector a;const vector b)
{
    //自定义函数里面可以使用vex自带函数
    //a = set(a.y*b.z-a.z*b.y, a.z*b.x-a.x*b.z, a.x*b.y-a.y*b.x);
    //自定义函数里面还可以再次自定义函数[nested function]
    function vector myset(const float a,b,c)
    {
        vector result = 0;
        result.x = a;
        result.y = b;
        result.z = c;
        return result;
    }
    a = myset(a.y*b.z-a.z*b.y, a.z*b.x-a.x*b.z, a.x*b.y-a.y*b.x);
}
@N = @P;
//@N = cross({0,1,0},@N); //vex自带cross不修改参数，而是返回向量
//acrossb({0,1,0},@N); //不能改变纯数值
acrossb(@N, {0,1,0}); //改变了@N

