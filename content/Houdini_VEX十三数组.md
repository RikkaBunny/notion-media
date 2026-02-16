# Houdini VEX(十三)数组

一、创建数组

![9148742-6e2e1cc87aa3e358.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-6e2e1cc87aa3e358.webp)

- 可以用纯数字创建数组
vector color[] = { {1,0,0}, {0,1,0}, {0,0,1}, {1,1,0} };
@Cd = color[@primnum];

- 使用变量或表达式创建时，要用array()，没有数量限制
float alpha[] = array(chf('a0'),chf('a1'),chf('a2'),chf('a3'), 0.1, color[1][1] );
@Alpha = alpha[@primnum];

二、练习

- 向量数组展平成浮点数组
vector pos[]; //创建空的向量数组
float pos_float[]; //空的浮点数组

//下列代码将向量数组展平成浮点数组
for(int i=0; i<@numpt; ++i)       //有多少点，循环多少次（4个点）
{
    pos[i] = point(0,'P',i);      //将位置数据写入数组元素
    pos_float[i*3+0] = pos[i].x;  // [id] 读取元素值 .x 读取向量的第一个分量(xyz)
    pos_float[i*3+1] = pos[i].g;  // .g 读取向量的第二个分量(rgb)
    pos_float[i*3+2] = pos[i][2]; // [2] 读取向量的第三个分量(012)
    /*
    for(int j=0; j<3; ++j)            //每个点再循环3次
        pos_float[i*3+j] = pos[i][j]; //每次都把向量的每个分量填入数组
    */                                //pos[i][j] ,获取pos数组中的第i个元素，
                                      //这个元素是向量，[j]获取这个向量的第j个分量
}

v[]@pos = pos;  //创建属性，类型为向量数组，然后将变量pos的值赋予给属性
f[]@pos_float = pos_float; //浮点数组属性

- 读写元素值
float not_exist = f[]@pos_float[12]; //读取数组属性要写清楚类型，如v[],f[]...
printf('%s\n',not_exist); // 不存在12号元素，返回0；如果是字符串数组则返回""
printf('%s\n', len(f[]@pos_float) ); // 目前还是12个元素
 
f[]@pos_float[13] = 110;    //写入原本不存在的13号元素,数组会变大,没指定的元素会是0
printf('%s\n', len(f[]@pos_float) ); //现在数组中有14个元素
printf('%s\n',f[]@pos_float[12]);    //打印12号元素的值
printf('%s\n',f[]@pos_float[13]);    //打印13号元素的值

1）取数组属性要写清楚类型，如v[],f[]...
2）不存在的元素，返回0；如果是字符串数组则返回""，会自动增加数组长度

三、Slicing：分割、提取、排序数组

- 用法：value[start:end:step] 开始 结束 步幅
- 代码：
int nums[] = { 0, 1, 2, 3, 4, 5 };
//            -6 -5 -4 -3 -2 -1

//value[start:end:step] 开始 结束 步幅
int start[] = nums[0:2];  // { 0, 1 } 0号开始，2号之前结束(取不到2号)
int end[] = nums[-2:];  // { 4, 5 }  倒数第2个开始，没有结束(意味着直到尽头)

int rev[] = nums[::-1];  // { 5, 4, 3, 2, 1, 0 } -反着数 1每次加一
int rev2[] = nums[-1:-4:-1];  // { 5, 4, 3 }  倒数第1个开始反着数，
                                           //倒数第4个之前结束(取不到倒数第4个)
int odd[] = nums[1::2]; // { 1, 3, 5 } 从1号开始，步幅为2，隔一个取一个，直到结尾
int odd2[] = nums[1:len(nums):2]; // { 1, 3, 5 } 同上，len()数组元素数量
//可以打印出值，方便观察
printf('start %s\n',start);
printf('end %s\n',end);
printf('rev %s\n',rev);
printf('rev2 %s\n',rev2);
printf('odd %s\n',odd);
printf('odd2 %s\n',odd2);

四、数组与其他数据类型互相拷贝

//数组与其他数据类型互相拷贝
vector pos = {1,2,3};
vector cd = 0;
float x[];

//float x[] = pos; *Wrong*
x = set(pos); // x={1,2,3} 向量赋予给数组
cd = set(x);  // cd={1,2,3} 数组拷贝给向量

x = {1,9};    //对数组重新赋值
cd = set(x);  // cd={1,9,0} 数组拷贝给向量,不够给,后面为0

printf('%s\n',x);
printf('%s\n',cd);

// 详见Arrays -> Copying between arrays and vectors/matrices

