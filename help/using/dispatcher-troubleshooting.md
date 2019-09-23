---
title: 调度程序问题疑难解答
seo-title: AEM Dispatcher问题疑难解答
description: 了解如何解决Dispatcher问题。
seo-description: 了解如何对AEM Dispatcher问题进行疑难解答。
uuid: 9c109a48-d921-4b6e-9626-1158cebc41e7
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: '1193211344162'
template: /apps/docs/templates/contentpage
contentOwner: 用户
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: 调度程序
content-type: 引用
discoiquuid: a612e745-f1e6-43de-b25a-9adcaadab5cf
translation-type: tm+mt
source-git-commit: 76cffbfb616cd5601aed36b7076f67a2faf3ed3b

---


# 调度程序问题疑难解答 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>调度程序版本与AEM无关，但Dispatcher文档已嵌入到AEM文档中。 请始终使用嵌入到AEM最新版本文档中的Dispatcher文档。
>
>如果您遵循了指向Dispatcher文档的链接（该链接嵌入在AEM先前版本的文档中），则您可能已被重定向到此页。

>[!NOTE]
>
>另请查阅调度程序 [知识库](https://helpx.adobe.com/cq/kb/index/dispatcher.html)、调度程序 [刷新问题疑难解答和调度程序热](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html) 门问题常见问题解答 [](dispatcher-faq.md) ，以了解更多信息。

## 检查基本配置 {#check-the-basic-configuration}

与往常一样，第一步是检查基础知识：

* [确认基本操作](#ConfirmBasicOperation)
* 检查Web服务器和调度程序的所有日志文件。 如有必要，请增 `loglevel` 加用于调度程序记 [录的](#Logging)。

* [检查配置](#ConfiguringtheDispatcher):

   * 您有多个调度程序吗？

      * 您确定哪个调度程序正在处理您正在调查的网站／页面？
   * 您是否实施了过滤器？

      * 这些是否会影响您所调查的事情？


## IIS Diagnostic Tools {#iis-diagnostic-tools}

IIS提供各种跟踪工具，具体取决于实际版本：

* IIS 6 —— 可下载并配置IIS诊断工具
* IIS 7 —— 跟踪完全集成

这些功能可帮助您监控活动。

## IIS and 404 Not Found {#iis-and-not-found}

When using IIS you might experience `404 Not Found` being returned in various scenarios. 如果是，请参阅以下知识库文章。

* [IIS 6/7: POST method returns 404](https://helpx.adobe.com/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS 6: Requests to URLs that contain the base path  return `/bin``404 Not Found`](https://helpx.adobe.com/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

You should also check that the dispatcher cache root and the IIS document root are set to the same directory.

## Problems Deleting Workflow Models {#problems-deleting-workflow-models}

**Symptoms**

Problems trying to delete workflow models when accessing an AEM author instance through the Dispatcher.

**Steps to reproduce:**

1. Log in to your author instance (confirm that requests are being routed through the dispatcher).
1. Create a new workflow; for example, with the Title set to workflowToDelete.
1. Confirm that the workflow was successfully created.
1. Select and right click on the workflow, then click Delete.****

1. 单击&#x200B;**是**&#x200B;以确认。
1. 将出现一个错误消息框，其中显示：\
   " `ERROR 'Could not delete workflow model!!`".

**分辨率**

Add the following headers to the  section of your  file:`/clientheaders``dispatcher.any`

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

## Interference with mod_dir (Apache) {#interference-with-mod-dir-apache}

这描述了调度程序如何与Apache `mod_dir` Web服务器内部进行交互，因为这可能导致各种可能意外的效果：

### Apache 1.3 {#apache}

在Apache 1.3中， `mod_dir` 处理URL映射到文件系统中某个目录的每个请求。

它或者：

* 将请求重定向到现有文 `index.html` 件
* 生成目录列表

当调度程序处于启用状态时，它通过将自身注册为内容类型的处理函数来处理此类请求 `httpd/unix-directory`。

### Apache 2.x {#apache-x}

在Apache 2.x中，情况不同。 模块可以处理请求的不同阶段，如URL修正。 `mod_dir` 通过将请求（当URL映射到目录时）重定向到附加了URL来处理此阶 `/` 段。

调度程序不会截 `mod_dir` 取修正，但会完全处理对重定向URL的请求(即附加的 `/` 请求)。 如果远程服务器（如AEM）以不同方式处理请求（当映射到现有目录时）, `/a_path` 这可能会 `/a_path/` 造成 `/a_path` 问题。

如果发生这种情况，您必须执行以下任一操作：

* 对调度 `mod_dir` 程序 `Directory` 处理的或 `Location` 子树禁用

* 使用 `DirectorySlash Off` 配置 `mod_dir` 不追加 `/`
