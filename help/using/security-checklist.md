---
title: Dispatcher 安全核对清单
seo-title: Dispatcher 安全核对清单
description: 应在开始生产前完成的安全核对清单。
seo-description: 应在开始生产前完成的安全核对清单。
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
exl-id: 49009810-b5bf-41fd-b544-19dd0c06b013
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: ht
source-wordcount: '653'
ht-degree: 100%

---

# Dispatcher 安全核对清单{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Adobe 强烈建议您在开始生产前完成以下核对清单。

>[!CAUTION]
>
>您还必须在上线之前完成 AEM 版本的安全核对清单。请参阅相应的 [Adobe Experience Manager 文档](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html)。

## 使用最新版本的 Dispatcher {#use-the-latest-version-of-dispatcher}

您应安装适用于您的平台的最新可用版本。您应将您的 Dispatcher 实例升级到最新版本以使用产品和安全改进功能。请参阅[安装 Dispatcher](dispatcher-install.md)。

>[!NOTE]
>
>您可以通过查看 Dispatcher 日志文件来查看 Dispatcher 安装的当前版本。
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>要查找日志文件，请检查 `httpd.conf` 中的 Dispatcher 配置。

## 限制可以刷新缓存的客户端 {#restrict-clients-that-can-flush-your-cache}

Adobe 建议您[限制可以刷新缓存的客户端。](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## 为传输层安全性启用 HTTPS {#enable-https-for-transport-layer-security}

Adobe 建议在创作实例和发布实例上启用 HTTPS 传输层。

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

配置 Dispatcher 时，您应尽可能限制外部访问。请参阅 Dispatcher 文档中的[示例 /filter 部分](dispatcher-configuration.md#main-pars_184_1_title)。

## 确保对管理 URL 的访问被拒绝 {#make-sure-access-to-administrative-urls-is-denied}

请务必使用过滤器阻止对任何管理 URL 的外部访问，例如 Web 控制台。

有关需要阻止的 URL 的列表，请参阅[测试 Dispatcher 安全性](dispatcher-configuration.md#testing-dispatcher-security)。

## 使用允许列表而非阻止列表 {#use-allowlists-instead-of-blocklists}

允许列表是一种提供访问控制的更好的方式，因为它们本身会假定所有访问请求都应被拒绝（明确属于允许列表的访问请求除外）。此模型可以对在特定配置阶段可能尚未审查或考虑的新请求进行更严格的控制。

## 以专用系统用户身份运行 Dispatcher {#run-dispatcher-with-a-dedicated-system-user}

在配置 Dispatcher 时，您应确保 Web 服务器由具有最低权限的专用用户运行。建议只授予对 Dispatcher 缓存文件夹的写访问权限。

此外，IIS 用户需要按如下方式配置其网站：

1. 在网站的物理路径设置中，选择&#x200B;**“以特定用户身份连接”**。
1. 设置用户。

## 防御拒绝服务 (DoS) 攻击 {#prevent-denial-of-service-dos-attacks}

拒绝服务 (DoS) 攻击是一种试图让计算机资源对其目标用户不可用的攻击。

在 Dispatcher 级别，可通过两种配置方法来防御 DoS 攻击：[](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (过滤器))

* 使用 mod_rewrite 模块（例如 [Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)）执行 URL 验证（如果 URL 模式规则不是太复杂）。

* 通过使用[过滤器](dispatcher-configuration.md#configuring-access-to-conten-tfilter)防止 Dispatcher 缓存带假扩展的 URL。\
   例如，更改缓存规则以仅允许缓存预期的 MIME 类型，例如：

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

   可以看到用于[限制外部访问](#restrict-access)的示例配置文件，其中包括对 MIME 类型的限制。

要在发布实例上安全地启用全部功能，请配置过滤器以阻止对以下节点的访问：

* `/etc/`
* `/libs/`

然后，配置过滤器以允许对以下节点路径的访问：

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*`（JS、CSS 和 JSON）
* `/libs/cq/security/userinfo.json`（CQ 用户信息）
* `/libs/granite/security/currentuser.json`（**不得缓存数据**）

* `/libs/cq/i18n/*`（国际化）

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## 配置 Dispatcher 以防御 CSRF 攻击 {#configure-dispatcher-to-prevent-csrf-attacks}

AEM 提供了一个用于防御跨站点请求伪造攻击的[框架](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps)。要正确使用此框架，您需要将 Dispatcher 中的 CSRF 令牌支持列入允许列表。您可以执行以下操作来实现此目标：

1. 创建过滤器以允许 `/libs/granite/csrf/token.json` 路径；
1. 将 `CSRF-Token` 标头添加到 Dispatcher 配置的 `clientheaders` 部分。

## 防御点击劫持攻击 {#prevent-clickjacking}

要防御点击劫持攻击，建议您将 Web 服务器配置为将 `X-FRAME-OPTIONS` HTTP 标头集提供给 `SAMEORIGIN`。

有关[点击劫持攻击的更多信息，请参阅 OWASP 网站](https://www.owasp.org/index.php/Clickjacking)。

## 执行渗透测试 {#perform-a-penetration-test}

Adobe 强烈建议您在开始生产之前对 AEM 基础架构执行渗透测试。
