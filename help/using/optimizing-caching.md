---
title: 优化网站以提高缓存性能
seo-title: 优化网站以提高缓存性能
description: 了解如何设计您的网站以最大限度地利用缓存的好处。
seo-description: Dispatcher提供了许多可用于优化性能的内置机制。 了解如何设计您的网站以最大限度地利用缓存的好处。
uuid: 2d4114d1-f464-4e10-b25c-a1b9a9c715d1
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: ba323503-1494-4048-941d-c1d14f2e85b2
redirecttarget: https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
source-git-commit: 2ca816ac0776d72be651b76ff4f45e0c3ed1450f
workflow-type: tm+mt
source-wordcount: '1167'
ht-degree: 3%

---


# 优化网站以提高缓存性能{#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>各个 Dispatcher 版本与 AEM 相互独立。如果单击以前版本 AEM 文档中嵌入的 Dispatcher 文档链接，可能会重定向到此页面。

Dispatcher提供了许多可用于优化性能的内置机制。 本节将介绍如何设计网站以最大限度地发挥缓存的优势。

>[!NOTE]
>
>它可能有助于您记住，Dispatcher将缓存存储在标准Web服务器上。 这意味着您：
>
>* 可以使用URL缓存可存储为页面和请求的所有内容
>* 无法存储其他内容，如HTTP头、Cookie、会话数据和表单数据。

>
>
通常，许多缓存策略都涉及选择良好的URL，而不依赖这些附加数据。

## 使用一致的页面编码{#using-consistent-page-encoding}

HTTP请求标头不会缓存，因此，如果将页面编码信息存储在标头中，则可能会出现问题。 在这种情况下，当Dispatcher从缓存中提供页面时，将使用Web服务器的默认编码来进行页面。 有两种方法可以避免此问题：

* 如果只使用一种编码，请确保Web服务器上使用的编码与AEM网站的默认编码相同。
* 在HTML `head`部分中使用`<META>`标记来设置编码，如以下示例中所示：

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## 避免URL参数{#avoid-url-parameters}

如有可能，请避免为要缓存的页面使用URL参数。 例如，如果您有图片库，则永远不会缓存以下URL（除非Dispatcher已相应地配置[](dispatcher-configuration.md#main-pars_title_24)）：

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

但是，您可以将这些参数放入页面URL中，如下所示：

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>此URL调用与gallery.html相同的页面和模板。 在模板定义中，您可以指定渲染页面的脚本，也可以对所有页面使用相同的脚本。

## 通过URL {#customize-by-url}自定义

如果允许用户更改字体大小（或任何其他布局自定义），请确保URL中反映了不同的自定义设置。

例如，Cookie未缓存，因此如果您将字体大小存储在Cookie（或类似机制）中，则不会为缓存的页面保留字体大小。 因此，Dispatcher会随机返回任何字体大小的文档。

在URL中作为选择器包含字体大小可避免此问题：

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>对于大多数布局方面，还可以使用样式表和/或客户端脚本。 这些功能通常在缓存方面非常有效。
>
>此外，对于打印版本，在该版本中，您可以使用URL，例如：&quot;
>
>`www.myCompany.com/news/main.print.html`
>
>使用模板定义的脚本代位，您可以指定一个单独的脚本来呈现打印页面。

## 使用作标题的图像文件失效{#invalidating-image-files-used-as-titles}

如果您将页面标题或其他文本渲染为图片，则建议存储这些文件，以便在页面上的内容更新时将其删除：

1. 将图像文件放置在与页面相同的文件夹中。
1. 对图像文件使用以下命名格式：

   `<page file name>.<image file name>`

例如，您可以将页面myPage.html的标题存储在文件myPage.title.gif中。 如果页面已更新，则会自动删除此文件，因此对页面标题所做的任何更改都会自动反映在缓存中。

>[!NOTE]
>
>图像文件不一定在AEM实例上存在。 您可以使用动态创建图像文件的脚本。 然后，Dispatcher将文件存储在Web服务器上。

## 使用于导航的图像文件失效{#invalidating-image-files-used-for-navigation}

如果将图片用于导航条目，则方法与标题基本相同，只是略微复杂一些。 将所有导航图像与目标页面一起存储。 如果您使用两张图片用于正常和活动，则可以使用以下脚本：

* 正常情况下显示页面的脚本。
* 处理“.normal”的脚本请求并返回正常图片。
* 处理“.active”请求并返回激活图片的脚本。

请务必使用与页面相同的命名句柄创建这些图片，以确保内容更新会删除这些图片和页面。

对于未修改的页面，图片仍保留在缓存中，尽管页面本身通常会自动失效。

## 个性化 {#personalization}

Dispatcher无法缓存个性化数据，因此建议您将个性化限制为必要时。 说明原因：

* 如果您使用可自由自定义的开始页面，则每次用户请求时都必须撰写该页面。
* 相反，如果您提供了10个不同的开始页，则可以缓存其中每个开始页，从而提高性能。

>[!NOTE]
>
>如果个性化每个页面（例如，将用户名放入标题栏中），则无法缓存该页面，这可能会对性能产生重大影响。
>
>但是，如果必须执行此操作，您可以：
>
>* 使用iFrame将页面拆分为一个部分，该部分对所有用户都是相同的，而另一个部分对用户的所有页面是相同的。 然后，可以缓存这两个部分。
>* 使用客户端JavaScript来显示个性化信息。 但是，您必须确保在用户关闭JavaScript时，页面仍能正确显示。

>



## 粘性连接{#sticky-connections}

[置顶](dispatcher.md#TheBenefitsofLoadBalancing) 连接确保同一用户的文档全部在同一服务器上撰写。如果用户离开此文件夹，稍后又返回到该文件夹，则连接仍会保持。 定义一个文件夹，以保存要求网站具有粘性连接的所有文档。 尽量不要在其中包含其他文档。 如果您使用个性化的页面和会话数据，这会影响负载平衡。

## MIME类型{#mime-types}

浏览器可通过两种方式来确定文件类型：

1. 扩展(例如.html、.gif、.jpg等)
1. 按服务器随文件发送的MIME类型。

对于大多数文件，文件扩展名中包含MIME类型。 i.e.:

1. 扩展(例如.html、.gif、.jpg等)
1. 按服务器随文件发送的MIME类型。

如果文件名没有扩展名，则会显示为纯文本。

MIME类型是HTTP标头的一部分，因此，Dispatcher不会缓存它。 如果您的AEM应用程序返回的文件没有可识别的文件结尾，而是依赖MIME类型，则这些文件可能会被错误地显示。

要确保正确缓存文件，请遵循以下准则：

* 确保文件的扩展名始终正确。
* 避免使用通用文件服务脚本，这些脚本具有URL，如download.jsp?file=2214。 重写脚本以使用包含文件规范的URL;对于上一个示例，此为download.2214.pdf。

