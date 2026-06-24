# OpenCode部署与基本配置

- [OpenCode部署与基本配置](#opencode部署与基本配置)
  - [安装与模型配置](#安装与模型配置)
  - [OMO插件](#omo插件)
    - [安装方法](#安装方法)
    - [模型配置](#模型配置)
  - [Skills生成与安装（以GJB-8114代码规范为例）](#skills生成与安装以gjb-8114代码规范为例)
    - [纯文本预处理](#纯文本预处理)
    - [使用DeepSeek Flash生成Skill内容](#使用deepseek-flash生成skill内容)
    - [将Skill加入到OpenCode](#将skill加入到opencode)


## 安装与模型配置

在终端中执行：

```Shell
sudo npm install -g opencode-ai
```

如果没有`npm`，则使用APT安装：

```Shell
sudo apt install npm -y
```

然后关闭此终端，在一个新的终端中执行：

```Shell
opencode
```

进入OpenCode之后，使用命令`/connnect`连接到模型提供商，使用`/models`更换模型。

如果要修改其他配置，可以修改`~/.config/opencode/opencode.jsonc`，具体参考官方修改说明：

> https://opencode.ai/docs/zh-cn/config/

## OMO插件

该插件是一个自动Agent编排器，可以根据任务需要自行调用不同的Agent，以尽量节省主任务上下文、加速并行任务执行。

### 安装方法

安装方法很简单，进入OpenCode之后，直接在OpenCode的对话框输入：

```Prompt
Install and configure oh-my-openagent by following the instructions here:
https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/refs/heads/dev/docs/guide/installation.md
```

然后按照要求回答问题即可，直到提示安装完毕。

### 模型配置

如果需要编辑默认的模型配置，可以编辑`~/.config/opencode/oh-my-openagent.json`文件：

```Shell
vim ~/.config/opencode/oh-my-openagent.json
```

以下展示我的配置（梁爷爷的恩情还不完）：

```txt
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/dev/assets/oh-my-opencode.schema.json",
  "agents": {
    "sisyphus": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "hephaestus": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "oracle": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "librarian": {
      "model": "deepseek/deepseek-v4-flash"
    },
    "explore": {
      "model": "deepseek/deepseek-v4-flash"
    },
    "multimodal-looker": {
      "model": "deepseek/deepseek-v4-flash"
    },
    "prometheus": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "metis": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "momus": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "atlas": {
      "model": "deepseek/deepseek-v4-flash"
    },
    "sisyphus-junior": {
      "model": "deepseek/deepseek-v4-pro"
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "ultrabrain": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "deep": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "artistry": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "quick": {
      "model": "deepseek/deepseek-v4-flash"
    },
    "unspecified-low": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "unspecified-high": {
      "model": "deepseek/deepseek-v4-pro"
    },
    "writing": {
      "model": "deepseek/deepseek-v4-pro"
    }
  }
}
```

## Skills生成与安装（以GJB-8114代码规范为例）

### 纯文本预处理

如果你的代码规范是一份文本PDF，可以直接跳过此步骤。

但是如果你拿到的文件是一份扫描件，就要使用一些方法（比如WPS的扫描件转文字）生成一个txt纯文本，以下是GJB-8114的一部分纯文本示例：

```txt
4 总则
　　本标准第5章规定了C 和C++   语言编程时应遵循的共用准则，第6章规定了C++   语言编程时应 遵循的专用准则，准则汇总索引见附录A。
　　本标准中的准则分为强制准则和建议准则，其中强制准则在软件编程中应遵循，建议准则在软件编 程中可参照执行。强制准则遵循情况的评价要求和方法见第7章。
5 C 和C++ 的共用准则
注：C 和C++ 的共用准则中，虽有个别准则只适用于C, 对C++  而言编译会报错，但本标准不再特意区分说明。
5.1 声明定义
5.1.1  强制准则
5.1.1.1 准则R-1-1-1
禁止通过宏定义改变关键字和基本类型含义。 
违背示例：

=define long 100          //违背1
int main(void)

int  i,
i =long,
return(0),





GJB8114-2013

遵循示例：

#define LONG_NUM 100      //遵循1
int main(void)

int  i,
i=LONG_NUM;
　　retum(0), {
```

格式不需要很好，乱乱的也行，反正也是给AI总结。

### 使用DeepSeek Flash生成Skill内容

准备好文本之后，直接打开DeepSeek的网页版（Flash可以上传文件，够用了，主要是不花钱），将文件上传到DeepSeek，并且输入以下提示词：

```Prompt
参考https://opencode.ai/docs/zh-cn/skills/，将文档总结为opencode可用的SKILL.md，以代码的方式呈现。
```

然后就可以得到一份Markdown代码，一部分的示例如下：

```Markdown
---
name: gjb-8114-2013
description: GJB 8114-2013 C/C++语言编程安全子集编码规范。涵盖152条强制准则和52条建议准则，适用于军用安全关键软件的C/C++开发。
license: 本标准为中华人民共和国国家军用标准，版权归中国人民解放军总装备部所有。
compatibility: opencode
metadata:
  standard: GJB 8114-2013
  category: coding-standards
  language: c-cpp
---

# GJB 8114-2013 C/C++ 语言编程安全子集

## 技能目标

本技能为 C/C++ 开发者提供 GJB 8114-2013《C/C++ 语言编程安全子集》的完整编码准则索引。该标准规定了严于语法规定的编程安全子集，适用于军用安全关键软件的 C/C++ 开发。

## 适用场景

- 军用安全关键软件的 C/C++ 代码编写与审查
- 代码静态分析工具的规则配置参考
- 编码规范的遵循性评价
- 安全关键系统的代码走查与评审

## 准则分类说明

本标准中的准则分为两类：
- **强制准则 (R)**：软件编程中**必须**遵循
- **建议准则 (A)**：软件编程中**推荐**参照执行

## 一、C/C++ 共用强制准则 (R-1-1-x ~ R-1-13-x)

<details>
<summary><b>点击展开：声明定义 (5.1) - 23条强制准则</b></summary>

| 编号    | 准则内容                                              |
| ------- | ----------------------------------------------------- |
| R-1-1-1 | 禁止通过宏定义改变关键字和基本类型含义                |
| R-1-1-2 | 禁止将其他标识宏定义为关键字和基本类型                |
| R-1-1-3 | 用typedef自定义的类型禁止被重新定义                   |
| R-1-1-4 | 禁止重新定义C或C++的关键字                            |
| R-1-1-5 | 禁止#define被重复定义（条件编译和#undef后再定义除外） |
| R-1-1-6 | 函数中的#define和#undef必须配对使用                   |
```

### 将Skill加入到OpenCode

然后在目录`~/.config/opencode/skills`（如果没有该目录则创建该目录）中创建一个新目录，比如`~/.config/opencode/skills/jgb-8114`，然后创建文件`~/.config/opencode/skills/jgb-8114/SKILL.md`，并且将DeepSeek生成的Markdown代码复制进这个文件，就完成了一个Skill的创建。

完成创建之后，直接打开OpenCode，在提示词中如果有“遵循GJB-8114”相关字眼，OpenCode就会自动调用这个Skill进行代码规范编写和检查，不需要手动进行调用，非常方便。
