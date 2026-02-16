# 导出Unreal自定义属性到Houdini

以尽量不修改引擎，只修改插件的原则来完成自定义数据导出的流程。

我们需要导出4.26Water插件的河流宽度、深度、速度大小等自定义数据怎么做，那么首先我们需要把我们工程变为C++工程，如以是C++工程不用管，注意插件我们需要放在工程的Plugins里面，而不是使用引擎的Plugins。

使用sln打开项目，F5运行，这里可以打开项目文件大致看一眼Water插件的.h文件名称，大概定位一下我们需要的东西在哪个class。

我们使用EditorUtilityWidgetBlueprint来把我们需要的属性导出到JOSN 或者 CSV等格式里面，houdni端在通过解析对应文件得到相应属性。

我们可以在蓝图里面创建一个WaterBodyRiver的对象来一步一步的获取它的属性，我们知道我们要的深度这些属性是依附在Spline上，所以我们可以搜索这个对象关于Spline的一些属性

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

可以看到有两个GetSpline，其实两个返回结果都一样，只不过一个是对象，一个方法。我们可以双击进入代码实现看见。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

然后再在蓝图中使用get等关键词查找或是查看相关变量都没有，我们在进入UE里面直接看看它的头文件有没有相关变量，我们发现这里并没有我们想要的数据，里面只有一个FWaterSplineCurveDefaults的结构体变量，通过F12查找，我们发现这个变量定义在一个WaterSplineMetaData的类，在往下翻发现河流的深度与宽度等属性都定义在这里。这就是我们所需要的数据拉。想知道这个类在哪里创建使用的，我们有两种常用的一种就直接Ctrl+F全局搜索，另一种就是断点其中一个比较Function并且触发它，通过调用堆栈一步步的索引找到。最终我们发现

WaterSplineMetaData创建是在WaterBodyActor里面创建，子类River继承了它，所以没有在WaterBodyRiverActor.h里面发现这个对象。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

我们可以发现这里UWaterSplineMetadata并没有暴露给蓝图，所以我们可以像上面的SplineComponent在UPROPERTY里加上BlueprintReadOnly这个flag，我们需要读取就好。重新启动，我们便可以在蓝图里面得到WaterSplineMetadata，通过WaterSplineMetada便可以得到它的RiverWidth等属性（如果没有则去代码里面加上BlueprintReadOnly这个flag）。可以发现我们得到的Depth等信息是FInterpCurveFloat类型，这里我们直接封装一个解析函数

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这样我们就可以得到河流的自定义属性，通过蓝图遍历我们选择的物体就可以得到我们需要的数据，这里我们需要保存我们需要的数据到CSV中，我们可以自己创建一个public的蓝图函数库类，自己封装一个保存Function。

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

这样数据就保存好了。

到houdini端就简单多了，直接使用tableimport节点既可解析出对应的数据。

