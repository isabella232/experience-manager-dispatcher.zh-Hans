---
title: AEM调度程序发行说明
seo-title: AEM调度程序发行说明
description: 特定于Adobe Experience Manager Dispatcher的发行说明
seo-description: 特定于Adobe Experience Manager Dispatcher的发行说明
uuid: ae3cf62-0514-4c03-a3 b9-71799a482 cbd
topic-tags: release-notes
content-type: 引用
products: SG_ EXPERIENCE MANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393 aac5
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# AEM调度程序发行说明{#aem-dispatcher-release-notes}

## 发行信息 {#release-information}

|  |
|--- |--- |
| 产品 | Adobe Experience Manager(AEM)调度程序 |
| 版本 | 4.3.2 |
| 类型 | 次要版本 |
| 日期 | 2019年月31日 |
| 下载URL | <ul><li>[Apache2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services(IIS)](release-notes.md#iis)</li></ul> |
| 兼容性 | AEM6.1或更高版本 |

## 系统要求和先决条件 {#system-requirements-and-prerequisites}

有关要求和先决条件，请参阅 [支持的平台](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html) 页面。

Adobe强烈建议使用最新版AEM Dispatcher来嵌入最新功能、最新错误修复以及最佳性能。

## 安装说明 {#installation-instructions}

有关详细说明，请参阅 [安装Dispatcher](dispatcher-install.md)。

## 发行历史记录 {#release-history}

### 版本4.3.2(2019年月31日) {#jan}

**错误修复**：

* DISP-734-如果未设置为处理函数，Dispatcher将导致插入_ output_ filter中崩溃
* DISP-735- Righten Linux无法使用RIA
* DISP-740-默认情况下，在macOS Mojave中加载调度程序处于禁用状态
* DISP-742-阻止的请求可能会泄露信息至身份检查器保护的资源

**改进**：

* DISP-746-调度程序中未加标签的字符串应生成警告

**新增功能**:

* DISP-747-在Apache环境中提供请求信息

### 版本4.3.1(2018年10月16日) {#oct}

**错误修复**：

* DISP-656- Dispatcher服务错误的eTag Header
* DISP-694-在保持活动连接过时时抑制警告
* DISP-714-基于Cookie的会话管理在IIS中无效
* DISP-715-渲染id cookie的安全旗标
* DISP-720-临时文件未关闭，可能导致完整的文件(过多打开的文件)
* DISP-721-当Apache完美地重新启动子级时，调度程序会中断投票()
* DISP-722-缓存文件使用八进制模式0600创建
* DISP-723-在渲染超时设置为时，隐式10分钟超时(并重试)
* DISP-725-字符串之后的尾部字符被无提示转换为未命名值
* DISP-726-当没有农场实际匹配到来的主机时记录警告
* DISP-727-调度程序检查空缓存文件的请求内容长度
* DISP-730-404当尝试通过调度程序访问头文件时
* DISP-731-调度程序容易受到日志注入
* DISP-732-调度程序应在URL中删除连续“/”
* DISP-733-调度程序应设置(计算)年龄标题

**改进**：

* DISP-656- Dispatcher服务错误的eTag Header
* DISP-694-在保持活动连接过时时抑制警告
* DISP-715-渲染id cookie的安全旗标
* DISP-722-缓存文件使用八进制模式0600创建
* DISP-726-当没有农场实际匹配到来的主机时记录警告

### 版本4.3.0(2018年月13日) {#jun}

**错误修复**：

* DISP-682-数字日志级别错误应用
* DISP-685-32位Solaris SPARC二进制文件具有对__ divdi的未定义引用
* DISP-688- Dispatcher在404响应上不返回“X-Cache-Info”头
* DISP-690-上次修改的标题不可缓存
* DISP-691- w3p. exe中的访问违规
* DISP-693-需要更新调度程序下载页面上Solaris服务器的架构详细信息
* DISP-695- Dispatcher模块4.2.3中的DispatcherLog级别问题
* DISP-698-调度程序TTL需要支持s-maxx和私有指令
* DISP-700-在Alpin Linux上无法正确使用模块
* DISP-704-包含%2b的浏览器请求被发送到发布者未编码
* DISP-705-由于双重免费或损坏导致调度程序崩溃(fasttop)
* DISP-706-在失效期间，调度程序返回引用的Symblink，这会导致无限循环
* DISP-709-阻止某些虚URL扩展
* DISP-710-针对Linux的构建不可用于Lent OS6

**改进**：

* DISP-652- Dispatcher提供错误的Date标题

## 有用的资源 {#helpful-resources}

* [AEM调度程序概述](dispatcher.md)

## 下载 {#downloads}

### Apache2.4 {#apache}

| 平台 | 架构 | SSL支持 | 下载 |
|---|---|---|---|
| AIX | PowerPC(32位) | 否 | [dispatcher-apache2.4-aix-powerpc-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-4.3.2.tar.gz) |
| AIX | PowerPC(32位) | 是 | [dispatcher-apache2.4-aix-powerpc-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-ssl-4.3.2.tar.gz) |
| AIX | PowerPC(64位) | 否 | [dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz) |
| AIX | PowerPC(64位) | 是 | [dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz) |
| Linux | i686(32位) | 否 | [dispatcher-apache2.4-linux-i686-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.2.tar.gz) |
| Linux | i686(32位) | 是 | [dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz) |
| Linux | x86_64(64位) | 否 | [dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz) |
| Linux | x86_64(64位) | 是 | [dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz) |
| macOS | x86_64(64位) | 否 | [dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz) |
| Solaris | AMD(32位) | 否 | [dispatcher-apache2.4-solaris-i386-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-i386-4.3.2.tar.gz) |
| Solaris | AMD(32位) | 是 | [dispatcher-apache2.4-solaris-i386-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-i386-ssl-4.3.2.tar.gz) |
| Solaris | AMD(64位) | 否 | [dispatcher-apache2.4-solaris-amd64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-amd64-4.3.2.tar.gz) |
| Solaris | AMD(64位) | 是 | [dispatcher-apache2.4-solaris-amd64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-amd64-ssl-4.3.2.tar.gz) |
| Solaris | SPARC(32位) | 否 | [dispatcher-apache2.4-solaris-sparc-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparc-4.3.2.tar.gz) |
| Solaris | SPARC(32位) | 是 | [dispatcher-apache2.4-solaris-sparc-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparc-ssl-4.3.2.tar.gz) |
| Solaris | SPARC(64位) | 否 | [dispatcher-apache2.4-solaris-sparcv9-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparcv9-4.3.2.tar.gz) |
| Solaris | SPARC(64位) | 是 | [dispatcher-apache2.4-solaris-sparcv9-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparcv9-ssl-4.3.2.tar.gz) |

### IIS {#iis}

| 平台 | 架构 | SSL支持 | 下载 |
|---|---|---|---|
| Windows | x86(32位) | 否 | [dispatcher-iis-windows-x86-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.2.zip) |
| Windows | x86(32位) | 是 | [dispatcher-iis-windows-x86-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl-4.3.2.zip) |
| Windows | x64(64位) | 否 | [dispatcher-iis-windows-x64-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.2.zip) |
| Windows | x64(64位) | 是 | [dispatcher-iis-windows-x64-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl-4.3.2.zip) |
