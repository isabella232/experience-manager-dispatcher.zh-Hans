---
title: 调度程序热点问题
seo-title: AEM Dispatcher的主要问题
description: AEM Dispatcher的主要问题
seo-description: AdobeAEM Dispatcher的主要问题
translation-type: tm+mt
source-git-commit: eed7c3f77ec64f2e7c5cfff070ef96108886a059
workflow-type: tm+mt
source-wordcount: '1644'
ht-degree: 13%

---


# AEM Dispatcher常见问题解答

![配置 Dispatcher](assets/CQDispatcher_workflow_v2.png)

## 简介

### 什么是调度程序？

调度程序是Adobe Experience Manager的缓存和／或负载平衡工具，可帮助实现快速、动态的Web创作环境。 对于缓存，调度程序作为HTTP服务器（如Apache）的一部分工作，目的是尽可能多地存储（或“缓存”）静态网站内容，并尽可能少地访问网站的布局引擎。 在负载平衡角色中，调度程序跨不同的AEM实例（呈现）分发用户请求（负载）。

对于缓存，调度程序模块使用Web服务器提供静态内容的能力。 调度程序将缓存的文档放在Web服务器的文档根中。

### 调度程序如何执行缓存？

调度程序使用Web服务器提供静态内容的能力。 调度程序将缓存的文档存储在Web服务器的文档根中。 Dispatcher 有两种主要方法可用于在对网站进行更改后更新缓存内容。

* **内容更新**&#x200B;可删除已更改的页面以及与其直接关联的文件。
* **自动失效**&#x200B;可在更新后自动使可能已过期的这部分缓存失效。例如，它可以有效地将相关页面标记为过时，而不会删除任何内容。

### 负载平衡有哪些好处？

负载平衡在多个AEM实例中分发用户请求（负载）。以下列表描述了负载平衡的优点：

* **增强的处理能力**:实际上，这意味着调度程序在AEM的多个实例之间共享文档请求。由于每个实例处理的文档更少，因此响应时间更短。 Dispatcher 保留每个文档类别的内部统计信息，以便能够估计负载并高效分发查询。
* **增加的故障保护范围**:如果调度程序未收到来自某个实例的响应，它将自动将请求中继到另一个实例中的一个。因此，如果一个实例变得不可用，唯一的结果就是站点速度减慢，这与计算能力损失成正比。

>[!NOTE]
>
>有关详细信息，请参阅[“调度程序概述”页](dispatcher.md)

## 安装和配置

### 从哪里下载Dispatcher模块？

您可以从[“调度程序发行说明”](release-notes.md)页面下载最新的调度程序模块。

### 如何安装Dispatcher模块？

请参阅[安装Dispatcher](dispatcher-install.md)页

### 如何配置调度程序模块？

请参阅[配置Dispatcher](dispatcher-configuration.md)页。

### 如何为创作实例配置调度程序？

有关详细步骤，请参阅[将Dispatcher与作者实例一起使用](dispatcher.md#using-a-dispatcher-with-an-author-server)。

### 如何配置具有多个域的调度程序？

您可以配置具有多个域的CQ调度程序，前提是这些域满足以下条件：

* 这两个域的Web内容都存储在单个AEM存储库中
* 对于每个域，调度程序缓存中的文件可以单独失效

有关更多详细信息，请阅读[使用带多个域的调度程序](dispatcher-domains.md)。

### 如何配置调度程序，使用户的所有请求都路由到同一个发布实例？

您可以使用[粘性连接](dispatcher-configuration.md#identifying-a-sticky-connection-folder-stickyconnectionsfor)功能，确保在AEM的同一实例上处理用户的所有文档。 如果您使用个性化的页面和会话数据，此功能很重要。 数据存储在实例中。 因此，来自同一用户的后续请求必须返回到该实例，否则数据将丢失。

由于粘性连接限制了调度程序优化请求的能力，因此您应仅在必要时使用此方法。 您可以指定包含“粘滞”文档的文件夹，从而确保在同一实例上为用户处理该文件夹中的所有文档。

### 我是否可以同时使用粘性连接和缓存？

对于使用粘滞连接的大多数页面，应关闭缓存。 否则，无论会话内容如何，都将向所有用户显示页面的同一实例。

对于某些应用程序，可以同时使用粘性连接和缓存。 例如，如果显示将数据写入会话的表单，则可以同时使用粘性连接和缓存。

### 调度程序和AEM发布实例是否可以驻留在同一物理计算机上？

是的，如果机器足够强大。 但是，建议您在不同的计算机上设置调度程序和AEM发布实例。

通常，Publish实例驻留在防火墙内，调度程序驻留在DMZ中。 如果您决定将Publish实例和Dispatcher都放在同一台物理计算机上，请确保防火墙设置禁止从外部网络直接访问Publish实例。

### 我是否只能缓存具有特定扩展名的文件？

是. 例如，如果只要缓存GIF文件，请在dispatcher.any配置文件的缓存部分中指定*.gif。

### 如何从缓存中删除文件？

可以使用HTTP请求从缓存中删除文件。 收到HTTP请求后，Dispatcher将从缓存中删除文件。 调度程序仅在收到页面的客户端请求时才再次缓存文件。 以这种方式删除缓存的文件对于不太可能同时收到同一页面请求的网站是合适的。

HTTP请求的语法如下：

```
POST /dispatcher/invalidate.cache HTTP/1.1
CQ-Action: Activate
CQ-Handle: path-pattern
Content-Length: 0
```

调度程序将删除名称与CQ-Handle头值匹配的缓存文件和文件夹。 例如，`/content/geomtrixx-outdoors/en`的CQ-Handle匹配以下项：

在geometrixx-outdoors目录中名为en的所有文件（任何文件扩展名的）
en目录下名为`_jcr_content`的任何目录（如果存在，该目录包含页子节点的缓存呈现）
仅当`CQ-Action`为`Delete`或`Deactivate`时，才删除目录en。

有关本主题的更多详细信息，请参阅[手动使调度程序缓存失效](page-invalidate.md)。

### 如何实现对权限敏感的缓存？

请参阅[缓存安全内容](permissions-cache.md)页。

### 如何保护调度程序和CQ实例之间的通信？

请参阅[调度程序安全清单](security-checklist.md)和[AEM安全清单](https://helpx.adobe.com/experience-manager/6-4/sites/administering/using/security-checklist.html)页。

### 调度程序问题`jcr:content`已更改为`jcr%3acontent`

**问题**:最近，在调度程序级别，我们遇到了一个问题，其中一个ajax调用从CQ存储库中获取某 `jcr:content` 些数据，并经过编码以 `jcr%3acontent` 导致错误的结果集。

**答案**:请使用 `ResourceResolver.map()` 方法获取要使用／发出的“友好”URL，并从中获取请求，同时解决Dispatcher的缓存问题。map()方法将`:`冒号编码为下划线，resolve()方法将其解码回SLING JCR可读格式。您需要使用map()方法生成Ajax调用中使用的URL。

进一步阅读：[https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling](https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling)

## 刷新调度程序

### 如何在发布实例上配置调度程序刷新代理？

请参阅[复制](https://helpx.adobe.com/content/help/en/experience-manager/6-4/sites/deploying/using/replication.html#ConfiguringyourReplicationAgents)页。

### 如何诊断调度程序刷新问题？

[请参阅此解答以](https://helpx.adobe.com/content/help/en/experience-manager/kb/troubleshooting-dispatcher-flushing-issues.html) 下问题的疑难解答文章：

* 如何调试未在调度程序缓存中保存内容的情况？
* 如何调试缓存文件未更新的情况？
* 如何调试与Dispatcher刷新无关的情况？

如果删除操作导致调度程序刷新，请[在Sensei Martin](https://mkalugin-cq.blogspot.in/2012/04/i-have-been-working-on-following.html)撰写的此社区博客文章中使用解决方法。

### 如何从调度程序缓存刷新DAM资产？

您可以使用“链式复制”功能。  启用此功能后，当从作者处收到复制时，调度程序刷新代理会发送刷新请求。

要启用它，请执行以下操作：

1. [按照以下步骤](page-invalidate.md#invalidating-dispatcher-cache-from-a-publishing-instance) 在发布时创建刷新代理
1. 转到这些代理的每个配置，在&#x200B;**触发器**&#x200B;选项卡上选中&#x200B;**接收时**&#x200B;框。

## 杂项

调度程序如何确定文档是否为最新？
要确定文档是否为最新，调度程序将执行以下操作：

它检查文档是否遵循自动失效机制。如果不遵循，则该文档是最新状态。如果该文档配置为自动失效，则 Dispatcher 检查它比最后一次可用更改旧还是新。如果较旧，则 Dispatcher 从 AEM 实例请求当前版本，并替换缓存中的版本。

### 调度程序如何返回文档?

您可以使用[调度程序配置](dispatcher-configuration.md)文件`dispatcher.any`定义调度程序是否缓存文档。 Dispatcher 根据可缓存文档列表检查请求。如果文档不在此列表中，则 Dispatcher 从 AEM 实例中请求该文档。

`/rules`属性控制根据文档路径缓存哪些文档。 无论`/rules`属性如何，在以下情况下，调度程序都不会缓存文档:

* 如果请求URI包含问号`(?)`。
* 这通常指示动态页面，如无需缓存的搜索结果。
* 缺失文件扩展名。
* Web 服务器需要扩展名来确定文档类型（比如 MIME 类型）。
* 设置了身份验证标头（此项可进行配置）
* 如果AEM实例使用以下标题做出响应：
   * 无缓存
   * 无商店
   * must-revalidate

调度程序将缓存的文件存储在Web服务器上，就像它们是静态网站的一部分一样。 如果用户请求缓存的文档，调度程序将检查文档是否存在于Web服务器的文件系统中。 如果是，调度程序将返回文档。 否则，调度程序从AEM实例请求文档。

>[!NOTE]
>
>GET 或 HEAD（针对 HTTP 标头）方法可由 Dispatcher 缓存。有关响应头缓存的其他信息，请参阅[缓存HTTP响应头](dispatcher-configuration.md#caching-http-response-headers)部分。

### 我是否可以在一个设置中实施多个调度程序？

是. 在这种情况下，请确保调度程序都可以直接访问AEM网站。 调度程序无法处理来自其他调度程序的请求。

