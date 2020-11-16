---
title: 在Unix下安装开发OCaml
layout: post
subtitle: 使用Visual Studio Code开发
date:       2020-11-16
author:     "Zhao"
header-img: "img/post-bg-unix-linux.jpg"
tags: 
   - Installation
---

此文章将介绍在类Unix系统下配置VSCode以开发OCaml。

# 物料

macOS或Linux发行版或WSL以及Visual Studio Code代码编辑软件。

# 命令行安装OCaml

1. 使用`apt install ocaml`(macOS请将`apt`或`apt-get`替换为`brew`，其他Liunx发行版使用自带的包管理器)安装OCaml
2. `apt-get install opam`安装opam包管理器。像pip一样，它被用来安装许多OCaml的依赖
3. `opam init`进行初始化
4. 如果你正在使用WSL，它可能不支持沙盒模式（在WSL上未经测试），那么你可以运行`opam init --disable-sandboxing` 来禁用沙盒模式。
5. `eval $(opam env)`升级现有shell环境。
6. 使用`mkdir` 和`cd` 创建并进入你的OCaml项目

# 安装依赖

我们将安装OCaml包下的两个依赖用来支持VS Code的运行

1.  使用`opam install merlin`安装merlin，用来支持VS Code对OCaml的自动补全功能
2. 使用`opam install ocp-indent`安装ocp-indent来支持VS Code下对OCaml的自动规范格式功能

# 设置VS Code

打开VS Code，在左侧插件选单中搜索"OCaml and Reason IDE"并安装，并重启VS Code。

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/Screenshot%202020-11-14%20at%2022.51.21.png)

在左下角找到Settings，搜索框中键入"Reason > Path"搜索对刚刚安装的插件的设置项，找到OCamlmerlin和Ocpindent的设置项，路径中输入这两个依赖安装所在文件夹的**绝对路径**（可以使用`which`来搜索opam包的安装位置，通常为~/.opam/default/bin/文件夹下，注意将~替换为绝对路径） 。重启VS code

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/Screenshot%202020-11-14%20at%2023.02.51.png)

# 编写.merlin文件

在项目根目录下，新建一个文件，命名为`.merlin`，这份文件帮助merlin来正确识别项目中的文件夹的属性，在文件中输入：

```
S .
B _build
```

此处S代表单词Source，表示源代码所在的文件夹，你也可以在项目根目录中新建src文件夹，将此处的 `.`替换为`src`；B代表编译产物所在的文件夹。 
