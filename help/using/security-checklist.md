---
title: 调度程序安全核对清单
seo-title: 调度程序安全核对清单
description: 在继续生产之前应完成的安全清单。
seo-description: 在继续生产之前应完成的安全清单。
uuid: 7bfa3202-03f6-48e9-8d2e-2a40e137ecs
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/Dispatcher
topic-tags: 调度程序
content-type: 引用
discoiquuid: fbfafa55-c029-4ed7-ab3 e-1bebde18248
jcr-lastmodifiedby: remove-legacypath-6-1
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: 8dd56f8b90331f0da43852e25893bc6f3e606a97

---


# 调度程序安全核对清单{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

作为前端系统的调度程序为Adobe Experience Manager基础结构提供了额外的安全性层。Adobe强烈建议您在开展生产之前填写以下清单。

>[!CAUTION]
>
>您还必须在上线前完成您的AEM版本的安全核对清单。请参阅相应 [的Adobe Experience Manager文档](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html)。

## 使用Dispatcher的最新版本 {#use-the-latest-version-of-dispatcher}

您应当安装适用于平台的最新可用版本。您应升级Dispatcher实例以使用最新版本以利用产品和安全增强。请参阅 [安装Dispatcher](dispatcher-install.md)。

>[!NOTE]
>
>您可以通过查看调度程序日志文件来检查调度程序安装的当前版本。
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>要查找日志文件，请检查您 `httpd.conf`的调度程序配置。

## 限制可刷新缓存的客户端 {#restrict-clients-that-can-flush-your-cache}

Adobe建议 [您限制可刷新缓存的客户端。](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## 为传输层安全启用HTTPS {#enable-https-for-transport-layer-security}

Adobe建议在创作实例和发布实例上启用HTTPS传输层。

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

## 限制访问 {#restrict-access}

配置调度程序时，您应尽可能限制外部访问。请参阅 [Dispatcher文档](dispatcher-configuration.md#main-pars_184_1_title) 中的示例/filter部分。

## 确保对管理URL的访问已拒绝 {#make-sure-access-to-administrative-urls-is-denied}

确保使用过滤器阻止对任何管理URL(如Web控制台)的外部访问。

有关需要阻止的URL列表，请参阅 [测试Dispatcher Security](dispatcher-configuration.md#testing-dispatcher-security) 。

## 使用白名单代替黑名单 {#use-whitelists-instead-of-blacklists}

从固有的角度来说，白名单是提供访问控制的更好方法，它们假定应拒绝所有访问请求，除非它们明确属于白名单中的一部分。此模型对在特定配置阶段可能尚未审查或考虑考虑的新请求提供更具限制性的控制。

## 使用专用系统用户运行Dispatcher {#run-dispatcher-with-a-dedicated-system-user}

配置Dispatcher时，应确保Web服务器由具有至少权限的专用用户运行。建议仅向调度程序缓存文件夹授予写入权限。

Adadonaly，IIS用户需要如下所示配置其网站：

1. 在网站的物理路径设置中，选择 **Connect作为特定用户**。
1. 设置用户。

## 防止拒绝服务(DoS)攻击 {#prevent-denial-of-service-dos-attacks}

拒绝服务(DoS)攻击旨在使计算机资源对其预期用户不可用。

在调度程序级别，有两种配置以防止DoS攻击： [](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (过滤器))

* 使用mod_ rewrite模块(例如 [Apache2.2](https://httpd.apache.org/docs/2.2/mod/mod_rewrite.html))执行URL验证(如果URL模式规则不太复杂)。

* 使用 [过滤器防止调度程序使用虚假扩展缓存URL](dispatcher-configuration.md#configuring-access-to-conten-tfilter)。\
   例如，更改缓存规则以将缓存限制为预期的MIME类型，如：

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`
   可查看示例配置文件 [以限制外部访问](#restrict-access)，这包括MIME类型限制。

要安全地在发布实例上启用功能，请配置过滤器以防止访问以下节点：

* `/etc/`
* `/libs/`

然后，配置过滤器以允许访问以下节点路径：

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` (JS、CSS和JSON)
* `/libs/cq/security/userinfo.json` (CQ用户信息)
* `/libs/granite/security/currentuser.json` (**数据不得缓存**)

* `/libs/cq/i18n/*` (Internation)

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## 配置Dispatcher以防止CSRF攻击 {#configure-dispatcher-to-prevent-csrf-attacks}

AEM提供了一个 [旨在](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps) 防止跨站点请求伪造攻击的框架。为了正确利用此框架，您需要在调度程序中列入白名单支持。您可以通过以下方式执行此操作：

1. 创建用于允许 `/libs/granite/csrf/token.json` 路径的过滤器；
1. 将 `CSRF-Token` 标题添加到Dispatcher配置的 `clientheaders` 部分。

## 防止点击劫持 {#prevent-clickjacking}

为了防止clickjacking，我们建议您配置网络服务器，以提供将 `X-FRAME-OPTIONS` HTTP头设置为的HTTP头 `SAMEORIGIN`。

有关点击劫持的详细 [信息，请参阅OWASP站点](https://www.owasp.org/index.php/Clickjacking)。

## 执行渗透测试 {#perform-a-penetration-test}

Adobe强烈建议在开展生产之前对AEM基础结构执行渗透测试。

