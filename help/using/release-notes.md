---
title: AEM Dispatcher 发行说明
seo-title: AEM Dispatcher 发行说明
description: 特定于Adobe Experience ManagerDispatcher的发行说明
seo-description: 特定于Adobe Experience ManagerDispatcher的发行说明
uuid: ae3ccf62-0514-4c03-a3b9-71799a482cbd
topic-tags: release-notes
content-type: reference
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
translation-type: tm+mt
source-git-commit: 328bc82673783b4a2df2d68481fa7eec88b74b01
workflow-type: tm+mt
source-wordcount: '805'
ht-degree: 9%

---


# AEM Dispatcher 发行说明{#aem-dispatcher-release-notes}

## 发行信息 {#release-information}

|  |  |
|--- |--- |
| 产品 | Adobe Experience Manager(AEM)调度程序 |
| 版本 | 4.3.3 |
| 类型 | 次要版本 |
| 日期 | 2019 年 18 月 10 日 |
| 下载 URL | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet信息服务(IIS)](release-notes.md#iis)</li></ul> |
| 兼容性 | AEM 6.1或更高版本 |

## 系统要求和先决条件 {#system-requirements-and-prerequisites}

有关要求和先决条件的详细信息，请参阅[支持的平台](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html)页。

Adobe强烈建议使用最新版AEM Dispatcher来使用最新功能、最新错误修复和最佳性能。

## 安装说明 {#installation-instructions}

有关详细说明，请参阅[安装Dispatcher](dispatcher-install.md)。

## 发布历史记录{#release-history}

### 版本4.3.3（2019年10月18日）{#october}

**错误修复**:

* DISP-739 - LogLevel调度程序：**level**&#x200B;无效
* DISP-749 - Alpine Linux调度程序因跟踪日志级别崩溃

**改进**:

* DISP-813 - Dispatcher中的openssl 1.1.x支持
* DISP-814 —— 缓存刷新期间出现Apache 40x错误
* DISP-818 - mod_expires为不可访问的内容添加缓存控制标头
* DISP-821 —— 不在套接字中存储日志上下文
* DISP-822 —— 调度程序应使用ppoll而不是pselect
* DISP-824 —— 安全DispatcherUseForwardedHost
* DISP-825 —— 当磁盘上没有更多空间时记录特殊消息
* DISP-826 —— 支持使用查询字符串重取URI

**新增功能**:

* DISP-703 —— 场特定缓存命中率
* DISP-827 —— 用于测试的本地服务器
* DISP-828 —— 为调度程序创建测试docker映像

### 版本4.3.2（2019年1月31日）{#jan}

**错误修复**:

* DISP-734 —— 如果未设置为处理程序，调度程序会导致insert_output_filter中的崩溃
* DISP-735 - RE在Alpine Linux上不工作
* DISP-740 —— 默认情况下，在macOS Mojave中加载调度程序处于禁用状态
* DISP-742 —— 被阻止的请求可能会将信息泄露给身份验证检查器受保护的资源

**改进**:

* DISP-746 —— 调度程序。any中未标记的字符串应生成警告

**新增功能**:

* DISP-747 —— 在Apache环境中提供请求信息

### 版本4.3.1（2018年10月16日）{#oct}

**错误修复**:

* DISP-656 - Dispatcher服务错误的ETag标头
* DISP-694 —— 当保持活动连接失效时禁止警告
* DISP-714 —— 基于Cookie的会话管理在IIS中无效
* DISP-715 —— 渲染器id cookie的安全标志
* DISP-720 —— 未关闭的临时文件可能导致用完（打开的文件过多）
* DISP-721 —— 当Apache正常重新启动子项时，调度程序中断poll()
* DISP-722 —— 使用八进制模式0600创建缓存文件
* DISP-723 —— 当渲染超时设置为0时，隐式10分钟超时（并重试）
* DISP-725 —— 字符串后的尾随字符将静默转换为未命名值
* DISP-726 —— 当没有与传入主机真正匹配的场时记录警告
* DISP-727 —— 调度程序检查空缓存文件的请求内容长度
* DISP-730 - 404，当尝试通过调度程序访问头文件时
* DISP-731 —— 调度程序易受日志注入的影响
* DISP-732 —— 调度程序应删除URL中连续的“/”
* DISP-733 —— 调度程序应设置（计算）年龄报头

**改进**:

* DISP-656 - Dispatcher服务错误的ETag标头
* DISP-694 —— 当保持活动连接失效时禁止警告
* DISP-715 —— 渲染器id cookie的安全标志
* DISP-722 —— 使用八进制模式0600创建缓存文件
* DISP-726 —— 当没有与传入主机真正匹配的场时记录警告

### 版本4.3.0（2018年6月13日）{#jun}

**错误修复**:

* DISP-682 —— 数字日志级别应用不正确
* DISP-685 - 32位Solaris SPARC二进制文件对__divdi3的引用未定义
* DISP-688 —— 在404响应上，调度程序不返回“X-Cache-Info”标头
* DISP-690 —— 上次修改时的标头不可缓存
* DISP-691 - w3wp.exe中的访问违规
* DISP-693 —— 需要在调度程序下载页上更新Solaris服务器的体系结构详细信息
* DISP-695 —— 调度程序模块4.2.3中的DispatcherLog级别问题
* DISP-698 —— 调度程序TTL需要支持s-maxage和private指令
* DISP-700 —— 模块在Alpine Linux上无法正常工作
* DISP-704 —— 将包含%2b的浏览器请求发送到未编码的发布者
* DISP-705 —— 由于多次释放或损坏（快速顶部）导致调度程序崩溃
* DISP-706 —— 在失效过程中，调度程序正在跟踪可导致无限循环的反向引用符号链接
* DISP-709 —— 阻止某些虚URL扩展
* DISP-710 —— 针对Linux的内部版本在Cent OS 6上不可用

**改进**:

* DISP-652 - Dispatcher服务错误的日期标头

## 有用资源 {#helpful-resources}

* [AEM Dispatcher概述](dispatcher.md)

## 下载 {#downloads}

### Apache 2.4 {#apache}

| 平台 | 架构 | OpenSSL支持 | 下载 |
|---|---|---|---|
| Linux | i686（32位） | 无 | [dispatcher-apache2.4-linux-i686-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.3.tar.gz) |
| Linux | i686（32位） | 1.0 | [dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz) |
| Linux | i686（32位） | 1.1 | [dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz) |
| Linux | x86_64（64位） | 无 | [dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz) |
| Linux | x86_64（64位） | 1.0 | [dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz) |
| Linux | x86_64（64位） | 1.1 | [dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz) |
| macOS | x86_64（64位） | 无 | [dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz) |

### IIS {#iis}

| 平台 | 架构 | OpenSSL支持 | 下载 |
|---|---|---|---|
| Windows | x86（32位） | 无 | [dispatcher-iis-windows-x86-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.3.zip) |
| Windows | x86（32位） | 1.0 | [dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip) |
| Windows | x86（32位） | 1.1 | [dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip) |
| Windows | x64（64位） | 无 | [dispatcher-iis-windows-x64-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.3.zip) |
| Windows | x64（64位） | 1.0 | [dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip) |
| Windows | x64（64位） | 1.1 | [dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip) |
