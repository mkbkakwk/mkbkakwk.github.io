---
title: Winget 与 Scoop：Windows 装软件的现代姿势
published: 2026-09-04
updated: 2026-09-04
description: Windows 两大包管理器 Winget 与 Scoop 的安装与日常用法：装软件、更新、卸载、查找，一次讲清。
tags: [Windows, Winget, Scoop, 软件管理]
category: 工具
slug: winget-vs-scoop
pinned: false
draft: true
comment: true
---

## 为什么需要包管理器

在国内的 Windows 电脑上装软件，常见流程是：搜索 → 分辨官网和广告 → 躲开"高速下载"陷阱 → 提防捆绑 → 装完再手动找更新。每一步都是坑：

- **假网站与下载站**：点下的"下载"可能装的是全家桶
- **捆绑与广告**：安装器里偷偷勾选的推广软件
- **更新靠自觉**：软件不会通知你出了新版本
- **卸载不干净**：控制面板卸完，残留还在

手机上没这问题——装 App 走应用商店，一个入口管搜索、安装、更新、卸载。电脑上缺的正是这个"应用商店"。**包管理器就是电脑版应用商店**：把"找官网 → 下安装包 → 点下一步"换成一条命令：

```powershell
winget install 7zip.7zip
```

背后是从**官方维护的软件源**下载、校验、安装，更新和卸载同理。不用再自己找官网、没有捆绑广告、更新集中管理、卸载走官方渠道。老手还多一条：命令可复制、可写进脚本，重装系统后几十个软件一键归位。

Windows 上有两个主流选择：**Winget**（微软官方，系统自带）和 **Scoop**（社区开源，绿色便携）。怎么选、怎么用，下文分解。

> [!NOTE] 关于国内网络
> Winget 走微软官方 CDN（内容分发加速节点），国内通常稳定；Scoop 默认从 GitHub 拉取，**国内需配镜像**（见 FAQ）。

## 各自是什么：让系统装，还是装进自己背包

一句话建立认知：**Winget 是"让系统帮你装"，Scoop 是"把软件装进你自己的背包"**。

### Winget：微软官方，系统级安装

Windows 11 内置，Windows 10 较新版本可通过更新"应用安装程序"获得。它调度各软件的**官方安装器**：`winget install 7zip.7zip` 就是找到 7-Zip 官方下载地址 → 下载 → 用官方安装器装进 Program Files、写注册表——和你手动双击安装包**没有本质区别**。

- **优点**：系统级安装、行为最接近"正常装软件"；社区清单 + 商店双源，覆盖广；系统自带零成本
- **局限**：安装体验继承各家安装器（有的弹窗、带捆绑勾选），卸载干净度取决于软件自己的卸载器

### Scoop：社区开源，绿色便携

哲学与 Winget 相反：**尽量不碰系统**。软件全装进用户目录（`%USERPROFILE%\scoop\apps`），不写注册表、不要管理员权限；靠用户目录里的小启动器（shim）让命令行找到软件；卸载就是删目录。

- **优点**：免管理员（公司机也能装）；版本可控（多版本共存、随时回滚）；装更卸体验统一、无厂商安装器捣乱
- **局限**：覆盖面靠社区清单；GUI 大软件要去额外桶找；默认源在 GitHub，国内需镜像

> Winget = 请管家代劳：安置到位、登记在册，但每次要出示权限（UAC：系统弹窗让你确认，点"是"即可）；Scoop = 自带收纳箱：全塞进自己抽屉，一根标签条随取随用，不要了整包扔掉。系统级 vs 便携、留痕 vs 无痕，就是后面"选型"的判断依据。

## 先把自己装上

先解决一个基础问题：**命令在哪敲？** 按 `Win` 键，输入 `powershell` 回车，打开 PowerShell 窗口（Windows 11 也可以右键开始菜单 → 选"终端"）。下文所有命令都在这个窗口里输入、回车执行。

### Winget：自查 + 补装

```powershell
winget --version
```

能输出版本号 → 直接用；提示"无法识别" → 微软商店搜 **App Installer**（应用安装程序）装上。想手动升级它：

```powershell
winget upgrade Microsoft.DesktopAppInstaller
```

### Scoop：两条命令

PowerShell 里执行：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
```

第一行放开"运行远程脚本"权限（仅当前用户，官方推荐）；第二行跑安装脚本。验证：`scoop --version`。默认装到 `%USERPROFILE%\scoop`；想换到其他盘，先新建环境变量 `SCOOP`、值填目标目录再装（进阶操作，新手可跳过）。

> [!NOTE] 国内连不上 get.scoop.sh？
> 换 Gitee 官方镜像脚本：`irm https://gitee.com/glsnames/scoop-installer/raw/master/bin/install.ps1 | iex`。装完后的持续加速见 FAQ。

## 日常使用：查、装、更、卸

节奏比命令重要：**先查、再装**——装错软件比不装更麻烦。

### 查：有没有？哪个才是它？

先搜：

::: code-group labels=[winget, scoop]

```powershell
winget search 7zip
```

```powershell
scoop search 7zip
```

:::

`winget search 7zip` 的真实输出（整理成表格，完整结果更多）：

| 名称 | ID | 版本 | 匹配 | 源 |
| --- | --- | --- | --- | --- |
| 7-Zip | `7zip.7zip` | 26.02 | Moniker: 7zip | winget |
| NanaZip | `M2Team.NanaZip` | 6.5.1800.0 | Tag: 7zip | winget |
| 7-Zip ZS | `mcmilk.7zip-zstd` | 26.02-v1.5.7-R2 | Tag: 7zip | winget |
| 7zip解压缩软件 | `XP8C87R6J42SKH` | Unknown | — | msstore |

> 版本号是截图时刻的输出，会随上游更新——**以你搜索时为准**。

一眼全是"7-Zip"，其实**不是同一个东西**：

- **认主包**：ID = 发布者.产品名（`VideoLAN.VLC`、`7zip.7zip`）。再看"匹配"列——它告诉你**这个包是靠什么被搜到的**：`Moniker` 表示官方把 `7zip` 认领成了自己的别名（搜索词命中"本名"，强信号）；`Tag` 只是顺手打的标签（NanaZip 因为兼容 7z 格式被带了出来）。第一行 `Moniker: 7zip` + ID `7zip.7zip` 双重对上，就是正主
- **魔改版不是假货**：NanaZip、7-Zip ZS 是基于 7-Zip 源码的增强版，**除非明确想要，否则无视**
- **商店那行最可疑**：发布者不明的"7zip解压缩软件"是典型"看起来像官方"的包装货。拿不准就验货：

::: code-group labels=[winget, scoop]

```powershell
winget show XP8C87R6J42SKH --source msstore   # 商店应用必须带 --source msstore 才能查到
```

```powershell
scoop info 7zip              # 看清单里的官网与描述
```

:::

Homepage 不是 `7-zip.org` 就绕开。结论：**锁定官方主包 ID，拿不准验货，来路不明的商店应用默认不碰**。

Scoop 这边干净得多：`scoop search` 只搜你**已添加桶**里的结果——搜不到通常是桶没加，不是软件不存在。

> [!NOTE] 桶（bucket）是什么？
> Scoop 不存安装包，只存清单（软件从哪下载、怎么装），清单按主题分桶存放——桶就是"货架"。默认只带 `main` 桶，`main` 是官方精选、偏命令行工具；加桶方法见下一节。

查本机装了什么：`winget list` / `scoop list`；`scoop status` 看哪些有更新。

> [!TIP] 小白最短路径
> 只想快点装上？三步：① `winget search 7zip` → ② 认准 ID 是 `7zip.7zip` 的那行 → ③ `winget install 7zip.7zip`。其余结果全部无视，直接开装。

### 装：ID 已确认，一条命令

::: code-group labels=[winget, scoop]

```powershell
winget install --id Git.Git
winget install --id 7zip.7zip
```

```powershell
scoop install git
scoop install 7zip
```

:::

> [!NOTE] `--id` 不是必须
> ID 在源内全局唯一，所以 `winget install Git.Git` 不写 `--id` 也能装。加 `--id` 的意义是明确告诉命令行"只按 ID 字段匹配"，不做名称/别名猜测——结果可预期。临时安装带着稳，脚本里务必带。

Winget 有两个源：`winget`（社区维护清单）和 `msstore`（微软商店），默认前者；想装商店版加 `--source msstore`。

Scoop 默认只有 `main` 桶（git、7zip、python 这类精选工具）。GUI 大软件在额外桶里，装之前先加桶：

```powershell
scoop bucket add extras        # extras：GUI 软件；versions：多版本共存
scoop install extras/everything
```

> [!TIP] 一句话记住区别
> Winget 要管理员权限、系统级安装；Scoop 免管理员、全装用户目录。选型依据见后文。若安装时报权限错误，右键终端图标选**"以管理员身份运行"**再试。

### 更：一条命令全升

::: code-group labels=[winget, scoop]

```powershell
winget upgrade            # 列出可更新
winget upgrade --all      # 全部更新
```

```powershell
scoop update              # 更新 Scoop 自身
scoop update *            # 更新所有软件
```

:::

哲学差异在这里最明显：**Winget 每次走官方安装器**，可能弹 UAC、跳向导，偶尔静默失败（失败会列出，重试单个即可）；**Scoop 绿色覆盖**，不弹窗不写注册表，一条 `scoop update *` 全升完。

> [!NOTE] 带自动更新的软件（Edge、VS Code 等）
> `winget upgrade` 会跳过它们——因为软件自己会更新。想强制管可加 `--force`，但不建议依赖。

### 卸：来去都是一条命令

::: code-group labels=[winget, scoop]

```powershell
winget uninstall --id Git.Git
```

```powershell
scoop uninstall git
```

:::

Scoop 卸载默认保留个人配置（`~/scoop/persist`）；想连配置一起删，加 `-p`：`scoop uninstall git -p`。

## 常用命令速查表

两套体系互不知晓，命令一一对应：

| 操作 | Winget | Scoop |
| --- | --- | --- |
| 搜索 | `winget search <关键词>` | `scoop search <关键词>` |
| 查看详情 | `winget show <ID>` | `scoop info <包名>` |
| 安装 | `winget install --id <ID>` | `scoop install <包名>` |
| 安装（指定桶） | — | `scoop install <桶名>/<包名>` |
| 列出已装 | `winget list` | `scoop list` |
| 查看可更新 | `winget upgrade` | `scoop status` |
| 更新单个 | `winget upgrade --id <ID>` | `scoop update <包名>` |
| 更新全部 | `winget upgrade --all` | `scoop update *` |
| 更新自身 | `winget upgrade Microsoft.DesktopAppInstaller` | `scoop update` |
| 卸载 | `winget uninstall --id <ID>` | `scoop uninstall <包名>` |
| 卸载并清配置 | — | `scoop uninstall <包名> -p` |
| 添加软件源 | 商店安装 / `winget source add` | `scoop bucket add <桶名>` |

## 选型建议：别问哪个好，问你要什么

**默认选 Winget，不会错**：系统自带、覆盖全、行为和正常安装一致，普通用户最省事。

**选 Scoop**，命中任意一条：电脑**没有管理员权限**（公司机）；想要**绿色无残留**（不写注册表、卸载即删）；需要**多版本共存**；受够了安装器弹窗捆绑，想要**纯命令行体验**；终端重度用户。

**两者可以混用**，互不冲突。唯一原则：**同一个软件只走一个渠道**，别两套更新机制管同一个软件。

| 你的情况 | 用哪个 |
| --- | --- |
| 新手 / 只想安全省事装软件 | Winget |
| 公司电脑没管理员权限 | Scoop |
| 想要便携绿色、卸载干净 | Scoop |
| 需要多版本共存 | Scoop（`versions` 桶） |
| 想装商店 / 系统级应用 | Winget（含 `msstore` 源） |
| 开发主力机，命令行重度使用 | 混用：系统级走 Winget，工具走 Scoop |

## FAQ

**Q：需要管理员权限吗？**
Winget 装系统级软件会弹 UAC；Scoop 全程不需要，所有文件进用户目录——这是它能在无管理员电脑上用的原因。

**Q：Scoop 国内下载慢 / 失败？**
三招：① 安装时用 Gitee 官方镜像脚本（见上文）；② 桶换镜像源——桶本质是 git 仓库，可用社区 Gitee 镜像地址重加或改 remote（镜像地址变动频繁，用时搜最新）；③ **走代理**——在终端里先设置代理环境变量（`http_proxy` / `https_proxy`）再运行 Scoop，多数下载都能走通；不想折腾环境变量，就开能接管全局流量的代理工具。这条不受镜像时效影响，最省心。

**Q：Winget 下载也慢？**
Winget 只是调度员，安装包实际从**软件厂商服务器**下载——慢多半是厂商国内带宽问题。要么忍，要么用该软件的国内镜像/直链。

**Q：两个都用会不会冲突？**
不会打架，但别让同一软件被装两份：`scoop list` 里有的就别再用 winget 装。想换渠道，先在原渠道卸干净。

**Q：Scoop"绿色便携"安不安全？**
比手动下载站安全。清单开源可审，安装时校验安装包哈希，不符直接拒绝。它装的是正版软件本体，只是摆放方式不同。

## 写在最后

包管理器治不了所有厂商难缠的安装器，但它填上了最大的坑——"软件从哪来"。以后装软件先问一句：**这玩意儿能 `winget install` 或 `scoop install` 吗？** 不确定？把这个问题直接丢给 AI——让它给你精确的包 ID 和一条命令，比自己翻文档快得多。能一条命令解决的事，就别再去搜索引擎里冒险了。
