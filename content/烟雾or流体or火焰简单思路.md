# 烟雾or流体or火焰简单思路

https://twitter.com/SoerbGames/status/1270766023139119104

把物体移动移动轨迹记录下来，做历史帧叠加然后衰减，像做雪和草一样，然后对历史叠加帧没帧做一个偏移(向上或者向下 ，模仿重力下落或者往上飘的感觉)，对历史叠加帧做noise扭曲使其看起来更像火焰或者流体，最后blur一下就好了

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

