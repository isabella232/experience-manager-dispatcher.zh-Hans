---
title: 调度程序概述
seo-title: Adobe AEM调度程序概述
description: 本文提供Dispatcher的一般概述。
seo-description: 本文提供Adobe Experience Manager Dispatcher的一般概述。
uuid: 71766f86-5e91-446b-a078-061b179 d090 d
pageversionid: '1193211344162'
contentOwner: 用户
topic-tags: 调度程序
content-type: 引用
discoiquuid: 1d449ee2-4cdd-4b7a-8b4e-7e6fc0a1d7ee
translation-type: tm+mt
source-git-commit: 6d3ff696780ce55c077a1d14d01efeaebcb8db28

---


# Dispatcher Overview {#dispatcher-overview}

>[!NOTE]
>
>Dispatcher版本独立于AEM。如果您遵循了一个指向Dispatcher文档的链接，则可能已重定向到该页面，该链接嵌入到AEM的先前版本的文档中。

Dispatcher是Adobe Experience Manager的缓存和/或负载平衡工具。使用AEM的Dispatcher还有助于保护AEM服务器免遭攻击。因此，您可以通过将Dispatcher与企业级Web服务器结合使用来提高AEM实例的安全性。

部署Dispatcher的过程与Web服务器和所选操作系统平台无关：

1. 了解Dispatcher(此页)。Also, see [frequently asked questions about dispatcher](https://helpx.adobe.com/experience-manager/using/dispatcher-faq.html).
1. Install a [supported web server](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html) according to the web server documentation.

1. [在Web服务器](dispatcher-install.md) 上安装Dispatcher模块，并相应地配置Web服务器。
1. [配置调度程序](dispatcher-configuration.md) (调度程序. any文件)。

1. [配置AEM](page-invalidate.md) 以使内容更新使缓存失效。

>[!NOTE]
>
>To gain a better undestanding of how Dispatcher works with AEM see [Ask the AEM Community Experts for July 2017](https://bit.ly/ATACE0717).

根据需要使用以下信息：

* [调度程序安全核对清单](security-checklist.md)
* [调度程序知识库](https://helpx.adobe.com/cq/kb/index/dispatcher.html)
* [为缓存性能优化网站](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html)
* [使用Dispatcher和多个域](dispatcher-domains.md)
* [将SSL与调度程序结合使用](dispatcher-ssl.md)
* [实施权限敏感型缓存](permissions-cache.md)
* [调度程序问题疑难解答](dispatcher-troubleshooting.md)
* [调度程序热点问题常见问题解答](dispatcher-faq.md)

>[!NOTE]
>
>**Dispatcher** 最常见的用法是从AEM **发布实例** 缓存响应，以提高向外发布的网站的响应速度和安全性。大多数讨论侧重于这种情况。
>
>But, the Dispatcher can also be used to increase the responsiveness of your **author instance**, particularly if you have a large number users editing and updating your website. For details specific to this case see [Using a Dispatcher with an Author Server](#using-a-dispatcher-with-an-author-server), below.

## Why use Dispatcher to implement Caching? {#why-use-dispatcher-to-implement-caching}

Web发布有两种基本方法：

* **静态Web服务器**：如Apache或IIS，非常简单，但快速。
* **内容管理服务器**：它提供动态的实时智能内容，但需要更多的计算时间和其他资源。

Dispatcher有助于实现快速和动态的环境。它作为静态HTML服务器的一部分工作，如Apache，目标为：

* 以静态网站的形式存储(或“缓存”)尽可能多的站点内容
* 尽可能少地访问布局引擎。

这意味着：

* **静态内容** 的处理速度与静态Web服务器上的速度和易用性完全相同；*此外，您还可以使用用于静态Web服务器的管理和安全工具*。

* **动态内容** 是根据需要生成的，而不会使系统降低任何绝对必要的速度。

Dispatcher包含根据动态站点内容生成和更新静态HTML的机制。您可以详细指定将哪些文档存储为静态文件，以及哪些文档始终是动态生成的。

本节介绍了本准则背后的原则。

### Static Web Server {#static-web-server}

![](assets/chlimage_1-3.png)

静态Web服务器(如Apache或IIS)为网站访客提供静态HTML文件。静态页面只创建一次，因此将为每个请求提供相同的内容。

这一过程非常简单，因此非常高效。如果访客请求一个文件(例如HTML页面)，则通常会直接从内存中带走该文件，最差的是从本地驱动器读取该文件。静态Web服务器已有一段时间可用，因此有大量用于管理和安全管理的工具，它们与网络基础结构紧密集成。

### Content Management Servers {#content-management-servers}

![](assets/chlimage_1-4.png)

如果您使用内容管理服务器(如AEM)，高级布局引擎将处理访客请求。引擎从存储库中读取内容，该存储库与样式、格式和访问权限相结合，将内容转换为根据访客的需要和权限定制的文档。

这使您能够创建更丰富的动态内容，从而提高网站的灵活性和功能。但是，布局引擎需要的处理能力比静态服务器更高，因此如果许多访客使用系统，此设置可能会容易降低速度。

## How Dispatcher performs Caching {#how-dispatcher-performs-caching}

![](assets/chlimage_1-5.png)

**缓存目录** 缓存，Dispatcher模块使用Web服务器提供静态内容的能力。Dispatcher将缓存的文档放入Web服务器的文档根中。

>[!NOTE]
>
>缺少HTTP头缓存的配置时，Dispatcher仅存储页面的HTML代码-它不存储HTTP头。如果您在网站中使用不同的编码，这可能会是一个问题，因为这些可能会丢失。To enable HTTP Header Caching, see [Configuring the Dispatcher Cache.](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)

>[!NOTE]
>
>在网络连接存储(NAS)上找到Web服务器的文档根目录会导致性能降低。此外，当在多个Web服务器之间共享位于NAS上的文档根目录时，执行复制操作时可能会发生tant锁定。

>[!NOTE]
>
>调度程序将缓存的文档存储在与所请求的URL相等的结构中。
>
>文件名称长度可能存在操作系统级别限制；例如，如果您有一个包含大量选择器的URL。

### 缓存方法

Dispatcher有两种主要方法，用于在网站进行更改时更新缓存内容。

* **“内容更新** ”删除了已更改的页面以及与它们直接关联的文件。
* **自动失效会** 自动使缓存的那些部分失效，这些部分可能在更新后过期。i.e. 它有效地将相关页面标记为过时，而不删除任何内容。

### 内容更新

在内容更新中，一个或多个AEM文档会发生更改。AEM会向调度程序发送一个联合请求，该请求相应地更新缓存：

1. 它将从缓存中删除修改后的文件。
1. 它将删除从缓存中以相同句柄开始的所有文件。例如，如果文件/en/index.html已更新，则所有以/en/index开头的文件。。此机制允许您设计高速缓存的站点，特别是在图片导航方面。
1. It *touches* the so-called **statfile**; this updates the timestamp of the statfile to indicate the date of the last change.

应注意以下几点：

* 内容更新通常与创作系统一起使用，该系统“知道”必须替换哪些内容。
* 内容更新影响的文件会被删除，但不会立即替换。下次请求此类文件时，调度程序将从AEM实例获取新文件并将其放入缓存中，从而覆盖旧内容。
* 通常，自动生成的包含来自页面的文本的图片存储在以相同处理方式开始的图片文件中，从而确保该关联存在于删除之中。例如，您可以将页面mypage.html的标题文本存储在同一文件夹中的图片mypage. titlePicture. gif中。这样，每次更新页面时，图片都会自动从缓存中删除，因此您可以确保图片始终反映页面的当前版本。
* 您可能有多个状态，例如每个语言文件夹。If a page is updated, AEM looks for the next parent folder containing a statfile, and *touches* that file.

### 自动失效

自动失效会自动使缓存部分失效-无需删除任何文件。在每个内容更新过程中，都会触摸所谓的statfile，因此其时间戳反映上次内容更新。

Dispatcher有一个要自动失效的文件列表。请求该列表中的文档时，调度程序将缓存文档的日期与statfile的时间戳进行比较：

* 如果缓存的文档较新，Dispatcher将返回它。
* 如果较旧，Dispatcher将从AEM实例检索当前版本。

同样，应注意一些点：

* 当HTML页面的相互关系复杂时，通常使用自动失效。这些页面包含链接和导航条目，因此通常必须在内容更新后更新它们。如果您已自动生成PDF或图片文件，则您也可以选择自动使它们失效。
* 自动失效不涉及更新时调度程序执行的任何操作，但触摸状态除外。但是，触摸statfile会自动呈现缓存内容废弃，而不会将缓存从缓存中删除。

## How Dispatcher returns Documents {#how-dispatcher-returns-documents}

![](assets/chlimage_1-6.png)

### 确定文档是否可缓存

You can [define which documents the Dispatcher caches in the configuration file](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html). Dispatcher会根据可缓存文档列表检查请求。如果文档不在此列表中，Dispatcher将从AEM实例请求文档。

The Dispatcher *always* requests the document directly from the AEM instance in the following cases:

* 如果请求URI包含问号“？”。这通常表示动态页面(如搜索结果)，无需缓存该页面。
* 缺少文件扩展名。Web服务器需要该扩展来确定文档类型(MIME类型)。
* 设置了身份验证头(可以配置此功能)

>[!NOTE]
>
>GET或HEAD(对于HTTP头)方法由Dispatcher进行缓存。For additional information on response header caching, see the [Caching HTTP Response Headers](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html) section.

### 确定文档是否缓存

Dispatcher将缓存的文件存储在Web服务器上，就像它们是静态网站的一部分一样。如果用户请求cacheable文档，Dispatcher将检查该文档是否存在于Web服务器的文件系统中：

* 如果文档被缓存，Dispatcher将返回文件。
* 如果未缓存，Dispatcher将从AEM实例请求文档。

### 确定文档是否为最新

要了解文档是否是最新的，Dispatcher执行两个步骤：

1. 它检查文档是否要自动失效。如果没有，则将文档视为最新。
1. 如果将文档配置为自动失效，Dispatcher将检查它是否比上次可用的更改旧或更新。如果旧版本，Dispatcher将从AEM实例请求当前版本，并替换缓存中的版本。

>[!NOTE]
>
>Documents without **auto-invalidation** remain in the cache until they are physically deleted; e.g. by a content update on the web site.

## The Benefits of Load Balancing {#the-benefits-of-load-balancing}

负载平衡是指在AEM的多个实例中分发网站的计算负载的做法。

![](assets/chlimage_1-7.png)

您获得：

* **提高处理能力** 在实践中，这意味着Dispatcher共享多个AEM实例之间的文档请求。由于每个实例现在都有更少的文档处理，因此响应时间更短。Dispatcher保留每个文档类别的内部统计数据，因此它可以有效地估计加载和分发查询。

* **提高的故障安全覆盖** 如果Dispatcher没有收到来自实例的响应，它会自动将请求发送到其他实例之一。因此，如果某个实例变得不可用，则唯一的效果就是该站点的速度下降，这与损失的计算能力成正比。但是，所有服务都将继续。

* 您还可以管理同一静态Web服务器上的不同网站。

>[!NOTE]
>
>负载平衡可以有效地扩散负载，而缓存有助于减少负载。因此，在设置负载平衡之前，尝试优化缓存并减少整体负载。良好的缓存可能增加负载平衡器的性能或呈现不必要的负载平衡。

>[!CAUTION]
>
>虽然单个Dispatcher通常能够饱和可用发布实例的容量，但对于某些应用程序而言，对于某些应用程序来说，另外还可以平衡两个调度程序实例之间的负载。需要仔细考虑具有多个调度程序的配置，因为额外的Dispatcher将增加可用Publish实例上的负载，并且可以在大多数应用程序中轻松降低性能。

## How the Dispatcher performs Load Balancing {#how-the-dispatcher-performs-load-balancing}

### 性能统计数据

Dispatcher可保留有关AEM每实例处理文档的速度的内部统计数据。根据此数据，Dispatcher估计在回答请求时将提供最快的响应时间，因此它保留了该实例上必要的计算时间。

不同类型的请求可能具有不同的平均完成时间，因此调度程序允许您指定文档类别。计算时间估计时，会考虑这些问题。例如，您可以区分HTML页面和图像，因为典型的响应时间可能会很不同。

如果您使用精致的搜索功能，则可以为搜索查询创建新类别。这有助于调度程序向响应速度最快的实例发送搜索查询。这可防止在收到多个“昂贵”的搜索查询时停止较慢的实例，而另一些则获得“更便宜”的请求。

### 个性化内容(粘性连接)

粘性连接确保一个用户的文档都在AEM的同一实例中构成。如果您使用个性化的页面和会话数据，这很重要。数据存储在实例上，因此来自同一用户的后续请求必须返回到该实例，否则数据会丢失。

因为粘性连接限制了Dispatcher优化请求的能力，因此只应在需要时使用这些连接。您可以指定包含“粘性”文档的文件夹，从而确保每个用户在同一个实例上合成该文件夹中的所有文档。

>[!NOTE]
>
>对于使用粘性连接的大多数页面，必须关闭缓存-否则，无论会话内容如何，页面都将与所有用户相同。
>
>For a *few* applications, it can be possible to use both sticky connections and caching; for example, if you display a form that writes data to the session.

## Using Multiple Dispatchers {#using-multiple-dispatchers}

在复杂的设置中，您可以使用多个调度程序。例如，您可以使用：

* 一个Dispatcher，用于在Intranet上发布网站
* 在不同地址下和具有不同安全性设置的第二个调度程序下发布Internet上的相同内容。

在这种情况下，请确保每个请求只通过一个调度程序。Dispatcher不处理来自其他Dispatcher的请求。因此，请确保两个Dispatcher直接访问AEM网站。

## Using Dispatcher with a CDN {#using-dispatcher-with-a-cdn}

内容交付网络(CDN)，如Akamai Edge Delivery或Amazon Cloud Front，可从最终用户附近的位置交付内容。作者：

* 缩短最终用户的响应时间
* 在服务器上加载负载

作为HTTP基础结构组件，CDN类似于Dispatcher：当CDN节点接收到请求时，它会在可能的情况下从缓存中提供请求(资源在缓存中可用并且有效)。否则，它会到达下一个最接近的服务器，以检索资源并将其缓存以备进一步请求。

“下一个最接近服务器”取决于您的特定设置。例如，在Akamai设置中，请求可以采用以下路径：

* Akamai Edge节点
* Akamai Midgres图层
* 您的防火墙
* 负载平衡器
* 调度程序
* AEM

在大多数情况下，调度程序是下一个可能从缓存中服务文档并影响返回到CDN服务器的响应头的服务器。

## Controlling a CDN cache {#controlling-a-cdn-cache}

在重新访存源之前，CDN将缓存资源的方法有一个编号。

1. 显式配置\
   根据MIME类型、扩展、请求类型等，配置在CDN缓存中存放时间长短的特定资源。

1. 过期和缓存控制标题\
   Most CDNs will honor `Expires:` and `Cache-Control:` HTTP Headers if sent by the upstream server. This can be achieved e.g. by using the [mod_expires](https://httpd.apache.org/docs/2.4/mod/mod_expires.html) Apache Module.

1. 手动失效\
   CDN允许通过Web界面从缓存中删除资源。
1. 基于API的无效\
   大多数CDNS还提供REST和/或SOAP API，它们允许从缓存中删除资源。

在典型AEM设置中，由扩展和/或路径(可通过上面和上面的点实现)的配置可为经常使用的资源设置合理的缓存周期，这些资源通常不会经常发生更改，如设计图像和客户端库。部署新版本时，通常需要手动失效。

如果此方法用于缓存托管内容，则表示只有在配置缓存期结束且文档从Dispatcher中获取后，内容更改才会对最终用户可见。

为实现更精细的控制，基于API的无效允许您在调度程序缓存失效时使CDN缓存失效。Based on the CDNs API, you can implement your own [ContentBuilder](https://docs.adobe.com/docs/en/cq/current/javadoc/com/day/cq/replication/ContentBuilder.html) and [TransportHandler](https://docs.adobe.com/docs/en/cq/current/javadoc/com/day/cq/replication/TransportHandler.html) (if the API is not REST-based) and set up a Replication Agent that will use these to invalidate the CDN&#39;s cache.

>[!NOTE]
>
>See also [AEM (CQ) Dispatcher Security and CDN+Browser Caching](https://www.slideshare.net/andrewmkhoury/dispatcher-caching-aemgemspart2jan2015) and recorded presentation on [Dispatcher Caching](https://docs.adobe.com/content/ddc/en/gems/dispatcher-caching---new-features-and-optimizations.html).

## Using a Dispatcher with an Author Server {#using-a-dispatcher-with-an-author-server}

>[!CAUTION]
>
>if you are using [AEM with Touch UI](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html) you should **not** cache author instance content. 如果为作者实例启用了缓存，您需要禁用它并删除缓存目录的内容。To disable caching, you should edit the `author_dispatcher.any` file and modify the `/rule` property of the `/cache` section as follows:

```xml
/rules
{
/0000
{ /type "deny" /glob "*"}
}
```

Dispatcher可用于创作实例的前面以提高创作性能。要配置创作调度程序，请执行以下操作：

1. Install a Dispatcher in a web server (this could be Apache or IIS web server, see [Installing Dispatcher](dispatcher-install.md)).
1. 您可能希望针对正在运行的AEM发布实例测试新安装的Dispatcher，以确保基线正确安装已被修复。
1. 现在，确保Dispatcher能够通过TCP/IP连接到作者实例。
1. Replace the sample dispatcher.any file with the author_dispatcher.any file provided with the [Dispatcher download](release-notes.md#downloads).
1. Open the `author_dispatcher.any` in a text editor and make the following changes:

   1. Change the `/hostname` and `/port` of the `/renders` section to point to your author instance.
   1. Change the `/docroot` of the `/cache` section to point to a cache directory. In case you are using [AEM with Touch UI](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html), see the warning above.
   1. 保存更改。

1. Delete all existing files in the `/cache` &gt; `/docroot` directory which you configured above.
1. 重新启动Web服务器。

>[!NOTE]
>
>Please note that with the provided `author_dispatcher.any` configuration, when you install a CQ5 feature pack, hotfix, or application code package that affects any content under `/libs` or `/apps` then you must delete the cached files under those directories in your dispatcher cache to ensure that the next time they are requested the newly upgraded files are fetched, and not the old cached ones.

>[!CAUTION]
>
>If you have used the previously configured author dispatcher and enabled a *dispatcher flushing agent* then please do the following:

1. Delete or disable the **author dispatcher&#39;s** flushing agent on your AEM author instance.
1. 按照以上新增说明重新执行作者调度程序配置。

<!--
[Author Dispatcher configuration file (Dispatcher 4.1.2 or later)](assets/author_dispatchernew.any)
-->
<!--[!NOTE]
>
>A related knowledge base article can be found here:  
>[How to configure the dispatcher in front of an authoring environment](https://helpx.adobe.com/cq/kb/HowToConfigureDispatcherForAuthoringEnvironment.html)
-->
