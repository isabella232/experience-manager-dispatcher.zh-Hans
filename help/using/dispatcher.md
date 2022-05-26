---
title: Dispatcher 概述
seo-title: Adobe AEM Dispatcher Overview
description: 了解如何使用 Dispatcher 改进 AEM Cloud Services 的安全性、缓存等。
seo-description: This article provides a general overview of Adobe Experience Manager Dispatcher.
uuid: 71766f86-5e91-446b-a078-061b179d090d
pageversionid: 1193211344162
topic-tags: dispatcher
content-type: reference
discoiquuid: 1d449ee2-4cdd-4b7a-8b4e-7e6fc0a1d7ee
exl-id: c9266683-6890-4359-96db-054b7e856dd0
source-git-commit: 85a7f38ecb5f5fe620ff51a7cdbe29f546559288
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---

# Dispatcher 概述 {#dispatcher-overview}

>[!NOTE]
>
>Dispatcher 版本独立于 AEM。您可能是在单击以前版本的 AEM 文档中嵌入的 Dispatcher 文档链接后重定向到此页面。

Dispatcher 是 Adobe Experience Manager 的缓存和/或负载平衡工具，可与企业级 Web 服务器结合使用。

Dispatcher 的部署流程与所选的 Web 服务器和操作系统平台无关：

1. 了解 Dispatcher（此页面）。另请参阅[有关 Dispatcher 的常见问题解答](https://helpx.adobe.com/cn/experience-manager/using/dispatcher-faq.html)。
1. 按照 Web 服务器文档安装[支持的 Web 服务器](https://helpx.adobe.com/cn/experience-manager/6-5/sites/deploying/using/technical-requirements.html)。
1. 在 Web 服务器上[安装 Dispatcher 模块](dispatcher-install.md)，并相应地配置 Web 服务器。
1. [配置 Dispatcher](dispatcher-configuration.md)（dispatcher.any 文件）。
1. [配置 AEM](page-invalidate.md)，以便内容更新后，使缓存失效。

>[!NOTE]
>
>要更好地了解如何将 Dispatcher 与 AEM 配合使用，请执行以下操作：
>
>* 参阅 [2017 年 7 月版 AEM 社区专家疑难解答](https://bit.ly/ATACE0717)。
>* 访问[此存储库](https://github.com/adobe/aem-dispatcher-experiments)。它包含一系列“家用版”实验室格式的实验。



根据需要使用以下信息：

* [Dispatcher 安全核对清单](security-checklist.md)
* [Dispatcher 知识库](https://helpx.adobe.com/cn/cq/kb/index/dispatcher.html)
* [优化网站缓存性能](https://helpx.adobe.com/cn/experience-manager/6-4/sites/deploying/using/configuring-performance.html)
* [在多个域中使用 Dispatcher](dispatcher-domains.md)
* [将 SSL 与 Dispatcher 结合使用](dispatcher-ssl.md)
* [实施权限敏感型缓存](permissions-cache.md)
* [Dispatcher 问题疑难解答](dispatcher-troubleshooting.md)
* [Dispatcher 常见问题解答](dispatcher-faq.md)

>[!NOTE]
>
>**Dispatcher 最常见的用法**&#x200B;是缓存来自 AEM **发布实例**&#x200B;的响应，从而提高面向外部发布的网站的响应速度和安全性。大部分讨论主要介绍这种情况。
>
>但是，Dispatcher 也可以用于提高&#x200B;**创作实例**&#x200B;的响应速度，尤其是在有大量用户编辑和更新网站的情况下。有关这种情况的详细信息，请参阅下面的[将 Dispatcher 与作者服务器一起使用](#using-a-dispatcher-with-an-author-server)。

## 为何使用 Dispatcher 来实施缓存？ {#why-use-dispatcher-to-implement-caching}

发布 Web 有两种基本方法：

* **静态 Web 服务器**：例如 Apache 或 IIS，非常简单但速度很快。
* **内容管理服务器**：提供动态、实时、智能的内容，但需要更多的计算时间和其他资源。

Dispatcher 可帮助实现既快速又动态的环境。它在静态 HTML 服务器（比如 Apache）中使用的目的是：

* 以静态网站的形式尽可能多地存储（“缓存”）站点内容
* 尽可能少地访问布局引擎。

这意味着：

* 处理&#x200B;**静态内容**&#x200B;的速度和容易度与在静态 Web 服务器上的完全相同；*此外，您还可以使用为静态 Web 服务器提供的管理和安全工具*。

* 根据需要生成&#x200B;**动态内容**，完全没有必要再减慢系统速度。

Dispatcher 包含根据动态站点内容生成和更新静态 HTML 的机制。您可以详细指定将哪些文档存储为静态文件，哪些文档始终通过动态方式生成。

本节将阐明其中运用的原理。

### 静态 Web 服务器 {#static-web-server}

![](assets/chlimage_1-3.png)

静态 Web 服务器（如 Apache 或 IIS），负责向网站的访客提供静态 HTML 文件。静态页面只创建一次，因此针对每个请求都将交付相同的内容。

这个流程非常简单，因此相当高效。如果访客请求一个文件（例如 HTML 页面），通常将直接从内存中获取该文件，在最坏的情况下，也可以从本地驱动器读取该文件。静态 Web 服务器问世已有相当长的一段时间，因此有各种各样用于管理和安全管理的工具，而且它们能够很好地与网络基础结构集成。

### 内容管理服务器 {#content-management-servers}

![](assets/chlimage_1-4.png)

如果您使用内容管理服务器（如 AEM），则由高级布局引擎处理访客的请求。该引擎从存储库中读取内容，包括样式、格式和访问权限，并将内容转换为根据访客需求和权限定制的文档。

这使您能够创建更丰富的动态内容，从而提高网站的灵活性和功能性。但是，布局引擎需要比静态服务器更大的处理能力，因此，如果许多访客同时使用系统，则此设置的速度可能会减慢。

## Dispatcher 如何执行缓存 {#how-dispatcher-performs-caching}

![](assets/chlimage_1-5.png)

**缓存目录**：对于执行缓存，Dispatcher 模块利用 Web 服务器的功能来提供静态内容。Dispatcher 将缓存文档放在 Web 服务器的文档根目录下。

>[!NOTE]
>
>如果没有配置 HTTP 标头缓存，则 Dispatcher 仅存储页面的 HTML 代码 - 它不会存储 HTTP 标头。如果您在网站内使用不同的编码，这可能是个问题，因为这些编码可能会丢失。要启用 HTTP 标头缓存，请参阅[配置 Dispatcher 缓存。](https://helpx.adobe.com/cn/experience-manager/dispatcher/using/dispatcher-configuration.html)

>[!NOTE]
>
>将 Web 服务器的文档根目录放置在网络连接存储 (NAS) 上，会导致性能下降。此外，如果在多个 Web 服务器之间共享位于 NAS 上的文档根目录，则执行复制操作时可能会出现间歇性锁定。

>[!NOTE]
>
>Dispatcher 使用与请求的 URL 相同的结构存储缓存文档。
>
>文件名长度可能存在操作系统级别的限制；例如，您的 URL 中带有许多选择器。

### 缓存方法

Dispatcher 有两种主要方法可用于在对网站进行更改后更新缓存内容。

* **内容更新**&#x200B;可删除已更改的页面以及与其直接关联的文件。
* **自动失效**&#x200B;可在更新后自动使可能已过期的这部分缓存失效。也就是说，它可以有效地将相关页面标记为已过期，而不会删除任何内容。

### 内容更新

在内容更新中，有一个或多个 AEM 文档发生了变更。AEM 向 Dispatcher 发送联合请求，以相应地更新缓存：

1. 它从缓存中删除修改过的文件。
1. 它从缓存中删除以相同句柄开头的所有文件。例如，如果更新了 /en/index.html 文件，则以 /en/index. 开头的所有文件都会被删除。这种机制允许您设计有效的缓存站点，尤其是对于图片导航。
1. 它会处理&#x200B;**&#x200B;所谓的 **statfile**；这会更新 statfile 的时间戳记，指出最后一次更改日期。

应注意以下几点：

* 内容更新通常与创作系统结合使用，后者“知道”必须替换哪些内容。
* 会删除受内容更新影响的文件，但不会立即替换。下次再请求此类文件时，Dispatcher 将从 AEM 实例获取新文件，并将其放置在缓存中，从而覆盖旧内容。
* 通常情况下，自动生成的图片（包含来自页面的文本）将存储在以相同句柄开头的图片文件中 - 从而确保存在关联以便于删除。例如，您可以将 mypage.html 页面的标题文本作为 mypage.titlePicture.gif 图片存储在相同的文件夹中。这样每次更新页面后就会自动从缓存中删除图片，您就可以确保图片始终反映页面的最新版本。
* 您可能有多个 statfile，例如每个语言文件夹一个。如果页面已更新，AEM 将查找包含 statfile 的下一个父文件夹，然后处理&#x200B;**&#x200B;该文件。

### 自动失效

自动失效可自动使部分缓存失效 - 不会实际删除任何文件。在每次内容更新时，都会处理所谓的 statfile，因此其时间戳记可以反映最新的内容更新日期。

Dispatcher 有一个遵循自动失效机制的文件列表。当请求该列表中的文档时，Dispatcher 会将缓存文档的日期与 statfile 的时间戳记进行比较：

* 如果缓存文档的日期较新，则 Dispatcher 将返回缓存文档。
* 如果日期较旧，则 Dispatcher 将从 AEM 实例中检索最新版本。

同样，应当注意这几点：

* 当内部关系比较复杂（例如 HTML 页面）时，通常使用自动失效。这些页面包含链接和导航条目，因此通常在内容更新后必须进行更新。如果您有自动生成的 PDF 或图片文件，则也可以选择对这些文件使用自动失效方法。
* 除了处理 statfile，自动失效不需要 Dispatcher 在更新时执行任何操作。但是，处理 statfile 会自动使缓存内容过时，从而不会将其从缓存中实际删除。

## Dispatcher 如何返回文档 {#how-dispatcher-returns-documents}

![](assets/chlimage_1-6.png)

### 确定文档是否实施了缓存

您可以[在配置文件中定义 Dispatcher 缓存的文档](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)。Dispatcher 根据可缓存文档列表检查请求。如果文档不在此列表中，则 Dispatcher 从 AEM 实例中请求该文档。

在以下情况下，Dispatcher *始终*&#x200B;直接从 AEM 实例请求文档：

* 请求 URI 包含问号“?”。这通常表示无需缓存的动态页面，例如搜索结果。
* 缺失文件扩展名。Web 服务器需要扩展名来确定文档类型（比如 MIME 类型）。
* 设置了身份验证标头（此项可进行配置）

>[!NOTE]
>
>GET 或 HEAD（针对 HTTP 标头）方法可由 Dispatcher 缓存。有关响应标头缓存的其他信息，请参阅[缓存 HTTP 响应标头](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)部分。

### 确定文档是否已缓存

Dispatcher 将缓存文件存储在 Web 服务器上，当做静态网站的一部分。如果用户请求一个可缓存的文档，则 Dispatcher 检查该文档是否存在于 Web 服务器的文件系统中：

* 如果文档已缓存，则 Dispatcher 返回该文件。
* 如果未缓存，则 Dispatcher 从 AEM 实例请求该文档。

### 确定文档是否为最新状态

为确定文档是否为最新状态，Dispatcher 将执行两个步骤：

1. 它检查文档是否遵循自动失效机制。如果不遵循，则该文档是最新状态。
1. 如果该文档配置为自动失效，则 Dispatcher 检查它比最后一次可用更改旧还是新。如果较旧，则 Dispatcher 从 AEM 实例请求当前版本，并替换缓存中的版本。

>[!NOTE]
>
>没有配置为&#x200B;**自动失效**&#x200B;的文档仍保留在缓存中，直到被实际删除；例如，通过网站上的内容更新。

## 负载平衡的好处 {#the-benefits-of-load-balancing}

负载平衡就是在多个 AEM 实例间分发网站的计算负载。

![](assets/chlimage_1-7.png)

您将获得：

* **更大的处理能力**
在实践中，这意味着 Dispatcher 在多个 AEM 实例之间共享文档请求。由于现在每个实例处理的文档数量减少，您的响应速度将会加快。Dispatcher 保留每个文档类别的内部统计信息，以便能够估计负载并高效分发查询。

* **更大的故障保护范围**
如果 Dispatcher 未收到实例的响应，它会自动将请求中继到另一个实例。因此，如果一个实例变得不可用，唯一的影响就是站点速度减慢，这与计算能力损失成正比。但是，所有服务都将继续。

* 您还可以在同一个静态 Web 服务器上管理不同的网站。

>[!NOTE]
>
>负载平衡可以有效分布负载，而缓存有助于减少负载。因此，在设置负载平衡之前，请尝试优化缓存并降低整体负载。良好的缓存机制可能会提高负载平衡器的性能，或者弃用不必要的负载平衡。

>[!CAUTION]
>
>虽然单个 Dispatcher 通常就能够使可用发布实例的容量饱和，但对于某些极少数的应用程序而言，最好在两个 Dispatcher 实例之间额外实施负载平衡。配置多个 Dispatcher 需慎重考虑，因为额外的 Dispatcher 将增加可用发布实例的负载，而且容易降低大多数应用程序的性能。

## Dispatcher 如何执行负载平衡 {#how-the-dispatcher-performs-load-balancing}

### 性能统计数据

Dispatcher 保留有关 AEM 每个实例处理文档的速度的内部统计信息。Dispatcher 将根据这些数据估算哪个实例在响应请求时将提供最快的响应速度，以便它在该实例上预留所需的计算时间。

不同类型请求的平均完成时间可能各不相同，因此 Dispatcher 允许指定文档类别。然后在估计计算时间时考虑这些类别。例如，您可以将 HTML 页面和图像区分开来，因为它们的标准响应时间可能相差很多。

如果您使用复杂的搜索功能，则可以为搜索查询创建新的类别。这有助于 Dispatcher 将搜索查询发送到响应最快的实例。这样可以防止速度较慢的实例在收到多个“成本较高的”搜索查询时发生停滞，而其他实例收到“成本较低的”请求。

### 个性化的内容（粘性连接）

粘性连接可确保同一个用户的文档全部在 AEM 的同一个实例上撰写。如果您使用个性化的页面和会话数据，这一点很重要。数据存储在该实例上，则同一用户发出的后续请求必须返回到该实例，否则数据就会丢失。

由于粘性连接会限制 Dispatcher 优化请求的能力，因此应仅在需要时使用。您可以指定包含“粘性”文档的文件夹，从而确保该文件夹中每个用户的所有文档都在同一个实例上撰写。

>[!NOTE]
>
>对于使用粘性连接的大多数页面，必须关掉缓存 - 否则无论会话内容是什么，所有用户都将看到相同的页面内容。
>
>对于&#x200B;*少数*&#x200B;应用程序，可以同时使用粘性连接和缓存；例如，显示将数据写入会话的表单。

## 使用多个 Dispatcher {#using-multiple-dispatchers}

在复杂设置中，您可以使用多个 Dispatcher。例如，您可以使用：

* 一个 Dispatcher 用于在内联网上发布网站
* 第二个 Dispatcher，通过不同的地址和不同的安全设置，在内联网上发布相同的内容。

在这种情况下，请确保每个请求只通过一个 Dispatcher。一个 Dispatcher 不能处理来自另一个 Dispatcher 的请求。因此，请确保两个 Dispatcher 都能直接访问 AEM 网站。

## 将 Dispatcher 与 CDN 结合使用 {#using-dispatcher-with-a-cdn}

内容交付网络 (CDN)（如 Akamai Edge Delivery 或 Amazon Cloud Front）从距离最终用户较近的站点交付内容。这样，它可以

* 加快最终用户响应速度
* 减少服务器的负载

作为 HTTP 基础架构组件，CDN 的工作方式与 Dispatcher 非常相似：当 CDN 节点收到请求时，如果可能（缓存中提供该资源，而且是有效的）的话，它将从其缓存响应该请求。否则，它将连接下一个距离最近的服务器，以检索资源并缓存下来，以备响应后续请求（如果适用）。

“下一个距离最近的服务器”取决于您的具体设置。例如，在 Akamai 设置中，请求可以遵循以下路径：

* Akamai 边缘节点
* Akamai Midgres 图层
* 您的防火墙
* 您的负载平衡器
* Dispatcher
* AEM

在大多数情况下，Dispatcher 就是下一个服务器，它可以从缓存中提供文档，并影响返回到 CDN 服务器的响应标头。

## 控制 CDN 缓存 {#controlling-a-cdn-cache}

有许多种方法可以控制 CDN 从 Dispatcher 中缓存以重新获取资源的时间。

1. 显式配置\
   根据 mime 类型、扩展名、请求类型等，配置特定资源在 CDN 缓存中的保留时间。

1. 到期和缓存控制标头\
   如果由上游服务器发送，则大多数 CDN 都将采用 `Expires:` 和 `Cache-Control:` HTTP 标头。例如，可使用 [mod_expires](https://httpd.apache.org/docs/2.4/mod/mod_expires.html) Apache 模块实现此目的。

1. 手动失效\
   CDN 允许通过 Web 界面从缓存中删除资源。
1. 基于 API 的失效\
   大多数 CDN 还提供允许从缓存中删除资源的 REST 和/或 SOAP API。

在典型的 AEM 设置中，按扩展名和/或路径进行的配置（可通过以上第 1 点和第 2 点实现），使为经常使用但不经常变更的资源（如设计图像和客户端库）设置合理缓存期限成为了可能。在部署新版本时，通常需要手动进行失效操作。

如果将此方法用于缓存受管内容，则意味着仅在配置的缓存期限到期且再次从 Dispatcher 中获取文档后，内容变更才对最终用户可见。

为实现更细粒度的控制，基于 API 的失效允许您在 Dispatcher 缓存失效时使 CDN 的缓存失效。基于 CDN API，您可以实施自己的 [ContentBuilder](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/day/cq/replication/ContentBuilder.html) 和 [TransportHandler](https://helpx.adobe.com/cn/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/day/cq/replication/TransportHandler.html)（如果 API 不是基于 REST 的类型）并设置复制代理，它将利用这两个工具来使 CDN 的缓存失效。

>[!NOTE]
>
>另请参阅 [AEM (CQ) Dispatcher 安全性和 CDN+浏览器缓存](https://www.slideshare.net/andrewmkhoury/dispatcher-caching-aemgemspart2jan2015)和有关 [Dispatcher 缓存](https://docs.adobe.com/content/ddc/en/gems/dispatcher-caching---new-features-and-optimizations.html)的录制演讲。

## 将 Dispatcher 与创作服务器一起使用 {#using-a-dispatcher-with-an-author-server}

>[!CAUTION]
>
>如果您正在将 [AEM 与 Touch UI](https://helpx.adobe.com/cn/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html) 一起使用，则&#x200B;**不**&#x200B;应缓存创作实例内容。如果为创作实例启用了缓存，则需要禁用缓存并删除缓存目录的内容。要禁用缓存，应编辑 `author_dispatcher.any` 文件并修改 `/cache` 部分的 `/rule` 属性，如下所示：

```xml
/rules
{
/0000
{ /type "deny" /glob "*"}
}
```

Dispatcher 可在创作实例之前使用以提高创作性能。要配置创作 Dispatcher，请执行以下操作：

1. 在 Web 服务器中安装 Dispatcher（这可以是 Apache 或 IIS Web 服务器，请参阅[安装 Dispatcher](dispatcher-install.md)）。
1. 您可能希望针对正在工作的 AEM 发布实例测试新安装的 Dispatcher，以确保已实现基准正确安装。
1. 现在，确保 Dispatcher 能够通过 TCP/IP 连接到您的创作实例。
1. 将示例的 dispatcher.any 文件替换为随 [Dispatcher 下载](release-notes.md#downloads)提供的 author_dispatcher.any 文件。
1. 在文本编辑器中打开 `author_dispatcher.any`，并进行以下更改：

   1. 将 `/renders` 部分的 `/hostname` 和 `/port` 更改为指向创作实例。
   1. 将 `/cache` 部分的 `/docroot` 更改为指向缓存目录。如果您将 [AEM 与 Touch UI](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html) 一起使用，请查看上面的警告。
   1. 保存更改。

1. 删除您在前面配置的 `/cache` > `/docroot` 目录中的所有现有文件。
1. 重新启动 Web 服务器。

>[!NOTE]
>
>请注意，如果使用提供的 `author_dispatcher.any` 配置，当您安装影响 `/libs` 或 `/apps` 下任何内容的 CQ5 功能包、热修复或应用代码包时，必须删除 Dispatcher 缓存中这些目录下的缓存文件，以确保下次再请求时，能够获取新更新的文件，而不是旧的缓存文件。

>[!CAUTION]
>
>如果您使用之前配置的创作 Dispatcher 并启用 *Dispatcher 刷新代理*，请执行以下操作：

1. 在您的 AEM 创作实例上删除或禁用&#x200B;**创作 Dispatcher 的**&#x200B;刷新代理。
1. 按照以上新的指示说明重新执行创作 Dispatcher 配置。

<!--
[Author Dispatcher configuration file (Dispatcher 4.1.2 or later)](assets/author_dispatchernew.any)
-->
<!--[!NOTE]
>
>A related knowledge base article can be found here:  
>[How to configure the dispatcher in front of an authoring environment](https://helpx.adobe.com/cq/kb/HowToConfigureDispatcherForAuthoringEnvironment.html)
-->
