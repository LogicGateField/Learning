# AI使用实验记录

- [AI使用实验记录](#ai使用实验记录)
  - [2026年7月16日：重构初始化函数的方式](#2026年7月16日重构初始化函数的方式)
    - [任务概述](#任务概述)
    - [关键影响点](#关键影响点)
      - [初始化函数中的动态内存分配](#初始化函数中的动态内存分配)
      - [销毁函数不需要再释放句柄的内存](#销毁函数不需要再释放句柄的内存)
      - [初始化函数调用方的变更](#初始化函数调用方的变更)
    - [过程记录](#过程记录)
      - [第一次：弱提示测试](#第一次弱提示测试)
      - [第二次：强提示测试](#第二次强提示测试)

## 2026年7月16日：重构初始化函数的方式

本实验基于OpenCode的OMO插件进行，推理部分使用`DeepSeek V4 Pro`模型，简单任务使用`DeepSeek V4 Flash`模型。

### 任务概述

本次实验的任务目标是将初始化函数中使用动态分配内存返回的句柄改为调用方提供，以适应MISRA-C的车规级要求。

一个具体的实例如下：

```C
int fm_system_wrapper_init(FMSystemWrapperType **ret_handle, char *config_filepath)
```

以上形式需要被重构为如下形式：

```C
int fm_system_wrapper_init(FMSystemWrapperType *ret_handle, char *config_filepath)
```

### 关键影响点

#### 初始化函数中的动态内存分配

例如文件管理系统的初始化函数如下：

```C
int fm_system_wrapper_init(FMSystemWrapperType **ret_handle, char *config_filepath)
{
    int retcode = 0;

    if ((ret_handle == NULL) || (config_filepath == NULL))
    {
        retcode = EINVAL;
        goto out_0;
    }

    FMSystemWrapperType *new_ret_handle = malloc(sizeof(FMSystemWrapperType));
    if (new_ret_handle == NULL)
    {
        goto out_0;
    }

...

out_1:
    free(new_ret_handle);
out_0:
    return retcode;
}
```

需要更改为如下形式：

```C
int fm_system_wrapper_init(FMSystemWrapperType *ret_handle, char *config_filepath)
{
    int retcode = 0;

    if ((ret_handle == NULL) || (config_filepath == NULL))
    {
        retcode = EINVAL;
        goto out_0;
    }

...

out_0:
    return retcode;
}
```

动态内存分配的部分应当被去除。

#### 销毁函数不需要再释放句柄的内存

一个销毁函数的示例如下：

```C
int fm_system_wrapper_destroy(FMSystemWrapperType *handle)
{
    int retcode = 0;

    if (handle == NULL)
    {
        retcode = EINVAL;
        goto out_0;
    }

...

    free(handle);

out_0:
    return retcode;
}
```

需要更改为如下形式：

```C
int fm_system_wrapper_destroy(FMSystemWrapperType *handle)
{
    int retcode = 0;

    if (handle == NULL)
    {
        retcode = EINVAL;
        goto out_0;
    }

...

out_0:
    return retcode;
}
```

内存释放的部分应当被移除

#### 初始化函数调用方的变更

一个调用方的示例如下：

```C
FMSystemWrapperType *sys = NULL;
ret = fm_system_wrapper_init(&sys, TEST_SYS_CFG_LOOP);
```

需要被更改为如下形式：

```C
FMSystemWrapperType sys;
ret = fm_system_wrapper_init(&sys, TEST_SYS_CFG_LOOP);
```

### 过程记录

#### 第一次：弱提示测试

提示词：

```txt
ulw
将初始化函数中使用动态分配内存返回的句柄改为调用方提供，以适应MISRA-C的车规级要求。
一个具体的实例如下：
int fm_system_wrapper_init(FMSystemWrapperType **ret_handle, char *config_filepath)
以上形式需要被重构为如下形式：
int fm_system_wrapper_init(FMSystemWrapperType *ret_handle, char *config_filepath)
需要注意的变更一共有几个：
1、初始化函数中的动态内存分配需要被移除
2、销毁函数不需要再释放句柄的内存
3、初始化函数调用方的变更（调用者需要传入一个句柄结构体变量的地址，而非句柄结构体指针的地址）
先分析本目录下的代码，找到有哪些需要更改的模块，每一轮只更改一到两个模块，每一轮都需要重新跑通test目录下的所有local测试、重新编译成功所有remote测试。确保上一轮没有问题再进行下一轮。
```

AI表现良好，除了以上我提到的点，还将所有模块函数调用的传入句柄加了个取地址符。

后续追加双向链表的修改（双向链表的初始化函数形式跟别的模块不一样，句柄指针直接从返回值返回给调用者），也没有问题，测试成功通过。

#### 第二次：强提示测试
