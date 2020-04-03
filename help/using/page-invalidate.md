---
title: 使从 AEM 中缓存的页面失效
seo-title: 从Adobe AEM中使缓存的页面无效
description: 了解如何配置Dispatcher与AEM之间的交互，以确保有效的缓存管理。
seo-description: 了解如何配置Adobe AEM Dispatcher与AEM之间的交互以确保有效的缓存管理。
uuid: 66533299-55c0-4864-9beb-77e281af9359
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 79cd94be-a6bc-4d34-bfe9-393b4107925c
translation-type: tm+mt
source-git-commit: 85497651ce29c8564da4b52c60819a48b776af7b

---


# 使从 AEM 中缓存的页面失效 {#invalidating-cached-pages-from-aem}

将Dispatcher与AEM一起使用时，必须配置交互以确保有效的缓存管理。 根据您的环境，配置还可以提高性能。

## 设置AEM用户帐户 {#setting-up-aem-user-accounts}

默认用 `admin` 户帐户用于验证默认安装的复制代理。 您应该创建一个专用用户帐户以与复制代理一起使用。

有关详细信息，请参 [阅AEM安全核对清单的“配置复制和传输用户](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps) ”部分。

## 从创作环境中使调度程序缓存失效 {#invalidating-dispatcher-cache-from-the-authoring-environment}

AEM作者实例上的复制代理在发布页面时向Dispatcher发送缓存失效请求。 该请求会导致Dispatcher在发布新内容时最终刷新缓存中的文件。

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

请按照以下过程在AEM作者实例上配置复制代理，以在页面激活时使Dispatcher缓存失效：

1. 打开AEM工具控制台。(`https://localhost:4502/miscadmin#/etc`)
1. 在创作工具／复制／代理下打开所需的复制代理。 您可以使用默认安装的Dispatcher Flush代理。
1. 单击“编辑”，并在“设置”选项卡中确保选 **择“已** 启用”。

1. （可选）要启用别名或虚路径失效请求，请选择“别 **名更新** ”选项。
1. 在“传输”选项卡上，输入访问调度程序所需的URI。\
   如果您使用标准Dispatcher Flush代理，您可能需要更新主机名和端口；例如，https://&lt;*dispatcherHost*>:&lt;*portApache*>/dispatcher/invalidate.cache

   **注意：** 对于Dispatcher Flush代理，仅当使用基于路径的虚拟主机条目来区分农场时，才使用URI属性。 您使用此字段目标要失效的农场。 例如，农场#1的虚拟主机为， `www.mysite.com/path1/*` 而农场#2的虚拟主机为 `www.mysite.com/path2/*`。 您可以使用的URL `/path1/invalidate.cache` 目标第一个农场和 `/path2/invalidate.cache` 目标第二个农场。 有关详细信息，请参 [阅将Dispatcher与多个域一起使用](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 单击确定以激活代理。

或者，您也可以从 [AEM Touch UI访问和配置Dispatcher Flush代理](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/replication.html#ConfiguringaDispatcherFlushagent)。

有关如何启用对虚URL的访问的其他详细信息，请参 [阅启用对虚URL的访问](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls)。

>[!NOTE]
>
>用于刷新调度程序缓存的代理不必具有用户名和口令，但是，如果配置了这些用户名和密码，将使用基本身份验证发送。

此方法存在两个潜在问题：

* 必须可以从创作实例访问调度程序。 如果您的网络（例如防火墙）配置为限制两者之间的访问，则可能不是这种情况。

* 发布和缓存失效同时发生。 根据时间安排，用户可能会在从缓存中删除页面之后以及新页面发布之前请求该页面。 AEM现在返回旧页面，调度程序将再次缓存它。 这对于大型网站来说更为重要。

## 从发布实例使调度程序缓存失效 {#invalidating-dispatcher-cache-from-a-publishing-instance}

在某些情况下，可通过将缓存管理从创作环境转移到发布实例来提高性能。 随后，它将是在收到已发布页面时向Dispatcher发送缓存失效请求的发布环境(而非AEM创作环境)。

此类情况包括：

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* 防止Dispatcher与发布实例之间可能发生的时间冲突(请参阅“创作”环境 [中的“使Dispatcher缓存失效](#invalidating-dispatcher-cache-from-the-authoring-environment)”)。
* 该系统包括驻留在高性能服务器上的多个发布实例，并且只包括一个创作实例。

>[!NOTE]
>
>使用此方法的决定应由经验丰富的AEM管理员做出。

调度程序刷新由在发布实例上操作的复制代理控制。 但是，配置是在创作环境上进行的，然后通过激活代理来传输：

1. 打开AEM工具控制台。
1. 在发布时，在“工具”/“复制”/“代理”下打开所需的复制代理。 您可以使用默认安装的Dispatcher Flush代理。
1. 单击“编辑”，并在“设置”选项卡中确保选 **择“已** 启用”。
1. （可选）要启用别名或虚路径失效请求，请选择“别 **名更新** ”选项。
1. 在“传输”选项卡上，输入访问调度程序所需的URI。\
   如果您使用标准Dispatcher Flush代理，您可能需要更新主机名和端口；例如， `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **注意：** 对于Dispatcher Flush代理，仅当使用基于路径的虚拟主机条目来区分农场时，才使用URI属性。 您使用此字段目标要失效的农场。 例如，农场#1的虚拟主机为， `www.mysite.com/path1/*` 而农场#2的虚拟主机为 `www.mysite.com/path2/*`。 您可以使用的URL `/path1/invalidate.cache` 目标第一个农场和 `/path2/invalidate.cache` 目标第二个农场。 有关详细信息，请参 [阅将Dispatcher与多个域一起使用](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 对受影响的每个发布实例重复上述步骤。

配置完成后，当您从作者激活页面以进行发布时，此代理将启动标准复制。 日志包含指示来自发布服务器的请求的消息，与以下示例类似：

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## 手动使调度程序缓存失效 {#manually-invalidating-the-dispatcher-cache}

要在不激活页面的情况下使Dispatcher缓存失效（或刷新），可向调度程序发出HTTP请求。 例如，您可以创建一个AEM应用程序，该应用程序使管理员或其他应用程序能够刷新缓存。

HTTP请求导致Dispatcher从缓存中删除特定文件。 或者，调度程序随后会使用新副本刷新缓存。

### 删除缓存的文件 {#delete-cached-files}

发出HTTP请求，使Dispatcher从缓存中删除文件。 仅当调度程序收到对页面的客户端请求时，才会再次缓存文件。 以这种方式删除缓存的文件适用于不太可能同时收到同一页面请求的网站。

HTTP请求具有以下形式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

调度程序刷新（删除）名称与标题值匹配的缓存文件和文件 `CQ-Handler` 夹。 例如，一个 `CQ-Handle` 项 `/content/geomtrixx-outdoors/en` 与以下项匹配：

* 目录中命名的所有文件(任何文件扩展名 `en` 的文件 `geometrixx-outdoors` )

* en目录下名 `_jcr_content`为“”的任何目录（如果存在，则包含页面子节点的缓存呈现）

调度程序缓存中的所有其他文件（或者，根据设置，最高达到特定级别） `/statfileslevel` 通过触摸文件而失效 `.stat` 。 将此文件的上次修改日期与缓存的文档的上次修改日期进行比较，如果文件较新，则重新获取该 `.stat` 文档。 有关详 [细信息，请参阅按文件夹级别使文件失效](dispatcher-configuration.md#main-pars_title_26) 。

通过发送其他标题可以防止失效（即。stat文件的触摸） `CQ-Action-Scope: ResourceOnly`。 这可用于刷新特定资源而不使缓存的其他部分失效，例如动态创建并需要定期刷新的JSON数据（例如，表示从第三方系统获取的数据以显示新闻、股票行情等）。

### 删除和重新缓存文件 {#delete-and-recache-files}

发出HTTP请求，使Dispatcher删除缓存的文件，并立即检索并重新缓存文件。 当网站可能收到同一页面的同时客户端请求时，删除并立即重新缓存文件。 立即重新缓存可确保Dispatcher只检索并缓存页面一次，而不是同时为每个客户端请求一次。

**注意：** 应仅对发布实例执行删除和重新缓存文件。 当从作者实例执行时，当重新缓存资源尝试在发布之前发生时，会发生竞争条件。

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

到立即重新缓存的页面路径列在消息正文的单独行中。 值是使 `CQ-Handle` 要重新缓存的页面无效的页面路径。 (请参阅 `/statfileslevel` 缓存配置项 [的参数](dispatcher-configuration.md#main-pars_146_44_0010) 。)以下示例HTTP请求消息删除并重新缓存 `/content/geometrixx-outdoors/en.html page`:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### 刷新servlet示例 {#example-flush-servlet}

以下代码实现了向Dispatcher发送无效请求的Servlet。 Servlet接收包含参数和参数的请 `handle` 求 `page` 消息。 这些参数分别提供 `CQ-Handle` 要重新缓存的页面的标题和路径的值。 Servlet使用这些值为调度程序构建HTTP请求。

将servlet部署到发布实例时，以下URL会导致Dispatcher删除/content/geometrixx-outdoors/en.html页面，然后缓存新副本。

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>此示例servlet不安全，它仅演示了HTTP Post请求消息的使用。 您的解决方案应确保对servlet的访问安全。


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

