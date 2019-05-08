---
title: 缓存安全内容
seo-title: 在AEM Dispatcher中缓存安全内容
description: 了解权限敏感缓存在Dispatcher中的工作原理。
seo-description: 了解权限敏感缓存在AEM Dispatcher中的工作原理。
uuid: abded68a-2efe-45f6-bdf7-2284931629d6
contentOwner: 用户
products: SG_ EXPERIENCE MANAGER/Dispatcher
topic-tags: 调度程序
content-type: 引用
discoiquuid: 4f9b2bc8-a309-47bc-b70 d-a1 c0 da78 d464
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# 缓存安全内容 {#caching-secured-content}

权限敏感性缓存允许您缓存安全页面。Dispatcher在传送缓存页面之前检查用户对页面的访问权限。

调度程序包括用于实现权限敏感缓存的autheChker模块。激活模块时，渲染将调用AEM Servlet以对请求的内容执行用户身份验证和授权。servlet响应决定内容是否传递到Web浏览器。

由于身份验证和授权的方法特定于AEM部署，因此需要创建servlet。

>[!NOTE]
>
>使用 `deny` 过滤器来实施覆盖安全限制。对配置为允许对子集或用户组子集进行访问的页面使用权限敏感型缓存。

以下示意图说明当Web浏览器请求使用权限敏感缓存的页面时发生的事件顺序。

## 页面已缓存，用户已获得授权 {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. 调度程序确定已请求的内容已缓存并有效。
1. 调度程序向渲染发送请求消息。HEAD部分包括浏览器请求中的所有标题行。
1. 渲染调用authorizer执行安全检查并响应Dispatcher。响应消息包含HTTP状态代码200，表示用户已获得授权。
1. 调度程序向浏览器发送响应消息，该浏览器由渲染响应中的标题行和正文中缓存的内容组成。

## 页面未缓存，用户已获得授权 {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. 调度程序确定内容未缓存或需要更新。
1. Dispatcher将原始请求转发给渲染。
1. 渲染调用authorizer servlet以执行安全检查。用户获得授权后，渲染将包含响应消息正文中呈现的页面。
1. 调度程序将响应转发到浏览器。调度程序将渲染的响应消息的正文添加到缓存。

## 用户未获得授权 {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. 调度程序检查缓存。
1. 调度程序向渲染发送请求消息，其中包括浏览器请求中的所有标题行。
1. 渲染调用authorizer servlet以执行安全检查，它将失败，呈现将原始请求转发到调度程序。

## 实施权限敏感型缓存 {#implementing-permission-sensitive-caching}

要实施权限敏感型缓存，请执行以下任务：

* 开发执行身份验证和授权的servlet
* 配置调度程序

>[!NOTE]
>
>通常，安全资源存储在非安全文件的单独文件夹中。例如，/content/secure/


## 创建授权servlet {#create-the-authorization-servlet}

创建并部署一个servlet，它执行请求Web内容的用户的身份验证和授权。servlet可使用任何身份验证和授权方法，如AEM用户帐户和存储库ACL或LDAP查找服务。您将servlet部署到Dispatcher用作渲染的AEM实例。

所有用户都可访问该servlet。因此，servlet应该扩展类， `org.apache.sling.api.servlets.SlingSafeMethodsServlet` 它提供对系统的只读访问权限。

servlet只接收渲染的HEAD请求，因此您只需要实现 `doHead` 该方法。

渲染包含请求资源的URI作为HTTP请求的参数。例如，访问授权servlet `/bin/permissioncheck`。要在/content/geometrixx-outdoors/en.html页面上执行安全检查，渲染功能在HTTP请求中包括以下URL：

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

servlet响应消息必须包含以下HTTP状态代码：

* 200：已通过身份验证和授权。

以下示例servlet从HTTP请求获取请求资源的URL。代码使用Felix SCR `Property` 注释将 `sling.servlet.paths` 属性的值设置为/bin/permissioncheck。`doHead` 在该方法中，servlet获取会话对象，并使用 `checkPermission` 该方法确定相应的响应代码。

>[!NOTE]
>
>sling Servlet. paths属性的值必须在Sling Servlet Resolver(org. apache. sling. servlets. resolver. SlingServlResolver)服务中启用。

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

调度程序的auth_ checker部分。任何文件都控制权限敏感型缓存的行为。auth_ checker部分包括以下小节：

* `url`：执行安全检查的servlet `sling.servlet.paths` 属性的值。

* `filter`：指定应用权限敏感缓存的文件夹的过滤器。通常， `deny` 过滤器应用于所有文件夹， `allow` 过滤器会应用到安全文件夹。

* `headers`：指定授权servlet包含在响应中的HTTP头。

调度程序启动时，调度程序日志文件包括以下调试级别消息：

`AuthChecker: initialized with URL 'configured_url'.`

以下示例auth_ checker部分配置Dispatcher以使用prevoius主题的servlet。过滤器部分导致仅对安全HTML资源执行权限检查。

### 示例配置 {#example-configuration}

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

