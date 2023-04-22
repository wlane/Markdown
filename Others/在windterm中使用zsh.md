---
title: 在windterm中使用zsh
tags:
- zsh
categories:
- blog
---

# 1.安装windterm

参考：[github-windterm](https://github.com/kingToolbox/WindTerm)

这里选择版本：

> WindTerm_2.5.0_Windows_Portable_x86_64.zip

解压缩后即可使用。

关于windterm的配置参考：

[WindTerm and WindEdit](https://kingtoolbox.github.io/)

安装之后默认情况下windterm不自带shell，使用的都是系统自带的shell，比如powershell，所以要想使用其他的shell，只需要在系统内安装好，windterm就可调用到。

# 2.在git bash中使用zsh

1. 安装git bash

   登录 https://git-scm.com/download/win 下载以下版本：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20220813225859.png)

   下载后双击选择安装目标位置解压缩，比如D:\PortableGit。

2. 下载zsh

   登录网站 https://packages.msys2.org/package/zsh?repo=msys&variant=x86_64 下载以下文件：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20220813230306.png)

   由于文件使用zstd压缩，所以如果默认无法解压缩该文件的话，可以将文件传至某个linux操作系统上解压：

   ~~~shell
   $ sudo yum install -y zstd
   $ mkdir zsh
   $ tar -I zstd -xvf zsh-5.8-5-x86_64.pkg.tar.zst  -C zsh
   $ ll -a zsh
   total 76
   drwxrwxr-x 4 figo figo  4096 Aug 13 23:09 .
   drwx------ 6 figo figo  4096 Aug 13 23:09 ..
   -rw-r--r-- 1 figo figo 13356 Jan 23  2021 .BUILDINFO
   drwxr-xr-x 3 figo figo  4096 Jan 23  2021 etc
   -rw-r--r-- 1 figo figo   265 Jan 23  2021 .INSTALL
   -rw-r--r-- 1 figo figo 36284 Jan 23  2021 .MTREE
   -rw-r--r-- 1 figo figo   518 Jan 23  2021 .PKGINFO
   drwxr-xr-x 5 figo figo  4096 Jan 23  2021 usr
   $ tar -cvf zsh.tar.gz zsh
   ~~~

   将zsh.tar.gz文件下载到windows本地，解压缩，注意usr/bin/zsh.exe大小不为0，否则需要将zsh-5.8.exe改为zsh.exe：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20220813231254.png)

   将上面zsh目录中的所有文件复制到安装的git bash目录下：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20220813231443.png)

   此时，双击git-bash.exe测试：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20220813231710.png)

   直接输入zsh，会出现下列显示，代表zsh已经成功安装好：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20220813231803.png)

3. 安装oh-my-zsh

   官网：[oh-my-zsh](https://ohmyz.sh/)

   zsh安装好之后，需要对它做一些相应的配置，比如主题，tab补全，历史记录等等，oh-my-zsh是一个比较成熟的专门用来配置zsh的框架。

   由于zsh的安装脚本在 https://raw.githubusercontent.com 上，需要科学上网，所以建议使用手动安装的方式。

   双击git-bash.exe打开一个shell窗口，下面的操作都在这个窗口完成：

   ~~~shell
   $ git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh
   $ cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
   $ vi ~/.bashrc
   # Launch Zsh
   if [ -t 1 ]; then
       exec zsh
   fi
   ~~~

   双击git-bash.exe打开另外一个git bash窗口，出现下图：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20220814174845.png)

   warning可忽略，证明已经安装正确，打开已经默认切换到oh-my-zsh的配置。

4. 配置oh-my-zsh

   各种插件可见：https://github.com/zsh-users/

   ~~~shell
   # 自动补全
   $ git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
   # 语法高亮
   $ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
   # 修改主题，添加插件和中文支持
   $ vi ~/.zshrc
   。。。。。。
   ZSH_THEME="ys"
   。。。。。。
   plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
   。。。。。。
   export LC_ALL=en_US.UTF-8  
   export LANG=en_US.UTF-8
   ~~~

5. 添加windterm支持

   在windterm上添加一个新的session：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20220814000819.png)

   登录之后如下图所示：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20220814001055.png)

   