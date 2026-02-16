# Houdini VEX(十一)自制Neighbours函数

一、neighbours函数：houdini自带的有，是用来查找邻居顶点的，如：我门输入的5
，则会返回一个数组，装着1、4、6

![9148742-d39cb21a1d052495.webp](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/9148742-d39cb21a1d052495.webp)

二、自制neighbours函数

- 写法1：
int prims[] = pointprims(0,@ptnum);//以5号点为例，5 1 0号面都包含了5号点

int pts[];
int nb[];
int npt;
int ptprev = -1;
int ptnext = -1;

foreach(int prim; prims)           //遍历5 1 0面
{    
    pts = primpoints(0,prim);      //5 1 0面中以5号面为例，有5 6 7 4号点，这个数组是按连接顺序排的的
    npt = len(pts);                //5号面的总点数量 是4
    int myIndex = find(pts,@ptnum);//查找5号点自身在这个数组中哪个位置
    
    if( myIndex == 0 )             //在首位，比如5 6 7 4
    {        
        ptprev = pts[npt-1];       //前一个是 该数组最后一个 4
        ptnext = pts[1];           //后一个是 自己的下一个   6
    }
    else if( myIndex == npt-1 )    //在尾部，比如6 7 4 5
    {
        ptprev = pts[myIndex-1];   //前一个是 自己的前一个  4
        ptnext = pts[0];           //后一个是 该数组第一个  6
    }    
    else                           //在中间，比如6 5 4 7       
    {
        ptprev = pts[myIndex-1];   //前一个是 自己的前一个 6
        ptnext = pts[myIndex+1];   //后一个是 自己的下一个 4
    }
    if( find(nb, ptprev) < 0 )     //5号面得出邻居46,1号面得出16,0得出14，有重复
        append(nb,ptprev);         //比如算完5号面已知邻居46，存放在数组nb
    if( find(nb, ptnext) < 0 )     //下一步1号面得出邻居1号点，往nb数组附加
        append(nb,ptnext);         //如果找不到点1在nb里面的位置(即nb里面不存在点1)，find()返回负值
}//则附加上去，再下一步1号面得出邻居6号点，发现6在nb里面的位置是1(即存在点6)，find()返回非负值，则不附加
i[]@nb = nb;

- len(参数1)函数：查看总点数
- find(参数1，参数2)函数：从参数一中找参数二，如果找不到会返回负值
- append(参数1，参数2)函数：将参数2添加到参数1中
- 写法2：
int primnum = -1;
int vtx     = -1;
int nvtx    = 0;
int ptprev = -1;
int ptnext = -1;
int nb[];

foreach(int lvtxnum; lvtx)//对于3个线性序号中的每一个线性序号
{
    primnum = vertexprim(0,lvtxnum);     //求面序号
    vtx     = vertexprimindex(0,lvtxnum);//求顶点序号
    nvtx    = primvertexcount(0,primnum);//求这个面的顶点总数
    //特殊情况vtx=0(第一个顶点) 或vtx=nvtx-1 (最后一个顶点)
    if( vtx == 0 )                            //primpoint()通过面序号和顶点序号 来获取点序号
    {                                         //顶点序号为0
        ptprev = primpoint(0,primnum,nvtx-1); // 一个prim可能有n个顶点,前一个为n
        ptnext = primpoint(0,primnum,1);      // 后一个1
    }
    else if( vtx == nvtx-1 )                  //顶点为n
    {
        ptprev = primpoint(0,primnum,vtx-1);  // 前一个n-1
        ptnext = primpoint(0,primnum,0);      // 后一个0
    }
    else                                      //顶点为任何中间顶点i
    {
        ptprev = primpoint(0,primnum,vtx-1); // 前一个i-1
        ptnext = primpoint(0,primnum,vtx+1); // 后一个i+1
    }
    if( find(nb,ptprev) < 0 )
        append(nb,ptprev);
    if( find(nb,ptnext) < 0 )
        append(nb,ptnext); 
}
i[]@nb = nb;

- 将写法1写成方法(函数)：
//function 返回值类型 方法名(形参类型 形参名...)
function int[] nb(int ptnum)
{    
    int prims[] = pointprims(0,ptnum);
    
    int pts[];
    int nb[];
    int npt;
    int ptprev = -1;
    int ptnext = -1;
    
    foreach(int prim; prims)
    {    
        pts = primpoints(0,prim);    
        npt = len(pts);
        int myIndex = find(pts,ptnum);
        
        if( myIndex == 0 )
        {        
            ptprev = pts[npt-1];
            ptnext = pts[myIndex+1];
        }
        else if( myIndex == npt-1 )
        {
            ptprev = pts[myIndex-1];
            ptnext = pts[0];
        }    
        else
        {
            ptprev = pts[myIndex-1];
            ptnext = pts[myIndex+1];
        }
        if( find(nb, ptprev) < 0 )
            append(nb,ptprev);
        if( find(nb, ptnext) < 0 )
            append(nb,ptnext);
    }
    return nb;//对应方法的返回值类型
}
//调用上面写的nb方法
i[]@nb = nb(@ptnum);

- 写法2写成方法：
//function 返回值类型 方法名(形参类型 形参名...)
function int[] nb(int ptnum)
{
    int lvtx[] = pointvertices(0,ptnum);
                                       
    int primnum = -1;
    int vtx     = -1;
    int nvtx    = 0;
    int ptprev = -1;
    int ptnext = -1;
    int nb[];
    
    foreach(int lvtxnum; lvtx)
    {
        primnum = vertexprim(0,lvtxnum);    
        vtx     = vertexprimindex(0,lvtxnum);
        nvtx    = primvertexcount(0,primnum);
        if( vtx == 0 )                            
        {                                        
            ptprev = primpoint(0,primnum,nvtx-1); 
            ptnext = primpoint(0,primnum,1);     
        }
        else if( vtx == nvtx-1 )               
        {
            ptprev = primpoint(0,primnum,vtx-1);  
            ptnext = primpoint(0,primnum,0);    
        }
        else                                      
        {
            ptprev = primpoint(0,primnum,vtx-1);
            ptnext = primpoint(0,primnum,vtx+1);
        }
        if( find(nb,ptprev) < 0 )
            append(nb,ptprev);
        if( find(nb,ptnext) < 0 )
            append(nb,ptnext); 
    }
    return nb;//对应方法的返回值类型
}

//调用上面写的nb方法
i[]@nb = nb(@ptnum);

