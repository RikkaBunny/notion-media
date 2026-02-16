# Houdini VEX(二十)For循环补充

一、For循环补充

for(float num: {2,4,6,8} )//初始化列表 可以是浮点数组 向量数组等
    @P.y += num;
int num[] = {2,4,6,8};
for(int j: num)
    @P.y -= j;
foreach(int number; num)
    @P.y += number;
foreach(int id; int number; num)//id是0 1 2 3
    @P.y -= (id+1)*2; 
   
string names[] = {'piece0', 'piece1', 'piece2'};
for(string name: names)
    printf('%s\n', name);
foreach(string name; names) 
    printf('%s\n', name); 
    
vector cd[] = {{1,0,0}, {0,1,0}, {0,0,1}, {0,1,1}, {1,1,0}, {1,0,1}};
int idx = 0;     //红绿蓝青黄紫
for(vector color: cd)
{//只有当点序号等于所循环的次数时，该点才会获取对应次数的颜色
    @Cd = @ptnum == idx ? color:@Cd;
    ++idx;
}

- for可以用初始化列表，foreach不行
for(float num: {2,4,6,8} )       √
foreach(int number; {2,4,6,8})  ❌

- for中用的：，foreach中用的；
