---
title: 为缓存性能优化网站
seo-title: 为缓存性能优化网站
description: 了解如何设计网站，最大限度地利用缓存优势。
seo-description: Dispatcher提供了许多内置机制，可用于优化性能。了解如何设计网站，最大限度地利用缓存优势。
uuid: 2d4114d1-f464-4e10-b25 c-a1 b9 a9 c715 d1
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/Dispatcher
topic-tags: 调度程序
content-type: 引用
discoiquuid: ba323503-1494-4048-941d-c1 d14 f2 e85 b2
redirecttarget: https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# 为缓存性能优化网站 {#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>Dispatcher版本独立于AEM。如果您遵循了一个指向Dispatcher文档的链接，则可能已重定向到该页面，该链接嵌入到AEM的先前版本的文档中。

Dispatcher提供了许多内置机制，可用于优化性能。本节将告诉您如何设计网站，使缓存的优势最大化。

>[!NOTE]
>
>您可能会记住，Dispatcher将缓存存储在标准Web服务器上。这意味着您：
>
>* 可以缓存可存储为页面并使用URL请求的所有内容
>* 无法存储其他内容，如HTTP头、cookie、会话数据和表单数据。
>
>
通常，很多缓存策略都涉及选择良好的URL，而不依赖于此附加数据。

## 使用一致的页面编码 {#using-consistent-page-encoding}

HTTP请求标头未缓存，因此，如果您在标题中存储页面编码信息，则可能会发生问题。在这种情况下，当Dispatcher通过缓存为页面提供服务时，将使用Web服务器的默认编码。有两种方法可以避免此问题：

* 如果只使用一个编码，请确保Web服务器上使用的编码与AEM网站的默认编码相同。
* 使用 `<META>` HTML `head` 部分中的标记设置编码，如以下示例所示：

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## 避免URL参数 {#avoid-url-parameters}

如果可能，请避免为要缓存的页面提供URL参数。例如，如果您有图片画廊，则从不缓存以下URL(除非相应 [地配置Dispatcher](dispatcher-configuration.md#main-pars_title_24))：

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

但是，您可以将这些参数放入页面URL中，如下所示：

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>此URL调用相同的页面和与gallery. html相同的模板。在模板定义中，您可以指定要显示页面的脚本，也可以对所有页面使用相同的脚本。

## 按URL自定义 {#customize-by-url}

如果允许用户更改字体大小(或任何其他布局自定义)，请确保在URL中反映不同的自定义设置。

例如，cookies未缓存，因此如果将字体大小存储在cookie(或类似机制)中，则不会为缓存的页面保留字体大小。因此，Dispatcher随机返回任何字体大小的文档。

将URL中的字体大小包含在选择器中可避免此问题：

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>对于大多数布局方面，也可以使用样式表和/或客户端脚本。通常情况下，这些功能会非常适合缓存。
>
>这也适用于打印版本，在该版本中，您可以使用以下URL：¡¡
>
>`www.myCompany.com/news/main.print.html`
>
>使用模板定义的脚本全球化，您可以指定用于渲染打印页面的单独脚本。

## 使用作标题的图像文件失效 {#invalidating-image-files-used-as-titles}

如果您将页面标题或其他文本作为图片呈现，则建议存储文件，以便在页面上的内容更新时删除它们：

1. 将图像文件放在与页面相同的文件夹中。
1. 对图像文件使用以下命名格式：

   `<page file name>.<image file name>`

例如，您可以将页面的标题存储在MyPage. title. gif文件中。如果更新页面，此文件会自动删除，因此对页面标题的任何更改都会自动反映在缓存中。

>[!NOTE]
>
>AEM实例中不一定存在图像文件。您可以使用动态创建图像文件的脚本。然后调度程序将文件存储在Web服务器上。

## 使用于导航的图像文件失效 {#invalidating-image-files-used-for-navigation}

如果您对导航条目使用图片，则该方法与标题基本相同，只是稍微复杂一些。使用目标页面存储所有导航图像。如果您使用两张用于正常和活动的图片，则可以使用以下脚本：

* 像正常一样显示页面的脚本。
* 处理“. normal”请求并返回正常图片的脚本。
* 处理“. active”请求并返回激活的图片的脚本。

您必须使用与页面相同的命名柄来创建这些图片，以确保内容更新删除这些图片和页面。

对于未修改的页面，图片仍保留在缓存中，但页面本身通常会自动失效。

## 个性化 {#personalization}

Dispatcher无法缓存个性化数据，因此建议您将个性化限制在必要的位置。说明原因：

* 如果您使用可自由自定义的开始页面，则用户每次请求该页面时都必须进行合成。
* 相反，如果您选择了10个不同的开始页面，则可以缓存每个页面，从而提高性能。

>[!NOTE]
>
>如果您个性化每个页面(例如，将用户姓名放入标题栏)，则无法缓存该页面，这会造成主要性能影响。
>
>但是，如果必须这样做，您可以：
>
>* 使用iFrames将页面拆分为一个部分，该部分对于所有用户都是相同的，对于用户的所有页面而言，这一点是相同的。然后，您可以缓存这两个部分。
>* 使用客户端JavaScript显示个性化信息。但是，如果用户关闭JavaScript，则必须确保页面正确显示。
>



## 粘性连接 {#sticky-connections}

[粘性连接](dispatcher.md#TheBenefitsofLoadBalancing) 确保一个用户的文档全部在同一台服务器上构成。如果用户离开此文件夹，稍后又返回到该文件夹，连接仍将保持不变。定义一个文件夹，以保存需要网站连接连接的所有文档。请尽量避免在其中有其他文档。如果您使用个性化的页面和会话数据，这会影响负载平衡。

## MIME类型 {#mime-types}

浏览器可以通过两种方式确定文件的类型：

1. 按其扩展名(例如.html、. gif、.jpg等)
1. 由服务器随文件一起发送的MIME类型。

对于大多数文件，MIME类型在文件扩展名中隐含。例如：

1. 按其扩展名(例如.html、. gif、.jpg等)
1. 由服务器随文件一起发送的MIME类型。

如果文件名没有扩展名，则将其显示为纯文本。

MIME类型是HTTP头的一部分，因此，调度程序不缓存它。如果AEM应用程序返回的文件没有结束，但依赖MIME类型，则这些文件可能会错误显示。

要确保正确缓存文件，请遵循以下准则：

* 确保文件始终具有正确的扩展名。
* 避免通用文件服务脚本，这些脚本包含下载. jsp等URL？file=2214。重写脚本以使用包含文件规范的URL；上一示例将下载.214.pdf。

