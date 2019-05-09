---
title: 从AEM中使缓存页面失效
seo-title: 从Adobe AEM使缓存页面失效
description: 了解如何配置Dispatcher和AEM之间的交互以确保有效的缓存管理。
seo-description: 了解如何配置Adobe AEM Dispatcher和AEM之间的交互以确保有效的缓存管理。
uuid: 66533299-55c0-4864-9Beb-77e281af9359
cmgrlastmodified: 01.11.2007082229[aheimoz]
pageversionid: '1193211344162'
template: /apps/docs/templates/contentpage
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/Dispatcher
topic-tags: 调度程序
content-type: 引用
discoiquuid: 79cd94be-a6 bc-4d34-bfe9-393b4107925 c
translation-type: tm+mt
source-git-commit: 76cffbfb616cd5601aed36b7076f67a2faf3ed3b

---


# 从AEM中使缓存页面失效 {#invalidating-cached-pages-from-aem}

在将调度程序与AEM一起使用时，必须配置交互以确保有效地进行缓存管理。根据您的环境，配置还可以提高性能。

## 设置AEM用户帐户 {#setting-up-aem-user-accounts}

默认 `admin` 用户帐户用于验证默认安装的复制代理。您应创建专用用户帐户以与复制代理一起使用。 [](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps)

有关详细信息，请参阅AEM安全核对清单 [的配置复制和传输用户](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps) 部分。

## 使创作环境中的调度程序缓存失效 {#invalidating-dispatcher-cache-from-the-authoring-environment}

在发布页面时，AEM作者实例上的复制代理将缓存失效请求发送到调度程序。此请求导致调度程序最终在新内容发布时刷新缓存中的文件。

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-0436B4A35714BFF67F000101@AdobeID)
Last Modified Date: 2017-05-25T10:37:23.679-0400

<p>Hiding this information due to <a href="https://jira.corp.adobe.com/browse/CQDOC-5805">CQDOC-5805</a>.</p>

 -->

请按照以下过程在AEM作者实例上配置复制代理，以便在页面激活后失效调度程序缓存：

1. 打开AEM工具控制台。`https://localhost:4502/miscadmin#/etc`()
1. 在作者的工具/复制/代理下打开所需的复制代理。您可以使用默认安装的调度程序刷新代理。
1. 单击编辑，然后在设置选项卡中确保已选中 **已启用** 。

1. (可选)要启用别名或虚路径失效请求，请选择 **别名更新** 选项。
1. 在传输选项卡上，输入访问Dispatcher所需的URI。\
   如果您使用的是标准调度程序刷新代理，您很可能需要更新主机名和端口；例如，https://&lt;*DispatcherHost*&gt;：&lt;*portApache*&gt;/dispatcher/invalidate. cache

   **注意：** 对于调度程序刷新代理，仅当您使用基于路径的虚拟化主机条目来区分农场时，才使用URI属性。您使用此字段将农场定位为无效。例如，农场#有虚拟主机， `www.mysite.com/path1/*` 农场#有虚拟主机 `www.mysite.com/path2/*`。您可以使用URL `/path1/invalidate.cache` 定位第一个农场并 `/path2/invalidate.cache` 瞄准第二个农场。有关详细信息，请参阅 [使用Dispatcher和多个域](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 单击确定以激活代理。

或者，也可以从 [AEM Touch UI访问和配置调度程序刷新代理](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/replication.html#ConfiguringaDispatcherFlushagent)。

有关如何启用虚URL的其他详细信息，请参阅 [启用访问虚URL](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls)。

>[!NOTE]
>
>用于刷新调度程序缓存的代理无需使用用户名和口令，但如果配置了该代理，则将通过基本身份验证发送。

这种方法有两个潜在问题：

* Dispatcher必须可从创作实例中访问。如果您的网络(例如防火墙)被配置为在两者之间访问，则可能不是这种情况。

* 发布和缓存无效同时发生。根据时间的不同，用户可以直接在从缓存中删除页面，以及在新页面发布之前请求页面。AEM现在返回旧页面，Dispatcher再次缓存它。这是大型站点的一个问题。

## 使发布实例中的调度程序缓存失效 {#invalidating-dispatcher-cache-from-a-publishing-instance}

在某些情况下，可以通过将缓存管理从创作环境传输到发布实例来实现性能提升。随后将作为接收已发布页面时向调度程序发送缓存失效请求的发布环境(而非AEM创作环境)。

这种情况包括：

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* 防止调度程序与发布实例之间发生时间冲突(请参阅 [创作环境中的无效调度程序缓存](#invalidating-dispatcher-cache-from-the-authoring-environment))。
* 该系统包括多个在高性能服务器上驻留的发布实例，而只有一个创作实例。

>[!NOTE]
>
>使用此方法的决定应由经验丰富的AEM管理员决定。

调度程序刷新由在发布实例上操作的复制代理控制。但是，该配置在创作环境中进行，然后通过激活代理进行传输：

1. 打开AEM工具控制台。
1. 在发布时打开工具/复制/代理下所需的复制代理。您可以使用默认安装的调度程序刷新代理。
1. 单击编辑，然后在设置选项卡中确保已选中 **已启用** 。
1. (可选)要启用别名或虚路径失效请求，请选择 **别名更新** 选项。
1. 在传输选项卡上，输入访问Dispatcher所需的URI。\
   如果您使用的是标准调度程序刷新代理，您很可能需要更新主机名和端口；例如， `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **注意：** 对于调度程序刷新代理，仅当您使用基于路径的虚拟化主机条目来区分农场时，才使用URI属性。您使用此字段将农场定位为无效。例如，农场#有虚拟主机， `www.mysite.com/path1/*` 农场#有虚拟主机 `www.mysite.com/path2/*`。您可以使用URL `/path1/invalidate.cache` 定位第一个农场并 `/path2/invalidate.cache` 瞄准第二个农场。有关详细信息，请参阅 [使用Dispatcher和多个域](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 对受影响的每个发布实例重复上述操作。

配置后，当您将页面从作者激活到发布之后，此代理将启动标准复制。日志包含指示来自发布服务器请求的消息，类似于以下示例：

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## 手动失效调度程序缓存 {#manually-invalidating-the-dispatcher-cache}

要使调度程序缓存失效(或刷新)而不激活页面，您可以向调度程序发出HTTP请求。例如，您可以创建一个AEM应用程序，它使管理员或其他应用程序能够刷新缓存。

HTTP请求导致调度程序从缓存中删除特定文件。或者，Dispatcher可使用新的副本refre缓存。

### 删除缓存的文件 {#delete-cached-files}

发出导致调度程序从缓存中删除文件的HTTP请求。仅当调度程序收到页面请求时，调度程序才会再次缓存这些文件。以此方式删除缓存的文件适用于不可能接收同一页面并发请求的网站。

HTTP请求具有以下形式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

调度程序fluder(删除)缓存的文件和文件夹，它们的名称与 `CQ-Handler` 标题的值匹配。例如，一个 `CQ-Handle``/content/geomtrixx-outdoors/en` 匹配以下项目：

* 目录中命名 `en` 的所有文件(包括任何文件 `geometrixx-outdoors` 扩展名)

* 位于en目录之下的任何目录( `_jcr_content`如果它存在，则包含页面子节点的缓存的演绎版)

调度程序缓存中的所有其他文件(或最多一个特定级别的 `/statfileslevel` 文件)都通过触摸 `.stat` 文件无效。将该文件的上次修改日期与缓存文档的上次修改日期进行比较，如果 `.stat` 文件较新，则重新访存文档。有关 [详细信息，请参阅按文件夹级别](dispatcher-configuration.md#main-pars_title_26) 使文件失效。

无法通过发送额外的Header来阻止失效(即touch. stat文件) `CQ-Action-Scope: ResourceOnly`。这可用于刷新特定的资源，而不会使缓存的其他部分失效，如动态创建的JSON数据，并且需要与缓存定期刷新(例如，表示从第三方系统获取的数据以显示新闻、股票行情等)。

### 删除和重新缓存文件 {#delete-and-recache-files}

发出一个HTTP请求，它导致调度程序删除缓存的文件，并立即检索和重新缓存该文件。当网站可能收到同一页面并发请求时，删除并立即重新缓存文件。即时重新截取可确保Dispatcher只检索一次并缓存页面一次，而不是针对同时客户端请求中的每一个请求一次。

**注意：** 删除和重新缓存文件应仅在发布实例上执行。当从作者实例执行时，在资源发布之前尝试重新缓存资源时会发生种族条件。

HTTP请求具有以下形式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate   
`Content-Type: text/plain  
CQ-Handle: path-pattern  
Content-Length: numchars in bodypage_path0

page_path1 
...  
page_pathn
```

要立即重复使用的页面路径列在邮件正文的单独行中。值是 `CQ-Handle` 使页面失效的页面路径。(请参阅 `/statfileslevel`[缓存](dispatcher-configuration.md#main-pars_146_44_0010) 配置项目的参数。)以下示例HTTP请求消息删除并重新缓存 `/content/geometrixx-outdoors/en.html page`：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### 刷新servlet示例 {#example-flush-servlet}

以下代码实现了一个servlet，它将无效请求发送到调度程序。servlet收到一条包含 `handle` 和 `page` 参数的请求消息。这些参数分别提供 `CQ-Handle` 标题的值和页面的路径要重新缓存。servlet使用这些值构建Dispatcher的HTTP请求。

将servlet部署到发布实例后，以下URL会导致Dispatcher删除/content/geometrixx-outdoors/en.html页面，然后缓存一个新副本。

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>此示例servlet不安全，只演示了HTTP帖子请求消息的使用。您的解决方案应安全地访问servlet。


```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.apache.commons.httpclient.*;
import org.apache.commons.httpclient.methods.PostMethod;
import org.apache.commons.httpclient.methods.StringRequestEntity;

@Component(metatype=true)
@Service
public class Flushcache extends SlingSafeMethodsServlet {

 @Property(value="/bin/flushcache")
 static final String SERVLET_PATH="sling.servlet.paths";

 private Logger logger = LoggerFactory.getLogger(this.getClass());

 public void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response) {
  try{ 
      //retrieve the request parameters
      String handle = request.getParameter("handle");
      String page = request.getParameter("page");

      //hard-coding connection properties is a bad practice, but is done here to simplify the example
      String server = "localhost"; 
      String uri = "/dispatcher/invalidate.cache";

      HttpClient client = new HttpClient();

      PostMethod post = new PostMethod("https://"+host+uri);
      post.setRequestHeader("CQ-Action", "Activate");
      post.setRequestHeader("CQ-Handle",handle);
   
      StringRequestEntity body = new StringRequestEntity(page,null,null);
      post.setRequestEntity(body);
      post.setRequestHeader("Content-length", String.valueOf(body.getContentLength()));
      client.executeMethod(post);
      post.releaseConnection();
      //log the results
      logger.info("result: " + post.getResponseBodyAsString());
      }
  }catch(Exception e){
      logger.error("Flushcache servlet exception: " + e.getMessage());
  }
 }
}
```

