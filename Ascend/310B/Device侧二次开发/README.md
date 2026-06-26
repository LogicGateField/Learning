# Device侧二次开发

- [Device侧二次开发](#device侧二次开发)
  - [修改内核源码](#修改内核源码)
    - [提取修补后的内核源码（首次）](#提取修补后的内核源码首次)
    - [初始化GIT（首次）](#初始化git首次)
    - [修改源码](#修改源码)
    - [生成修补文件](#生成修补文件)
    - [修补的并入](#修补的并入)
  - [新增驱动](#新增驱动)
  - [修改文件系统](#修改文件系统)
  - [其他技巧](#其他技巧)
    - [权限问题](#权限问题)


本文基于Device侧源码包，名称为`Ascend-hdk-xxx-sdk-ep_device_xxx.zip`。

首先需要按照官方论坛提供的方式完成一次Device侧内核源码编译，参考链接如下：

> https://developer.huawei.com/home/forum/ascend/thread-02168212749543786015-1-1.html

## 修改内核源码

昇腾内核的编译过程分为解压、修补和编译三步。这里我们需要先提取修补后的源码，然后在此基础上进行修改后形成修补文件，最后再将修补文件并入编译流程中。

本节以`/data`为主要工作目录，此工作目录可以换成其他目录。

具有“首次”标志的任务项只需要执行一次，目的是建立一个初始的GIT仓库，后续所有的更改都可以基于此仓库进行，无需再次从昇腾的SDK文件中将内核复制到工作区和新建GIT仓库。

### 提取修补后的内核源码（首次）

修补后的源码在`/opt/Ascend310PEP-device-source/kernel/kernel/kernel/out/linux-4.19`目录中。虽然该目录看似是4.19的内核，其实这估计是个历史遗留问题，不用管那个版本号的事儿，这个源码就是源码包里面的版本。

首先，我们进入此目录，并且执行：

```Shell
cp -rf ./ /data/kernel
```

### 初始化GIT（首次）

修补文件需要使用git进行生成。如果是第一次复制出来的代码，需要先初始化源码库（耗费较长时间，耐心等待即可）：

```Shell
git init
git add .
git commit -m "init"
```

### 修改源码

直接修改`/data/kernel`目录中的源码即可。这里演示修改`/data/kernel/drivers/amba/bus.c`文件：


```C
// SPDX-License-Identifier: GPL-2.0-only
/*
 *  linux/arch/arm/common/amba.c
 *
 *  Copyright (C) 2003 Deep Blue Solutions Ltd, All Rights Reserved.
 */
#include <linux/module.h>
#include <linux/init.h>
#include <linux/device.h>
#include <linux/string.h>
#include <linux/slab.h>
#include <linux/io.h>
#include <linux/pm.h>
#include <linux/pm_runtime.h>
#include <linux/pm_domain.h>
#include <linux/amba/bus.h>
#include <linux/sizes.h>
#include <linux/limits.h>






#include <linux/clk/clk-conf.h>
#include <linux/platform_device.h>
#include <linux/reset.h>

#include <asm/irq.h>

#define to_amba_driver(d)	container_of(d, struct amba_driver, drv)

/* called on periphid match and class 0x9 coresight device. */
static int
amba_cs_uci_id_match(const struct amba_id *table, struct amba_device *dev)

...
```

对的我只加了几行空行，这对于演示来说够用了。

### 生成修补文件

在`/data/kernel`目录下执行命令：

```Shell
git add .
git commit -m "demo" # 双引号内的字符串可以换成其他你喜欢的
```

然后输入`git log`指令查看历史提交：

```Shell
ommit 0f4cdfeca749eb4e35bb46b89776750b83a77cc0 (HEAD -> master)
Author: Zhang Fan <zfnemail@126.com>
Date:   Thu Jun 25 10:05:50 2026 +0800

    demo

commit 76acd024d45e921de6d44cf859dacce737c12492
Author: Zhang Fan <zfnemail@126.com>
Date:   Thu Jun 25 10:05:22 2026 +0800

    init
```

执行以下命令生成修补文件：

```Shell
git format-patch -1 0f4cdfeca749eb4e35bb46b89776750b83a77cc0
```

然后可以在`/data/kernel`目录下找到一个`0001-demo.patch`文件，其内容如下：

```txt
From 0f4cdfeca749eb4e35bb46b89776750b83a77cc0 Mon Sep 17 00:00:00 2001
From: Zhang Fan <zfnemail@126.com>
Date: Thu, 25 Jun 2026 10:05:50 +0800
Subject: [PATCH] demo

---
 drivers/amba/bus.c | 6 ++++++
 1 file changed, 6 insertions(+)

diff --git a/drivers/amba/bus.c b/drivers/amba/bus.c
index 52ab582..44a4604 100644
--- a/drivers/amba/bus.c
+++ b/drivers/amba/bus.c
@@ -16,6 +16,12 @@
 #include <linux/amba/bus.h>
 #include <linux/sizes.h>
 #include <linux/limits.h>
+
+
+
+
+
+
 #include <linux/clk/clk-conf.h>
 #include <linux/platform_device.h>
 #include <linux/reset.h>
-- 
2.25.1

```

这样就完成了修补文件的生成了。

### 修补的并入

将生成的`0001-demo.patch`文件复制到`/opt/Ascend310PEP-device-source/kernel/kernel/kernel/patches/product`目录中，并且编辑`/opt/Ascend310PEP-device-source/kernel/kernel/kernel/product_series.conf`文件，添加如下行：

```txt
patches/product/0001-demo.patch
```

然后再次编译内核，完成图形界面配置之后就可以查看`/opt/Ascend310PEP-device-source/kernel/kernel/kernel/out/linux-4.19/drivers/amba/bus.c`文件的内容了：

```C
// SPDX-License-Identifier: GPL-2.0-only
/*
 *  linux/arch/arm/common/amba.c
 *
 *  Copyright (C) 2003 Deep Blue Solutions Ltd, All Rights Reserved.
 */
#include <linux/module.h>
#include <linux/init.h>
#include <linux/device.h>
#include <linux/string.h>
#include <linux/slab.h>
#include <linux/io.h>
#include <linux/pm.h>
#include <linux/pm_runtime.h>
#include <linux/pm_domain.h>
#include <linux/amba/bus.h>
#include <linux/sizes.h>
#include <linux/limits.h>






#include <linux/clk/clk-conf.h>
#include <linux/platform_device.h>
#include <linux/reset.h>

#include <asm/irq.h>

#define to_amba_driver(d)	container_of(d, struct amba_driver, drv)

/* called on periphid match and class 0x9 coresight device. */
static int
amba_cs_uci_id_match(const struct amba_id *table, struct amba_device *dev)

...
```

确实发生了更改，可见修补文件成功并入到内核源码之中。

完成并入之后，记得将`/data/kernel`目录下生成的修补文件删除哦，不然下一次修补的时候就会连着修补文件一起修补进内核了。

## 新增驱动

官方指导：

> https://support.huawei.com/enterprise/zh/doc/EDOC1100548032/23b088c0?idPath=23710424|251366513|254884019|261408772|263024819

## 修改文件系统

官方指导：

> https://support.huawei.com/enterprise/zh/doc/EDOC1100548032/5eea6d13

## 其他技巧

### 权限问题

在华为官方的SDK使用指导中，需要将SDK全部解压到`/opt`下面，并且进入root用户进行编译。其实使用`chown`指令更改SDK目录的所有者之后，也可以顺利进行编译。

使用该种方法可以确保权限最小化，避免root权限对系统其他文件产生干扰，提高了编译过程的安全性。
