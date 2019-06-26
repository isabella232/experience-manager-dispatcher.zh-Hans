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
source-git-commit: 6d3ff696780ce55c077a1d14d01efeaebcb8db28

---


# The Dispatcher Security Checklist{#the-dispatcher-security-checklist}

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
>您还必须在上线前完成您的AEM版本的安全核对清单。Please refer to the corresponding [Adobe Experience Manager documentation](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html).

## Use the Latest Version of Dispatcher {#use-the-latest-version-of-dispatcher}

您应当安装适用于平台的最新可用版本。您应升级Dispatcher实例以使用最新版本以利用产品和安全增强。See [Installing Dispatcher](dispatcher-install.md).

>[!NOTE]
>
>您可以通过查看调度程序日志文件来检查调度程序安装的当前版本。
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>To find the log file, inspect the dispatcher configuration in your `httpd.conf`.

## Restrict Clients that Can Flush Your Cache {#restrict-clients-that-can-flush-your-cache}

Adobe recommends that you [limit the clients that can flush your cache.](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## Enable HTTPS for transport layer security {#enable-https-for-transport-layer-security}

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

## Restrict Access {#restrict-access}

配置调度程序时，您应尽可能限制外部访问。See [Example /filter Section](dispatcher-configuration.md#main-pars_184_1_title) in the Dispatcher documentation.

## Make Sure Access to Administrative URLs is Denied {#make-sure-access-to-administrative-urls-is-denied}

确保使用过滤器阻止对任何管理URL(如Web控制台)的外部访问。

See [Testing Dispatcher Security](dispatcher-configuration.md#testing-dispatcher-security) for a list of URLs that need to be blocked.

## Use Whitelists Instead Of Blacklists {#use-whitelists-instead-of-blacklists}

从固有的角度来说，白名单是提供访问控制的更好方法，它们假定应拒绝所有访问请求，除非它们明确属于白名单中的一部分。此模型对在特定配置阶段可能尚未审查或考虑考虑的新请求提供更具限制性的控制。

## Run Dispatcher with a Dedicated System User {#run-dispatcher-with-a-dedicated-system-user}

配置Dispatcher时，应确保Web服务器由具有至少权限的专用用户运行。建议仅向调度程序缓存文件夹授予写入权限。

Adadonaly，IIS用户需要如下所示配置其网站：

1. In the physical path setting for your web site, select **Connect as specific user**.
1. 设置用户。

## Prevent Denial of Service (DoS) Attacks {#prevent-denial-of-service-dos-attacks}

拒绝服务(DoS)攻击旨在使计算机资源对其预期用户不可用。

At the dispatcher level, there are two methods of configuring to prevent DoS attacks: [](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (Filters))

* Use the mod_rewrite module (for example, [Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)) to perform URL validations (if the URL pattern rules are not too complex).

* Prevent the dispatcher from caching URLs with spurious extensions by using [filters](dispatcher-configuration.md#configuring-access-to-conten-tfilter).\
   例如，更改缓存规则以将缓存限制为预期的MIME类型，如：

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`
   An example configuration file can be seen for [restricting external access](#restrict-access), this includes restrictions for mime types.

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

## Configure Dispatcher to prevent CSRF Attacks {#configure-dispatcher-to-prevent-csrf-attacks}

AEM provides a [framework](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps) aimed at preventing Cross-Site Request Forgery attacks. 为了正确利用此框架，您需要在调度程序中列入白名单支持。您可以通过以下方式执行此操作：

1. Creating a filter to allow the `/libs/granite/csrf/token.json` path;
1. Add the `CSRF-Token` header to the `clientheaders` section of the Dispatcher configuration.

## Prevent Clickjacking {#prevent-clickjacking}

To prevent clickjacking we recommend that you configure your webserver to provide the `X-FRAME-OPTIONS` HTTP header set to `SAMEORIGIN`.

For more [information on clickjacking please see the OWASP site](https://www.owasp.org/index.php/Clickjacking).

## Perform a Penetration Test {#perform-a-penetration-test}

Adobe强烈建议在开展生产之前对AEM基础结构执行渗透测试。

