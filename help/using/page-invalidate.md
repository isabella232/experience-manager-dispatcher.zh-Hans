---
title: 使从 AEM 中缓存的页面失效
seo-title: 使缓存的页面从AdobeAEM失效
description: 了解如何配置Dispatcher与AEM之间的交互，以确保有效的缓存管理。
seo-description: 了解如何配置AdobeAEM Dispatcher与AEM之间的交互，以确保有效的缓存管理。
uuid: 66533299-55c0-4864-9beb-77e281af9359
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 79cd94be-a6bc-4d34-bfe9-393b4107925c
exl-id: 90eb6a78-e867-456d-b1cf-f62f49c91851
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '1427'
ht-degree: 0%

---

# 使从 AEM 中缓存的页面失效 {#invalidating-cached-pages-from-aem}

将Dispatcher与AEM结合使用时，必须配置交互以确保有效的缓存管理。 根据您的环境，配置还可以提高性能。

## 设置AEM用户帐户{#setting-up-aem-user-accounts}

默认的`admin`用户帐户用于验证默认安装的复制代理。 您应该创建一个专用用户帐户，以便与复制代理一起使用。

有关详细信息，请参阅AEM安全检查表的[配置复制和传输用户](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps)部分。

## 使创作环境{#invalidating-dispatcher-cache-from-the-authoring-environment}中的Dispatcher缓存失效

页面发布后，AEM创作实例上的复制代理会向Dispatcher发送缓存失效请求。 请求会导致Dispatcher在发布新内容时最终刷新缓存中的文件。

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

请按照以下过程在AEM创作实例上配置复制代理，以在页面激活时使Dispatcher缓存失效：

1. 打开AEM工具控制台。(`https://localhost:4502/miscadmin#/etc`)
1. 在创作时的工具/复制/代理下打开所需的复制代理。 您可以使用默认安装的调度程序刷新代理。
1. 单击编辑，然后在“设置”选项卡中确保选中&#x200B;**已启用**。

1. （可选）要启用别名或虚路径失效请求，请选择&#x200B;**别名更新**&#x200B;选项。
1. 在“传输”选项卡上，输入访问Dispatcher所需的URI。\
   如果您使用标准Dispatcher刷新代理，则可能需要更新主机名和端口；例如， https://*dispatcherHost*>:*portApache*>/dispatcher/invalidate.cache

   **注意：** 对于调度程序刷新代理，仅当使用基于路径的虚拟主机条目来区分场时，才使用URI属性。使用此字段可将场定位为无效。 例如，场#1的虚拟主机为`www.mysite.com/path1/*`，场#2的虚拟主机为`www.mysite.com/path2/*`。 您可以使用`/path1/invalidate.cache`的URL来定位第一个场，使用`/path2/invalidate.cache`来定位第二个场。 有关更多信息，请参阅[将Dispatcher与多个域结合使用](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 单击确定以激活代理。

或者，您也可以从[AEM Touch UI](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/replication.html#ConfiguringaDispatcherFlushagent)访问和配置Dispatcher刷新代理。

有关如何启用对虚URL的访问的其他详细信息，请参阅[启用对虚URL的访问](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls)。

>[!NOTE]
>
>用于刷新调度程序缓存的代理不必具有用户名和密码，但如果配置，将通过基本身份验证发送它们。

此方法存在两个潜在问题：

* 必须从创作实例访问Dispatcher。 如果您的网络（例如防火墙）配置为限制两者之间的访问，则情况可能并非如此。

* 发布和缓存失效同时进行。 根据时间，用户可能会在从缓存中删除页面后以及发布新页面之前请求页面。 AEM现在会返回旧页面，Dispatcher会再次缓存该页面。 对于大型网站而言，这更是一个问题。

## 从发布实例{#invalidating-dispatcher-cache-from-a-publishing-instance}中使调度程序缓存失效

在某些情况下，可通过将缓存管理从创作环境转移到发布实例来提高性能。 随后，将是发布环境(而非AEM创作环境)，在收到已发布的页面时，向Dispatcher发送缓存失效请求。

此类情况包括：

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* 防止Dispatcher与发布实例之间可能存在的时间冲突（请参阅[从创作环境中使Dispatcher缓存失效](#invalidating-dispatcher-cache-from-the-authoring-environment)）。
* 该系统包含多个驻留在高性能服务器上的发布实例，并且只包含一个创作实例。

>[!NOTE]
>
>使用此方法的决策应由经验丰富的AEM管理员做出。

调度程序刷新由在发布实例上运行的复制代理控制。 但是，配置是在创作环境中进行的，然后通过激活代理进行转移：

1. 打开AEM工具控制台。
1. 在发布时打开工具/复制/代理下方的所需复制代理。 您可以使用默认安装的调度程序刷新代理。
1. 单击编辑，然后在“设置”选项卡中确保选中&#x200B;**已启用**。
1. （可选）要启用别名或虚路径失效请求，请选择&#x200B;**别名更新**&#x200B;选项。
1. 在“传输”选项卡上，输入访问Dispatcher所需的URI。\
   如果您使用标准Dispatcher刷新代理，则可能需要更新主机名和端口；例如，`http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **注意：** 对于调度程序刷新代理，仅当使用基于路径的虚拟主机条目来区分场时，才使用URI属性。使用此字段可将场定位为无效。 例如，场#1的虚拟主机为`www.mysite.com/path1/*`，场#2的虚拟主机为`www.mysite.com/path2/*`。 您可以使用`/path1/invalidate.cache`的URL来定位第一个场，使用`/path2/invalidate.cache`来定位第二个场。 有关更多信息，请参阅[将Dispatcher与多个域结合使用](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 对每个受影响的发布实例重复执行上述步骤。

配置后，在将页面从创作激活到发布时，此代理将启动标准复制。 日志包含指示来自发布服务器的请求的消息，如下例所示：

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## 手动使调度程序缓存失效{#manually-invalidating-the-dispatcher-cache}

要在不激活页面的情况下使Dispatcher缓存失效（或刷新），您可以向Dispatcher发出HTTP请求。 例如，您可以创建一个AEM应用程序，使管理员或其他应用程序能够刷新缓存。

HTTP请求会导致Dispatcher从缓存中删除特定文件。 （可选）然后，调度程序使用新副本刷新缓存。

### 删除缓存的文件{#delete-cached-files}

发出HTTP请求，导致Dispatcher从缓存中删除文件。 仅当Dispatcher收到页面的客户端请求时，才会再次缓存文件。 以这种方式删除缓存文件对于不太可能同时收到同一页面请求的网站是合适的。

HTTP请求具有以下形式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher刷新（删除）名称与`CQ-Handler`标头值匹配的缓存文件和文件夹。 例如，`/content/geomtrixx-outdoors/en`的`CQ-Handle`与以下项目匹配：

* `geometrixx-outdoors`目录中名为`en`的所有文件（任何文件扩展名的）

* en目录下方名为“ `_jcr_content`”的任何目录（如果存在，则包含页面子节点的缓存渲染）

通过触摸`.stat`文件，调度程序缓存中的所有其他文件（或达到特定级别，取决于`/statfileslevel`设置）将失效。 此文件的上次修改日期将与缓存文档的上次修改日期进行比较，如果`.stat`文件较新，则会重新获取文档。 有关详细信息，请参阅[按文件夹级别使文件失效](dispatcher-configuration.md#main-pars_title_26)。

通过发送额外的标头`CQ-Action-Scope: ResourceOnly`，可以阻止失效（即.stat文件的处理）。 这可用于刷新特定资源而不使缓存的其他部分失效，例如动态创建的JSON数据，它需要独立于缓存进行定期刷新（例如，表示从第三方系统获取的用于显示新闻的数据、股票报价等）。

### 删除和重新缓存文件{#delete-and-recache-files}

发出HTTP请求，导致Dispatcher删除缓存文件，并立即检索和重新缓存文件。 当网站可能收到同一页面的同时客户端请求时，请删除并立即重新缓存文件。 立即重新缓存可确保Dispatcher只检索和缓存一次页面，而不是针对每个同时发送的客户端请求一次。

**注意：** 只应对发布实例执行删除和重新缓存文件操作。从创作实例执行时，如果在资源发布之前尝试重新缓存资源，则会出现争用情况。

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

要立即重新缓存的页面路径在消息正文的单独行中列出。 值`CQ-Handle`是使要缓存的页面失效的页面路径。 （请参阅[Cache](dispatcher-configuration.md#main-pars_146_44_0010)配置项的`/statfileslevel`参数。） 以下HTTP请求消息示例删除并重新缓存`/content/geometrixx-outdoors/en.html page`:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### 示例刷新servlet {#example-flush-servlet}

以下代码实现一个Servlet，该Servlet向Dispatcher发送无效请求。 Servlet接收包含`handle`和`page`参数的请求消息。 这些参数分别提供`CQ-Handle`标头值和要缓存的页面路径。 Servlet使用值构建Dispatcher的HTTP请求。

将Servlet部署到发布实例后，以下URL会导致Dispatcher删除/content/geometrixx-outdoors/en.html页面，然后缓存新副本。

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>此示例Servlet不安全，仅演示了HTTP Post请求消息的使用。 您的解决方案应该能够安全地访问Servlet。


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
