# Shader宏分支

在unity包中的CGIncludes\HLSLSupport.cginc中可以找到定义：
#if defined(UNITY_COMPILER_HLSL)    #define UNITY_BRANCH    [branch]    #define UNITY_FLATTEN   [flatten]    #define UNITY_UNROLL    [unroll]    #define UNITY_LOOP      [loop]    #define UNITY_FASTOPT   [fastopt]#else    #define UNITY_BRANCH    #define UNITY_FLATTEN    #define UNITY_UNROLL    #define UNITY_LOOP    #define UNITY_FASTOPT#endif
官方DX有具体文档的：
https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-while
UNITY_FLATTEN 执行分支的所有条件，并且在执行后选择结果。
UNITY_BRANCH 为在进行if条件语句时使用动态分支进行处理， 太多的话有时候会报错。
UNITY_LOOP 表示shader在编译时不进行循环展开检查。
UNITY_UNROLL(x) 表示编译时尝试展开循环，x为尝试展开循环的最大次数。

