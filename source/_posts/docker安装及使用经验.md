---
title: docker安装及使用经验
date: 2022-06-03 08:31:14
tags:
---

# docker

[TOC]





前提是在vscode上面使用docker命令时发现命令无法识别，在vscode中下载扩展docker后也依旧不行，后推断原因是本机并没有下载过docker,因此下面探究如何在**windows系统下**下载并使用docker。整个思路：首先下载docker desktop并配置好，之后在vscode中下载扩展，最后在程序中使用docker命令进行操作。

**本机以前装过虚拟机，在很多教程中都需要下载hype，但是有解释说hyper下载后其他的虚拟机不能使用，所以本教程是在未下载hyper的情况下使用docker**

##### 关键词：docker+wsl 2， docker+vscode，远程连接已有的docker镜像并上传内容 #####

##  安装docker ##

1. 搜索docker官网，下载适合于windows版本的docker desktop，下载成功后可以通过命令行cmd输入

`docker --version`

![image-20220604205534083](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604205534083.png)

查看是否下载docker成功。

2. 运行docker run hello-world时发现报错，打开docker desktop 同样发现报错，提示未下载WL2，网上教程和windows官方教程开始下载 WL2.



##  下载WL2 ##

1. 以管理员身份运行powershell  ==  （win+r 输入powershell ）  +  （打开后输入*start-process PowerShell -verb runas*即可进入管理员 模式的powershell，输入下面的代码：

`*wsl --install*`

若运行之后返回帮助文档，即下列情况：

![image-20220604210057562](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604210057562.png)那么这个时候需要按顺序输入：

*`wsl --list --online`*→查看可用发行版本

运行*`wsl --install -d <DistroName>`*→我选的ubuntu，把-d后面的替换掉即可==*wsl --install -d ubuntu*

之后会要求设置账号和密码，设置linux用户名和密码，在下载时系统会提醒设置username和password，此用户名和密码特定于安装的每个单独的linux分发版，与windows用户名无关；能够运行sudo（super user do）管理命令。

```
账号：amang

密码：


```

2. 输入命令

`wsl --list --verbose`

 查看安装在 Windows 计算机上的 Linux 发行版列表，其中包括状态（发行版是正在运行还是已停止）。

ubuntu默认的wsl版本是1，此时不要慌张，上述步骤是wsl官方给的安装目录下的第一个安装教程，接下来跳到第二个安装教程：**[旧版本的手动安装步骤](https://docs.microsoft.com/zh-cn/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package)**。

**启用适用于linux的windows子系统**

（1） 启用适用于 Linux 的 Windows 子系统：

以管理员身份运行powershell

`dism.exe /online /enable-feature /featurename:Microsoft-Windodiws-Subsystem-Linux /all /norestart`

结果会出现：

![image-20220603084232285](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220603084232285.png)

-（2）检查自己电脑是否满足wl2的运行要求

以本机为例：X64，内部版本号19043，次要版本号1706，均高于所需的。查看自己电脑的版本。win+r输入winver，可以查看到

![image-20220603083827521](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220603083827521.png)

-（3）启用虚拟机功能

安装 WSL 2 之前，必须启用“虚拟机平台”可选功能。 计算机需要*虚拟化功能*才能使用此功能。

在powershell下面运行：

`dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart~~`

-（4）**下载linux内核更新包**← 非常重要！！！！非常重要

[下载适用于x64计算的WSL2 linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

下载完后打开docker desktop会发现不报错了，不会再提醒WSL2没下载成功！！！

查看wsl，会发现

![image-20220604223751498](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604223751498.png)

此时代表docker  desktop可以打开了！

##  配置docker ##

首先知道报错的类型：

1. ![image-20220604212919357](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604212919357.png)

 代表docker没有打开，这个时候可以看 docker desktop打开后是否表明正在运行

![image-20220604213046801](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604213046801.png)

↑ 这里是绿色，表示正在运行！！  如果是别的颜色，表示没有运行。

上述报错可以从以下考虑：

-（1）wsl2没配置好--可以从docker  desktop上提示看到

...



2. ​      https和http问题

docker没有配置信任路径：![image-20220604215001126](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604215001126.png)

上述是原始代码，需要加入

 "insecure-registries": [
    "信任路径"]

![image-20220604220636148](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604220636148.png)

然后登录，可以发现ok了。



##  在vscode中使用docker ##

1. 下载扩展

   ![image-20220604221121829](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604221121829.png)



![image-20220604221141831](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604221141831.png)



之后就可以在vscode中使用docker了。

如何在连接docker的问题：通过登录docker

docker login dorker镜像网址

docker pull 路径:latest

docker build -t 路径latest  .  ← 有一个点，代表一层

docker push hub.路径:latest

2.出错类型及解决：

（1）build时报错![image-20220604222145276](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604222145276.png)

push时报错![image-20220604222259184](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604222259184.png)

这个时候虽然镜像容器会create，但是实则不会push成功。

使用时需要查看有没有![image-20220604221851726](D:/hexo/blog/source/_posts/image-20220604221851726-16543523994561.png)这个文件，同时根据需要修改![image-20220604222515634](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604222515634.png)后面的路径

（2）当（1）中的问题解决以后，会出现![image-20220604222658069](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604222658069.png)

X509错误，这个时候build之前最好pull一下之前的文件，之后build，push，会发现成功了。

 



总结：

（1）docker最好用在虚拟环境下，但通过本教程可以在windows下面使用docker.

（2）doker下载完之后需要配置WSL2，如果没有的话需要自行配置，不管是wsl 1 还是wsl 2 ，可以在控制面板→程序→启用或关闭windows功能，查看

![image-20220604223117272](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604223117272.png)

这个是否勾选。

（3）下载过程中发现了一个很好用的软件：windows teminals。

- 安装

[Windows 终端安装 | Microsoft Docs](https://docs.microsoft.com/zh-CN/windows/terminal/install)

Windows Terminal 可以使用命令行接口运行任何应用程序。 它的主要功能包括多个选项卡、窗格、Unicode 和 UTF-8 字符支持、GPU 加速文本呈现引擎，你还可用它来创建你自己的主题并自定义文本、颜色、背景和快捷方式。

每当安装新的 WSL Linux 发行版时，都会在 Windows Terminal 中为其创建一个新实例，该实例可根据你的偏好进行自定义。

（4）开始通过适用于 Linux 的 Windows 子系统使用 Visual Studio Code

- a 在vscode中下载扩展 Remote - WSL。

- b 在vscode中打开WSL项目。

在刚才下载的windows终端输入

 `code .`

![image-20220603094337756](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220603094337756.png)



-  c 从vscode中下载

还可以通过使用 VS Code 中的快捷方式 `CTRL+SHIFT+P` 调出命令面板，以访问更多 VS Code 远程选项。 如果随后键入 `Remote-WSL`，将看到可用的 VS Code 远程选项列表，使你可以在远程会话中重新打开文件夹，指定要在哪个发行版中打开，等等。

- （5）用第一种安装方式安装WSL时选择ubuntu ，通过windows teminals可以发现建立了一个ubuntu虚拟机，

  ![image-20220604223939423](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604223939423.png)

而且对应的wsl是 1，可以在windows powershell 管理员模式下输入wsl --set-version <distro name> 2，<distro name> 就是更新的linux发行版的名称，比如我最开始使用的是ubuntu，这里直接替换为`wsl --set-version ubuntu 2`，通过命令`wsl --list --verbose`可以看到版本变了：![image-20220604224415870](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604224415870.png)

这里有个思考，其实docker desktop用的其实是手动安装的wsl安装包下载的wsl2.

- （6）一些wsl 2代码

   `wsl --set-version`==`wsl -l -v` →查看linux发行版的wsl版本

  `wsl --install`

  `wsl --list --online`

  `wsl --install -d <DistroName>`

  - （7）关于图片加载不成功的问题
  
    绝对路径和相对路径
  
    ![image-20220604231613693](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604231613693.png)

偏好设置中勾选优先使用相对路径，这个上传到github上才能看到 。

绝对：前面有磁盘等要素

相对![image-20220604231816548](docker%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C/image-20220604231816548.png)

   2022.06.04完成

