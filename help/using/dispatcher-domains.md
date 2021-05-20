---
title: '在多个域中使用 Dispatcher '
seo-title: '在多个域中使用 Dispatcher '
description: 了解如何使用Dispatcher处理多个Web域中的页面请求。
seo-description: 了解如何使用Dispatcher处理多个Web域中的页面请求。
uuid: 7342a1c2-fe61-49be-a240-b487d53c7ec1
contentOwner: User
cq-exporttemplate: /etc/contentsync/templates/geometrixx/page/rewrite
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 40d91d66-c99b-422d-8e61-c0ced23272ef
exl-id: 1470b636-7e60-48cc-8c31-899f8785dafa
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '2983'
ht-degree: 0%

---

# 在多个域中使用 Dispatcher {#using-dispatcher-with-multiple-domains}

>[!NOTE]
>
>各个 Dispatcher 版本与 AEM 相互独立。如果您点击指向AEM或CQ文档中嵌入的Dispatcher文档的链接，可能会被重定向到此页面。

在支持以下条件的同时，使用Dispatcher在多个Web域中处理页面请求：

* 两个域的Web内容都存储在单个AEM存储库中。
* Dispatcher缓存中的文件可针对每个域单独失效。

例如，一家公司为其两个品牌发布网站：品牌A和品牌B。网站页面的内容在AEM中创作，并存储在同一存储库工作区中：

```
/
| - content  
   | - sitea  
       | - content nodes  
   | - siteb  
       | - content nodes
```

`BrandA.com`的页面存储在`/content/sitea`下方。 在`/content/sitea/en`节点的呈现页面中返回URL `https://BrandA.com/en.html`的客户端请求。 同样，`BrandB.com`的页面存储在`/content/siteb`下方。

使用Dispatcher缓存内容时，必须在客户端HTTP请求中的页面URL、缓存中相应文件的路径以及存储库中相应文件的路径之间进行关联。

## 客户端请求

当客户端向Web服务器发送HTTP请求时，必须将所请求页面的URL解析为Dispatcher缓存中的内容，并最终解析为存储库中的内容。

![](assets/chlimage_1-8.png)

1. 域名系统可发现在HTTP请求中为域名注册的Web服务器的IP地址。
1. HTTP请求将发送到Web服务器。
1. HTTP请求将传递到Dispatcher。
1. Dispatcher确定缓存的文件是否有效。 如果有效，则会将缓存的文件提供给客户端。
1. 如果缓存的文件无效，则Dispatcher从AEM发布实例请求新渲染的页面。

## 缓存失效

当Dispatcher刷新复制代理请求Dispatcher使缓存文件失效时，存储库中内容的路径必须解析为缓存中的内容。

![](assets/chlimage_1-9.png)

1. 页面在AEM创作实例上激活，内容会复制到发布实例。
1. 调度程序刷新代理调用调度程序以使复制内容的缓存失效。
1. Dispatcher会处理一个或多个.stat文件，以使缓存文件失效。

要将Dispatcher与多个域一起使用，您需要配置AEM、Dispatcher和Web服务器。 本页中介绍的解决方案是常规的，并且适用于大多数环境。 由于某些AEM拓扑的复杂性，您的解决方案可能需要进一步的自定义配置来解决特定问题。 您可能需要调整示例以满足您现有的IT基础架构和管理策略。

## URL映射{#url-mapping}

要使域URL和内容路径能够解析为缓存文件，在过程中的某个时刻必须翻译文件路径或页面URL。 提供了以下常见策略的描述，其中路径或URL转换在过程中的不同点发生：

* （推荐）AEM发布实例使用Sling映射来解决资源问题，以实施内部URL重写规则。 域URL已转换为内容存储库路径。 请参阅[AEM重写传入URL](#aem-rewrites-incoming-urls)。
* Web服务器使用内部URL重写规则，将域URL转换为缓存路径。 请参阅[Web服务器重写传入的URL](#the-web-server-rewrites-incoming-urls)。

通常需要为网页使用短URL。 通常，页面URL会镜像包含Web内容的存储库文件夹的结构。 但是，URL不会显示最顶部的存储库节点，如`/content`。 客户端不一定了解AEM存储库的结构。

## 一般要求{#general-requirements}

您的环境必须实施以下配置来支持Dispatcher使用多个域：

* 每个域的内容位于存储库的单独分支中（请参阅下面的示例环境）。
* 在AEM发布实例上配置调度程序刷新复制代理。 （请参阅[从发布实例中使Dispatcher缓存失效](page-invalidate.md)。）
* 域名系统将域名解析为Web服务器的IP地址。
* Dispatcher缓存镜像AEM内容存储库的目录结构。 Web服务器文档根目录下的文件路径与存储库中文件的路径相同。

## 提供的示例{#environment-for-the-provided-examples}的环境

提供的示例解决方案适用于具有以下特征的环境：

* AEM创作和发布实例部署在Linux系统上。
* Apache HTTPD是部署在Linux系统上的Web服务器。
* AEM内容存储库和Web服务器的文档根目录使用以下文件结构(Apache Web服务器的文档根目录为/`usr/lib/apache/httpd-2.4.3/htdocs)`:

   **存储库**

```
  | - /content  
    | - sitea  
  |    | - content nodes 
    | - siteb  
       | - conent nodes
```

**Web服务器的文档根目录**

```
  | - /usr  
    | - lib  
      | - apache  
        | - httpd-2.4.3  
          | - htdocs  
            | - content  
              | - sitea  
                 | - content nodes 
              | - siteb  
                 | - content nodes
```

## AEM重写传入URL {#aem-rewrites-incoming-urls}

用于资源解析的Sling映射允许您将传入URL与AEM内容路径关联。 在AEM发布实例上创建映射，以便从Dispatcher渲染请求解析到存储库中的正确内容。

调度程序页面渲染请求使用从Web服务器传递的URL来标识页面。 当URL包含域名时，Sling映射会将URL解析为内容。 下图说明了`branda.com/en.html` URL到`/content/sitea/en`节点的映射。

![](assets/chlimage_1-10.png)

Dispatcher缓存镜像存储库节点结构。 因此，当发生页面激活时，导致的无效编辑缓存页面的请求不需要URL或路径转换。

![](assets/chlimage_1-11.png)

## 在Web服务器{#define-virtual-hosts-on-the-web-server}上定义虚拟主机

在Web服务器上定义虚拟主机，以便可以将不同的文档根目录分配给每个Web域：

* Web服务器必须为您的每个Web域定义一个虚拟域。
* 对于每个域，将文档根配置为与包含域Web内容的存储库中的文件夹一致。
* 每个虚拟域还必须包含与Dispatcher相关的配置，如[安装Dispatcher](dispatcher-install.md)页面中所述。

以下示例`httpd.conf`文件为Apache Web服务器配置了两个虚拟域：

* 服务器名称（与域名一致）是branda.com（第16行）和brandb.com（第30行）。
* 每个虚拟域的文档根目录是Dispatcher缓存中包含站点页面的目录。 （第17和31行）

通过此配置，Web服务器在收到`https://branda.com/en/products.html`请求时会执行以下操作：

* 将URL与虚拟主机关联，虚拟主机的`ServerName`为`branda.com.`

* 将URL转发到Dispatcher。

### httpd.conf {#httpd-conf}

```xml
# load the Dispatcher module
LoadModule dispatcher_module modules/mod_dispatcher.so
# configure the Dispatcher module
<IfModule disp_apache2.c>
 DispatcherConfig conf/dispatcher.any
 DispatcherLog    logs/dispatcher.log  
 DispatcherLogLevel 3
 DispatcherNoServerHeader 0 
 DispatcherDeclineRoot 0
 DispatcherUseProcessedURL 0
 DispatcherPassError 0
</IfModule>

# Define virtual host for brandA.com
<VirtualHost *:80>
  ServerName branda.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# define virtual host for brandB.com
<VirtualHost *:80>
  ServerName brandB.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# document root for web server
DocumentRoot "/usr/lib/apache/httpd-2.4.3/htdocs"
```

请注意，虚拟主机将继承在主服务器部分中配置的[DispatcherConfig](dispatcher-install.md#main-pars-67-table-7)属性值。 虚拟主机可以包含其自己的DispatcherConfig属性以覆盖主服务器配置。

### 配置Dispatcher以处理多个域{#configure-dispatcher-to-handle-multiple-domains}

要支持包含域名及其相应虚拟主机的URL，请定义以下调度程序场：

* 为每个虚拟主机配置一个Dispatcher场。 这些场会处理来自Web服务器的每个域的请求、检查缓存文件以及从呈现器中请求页面。
* 配置用于使缓存内容失效的Dispatcher场，而不考虑内容属于哪个域。 此场处理来自刷新调度程序复制代理的文件失效请求。

### 为虚拟主机创建调度程序场

虚拟主机的场必须具有以下配置，以便客户端HTTP请求中的URL能够解析为Dispatcher缓存中的正确文件：

* 将`/virtualhosts`属性设置为域名。 此属性使Dispatcher能够将场与域关联。
* `/filter`属性允许访问在域名部分之后截断的请求URL的路径。 例如，对于`https://branda.com/en.html` URL，路径被解释为`/en.html`，因此过滤器必须允许访问此路径。

* 将`/docroot`属性设置为Dispatcher缓存中域站点内容根目录的路径。 此路径用作原始请求中连接的URL的前缀。 例如，`/usr/lib/apache/httpd-2.4.3/htdocs/sitea`的docroot会使`https://branda.com/en.html`请求解析到`/usr/lib/apache/httpd-2.4.3/htdocs/sitea/en.html`文件。

此外，必须将AEM发布实例指定为虚拟主机的呈现器。 根据需要配置其他场属性。 以下代码是branda.com域的简化场配置：

```xml
/farm_sitea  {     
    ...
    /virtualhosts { "branda.com" }
    /renders {
      /rend01  { /hostname "127.0.0.1"  /port "4503" }
    }
    /filter {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/en*" }  
      ...
     }
    /cache {
      /docroot "/usr/lib/apache/httpd-2.4.3/htdocs/content/sitea"
      ...
   }
   ...
}
```

### 创建用于缓存失效的Dispatcher场

处理使缓存文件失效的请求需要Dispatcher场。 此场必须能够访问每个虚拟主机的docroot目录中的.stat文件。

以下属性配置使Dispatcher能够从缓存中的文件解析AEM内容存储库中的文件：

* 将`/docroot`属性设置为Web服务器的默认docroot。 通常，这是创建`/content`文件夹的目录。 Linux上Apache的示例值为`/usr/lib/apache/httpd-2.4.3/htdocs`。
* `/filter`属性允许访问`/content`目录下的文件。

`/statfileslevel`属性必须足够高，以便在每个虚拟主机的根目录中创建.stat文件。 此属性可使每个域的缓存单独失效。 对于示例设置，`/statfileslevel`值`2`会在`*docroot*/content/sitea`目录和`*docroot*/content/siteb`目录中创建.stat文件。

此外，必须将发布实例指定为虚拟主机的呈现器。 根据需要配置其他场属性。 以下代码是用于使缓存失效的场的简化配置：

```xml
/farm_flush {  
    ...
    /virtualhosts   { "invalidation_only" }
    /renders  {
      /rend01  { /hostname "127.0.0.1" /port "4503" }
    }
    /filter   {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/content*" } 
      ...
      }
    /cache  {
       /docroot "/usr/lib/apache/httpd-2.4.3/htdocs"
       /statfileslevel "2"
       ...
   }
   ...
}
```

启动Web服务器时，调度程序日志（在调试模式下）指示所有场的初始化：

```shell
Dispatcher initializing (build 4.1.2)
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_sitea].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_siteb].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_flush].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs
[Fri Nov 02 16:27:18 2012] [I] [24974(140006182991616)] Dispatcher initialized (build 4.1.2)
```

### 为资源解析{#configure-sling-mapping-for-resource-resolution}配置Sling映射

使用Sling映射进行资源解析，以便基于域的URL解析为AEM发布实例上的内容。 资源映射将传入的URL从Dispatcher（最初来自客户端HTTP请求）转换为内容节点。

要了解Sling资源映射，请参阅Sling文档中的[资源解析的映射](https://sling.apache.org/site/mappings-for-resource-resolution.html)。

通常，需要为以下资源进行映射，但可能需要其他映射：

* 内容页面的根节点（`/content`下）
* 页面使用的设计节点（`/etc/designs`下）
* `/libs`文件夹

在为内容页面创建映射后，若要发现其他必需的映射，请使用Web浏览器在Web服务器上打开页面。 在发布实例的error.log文件中，找到有关未找到的资源的消息。 以下示例消息指示需要`/etc/clientlibs`的映射：

```shell
01.11.2012 15:59:24.601 *INFO* [10.36.34.243 [1351799964599] GET /etc/clientlibs/foundation/jquery.js HTTP/1.1] org.apache.sling.engine.impl.SlingRequestProcessorImpl service: Resource /content/sitea/etc/clientlibs/foundation/jquery.js not found
```

>[!NOTE]
>
>默认Apache Sling重写程序的linkchecker转换器会自动修改页面中的超链接，以防止链接断开。 但是，仅当链接目标为HTML或HTM文件时，才会执行链接重写。 要更新指向其他文件类型的链接，请创建变压器组件并将其添加到HTML重写器管道。

### 资源映射节点示例

下表列出了为branda.com域实施资源映射的节点。 为`brandb.com`域创建类似的节点，如`/etc/map/http/brandb.com`。 在所有情况下，当页面HTML中的引用无法在Sling上下文中正确解析时，都需要映射。

| 节点路径 | 类型 | 属性 |
|--- |--- |--- |
| `/etc/map/http/branda.com` | sling：映射 | 名称：sling:internalRedirect类型：字符串值：/content/sitea |
| `/etc/map/http/branda.com/libs` | sling：映射 | 名称：sling:internalRedirect <br/>类型：字符串<br/>值：/libs |
| `/etc/map/http/branda.com/etc` | sling：映射 |  |
| `/etc/map/http/branda.com/etc/designs` | sling：映射 | 名称：sling:internalRedirect <br/>VType:字符串<br/>值：/etc/designs |
| `/etc/map/http/branda.com/etc/clientlibs` | sling：映射 | 名称：sling:internalRedirect <br/>VType:字符串<br/>值：/etc/clientlibs |

## 配置调度程序刷新复制代理{#configuring-the-dispatcher-flush-replication-agent}

AEM发布实例上的调度程序刷新复制代理必须将失效请求发送到正确的调度程序场。 要定位场，请使用调度程序刷新复制代理的URI属性（在“传输”选项卡上）。 包括为使缓存失效而配置的Dispatcher场的`/virtualhost`属性值：

`https://*webserver_name*:*port*/*virtual_host*/dispatcher/invalidate.cache`

例如，要使用上一个示例的`farm_flush`场，URI为`https://localhost:80/invalidation_only/dispatcher/invalidate.cache`。

![](assets/chlimage_1-12.png)

## Web服务器重写传入的URL {#the-web-server-rewrites-incoming-urls}

使用Web服务器的内部URL重写功能，将基于域的URL转换为Dispatcher缓存中的文件路径。 例如，对`https://brandA.com/en.html`页面的客户端请求被转换为Web服务器文档根目录中的`content/sitea/en.html`文件。

![](assets/chlimage_1-13.png)

Dispatcher缓存镜像存储库节点结构。 因此，当页面激活时，生成的使缓存页面失效的请求不需要URL或路径转换。

![](assets/chlimage_1-14.png)

## 在Web服务器{#define-virtual-hosts-and-rewrite-rules-on-the-web-server}上定义虚拟主机和重写规则

在Web服务器上配置以下方面：

* 为每个Web域定义虚拟主机。
* 对于每个域，将文档根配置为与包含域Web内容的存储库中的文件夹一致。
* 对于每个虚拟域，创建一个URL重命名规则，以将传入的URL转换为缓存文件的路径。
* 每个虚拟域还必须包含与Dispatcher相关的配置，如[安装Dispatcher](dispatcher-install.md)页面中所述。
* 必须将调度程序模块配置为使用Web服务器已重写的URL。 （请参阅[安装Dispatcher](dispatcher-install.md)中的`DispatcherUseProcessedURL`属性。）

以下示例httpd.conf文件为Apache Web服务器配置两个虚拟主机：

* 服务器名称（与域名一致）为`brandA.com`（第16行）和`brandB.com`（第32行）。

* 每个虚拟域的文档根目录是Dispatcher缓存中包含站点页面的目录。 （第20和33行）
* 每个虚拟域的URL重写规则是一个正则表达式，用于为请求的页面的路径添加缓存中页面的路径的前缀。 （第19和35行）
* 将`DispatherUseProcessedURL`属性设置为`1`。 （行10）

例如，Web服务器在收到具有`https://brandA.com/en/products.html` URL的请求时，会执行以下操作：

* 将URL与虚拟主机关联，虚拟主机的`ServerName`为`brandA.com.`
* 将URL重写为`/content/sitea/en/products.html.`
* 将URL转发到Dispatcher。

### httpd.conf {#httpd-conf-1}

```xml
# load the Dispatcher module
LoadModule dispatcher_module modules/mod_dispatcher.so
# configure the Dispatcher module
<IfModule disp_apache2.c>
 DispatcherConfig conf/dispatcher.any
 DispatcherLog    logs/dispatcher.log  
 DispatcherLogLevel 3
 DispatcherNoServerHeader 0 
 DispatcherDeclineRoot 0
 DispatcherUseProcessedURL 1
 DispatcherPassError 0
</IfModule>

# Define virtual host for brandA.com
<VirtualHost *:80>
  ServerName branda.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
  RewriteEngine  on
  RewriteRule    ^/(.*)\.html$  /content/sitea/$1.html [PT]
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# define virtual host for brandB.com
<VirtualHost *:80>
  ServerName brandB.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
  RewriteEngine  on
  RewriteRule    ^/(.*)\.html$  /content/siteb/$1.html [PT]
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# document root for web server
DocumentRoot "/usr/lib/apache/httpd-2.4.3/htdocs"
```

### 配置调度程序场{#configure-a-dispatcher-farm}

当Web服务器重写URL时，Dispatcher需要根据[Configuring Dispatcher](dispatcher-configuration.md)定义的单个场。 支持Web服务器虚拟主机和URL重命名规则需要以下配置：

* `/virtualhosts`属性必须包含所有VirtualHost定义的ServerName值。
* `/statfileslevel`属性必须足够高，才能在包含每个域的内容文件的目录中创建.stat文件。

以下示例配置文件基于随Dispatcher一起安装的示例`dispatcher.any`文件。 需要进行以下更改才能支持以前`httpd.conf`文件的Web服务器配置：

* `/virtualhosts`属性使Dispatcher处理`brandA.com`和`brandB.com`域的请求。 （行12）
* 将`/statfileslevel`属性设置为2，以便在包含域Web内容的每个目录中创建stat文件（第41行）：`/statfileslevel "2"`

与往常一样，缓存的文档根与Web服务器的文档根相同（第40行）：`/usr/lib/apache/httpd-2.4.3/htdocs`

### `dispatcher.any` {#dispatcher-any}

```xml
/name "testDispatcher"
/farms
  {
  /dispfarm0
    {  
    /clientheaders
      {
      "*"
      }      
    /virtualhosts
      {
      "brandA.com" "brandB.com"
      }
    /renders
      {
      /rend01    {  /hostname "127.0.0.1"   /port "4503"  }
      }
    /filter
      {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/content*" }  # disable this rule to allow mapped content only
      /0041 { /type "allow" /glob "* *.css *"   }  # enable css
      /0042 { /type "allow" /glob "* *.gif *"   }  # enable gifs
      /0043 { /type "allow" /glob "* *.ico *"   }  # enable icos
      /0044 { /type "allow" /glob "* *.js *"    }  # enable javascript
      /0045 { /type "allow" /glob "* *.png *"   }  # enable png
      /0046 { /type "allow" /glob "* *.swf *"   }  # enable flash
      /0061 { /type "allow" /glob "POST /content/[.]*.form.html" }  # allow POSTs to form selectors under content
      /0062 { /type "allow" /glob "* /libs/cq/personalization/*"  }  # enable personalization
      /0081 { /type "deny"  /glob "GET *.infinity.json*" }
      /0082 { /type "deny"  /glob "GET *.tidy.json*"     }
      /0083 { /type "deny"  /glob "GET *.sysview.xml*"   }
      /0084 { /type "deny"  /glob "GET *.docview.json*"  }
      /0085 { /type "deny"  /glob "GET *.docview.xml*"  }      
      /0086 { /type "deny"  /glob "GET *.*[0-9].json*" }
      /0090 { /type "deny"  /glob "* *.query.json*" }
      }
    /cache
      {
      /docroot "/usr/lib/apache/httpd-2.4.3/htdocs"
      /statfileslevel "2"
      /allowAuthorized "0"
      /rules
        {
        /0000  { /glob "*"     /type "allow"  }
        }
      /invalidate
        {
        /0000  {   /glob "*" /type "deny"  }
        /0001 {  /glob "*.html" /type "allow"  }
        }
      /allowedClients
        {
        }     
      }
    /statistics
      {
      /categories
        {
        /html  { /glob "*.html" }
        /others  {  /glob "*"  }
        }
      }
    }
  }
```

>[!NOTE]
>
>由于定义了单个Dispatcher场，因此AEM发布实例上的Dispatcher刷新复制代理不需要特殊配置。

## 重写指向非HTML文件的链接{#rewriting-links-to-non-html-files}

要重写对扩展名不为.html或.htm的文件的引用，请创建Sling重写器变压器组件，并将其添加到默认重写器管道。

当资源路径在Web服务器上下文中无法正确解析时，将重写引用。 例如，当图像生成组件创建链接(如/content/sitea/en/products.navimage.png)时，需要使用变压器。 [如何创建功能齐全的Internet网站](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/the-basics.html)的顶部导航组件会创建此类链接。

[Sling rewriter](https://sling.apache.org/documentation/bundles/output-rewriting-pipelines-org-apache-sling-rewriter.html)是后处理Sling输出的模块。 重写器的SAX管线实现由发生器、一个或多个变压器和序列化器组成：

* **生成器：** 解析Sling输出流（HTML文档），并在遇到特定元素类型时生成SAX事件。
* **变压器：** 侦听SAX事件，并随后修改事件目标（HTML元素）。重写器管道包含零个或多个变压器。 按顺序执行变压器，将SAX事件传递给序列中的下一个变压器。
* **序列化器：** 序列化输出，包括来自每个变压器的修改。

![](assets/chlimage_1-15.png)

### AEM默认重写器管道{#the-aem-default-rewriter-pipeline}

AEM使用默认的管道重写器处理文本/html类型的文档：

* 生成器会解析HTML文档，并在遇到a、img、区域、表单、基、链接、脚本和正文元素时生成SAX事件。 生成器别名为`htmlparser`。
* 该管道包括以下变压器：`linkchecker`、`mobile`、`mobiledebug`、`contentsync`。 `linkchecker`转换器将引用的HTML或HTM文件的路径外部化，以防止链接断开。
* 序列化器写入HTML输出。 序列化程序别名为htmlwriter。

`/libs/cq/config/rewriter/default`节点定义管道。

### 创建变压器{#creating-a-transformer}

执行以下任务以创建变压器组件并将其用在管道中：

1. 实施`org.apache.sling.rewriter.TransformerFactory`接口。 此类创建变压器类的实例。 指定`transformer.type`属性的值（转换器别名），并将类配置为OSGi服务组件。
1. 实施`org.apache.sling.rewriter.Transformer`接口。 要最大限度地减少工作量，可以扩展`org.apache.cocoon.xml.sax.AbstractSAXPipe`类。 覆盖startElement方法以自定义重写行为。 对传递到变压器的每个SAX事件都调用此方法。
1. 捆绑和部署类。
1. 向AEM应用程序添加配置节点，以将转换器添加到管线。

>[!TIP]
>您可以改为配置TransformerFactory ，以便将变压器插入每个已定义的重写器中。 因此，您无需配置管道：
>
>* 将`pipeline.mode`属性设置为`global`。
>* 将`service.ranking`属性设置为正整数。
>* 请勿包含`pipeline.type`属性。


>[!NOTE]
>
>使用内容包Maven插件的[multimodule](https://helpx.adobe.com/cn/experience-manager/aem-previous-versions.html)原型创建Maven项目。 POM会自动创建和安装内容包。

以下示例实施了一个转换器，该转换器会重写对图像文件的引用。

* MyRewriterTransformerFactory类可实例化MyRewriterTransformer对象。 pipeline.type属性将变压器别名设置为mytransformer。 为了将别名包括在管道中，管道配置节点将此别名包括在变压器列表中。
* MyRewriterTransformer类覆盖AbstractSAXTransfer类的startElement方法。 startElement方法会重写img元素的src属性值。

这些示例不强大，不应在生产环境中使用。

### TransformerFactory实施示例{#example-transformerfactory-implementation}

```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.rewriter.Transformer;
import org.apache.sling.rewriter.TransformerFactory;

@Component
@Service
public class MyRewriterTransformerFactory implements TransformerFactory {
    /* Define the alias */
    @Property(value="mytransformer")
    static final String PIPELINE_TYPE ="pipeline.type";
 
    public Transformer createTransformer() {
        
        return new MyRewriterTransformer ();
    }
}
```

### 变压器实现示例{#example-transformer-implementation}

```java
package com.adobe.example;

import java.io.IOException;

import org.apache.cocoon.xml.sax.AbstractSAXPipe;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.rewriter.ProcessingComponentConfiguration;
import org.apache.sling.rewriter.ProcessingContext;
import org.apache.sling.rewriter.Transformer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.AttributesImpl;

import javax.servlet.http.HttpServletRequest;

public class MyRewriterTransformer extends AbstractSAXPipe implements Transformer {

 private static final Logger log = LoggerFactory.getLogger(MyRewriterTransformer.class);
 private SlingHttpServletRequest httpRequest; 
 /* The element and attribute to act on  */
 private static final String ATT_NAME = new String("src");
 private static final String EL_NAME = new String("img");

 public MyRewriterTransformer () {
 }
 public void dispose() {
 }
 public void init(ProcessingContext context, ProcessingComponentConfiguration config) throws IOException {
  this.httpRequest = context.getRequest();
  log.debug("Transforming request {}.", httpRequest.getRequestURI());
 }
 @Override
 public void startElement (String nsUri, String localname, String qname, Attributes atts) throws SAXException {
  /* copy the element attributes */
  AttributesImpl linkAtts = new AttributesImpl(atts); 
  /* Only interested in EL_NAME elements */
  if(EL_NAME.equalsIgnoreCase(localname)){

   /* iterate through the attributes of the element and act only on ATT_NAME attributes */
   for (int i=0; i < linkAtts.getLength(); i++) {
    if (ATT_NAME.equalsIgnoreCase(linkAtts.getLocalName(i))) {
     String path_in_link = linkAtts.getValue(i);

     /* use the resource resolver of the http request to reverse-resolve the path  */
     String mappedPath = httpRequest.getResourceResolver().map(httpRequest, path_in_link);

     log.info("Tranformed {} to {}.", path_in_link,mappedPath);

     /* update the attribute value */
     linkAtts.setValue(i,mappedPath);
    }
   }

  }
        /* return updated attributes to super and continue with the transformer chain */
 super.startElement(nsUri, localname, qname, linkAtts);
 }
}
```

### 将变压器添加到重写器管线{#adding-the-transformer-to-a-rewriter-pipeline}

创建一个JCR节点，该节点定义使用变压器的管线。 以下节点定义会创建处理文本/html文件的管道。 将使用HTML的默认AEM生成器和解析器。

>[!NOTE]
>
>如果将Transformer属性`pipeline.mode`设置为`global`，则无需配置管线。 `global`模式将变压器插入所有管道。

### 重写器配置节点 — XML表示{#rewriter-configuration-node-xml-representation}

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="https://www.jcp.org/jcr/1.0" xmlns:nt="https://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="nt:unstructured"
    contentTypes="[text/html]"
    enabled="{Boolean}true"
    generatorType="htmlparser"
    order="5"
    serializerType="htmlwriter"
    transformerTypes="[mytransformer]">
</jcr:root>
```

下图显示了节点的CRXDE Lite表示形式：

![](assets/chlimage_1-16.png)
