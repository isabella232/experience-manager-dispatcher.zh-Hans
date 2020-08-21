---
title: Dispatcher 问题疑难解答
seo-title: AEM Dispatcher问题疑难解答
description: 了解如何对Dispatcher问题进行疑难解答。
seo-description: 了解AEM Dispatcher问题疑难解答。
uuid: 9c109a48-d921-4b6e-9626-1158cebc41e7
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: a612e745-f1e6-43de-b25a-9adcaadab5cf
translation-type: tm+mt
source-git-commit: 5734e601379fda9a62eda46bded493b8dbd49a4c
workflow-type: tm+mt
source-wordcount: '553'
ht-degree: 7%

---


# Dispatcher 问题疑难解答 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>调度程序版本独立于AEM，但调度程序文档嵌入在AEM文档中。 请始终使用文档中嵌入的用于最新版AEM的Dispatcher文档。
>
>如果单击以前版本 AEM 文档中嵌入的 Dispatcher 文档链接，可能会重定向到此页面。

>[!NOTE]
>
>另请查阅调度程 [序知识库](https://helpx.adobe.com/cq/kb/index/dispatcher.html)、调 [度程序刷新问题疑难解答](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html) 和调度程 [序热点问题常见问题解答](dispatcher-faq.md) ，以获取更多信息。

## 检查基本配置 {#check-the-basic-configuration}

与往常一样，第一步是检查基础知识：

* [确认基本操作](#ConfirmBasicOperation)
* 检查Web服务器和调度程序的所有日志文件。 如有必要，请增 `loglevel` 加用于调度程序日 [志记录](#Logging)。

* [检查配置](#ConfiguringtheDispatcher):

   * 您有多个调度程序吗？

      * 您确定哪个调度程序正在处理您正在调查的网站／页面？
   * 您实施了过滤器吗？

      * 这些是否会影响你调查的事情？


## IIS诊断工具 {#iis-diagnostic-tools}

IIS提供各种跟踪工具，具体取决于实际版本：

* IIS 6 —— 可以下载和配置IIS诊断工具
* IIS 7 —— 跟踪完全集成

这些功能可以帮助您监控活动。

## 找不到IIS和404 {#iis-and-not-found}

使用IIS时，您可能会 `404 Not Found` 遇到在各种情况下返回的情况。 如果是，请参阅以下知识库文章。

* [IIS 6/7:POST方法返回404](https://helpx.adobe.com/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS 6:对包含基本路径的URL的请求将返 `/bin` 回 `404 Not Found`](https://helpx.adobe.com/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

还应检查调度程序缓存根目录和IIS文档根目录是否设置为同一目录。

## 删除工作流模型时出现问题 {#problems-deleting-workflow-models}

**症状**

在通过调度程序访问AEM作者实例时尝试删除工作流模型时出现的问题。

**复制步骤：**

1. 登录到您的作者实例（确认请求是通过调度程序发送的）。
1. 创建新的工作流；例如，将“标题”设置为workflowToDelete。
1. 确认已成功创建工作流。
1. 选择并右键单击工作流，然后单击“ **删除**”。

1. 单击&#x200B;**是**&#x200B;以确认。
1. 将出现一个错误消息框，其中显示：\
   &quot; `ERROR 'Could not delete workflow model!!`&quot;.

**分辨率**

将以下标题添加到文 `/clientheaders` 件的部分 `dispatcher.any` 中：

* `x-http-method-override`
* `x-requested-with`

```
{  
{  
/clientheaders  
{  
...  
"x-http-method-override"  
"x-requested-with"  
}
```

## 与mod_dir(Apache)的干扰 {#interference-with-mod-dir-apache}

这描述了调度程序如何与Apache `mod_dir` Web服务器内部进行交互，因为这会导致各种可能意外的效果：

### Apache 1.3 {#apache}

在Apache 1.3 `mod_dir` 中，处理URL映射到文件系统目录的每个请求。

它会：

* 将请求重定向到现有文 `index.html` 件
* 生成目录列表

启用调度程序后，它将自身注册为内容类型的处理程序，从而处理此类请求 `httpd/unix-directory`。

### Apache 2.x {#apache-x}

在Apache 2.x中，情况不同。 模块可以处理请求的不同阶段，如URL修正。 `mod_dir` 通过将请求（当URL映射到目录时）重定向到附加的URL来处理此 `/` 阶段。

调度程序不会截 `mod_dir` 取修正，但会完全处理对重定向URL(即附加的 `/` 请求)。 如果远程服务器(如AEM)以不同方式处理请求(当映射到现 `/a_path` 有目录时), `/a_path/` 这 `/a_path` 可能会造成问题。

如果发生这种情况，您必须执行以下任一操作：

* 禁 `mod_dir` 用调度 `Directory` 程序处 `Location` 理的或子树

* 使用 `DirectorySlash Off` 配置 `mod_dir` 不追加 `/`
