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
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: ht
source-wordcount: '543'
ht-degree: 100%

---

# Dispatcher 问题疑难解答 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>虽然 Dispatcher 版本独立于 AEM，但 Dispatcher 文档会嵌入到 AEM 文档中。始终使用嵌入到最新版本的 AEM 文档中的 Dispatcher 文档。
>
>您可能是在单击以前版本的 AEM 文档中嵌入的 Dispatcher 文档链接后重定向到此页面。

>[!NOTE]
>
>此外，请查看 [Dispatcher 知识库](https://helpx.adobe.com/cn/cq/kb/index/dispatcher.html)、[解决 Dispatcher 刷新问题](https://helpx.adobe.com/cn/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html)和 [Dispatcher 常见问题解答](dispatcher-faq.md)以了解更多信息。

## 检查基本配置 {#check-the-basic-configuration}

与往常一样，首要步骤是检查基本情况：

* [确认基本操作](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* 查看 Web 服务器和 Dispatcher 的所有日志文件。如有必要，提高用于 Dispatcher [记录](/help/using/dispatcher-configuration.md#logging)的 `loglevel`。

* [检查您的配置](/help/using/dispatcher-configuration.md)：

   * 您是否有多个 Dispatcher？

      * 您是否已确定哪个 Dispatcher 正在处理您所调查的网站/页面？
   * 您是否已实施过滤器？

      * 以下各项是否会对您调查的内容产生影响？


## IIS 诊断工具 {#iis-diagnostic-tools}

IIS 提供了各种跟踪工具，具体取决于实际版本：

* IIS 6 - 可以下载和配置 IIS 诊断工具
* IIS 7 - 跟踪已完全集成

它们可以帮助您监控活动。

## IIS 和 404 未找到 {#iis-and-not-found}

在使用 IIS 时，您可能会在各种场景中遇到返回 `404 Not Found` 的情况。如果是这样，请参阅以下知识库文章。

* [IIS 6/7：POST 方法返回 404](https://helpx.adobe.com/cn/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS 6：对包含基本路径 `/bin` 的 URL 的请求返回 `404 Not Found`](https://helpx.adobe.com/cn/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

您还应检查 Dispatcher 缓存根目录和 IIS 文档根目录是否已设为同一目录。

## 删除工作流模型时出现问题 {#problems-deleting-workflow-models}

**症状**

在通过 Dispatcher 访问 AEM 创作实例的情况下尝试删除工作流模型时出现问题。

**重现步骤：**

1. 登录您的创作实例（确认请求正在通过 Dispatcher 路由）。
1. 创建一个新的工作流；例如，将“标题”设置为 workflowToDelete。
1. 确认已成功创建该工作流。
1. 选择并右键单击该工作流，然后单击&#x200B;**“删除”**。

1. 单击&#x200B;**“是”**&#x200B;以确认。
1. 一个错误消息框随即出现，其中显示：\
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

这描述了 Dispatcher 如何与 Apache Web Server 中的 `mod_dir` 进行交互，因为这可能会产生各种意外的效果：

### Apache 1.3 {#apache}

在 Apache 1.3 中，`mod_dir` 处理 URL 映射到文件系统中的目录的每个请求。

它会：

* 将请求重定向到现有 `index.html` 文件
* 生成目录列表

Dispatcher 一经启用，就会将自身注册为内容类型 `httpd/unix-directory` 的处理程序来处理此类请求。

### Apache 2.x {#apache-x}

在 Apache 2.x 中，情况则有所不同。模块可以处理请求的不同阶段，例如 URL 修复。`mod_dir` 通过将请求（当 URL 映射到目录时）重定向到追加了 `/` 的 URL 来处理此阶段。

Dispatcher 不会拦截 `mod_dir` 修复，而是完全处理对重定向的 URL（已追加 `/`）的请求。如果远程服务器（例如 AEM）处理对 `/a_path` 的请求的方式与处理对 `/a_path/` 的请求（当 `/a_path` 映射到现有目录时）的方式不同，这可能会引发问题。

如果出现这种情况，您必须：

* 为由 Dispatcher 处理的 `Directory` 或 `Location` 子树禁用 `mod_dir`

* 使用 `DirectorySlash Off` 配置 `mod_dir` 而不追加 `/`
