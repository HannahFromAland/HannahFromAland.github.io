---
title: 【Ubuntu】VM tools drag file realization
date: 2020-12-10 17:52:42
tags: Ubuntu
category: Debug
---

记录完美解决虚拟机安装VM tools之后仅能实现屏幕自适应，无法进行物理机和虚拟机之间复制粘贴+文件拖拽功能的方法。

<!-- more -->

> 参考链接：[针对VMware的拖拽文件问题的终极解决方案！！](https://blog.csdn.net/qq_28912651/article/details/82848761?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)

首先要区分VM tools及Open VM Tools，在Ubuntu16.04之后官方推荐的应该是Open VM Tools，安装虚拟机系统时会有自带（但是会出现开头所说的bug），这也是为什么在虚拟机系统外`安装 VM Tools` 及虚拟机系统内`重新安装VM Tools`的选项均是灰色的.

补全原有Open VM Tools无法实现的两个功能的操作步骤如下：

- ### Uninstall just open-vm-tools

```bash
sudo apt-get remove open-vm-tools
```

- ### Uninstall open-vm-tools and its dependencies

```bash
sudo apt-get remove --auto-remove open-vm-tools
```

- ### Purging your config/data

```bash
sudo apt-get purge open-vm-tools
sudo apt-get purge --auto-remove open-vm-tools
```

- ### download patches

```bash
git clone https://github.com/rasa/vmware-tools-patches.git
```

- ### Reinstall open-vm-tools

```bash
sudo apt-get install open-vm-tools
sudo apt-get install open-vm-tools-desktop
```

- ### Reboot the system

```bash
reboot
```



