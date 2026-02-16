# speedtree   ST格式数据

## SpeedTree的设置和导出

1.SpeedTree导入UE需要使用.st格式,可以包含风场,材质等数据

![925ed6c184534c04deaabd7c37bcf709-2834](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/925ed6c184534c04deaabd7c37bcf709-2834)

![234c5382b04a7aaf8fbea63a7643f573-25586](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/234c5382b04a7aaf8fbea63a7643f573-25586)

2.SpeedTreeWind 有一些预设的风的质量

![0142730c2fd637e523f894483d375159-4042](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/0142730c2fd637e523f894483d375159-4042)

![891c526be23cfe9fd5c8e52bab13d1d0-7665](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/891c526be23cfe9fd5c8e52bab13d1d0-7665)

## 数据的分析

下文所有UV[Index]都是以max中为准,从1开始

![529ee28c9adf61b213d951dfa451f79a-26103](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/529ee28c9adf61b213d951dfa451f79a-26103)

### Fastest

uv3:(0,7.492)

uv4:localPos.xy

uv5:(localPos.z,1)

uv6:(0,0)

uv7:(0,0)

uv8:(0,0)

### Fast

uv3:(0,7.492)

uv4:localPos.xy

uv5:(localPos.z,每个叶片有一个值,主干和插片枝干是1,约靠上的叶片这个值越大)

uv6: 主干和插片的枝干为(0,0)

每一片叶子有一个值,x≠y,y值偏大 y像是当前叶片中心点的高度

uv7:主干和插片的枝干为(0,0)

x:叶片的渐变,y轴是每个叶片向y的正方向偏移一个值,貌似与高度无关

uv8:主干和插片的枝干为(0,0)

(x是一个渐变值,从根部到顶部,渐变的过度与叶片大小无关,y都是0)

### Better

uv3:(0,7.492)

uv4:localPos.xy

uv5:(localPos.z,每个叶片有一个值,主干和插片枝干是1,约靠上的叶片这个值越大)

uv6: 主干和插片的枝干为(0,0)

每一片叶子有一个值,x≠y,y值偏大 y像是当前叶片中心点的高度

uv7:主干和插片的枝干为(0,0)

x:叶片的渐变,y轴每个叶片向y的正方向偏移一个值,貌似与高度无关

uv8:主干和插片的枝干为(0,0)

(x是一个渐变值,从根部到顶部,渐变的过度与叶片大小无关,y都是0)

### Best

uv3:(0,7.492)

uv4:localPos.xy

uv5:(localPos.z,每个叶片有一个值,主干和插片枝干是1,约靠上的叶片这个值越大)

uv6: 主干和插片的枝干为(0,0)

每一片叶子有一个值,x≠y,y值偏大 y像是当前叶片中心点的高度

y轴可能减去uv3.y 与x组成一个与位置相关的数据

uv7:主干和插片的枝干为(0,0)

x:叶片的渐变,y轴每个叶片向y的正方向偏移一个值,貌似与高度无关

uv8:主干和插片的枝干为(0,0)

(x是一个渐变值,从根部到顶部,渐变的过度与叶片大小无关,y都是0)

### Palm

uv3:(0,7.492)

uv4:localPos.xy

uv5:(localPos.z,每个叶片有一个值,主干和插片枝干是1,约靠上的叶片这个值越大)

uv6:树干是(0,0)

插片的树枝是,x方向是感觉是个随机值没啥规律,y反向是(-1或1)

叶片的渐变,每个叶片向y的正方向偏移一个值,貌似与高度无关

uv7:主干和插片的枝干为(0,0)

插片的树枝,x轴感觉是描述左右,y轴为0

x:叶片的渐变,y轴是每个叶片向y的正方向偏移一个值,貌似与高度无关

uv8:主干和插片的枝干为(0,0)

(x是一个渐变值,从根部到顶部,渐变的过度与叶片大小无关,y都是0)

## UE中代码

SpeedTree风场代码在MaterialTemplate.ush中调用,具体的方法在SpeedTreeCommon.ush中定义

float3 GetSpeedTreeVertexOffsetInner(FMaterialVertexParameters Parameters, int 
GeometryType, int WindType, int LODType, float BillboardThreshold, bool 
bExtraBend, float3 ExtraBend, FSpeedTreeData STData)

### SpeedTreeData结构体定义

从C++传的参数,与风场相关

struct FSpeedTreeData
{
       float4 WindVector;
       float4 WindGlobal;
       float4 WindBranch;
       float4 WindBranchTwitch; //晃动
       float4 WindBranchWhip;//抽打
       float4 WindBranchAnchor;//锚点
       float4 WindBranchAdherences;//依附
       float4 WindTurbulences;//湍流
       float4 WindLeaf1Ripple;//波纹
       float4 WindLeaf1Tumble;//翻滚
       float4 WindLeaf1Twitch;//晃动
       float4 WindLeaf2Ripple;
       float4 WindLeaf2Tumble;
       float4 WindLeaf2Twitch;
       float4 WindFrondRipple;//蕨类叶波纹
       float4 WindRollingBranch;//滚动树枝
       float4 WindRollingLeafAndDirection;
       float4 WindRollingNoise;
       float4 WindAnimation;
};

GetPreviousSpeedTreeData()
GetCurrentSpeedTreeData()

### RippleFrond

### LeafWind

---

## UV数据

- uv1-采样贴图的uv
- uv2-LightMap的uv
### SmoothLod

- (uv4.xy,uv5.x)-是lod的Pos,用来实现Lod切换时顶点的偏移
![f3e502fc14522608d4c632ee4ca6b758-15675](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/f3e502fc14522608d4c632ee4ca6b758-15675)

### Branch

- uv3-VertBranchWind
- 树干和大树枝都是(0,7.492) - 解码出的向量为(0,0,0.75)
- 在SpeedTree根据树枝类型区分?
- uv3.x Weight-树枝分簇的渐变幅度
- uv3.y Offset 根据该值解码出树枝偏移的方向
float3 UnpackNormalFromFloat(float fValue)
{
       float3 vDecodeKey = float3(16.0, 1.0, 0.0625);

       // decode into [0,1] range
       float3 vDecodedValue = frac(fValue / vDecodeKey);

       // move back into [-1,1] range & normalize
       return (vDecodedValue * 2.0 - 1.0);
}

![54438f039165f34215399d05ddafa9b1-12190](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/54438f039165f34215399d05ddafa9b1-12190)

### Frond && Palm

- uv6.x fPackedRippleDir
- uv6.y fRippleScale
- uv7.x fLenghtPercent
![f578ca05ecc18ee1fb60bcf4e15e68c3-20578](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/f578ca05ecc18ee1fb60bcf4e15e68c3-20578)

### Leaves

- (uv5.y,uv6.x,uv6.y)-leafAnchor,叶子的轴点
![634bd54a106072dc9d5a9f4076e4fb21-18649](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/634bd54a106072dc9d5a9f4076e4fb21-18649)

- uv7.x Scale-幅度 就是叶片的渐变,描述叶片运动的幅度
- uv7.y PackedGrowthDir 生长方向 一个控制树叶翻滚的旋转轴
float3 LeafTumble(FSpeedTreeData STData,
       float3 vPos, 
       inout float3 vDirection, 
       float fScale, 
       float3 vAnchor, 
       float3 vGrowthDir, 
       float fTrigOffset, 
       float fTime, 
       float fFlip, 
       float fTwist, 
       float fAdherence, 
       float3 vTwitch, 
       float4 vRoll,
       int WindType,
       bool bTwitch, 
       bool bRoll)
{
       // compute all oscillations up front
       float3 vFracs = frac((vAnchor + fTrigOffset) * 30.3);
       float fOffset = vFracs.x + vFracs.y + vFracs.z;
       float4 vOscillations = TrigApproximate(float4(fTime + fOffset, fTime * 0.75 
- fOffset, fTime * 0.01 + fOffset, fTime * 1.0 + fOffset));
       
       // move to the origin and get the growth direction
       float3 vOriginPos = vPos.xyz - vAnchor;
       float fLength = length(vOriginPos);

       // twist
       float fOsc = vOscillations.x + vOscillations.y * vOscillations.y;
       float3x3 matTumble = RotationMatrix(vGrowthDir, fScale * fTwist * fOsc);

       if (WindType >= SPEEDTREE_WIND_TYPE_BEST_PLUS)
       {
              // with wind
              float3 vAxis = wind_cross(vGrowthDir, STData.WindVector.xyz);
              float fDot = clamp(dot(STData.WindVector.xyz, vGrowthDir), -1.0, 
1.0);
#ifdef SPEEDTREE_Z_UP
              vAxis.z += fDot;
#else
              vAxis.y += fDot;
#endif
              vAxis = normalize(vAxis);

              float fAngle = acos(fDot);

              float fAdherenceScale = 1.0;
              //if (bRoll)
              //     fAdherenceScale = Roll(fAdherenceScale, vRoll.x, vRoll.y, 
vRoll.z, vRoll.w, vAnchor.xyz, fTime/* + fOffset*/);

              fOsc = vOscillations.y - vOscillations.x * vOscillations.x;

              float fTwitch = 0.0;
              if (bTwitch)
                     fTwitch = Twitch(vAnchor.xyz, vTwitch.x, vTwitch.y, vTwitch.z 
+ fOffset);


              matTumble = mul_float3x3_float3x3(matTumble, RotationMatrix(vAxis, 
fScale * (fAngle * fAdherence * fAdherenceScale + fOsc * fFlip + fTwitch)));
              //     matTumble = mul_float3x3_float3x3(matTumble, 
RotationMatrix(vAxis, fAngle));
       }
 
       vDirection = mul_float3x3_float3(matTumble, vDirection);
       vOriginPos = mul_float3x3_float3(matTumble, vOriginPos);

       vOriginPos = normalize(vOriginPos) * fLength;

       return (vOriginPos + vAnchor);
}

- uv8.x PackedRippleDir 编码的波浪方向 根据该值解码出树叶偏移的方向
float3 LeafRipple(float3 vPos, 
                             inout float3 vDirection, 
                             float fScale, 
                             float fPackedRippleDir, 
                             float fTime, 
                             float fAmount, 
                             bool bDirectional,
                             float fTrigOffset)
{
       // compute how much to move
       float4 vInput = float4(fTime + fTrigOffset, 0.0, 0.0, 0.0);
       float fMoveAmount = fAmount * TrigApproximate(vInput).x;
       if (bDirectional)
       {
              vPos.xyz += vDirection.xyz * fMoveAmount * fScale;
       }
       else
       {
              float3 vRippleDir = UnpackNormalFromFloat(fPackedRippleDir);
              vPos.xyz += vRippleDir * fMoveAmount * fScale;
       }
       return vPos;
}

- uv8.y 标识是否是leaf2,多层树叶的时候的数据
![32f23699a90351ecf15ad009e5e59aa6-24266](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/32f23699a90351ecf15ad009e5e59aa6-24266)

// leaf wind
if (WindType > SPEEDTREE_WIND_TYPE_FASTEST && WindType != 
    SPEEDTREE_WIND_TYPE_PALM) 
{
  float4 WindExtra = float4(Parameters.TexCoords[6].x, 
                            Parameters.TexCoords[6].y, Parameters.TexCoords[7].x, Parameters.TexCoords[7].y);
  float LeafWindTrigOffset = Anchor.x + Anchor.y;
  FinalPosition = LeafWind(STData, WindExtra.w > 0.0, 
                           FinalPosition, Normal, WindExtra.x, float3(0,0,0), WindExtra.y, WindExtra.z, 
                           LeafWindTrigOffset, WindType);
}

### GlobalWind

- 树整体的风场
if (bExtraBend || bHasGlobal)
       {
              FinalPosition = GlobalWind(STData, FinalPosition, TreePos, true, bHasGlobal, bExtraBend, ExtraBend);
       }

### Houdini工具(UV从0开始)

1.AO数据处理:顶点色R通道的数据传到uv1的G通道

2.叶子轴点数据:(uv4.y,uv5.x,uv5.y)传递到(uv2.r,uv2.g,uv3.r)   拥有相同轴点数据的叶子作为一簇  树干的轴点数据都是(0,0,0)

3.叶子渐变梯度:计算相同轴点位置的顶点到轴点的距离/最大距离 存在uv3.g 树干的叶子渐变值是0

4.树干渐变梯度:计算顶点到树轴点的距离/最大距离,低于轴点的顶点直接给0  存在uv4.r  所有顶点统一计算,树叶也算

5.根据轴点位置计算一个(0,1)随机数 存在uv4.g

