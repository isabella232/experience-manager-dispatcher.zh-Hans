---
title: 调度程序概述
seo-title: Adobe AEM Dispatcher概述
description: 本文提供Dispatcher的一般概述。
seo-description: 本文提供Adobe Experience Manager Dispatcher的概述。
uuid: 71766f86-5e91-446b-a078-061b179d090d
pageversionid: '1193211344162'
topic-tags: 调度程序
content-type: 引用
discoiquuid: 1d449ee2-4cdd-4b7a-8b4e-7e6fc0a1d7ee
translation-type: tm+mt
source-git-commit: de6a513baf3e6b1a1463a442fa840e59f2196e8e

---


# 调度程序概述 {#dispatcher-overview}

>[!NOTE]
>
>调度程序版本独立于AEM。 如果您遵循了指向Dispatcher文档的链接（该链接嵌入在AEM先前版本的文档中），则您可能已被重定向到此页。

调度程序是Adobe Experience Manager的缓存和／或负载平衡工具。 使用AEM的Dispatcher还有助于保护AEM服务器免受攻击。 因此，您可以通过将Dispatcher与企业级Web服务器结合使用来提高AEM实例的安全性。

部署Dispatcher的过程独立于所选的Web服务器和操作系统平台：

1. 了解Dispatcher（此页）。 此外，请参阅有 [关调度程序的常见问题解答](https://helpx.adobe.com/experience-manager/using/dispatcher-faq.html)。
1. 根据Web服 [务器文档](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html) ，安装支持的Web服务器。

1. [在Web服务器上安装Dispatcher模块](dispatcher-install.md) ，并相应地配置Web服务器。
1. [配置Dispatcher](dispatcher-configuration.md) （dispatcher.any文件）。

1. [配置AEM](page-invalidate.md) ，以便内容更新使缓存失效。

>[!NOTE]
>
>要更好地了解Dispatcher如何与AEM配合使用，请参 [阅2017年7月向AEM社区专家咨询](https://bit.ly/ATACE0717)。

根据需要使用以下信息：

* [调度程序安全核对清单](security-checklist.md)
* [调度程序知识库](https://helpx.adobe.com/cq/kb/index/dispatcher.html)
* [为缓存性能优化网站](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html)
* [将调度程序与多个域一起使用](dispatcher-domains.md)
* [将SSL与Dispatcher结合使用](dispatcher-ssl.md)
* [实施权限敏感型缓存](permissions-cache.md)
* [调度程序问题疑难解答](dispatcher-troubleshooting.md)
* [调度程序常见问题解答](dispatcher-faq.md)

>[!NOTE]
>
>**Dispatcher的最常见用途是缓存来自AEM发布实例的响** 应 ****，从而提高面向外部的发布网站的响应性和安全性。 大部分讨论都集中在此案件上。
>
>但是，Dispatcher还可用于提高作者实例的响应度 ****，尤其是当有大量用户编辑和更新网站时。 有关此情况的特定详细信息，请 [参阅下面的将Dispatcher与作者服务器一起使用](#using-a-dispatcher-with-an-author-server)。

## 为何使用Dispatcher实现缓存？ {#why-use-dispatcher-to-implement-caching}

Web发布有两种基本方法：

* **静态Web服务器**:如Apache或IIS，非常简单，但速度很快。
* **内容管理服务器**:它提供动态、实时、智能的内容，但需要更多的计算时间和其他资源。

Dispatcher有助于实现快速且动态的环境。 它作为静态HTML服务器（如Apache）的一部分工作，其目的是：

* 以静态网站的形式存储（或“缓存”）尽可能多的站点内容
* 尽可能少地访问布局引擎。

这意味着：

* **静态内容** ，处理速度和简单性与静态Web服务器完全相同；此外，*您还可以使用静态Web服务器提供的管理和安全工具*。

* **根据需要** ，生成动态内容，而不会使系统放慢速度，而非绝对必要。

调度程序包含基于动态站点内容生成和更新静态HTML的机制。 您可以详细指定将哪些文档存储为静态文件以及始终动态生成哪些文档。

本节说明了其中的原则。

### 静态Web服务器 {#static-web-server}

![](assets/chlimage_1-3.png)

静态Web服务器（如Apache或IIS）向网站的访客提供静态HTML文件。 静态页面只创建一次，因此每个请求都会提供相同的内容。

该方法非常简单，因此非常有效。 如果访客请求文件（例如HTML页面），则通常直接从内存中获取该文件，往坏了说，该文件会从本地驱动器中读取。 静态Web服务器已经有相当长的一段时间了，因此有各种用于管理和安全管理的工具，并且它们与网络基础结构完美集成。

### 内容管理服务器 {#content-management-servers}

![](assets/chlimage_1-4.png)

如果您使用Content Management Server（如AEM），则高级布局引擎将处理访客的请求。 该引擎从存储库中读取内容，该存储库与样式、格式和访问权限相结合，将内容转换为适合访客需求和权限的文档。

这使您能够创建更丰富的动态内容，从而提高网站的灵活性和功能性。 但是，布局引擎比静态服务器需要更大的处理能力，因此，如果许多访问者使用系统，则此设置可能会降低。

## Dispatcher如何执行缓存 {#how-dispatcher-performs-caching}

![](assets/chlimage_1-5.png)

**缓存目录** “调度程序”模块用于缓存，它使用Web服务器提供静态内容的能力。 Dispatcher将缓存的文档放在Web服务器的文档根目录中。

>[!NOTE]
>
>当缺少HTTP头缓存配置时，调度程序仅存储页面的HTML代码——它不存储HTTP头。 如果您在网站中使用不同的编码，这可能是一个问题，因为这些编码可能会丢失。 要启用HTTP头缓存，请参阅 [配置调度程序缓存。](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)

>[!NOTE]
>
>在网络连接存储(NAS)上查找Web服务器的文档根目录会导致性能降级。 此外，当位于NAS上的文档根在多个Web服务器之间共享时，执行复制操作时可能会发生间歇性锁定。

>[!NOTE]
>
>调度程序将缓存文档存储在与所请求的URL相等的结构中。
>
>文件名的长度可能存在操作系统级别的限制；即，如果您的URL包含许多选择器。

### 缓存方法

调度程序有两种主要方法，用于在对网站进行更改时更新缓存内容。

* **内容更新** ，删除已更改的页面以及与其直接关联的文件。
* **自动失效** (Auto-Invalidation)会自动使更新后可能过期的缓存部分失效。 例如，它可以将相关页面有效地标记为过期，而不会删除任何内容。

### 内容更新

在内容更新中，一个或多个AEM文档发生更改。 AEM会向调度程序发送联合请求，调度程序会相应地更新缓存：

1. 它会从缓存中删除修改过的文件。
1. 它将从缓存中删除以相同句柄开始的所有文件。 例如，如果更新了文件/en/index.html，则所有以/cn/index开头的文件。 。 此机制允许您设计高效缓存的站点，特别是在图片导航方面。
1. 它 *触及* 所谓的 **statfile**;这将更新statfile的时间戳以指示上次更改的日期。

应当注意以下几点：

* 内容更新通常与创作系统一起使用，该创作系统“知道”必须替换的内容。
* 受内容更新影响的文件将被删除，但不会立即替换。 下次请求此类文件时，调度程序将从AEM实例中获取新文件并将其放入缓存中，从而覆盖旧内容。
* 通常，自动生成的包含页面文本的图片会以相同的句柄开始存储在图片文件中，从而确保存在用于删除的关联。 例如，您可以将页面mypage.html的标题文本存储为图片mypage.titlePicture.gif，位于同一文件夹中。 这样，图片在每次更新页面时都会从缓存中自动删除，因此您可以确保图片始终反映页面的当前版本。
* 您可能有多个statfile，例如每个语言文件夹一个。 如果页面已更新，AEM会查找包含statfile的下一个父文件夹，然后触 *及该* 文件。

### 自动失效

自动失效会自动使部分缓存失效——无需物理删除任何文件。 在每次内容更新时，都会触及所谓的statfile，因此其时间戳反映上次的内容更新。

调度程序有一个受自动失效影响的文件列表。 当请求来自该列表的文档时，调度程序将缓存文档的日期与statfile的时间戳进行比较：

* 如果缓存的文档较新，调度程序将返回它。
* 如果版本较旧，则调度程序将从AEM实例中检索当前版本。

同样，应该指出一些要点：

* 通常，当内部关系复杂（如HTML页面）时，会使用自动失效。 这些页面包含链接和导航条目，因此通常必须在内容更新后更新它们。 如果您已自动生成PDF或图片文件，则也可以选择自动使这些文件失效。
* 自动失效不涉及调度程序在更新时执行的任何操作，只涉及到statfile。 但是，触摸statfile会自动使缓存内容废弃，而无需将其从缓存中实际删除。

## Dispatcher如何返回文档 {#how-dispatcher-returns-documents}

![](assets/chlimage_1-6.png)

### 确定文档是否受缓存的影响

您可以 [定义Dispatcher在配置文件中缓存的文档](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)。 调度程序根据可缓存文档列表检查请求。 如果文档不在此列表中，则调度程序将从AEM实例请求该文档。

在以下情 *况下* ，调度程序始终直接从AEM实例请求文档：

* 如果请求URI包含问号“?”。 这通常表示无需缓存的动态页面，如搜索结果。
* 缺少文件扩展名。 Web服务器需要扩展来确定文档类型（MIME类型）。
* 身份验证头已设置（可以配置）

>[!NOTE]
>
>GET或HEAD（对于HTTP头）方法可由调度程序缓存。 有关响应头缓存的其他信息，请参阅 [缓存HTTP响应头部分](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html) 。

### 确定文档是否已缓存

调度程序将缓存的文件存储在Web服务器上，就像它们是静态网站的一部分一样。 如果用户请求可缓存的文档，调度程序将检查该文档是否存在于Web服务器的文件系统中：

* 如果文档已缓存，则Dispatcher将返回该文件。
* 如果未缓存该文档，则调度程序将从AEM实例请求该文档。

### 确定文档是否为最新

要了解文档是否为最新文档，调度程序执行两个步骤：

1. 它检查文档是否会自动失效。 否则，文档将被视为最新文档。
1. 如果文档已配置为自动失效，调度程序将检查文档是否比上次可用更改旧或更新。 如果版本较旧，则调度程序从AEM实例请求当前版本，并替换缓存中的版本。

>[!NOTE]
>
>未自动 **失效的文档** ，会一直保留在缓存中，直到被物理删除；例如，通过网站上的内容更新。

## 负载平衡的优势 {#the-benefits-of-load-balancing}

负载平衡是在多个AEM实例中分配网站计算负载的实践。

![](assets/chlimage_1-7.png)

您将获得：

* **增强的处理**&#x200B;能力在实践中，这意味着Dispatcher在AEM的多个实例之间共享文档请求。 由于每个实例现在处理的文档更少，因此您的响应时间更短。 调度程序保留每个文档类别的内部统计信息，以便能够估计负载并高效地分发查询。

* **增加的故障保护范围**&#x200B;如果调度程序未收到来自实例的响应，它将自动将请求中继到另一个实例中的一个。 因此，如果一个实例变得不可用，则唯一的效果是网站的减速，与计算能力损失成正比。 但是，所有服务都将继续。

* 您还可以在同一静态Web服务器上管理不同的网站。

>[!NOTE]
>
>在负载平衡有效扩展负载的同时，缓存有助于减少负载。 因此，在设置负载平衡之前，请尝试优化缓存并降低整体负载。 良好的缓存可能会提高负载平衡器的性能，或者渲染负载平衡时不必要。

>[!CAUTION]
>
>虽然单个调度程序通常能够使可用Publish实例的容量饱和，但对于某些稀有应用程序而言，还可以另外平衡两个调度程序实例之间的负载。 需要仔细考虑具有多个调度程序的配置，因为额外的调度程序将增加可用Publish实例的负载，并且可以轻松降低大多数应用程序的性能。

## 调度程序如何执行负载平衡 {#how-the-dispatcher-performs-load-balancing}

### 性能统计数据

Dispatcher会保留有关AEM的每个实例处理文档的速度的内部统计信息。 调度程序根据这些数据估计哪个实例在响应请求时提供最快的响应时间，因此它保留该实例的必要计算时间。

不同类型的请求可能具有不同的平均完成时间，因此调度程序允许您指定文档类别。 然后在计算时间估计时考虑这些。 例如，您可以区分HTML页面和图像，因为通常的响应时间很可能不同。

如果您使用复杂的搜索功能，则可以为搜索查询创建新类别。 这有助于调度程序将搜索查询发送到响应最快的实例。 这样，在收到多个“昂贵”搜索查询时，速度较慢的实例将不会停滞，而其他实例将收到“较便宜”的请求。

### 个性化内容（粘性连接）

粘性连接可确保一个用户的文档全部由AEM的同一实例组成。 如果您使用个性化的页面和会话数据，这一点很重要。 数据存储在实例上，因此来自同一用户的后续请求必须返回到该实例，否则数据将丢失。

由于粘性连接限制了调度程序优化请求的能力，因此您应仅在需要时使用它们。 您可以指定包含“粘性”文档的文件夹，从而确保该文件夹中的所有文档都由每个用户在同一实例上组成。

>[!NOTE]
>
>对于使用粘性连接的大多数页面，您必须关闭缓存——否则，无论会话内容如何，页面在所有用户看来都一样。
>
>对于少 *数应用* ，可以同时使用粘性连接和缓存；例如，如果显示将数据写入会话的表单。

## 使用多个调度程序 {#using-multiple-dispatchers}

在复杂的设置中，您可以使用多个调度程序。 例如，您可以使用：

* 一个调度程序，用于在Intranet上发布网站
* 另一个调度程序位于不同的地址下，并具有不同的安全设置，用于在Internet上发布相同的内容。

在这种情况下，请确保每个请求只通过一个调度程序。 调度程序不处理来自其他调度程序的请求。 因此，请确保两个Dispatcher都直接访问AEM网站。

## 将Dispatcher与CDN结合使用 {#using-dispatcher-with-a-cdn}

内容交付网络(CDN)（如Akamai Edge Delivery或Amazon Cloud Front）从接近最终用户的位置交付内容。 就这个

* 加快最终用户的响应时间
* 加载服务器

作为HTTP基础架构组件，CDN的工作方式与Dispatcher非常相似：当CDN节点收到请求时，如果可能，它会从其缓存中提供请求（该资源在缓存中可用且有效）。 否则，它会延伸到下一个最接近的服务器，以检索资源并缓存它以备进一步请求（如果适用）。

“下一台最接近的服务器”取决于您的特定设置。 例如，在Akamai设置中，请求可以采用以下路径：

* Akamai Edge节点
* 阿卡迈中间层
* 您的防火墙
* 负载平衡器
* 调度程序
* AEM

在大多数情况下，Dispatcher是下一台服务器，它可能会从缓存中提供文档并影响返回到CDN服务器的响应头。

## 控制CDN缓存 {#controlling-a-cdn-cache}

有许多方法可以控制CDN在从Dispatcher中重新获取资源之前缓存资源的时间。

1. 显式配置\
   配置，根据mime类型、扩展、请求类型等，在CDN的缓存中保留特定资源的时间。

1. 过期和缓存控制标头\
   如果上游服务器 `Expires:` 发送， `Cache-Control:` 则大多数CDN都将遵守HTTP头和HTTP头。 这可以实现，例如，使用 [mod_expires](https://httpd.apache.org/docs/2.4/mod/mod_expires.html) Apache Module。

1. 手动失效\
   CDN允许通过Web界面从缓存中删除资源。
1. 基于API的失效\
   大多数CDN还提供REST和／或SOAP API，允许从缓存中删除资源。

在典型的AEM设置中，通过扩展和／或路径进行配置（可通过以上第1点和第2点实现）为经常使用的资源（如设计图像和客户端库）设置合理的缓存期。 部署新版本时，通常需要手动失效。

如果此方法用于缓存受管内容，则意味着只有在配置的缓存期到期且再次从Dispatcher中获取文档后，内容更改才对最终用户可见。

为了更细粒度的控制，基于API的失效允许您在Dispatcher缓存失效时使CDN缓存失效。 根据CDN API，您可以实施自己的 [ContentBuilder](https://docs.adobe.com/docs/en/cq/current/javadoc/com/day/cq/replication/ContentBuilder.html) 和 [TransportHandler](https://docs.adobe.com/docs/en/cq/current/javadoc/com/day/cq/replication/TransportHandler.html) （如果API不是基于REST的），并设置一个复制代理，使用这些代理使CDN缓存失效。

>[!NOTE]
>
>另请参 [阅AEM(CQ)Dispatcher Security和CDN+Browser Caching](https://www.slideshare.net/andrewmkhoury/dispatcher-caching-aemgemspart2jan2015) ，以及有关 [Dispatcher Caching的录制演示](https://docs.adobe.com/content/ddc/en/gems/dispatcher-caching---new-features-and-optimizations.html)。

## 将调度程序与作者服务器一起使用 {#using-a-dispatcher-with-an-author-server}

>[!CAUTION]
>
>如果您正在将 [AEM与触屏UI结合使用](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html) ，则不 **应缓存作者实** 例内容。 如果为作者实例启用了缓存，则需要禁用该缓存并删除缓存目录的内容。 要禁用缓存，您应编辑文 `author_dispatcher.any` 件并修改该部 `/rule` 分的属 `/cache` 性，如下所示：

```xml
/rules
{
/0000
{ /type "deny" /glob "*"}
}
```

Dispatcher可以在创作实例前使用，以提高创作性能。 要配置创作调度程序，请执行以下操作：

1. 在Web服务器中安装Dispatcher(这可以是Apache或IIS web服务器，请参阅安 [装Dispatcher](dispatcher-install.md))。
1. 您可能希望针对正在工作的AEM发布实例测试新安装的Dispatcher，以确保已实现基准正确安装。
1. 现在，确保调度程序能够通过TCP/IP连接到您的创作实例。
1. 将示例dispatcher.any文件替换为随 [Dispatcher下载提供的author_dispatcher.any文件](release-notes.md#downloads)。
1. 在文本 `author_dispatcher.any` 编辑器中打开，并进行以下更改：

   1. 将部分 `/hostname` 和 `/port` 部分更 `/renders` 改为指向您的创作实例。
   1. 将部分 `/docroot` 的内 `/cache` 容更改为指向缓存目录。 如果您正在将 [AEM与触屏UI结合使用](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html)，请参阅上述警告。
   1. 保存更改。

1. 删除您在上面配置的 `/cache` &gt;目 `/docroot` 录中的所有现有文件。
1. 重新启动Web服务器。

>[!NOTE]
>
>请注意，在提供的配置中，当您安装CQ5功能包、修补程序或影响位于或下面的任何内容的应用程序代码包时，您必须删除调度程序缓存中那些目录下的缓存文件，以确保下次请求它们时，将获取新升级的文件，而不是旧的缓存文件。 `author_dispatcher.any``/libs``/apps`

>[!CAUTION]
>
>如果您使用了之前配置的作者调度程序并启用了调 *度程序刷新代理* ，请执行以下操作：

1. 在您的AEM作者实 **例上删除或禁用作者调度程序的** flushing Agent。
1. 按照以上新说明重新执行作者调度程序配置。

<!--
[Author Dispatcher configuration file (Dispatcher 4.1.2 or later)](assets/author_dispatchernew.any)
-->
<!--[!NOTE]
>
>A related knowledge base article can be found here:  
>[How to configure the dispatcher in front of an authoring environment](https://helpx.adobe.com/cq/kb/HowToConfigureDispatcherForAuthoringEnvironment.html)
-->
