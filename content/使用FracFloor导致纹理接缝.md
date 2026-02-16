# 使用Frac/Floor导致纹理接缝

起因：

使用世界坐标做贴图UV时，使用了Frac函数使世界空间UV转换到0到1平铺。发现出现uv 0-1跳转，出现接缝。

![v2-a0c8c94a7039ffb9f48ae2c151cc22c2_720w.jpg](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/v2-a0c8c94a7039ffb9f48ae2c151cc22c2_720w.jpg)

原因：

通常，接缝会由于两种原因造成，一方面是由于没有Padding，采样时纹理会做线性插值，导致图集边缘处像素模糊，而计算UV时没注意就会采样到邻界像素。解决办法一是需要处理Padding，二是计算UV时需要进行收缩处理。

另一种像锯齿一样的接缝是由于Frac导致。如图做了简单测试Shader，为了规避Padding导致的接缝，纹理的FillterModel为Point。

Frac算法计算Tilling时，在边界处会采样另一侧像素，GPU计算ddx ddy的时候会当成非常大的跨度计算，得到非常大的mip值，很多时候就是最模糊的那层，结果边界像素就出现“硬边”问题。

解决方法：

解决方法有两种，一种是通过计算像素到相机的距离得到想要的miplevel（不推荐），

另一种是通过传入原贴图的uv，让硬件算原贴图ddx ddy。

uv为frac后0-1的平铺uv，uv1为frac前连续uv

return Tex.SampleGrad(TexSampler,UV,ddx(UV1),ddy(UV1));

