---
title: Dispatcher 问题疑难解答
seo-title: Troubleshooting AEM Dispatcher Problems
description: 了解如何解决 Dispatcher 问题。
seo-description: Learn to troubleshoot AEM Dispatcher issues.
uuid: 9c109a48-d921-4b6e-9626-1158cebc41e7
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: a612e745-f1e6-43de-b25a-9adcaadab5cf
exl-id: 29f338ab-5d25-48a4-9309-058e0cc94cff
source-git-commit: 26c8edbb142297830c7c8bd068502263c9f0e7eb
workflow-type: tm+mt
source-wordcount: '560'
ht-degree: 43%

---

# Dispatcher 问题疑难解答 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>虽然 Dispatcher 版本独立于 AEM，但 Dispatcher 文档会嵌入到 AEM 文档中。始终使用嵌入到最新版本的 AEM 文档中的 Dispatcher 文档。
>
>您可能是在单击以前版本的 AEM 文档中嵌入的 Dispatcher 文档链接后重定向到此页面。

>[!NOTE]
>
>检查 [Dispatcher知识库](https://helpx.adobe.com/experience-manager/kb/index/dispatcher.html), [Dispatcher刷新问题疑难解答](https://experienceleague.adobe.com/search.html?lang=en#q=troubleshooting%20dispatcher%20flushing%20issues&amp;sort=relevancy&amp;f:el_product=[Experience%20Manager]) 和 [Dispatcher常见问题解答](dispatcher-faq.md) 以了解更多信息。

## 检查基本配置 {#check-the-basic-configuration}

与往常一样，首要步骤是检查基本情况：

* [确认基本操作](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* 检查Web服务器和Dispatcher的所有日志文件。 如有必要，请将 `loglevel` 用于Dispatcher [记录](/help/using/dispatcher-configuration.md#logging).

* [检查您的配置](/help/using/dispatcher-configuration.md)：

   * 您是否有多个 Dispatcher？

      * 您是否已确定哪个 Dispatcher 正在处理您所调查的网站/页面？
   * 您是否已实施过滤器？

      * 这些过滤器是否会影响您正在调查的事项？


## IIS 诊断工具 {#iis-diagnostic-tools}

IIS 提供了各种跟踪工具，具体取决于实际版本：

* IIS 6 - 可以下载和配置 IIS 诊断工具
* IIS 7 - 跟踪已完全集成

这些工具可帮助您监控活动。

## IIS 和 404 未找到 {#iis-and-not-found}

使用IIS时，您可能会体验到 `404 Not Found` 在各种情况下返回。 如果是这样，请参阅以下知识库文章。

* [IIS 6/7：POST 方法返回 404](https://helpx.adobe.com/experience-manager/kb/IIS6IsapiFilters.html)
* [IIS 6:对包含基本路径的URL的请求 `/bin` 返回 `404 Not Found`](https://helpx.adobe.com/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

另外，请检查Dispatcher缓存根目录和IIS文档根目录是否设置为同一目录。

## 删除工作流模型时出现问题 {#problems-deleting-workflow-models}

**症状**

在通过 Dispatcher 访问 AEM 创作实例的情况下尝试删除工作流模型时出现问题。

**重现步骤：**

1. 登录到您的创作实例（确认请求通过Dispatcher路由）。
1. 创建工作流；例如，将标题设置为workflowToDelete。
1. 确认已成功创建该工作流。
1. 选择并右键单击工作流，然后单击 **删除**.

1. 单击&#x200B;**“是”**&#x200B;以确认。
1. 出现一个错误消息框，其中显示了以下内容：\
   &quot; `ERROR 'Could not delete workflow model!!`&quot;.

**解决方法**

将以下标头添加到 `dispatcher.any` 文件的 `/clientheaders` 部分：

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

## 对 mod_dir 的干预 (Apache) {#interference-with-mod-dir-apache}

此过程介绍Dispatcher如何与 `mod_dir` 在Apache Web服务器中，因为这可能会导致各种可能意外的效果：

### Apache 1.3 {#apache}

在Apache 1.3中， `mod_dir` 处理URL映射到文件系统中目录的每个请求。

它会：

* 将请求重定向到现有 `index.html` 文件
* 生成目录列表

启用Dispatcher后，它通过将自身注册为内容类型的处理程序来处理此类请求 `httpd/unix-directory`.

### Apache 2.x {#apache-x}

在Apache 2.x中，情况有所不同。 模块可以处理请求的不同阶段，例如 URL 修复。的 `mod_dir` 通过以下方法来处理此阶段：将请求（当URL映射到目录时）重定向到URL，该URL包含 `/` 已附加。

Dispatcher不会截获 `mod_dir` 修复，但会完全处理对重定向URL的请求(即， `/` 已附加)。 如果远程服务器(例如AEM)处理 `/a_path` 与 `/a_path/` (当 `/a_path` 映射到现有目录)。

如果出现这种情况，您必须执行以下任一操作：

* 禁用 `mod_dir` 对于 `Directory` 或 `Location` 调度程序处理的子树

* 使用 `DirectorySlash Off` 配置 `mod_dir` 而不追加 `/`
