# VSCode调试HoudiniPython

调试 Houdini Python 代码官方默认只能使用 print 等原始方法调试，无法设置断点，如 果使用 pdb 将导致整个 
Houdini 软件无响应，这里讲述了如何在 VSCode 中使用 debugpy 库远程调试 Houdini Python 
代码。这里说的"远程"是指VSCode和Houdini软件可以不在同 
一个机器(在一个机器也行，只不过是不同的进程)，当然绝大多数情况我们只是希望能设置 
断点调试Houdini脚本，但是目前只能通过这种"远程"方法实现。

说明: 本文所讲的远程调试只能调试 Houdini "落地"的 Python 脚本，不能调试直接在 Houdini 编辑器里面些的 inline 脚本，也就是脚本必须保存在磁盘上，在Houdini软件里 面 import 的那些脚本文件。

## 1 准备工作

1. 安装好 VSCode 并且安装上 Python 扩展插件。
1. 下载安装一个标准版 Python27 (这是只是为了安装 debugpy 这个 Python 库用的，装完就没用了)
## 2 Houdini 端配置

1. 下载 debugpy

先用 pip 命令下载 debugpy，下面假定你Python27安装在c:盘根目录。

>cd C:\Python27
>python.exe -m pip install debugpy

安装完成后 `c:\Python27\Lib\site-packages\debugpy` 这个目录应该出现了。

关键性的修改

debugpy(本文制作是版本为:
 1.4.1)在判断Python执行程序时用了 `sys.executable` 来判 断，而 Houdini 使用 Python 的方式为 
Embeded 方式，最外层启动程序不是 Python.exe， 为了修复这个问题这里需要 Hack debugpy 的一行代码。

打开 debugpy\server\api.py 修改 "python" 处为你Houdini的Python执行程序:

_config = {
    "qt": "none",
    "subProcess": True,
    # "python": sys.executable,
    "python" : r"c:\Program Files\Side Effects Software\Houdini 18.0.499\python27\python.exe"
}

好了，debugpy 准备完成。

2. 让 Houdini Python 可以使用 debugpy

在
 Houdini Python 库目录下创建一个名叫 local.pth 的文本文件，一般位置是 `c:\Program Files\Side 
Effects Software\Houdini 18.0.499\python27\lib\site-packages\local.pth` 
，文件内容如下:

C:\Python27\Lib\site-packages

完成这一步后在 Houdini 里面应该可以 "import debupy" 了，下面开始配置下 VSCode。

## 3 VSCode 端配置

VSCode 里面创建好 Workspace 后给你的工作空间按下图方式设置远程 Python 调试配置

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

配置文件内容如下:

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

端口号和IP可根据自己情况配置。

## 4 开始调试

Houdini Python 控制台执行如下脚本:

```plain text
import debugpy
debugpy.listen(3000)
debugpy.wait_for_client()

```

执行 `wait_for_client` 后 Python 将卡死等待调试器 Attach，这时候就可以在 VSCode 里面 F5 了。如果VSCode里面设置有断点，断点将起效果:

![Untitled.png](https://raw.githubusercontent.com/RikkaBunny/notion-media/main/images/Untitled.png)

## 5 一些说明

1. `listen` 和 `wait_for_client` 只需要执行一次，一旦和 VSCode 链接上后就不用在 执行了，所以把这两个代码放到HDA或者Script这样的节点里面可能会出问题，因为这些 地方脚本可能会被执行多次。
2. 如果你试图绕过
 local.pth 文件而是直接把 debugpy 目录copy到 Houdini 目录，使用 时会遇到一个 "c:\Program 
Files.." 文件夹中间有空格的一个错，这应该是 debugpy 的 bug，如果官方修正后应该就没问题了。

