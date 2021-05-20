---
title: 配置 Dispatcher
description: 了解如何配置Dispatcher。
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '8513'
ht-degree: 2%

---

# 配置 Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>各个 Dispatcher 版本与 AEM 相互独立。如果单击以前版本 AEM 文档中嵌入的 Dispatcher 文档链接，可能会重定向到此页面。

以下各节介绍如何配置Dispatcher的各个方面。

## 支持IPv4和IPv6 {#support-for-ipv-and-ipv}

AEM和Dispatcher的所有元素都可以安装在IPv4和IPv6网络中。 请参阅[IPV4和IPV6](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/introduction/technical-requirements.html?lang=en#ipv-and-ipv)。

## 调度程序配置文件{#dispatcher-configuration-files}

默认情况下，Dispatcher配置存储在`dispatcher.any`文本文件中，不过您可以在安装过程中更改此文件的名称和位置。

配置文件包含一系列可控制Dispatcher行为的单值或多值属性：

* 属性名称前缀为正斜杠`/`。
* 多值属性使用大括号`{ }`将子项括起来。

示例配置的结构如下所示：

```xml
# name of the dispatcher
/name "internet-server"

# each farm configures a set off (loadbalanced) renders
/farms
 {
  # first farm entry (label is not important, just for your convenience)
   /website
     {  
     /clientheaders
       {
       # List of headers that are passed on
       }
     /virtualhosts
       {
       # List of URLs for this Web site
       }
     /sessionmanagement
       {
       # settings for user authentification
       }
     /renders
       {
       # List of AEM instances that render the documents
       }
     /filter
       {
       # List of filters
       }
     /vanity_urls
       {
       # List of vanity URLs
       }
     /cache
       {
       # Cache configuration
       /rules
         {
         # List of cachable documents
         }
       /invalidate
         {
         # List of auto-invalidated documents
         }
       }
     /statistics
       {
       /categories
         {
         # The document categories that are used for load balancing estimates
         }
       }
     /stickyConnectionsFor "/myFolder"
     /health_check
       {
       # Page gets contacted when an instance returns a 500
       }
     /retryDelay "1"
     /numberOfRetries "5"
     /unavailablePenalty "1"
     /failover "1"
     }
 }
```

您可以包含对配置有贡献的其他文件：

* 如果您的配置文件较大，则可以将其拆分为多个较小的文件（更易于管理），然后包含这些文件。
* 包含自动生成的文件。

例如，要在/farms配置中包含文件myFarm.any，请使用以下代码：

```xml
/farms
  {
  $include "myFarm.any"
  }
```

使用星号(`*`)作为通配符指定要包含的文件范围。

例如，如果文件`farm_1.any`到`farm_5.any`包含1到5个场的配置，则可以按如下方式包含它们：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用环境变量{#using-environment-variables}

您可以在dispatcher.any文件中的字符串值属性中使用环境变量，而不是硬编码值。 要包含环境变量的值，请使用格式`${variable_name}`。

例如，如果dispatcher.any文件与缓存目录位于同一目录中，则可以使用[docroot](#specifying-the-cache-directory)属性的以下值：

```xml
/docroot "${PWD}/cache"
```

另例如，如果您创建一个名为`PUBLISH_IP`的环境变量来存储AEM发布实例的主机名，则可以使用[/renders](#defining-page-renderers-renders)属性的以下配置：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 命名调度程序实例{#naming-the-dispatcher-instance-name}

使用`/name`属性指定唯一名称以标识Dispatcher实例。 `/name`属性是配置结构中的顶级属性。

## 定义场{#defining-farms-farms}

`/farms`属性定义一组或多组Dispatcher行为，其中每组都与不同的网站或URL关联。 `/farms`属性可以包含单个场或多个场：

* 当您希望Dispatcher以相同方式处理所有网页或网站时，请使用单个场。
* 当网站的不同区域或不同网站需要不同的调度程序行为时，创建多个场。

`/farms`属性是配置结构中的顶级属性。 要定义场，请将子属性添加到`/farms`属性中。 使用唯一标识Dispatcher实例中场的属性名称。

`/farmname`属性具有多值，并且包含定义Dispatcher行为的其他属性：

* 场应用到的页面的URL。
* 用于渲染文档的一个或多个服务URL(通常为AEM发布实例)。
* 用于负载平衡多个文档渲染器的统计信息。
* 其他一些行为，例如要缓存哪些文件和位置。

值可以包含任何字母数字(a-z、0-9)字符。 以下示例显示了名为`/daycom`和`/docsdaycom`的两个场的骨架定义：

```xml
#name of dispatcher
/name "day sites"

#farms section defines a list of farms or sites
/farms
{
   /daycom
   {
       ...
   }
   /docdaycom
   {
      ...
   }
}
```

>[!NOTE]
>
>如果使用多个渲染场，则将从下而上评估列表。 在为您的网站定义[虚拟主机](#identifying-virtual-hosts-virtualhosts)时，这特别相关。

每个场属性都可以包含以下子属性：

| 属性名称 | 描述 |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | 默认主页（可选）（仅IIS） |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | 要传递的客户端HTTP请求中的标头。 |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | 此场的虚拟主机。 |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | 支持会话管理和身份验证。 |
| [/renders](#defining-page-renderers-renders) | 提供已渲染页面的服务器(通常为AEM发布实例)。 |
| [/filter](#configuring-access-to-content-filter) | 定义Dispatcher允许访问的URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 配置对虚URL的访问权限。 |
| [/propagateSyncPost](#forwarding-syndication-requests-propagatesyndpost) | 支持转发联合请求。 |
| [/cache](#configuring-the-dispatcher-cache-cache) | 配置缓存行为。 |
| [/statistics](#configuring-load-balancing-statistics) | 为负载平衡计算定义统计类别。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | 包含置顶文档的文件夹。 |
| [/health_check](#specifying-a-health-check-page) | 用于确定服务器可用性的URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重试失败连接之前的延迟。 |
| [/unavailableDecamty](#reflecting-server-unavailability-in-dispatcher-statistics) | 影响负载平衡计算统计的处罚。 |
| [/failover](#using-the-failover-mechanism) | 当原始请求失败时，向不同的呈现重新发送请求。 |
| [/auth_checker](permissions-cache.md) | 有关权限敏感型缓存，请参阅[缓存安全内容](permissions-cache.md)。 |

## 指定默认页面（仅限IIS） — /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>`/homepage`参数（仅限IIS）不再有效。 您而是应使用[IIS URL重写模块](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)。
>
>如果您使用的是Apache，则应使用`mod_rewrite`模块。 有关`mod_rewrite`的信息，请参阅Apache网站文档（例如，[Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)）。 使用`mod_rewrite`时，建议使用标志&#x200B;**[&#39;passthrough|PT&#39;（传递到下一个处理程序）](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)**&#x200B;强制重写引擎将内部`request_rec`结构的`uri`字段设置为`filename`字段的值。

<!-- 

Comment Type: draft

<p>The optional /homepage parameter specifies the page that Dispatcher returns when a client requests an undeterminable page or file.</p> 
<p>Typically this situation occurs when a user specifies an URL for which neither IIS or AEM provides an automatic redirection target. For example, if the AEM render instance is shut down after the content is cached, the content redirect URL is unavailable.</p> 
<p>The following example configuration displays the <span class="code">index.html</span> page in such circumstances:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /homepage&nbsp;"/index.html" 
</codeblock>

 -->

<!-- 

Comment Type: draft

<p>The <span class="code">/homepage</span> section is located inside the <span class="code">/farms</span> section, for example:<br /> </p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  #name&nbsp;of&nbsp;dispatcher!!discoiqbr!!/name&nbsp;"day&nbsp;sites"!!discoiqbr!!!!discoiqbr!!#farms&nbsp;section&nbsp;defines&nbsp;a&nbsp;list&nbsp;of&nbsp;farms&nbsp;or&nbsp;sites!!discoiqbr!!/farms!!discoiqbr!!{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/daycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/homepage&nbsp;"/index.html"!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!&nbsp;&nbsp;&nbsp;/docdaycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!} 
</codeblock>

 -->

## 指定要传递{#specifying-the-http-headers-to-pass-through-clientheaders}的HTTP标头

`/clientheaders`属性定义Dispatcher从客户端HTTP请求传递到呈现器(AEM实例)的HTTP标头列表。

默认情况下，Dispatcher会将标准HTTP标头转发到AEM实例。 在某些情况下，您可能希望转发其他标头，或删除特定的标头：

* 添加AEM实例在HTTP请求中所需的标头，如自定义标头。
* 删除仅与Web服务器相关的标头，如身份验证标头。

如果您自定义要传递的标头集，则必须指定详尽的标头列表，包括通常默认包含的标头。

例如，处理发布实例的页面激活请求的Dispatcher实例需要`/clientheaders`部分中的`PATH`标头。 `PATH`标头支持复制代理与调度程序之间的通信。

以下代码是`/clientheaders`的示例配置：

```shell
/clientheaders
  {
  "CSRF-Token"
  "X-Forwarded-Proto"
  "referer"
  "user-agent"
  "authorization"
  "from"
  "content-type"
  "content-length"
  "accept-charset"
  "accept-encoding"
  "accept-language"
  "accept"
  "host"
  "if-match"
  "if-none-match"
  "if-range"
  "if-unmodified-since"
  "max-forwards"
  "proxy-authorization"
  "proxy-connection"
  "range"
  "cookie"
  "cq-action"
  "cq-handle"
  "handle"
  "action"
  "cqstats"
  "depth"
  "translate"
  "expires"
  "date"
  "dav"
  "ms-author-via"
  "if"
  "lock-token"
  "x-expected-entity-length"
  "destination"
  "PATH"
  }
```

## 识别虚拟主机{#identifying-virtual-hosts-virtualhosts}

`/virtualhosts`属性定义Dispatcher接受此场的所有主机名/URI组合的列表。 可以使用星号(`*`)字符作为通配符。 / `virtualhosts`属性的值使用以下格式：

```xml
[scheme]host[uri][*]
```

* `scheme`:（可选） `https://` 或  `https://.`
* `host`:主机的名称或IP地址以及端口号（如果需要）。(请参阅[https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`:（可选）资源的路径。

以下示例配置处理myCompany的.com和.ch域以及mySubDivision的所有域的请求：

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

以下配置处理&#x200B;*所有*&#x200B;请求：

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解析虚拟主机{#resolving-the-virtual-host}

当Dispatcher收到HTTP或HTTPS请求时，它会找到与请求的`host,` `uri`和`scheme`标头最匹配的虚拟主机值。 Dispatcher按以下顺序计算`virtualhosts`属性中的值：

* Dispatcher从最低的场开始，并在dispatcher.any文件中向上运行。
* 对于每个场，Dispatcher以`virtualhosts`属性中最顶部的值开头，并向下显示值列表。

Dispatcher通过以下方式查找最匹配的虚拟主机值：

* 将使用与请求的`host`、`scheme`和`uri`中所有三个匹配的第一个虚拟主机。
* 如果没有`virtualhosts`值的`scheme`和`uri`部分与请求的`scheme`和`uri`部分均匹配，则使用与请求的`host`匹配的首次遇到的虚拟主机。
* 如果没有`virtualhosts`值的主机部分与请求的主机相匹配，则使用最顶端场的最顶端虚拟主机。

因此，您应将默认虚拟主机放在`virtualhosts`属性的顶部，位于`dispatcher.any`文件最上方的场中。

### 虚拟主机分辨率{#example-virtual-host-resolution}示例

以下示例代表`dispatcher.any`文件中定义两个Dispatcher场的代码片段，每个场定义一个`virtualhosts`属性。

```xml
/farms
  {
  /myProducts
    {
    /virtualhosts
      {
      "www.mycompany.com"
      }
    /renders
      {
      /hostname "server1.myCompany.com"
      /port "80"
      }
    }
  /myCompany
    {
    /virtualhosts
      {
      "www.mycompany.com/products/*"
      }
    /renders
      {
      /hostname "server2.myCompany.com"
      /port "80"
      }
    }
  }
```

使用此示例，下表显示了为给定HTTP请求解析的虚拟主机：

| 请求URL | 已解析的虚拟主机 |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 启用安全会话 — /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` **** 要启用此功 `"0"` 能， `/cache` 必须在部分中将设置为。

创建安全会话以访问呈现场，以便用户需要登录才能访问场中的任何页面。 登录后，用户可以访问场中的页面。 有关将此功能与CUG结合使用的信息，请参阅[创建封闭用户组](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/cug.html?lang=en#creating-the-user-group-to-be-used)。 此外，在上线之前，请参阅Dispatcher [安全检查列表](/help/using/security-checklist.md)。

`/sessionmanagement`属性是`/farms`的子属性。

>[!CAUTION]
>
>如果您网站的某些部分使用不同的访问要求，则需要定义多个场。

**/** sessionmanagement作为多个子参数：

**/directory** （必填）

存储会话信息的目录。 如果目录不存在，则创建该目录。

>[!CAUTION]
>
> 配置目录子参数&#x200B;**时，请勿**&#x200B;指向根文件夹(`/directory "/"`)，因为它可能会导致严重问题。 您应始终指定用于存储会话信息的文件夹的路径。 例如：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** （可选）

会话信息的编码方式。 使用`md5`进行使用md5算法加密，或使用`hex`进行十六进制编码。 如果加密会话数据，则具有文件系统访问权限的用户将无法读取会话内容。 默认为 `md5`.

**/header** （可选）

存储授权信息的HTTP标头或Cookie的名称。 如果将信息存储在http标头中，请使用`HTTP:<header-name>`。 要在Cookie中存储信息，请使用`Cookie:<header-name>`。 如果未指定值`HTTP:authorization` ，则使用。

**/timeout** （可选）

会话在上次使用后超时的秒数。 如果未指定`"800"`，则会话会在用户最后一次请求后13分钟多一点超时。

配置示例如下所示：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  /encode "md5"
  /header "HTTP:authorization"
  /timeout "800"
  }
```

## 定义页面渲染器{#defining-page-renderers-renders}

/renders属性定义Dispatcher将请求发送到的URL以呈现文档。 以下示例`/renders`部分标识要渲染的单个AEM实例：

```xml
/renders
  {
    /myRenderer
      {
      # hostname or IP of the renderer
      /hostname "aem.myCompany.com"
      # port of the renderer
      /port "4503"
      # connection timeout in milliseconds, "0" (default) waits indefinitely
      /timeout "0"
      }
  }
```

以下示例/renders部分标识与调度程序在同一台计算机上运行的AEM实例：

```xml
/renders
  {
    /myRenderer
     {
     /hostname "127.0.0.1"
     /port "4503"
     }
  }
```

以下示例/renders部分在两个AEM实例之间平均分发渲染请求：

```xml
/renders
  {
    /myFirstRenderer
      {
      /hostname "aem.myCompany.com"
      /port "4503"
      }
    /mySecondRenderer
      {
      /hostname "127.0.0.1"
      /port "4503"
      }
  }
```

### 呈现选项{#renders-options}

**/timeout**

指定访问AEM实例的连接超时（以毫秒为单位）。 默认值为`"0"`，导致Dispatcher无限期等待。

**/receiveTimeout**

指定允许响应花费的时间（以毫秒为单位）。 默认值为`"600000"`，导致Dispatcher等待10分钟。 设置`"0"`可完全消除超时。

如果在解析响应标头时达到超时，则会返回HTTP状态504（网关错误）。 如果在读取响应正文时达到超时，则Dispatcher将向客户端返回不完整的响应，但删除可能已写入的任何缓存文件。

**/ipv4**

指定Dispatcher是使用`getaddrinfo`函数（对于IPv6）还是`gethostbyname`函数（对于IPv4）来获取呈现器的IP地址。 值为0时，会使用`getaddrinfo`。 值`1`会导致使用`gethostbyname`。 默认值为 `0`.

`getaddrinfo`函数返回IP地址列表。 Dispatcher迭代地址列表，直到它建立TCP/IP连接。 因此，当呈现主机名与多个IP地址关联时， `ipv4`属性很重要，并且主机响应`getaddrinfo`函数返回始终按相同顺序排列的IP地址列表。 在这种情况下，您应使用`gethostbyname`函数，以便Dispatcher与之连接的IP地址是随机的。

Amazon Elastic Load Balancing(ELB)是一项服务，它以可能相同的IP地址列表来响应getaddrinfo。

**/secure**

如果`/secure`属性的值为`"1"`，则Dispatcher使用HTTPS与AEM实例通信。 有关更多详细信息，另请参阅[将Dispatcher配置为使用SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl)。

**/always-resolve**

使用Dispatcher版本&#x200B;**4.1.6**，您可以按如下方式配置`/always-resolve`属性：

* 当设置为`"1"`时，它将在每个请求中解析主机名（Dispatcher将永远不会缓存任何IP地址）。 由于需要额外调用来获取每个请求的主机信息，因此可能会对性能产生轻微影响。
* 如果未设置属性，则默认情况下会缓存IP地址。

此外，当您遇到动态IP解析问题时，还可以使用此属性，如以下示例所示：

```xml
/renders {
  /0001 {
     /hostname "host-name-here"
     /port "4502"
     /ipv4 "1"
     /always-resolve "1"
     }
  }
```

## 配置对内容的访问{#configuring-access-to-content-filter}

使用`/filter`部分指定Dispatcher接受的HTTP请求。 所有其他请求都将发送回带有404错误代码（页面未找到）的Web服务器。 如果不存在`/filter`部分，则接受所有请求。

**注意：** statfile的请 [](#naming-the-statfile) 求始终被拒绝。

>[!CAUTION]
>
>请参阅[Dispatcher安全检查列表](security-checklist.md)，以了解在使用Dispatcher限制访问时的进一步注意事项。 另外，请阅读[AEM安全检查列表](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/security-checklist.html?lang=en#security)，了解有关AEM安装的其他安全详细信息。

`/filter`部分由一系列规则组成，这些规则根据HTTP请求的请求行部分中的模式拒绝或允许访问内容。 您应该为`/filter`部分使用允许列表策略：

* 首先，拒绝访问所有内容。
* 允许根据需要访问内容。

### 定义过滤器{#defining-a-filter}

`/filter`部分中的每个项目都包括与请求行或整个请求行的特定元素匹配的类型和模式。 每个过滤器可以包含以下项目：

* **类型**:指 `/type` 示是允许还是拒绝对与模式匹配的请求的访问。值可以是`allow`或`deny`。

* **请求行的元素：** 包含 `/method`、 `/url`、 `/query`或 `/protocol` ，以及用于根据HTTP请求的请求行部分的这些特定部分过滤请求的模式。首选的筛选方法是对请求行的元素（而不是整个请求行）进行筛选。

* **请求行的高级元素：** 从Dispatcher 4.2.0开始，提供了四个新的过滤器元素供使用。这些新元素分别为`/path`、`/selectors`、`/extension`和`/suffix`。 包括一个或多个这些项目以进一步控制URL模式。

>[!NOTE]
>
>有关每个元素引用的请求行中哪一部分的更多信息，请参阅[Sling URL分解](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html)wiki页面。

* **全局属性**:属 `/glob` 性用于匹配HTTP请求的整个请求行。

>[!CAUTION]
>
>Dispatcher中已弃用使用glob进行筛选。 因此，您应避免在`/filter`部分中使用全局，因为它可能会导致安全问题。 因此，不是：
>
>`/glob "* *.css *"`
>
>您应使用
>
>`/url "*.css"`

#### HTTP请求的请求行部分{#the-request-line-part-of-http-requests}

HTTP/1.1定义[request-line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html)，如下所示：

`Method Request-URI HTTP-Version<CRLF>`

`<CRLF>`字符表示回车符后跟换行符。 以下示例是客户请求WKND网站的美英页面时收到的请求行：

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

您的模式必须考虑请求行中的空格字符和`<CRLF>`字符。

#### 双引号与单引号{#double-quotes-vs-single-quotes}

创建过滤器规则时，对于简单模式，请使用双引号`"pattern"`。 如果您使用的是Dispatcher 4.2.0或更高版本并且您的模式包含正则表达式，则必须在单引号内括住正则表达式模式`'(pattern1|pattern2)'`。

#### 正则表达式{#regular-expressions}

在高于4.2.0的Dispatcher版本中，您可以在过滤器模式中包含POSIX扩展正则表达式。

#### 过滤器{#troubleshooting-filters}故障诊断

如果您的过滤器未按预期方式触发，请在调度程序上启用[跟踪日志记录](#trace-logging)，以便您能够看到哪个过滤器正在拦截请求。

#### 示例过滤器：全部拒绝{#example-filter-deny-all}

以下示例过滤器部分使Dispatcher拒绝所有文件的请求。 您应拒绝访问所有文件，然后允许访问特定区域。

```xml
  /0001  { /glob "*" /type "deny" }
```

对明确拒绝区域的请求会导致返回404错误代码（页面未找到）。

#### 示例过滤器：拒绝访问特定区域{#example-filter-deny-access-to-specific-areas}

过滤器还允许您拒绝访问各种元素，例如ASP页面和发布实例中的敏感区域。 以下过滤器拒绝访问ASP页面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 示例过滤器：启用POST请求{#example-filter-enable-post-requests}

以下示例过滤器允许使用POST方法提交表单数据：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 示例过滤器：允许访问工作流控制台{#example-filter-allow-access-to-the-workflow-console}

以下示例显示了用于拒绝外部访问工作流控制台的过滤器：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

如果您的发布实例使用Web应用程序上下文（例如发布），则也可以将此上下文添加到过滤器定义中。

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

如果您仍需要访问限制区域内的单个页面，则可以允许访问这些页面。 例如，要允许访问工作流控制台中的存档选项卡，请添加以下部分：

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>当多个过滤器模式应用于请求时，应用的最后一个过滤器模式将有效。

#### 示例过滤器：使用正则表达式{#example-filter-using-regular-expressions}

此过滤器使用正则表达式在非公共内容目录中启用扩展，此处在单引号之间定义此表达式：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 示例过滤器：筛选请求URL的其他元素{#example-filter-filter-additional-elements-of-a-request-url}

下面是一个规则示例，它使用路径、选择器和扩展的过滤器阻止从`/content`路径及其子树中捕获内容：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### 示例/filter部分{#example-filter-section}

配置Dispatcher时，应尽可能限制外部访问。 以下示例为外部访客提供了最少的访问权限：

* `/content`
* 其他内容，如设计和客户端库；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

创建过滤器后，请[测试页面访问](#testing-dispatcher-security)以确保AEM实例的安全。

`dispatcher.any`文件的以下`/filter`部分可用作[Dispatcher配置文件的基础。](#dispatcher-configuration-files)

此示例基于随Dispatcher提供的默认配置文件，该文件旨在作为生产环境中使用的示例。 前缀为`#`的项目被停用（注释掉），如果您决定激活其中的任意项目（通过删除该行上的`#`），则应当小心，因为这可能会对安全产生影响。

您应拒绝访问所有内容，然后允许访问特定（有限）元素：

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:32:37.986-0400

<p>We should mention the config files that are shipped with the dispatcher distribution and only give a few examples here. This aims to avoid confusion and reduce content maintenance.<br /> </p>

 -->

```xml
  /filter
      {
      # Deny everything first and then allow specific entries
      /0001 { /type "deny" /glob "*" }

      # Open consoles
#     /0011 { /type "allow" /url "/admin/*"  }  # allow servlet engine admin
#     /0012 { /type "allow" /url "/crx/*"    }  # allow content repository
#     /0013 { /type "allow" /url "/system/*" }  # allow OSGi console

      # Allow non-public content directories
#     /0021 { /type "allow" /url "/apps/*"   }  # allow apps access
#     /0022 { /type "allow" /url "/bin/*"    }
      /0023 { /type "allow" /url "/content*" }  # disable this rule to allow mapped content only

#     /0024 { /type "allow" /url "/libs/*"   }
#     /0025 { /type "deny"  /url "/libs/shindig/proxy*" } # if you enable /libs close access to proxy

#     /0026 { /type "allow" /url "/home/*"   }
#     /0027 { /type "allow" /url "/tmp/*"    }
#     /0028 { /type "allow" /url "/var/*"    }

      # Enable extensions in non-public content directories, using a regular expression
      /0041
        {
        /type "allow"
        /extension '(css|gif|ico|js|png|swf|jpe?g)'
        }

      # Enable features
      /0062 { /type "allow" /url "/libs/cq/personalization/*"  }  # enable personalization

      # Deny content grabbing, on all accessible pages, using regular expressions
      /0081
        {
        /type "deny"
        /selectors '((sys|doc)view|query|[0-9-]+)'
        /extension '(json|xml)'
        }
      # Deny content grabbing for /content and its subtree
      /0082
        {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }

#     /0087 { /type "allow" /method "GET" /extension 'json' "*.1.json" }  # allow one-level json requests
}
```

>[!NOTE]
>
>与Apache一起使用时，请根据Dispatcher模块的DispatcherUseProcessedURL属性设计过滤器URL模式。 （请参阅[Apache Web Server — 为Dispatcher配置Apache Web Server](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher)。）

>[!NOTE]
>
>与Dynamic Media相关的过滤器`0030`和`0031`适用于AEM 6.0及更高版本。

如果您选择扩展访问权限，请考虑以下建议：

* 如果您使用的是CQ版本5.4或更早版本，则对`/admin`的外部访问应始终完全禁用&#x200B;**。

* 允许访问`/libs`中的文件时，必须小心。 应允许单独访问。
* 拒绝对复制配置的访问，以便无法看到该配置：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒绝访问Google Gadgets反向代理：

   * `/libs/opensocial/proxy*`

根据您的安装，`/libs`、`/apps`或其他位置下可能存在其他资源，这些资源必须可用。 您可以将`access.log`文件用作确定外部正在访问的资源的一种方法。

>[!CAUTION]
>
>访问控制台和目录可能会对生产环境带来安全风险。 除非您有明确的理由，否则应保持停用状态（注释掉）。

>[!CAUTION]
>
>如果您[在发布环境中使用报表](https://experienceleague.adobe.com/docs/experience-manager-65/administering/operations/reporting.html?lang=en#using-reports-in-a-publish-environment)，则应将Dispatcher配置为拒绝外部访客访问`/etc/reports`。

### 限制查询字符串{#restricting-query-strings}

自Dispatcher版本4.1.5起，使用`/filter`部分限制查询字符串。 强烈建议通过`allow`筛选器元素明确允许查询字符串并排除一般允许。

单个条目可以具有`glob`或`method`、`url`、`query`和`version`的某种组合，但不能同时具有这两种组合。 以下示例允许`a=*`查询字符串并拒绝解析到`/etc`节点的URL的所有其他查询字符串：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果规则包含`/query`，则它将仅匹配包含查询字符串的请求并匹配提供的查询模式。
>
>在上例中，如果对`/etc`的请求没有查询字符串，则还应当允许使用以下规则：


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 测试Dispatcher安全性{#testing-dispatcher-security}

Dispatcher过滤器应阻止访问AEM发布实例上的以下页面和脚本。 使用Web浏览器尝试像站点访客那样打开以下页面，并验证是否返回了代码404。 如果获得任何其他结果，请调整过滤器。

请注意，您应会看到`/content/add_valid_page.html?debug=layout`的正常页面呈现。

* `/admin`
* `/system/console`
* `/dav/crx.default`
* `/crx`
* `/bin/crxde/logs`
* `/jcr:system/jcr:versionStorage.json`
* `/_jcr_system/_jcr_versionStorage.json`
* `/libs/wcm/core/content/siteadmin.html`
* `/libs/collab/core/content/admin.html`
* `/libs/cq/ui/content/dumplibs.html`
* `/var/linkchecker.html`
* `/etc/linkchecker.html`
* `/home/users/a/admin/profile.json`
* `/home/users/a/admin/profile.xml`
* `/libs/cq/core/content/login.json`
* `/content/../libs/foundation/components/text/text.jsp`
* `/content/.{.}/libs/foundation/components/text/text.jsp`
* `/apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata`
* `/libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet`
* `/content.pages.json`
* `/content.languages.json`
* `/content.blueprint.json`
* `/content.-1.json`
* `/content.10.json`
* `/content.infinity.json`
* `/content.tidy.json`
* `/content.tidy.-1.blubber.json`
* `/content/dam.tidy.-100.json`
* `/content/content/geometrixx.sitemap.txt `
* `/content/add_valid_page.query.json?statement=//*`
* `/content/add_valid_page.qu%65ry.js%6Fn?statement=//*`
* `/content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)`
* `/content/add_valid_path_to_a_page/_jcr_content.json`
* `/content/add_valid_path_to_a_page/jcr:content.json`
* `/content/add_valid_path_to_a_page/_jcr_content.feed`
* `/content/add_valid_path_to_a_page/jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename._jcr_content.feed`
* `/content/add_valid_path_to_a_page/pagename.jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename.docview.xml`
* `/content/add_valid_path_to_a_page/pagename.docview.json`
* `/content/add_valid_path_to_a_page/pagename.sysview.xml`
* `/etc.xml`
* `/content.feed.xml`
* `/content.rss.xml`
* `/content.feed.html`
* `/content/add_valid_page.html?debug=layout`
* `/projects`
* `/tagging`
* `/etc/replication.html`
* `/etc/cloudservices.html`
* `/welcome`

在终端或命令提示符下发出以下命令，以确定是否启用了匿名写访问。 您不应能将数据写入节点。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

在终端或命令提示符下发出以下命令以尝试使Dispatcher缓存失效，并确保收到代码404响应：

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## 启用对虚URL的访问{#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

配置Dispatcher以启用对为您的AEM页面配置的虚URL的访问权限。

启用对虚URL的访问后，Dispatcher会定期调用在呈现实例上运行的服务，以获取虚URL列表。 Dispatcher将此列表存储在本地文件中。 当由于`/filter`部分中的过滤器而拒绝页面请求时，Dispatcher将查阅虚URL的列表。 如果拒绝的URL在列表中，则Dispatcher允许访问虚URL。

要启用对虚URL的访问，请在`/farms`部分添加`/vanity_urls`部分，如下例所示：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls`部分包含以下属性：

* `/url`:在呈现实例上运行的虚URL服务的路径。此属性的值必须为`"/libs/granite/dispatcher/content/vanityUrls.html"`。

* `/file`:Dispatcher存储虚URL列表的本地文件的路径。确保Dispatcher对此文件具有写权限。
* `/delay`:（秒）对虚URL服务的调用之间间隔的时间。

>[!NOTE]
>
>如果您的呈现器是AEM的实例，则必须从Software Distribution](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components)安装[FanityURLS-Components包才能启用虚URL服务。 （有关更多详细信息，请参阅[Software Distribution](https://experienceleague.adobe.com/docs/experience-manager-65/administering/contentmanagement/package-manager.html?lang=en#software-distribution)。）

请按照以下过程启用对虚URL的访问权限。

1. 如果您的呈现服务是AEM实例，请在发布实例上安装“com.adobe.granite.dispatcher.vanityurl.content”包（请参阅上面的注释）。
1. 对于您为AEM或CQ页面配置的每个虚URL，请确保[`/filter`](#configuring-access-to-content-filter)配置拒绝该URL。 如有必要，请添加一个过滤器以拒绝URL。
1. 在`/farms`下添加`/vanity_urls`部分。
1. 重新启动Apache Web服务器。

## 转发联合请求 — /propagateSyncPost {#forwarding-syndication-requests-propagatesyndpost}

联合请求通常仅面向Dispatcher，因此默认情况下，它们不会发送到呈现器(例如，AEM实例)。

如有必要，请将`/propagateSyndPost`属性设置为`"1"`以将联合请求转发到Dispatcher。 如果已设置，则必须确保过滤器部分中不拒绝POST请求。

## 配置调度程序缓存 — /cache {#configuring-the-dispatcher-cache-cache}

`/cache`部分控制Dispatcher如何缓存文档。 配置多个子属性以实施缓存策略：

* `/docroot`
* `/statfile`
* `/serveStaleOnError`
* `/allowAuthorized`
* `/rules`
* `/statfileslevel`
* `/invalidate`
* `/invalidateHandler`
* `/allowedClients`
* `/ignoreUrlParams`
* `/headers`
* `/mode`
* `/gracePeriod`
* `/enableTTL`

缓存部分的示例如下所示：

```xml
/cache
  {
  /docroot "/opt/dispatcher/cache"
  /statfile  "/tmp/dispatcher-website.stat"
  /allowAuthorized "0"

  /rules
    {
    # List of files that are cached
    }

  /invalidate
    {
    # List of files that are auto-invalidated
    }
  }
  
```

>[!NOTE]
>
>对于权限敏感型缓存，请阅读[Caching Secured Content](permissions-cache.md)。

### 指定缓存目录{#specifying-the-cache-directory}

`/docroot`属性标识存储缓存文件的目录。

>[!NOTE]
>
>值必须与Web服务器的文档根路径完全相同，以便Dispatcher和Web服务器处理相同的文件。\
>Web服务器在使用调度程序缓存文件时负责提供正确的状态代码，因此，它也必须找到它。

如果您使用多个场，则每个场必须使用不同的文档根。

### 命名Statfile {#naming-the-statfile}

`/statfile`属性标识要用作statfile的文件。 Dispatcher使用此文件注册最近内容更新的时间。 statfile可以是Web服务器上的任何文件。

statfile没有内容。 更新内容后，Dispatcher会更新时间戳。 默认的statfile名为`.stat`，存储在docroot中。 Dispatcher阻止访问statfile。

>[!NOTE]
>
>如果配置了`/statfileslevel`，则Dispatcher将忽略`/statfile`属性并使用`.stat`作为名称。

### 在出现{#serving-stale-documents-when-errors-occur}错误时提供过时的文档

`/serveStaleOnError`属性控制当渲染服务器返回错误时，Dispatcher是否返回无效的文档。 默认情况下，当处理statfile并使缓存内容失效时，Dispatcher会在下次请求时删除缓存内容。

如果将`/serveStaleOnError`设置为`"1"`，则除非呈现服务器返回成功响应，否则Dispatcher不会从缓存中删除无效的内容。 来自AEM的5xx响应或连接超时导致Dispatcher提供过时的内容，并使用HTTP状态111做出响应（重新验证失败）。

### 使用身份验证时缓存{#caching-when-authentication-is-used}

`/allowAuthorized`属性控制是否缓存包含以下任何身份验证信息的请求：

* `authorization`标头
* 名为`authorization`的Cookie
* 名为`login-token`的Cookie

默认情况下，不会缓存包含此身份验证信息的请求，因为在将缓存的文档返回给客户端时，不会执行身份验证。 此配置可阻止Dispatcher向没有必要权限的用户提供缓存的文档。

但是，如果您的要求允许缓存已验证的文档，请将`/allowAuthorized`设置为：

`/allowAuthorized "1"`

>[!NOTE]
>
>要启用会话管理（使用`/sessionmanagement`属性），必须将`/allowAuthorized`属性设置为`"0"`。

### 指定要缓存的文档{#specifying-the-documents-to-cache}

`/rules`属性控制根据文档路径缓存哪些文档。 无论`/rules`属性如何，Dispatcher在以下情况下都永远不会缓存文档：

* 如果请求URI包含问号(`?`)。
   * 这通常表示动态页面，例如不需要缓存的搜索结果。
* 缺失文件扩展名。
   * Web 服务器需要扩展名来确定文档类型（比如 MIME 类型）。
* 设置了身份验证标头（此项可进行配置）.
* 如果AEM实例使用以下标头做出响应：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>GET 或 HEAD（针对 HTTP 标头）方法可由 Dispatcher 缓存。有关响应标头缓存的其他信息，请参阅[缓存HTTP响应标头](#caching-http-response-headers)一节。

`/rules`属性中的每个项目都包括[`glob`](#designing-patterns-for-glob-properties)模式和类型：

* `glob`模式用于匹配文档的路径。
* 类型指示是否缓存与`glob`模式匹配的文档。 该值可以是允许（缓存文档）或拒绝（始终渲染文档）。

如果您没有动态页面（除了上述规则已排除的页面之外），则可以配置Dispatcher以缓存所有内容。 其规则部分如下所示：

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

有关全局属性的信息，请参阅[为全局属性设计模式](#designing-patterns-for-glob-properties)。

如果您页面的某些部分是动态的（例如，新闻应用程序），或者位于已关闭的用户组内，则可以定义例外：

>[!NOTE]
>
>不得缓存已关闭的用户组，因为未为缓存的页面检查用户权限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**压缩**

在Apache Web服务器上，您可以压缩缓存的文档。 如果客户端提出压缩请求，则压缩允许Apache以压缩形式返回文档。 通过启用Apache模块`mod_deflate`可自动进行压缩，例如：

```xml
AddOutputFilterByType DEFLATE text/plain
```

默认情况下，该模块随Apache 2.x一起安装。

<!-- 

Comment Type: draft

<note type="note"> 
 <p>Depending on the Apache web server version you can enable <span class="code">gzip</span> compression as follows:</p> 
 <ul> 
  <li>For Apache 1.3 you can enable the <span class="code">mod_gzip </span>module. The module must be downloaded and installed.</li> 
  <li>For Apache 2.x you can enable the <span class="code">mod_deflate</span> module. The module is installed by default with Apache 2.x.<br /> </li> 
 </ul> 
 <p> </p> 
</note>

 -->

<!-- 

Comment Type: draft

<p>The following rule caches all documents in compressed form; Apache can return either the uncompressed or the compressed form to the client:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /rules!!discoiqbr!!&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/rulelabel&nbsp;&nbsp;{&nbsp;&nbsp;/glob&nbsp;"*"&nbsp;/type&nbsp;"allow"&nbsp;&nbsp;/compress&nbsp;"gzip"&nbsp;}!!discoiqbr!!&nbsp;&nbsp;} 
</codeblock>

 -->

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-11-13T09:23:24.326-0500

<p>Hidden the <span class="code">mod_gzip</span> content as requested in CQDOC-11124.</p>

 -->

### 按文件夹级别{#invalidating-files-by-folder-level}使文件失效

使用`/statfileslevel`属性根据缓存文件的路径使其失效：

* Dispatcher在从Docroot文件夹到您指定的级别的每个文件夹中创建`.stat`文件。 docroot文件夹为0级。
* 通过处理`.stat`文件，文件将失效。 将`.stat`文件的上次修改日期与缓存文档的上次修改日期进行比较。 如果`.stat`文件较新，则会重新获取文档。

* 当位于某一级别的文件失效时，将处理从docroot **到**&#x200B;的所有&#x200B;**`.stat`文件，其中无效文件或配置的`statsfilevel`（以较小者为准）的级别。**

   * 例如：如果将`statfileslevel`属性设置为6，并且在级别5中某个文件失效，则将处理从docroot到5的每个`.stat`文件。 继续此示例，如果文件在级别7失效，则每次。 `stat` 文件从docroot到6将被触碰(自 `/statfileslevel = "6"`)。

只有路径&#x200B;**到无效文件的资源**&#x200B;会受到影响。 请考虑以下示例：网站使用结构`/content/myWebsite/xx/.`如果将`statfileslevel`设置为3，则会创建如下`.stat`文件：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

当`/content/myWebsite/xx`中的文件失效时，将处理从docroot向下到`/content/myWebsite/xx`的每个`.stat`文件。 这将仅适用于`/content/myWebsite/xx`，而不适用于`/content/myWebsite/yy`或`/content/anotherWebSite`。

>[!NOTE]
>
>通过发送额外的标头`CQ-Action-Scope:ResourceOnly`，可以阻止失效。 这可用于刷新特定资源，而不使缓存的其他部分失效。 有关更多详细信息，请参阅[此页面](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html)和[手动使调度程序缓存失效](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/page-invalidate.html?lang=en#configuring)。

>[!NOTE]
>
>如果为`/statfileslevel`属性指定值，则将忽略`/statfile`属性。

### 自动使缓存文件{#automatically-invalidating-cached-files}失效

`/invalidate`属性定义更新内容时自动失效的文档。

如果自动失效，Dispatcher不会在内容更新后删除缓存的文件，但会在下次请求时检查其有效性。 缓存中未自动失效的文档将保留在缓存中，直到内容更新明确删除它们为止。

HTML页面通常使用自动失效功能。 HTML页面通常包含指向其他页面的链接，因此很难确定内容更新是否会影响页面。 为确保所有相关页面在内容更新时失效，请自动使所有HTML页面失效。 以下配置会使所有HTML页面失效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

有关全局属性的信息，请参阅[为全局属性设计模式](#designing-patterns-for-glob-properties)。

激活`/content/wknd/us/en`时，此配置会导致以下活动：

* 所有模式为en的文件。*已从`/content/wknd/us`文件夹中删除。
* 将删除`/content/wknd/us/en./_jcr_content`文件夹。
* 所有与`/invalidate`配置匹配的其他文件不会立即删除。 下次请求时，这些文件将被删除。 在我们的示例中，未删除`/content/wknd.html`，将在请求`/content/wknd.html`时将其删除。

如果您提供自动生成的PDF和ZIP文件供下载，则可能还必须自动使这些文件失效。 配置示例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM与Adobe Analytics的集成可在您网站的`analytics.sitecatalyst.js`文件中提供配置数据。 随Dispatcher提供的示例`dispatcher.any`文件包含以下此文件的失效规则：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自定义失效脚本{#using-custom-invalidation-scripts}

利用`/invalidateHandler`属性，可定义一个脚本，该脚本将为Dispatcher收到的每个失效请求调用。

使用以下参数调用：

* 句柄 — 无效的内容路径
* 操作 — 复制操作（例如激活、停用）
* 操作范围 — 复制操作的范围(为空，除非发送`CQ-Action-Scope: ResourceOnly`的标头，否则请参阅[使从AEM中缓存的页面失效以了解详细信息)](page-invalidate.md)

这可用于覆盖许多不同的用例，如使其他应用程序特定缓存失效，或处理页面的外部化URL及其在docroot中的位置与内容路径不匹配的情况。

以下示例脚本将每个无效请求记录到一个文件。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### 失效处理程序脚本{#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### 限制可刷新缓存{#limiting-the-clients-that-can-flush-the-cache}的客户端

`/allowedClients`属性定义允许刷新缓存的特定客户端。 通配模式与IP匹配。

以下示例：

1. 拒绝访问任何客户
1. 明确允许访问localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

有关全局属性的信息，请参阅[为全局属性设计模式](#designing-patterns-for-glob-properties)。

>[!CAUTION]
>
>建议您定义`/allowedClients`。
>
>如果未执行此操作，任何客户端都可以发出调用以清除缓存；如果反复执行此操作，可能会严重影响网站性能。

### 忽略URL参数{#ignoring-url-parameters}

`ignoreUrlParams`部分定义在确定页面是缓存还是从缓存中传送时，将忽略哪些URL参数：

* 当请求URL包含所有被忽略的参数时，将缓存页面。
* 当请求URL包含一个或多个不会忽略的参数时，不会缓存页面。

当某个页面的参数被忽略时，会在首次请求该页面时缓存该页面。 无论请求中的参数值如何，缓存的页面都会提供该页面的后续请求。

要指定要忽略的参数，请将全局规则添加到`ignoreUrlParams`属性：

* 要忽略参数，请创建允许该参数的全局属性。
* 要阻止缓存页面，请创建一个全局属性以拒绝该参数。

以下示例会导致Dispatcher忽略`q`参数，以便缓存包含q参数的请求URL:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

使用示例`ignoreUrlParams`值，由于`q`参数被忽略，以下HTTP请求会导致页面被缓存：

```xml
GET /mypage.html?q=5
```

使用示例`ignoreUrlParams`值，以下HTTP请求会导致页面缓存&#x200B;**not**，因为`p`参数未被忽略：

```xml
GET /mypage.html?q=5&p=4
```

有关全局属性的信息，请参阅[为全局属性设计模式](#designing-patterns-for-glob-properties)。

### 缓存HTTP响应标头{#caching-http-response-headers}

>[!NOTE]
>
>此功能在Dispatcher版本&#x200B;**4.1.11**&#x200B;中可用。

利用`/headers`属性，可定义Dispatcher要缓存的HTTP标头类型。 在向未缓存资源发出第一个请求时，所有与其中一个已配置值匹配的标头（请参阅下面的配置示例）都存储在缓存文件旁边的单独文件中。 在对缓存资源的后续请求中，存储的标头将添加到响应中。

以下是默认配置的示例：

```xml
/cache {
  ...
  /headers {
    "Cache-Control"
    "Content-Disposition"
    "Content-Type"
    "Expires"
    "Last-Modified"
    "X-Content-Type-Options"
    "Last-Modified"
  }
}
```

>[!NOTE]
>
>另请注意，不允许使用文件代换字符。 有关更多详细信息，请参阅[为全局属性设计模式](#designing-patterns-for-glob-properties)。

>[!NOTE]
>
>如果您需要Dispatcher从AEM存储和交付ETag响应标头，请执行以下操作：
>
>* 在`/cache/headers`部分中添加标头名称。
>* 在与Dispatcher相关的部分中添加以下[Apache指令](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag):

>
>
```xml
>FileETag none
>```

### 调度程序缓存文件权限{#dispatcher-cache-file-permissions}

`mode`属性指定将哪些文件权限应用于缓存中的新目录和文件。 此设置受调用进程`umask`的限制。 它是使用以下一个或多个值之和构建的八进制数字：

* `0400` 允许所有者读取。
* `0200` 允许所有者写入。
* `0100` 允许所有者在目录中搜索。
* `0040` 允许群组成员读取。
* `0020` 允许由群组成员写入。
* `0010` 允许群组成员在目录中搜索。
* `0004` 允许他人阅读。
* `0002` 允许他人写。
* `0001` 允许其他人在目录中搜索。

默认值为`0755`，该值允许所有者读取、写入或搜索，组和其他人读取或搜索。

### 限制.stat文件处理{#throttling-stat-file-touching}

使用默认的`/invalidate`属性，每次激活都会有效地使所有`.html`文件失效（当其路径与`/invalidate`部分匹配时）。 如果某个网站的流量很大，则多次后续激活会增加后端的cpu负载。 在这种情况下，最好“限制”`.stat`文件接触以保持网站响应。 您可以使用`/gracePeriod`属性执行此操作。

`/gracePeriod`属性定义在上次发生激活后，仍可从缓存中提供失效的自动失效资源的秒数。 该属性可用于设置中，否则，若进行批量激活，则会重复使整个缓存失效。 建议的值为2秒。

有关更多详细信息，请参阅上面的`/invalidate`和`/statfileslevel`部分。

### 配置基于时间的缓存失效 — /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

如果设置，`/enableTTL`属性将从后端评估响应标头，如果响应标头包含`Cache-Control` max-age或`Expires`日期，则会创建缓存文件旁边的辅助空文件，修改时间等于到期日期。 在修改时间之后请求缓存文件时，会自动从后端重新请求缓存文件。

>[!NOTE]
>
>此功能在Dispatcher的&#x200B;**4.1.11**&#x200B;或更高版本中可用。

## 配置负载平衡 — /statistics {#configuring-load-balancing-statistics}

`/statistics`部分定义Dispatcher为其对每个呈现器的响应性进行评分的文件类别。 Dispatcher使用得分确定要发送请求的呈现者。

您创建的每个类别都定义全局模式。 Dispatcher将请求内容的URI与这些模式进行比较，以确定请求内容的类别：

* 类别的顺序决定了它们与URI的比较顺序。
* 与URI匹配的第一个类别模式是文件的类别。 不再评估类别模式。

Dispatcher最多支持8个统计类别。 如果定义的类别超过8个，则仅使用前8个类别。

**渲染选择**

每次Dispatcher需要渲染的页面时，它都会使用以下算法来选择渲染器：

1. 如果请求在`renderid` Cookie中包含呈现名称，则Dispatcher会使用该呈现。
1. 如果请求不包含`renderid` Cookie，则Dispatcher会比较呈现统计信息：

   1. Dispatcher确定请求URI的类别。
   1. Dispatcher确定哪个呈现器对该类别的响应得分最低，然后选择该呈现器。

1. 如果尚未选择任何渲染，则使用列表中的第一个渲染。

呈现器类别的得分基于以前的响应时间以及Dispatcher尝试的以前失败和成功的连接。 对于每次尝试，将更新所请求URI类别的得分。

>[!NOTE]
>
>如果不使用负载平衡，则可以忽略此部分。

### 定义统计类别{#defining-statistics-categories}

为要保留用于呈现选择的统计信息的每种类型的文档定义类别。 `/statistics`部分包含`/categories`部分。 要定义类别，请在`/categories`部分下添加一行，该行的格式如下：

`/name { /glob "pattern"}`

类别`name`必须对场是唯一的。 `pattern`在[为全局属性设计模式](#designing-patterns-for-glob-properties)一节中有介绍。

为确定URI的类别，Dispatcher会将URI与每个类别模式进行比较，直到找到匹配项。 Dispatcher从列表中的第一个类别开始，并按顺序继续。 因此，请首先放置具有更具体模式的类别。

例如，默认`dispatcher.any`文件的Dispatcher定义了HTML类别和其他类别。 HTML类别更具体，因此它首先显示：

```xml
/statistics
  {
  /categories
    {
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

以下示例还包括搜索页面的类别：

```xml
/statistics
  {
  /categories
    {
      /search { /glob "*search.html" }
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

### 在调度程序统计数据{#reflecting-server-unavailability-in-dispatcher-statistics}中反映服务器不可用性

`/unavailablePenalty`属性设置在与渲染器的连接失败时应用于渲染统计信息的时间（以秒为单位）。 Dispatcher将时间添加到与请求的URI匹配的统计类别中。

例如，当无法建立到指定主机名/端口的TCP/IP连接时，将应用惩罚，原因可能是AEM未运行（且未侦听），或者由于网络相关问题。

`/unavailablePenalty`属性是`/farm`部分（`/statistics`部分的同级项）的直接子项。

如果不存在`/unavailablePenalty`属性，则使用值`"1"`。

```xml
/unavailablePenalty "1"
```

## 识别粘性连接文件夹 — {#identifying-a-sticky-connection-folder-stickyconnectionsfor}的/stickyConnections

`/stickyConnectionsFor`属性定义一个包含粘性文档的文件夹；将使用URL访问该地址。 Dispatcher将此文件夹中的所有请求从单个用户发送到同一呈现实例。 粘性连接可确保所有文档的会话数据都存在且一致。 此机制使用`renderid` Cookie。

以下示例定义到/products文件夹的置顶连接：

```xml
/stickyConnectionsFor "/products"
```

当页面由来自多个内容节点的内容组成时，请包括列出内容路径的`/paths`属性。 例如，页面包含`/content/image`、`/content/video`和`/var/files/pdfs`中的内容。 以下配置为页面上的所有内容启用置顶连接：

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### httpOnly {#httponly}

启用粘性连接后，调度程序模块会设置`renderid` Cookie。 此Cookie没有`httponly`标记，应添加该标记以增强安全性。 为此，可以在`dispatcher.any`配置文件的`/stickyConnections`节点中设置`httpOnly`属性。 属性的值（`0`或`1`）定义`renderid` Cookie是否附加了`HttpOnly`属性。 默认值为`0`，这表示不会添加属性。

有关`httponly`标记的其他信息，请阅读[此页面](https://www.owasp.org/index.php/HttpOnly)。

### 安全 {#secure}

启用粘性连接后，调度程序模块会设置`renderid` Cookie。 此Cookie没有`secure`标记，应添加该标记以增强安全性。 为此，可以在`dispatcher.any`配置文件的`/stickyConnections`节点中设置`secure`属性。 属性的值（`0`或`1`）定义`renderid` Cookie是否附加了`secure`属性。 默认值为`0`，这表示如果&#x200B;**传入请求安全，将添加属性**。 如果将该值设置为`1`，则将添加安全标志，无论传入请求是否安全。

## 处理渲染连接错误{#handling-render-connection-errors}

在呈现服务器返回500错误或不可用时配置Dispatcher行为。

### 指定运行状况检查页面{#specifying-a-health-check-page}

使用`/health_check`属性指定在发生500状态代码时要检查的URL。 如果此页还返回500状态代码，则实例将被视为不可用，并且在重试之前对渲染应用可配置的时间惩罚(`/unavailablePenalty`)。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定页面重试延迟{#specifying-the-page-retry-delay}

`/retryDelay`属性可设置Dispatcher在场呈现的连接尝试轮次之间等待的时间（以秒为单位）。 对于每轮，Dispatcher尝试连接到渲染器的最大次数是场中的渲染次数。

如果未明确定义`/retryDelay`，则Dispatcher使用值`"1"`。 默认值在大多数情况下都适用。

```xml
/retryDelay "1"
```

### 配置重试次数{#configuring-the-number-of-retries}

`/numberOfRetries`属性设置Dispatcher对呈现器执行的连接尝试的最大轮次数。 如果Dispatcher在进行此次重试后无法成功连接到呈现器，则Dispatcher返回失败的响应。

对于每轮，Dispatcher尝试连接到渲染器的最大次数是场中的渲染次数。 因此，Dispatcher尝试连接的最大次数为(`/numberOfRetries`)x（呈现次数）。

如果未明确定义该值，则其默认值为`5`。

```xml
/numberOfRetries "5"
```

### 使用故障转移机制{#using-the-failover-mechanism}

在原始请求失败时，在Dispatcher场上启用故障转移机制，以向不同呈现器重新发送请求。 启用故障转移后，Dispatcher的行为如下：

* 当对呈现器的请求返回HTTP状态503（不可用）时，Dispatcher会将该请求发送到其他呈现器。
* 当对呈现器的请求返回HTTP状态50x（503除外）时，Dispatcher会发送针对为`health_check`属性配置的页面的请求。
   * 如果运行状况检查返回500(INTERNAL_SERVER_ERROR)，则Dispatcher将原始请求发送到其他呈现器。
   * 如果运行状况检查返回HTTP状态200，则Dispatcher会将初始HTTP 500错误返回给客户端。

要启用故障转移，请将以下行添加到场（或网站）：

```xml
/failover "1"
```

>[!NOTE]
>
>要重试包含正文的HTTP请求，Dispatcher会先向呈现器发送`Expect: 100-continue`请求标头，然后再假设实际内容。 带有CQSE的CQ 5.5随后会立即回答100（继续）或错误代码。 其他Servlet容器也应支持此功能。

## 忽略中断错误 — /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此选项。 只有在看到以下日志消息时，才需要使用此功能：
>
>`Error while reading response: Interrupted system call`

如果系统调用的对象位于通过NFS访问的远程系统上，则任何面向文件系统的系统调用都可能中断`EINTR`。 这些系统调用是否可以超时或中断取决于底层文件系统在本地计算机上的装载方式。

如果您的实例具有此类配置，并且日志包含以下消息，请使用`/ignoreEINTR`参数：

`Error while reading response: Interrupted system call`

在内部，Dispatcher使用可表示为以下代码的循环从远程服务器(即AEM)读取响应：

```text
while (response not finished) {  
read more data  
}
```

当`EINTR`出现在“`read more data`”部分中时，可以生成此类消息，这些消息是在收到任何数据之前接收信号所引起的。

要忽略此类中断，可以将以下参数添加到`dispatcher.any`（`/farms`之前）：

`/ignoreEINTR "1"`

将`/ignoreEINTR`设置为`"1"`会导致Dispatcher继续尝试读取数据，直到读取完整响应。 默认值为`0`并停用选项。

## 设计全局属性的模式{#designing-patterns-for-glob-properties}

Dispatcher配置文件的几个部分使用`glob`属性作为客户端请求的选择标准。 `glob`属性的值是Dispatcher与请求方面（如请求资源的路径或客户端的IP地址）进行比较的模式。 例如，`/filter`部分中的项目使用`glob`模式来标识Dispatcher执行或拒绝操作的页面的路径。

`glob`值可以包含通配符和字母数字字符来定义模式。

| 通配符 | 描述 | 示例 |
|--- |--- |--- |
| `*` | 匹配字符串中任意字符的零个或多个连续实例。 匹配的最终字符由以下任一情况确定：<br/>字符串中的字符与模式中的下一个字符匹配，并且模式字符具有以下特征：<br/><ul><li>不是*</li><li>不是？</li><li>文字字符（包括空格）或字符类。</li><li>到达模式的结尾。</li></ul>在字符类中，字符会按字面方式进行解释。 | `*/geo*` 匹配节点和节点 `/content/geometrixx` 下的任何 `/content/geometrixx-outdoors` 页面。以下HTTP请求与全局模式匹配：<br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>匹配节点下的任意 `/content/geometrixx-outdoors` 页面。例如，以下HTTP请求与全局模式匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 匹配任意单个字符。 使用外部字符类。 在字符类中，该字符是字面解释的。 | `*outdoors/??/*`<br/> 匹配geometrixx-outdoors站点中任何语言的页面。例如，以下HTTP请求与全局模式匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>以下请求与全局模式不匹配：  <br/><ul><li>&quot;GET/content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | 取消标记字符类的开头和结尾。 字符类可以包括一个或多个字符范围和单个字符。<br/>如果目标字符与字符类中的任意字符或在定义的范围内匹配，则会发生匹配。<br/>如果不包括右括号，则图案不产生匹配。 | `*[o]men.html*`<br/> 匹配以下HTTP请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>匹配以下HTTP请求：  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示字符范围。 用于字符类。  在字符类之外，该字符将以字面形式进行解释。 | `*[m-p]men.html*` 匹配以下HTTP请求：  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定后续的字符或字符类。 仅用于否定字符类中的字符和字符范围。 等同于`^ wildcard`。 <br/>在字符类之外，该字符将以字面形式进行解释。 | `*[!o]men.html*`<br/> 匹配以下HTTP请求：  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>与以下HTTP请求不匹配：  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定后续的字符或字符范围。 仅用于否定字符类中的字符和字符范围。 等同于`!`通配符。 <br/>在字符类之外，该字符将以字面形式进行解释。 | 应用`!`通配符的示例，将示例模式中的`!`字符替换为`^`字符。 |


<!--- need to troubleshoot table

The following table describes the wildcard characters.

<table border="1" cellpadding="1" cellspacing="0" width="100%"> 
 <tbody> 
  <tr> 
   <th>Wildcard character</th> 
   <th>Description</th> 
   <th>Examples</th> 
  </tr> 
  <tr> 
   <td>*</td> 
   <td><p>Matches zero or more contiguous instances of any character in the string. The final character of the match is determined by either of the following situations:</p> 
    <ul> 
     <li>A character in the string matches the next character in the pattern, and the pattern character has the following characteristics: 
      <ul> 
       <li>Not a *</li> 
       <li>Not a ?</li> 
       <li>A literal character (including a space) or a character class.</li> 
      </ul> </li> 
     <li>The end of the pattern is reached.</li> 
    </ul> <p>Inside a character class, the character is interpreted literally.</p> </td> 
   <td><p>*/geo*</p> <p>Matches any page below the /content/geometrixx node and the /content/geometrixx-outdoors node. The following HTTP requests match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx/en.html"</li> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> <p>*outdoors/*</p> <p>Matches any page below the /content/geometrixx-outdoors node. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html"</li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>?</td> 
   <td><p>Matches any single character. Use outside character classes.</p> <p>Inside a character class, this character is interpreted literally.</p> </td> 
   <td><p>*outdoors/??/*</p> <p>Matches the pages for any language in the geometrixx-outdoors site. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>The following request does not match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>[ and ]</td> 
   <td><p>Demarks the beginning and end of a character class.</p> <p>Character classes can include one or more character ranges and single characters.</p> <p>A match occurs if the target character matches any of the characters in the character class, or within a defined range.</p> <p>If the closing bracket is not included, the pattern produces no matches.</p> <p></p> <p></p> <p></p> </td> 
   <td><p>*[o]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p>*[o/]men.html*</p> <p>Matches the following HTTP requests: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
     <li> "GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>-</td> 
   <td><p>Denotes a range of characters. For use in character classes.<br /> </p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[m-p]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>!</td> 
   <td><p>Negates the character or character class that follows. Use only for negating characters and character ranges inside character classes. Equivalent to the ^ wildcard.</p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[!o]men.html*</p> <p>Matches the following HTTP request: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>Does not match the following HTTP request</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
    </ul> <p>*[!o!/]men.html*</p> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" or "GET /content/geometrixx-outdoors/en/men. html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>^</td> 
   <td><p>Negates the character or character range that follows. Use for negating only characters and character ranges inside character classes. Equivalent to the ! wildcard character.</p> <p>Outside of a character class, this charcter is interpreted literally.</p> </td> 
   <td>The examples for the ! wildcard character apply, substituting the ! characters in the example patterns with ^ characters.</td> 
  </tr> 
 </tbody> 
</table>
-->

## 记录 {#logging}

在Web服务器配置中，您可以设置：

* Dispatcher日志文件的位置。
* 日志级别。

有关更多信息，请参阅Web服务器文档和Dispatcher实例的自述文件。

**Apache旋转/管道日志**

如果使用&#x200B;**Apache** Web服务器，则可以对旋转和/或管道日志使用标准功能。 例如，使用管道日志：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

此操作将自动旋转：

* 调度程序日志文件；扩展中具有时间戳(`logs/dispatcher.log%Y%m%d`)。
* 每周(60 x 60 x 24 x 7 = 604800秒)。

请参阅Log Rotation和Puical Logs上的Apache Web服务器文档；例如[Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html)。

>[!NOTE]
>
>安装后，默认日志级别很高（即级别3 =调试），因此Dispatcher记录所有错误和警告。 这在最初阶段非常有用。
>
>但是，这需要额外的资源，因此当Dispatcher根据您的要求&#x200B;*顺利运行*&#x200B;时，您可以（应该）降低日志级别。

### 跟踪日志记录{#trace-logging}

在Dispatcher的其他增强功能中，版本4.2.0还引入了跟踪日志记录。

这比调试日志记录级别更高，日志中显示了其他信息。 它会为添加日志记录：

* 转发标头的值；
* 正在应用于特定操作的规则。

您可以通过在Web服务器中将日志级别设置为`4`来启用跟踪日志记录。

以下是启用跟踪的日志示例：

```xml
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Host] = "localhost:8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[User-Agent] = "curl/7.43.0"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Accept] = "*/*"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Client-Cert] = "(null)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Via] = "1.1 localhost:8443 (dispatcher)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-For] = "::1"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL] = "on"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Cipher] = "DHE-RSA-AES256-SHA"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Session-ID] = "ba931f5e4925c2dde572d766fdd436375e15a0fd24577b91f4a4d51232a934ae"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-Port] = "8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Server-Agent] = "Communique-Dispatcher"
```

请求与阻止规则匹配的文件时记录的事件：

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## 确认基本操作{#confirming-basic-operation}

要确认Web服务器、Dispatcher和AEM实例的基本操作和交互，您可以执行以下步骤：

1. 将`loglevel`设置为`3`。

1. 启动Web服务器；这也会启动Dispatcher。
1. 启动AEM实例。
1. 检查Web服务器和Dispatcher的日志和错误文件。
   * 根据您的Web服务器，您应会看到以下消息：
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` 和
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 通过Web服务器浏览网站。 确认内容是否按需要显示。\
   例如，在本地安装中，AEM在端口`4502`上运行，而Web服务器在`80`上则使用以下两种方式访问网站控制台：
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * 结果应该相同。 使用相同机制确认对其他页面的访问权限。

1. 检查缓存目录是否已填写。
1. 激活页面以检查缓存是否正确刷新。
1. 如果一切运行正常，您可以将`loglevel`降至`0`。

## 使用多个 Dispatcher {#using-multiple-dispatchers}

在复杂设置中，您可以使用多个 Dispatcher。例如，您可以使用：

* 一个 Dispatcher 用于在内联网上发布网站
* 第二个 Dispatcher，通过不同的地址和不同的安全设置，在内联网上发布相同的内容。

在这种情况下，请确保每个请求只通过一个 Dispatcher。一个 Dispatcher 不能处理来自另一个 Dispatcher 的请求。因此，请确保两个 Dispatcher 都能直接访问 AEM 网站。

## 调试 {#debugging}

在向请求添加标头`X-Dispatcher-Info`时，Dispatcher会回答目标是否已缓存、从缓存返回还是根本不可缓存。 响应标头`X-Cache-Info`以可读形式包含此信息。 您可以使用这些响应标头来调试与由Dispatcher缓存的响应有关的问题。

默认情况下，此功能未启用，因此要包含响应标头`X-Cache-Info`，场必须包含以下条目：

```xml
/info "1"
```

例如，

```xml
/farm
{
    /mywebsite
    {
        # Include X-Cache-Info response header if X-Dispatcher-Info is in request header
        /info "1"
    }
}
```

此外，`X-Dispatcher-Info`标头不需要值，但如果使用`curl`进行测试，则必须提供值才能发送标头，例如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

以下是包含`X-Dispatcher-Info`将返回的响应标头的列表：

* **缓存**\
   目标文件包含在缓存中，调度程序已确定交付该文件是有效的。
* **缓存**\
   目标文件未包含在缓存中，调度程序已确定缓存输出并交付该输出是有效的。
* **缓存：stat文件更新**
目标文件包含在缓存中，但是，由更新的stat文件使其失效。调度程序将删除目标文件，从输出中重新创建并交付该文件。
* **无法缓存：无文**
档根场的配置不包含文档根（配置元素） 
`cache.docroot`)。
* **无法缓存：缓存文件路径太长**\
   目标文件（文档根文件和URL文件的拼接）超过了系统上可能的最长文件名。
* **无法缓存：临时文件路径太长**\
   临时文件名模板超出了系统上可能的最长文件名。 调度程序会先创建临时文件，然后再实际创建或覆盖缓存的文件。 临时文件名是目标文件名，其后面附加有字符`_YYYYXXXXXX`，其中将替换`Y`和`X`以创建唯一名称。
* **无法缓存：请求URL没有扩展**\
   请求URL没有扩展名，或者文件扩展名后有一个路径，例如：`/test.html/a/path`。
* **无法缓存：请求不是GET或**
HEAD HTTP方法既不是GET也不是HEAD。调度程序假定输出将包含不应缓存的动态数据。
* **无法缓存：包含查询字符串的请求**\
   请求包含查询字符串。 调度程序假定输出取决于给定的查询字符串，因此不进行缓存。
* **无法缓存：会话管理器未验证**\
   场的缓存由会话管理器（配置包含`sessionmanagement`节点）管理，并且请求中不包含相应的身份验证信息。
* **无法缓存：请求包含授权**\
   不允许场缓存输出(`allowAuthorized 0`)，并且请求包含身份验证信息。
* **无法缓存：target是一个目录**\
   目标文件是目录。 这可能会指出一些概念性错误，即URL和某些子URL都包含可缓存的输出，例如，如果对`/test.html/a/file.ext`的请求排在首位并包含可缓存的输出，则调度程序将无法将后续请求的输出缓存到`/test.html`。
* **无法缓存：请求URL具有尾随斜杠**\
   请求URL具有尾随斜杠。
* **无法缓存：请求URL不在缓存规则中**\
   场的缓存规则明确拒绝缓存某些请求URL的输出。
* **无法缓存：授权检查程序被拒绝访问**\
   场的授权检查程序拒绝访问缓存的文件。
* **无法缓存：会话无**
效场的缓存受会话管理器(配置包含节 `sessionmanagement` 点)控制，并且用户的会话无效或不再有效。
* **无法缓存：响应包`no_cache`**
含远程服务器返回 
`Dispatcher: no_cache` 标头，禁止调度程序缓存输出。
* **无法缓存：响应内容长**
度为零响应的内容长度零；调度程序不会创建零长度文件。
