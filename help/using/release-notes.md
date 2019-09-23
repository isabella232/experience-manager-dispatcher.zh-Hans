---
title: AEM Dispatcher发行说明
seo-title: AEM Dispatcher发行说明
description: Adobe Experience Manager Dispatcher的发行说明
seo-description: Adobe Experience Manager Dispatcher的发行说明
uuid: ae3ccf62-0514-4c03-a3b9-7179a482cbd
topic-tags: 发行说明
content-type: 引用
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
translation-type: tm+mt
source-git-commit: 2d72839d459973cba40f6a938ee157198c7cf50a

---


# AEM Dispatcher发行说明{#aem-dispatcher-release-notes}

## 发行信息 {#release-information}

|  |  |
|--- |--- |
| 产品 | Adobe Experience Manager(AEM)Dispatcher |
| 版本 | 4.3.2 |
| 类型 | 次要版本 |
| 日期 | 2019年1月31日 |
| 下载URL | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services(IIS)](release-notes.md#iis)</li></ul> |
| 兼容性 | AEM 6.1或更高版本 |

## 系统要求和先决条件 {#system-requirements-and-prerequisites}

有关要求和先决条件 [的更多信息](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html) ，请参阅支持的平台页面。

Adobe强烈建议使用最新版AEM Dispatcher来使用最新功能、最新错误修复和尽可能最佳的性能。

## 安装说明 {#installation-instructions}

有关详细说明，请参 [阅安装Dispatcher](dispatcher-install.md)。

## 发布历史 {#release-history}

### 版本4.3.2（2019年1月31日） {#jan}

**错误修复**:

* DISP-734 —— 如果未设置为处理程序，调度程序会在insert_output_filter中导致崩溃
* DISP-735 - RE在Alpine linux上不工作
* DISP-740 —— 默认情况下，在macOS Mojave中加载调度程序处于禁用状态
* DISP-742 —— 被阻止的请求可能会将信息泄露给身份验证检查器受保护的资源

**改进**:

* DISP-746 - dispatcher.any中未标记的字符串应生成警告

**新增功能**:

* DISP-747 —— 在Apache环境中提供请求信息

### 版本4.3.1（2018年10月16日） {#oct}

**错误修复**:

* DISP-656 - Dispatcher服务错误的ETag Header
* DISP-694 —— 在保持活动连接过时时禁止警告
* DISP-714 —— 基于Cookie的会话管理在IIS中无效
* DISP-715 - renderid cookie的安全标记
* DISP-720 —— 未关闭的临时文件可能导致用尽（打开的文件过多）
* DISP-721 —— 当Apache正常重新启动子项时，调度程序中断poll()
* DISP-722 —— 缓存文件是使用八进制模式0600创建的
* DISP-723 —— 当渲染超时设置为0时，隐式10分钟超时（并重试）
* DISP-725 —— 字符串后的尾部字符将无提示地转换为未命名值
* DISP-726 —— 当没有真正与传入主机匹配的农场时记录警告
* DISP-727 —— 调度程序检查空缓存文件的请求内容长度
* DISP-730 - 404，当尝试通过调度程序访问头文件时
* DISP-731 —— 调度程序易受日志注入的攻击
* DISP-732 —— 调度程序应删除URL中连续的“/”
* DISP-733 —— 调度程序应设置（计算）Age Header

**改进**:

* DISP-656 - Dispatcher服务错误的ETag Header
* DISP-694 —— 在保持活动连接过时时禁止警告
* DISP-715 - renderid cookie的安全标记
* DISP-722 —— 缓存文件是使用八进制模式0600创建的
* DISP-726 —— 当没有真正与传入主机匹配的农场时记录警告

### 版本4.3.0（2018年6月13日） {#jun}

**错误修复**:

* DISP-682 —— 数字日志级别应用不正确
* DISP-685 - 32位Solaris SPARC二进制文件对__divdi3的引用未定义
* DISP-688 —— 在404响应中，调度程序不返回“X-Cache-Info”头
* DISP-690 —— 上次修改时间的标题不可缓存
* DISP-691 —— 访问w3wp.exe中的违规操作
* DISP-693 —— 需要在调度程序下载页上更新Solaris服务器的架构详细信息
* DISP-695 - Dispatcher模块4.2.3中DispatcherLog级别的问题
* DISP-698 —— 调度程序TTL需要支持s-maxage和private指令
* DISP-700 —— 模块在Alpine linux上无法正确工作
* DISP-704 —— 将包含%2b的浏览器请求发送到未编码的发布者
* DISP-705 —— 由于两次释放或损坏(fasttop)导致调度程序崩溃
* DISP-706 —— 在失效过程中，调度程序会跟踪可导致无限循环的返回引用符号链接
* DISP-709 —— 阻止某些虚URL扩展
* DISP-710 —— 针对Linux的内部版本在Cent OS 6上不可用

**改进**:

* DISP-652 - Dispatcher提供错误的日期标题

## 实用资源 {#helpful-resources}

* [AEM Dispatcher概述](dispatcher.md)

## 下载 {#downloads}

### Apache 2.4 {#apache}

| 平台 | 架构 | SSL支持 | 下载 |
|---|---|---|---|
| AIX | PowerPC（32位） | 否 | [dispatcher-apache2.4.aix-powerpc-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-4.3.2.tar.gz) |
| AIX | PowerPC（32位） | 是 | [dispatcher-apache2.4.aix-powerpc-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-ssl-4.3.2.tar.gz) |
| AIX | PowerPC（64位） | 否 | [dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz) |
| AIX | PowerPC（64位） | 是 | [dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz) |
| Linux | i686（32位） | 否 | [dispatcher-apache2.4-linux-i686-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.2.tar.gz) |
| Linux | i686（32位） | 是 | [dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz) |
| Linux | x86_64（64位） | 否 | [dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz) |
| Linux | x86_64（64位） | 是 | [dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz) |
| macOS | x86_64（64位） | 否 | [dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz) |
| Solaris | AMD（32位） | 否 | [dispatcher-apache2.4-solaris-i386-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-i386-4.3.2.tar.gz) |
| Solaris | AMD（32位） | 是 | [dispatcher-apache2.4-solaris-i386-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-i386-ssl-4.3.2.tar.gz) |
| Solaris | AMD（64位） | 否 | [dispatcher-apache2.4-solaris-amd64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-amd64-4.3.2.tar.gz) |
| Solaris | AMD（64位） | 是 | [dispatcher-apache2.4-solaris-amd64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-amd64-ssl-4.3.2.tar.gz) |
| Solaris | SPARC（32位） | 否 | [dispatcher-apache2.4-solaris-sparc-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparc-4.3.2.tar.gz) |
| Solaris | SPARC（32位） | 是 | [dispatcher-apache2.4-solaris-sparc-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparc-ssl-4.3.2.tar.gz) |
| Solaris | SPARC（64位） | 否 | [dispatcher-apache2.4.solaris-sparcv9-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparcv9-4.3.2.tar.gz) |
| Solaris | SPARC（64位） | 是 | [dispatcher-apache2.4.solaris-sparcv9-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparcv9-ssl-4.3.2.tar.gz) |

### IIS {#iis}

| 平台 | 架构 | SSL支持 | 下载 |
|---|---|---|---|
| Windows | x86（32位） | 否 | [dispatcher-iis-windows-x86-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.2.zip) |
| Windows | x86（32位） | 是 | [dispatcher-iis-windows-x86-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl-4.3.2.zip) |
| Windows | x64（64位） | 否 | [dispatcher-iis-windows-x64-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.2.zip) |
| Windows | x64（64位） | 是 | [dispatcher-iis-windows-x64-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl-4.3.2.zip) |
