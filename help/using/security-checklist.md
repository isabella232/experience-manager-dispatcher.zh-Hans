---
title: Dispatcher 安全核对清单
seo-title: Dispatcher 安全核对清单
description: 在开始生产之前应完成的安全清单。
seo-description: 在开始生产之前应完成的安全清单。
uuid: 7bfa3202-03f6-48e9-8d2e-2a40e137ecbe
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: fbfafa55-c029-4ed7-ab3e-1bebfde18248
jcr-lastmodifiedby: remove-legacypath-6-1
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: 7889c025fb8fb29e6f11ea01c5248470556d3160
workflow-type: tm+mt
source-wordcount: '653'
ht-degree: 1%

---


# Dispatcher 安全核对清单{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Adobe强烈建议您在开始生产之前完成以下核对清单。

>[!CAUTION]
>
>您还必须在AEM上市前完成版本的安全核对清单。 请参阅相应的[Adobe Experience Manager文档](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html)。

## 使用最新版本的调度程序{#use-the-latest-version-of-dispatcher}

您应安装适用于您的平台的最新可用版本。 您应升级Dispatcher实例，以使用最新版本来利用产品和安全增强功能。 请参阅[安装Dispatcher](dispatcher-install.md)。

>[!NOTE]
>
>您可以通过查看调度程序日志文件来检查调度程序安装的当前版本。
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>要查找日志文件，请检查`httpd.conf`中的调度程序配置。

## 限制可刷新缓存的客户端{#restrict-clients-that-can-flush-your-cache}

Adobe建议您[限制可刷新缓存的客户端。](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## 为传输层安全性启用HTTPS {#enable-https-for-transport-layer-security}

Adobe建议在创作和发布实例上启用HTTPS传输层。

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:41:28.841-0400

<p>Recommended to have SSL termination, front end SSL.</p> 
<p>Question is do we want to have SSL communication between dispatcher and AEM instances (publish and/or author).</p> 
<p>We might want to have two items:</p> 
<ul> 
 <li>MUST HTTPS clients -&gt; dispatcher / load balancer</li> 
 <li>NICE load balancer -&gt; dispatcher<br /> </li> 
 <li>NICE dispatcher -&gt; instances if sensitive information such as credit cards / or infrastructure requirements such as DMZ</li> 
</ul>

 -->

## 限制访问{#restrict-access}

配置调度程序时，应尽可能限制外部访问。 请参阅调度程序文档中的[示例/filter Section](dispatcher-configuration.md#main-pars_184_1_title)。

## 确保拒绝访问管理URL {#make-sure-access-to-administrative-urls-is-denied}

确保使用过滤器阻止对任何管理URL（如Web控制台）的外部访问。

有关需要阻止的URL的列表，请参阅[测试调度程序安全性](dispatcher-configuration.md#testing-dispatcher-security)。

## 使允许列表用阻止列表代替{#use-allowlists-instead-of-blocklists}

允许列表是提供访问控制的更好方式，因为它们本身就认为，除非明确属于，否则应拒绝所有访问请求允许列表。 此模型对某些配置阶段可能尚未审核或考虑的新请求提供更严格的控制。

## 使用专用系统用户{#run-dispatcher-with-a-dedicated-system-user}运行调度程序

配置调度程序时，您应确保Web服务器由具有最少权限的专用用户运行。 建议仅授予对调度程序缓存文件夹的写访问权限。

此外，IIS用户需要按如下方式配置其网站：

1. 在网站的物理路径设置中，选择&#x200B;**作为特定用户连接**。
1. 设置用户。

## 防止拒绝服务(DoS)攻击{#prevent-denial-of-service-dos-attacks}

拒绝服务(DoS)攻击是试图使计算机资源对其预期用户不可用。

在调度程序级别，有两种配置方法可防止DoS攻击：[](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (筛选器))

* 使用mod_rewrite模块（例如，[Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)）执行URL验证（如果URL模式规则不太复杂）。

* 通过使用[过滤器](dispatcher-configuration.md#configuring-access-to-conten-tfilter)防止调度程序缓存具有虚假扩展的URL。\
   例如，更改缓存规则以将缓存限制为预期的mime类型，例如：

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

   可以为限制外部访问](#restrict-access)的[看到一个配置文件示例，其中包括MIME类型的限制。

要安全地在发布实例上启用完整功能，请配置过滤器以阻止访问以下节点：

* `/etc/`
* `/libs/`

然后，配置过滤器以允许访问以下节点路径：

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` （JS、CSS和JSON）
* `/libs/cq/security/userinfo.json` （CQ用户信息）
* `/libs/granite/security/currentuser.json` (**数据不得缓存**)

* `/libs/cq/i18n/*` （内部化）

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## 配置调度程序以防止CSRF攻击{#configure-dispatcher-to-prevent-csrf-attacks}

AEM提供[framework](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps)，旨在防止跨站点请求伪造攻击。 为了正确利用此框架，您需要在调度程允许列表序中CSRF令牌支持。 可通过以下方式执行此操作：

1. 创建允许`/libs/granite/csrf/token.json`路径的过滤器；
1. 将`CSRF-Token`头添加到调度程序配置的`clientheaders`部分。

## 防止Clickjacking {#prevent-clickjacking}

为防止点击劫持，我们建议您配置Web服务器以提供设置为`SAMEORIGIN`的`X-FRAME-OPTIONS` HTTP头。

有关点击劫持的更多[信息，请参阅OWASP站点](https://www.owasp.org/index.php/Clickjacking)。

## 执行渗透测试{#perform-a-penetration-test}

Adobe强烈建议在开始生产之前对AEM基础架构执行渗透测试。

