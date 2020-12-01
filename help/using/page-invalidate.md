---
title: 使从 AEM 中缓存的页面失效
seo-title: 从AdobeAEM中使缓存页面失效
description: 了解如何配置Dispatcher与AEM之间的交互以确保有效的缓存管理。
seo-description: 了解如何配置AdobeAEM调度程序与AEM之间的交互以确保有效的缓存管理。
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
workflow-type: tm+mt
source-wordcount: '1427'
ht-degree: 0%

---


# 使从 AEM 中缓存的页面失效 {#invalidating-cached-pages-from-aem}

将Dispatcher与AEM一起使用时，必须配置交互以确保有效的缓存管理。 根据您的环境，配置还可以提高性能。

## 设置AEM用户帐户{#setting-up-aem-user-accounts}

默认的`admin`用户帐户用于验证默认安装的复制代理。 您应创建专用用户帐户以用于复制代理。

有关详细信息，请参阅AEM安全核对清单的[配置复制和传输用户](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps)部分。

## 从创作环境{#invalidating-dispatcher-cache-from-the-authoring-environment}中使调度程序缓存失效

在发布页面时，AEM作者实例上的复制代理向调度程序发送缓存失效请求。 此请求会导致Dispatcher最终在新内容发布时刷新缓存中的文件。

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

请按照以下过程在AEM作者实例上配置复制代理，以便在页面激活时使调度程序缓存失效：

1. 打开AEM工具控制台。(`https://localhost:4502/miscadmin#/etc`)
1. 在创作时的工具／复制／代理下打开所需的复制代理。 您可以使用默认安装的调度程序刷新代理。
1. 单击“编辑”，在“设置”选项卡中确保已选择&#x200B;**已启用**。

1. （可选）要启用别名或虚路径失效请求，请选择&#x200B;**别名更新**&#x200B;选项。
1. 在“传输”选项卡上，输入访问调度程序所需的URI。\
   如果您使用标准Dispatcher Flush代理，您可能需要更新主机名和端口；例如，https://&lt;*dispatcherHost*:&lt;*portApache*/dispatcher/invalidate.cache

   **注意：** 对于调度程序刷新代理，仅当使用基于路径的虚拟主机条目来区分场时，才使用URI属性。您使用此字段目标要失效的场。 例如，场#1的虚拟主机为`www.mysite.com/path1/*`，场#2的虚拟主机为`www.mysite.com/path2/*`。 可以使用`/path1/invalidate.cache`的URL目标第一个场，使用`/path2/invalidate.cache`目标第二个场。 有关详细信息，请参阅[将调度程序与多个域一起使用](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 单击确定以激活代理。

或者，您也可以从[AEM Touch UI](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/replication.html#ConfiguringaDispatcherFlushagent)访问和配置调度程序刷新代理。

有关如何启用对虚URL的访问的更多详细信息，请参阅[启用对虚URL的访问](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls)。

>[!NOTE]
>
>用于刷新调度程序缓存的代理不必具有用户名和密码，但如果配置了，将使用基本身份验证发送它们。

这种方法存在两个潜在问题：

* 必须可以从创作实例访问调度程序。 如果您的网络（例如防火墙）配置为限制两者之间的访问，则可能不是这种情况。

* 发布和缓存失效同时发生。 根据时间，用户可能在从缓存中删除某个页面后以及新页面发布之前请求该页面。 AEM现在返回旧页面，调度程序再次缓存它。 这对于大型网站来说更是一个问题。

## 从发布实例{#invalidating-dispatcher-cache-from-a-publishing-instance}中使调度程序缓存失效

在某些情况下，可以通过将缓存管理从创作环境转移到发布实例来提高性能。 接下来，将是发布环境(而非AEM创作环境)，在收到已发布页面时向Dispatcher发送缓存失效请求。

此类情况包括：

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* 防止Dispatcher与发布实例之间可能发生的时间冲突(请参阅“创作”环境](#invalidating-dispatcher-cache-from-the-authoring-environment)中的[使Dispatcher缓存失效)。
* 该系统包括驻留在高性能服务器上的多个发布实例，并且只包含一个创作实例。

>[!NOTE]
>
>应由经验丰富的AEM管理员决定使用此方法。

调度程序刷新由在发布实例上运行的复制代理控制。 但是，配置是在创作环境上进行的，然后通过激活代理来传输：

1. 打开AEM工具控制台。
1. 在发布时，在“工具／复制／代理”下打开所需的复制代理。 您可以使用默认安装的调度程序刷新代理。
1. 单击“编辑”，在“设置”选项卡中确保已选择&#x200B;**已启用**。
1. （可选）要启用别名或虚路径失效请求，请选择&#x200B;**别名更新**&#x200B;选项。
1. 在“传输”选项卡上，输入访问调度程序所需的URI。\
   如果您使用标准Dispatcher Flush代理，您可能需要更新主机名和端口；例如`http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **注意：** 对于调度程序刷新代理，仅当使用基于路径的虚拟主机条目来区分场时，才使用URI属性。您使用此字段目标要失效的场。 例如，场#1的虚拟主机为`www.mysite.com/path1/*`，场#2的虚拟主机为`www.mysite.com/path2/*`。 可以使用`/path1/invalidate.cache`的URL目标第一个场，使用`/path2/invalidate.cache`目标第二个场。 有关详细信息，请参阅[将调度程序与多个域一起使用](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 对受影响的每个发布实例重复上述步骤。

配置后，当您将页面从作者激活到发布时，此代理将启动标准复制。 日志包含指示来自发布服务器的请求的消息，与以下示例类似：

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## 手动使调度程序缓存{#manually-invalidating-the-dispatcher-cache}失效

要使调度程序缓存失效（或刷新）而不激活页面，可向调度程序发出HTTP请求。 例如，您可以创建AEM应用程序，使管理员或其他应用程序能够刷新缓存。

HTTP请求会导致调度程序从缓存中删除特定文件。 （可选）调度程序随后使用新副本刷新缓存。

### 删除缓存的文件{#delete-cached-files}

发出HTTP请求，使Dispatcher从缓存中删除文件。 调度程序仅在收到页面的客户端请求时才再次缓存文件。 以这种方式删除缓存的文件对于不太可能接收同一页面同时请求的网站是合适的。

HTTP请求具有以下形式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

调度程序刷新（删除）名称与`CQ-Handler`头值匹配的缓存文件和文件夹。 例如，`/content/geomtrixx-outdoors/en`的`CQ-Handle`与以下项匹配：

* `geometrixx-outdoors`目录中名为`en`的所有文件（任何文件扩展名的文件）

* en目录下名为“ `_jcr_content`”的任何目录（如果存在，则包含页子节点的缓存呈现）

调度程序缓存中的所有其他文件（或直到特定级别，取决于`/statfileslevel`设置）都通过触摸`.stat`文件而失效。 将此文件的上次修改日期与缓存文档的上次修改日期进行比较，如果`.stat`文件较新，则重新获取文档。 有关详细信息，请参阅[按文件夹级别](dispatcher-configuration.md#main-pars_title_26)取消验证文件。

通过发送额外的标头`CQ-Action-Scope: ResourceOnly`可以防止失效（即。stat文件的触摸）。 这可用于刷新特定资源而不使缓存的其他部分失效，如动态创建并需要定期刷新的JSON数据，而不依赖于缓存（例如，表示从第三方系统获取的数据以显示新闻、股票行情等）。

### 删除和重新缓存文件{#delete-and-recache-files}

发出HTTP请求，使Dispatcher删除缓存的文件，并立即检索并重新缓存文件。 当网站可能收到同一页面的同时客户端请求时，删除并立即重新缓存文件。 立即重新缓存可确保Dispatcher只检索并缓存页面一次，而不是对每个同步客户端请求一次。

**注意：** 仅应对发布实例执行删除和重新缓存文件。从作者实例执行时，当资源在发布之前尝试重新缓存时，会发生竞争条件。

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

要立即重新缓存的页面路径列在邮件正文的单独行中。 值`CQ-Handle`是使要重新缓存的页面无效的页面路径。 （请参阅[Cache](dispatcher-configuration.md#main-pars_146_44_0010)配置项的`/statfileslevel`参数。） 以下示例HTTP请求消息删除并重新缓存`/content/geometrixx-outdoors/en.html page`:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### 刷新servlet {#example-flush-servlet}示例

以下代码实现向Dispatcher发送无效请求的Servlet。 Servlet接收包含`handle`和`page`参数的请求消息。 这些参数分别提供`CQ-Handle`头的值和要重新缓存的页面的路径。 Servlet使用这些值为调度程序构建HTTP请求。

将Servlet部署到发布实例时，以下URL会导致Dispatcher删除/content/geometrixx-outdoors/en.html页面，然后缓存新副本。

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>此示例servlet不安全，它仅演示了HTTP Post请求消息的使用。 您的解决方案应确保对servlet的安全访问。


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

