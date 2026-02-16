# Houdini VEX(十五)数组相关函数

一、数组相关函数

int num[10] = {5,2,3,3,4,8,1}; //用户设置的初始长度会被忽略
printf('%s\n',num); 
printf('=================================\n');

resize(num,10);    //设置数组的长度，变长，新值为0或''
printf('%s\n',num);//注意这类function不返回数据，而直接修改所传入的数组变量
printf('=================================\n');

resize(num,6);     //设置数组的长度，变短，当然会删去已有元素
printf('%s\n',num);
printf('=================================\n');

string word[] = {'$','HIP','/'}; //创建字符串数组
printf('%s\n',word         );
printf('%s\n',len(word)    ); //len(array) 返回数组的长度(元素数量)
printf('%s\n',len(word[1]) ); //len(string) 返回字符的数量
printf('=================================\n');

string last = pop(word); //pop() 将数组最后一个元素抽去并返回，数组长度减一
int numm[] = {5,2,3,3,4,8,1};
printf('%s\n',last ); 
printf('%s\n',word ); 
pop(word,0);             //依据id将指定元素抽离
printf('%s\n',word );
pop(numm,-2);             //删去倒数第二个
printf('%s\n',numm ); 
//removeindex(&array[], index)有相同效果 
int ture = removevalue(numm,3); //删除数组中发现的第一个数值为3的元素，
printf('%s\n',ture ); //删除成功返回1，否则返回0
printf('%s\n',numm );
printf('=================================\n');

append(word,'/geo');     //在数组的末尾添加一个元素，相同功能的还有push()
word[5] = '/agents';
append(word, {'/zombie','/clip'} ); //在数组的末尾拼接另一个数组(literal)
string test[] = {'/walk'}; 
append(word,test);                  //在数组的末尾拼接另一个数组(variable)
printf('%s\n',word ); 
printf('=================================\n');

int pts[] = array(1, 2.5, 'k', {5,4,3} ); //用 各种数据和变量 创建一个数组
printf('%s\n',pts );                      //都会被转成指定类型，这里是整数
printf('=================================\n');

vector  pos[] = { {1,2,3}, {4,5,6} };
matrix3 rot[] = { {{1,0,0}, {0,1,0}, {0,0,1}} , {{0,0,2}, {2,0,0}, {0,2,0}} };
float posfloat[] = serialize(pos); //将向量数组 展平 成浮点数组
float rotfloat[] = serialize(rot); //将矩阵数组 展平 ，每个分量都是一个浮点元素
printf('%s\n',posfloat ); 
printf('%s\n',rotfloat );
printf('=================================\n');

vector vel[];
matrix3 scale[];
float flt[] = { 1, 1, 0, 1, 2, 0, 3, 1, 3 };
vel = unserialize(flt);  //将浮点数组 组合 成向量数组，3个浮点一个向量
scale = unserialize(flt);//将浮点数组 组合 成矩阵数组
printf('%s\n',vel );
printf('%s\n',scale );
printf('=================================\n');

int number[] = { 5, 2, 3, 1, 4, 6, 7, 9, 0 };
printf('%s\n',min(number) );
printf('%s\n',max(number) );
printf('%s\n',avg(number) );
printf('%s\n',sort(number) );//返回新的排好序的数组，排序方式从小到大，原数组不变
printf('%s\n',argsort(number) );//返回数组，用这个数组的每一个值去给原数组排序，就是从小到大的
int sorted[];
for(int i=0; i<len(number); ++i)
{
    int temp[] = argsort(number);
    sorted[i] = number[temp[i]];
}
printf('%s\n',sorted );
float re[] = reorder(flt,argsort(flt));
printf('%s\n',re ); 
reverse(sorted);//返回新的数组，反转顺序，原数组不变，还可以用来反转字符串"hello" -> "olleh"
printf('%s\n',reverse(sorted) );
printf('=================================\n');

int digit[] = { 5, 2, 3, 1, 4, 6, 7, 9, 0 };
insert(digit,1,23);  //在第n 号 元素前面插入元素
insert(digit,-1,43); //在倒数第n 个 元素前面插入元素 
printf('%s\n',digit );
int a[] = {8,8,8};
insert(digit,3,a);   //插入数组变量
printf('%s\n',digit );
printf('=================================\n');

int data[] = { 5, 2, 3, 1, 4, 6, 7, 9, 0 };
printf('%s\n', isvalidindex(data,-10) );
//该数组序号范围0~8或 -1 ~ -9(倒数第一个~倒数第九个)
//如果序号在这数组中有，isvalidindex()返回真，否则返回假
//等价于index < len(array) && index >= -len(array)
//          这个例子中 id<9 && id>=-9  
printf('=================================\n');

//find() 回看自制neighbours函数视频

