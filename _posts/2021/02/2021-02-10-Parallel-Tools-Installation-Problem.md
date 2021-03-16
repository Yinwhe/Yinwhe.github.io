---
layout:	post
title: "Kali 安装Parallels Tools的问题与解决"
subtitle: ""
date: 2021-02-10 23:00:00
author: "Yinwhe"
header-style: text
tags:
    - Solution
---

<details><summary>Source Web</summary>
  <ul>
  	<li><a href="https://blog.csdn.net/HonkerGaden/article/details/87096942?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control">Kali Linux 内核头文件安装</a></li>
    <li><a href="https://haxibiao.com/article/128924">解决Parallels Desktop(PD)上kali2020.2安装ParallesTools出错</a></li>
  </ul>
</details>



# 问题描述

<img src="https://i.loli.net/2021/02/11/zrJl6TjwSEUIZmi.png" style="zoom:25%;" />

主要问题是 - 缺少 `linux-headers-5.9.0-kali1-amd64`

### 解决

通过apt是安装不了的，显示只有5.10.0的版本，故在官网进行下载和安装

1. 进入[kali官网pool]([Index of /kali/pool/main/l/linux](http://http.kali.org/kali/pool/main/l/linux/))

2. 下载文件

   linux-kbuild-5.9_5.9.1-1kali2_amd64.deb

   linux-headers-5.9.0-kali1-common_5.9.1-1kali2_all.deb

   linux-headers-5.9.0-kali1-amd64_5.9.1-1kali2_amd64.deb

   具体文件可通过 `uname -a` 查看当前内核版本后，进行相应的调整

3. 依次安装 kbuild common 和 headers 文件，命令如下

   ```shell
   sudo dpkg -i linux-kbuild-5.9_5.9.1-1kali2_amd64.deb
   sudo dpkg -i linux-headers-5.9.0-kali1-common_5.9.1-1kali2_all.deb
   sudo dpkg -i linux-headers-5.9.0-kali1-amd64_5.9.1-1kali2_amd64.deb
   ```

   注意文件的位置要先放在当前目录；另外，如果出现`缺少linux-complier-gcc-10-x86`的报错，直接通过`apt`安装即可。



>  此时 headers 文件缺失的问题就解决了，但如果直接安装的话还是会出现 `cannot build linux kernel`的错误，原因好像是因为内核文件里面一些名称改了？

4. 进入`parallel tools`的`kmods`文件夹，进行如下修改：

   ```shell
   //先解压压缩包
   cd kmods/
   tar -xzf prl_mod.tar.gz
   rm prl_mod.tar.gz
   
   //修改prl_fs/SharedFolders/Guest/Linux/prl_fs/inode.c文件；
   vi prl_fs/SharedFolders/Guest/Linux/prl_fs/inode.c
   //添加如下代码：
   #define segment_eq(a, b) ((a).seg == (b).seg)
   
   //修改prl_fs_freeze/Snapshot/Guest/Linux/prl_freeze/prl_fs_freeze.c文件；
   vi prl_fs_freeze/Snapshot/Guest/Linux/prl_freeze/prl_fs_freeze.c
   //添加如下代码：
   #include <linux/blkdev.h>
   
   //修改完成后，重新打包；
   tar -zcvf prl_mod.tar.gz . dkms.conf Makefile.kmods
   ```

   然后再进行安装即可







> emmmm, 然后安装完了，然后kali坏了hhh，原因并不太清楚... 试过直接升级到5.10.0，但也会出现各种各样的问题...