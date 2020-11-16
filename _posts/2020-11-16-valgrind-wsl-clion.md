---
title: WSL使用CLion配合Valgrind
layout: post
subtitle: WSL与CLion
date:       2020-11-16
author:     "Zhao"
header-img: "img/post-bg-unix-linux.jpg"
tags: 
   - Installation
---

对于C/C++开发者，可供选择的IDE有很多，例如Eclipse，VS Code，Visual C/C++。今天介绍一款JetBrains旗下的收费软件：CLion。

# 取得授权

**请确保您在证书允许范围内使用正版软件。**

对于学生用户，可使用学校etu邮箱申请免费证书，有效期可覆盖至学生邮箱失效。

您也可以试用30天后再购买证书。

# 简单使用

安装完成后，您可以选择New Project从头开始项目；在已有代码的情况下，可以选择New CMake Project from sources，将Eclipse项目变为CMake项目。

# 与Valgrind结合使用

下面介绍在Windows平台使用CLion，同时结合WSL下的Valgrind开发准备：

1. 打开wsl终端，确保已安装CMake：`sudo apt install cmake`以及Valgrind：`sudo apt-get install valgrind`

2. 确保已安装ssh：`sudo apt-get install openssh-server ssh`

3. 编辑ssh配置文件：`sudo vi /etc/ssh/sshd_config` （不熟悉vim的用户可用`nano`打开）

4. 1. 去掉`#PORT 22`前的#，也就是去掉这个注释符号
   2. 编辑`PasswordAuthentication yes`，确保其没有被注释掉
   3. `PermitRootLogin yes`，允许root用户登录

5. 重启ssh服务：`sudo service ssh restart`

6. 获取自己的ip地址：`ifconfig`

   （图中红框处本机ip地址）如若忘记了wsl的用户名或密码，可使用`cat /etc/passwd`查看用户组和`passwd `来修改密码

   ![img](https://raw.githubusercontent.com/valeeraZ/-image-host/master/640.png)

7. 从windows本机的终端，如Windows Powershell，尝试连接：`ssh <username>@<ipaddress>`

8. 成功后打开CLion，从左上角File找到**Settings / Preferences | Build, Execution, Deployment | ToolChains** ，右侧使用向上箭头将WSL调至最上方，修改右侧的参数：Credentials项点击右侧齿轮设置，添加配置并Test connection，应用。

   ![img](https://raw.githubusercontent.com/valeeraZ/-image-host/master/640-20201116073210224.png)

   ![img](https://raw.githubusercontent.com/valeeraZ/-image-host/master/640-20201116073219312.png)

9. 从CLion左上角File找到**Settings / Preferences | Build, Execution, Deployment | Dynamic Analysis Tools | Valgrind**，设置Valgrind的安装地址。在wsl中使用`which valgrind`来获得安装地址

   ![img](https://raw.githubusercontent.com/valeeraZ/-image-host/master/640-20201116073226080.png)

10. 使用Valgrind来运行项目（图中红框），在下方Run中，查看Valgrind的报错信息，若无错误则显示空白。

    ![img](https://raw.githubusercontent.com/valeeraZ/-image-host/master/640-20201116073235310.png)