# Houdini VEX(十四)ForEach语句

一、简单的foreach

![9148742-9db0043e03aeccbe.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-9db0043e03aeccbe.webp)

- 代码：
//对于体积，并非只能用Volume Wrangle
//正如Attribute Wrangle其名，有属性就能用它;有point prim vertex就能用
//   aw可以读取和修改属性，vw可以读取和修改体积(体素)

float density[] = primintrinsic(0,'voxeldata',0);//获取intrinsic属性，
                                 //voxeldata:体素值的浮点数组
                                 
float maxValue = 0; //该变量用来存放最大值

foreach(float value; density)       //遍历density数组中的每一个浮点value
    maxValue = max(maxValue,value); // 上次的值 与 这次的值 对比，取最大的

setprimattrib(0,'name',0,sprintf('%s',maxValue)); 
//转成字符串赋予给name属性(也就是体积的名字)
//中键按住即可方便地观察体素的最大值

- max函数：两个参数值进行比较，取最大值返回
二、for each 带序号形式

- 代码：
//下列代码求 体积中 体素值在rangeMin~rangeMax范围内的体素 的数量占总体素量的百分比
float density[] = primintrinsic(0,'voxeldata',0);

float maxValue = 0;  //存放最大值
float minValue = 0;  //存放最小值
float rangeMax = chf('range_max');  //最大范围
float rangeMin = chf('range_min');  //最小范围
int sampleRate = int( rint( 1/chf('sample_rate') ) );
//采样率(跳过体素个数):用户输入0~1; rint()取最近整数返回浮点,如2.5=3.0,2.49=2.0，-0.6=-1; int()取整
//例如 用户输入   0 0.5 0.3 0.01  1    恰好0和1意味着完全采样
//sampleRate =  0  2   3  100   0
int count = 0;  //计数器

foreach(int id; float value; density) //遍历density数组中的每一个浮点value，
                                      //每循环一次，id+1
{                             
    if( id % sampleRate == 0 )  // 如sampleRate = 5，则每隔5个体素运行下列代码一次
    {
        maxValue = max(maxValue,value);
        minValue = min(minValue,value);
        if( value < rangeMax && value > rangeMin )//如果体素值符合范围
            ++count;                              //计数器+1
    }
}
printf('max = %s\n',maxValue);
printf('min = %s\n',minValue);
printf('percent = %s\n',float(count*sampleRate)/len(density));
// 百分比 = 计数结果*跳过体素个数 / 体素总数量

作者：Joe_Game

链接：https://www.jianshu.com/p/75b41122f1d2

来源：简书

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

