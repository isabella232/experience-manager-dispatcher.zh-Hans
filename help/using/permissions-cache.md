---
title: 缓存安全内容
seo-title: 在AEM Dispatcher中缓存受保护的内容
description: 了解权限敏感型缓存在Dispatcher中的工作方式。
seo-description: 了解权限敏感型缓存在AEM Dispatcher中的工作方式。
uuid: abfed68a-2efe-45f6-bdf7-2284931629d6
contentOwner: 用户
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: 调度程序
content-type: 引用
discoiquuid: 4f9b2bc8-a309-47bc-b70d-a1c0da78d464
translation-type: tm+mt
source-git-commit: 8dd56f8b90331f0da43852e25893bc6f3e606a97

---


# 缓存安全内容 {#caching-secured-content}

对权限敏感的缓存允许您缓存受保护的页面。 在传送缓存页面之前，调度程序会检查用户对页面的访问权限。

Dispatcher包括AuthChecker模块，用于实现对权限敏感的缓存。 激活模块后，渲染器将调用AEM servlet以对所请求的内容执行用户身份验证和授权。 Servlet响应确定内容是否被传送到Web浏览器。

由于身份验证和授权方法特定于AEM部署，因此您需要创建servlet。

>[!NOTE]
>
>使用 `deny` 过滤器来实施一揽子安全限制。 对配置为允许访问某个用户或用户组子集的页面使用权限敏感型缓存。

下图说明了当Web浏览器请求使用权限敏感型缓存的页面时发生的事件的顺序。

## 页面已缓存，用户已获得授权 {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. 调度程序确定所请求的内容已缓存且有效。
1. 调度程序向渲染器发送请求消息。 HEAD部分包括浏览器请求中的所有标题行。
1. 渲染器调用授权者执行安全检查并响应调度程序。 响应消息包括HTTP状态代码200，以指示用户已获得授权。
1. 调度程序向浏览器发送响应消息，该消息由呈现响应中的标题行和主体中缓存的内容组成。

## 页面未缓存且用户已授权 {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. 调度程序确定内容未缓存或需要更新。
1. 调度程序将原始请求转发到渲染器。
1. 渲染器调用授权器servlet以执行安全检查。 当用户被授权时，渲染器在响应消息的正文中包括渲染的页面。
1. 调度程序将响应转发到浏览器。 调度程序将渲染的响应消息的正文添加到缓存中。

## 用户未获得授权 {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. 调度程序检查缓存。
1. 调度程序向渲染器发送请求消息，该消息包括来自浏览器请求的所有标题行。
1. 渲染器调用授权器servlet以执行安全检查，该检查失败，渲染器将原始请求转发给调度程序。

## 实施权限敏感型缓存 {#implementing-permission-sensitive-caching}

要实现对权限敏感的缓存，请执行以下任务：

* 开发执行身份验证和授权的Servlet
* 配置调度程序

>[!NOTE]
>
>通常，安全资源存储在非安全文件之外的单独文件夹中。 例如，/content/secure/


## 创建授权servlet {#create-the-authorization-servlet}

创建和部署Servlet，它执行请求Web内容的用户的身份验证和授权。 Servlet可以使用任何身份验证和授权方法，如AEM用户帐户和存储库ACL或LDAP查找服务。 您将servlet部署到Dispatcher用作渲染的AEM实例。

Servlet必须可供所有用户访问。 因此，Servlet应扩展类， `org.apache.sling.api.servlets.SlingSafeMethodsServlet` 该类提供对系统的只读访问。

Servlet只接收呈现器中的HEAD请求，因此您只需实现该方 `doHead` 法。

渲染器包含所请求资源的URI作为HTTP请求的参数。 例如，通过访问授权servlet `/bin/permissioncheck`。 要对/content/geometrixx-outdoors/en.html页面执行安全检查，渲染器在HTTP请求中包括以下URL:

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

servlet响应消息必须包含以下HTTP状态代码：

* 200:通过身份验证和授权。

以下示例servlet从HTTP请求中获取所请求资源的URL。 代码使用Felix SCR注 `Property` 释将属性的值设 `sling.servlet.paths` 置为/bin/permissioncheck。 在该方 `doHead` 法中，servlet获取会话对象，并使用该方 `checkPermission` 法确定相应的响应代码。

>[!NOTE]
>
>必须在Sling Servlet解析器(org.apache.sling.servlets.resolver.SlingServletResolver)服务中启用sling.servlet.paths属性的值。

### 示例servlet {#example-servlet}

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

import javax.jcr.Session;

@Component(metatype=false)
@Service
public class AuthcheckerServlet extends SlingSafeMethodsServlet {
 
    @Property(value="/bin/permissioncheck")
    static final String SERVLET_PATH="sling.servlet.paths";
    
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    
    public void doHead(SlingHttpServletRequest request, SlingHttpServletResponse response) {
     try{ 
      //retrieve the requested URL
      String uri = request.getParameter("uri");
      //obtain the session from the request
      Session session = request.getResourceResolver().adaptTo(javax.jcr.Session.class);     
      //perform the permissions check
      try {
       session.checkPermission(uri, Session.ACTION_READ);
       logger.info("authchecker says OK");
       response.setStatus(SlingHttpServletResponse.SC_OK);
      } catch(Exception e) {
       logger.info("authchecker says READ access DENIED!");
       response.setStatus(SlingHttpServletResponse.SC_FORBIDDEN);
      }
     }catch(Exception e){
      logger.error("authchecker servlet exception: " + e.getMessage());
     }
    }
}
```

## 为权限敏感型缓存配置Dispatcher {#configure-dispatcher-for-permission-sensitive-caching}

dispatcher.any文件的auth_checker部分控制对权限敏感的缓存行为。 auth_checker部分包括以下子部分：

* `url`:执行安全检 `sling.servlet.paths` 查的servlet的属性的值。

* `filter`:指定将权限敏感型缓存应用到的文件夹的过滤器。 通常，过滤器 `deny` 应用于所有文件夹，过滤器 `allow` 应用于受保护的文件夹。

* `headers`:指定授权servlet在响应中包含的HTTP头。

当Dispatcher启动时，Dispatcher日志文件包含以下调试级别消息：

`AuthChecker: initialized with URL 'configured_url'.`

以下auth_checker示例部分将Dispatcher配置为使用上一个主题的servlet。 过滤器部分导致仅对安全HTML资源执行权限检查。

### 配置示例 {#example-configuration}

```xml
/auth_checker
  {
  # request is sent to this URL with '?uri=<page>' appended
  /url "/bin/permissioncheck"
      
  # only the requested pages matching the filter section below are checked,
  # all other pages get delivered unchecked
  /filter
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "/content/secure/*.html"
      /type "allow"
      }
    }
  # any header line returned from the auth_checker's HEAD request matching
  # the section below will be returned as well
  /headers
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "Set-Cookie:*"
      /type "allow"
      }
    }
  }
```

