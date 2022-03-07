---
layout: post
title: 服务器寻路00-RecastDemo最新编译可以执行断点操作步骤详解
categories: Path
description: RecastDemo最新编译可以执行断点操作步骤详解
keywords: 游戏，寻路, RecastDemo最新编译，断点
---

RecastDemo最新编译可以执行断点操作步骤详解

**目录**

* TOC
{:toc}

## 本次我们用到的编译idea（两个):

```sh
1.VSCode2017 (其它版本也可以)
2.Rider for Unreal Engine 2021.3.1  (注意不是c#的Rider, 而是给c++使用的 for Unreal Engine, 别装错了)  
```

## 编译步骤

### 1.下载准备

```sh
1.从github下载RecastNavigation项目：https://github.com/recastnavigation/recastnavigation
2.下载SDL2的源码：https://www.libsdl.org/download-2.0.php
3.下载Premake工具：https://github.com/recastnavigation/recastnavigation
```

### 2.编译

![](/images/posts/findpath/7.png)

```sh
1.SDL2源码解压后的文件夹放到RecastNavigation/RecastDemo/Contrib/目录下
（目录名如果是SDL-2.xx需要改名为SDL）
```

```sh
2.进入RecastDemo/Contrib/SDL/VisualC，用visual studio打开SDL.sln，
    执行编译生成解决方案（debug+win64模式编译）：
    recastnavigation-master\RecastDemo\SDL\VisualC\Win64\Debug该目录下就会生成需要的三个文件：
    SDL2.dll、SDL2.lib、SDL2main.lib
```

![](/images/posts/findpath/7.png)

```sh
3.编译成功后将RecastDemo/Contrib/SDL/VisualC/x64/Debug下的SDL2.dll、SDL2.lib、SDL2main.lib
复制到RecastDemo/Contrib/SDL/lib/x64：
    注意：lib/x64本身文件夹不存在，我们需要新建文件夹，
    然后SDL2.dll、SDL2.lib、SDL2main.lib都复制到该文件夹下
```

```sh
4.将premake5.exe拷贝到工程目录RecastNavigation/RecastDemo/下，cmd命令行进入此目录：
执行：
  .\premake5 vs2017（根据自己装的vs版本）
```

```sh
5.进入/RecastDemo/Build/vs2017，用vs2017 打开recastnavigation.sln
  执行：
      a.Debug-----Win64模式下
      b.点击项目-重定解决方案目标
```

```sh
8.进入/RecastDemo/Build/vs2017，用.Rider for Unreal Engine 打开recastnavigation.sln
  打开后，选中RecastDemo，右键Debug运行：
```
![](/images/posts/findpath/8.png)

![](/images/posts/findpath/9.png)

```sh
9.执行断点调试：
```
![](/images/posts/findpath/10.png)