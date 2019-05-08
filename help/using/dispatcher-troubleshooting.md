---
title: 调度程序问题疑难解答
seo-title: AEM调度程序问题疑难解答
description: 了解如何解决Dispatcher问题。
seo-description: 了解如何解决AEM调度程序问题。
uuid: 9c109a48-d921-4b6 e-9626-1158cfc41 e7
cmgrlastmodified: 01.11.2007082229[aheimoz]
pageversionid: '1193211344162'
template: /apps/docs/templates/contentpage
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/Dispatcher
topic-tags: 调度程序
content-type: 引用
discoiquuid: a612e745-f1 e6-43de-b25 a-9adcaadab5 cf
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# 调度程序问题疑难解答 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Dispatcher版本独立于AEM，但Dispatcher文档嵌入AEM文档。始终使用嵌入到最新版AEM文档中的Dispatcher文档。
>
>如果您遵循了一个指向Dispatcher文档的链接，则可能已重定向到该页面，该链接嵌入到AEM的先前版本的文档中。

>[!NOTE]
>
>另请检查 [调度程序知识库](https://helpx.adobe.com/cq/kb/index/dispatcher.html)、 [疑难解答调度程序弹出问题](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html) 和 [调度程序热点问题常见问题解答](dispatcher-faq.md) 以了解更多信息。

## 检查基本配置 {#check-the-basic-configuration}

通常，第一步是检查基础知识：

* [确认基本操作](#ConfirmBasicOperation)
* 检查Web服务器和调度程序的所有日志文件。如有必要，请增加用于调度 `loglevel`[程序日志记录](#Logging)的使用。

* [检查配置](#ConfiguringtheDispatcher)：

   * 您有多个调度程序吗？

      * 您确定了哪些Dispatcher处理了正在调查的网站/页面？
   * 是否已实现过滤器？

      * 这些影响您正在调查的事项吗？


## IIS诊断工具 {#iis-diagnostic-tools}

IIS提供各种跟踪工具，具体取决于实际版本：

* IIS-可下载和配置IIS诊断工具
* IIS-跟踪完全集成

这些功能可帮助您监控活动。

## 找不到IIS和404 {#iis-and-not-found}

使用IIS时，您可能会遇到 `404 Not Found` 在各种情况下返回的体验。如果是，请参阅以下知识库文章。

* [IIS6/7：POST方法返回404](https://helpx.adobe.com/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS6：对包含基本路径 `/bin` 返回的URL的请求 `404 Not Found`](https://helpx.adobe.com/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

还应检查调度程序缓存根目录和IIS文档根目录是否已设置为同一目录。

## 删除工作流模型时出错 {#problems-deleting-workflow-models}

**症状**

通过Dispatcher访问AEM作者实例时尝试删除工作流模型的问题。

**重现的步骤：**

1. 登录到您的作者实例(确认通过调度程序路由请求)。
1. 创建新工作流程；例如，标题设置为WorkflowToDelete。
1. 确认已成功创建工作流。
1. 选择并右键单击工作流，然后单击 **删除**。

1. 单击**是**以确认。
1. 此时将显示一条错误消息框：\
   &quot; `ERROR 'Could not delete workflow model!!`&quot;.

**分辨率**

在文件 `/clientheaders` 的部分添加以下标题 `dispatcher.any` ：

* `x-http-method-override`
* `x-requested-with`

`{  
{  
/clientheaders  
{  
...  
"x-http-method-override"  
"x-requested-with"  
}`

## 干扰mod_ dir(Apache) {#interference-with-mod-dir-apache}

这描述调度程序与Apache Webserver `mod_dir` 内部交互的方式，因为这可能会导致各种潜在的意外效果：

### Apache1.3 {#apache}

在Apache1.3 `mod_dir` 中，处理URL映射到文件系统中某个目录的每个请求。

它将：

* 将请求重定向到现有 `index.html` 文件
* 生成目录列表

启用调度程序后，它将此类请求注册为内容类型 `httpd/unix-directory`的处理函数，从而处理此类请求。

### Apache2.x {#apache-x}

Apache2.x中的内容有所不同。模块可以处理请求的不同阶段，如URL修正。`mod_dir` 通过将请求(URL映射到目录)重定向到带有 `/` 附加内容的URL来处理此阶段。

Dispatcher不截取 `mod_dir` 修正，但将请求完全处理到重定向的URL(即 `/` 附加)。如果远程服务器(例如AEM)处理请求以与请求 `/a_path` 不同(映射到现有目录时 `/a_path/``/a_path` )，这可能会导致问题。

如果发生这种情况，您必须执行以下操作之一：

* 为 `mod_dir` 调度程序 `Directory` 处理的 `Location` 或子树禁用

* 用于 `DirectorySlash Off` 配置 `mod_dir` 不追加 `/`
