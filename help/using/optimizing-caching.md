---
title: 为缓存性能优化网站
seo-title: 为缓存性能优化网站
description: 了解如何设计网站以最大化缓存的益处。
seo-description: Dispatcher提供了许多可用于优化性能的内置机制。 了解如何设计网站以最大化缓存的益处。
uuid: 2d4114d1-f464-4e10-b25c-a1b9a9c715d1
contentOwner: 用户
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: 调度程序
content-type: 引用
discoiquuid: ba323503-1494-4048-941d-c1d14f2e85b2
redirecttarget: https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: 2ca816ac0776d72be651b76ff4f45e0c3ed1450f

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
>调度程序版本独立于AEM。 如果您遵循了指向Dispatcher文档的链接（该链接嵌入在AEM先前版本的文档中），则您可能已被重定向到此页。

Dispatcher提供了许多可用于优化性能的内置机制。 本节将告诉您如何设计网站以最大化缓存的益处。

>[!NOTE]
>
>它可能有助于您记住，调度程序将缓存存储在标准Web服务器上。 这意味着您：
>
>* 可以缓存可存储为页面并使用URL请求的所有内容
>* 无法存储其他内容，如HTTP头、cookie、会话数据和表单数据。
>
>
通常，许多缓存策略都涉及选择好的URL，而不依赖此附加数据。

## 使用一致的页面编码 {#using-consistent-page-encoding}

HTTP请求标头不会缓存，因此，如果在标头中存储页面编码信息，则可能会出现问题。 在这种情况下，当Dispatcher从缓存中提供页面时，将使用Web服务器的默认编码进行页面编码。 有两种方法可以避免此问题：

* 如果只使用一种编码，请确保Web服务器上使用的编码与AEM网站的默认编码相同。
* 使用 `<META>` HTML部分中的标 `head` 签设置编码，如以下示例所示：

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## 避免URL参数 {#avoid-url-parameters}

如果可能，请避免要缓存的页面的URL参数。 例如，如果您有图片库，则不会缓存以下URL(除非相应地配置了调 [度程序](dispatcher-configuration.md#main-pars_title_24)):

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

但是，您可以将这些参数放入页面URL中，如下所示：

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>此URL调用与gallery.html相同的页面和模板。 在模板定义中，您可以指定呈现页面的脚本，也可以为所有页面使用相同的脚本。

## 按URL自定义 {#customize-by-url}

如果允许用户更改字体大小（或任何其他布局自定义），请确保URL中反映不同的自定义。

例如，Cookies不会缓存，因此如果您将字体大小存储在Cookie（或类似机制）中，则缓存的页面不会保留字体大小。 因此，Dispatcher会随机返回任何字体大小的文档。

在URL中包含字体大小作为选择器可避免此问题：

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>对于大多数布局方面，也可以使用样式表和／或客户端脚本。 这些功能通常在缓存方面非常有效。
>
>这对于打印版本也很有用，在打印版本中，您可以使用URL，例如："
>
>`www.myCompany.com/news/main.print.html`
>
>使用模板定义的脚本覆盖，您可以指定一个用于呈现打印页面的单独脚本。

## 使用作标题的图像文件失效 {#invalidating-image-files-used-as-titles}

如果您将页面标题或其他文本渲染为图片，则建议存储这些文件，以便在页面内容更新时删除它们：

1. 将图像文件放在与页面相同的文件夹中。
1. 对图像文件使用以下命名格式：

   `<page file name>.<image file name>`

例如，可将页面myPage.html的标题存储在文件myPage.title.gif中。 如果页面已更新，则会自动删除此文件，因此对页面标题所做的任何更改都会自动反映在缓存中。

>[!NOTE]
>
>AEM实例上不一定存在图像文件。 您可以使用动态创建图像文件的脚本。 然后，调度程序将文件存储在Web服务器上。

## 导航使用的图像文件无效 {#invalidating-image-files-used-for-navigation}

如果将图片用于导航条目，则该方法与标题基本相同，只是稍微复杂一些。 将所有导航图像与目标页面一起存储。 如果对正常和活动图片使用两张图片，则可以使用以下脚本：

* 正常情况下显示页面的脚本。
* 处理“.normal”请求并返回正常图片的脚本。
* 处理“.active”请求并返回已激活图片的脚本。

使用与页面相同的命名句柄创建这些图片很重要，这样可确保内容更新同时删除这些图片和页面。

对于未修改的页面，图片仍保留在缓存中，尽管页面本身通常自动失效。

## 个性化 {#personalization}

Dispatcher无法缓存个性化数据，因此建议您将个性化限制在必要的位置。 为说明原因：

* 如果您使用可自由自定义的起始页，则每次用户请求该页面时，都必须合成该页面。
* 相反，如果您提供10个不同的开始页面选项，则可以缓存每个页面，从而提高性能。

>[!NOTE]
>
>如果您个性化每个页面（例如，将用户名放入标题栏），则无法缓存该页面，这可能会对性能产生重大影响。
>
>但是，如果您必须这样做，您可以：
>
>* 使用iFrames将页面拆分为一个部分，该部分对于所有用户是相同的，而一个部分对于用户的所有页面是相同的。 然后，可以缓存这两个部分。
>* 使用客户端JavaScript显示个性化信息。 但是，您必须确保在用户关闭JavaScript时页面仍能正确显示。
>



## 粘性连接 {#sticky-connections}

[粘性连接](dispatcher.md#TheBenefitsofLoadBalancing) ，确保一个用户的文档都在同一台服务器上完成。 如果用户离开此文件夹，稍后又返回到该文件夹，则连接仍然保持。 定义一个文件夹，以保存所有需要网站粘性连接的文档。 请尽量不要在其中包含其他文档。 如果使用个性化的页面和会话数据，这会影响负载平衡。

## MIME Types {#mime-types}

浏览器可以通过两种方式确定文件类型：

1. 通过其扩展(例如，.html、.gif、.jpg等)
1. 按服务器随文件发送的MIME类型。

对于大多数文件，MIME类型隐含在文件扩展名中。 即：

1. 通过其扩展(例如，.html、.gif、.jpg等)
1. 按服务器随文件发送的MIME类型。

如果文件名没有扩展名，则其显示为纯文本。

MIME类型是HTTP头的一部分，因此，调度程序不缓存它。 如果AEM应用程序返回的文件没有可识别的文件结尾，但依赖MIME类型，则这些文件可能显示不正确。

要确保正确缓存文件，请遵循以下准则：

* 确保文件始终具有正确的扩展名。
* 避免使用通用文件服务脚本，这些脚本具有URL，如download.jsp?file=2214。 重新编写脚本以使用包含文件规范的URL;对于上一个示例，此示例为download.2214.pdf。

