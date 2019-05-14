---
title: 配置调度程序
seo-title: 配置调度程序
description: 了解如何配置Dispatcher。
seo-description: 了解如何配置Dispatcher。
uuid: 253ef0f7-2491-4cec-ab22-97439df29 fd6
cmgrlastmodified: 01.11.2007082229[aheimoz]
pageversionid: '1193211344162'
topic-tags: 调度程序
content-type: 引用
discoiquuid: aefee8e-bb34-42a7-9a5 e-b7 d0 e84391 a
translation-type: tm+mt
source-git-commit: 2f0ca874c23cb7aecbcedc22802c46a295bb4d75

---


# 配置调度程序{#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher版本独立于AEM。如果您遵循了一个指向Dispatcher文档的链接，则可能已重定向到该页面，该链接嵌入到AEM的先前版本的文档中。

以下部分介绍了如何配置Dispatcher的各个方面。

## 支持IPv和IPv6 {#support-for-ipv-and-ipv}

AEM和Dispatcher的所有元素都可以安装在IPv和IPv网络中。请参阅 [IPv和IPv6](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html#AdditionalPlatformNotes)。

## 调度程序配置文件 {#dispatcher-configuration-files}

默认情况下，Dispatcher配置存储在 `dispatcher.any` 文本文件中，但您可以在安装过程中更改该文件的名称和位置。

配置文件包含一系列用于控制Dispatcher行为的单值或多值属性：

* 属性名称前缀为正斜杠 `/`。
* 多值属性使用大括号括住子项目 `{ }`。

示例配置的构造如下：

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

您可以包含其他用于配置的文件：

* 如果配置文件较大，则可以将其拆分为几个较小的文件(更易于管理)，然后包括这些文件。
* 包括自动生成的文件。

例如，要在/farms配置中包括MyFarm. any，请使用以下代码：

```xml
/farms
  {
  $include "myFarm.any"
  }
```

使用星号(“*”)作为一个通配符，指定要包含的文件范围。

例如，如果文件 `farm_1.any` 之间包含 `farm_5.any` 农场配置到5，则可以按如下方式包括这些文件：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用环境变量 {#using-environment-variables}

您可以在调度程序.任何文件中使用字符串值属性中的环境变量，而不是对这些值进行硬编码。要包含环境变量的值，请使用该格式 `${variable_name}`。

例如，如果调度程序与缓存目录位于同一目录中，则可以使用 [docroot](dispatcher-configuration.md#main-pars-title-30) 属性的以下值：

```xml
/docroot "${PWD}/cache"
```

例如，如果您创建一个名为 `PUBLISH_IP` 存储AEM发布实例主机名的环境变量，则可以使用 [/renders](dispatcher-configuration.md#main-pars-127-25-0008) 属性的以下配置：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 命名调度程序实例 {#naming-the-dispatcher-instance-name}

使用 `/name` 该属性指定唯一名称以标识您的调度程序实例。`/name` 该属性是配置结构中的顶级属性。

## 定义农场 {#defining-farms-farms}

`/farms` 该属性定义一组或多组调度程序行为，其中每个集合都与不同的网站或URL相关联。`/farms` 该属性可包括一个农场或多个农场：

* 当您希望Dispatcher以相同的方式处理所有网页或网站时使用单一农场。
* 当网站或不同网站的不同区域需要不同的调度程序行为时，创建多个农场。

`/farms` 该属性是配置结构中的顶级属性。要定义农场，请向 `/farms` 该属性添加子属性。使用可唯一标识调度程序实例中的农场的属性名称。

`/*farmname*` 该属性具有多个值，并且包含定义Dispatcher行为的其他属性：

* 农场应用的页面URL。
* 用于渲染文档的一个或多个服务URL(通常是AEM发布实例)。
* 用于对多个文档渲染器进行负载平衡的统计数据。
* 其他一些行为，如要缓存的文件和位置。

该值可能包含任何字母数字(a-z、0-9)字符。以下示例显示了两个名称命名的农场的骨骼定义 `/daycom` 和 `/docsdaycom`：

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
>如果您使用多个渲染农场，则会对列表进行底部评估。这在为您的网站定义 [虚拟主机](dispatcher-configuration.md#main-pars-117-15-0006) 时尤为相关。

每个农场属性都可以包含以下子属性：

| 属性名称 | 描述 |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | 默认主页(可选)(仅限IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | 要传递的客户端HTTP请求的标题。 |
| [/virtualhosts](#identifying-virtual-hosts-virtual-hosts) | 此农场的虚拟主机。 |
| [/sessionmanagement](#enabling-secure-sessions-session-management) | 支持会话管理和身份验证。 |
| [/renders](#defining-page-renderers-renders) | 提供渲染页面(通常是AEM发布实例)的服务器。 |
| [/filter](#configuring-access-to-content-filter) | 定义Dispatcher支持访问的URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 配置访问虚URL。 |
| [/propagateSyndPost](#forwarding-syndication-requests-propagate-syndpost) | 支持转发请求请求。 |
| [/cache](#configuring-the-dispatcher-cache-cache) | 配置缓存行为。 |
| [/statistics](#configuring-load-balancing-statistics) | 为负载平衡计算定义统计类别。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-sticky-connections-for) | 包含粘性文档的文件夹。 |
| [/health_check](#specifying-a-health-check-page) | 用于确定服务器可用性的URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重试失败的连接之前的延迟。 |
| [/unavailablePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | 影响负载平衡计算统计数据的处罚。 |
| [/failover](#using-the-fail-over-mechanism) | 在原始请求失败时，将请求重置为不同的渲染。 |

## 指定默认页面(仅IIS)-/homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>`/homepage`参数(仅IIS)不再有效。相反，您应使用 [IIS URL重写模块](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)。
>
>如果您使用的是Apache，则应使用 `mod_rewrite` 模块。有关有关(例如 `mod_rewrite`[Apache2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html))的信息，请参阅Apache网站文档。使用 `mod_rewrite`时，使用旗标** [&#39;passthrough| PT&#39;(传递到下一个句柄)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)**以强制重写引擎将 `uri` 内部 `request_rec` 结构的字段设置为 `filename` 字段的值。

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

## 指定要通过的HTTP头 {#specifying-the-http-headers-to-pass-through-clientheaders}

`/clientheaders` 该属性定义Dispatcher从客户端HTTP请求传递给渲染器(AEM实例)的HTTP头列表。

默认情况下，Dispatcher将标准HTTP头转发到AEM实例。在某些情况下，您可能希望转发其他标题或删除特定标题：

* 添加AEM实例在HTTP请求中所需的标题，如自定义标题。
* 删除仅与Web服务器相关的标题，如身份验证标题。

如果您自定义要传递的标题集，则必须指定一个完整的标题列表，包括通常包含的标题。

例如，处理发布实例页面激活请求的调度程序实例需要部分 `PATH` 标题 `/clientheaders` 。`PATH` 标题支持复制代理与调度程序之间的通信。

以下代码是以下代码的示例配置 `/clientheaders`：

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

## 识别虚拟主机 {#identifying-virtual-hosts-virtualhosts}

`/virtualhosts` 该属性定义Dispatcher接受此农场的所有主机名/URI组合的列表。您可以使用星号(“*”)作为通配符。/ `virtualhosts` 属性的值使用以下格式：

```xml
[scheme]host[uri][*]
```

* `scheme`: (Optional) Either `https://` or `https://.`
* `host`：主机的名称或IP地址(如有必要)。(请参阅 [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23)
* `uri`：(可选)资源的路径。

以下示例配置处理MyCompany的.com和. ch域以及MySubDivision的所有域的请求：

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

以下配置处理 *所有* 请求：

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解析虚拟主机 {#resolving-the-virtual-host}

当Dispatcher收到HTTP或HTTPS请求时，它会找到最匹配请求的和 `host,``uri``scheme` 头的虚拟主机值。Dispatcher按以下顺序评估 `virtualhosts` 属性中的值：

* 调度程序从最低农场开始，并在调度程序.任何文件中向上行进。
* 对于每个农场，调度程序以 `virtualhosts` 属性中最高的值开头，并向下遍历值列表。

Dispatcher以以下方式查找匹配的虚拟主机值：

* 系统会使用一个与所有三个 `host`、所有 `scheme`请求和请求的 `uri` 全部匹配的首次遇到的虚拟主机。
* 如果没有 `virtualhosts` 值和 `scheme``uri` 与请求的 `scheme` 请求相匹配 `uri` 的部分，则会使用与请求匹配 `host` 的首次遇到的虚拟主机。
* 如果没有 `virtualhosts` 任何值的主机部分与请求的主机相匹配，则使用最高的农场虚拟主机。

因此，应将默认虚拟主机放在调度程序的最顶部农场(任何文件)中 `virtualhosts` 的属性顶部。

### 虚拟主机分辨率示例 {#example-virtual-host-resolution}

以下示例代表调度程序中的一个片段，该代码用于定义两个调度程序农场，每个农场定义一 `virtualhosts` 个属性。

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

| 请求URL | 解决了虚拟主机 |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/*;` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 启用安全会话-/sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized`**必须** 在 `"0"``/cache` 部分中设置才能启用此功能。

创建一个安全会话以访问渲染农场，以便用户需要登录才能访问农场中的任何页面。登录后，用户可以访问农场中的所有页面。有关将此功能与CUG一起使用的信息，请参阅 [创建已关闭的用户组](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/cug.html#CreatingTheUserGroupToBeUsed) 。

`/sessionmanagement` 该属性是一个子属性 `/farms`。

>[!CAUTION]
>
>如果网站的部分使用不同的访问要求，则需要定义多个农场。

**/sessionmanagement** 有几个子参数：

**/directory** (强制)

存储会话信息的目录。如果该目录不存在，则创建该目录。

**/encode** (可选)

会话信息的编码方式。使用“md5”加密加密算法，或使用“十六进制”进行十六进制编码。如果您加密会话数据，有权访问文件系统的用户无法读取会话内容。默认值为“md5”。

**/header** (可选)

存储授权信息的HTTP头或cookie的名称。如果将信息存储在http标题中，请使用 `HTTP:<*header-name*>`。要将信息存储在cookie中，请使用 `Cookie:<header-name>`。如果未指定 `HTTP:authorization` 值，则会使用值。

**/timeout** (可选)

会话在最后一次使用之后超时的秒数。如果未指定“800”，则会话将在用户最后一次请求后的13分钟超时超时。

示例配置如下所示：

```xml
/sessionmanagement 
  { 
  /directory "/usr/local/apache/.sessions" 
  /encode "md5" 
  /header "HTTP:authorization" 
  /timeout "800" 
  }
```

## 定义页面渲染器 {#defining-page-renderers-renders}

/renders属性定义Dispatcher发送文档请求请求的URL。以下示例 `/renders` 部分标识了一个用于渲染的AEM实例：

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

以下示例/renders部分标识与Dispatcher在同一台计算机上运行的AEM实例：

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

### 渲染选项 {#renders-options}

**/timeout**

指定访问AEM实例(以毫秒为单位)的连接超时。默认值为“0”，导致调度程序等待失败。

**/receiveTimeout**

指定允许响应的时间(以毫秒为单位)。默认值为“6000”，导致Dispatcher等待10分钟。设置为“0”可完全消除超时。\
如果在解析响应头时到达超时，则返回HTTP状态504(不良网关)。如果在读取响应主体时达到超时，Dispatcher将返回对客户端的不完整响应，但删除可能已编写的任何缓存文件。

**/ipv4**

指定Dispatcher是否使用 `getaddrinfo` 函数(for IPv6)或 `gethostbyname` 函数(for IPv4)获取渲染的IP地址。要使用的值 `getaddrinfo` 的值。要使用的值 `gethostbyname` 有个。默认值为 0。

getaddrinfo函数返回IP地址列表。Dispatcher迭代地址列表，直到其建立TCP/IP连接。因此，当渲染主机名与多个IP地址关联时，如果主机为getaddrinfo函数返回一个始终处于同一顺序的IP地址列表，则IPv属性很重要。在这种情况下，您应使用gethostbyname函数，以便调度程序与调度程序连接的IP地址为randomized。

Amazon Elastic Load Dialization(ELB)是一种可响应getaddrinfo的服务，它具有一个潜在的IP地址有序列表。

**/secure**

如果 `/secure` 属性的值为“1”，则Dispatcher使用HTTPS与AEM实例进行通信。有关详细信息，另请参阅 [配置Dispatcher以使用SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl)。

**/always-resolve**

使用Dispatcher **4.1.6，**您可以如下配置属性 `/always-resolve` ：

* 设置为“1”时，它将解析每个请求的主机名(调度程序永远不会缓存任何IP地址)。由于获得每个请求的主机信息所需的额外呼叫，可能会造成轻微的性能影响。
* 如果未设置该属性，则默认情况下将缓存IP地址。

此外，可以使用此属性以应对动态IP分辨率问题，如以下示例所示：

```xml
/rend {
  /0001 {
     /hostname "host-name-here"
     /port "4502"
     /ipv4 "1"
     /always-resolve "1"
     }
  }
```

## 配置对内容的访问 {#configuring-access-to-content-filter}

使用 `/filter` 此部分指定Dispatcher接受的HTTP请求。所有其他请求都将返回到带有404错误代码的Web服务器(找不到页面)。如果不 `/filter` 存在任何部分，则接受所有请求。

**注意：** 始终拒绝 [对statfile](dispatcher-configuration.md#main-pars-title-28) 的请求。

>[!CAUTION]
>
>使用Dispatcher限制访问时，请参阅 [调度程序安全核对清单](security-checklist.md) 以了解更多注意事项。另外，请阅读 [AEM安全选项列表](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html) ，了解有关AEM安装的其他安全详细信息。

/filter部分由拒绝或允许根据HTTP请求的请求行部分中的模式访问内容的一系列规则组成。您应该为/filter部分使用一个有趣的策略：

* 首先，拒绝访问所有内容。
* 允许根据需要访问内容。

### 定义过滤器 {#defining-a-filter}

`/filter` 部分中的每个项目都包含一个类型和一个模式，该模式与请求行或整个请求行的特定元素相匹配。每个过滤器都可以包含以下项目：

* **类型**：它 `/type` 指示是否允许或拒绝对与模式匹配的请求的访问。值可以为 `allow` 或 `deny`.

* **请求行的元素：** 包括 `/method``/url`、或 `/query``/protocol` 一个模式，用于根据HTTP请求的请求线部分的特定部分过滤请求。过滤请求行(而不是整个请求行上的元素)的元素是首选过滤器方法。

* **请求行的高级元素：** 从调度程序4.2.0开始，可使用四个新的过滤元素。这些新元素分别为 `/path`、 `/selectors`和 `/extension``/suffix` 。包括一个或多个项目以进一步控制URL模式。

>[!NOTE]
>
>有关这些元素引用的请求行部分的详细信息，请参阅 [sling URL分解](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki页面。

* **glob属性**： `/glob` 此属性用于与HTTP请求的整个请求行匹配。

>[!CAUTION]
>
>使用globs过滤已在Dispatcher中弃用。因此，您应避免在 `/filter` 部分中使用全球化，因为它可能会导致安全问题。因此，不是：

`/glob "* *.css *"`

您应使用

`/url "*.css"`

#### HTTP请求的请求线部分 {#the-request-line-part-of-http-requests}

HTTP/1.1定义 [了请求线](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) ，如下所示：

*方法请求-URI HTTP-Version*&lt; CRLF&gt;

&lt; CRLF&gt;字符返回回车返回后跟换行源。以下示例是当客户端请求Geometrixx-Othors站点的en页面时接收的请求线：

GET/content/geometrixx-outdoors/en.htmlHTTP.1&lt; CRLF&gt;

您的模式必须考虑请求行和&lt; CRLF&gt;字符中的空格字符。

#### 双引号与单引号 {#double-quotes-vs-single-quotes}

创建过滤器规则时，对简单图案使用 `"pattern"` 双引号。如果您使用Dispatcher4.2.0或更高版本并且模式包含正则表达式，则必须在单引号中包含正则表达式模式 `'(pattern1|pattern2)'` 。

#### 正则表达式 {#regular-expressions}

调度程序4.2.0之后，您可以在筛选模式中包含POSIX Extended正则表达式。

#### 过滤器疑难解答 {#troubleshooting-filters}

如果过滤器未按您预期的方式触发，请启用 [跟踪在调度程序](#trace-logging) 上的跟踪日志记录，这样您可以看到哪个过滤器正在拦截请求。

#### 示例过滤器：拒绝全部 {#example-filter-deny-all}

以下示例过滤器部分将导致Dispatcher拒绝所有文件的请求。您应拒绝访问所有文件，然后允许访问特定区域。

```xml
  /0001  { /glob "*" /type "deny" }
```

请求显式拒绝区域导致返回404错误代码(找不到页面)。

#### 示例过滤器：拒绝特定区域的使用 {#example-filter-deny-acess-to-specific-areas}

过滤器还允许您拒绝访问各种元素(例如ASP页面和发布实例中的敏感区域)。以下过滤器拒绝访问ASP页面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 示例过滤器：启用POST请求 {#example-filter-enable-post-requests}

以下示例过滤器允许POST方法提交表单数据：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 示例过滤器：允许访问工作流控制台 {#example-filter-allow-access-to-the-workflow-console}

以下示例显示了一个过滤器，用于拒绝对工作流控制台的外部访问：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

如果发布实例使用Web应用程序上下文(例如发布)，则还可以将此内容添加到过滤器定义中。

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

如果仍需要在受限区域内访问单个页面，则可以允许访问这些页面。例如，要允许访问“工作流”控制台中的“存档”选项卡，请添加以下部分：

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>当应用于请求的多个过滤器模式时，应用的最后一个过滤模式有效。

#### 示例过滤器：使用正则表达式 {#example-filter-using-regular-expressions}

此过滤器使用正则表达式在非公开内容目录中启用扩展，在单引号之间定义此过滤器：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 示例过滤器：过滤请求URL的其他元素 {#example-filter-filter-additional-elements-of-a-request-url}

下面是一个规则示例，它使用路径、选择器和扩展过滤器阻止内容从 `/content` 路径及其子树抓取：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### 示例/filter部分 {#example-filter-section}

配置Dispatcher时，您应尽可能限制外部访问。以下示例为外部访客提供了最小的访问权限：

* `/content`
* 各种内容，如设计和客户端库；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

创建过滤器后 [，测试页面访问](dispatcher-configuration.md#main-pars-title-19) 权限以确保AEM实例安全。

调度程序的以下/filter部分。任何文件都可用作 [Dispatcher配置](dispatcher-configuration.md) 文件中的基础。

此示例基于随Dispatcher一起提供的默认配置文件，它用作在生产环境中使用的示例。使用#前缀的项目已停用(注释掉)，如果您决定激活其中的任何内容(通过删除该行上的#)，则应谨慎，因为这可能具有安全性影响。

您应拒绝访问所有内容，然后允许访问特定(受限)元素：

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
>与Apache一起使用时，根据调度程序模块的dispatcherUseProcesseDurl属性设计过滤器URL模式。(请参阅 [Apache Web Server-配置Apache Web Server for Dispatcher](dispatcher-install.md#main-pars-55-35-1022)。)

>[!NOTE]
>
>关于Dynamic Media的过滤器0030和0031适用于AEM6.0及更高版本。

如果您选择扩展访问权限，请考虑以下建议：

* 如果您使用 `/admin` CQ版本5.4 *或更早版本，应始终* 完全禁用外部访问权限。

* 在允许访问文件时，必须谨慎处理 `/libs`。可以单独访问。
* 拒绝访问复制配置，因此无法看到该配置：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒绝访问Google Gadgets反向代理：

   * `/libs/opensocial/proxy*`

根据您的安装，可能会在必须提供的其他资源下 `/libs`或 `/apps` 其他位置提供其他资源。您可以将 `access.log` 文件用作确定正在外部访问的资源的一种方法。

>[!CAUTION]
>
>访问控制台和目录可能会对生产环境带来安全风险。除非您有明确的选择，否则应取消激活(注释掉)。

>[!CAUTION]
>
>如果在发布环境 [](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/reporting.html#UsingReportsinaPublishEnvironment) 中使用报告，则应配置Dispatcher以拒绝外部 `/etc/reports` 访客访问。

### 限制查询字符串 {#restricting-query-strings}

从Dispatcher版本4.1.5开始，使用该 `/filter` 部分限制查询字符串。强烈建议明确允许查询字符串并通过 `allow` 过滤器元素排除通用津贴。

单个条目可以具有 *glob或**组合方法*、*url*、*查询* 和 *版本，* 但不能兼有二者。以下示例允许 `a=*` 查询字符串，并为解析到该 `/etc` 节点的URL拒绝所有其他查询字符串：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果规则包含一个 `/query`，它将仅匹配包含查询字符串并与提供的查询模式匹配的请求。
>
>在以上示例中， `/etc` 如果请求没有查询字符串，则需要遵循以下规则：


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 测试调度程序安全性 {#testing-dispatcher-security}

调度程序过滤器应阻止访问AEM发布实例上的以下页面和脚本。使用Web浏览器尝试像站点访客一样打开以下页面，并确认是否返回代码404。如果获得任何其他结果，请调整滤镜。

请注意，您应该看到/content/add_valid_page.html的正常页面渲染吗？debug=布局。


* /admin
* /system/console
* /dav/crx.default
* /crx
* /bin/crxde/logs
* /jcr：system/jcr：versionStorage. json
* /_jcr_system/_jcr_versionStorage.json
* /libs/wcm/core/content/siteadmin.html
* /libs/collab/core/content/admin.html
* /libs/cq/ui/content/dumplibs.html
* /var/linkchecker.html
* /etc/linkchecker.html
* /home/users/a/admin/profile.json
* /home/users/a/admin/profile.xml
* /libs/cq/core/content/login.json
* /content/../libs/foundation/components/text/text.jsp
* /content/.{.}/libs/foundation/components/text/text. jsp
* /apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata
* /libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet
* /content.pages.json
* /content.languages.json
* /content.blueprint.json
* /content.-1. json
* /content.10.json
* /content.infinity.json
* /content.tidy.json
* /content.tidy。-blubber. json
* /content/dam.tidy。-100. json
* /content/content/geometrixx.sitemap.txt
* /content/add_valid_page.query.json？statement=//*
* /content/add_valid_page.qu%65ry. js6Fn？statement=//*
* /content/add_valid_page.query.json？statement=//*[@ transfictPassword]/(@ translatPassword%20||%Por%20||%Por)
* /content/add_valid_path_to_a_page/_jcr_content.json
* /content/add_valid_path_to_a_page/jcr：content. json
* /content/add_valid_path_to_a_page/_jcr_content.feed
* /content/add_valid_path_to_a_page/jcr：content. feed
* /content/add_valid_path_to_a_page/pagename。_ jcr_ content. feed
* /content/add_valid_path_to_a_page/pagename.jcr：content. feed
* /content/add_valid_path_to_a_page/pagename.docview.xml
* /content/add_valid_path_to_a_page/pagename.docview.json
* /content/add_valid_path_to_a_page/pagename.sysview.xml
* /etc.xml
* /content.feed.xml
* /content.rss.xml
* /content.feed.html
* /content/add_valid_page.html？debug= layout
* /projects
* /tagging
* /etc/replication.html
* /etc/cloudservices.html
* /欢迎

在终端或命令提示符中发出以下命令以确定是否启用匿名写入访问。您不能将数据写入节点。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

在终端或命令提示符中发出以下命令以尝试使调度程序缓存失效，并确保您重新发出代码404响应：

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## 允许访问虚URL {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The com.adobe.granite.dispatcher.vanityurl.content package needs to be made public before publishing this contnet.</p>

 -->

配置Dispatcher以允许访问为CQ或AEM页面配置的虚URL。

启用虚URL访问时，调度程序定期调用渲染实例上运行的服务以获取虚URL列表。调度程序将此列表存储在本地文件中。`/filter` 当由于章节中的过滤器而拒绝页面请求被拒绝时，Dispatcher会查阅虚URL列表。如果列表中有被拒绝的URL，Dispatcher允许访问虚URL。

要启用虚URL访问，请向章节中添加一个 `/vanity_urls``/farms` 部分，类似于以下示例：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls` 该部分包含以下属性：

* `/url`：在渲染实例上运行的虚URL服务的路径。此属性的值 `"/libs/granite/dispatcher/content/vanityUrls.html"`必须为。

* `/file`：Dispatcher存储虚URL列表的本地文件的路径。确保Dispatcher具有对此文件的写入权限。
* `/delay`：(秒)调用虚URL服务之间的时间。

>[!NOTE]
>
>如果渲染是AEM的实例，则必须安装 [VanityURL-Components](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq600/component/vanityurls-components) 包以安装虚URL服务。(请参阅 [登录到包共享](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/package-manager.html#SigningIntoPackageShare)。)

使用以下过程可访问虚URL。

1. 如果渲染服务是AEM实例，请在发布实例上安装com. adobe. granite. dispatcher. vanityurl. content包(请参阅上面的备注)。
1. 对于您为AEM或CQ页面配置的每个虚URL，请确保配置 ` [/filter](dispatcher-configuration.md#main-pars_134_32_0009)` 拒绝URL。如有必要，请添加一个拒绝URL的过滤器。
1. 添加以下 `/vanity_urls` 部分 `/farms`。
1. 重新启动Apache Web服务器。

## 转发申请请求-/propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

聚合请求通常仅适用于调度程序，因此默认情况下，它们不会发送到呈示器(例如AEM实例)。

如有必要，请将“/propagateSyndPost”属性设置为“1”以转发到调度程序的请求请求。如果设置了此设置，则必须确保在筛选器部分中不拒绝POST请求。

## 配置调度程序缓存-/cache {#configuring-the-dispatcher-cache-cache}

`/cache` 此部分控制Dispatcher如何缓存文档。配置多个子属性以实施您的缓存策略：


* /docroot
* /statfile
* /serveStaleOnError
* /allowAuthorized
* /规则
* /statfileslevel
* /invalidate
* /invalidateHandler
* /allowedClients
* /ignoreUrlParams
* /headers
* /mode
* /gracePeriod


示例缓存部分可能如下所示：

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
>对于权限敏感型缓存，请阅读 [缓存安全内容](permissions-cache.md)。

### 指定缓存目录 {#specifying-the-cache-directory}

`/docroot` 该属性标识缓存缓存文件的目录。

>[!NOTE]
>
>该值必须与Web服务器的文档根完全相同，以便调度程序和Web服务器处理相同的文件。\
>Web服务器负责在使用调度程序缓存文件时提供正确的状态代码，这也说明它也很重要。

如果您使用多个农场，每个农场必须使用不同的文档根。

### 命名Statfile {#naming-the-statfile}

`/statfile` 该属性标识要用作statfile的文件。Dispatcher使用此文件注册最新内容更新的时间。statfile可以是Web服务器上的任何文件。

statfile没有内容。更新内容时，Dispatcher会更新时间戳。默认的stalfile名为. stat并存储在docroot中。调度程序阻止对statfile的访问。

>[!NOTE]
>
>如果配置了此属性， `/statfileslevel` Dispatcher将忽略 `/statfile` 属性并使用. stat作为名称。

### 发生错误时提供陈旧的文档 {#serving-stale-documents-when-errors-occur}

`/serveStaleOnError` 该属性控制当渲染服务器返回错误时，Dispatcher是否返回失效的文档。默认情况下，在触摸缓存的内容并使缓存的内容失效时，调度程序在下次请求缓存的内容时删除缓存内容。

如果 `/serveStaleOnError` 设置为“1”，则Dispatcher不会从缓存删除失效内容，除非渲染服务器返回成功响应。AEM或连接超时中的xx响应导致Dispatcher服务于过期的内容并响应，而HTTP状态为111(重新验证失败)。

### 使用身份验证时缓存 {#caching-when-authentication-is-used}

`/allowAuthorized` 该属性控制是否缓存包含下列任何身份验证信息的请求：

* `authorization` 标题。
* 名为 `authorization`cookie的cookie。
* 名为 `login-token`cookie的cookie。

默认情况下，不会缓存包含此身份验证信息的请求，因为在将缓存的文档返回给客户端时，不会执行身份验证。此配置可防止Dispatcher向没有必要权限的用户提供缓存文档。

但是，如果您的要求允许缓存已验证的文档，请将其设置为一个：

`/allowAuthorized "1"`

>[!NOTE]
>
>要启用会话管理(使用 `/sessionmanagement` 属性)，必须将 `/allowAuthorized` 该属性设置 `"0"`为。

### 将文档指定为高速缓存 {#specifying-the-documents-to-cache}

`/rules` 该属性控制根据文档路径缓存哪些文档。无论/rules属性如何，调度程序在以下情况下都不会缓存文档：

* 如果请求URI包含问号(“？.\
   这通常表示动态页面，如不需要缓存的搜索结果。
* 缺少文件扩展名。\
   Web服务器需要该扩展来确定文档类型(MIME类型)。
* 设置了身份验证头(可以配置此功能)
* 如果AEM实例响应以下标题：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>GET或HEAD(对于HTTP头)方法由Dispatcher进行缓存。有关响应头缓存的其他信息，请参阅 [“缓存HTTP响应头](dispatcher-configuration.md#caching-http-response-headers) ”部分。

/rules属性中的每个项目都包括一个 [glob](#designing-patterns-for-glob-properties) 图案和一个类型：

* glob模式用于匹配文档的路径。
* 类型指示是否缓存与glob模式匹配的文档。该值可以是允许的(用于缓存文档)或拒绝(以始终呈现文档)。

如果没有动态页面(超出以上规则已排除的页面除外)，则可以配置Dispatcher以缓存所有内容。此处的规则部分如下所示：

```xml
/rules
  { 
    /0000  {  /glob "*"   /type "allow" }
  }
```

有关glob属性的信息，请参阅 [为glob属性设计图案](#designing-patterns-for-glob-properties)。

如果您的页面的某些部分是动态的(例如新闻应用程序)或在封闭的用户组内，则可以定义例外情况：

>[!NOTE]
>
>不能缓存已关闭的用户组，因为未检查缓存页面的用户权限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }   
  }
```

**压缩**

在Apache Web服务器上，您可以压缩缓存的文档。压缩允许Apache在客户端请求时以压缩形式返回文档。通过启用Apache模块，可以自动完成压缩 `mod_deflate`，例如：

```xml
AddOutputFilterByType DEFLATE text/plain
```

默认情况下，该模块与Apache2.x一起安装。

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

### 按文件夹级别使文件失效 {#invalidating-files-by-folder-level}

使用属性 `/statfileslevel` 根据其路径使缓存的文件失效：

* Dispatcher在每个文件夹中创建 `.stat`文件，从docroot文件夹到您指定的级别。docroot文件夹为级别0。
* 文件通过触控 `.stat` 文件无效。`.stat` 将文件的上次修改日期与缓存文档的上次修改日期进行比较。`.stat` 如果文件较新，则重新访存文档。

* 当位于某个级别的文件失效时 **，将**`.stat` 会影响从docroot **到** valid文件级别或配置 `statsfilevel` (whicherver较小)的所有文件。

   * 例如：如果将该 `statfileslevel` 属性设置为6，且文件在级别无效，则将触摸docroot至5的每 `.stat` 个文件。继续执行此示例，如果文件在级别为7，则为无效。`stat` 从docroot到6的文件(因为 `/statfileslevel = "6"`)。

只有资源**沿着路径**才会影响无效文件。请考虑以下示例：网站使用该结构 `/content/myWebsite/xx/.` 如果您设置 `statfileslevel` 为3，则会创建 `.stat`一个文件，如下所示：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

When a file in `/content/myWebsite/xx` is invalidated then every `.stat` file from docroot down to `/content/myWebsite/xx`is touched. 这只适用于 `/content/myWebsite/xx` 示例，不适用于示例 `/content/myWebsite/yy` 或 `/content/anotherWebSite`。

>[!NOTE]
>
>可通过发送其他Header来防止失效 `CQ-Action-Scope:ResourceOnly`。这可用于刷新特定资源，而不会使缓存的其他部分失效。有关更多详细信息，请参阅 [此页面](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) 并 [手动使调度程序缓存](https://helpx.adobe.com/experience-manager/dispatcher/using/page-invalidate.html) 失效。

>[!NOTE]
>
>如果为 `/statfileslevel` 该属性指定值，则会忽略该 `/statfile` 属性。

### 自动使缓存的文件失效 {#automatically-invalidating-cached-files}

`/invalidate` 此属性定义更新内容时自动失效的文档。

通过自动失效，Dispatcher不会在内容更新后删除缓存的文件，但在下次请求时检查其有效性。缓存中未自动失效的文档将保留在缓存中，直到内容更新明确删除它们为止。

自动失效通常用于HTML页面。HTML页面通常包含指向其他页面的链接，因而很难确定内容更新是否会影响页面。要确保在更新内容时所有相关页面都失效，自动使所有HTML页面失效。以下配置将使所有HTML页面失效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

有关glob属性的信息，请参阅 [为glob属性设计图案](#designing-patterns-for-glob-properties)。

激活/content/geometrixx/en后，此配置会产生以下活动：

* 所有带图案的文件。*从/content/geometrixx/文件夹中删除。
* 此时将删除/content/geometrixx/en/_jcr_content文件夹。
* 所有与/invalidate配置匹配的其他文件不会立即删除。当下一个请求发生时，将删除这些文件。在我们的示例/content/geometrixx.html中不删除，请求/content/geometrixx.html时将会删除它。

如果您提供自动生成的PDF和ZIP文件供下载，您可能也必须自动使这些文件失效。配置示例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM与Adobe Analytics集成在您网站的分析. siteCatalyst. js文件中提供配置数据。示例调度程序。随Dispatcher一起提供的任何文件都包括以下文件的无效规则：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自定义无效脚本 {#using-custom-invalidation-scripts}

使用Details属性可定义为Dispatcher接收的每个无效请求调用的脚本。

使用以下参数调用它：

* 处理\
   无效的内容路径
* 操作\
   复制操作(例如激活、取消激活)
* 操作范围\
   复制操作的范围(空，除非发送的标题 `CQ-Action-Scope: ResourceOnly` 被发送，请参阅从AEM [](page-invalidate.md) 中将缓存的页面失效以了解详细信息)

这可用于覆盖许多不同的用例，如使其他应用程序特定缓存失效，或处理页面的外部URL和其在docroot中的外部URL与内容路径不匹配的情况。

下面的示例脚本将每个无效请求记录到一个文件。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### 示例失效处理函数脚本 {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### 限制可刷新缓存的客户端 {#limiting-the-clients-that-can-flush-the-cache}

Social属性定义允许刷新缓存的特定客户端。根据IP匹配全局模型。

以下示例：

1. 拒绝访问任何客户端
1. 显式允许访问localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

有关glob属性的信息，请参阅 [为glob属性设计图案](#designing-patterns-for-glob-properties)。

>[!CAUTION]
>
>建议您定义/allowedClients。
>
>如果未完成此操作，则任何客户端都可以发出清除缓存的调用；如果重复执行此操作，则会严重影响网站性能。

### 忽略URL参数 {#ignoring-url-parameters}

`ignoreUrlParams` 此部分定义在确定页面是否缓存或从缓存中传递页面时忽略哪些URL参数：

* 当请求URL包含所有被忽略的参数时，页面将缓存。
* 当请求URL包含一个或多个不被忽略的参数时，页面不会缓存。

当页面被忽略某个参数时，第一次请求该页面时将缓存该页面。对页面的后续请求将被缓存，而不管请求中参数的值如何。

要指定忽略的参数，请向属性 `ignoreUrlParams` 添加glob规则：

* 要忽略参数，请创建允许参数的glob属性。
* 要防止缓存页面，请创建拒绝参数的glob属性。

以下示例导致调度程序忽略“q”参数，因此包含q参数的请求URL已缓存：

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

使用示例 `ignoreUrlParams` 值，以下HTTP请求将缓存要缓存的页面，因为 `q` 参数被忽略：

```xml
GET /mypage.html?q=5
```

使用示例 `ignoreUrlParams` 值，以下HTTP请求将导致页面 **无法** 缓存，因为 `p` 不会忽略该参数：

```xml
GET /mypage.html?q=5&p=4
```

有关glob属性的信息，请参阅 [为glob属性设计图案](#designing-patterns-for-glob-properties)。

### 缓存HTTP响应标题 {#caching-http-response-headers}

>[!NOTE]
>
>此功能可与Dispatcher版本 **4.1.11** 一起使用。

`/headers` 该属性允许您定义将由Dispatcher缓存的HTTP头类型。在对未缓存资源的第一个请求中，与某个配置的值匹配的所有标题(请参阅下面的配置示例)存储在一个单独的文件中，该文件位于缓存文件旁边。在随后对缓存资源的请求中，存储的标题会添加到响应中。

下面是默认配置的示例：

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
>另外，请注意，不允许文件全局字符。有关更多详细信息，请参阅 [为glob属性设计图案](#designing-patterns-for-glob-properties)。

>[!NOTE]
>
>如果您需要Dispatcher来存储和提供AEM中的eTag响应标题，请执行以下操作：
>
>* 在 `/cache/headers`章节中添加标题名称。
>* 在Dispatcher相关部分中添加以下 [Apache指令](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) ：
>



```xml
FileETag none
```

### 调度程序缓存文件权限 {#dispatcher-cache-file-permissions}

`mode` 该属性指定了哪些文件权限应用于缓存中的新目录和文件。此设置受调用过程的 `umask` 限制。它是一个由以下一个或多个值的总和构成的八进制数字：

* 0400允许所有者阅读。
* 0200允许所有者编写。
* 0100允许所有者在目录中搜索。
* 0040允许用户组成员阅读。
* 0020允许按组成员写入。
* 0010允许用户组成员在目录中搜索。
* 04允许他人阅读。
* 0002允许他人编写。
* 001允许其他人在目录中搜索。

默认值为0755，允许所有者读取、写入或搜索，以及用户组和其他人进行读取或搜索。

### Throttling. stat文件触屏 {#throttling-stat-file-touching}

通过默认 `/invalidate` 属性，每次激活都会有效地使 `.html` 所有文件失效(当其路径与 `/invalidate` 章节相匹配时)。在具有大量流量的网站上，后续激活将增加后端的CPU负载。在这种情况下，需要“trowtle `.stat` ”文件触摸以保持网站响应。您可以使用 `/gracePeriod` 属性来执行此操作。

`/gracePeriod` 此属性定义了陈旧的自动失效资源的秒数，在最后一个发生激活后，该资源仍可能从缓存提供。该属性可在设置中使用，在此设置中，成批激活会以其他方式使整个缓存失效。建议的值为秒。

有关其他详细信息，请参阅上面的 `/invalidate` 和 `/statfileslevel`章节。

## 配置基于时间的缓存失效-/enableTTL {#configuring-time-based-cache-invalidation-enablettl}

如果设置此属性， `enableTTL` 属性将从后端评估响应头，如果它们包含 `Cache-Control` 最多年龄或 `Expires` 日期，则创建缓存文件旁边的一个空文件，修改时间等于到期日期。当缓存缓存的文件时，会在修改时自动请求该缓存的文件。

您可以通过向 `dispatcher.any` 文件添加此行来启用该功能：

```xml
/enableTTL "1"
```

>[!NOTE]
>
>此功能可与Dispatcher版本 **4.1.11** 一起使用。

## 配置负载平衡-/statistics {#configuring-load-balancing-statistics}

`/statistics` 该部分定义了Dispatcher为每个渲染的响应速度提供的文件类别。Dispatcher使用分数确定要发送请求的渲染。

您创建的每个类别都定义了一个glob图案。调度程序将请求内容的URI与这些模式进行比较，以确定所请求内容的类别：

* 类别的顺序决定了与URI相比的顺序。
* 与URI匹配的第一个类别模式是文件的类别。不会评估其他类别模式。

Dispatcher最多支持个统计类别。如果定义超过个类别，则仅使用前八个类别。

**渲染选择**

每次调度程序需要渲染的页面时，它都使用以下算法来选择渲染：

1. 如果请求中包含 `renderid` cookie中的渲染名称，Dispatcher将使用该渲染。
1. 如果请求不包括 `renderid` cookie，Dispatcher将比较渲染统计数据：

   1. 调度程序确定请求URI的冒号。
   1. 调度程序确定哪个渲染具有该类别的最低响应得分，并选择该渲染。

1. 如果尚未选择渲染，请使用列表中的第一个渲染。

渲染的类别得分基于之前的响应时间以及调度程序尝试的先前失败和成功的连接。对于每个尝试，将更新所请求URI类别的分数。

>[!NOTE]
>
>如果您不使用负载平衡，则可以忽略此部分。

### 定义统计数据类别 {#defining-statistics-categories}

为要为渲染选择保留统计数据的每种类型的文档定义一个类别。/statistics部分包含一个/categories部分。要定义某个类别，请在/categories部分下面添加一行，其格式如下：

`/name { /glob "pattern"}`

该类别 `name` 必须是农场特有的。`pattern`[designing属性](#designing-patterns-for-glob-properties) 部分的设计模式中介绍了这些功能。

要确定URI的类别，Dispatcher将URI与每个类别模式进行比较，直到找到匹配项。调度程序以列表和目录中的第一个类别开头。因此，请先将类别包含更多特定模式。

例如，调度程序默认调度程序。任何文件都定义HTML类别和其他类别。HTML类别更具体，因此它首先出现：

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

### 在Dispatcher统计数据中反映服务器不可用 {#reflecting-server-unavailability-in-dispatcher-statistics}

`/unavailablePenalty` 属性设置当连接到渲染失败时应用于渲染统计数据的时间(以秒为单位)。调度程序将时间添加到与所请求的URI匹配的统计数据类别。

例如，当无法建立指向指定主机名/端口的TCP/IP连接时，将应用罚款，这是因为AEM没有运行(并且不会侦听)或由于网络相关问题而导致。

`/unavailablePenalty` 该属性是章节 `/farm` 的直接子项( `/statistics` 章节的兄弟姐妹)。

如果不 `/unavailablePenalty` 存在任何属性，则使用“1”值。

```xml
/unavailablePenalty "1"
```

## 识别粘性连接文件夹-/stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

`/stickyConnectionsFor` 该属性定义一个包含粘性文档的文件夹；这将使用URL进行访问。调度程序将所有请求从一个用户发送到同一个渲染实例中。粘性连接确保了会话数据对所有文档都具有一致性和一致性。此机制使用 `renderid` cookie。

以下示例定义了与/products文件夹的粘性连接：

```xml
/stickyConnectionsFor "/products"
```

当页面由多个内容节点中的内容组成时，包括列出内容路径的 `/paths` 属性。例如，页面包含来自 `/content/image`、 `/content/video`和的内容 `/var/files/pdfs`。以下配置可为页面上的所有内容启用粘性连接：

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### HTTPOnly {#httponly}

启用黏性连接后，调度程序模块将设置 `renderid` cookie。此cookie没有 `httponly` 标记，应添加此标志以增强安全性。您可以通过在配置文件 `httpOnly` 的 `/stickyConnections` 节点中设置属性来 `dispatcher.any` 执行此操作。属性的值(或1)定义 `renderid` Cookie是否附加 `HttpOnly` 了属性。默认值为0，这表示不会添加属性。

有关 `httponly` 标记的其他信息，请阅读 [此页面](https://www.owasp.org/index.php/HttpOnly)。

### secure {#secure}

启用黏性连接后，调度程序模块将设置 `renderid` cookie。此cookie没有 **安全** 标志，应添加此标志以增强安全性。您可以通过在配置文件 `secure` 的 `/stickyConnections` 节点中设置属性来 `dispatcher.any` 执行此操作。属性的值(或1)定义 `renderid` Cookie是否附加 `secure` 了属性。默认值为0，这意味着将添加属性**传入请求是安全的。如果该值设置为1，则无论传入请求是否安全，都将添加安全标志。

## 处理渲染连接错误 {#handling-render-connection-errors}

在渲染服务器返回500错误或不可用时配置调度程序行为。

### 指定健康检查页面 {#specifying-a-health-check-page}

使用 `/health_check` 该属性指定在发生500个状态代码时选中的URL。如果此页面还返回500状态代码，则实例被认为不可用，并且可配置的时间罚款( `/unavailablePenalty`)会在重试之前应用于渲染。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定页面重试延迟 {#specifying-the-page-retry-delay}

/ `retryDelay` 属性设置调度程序在与农场渲染的循环尝试之间等待的时间(以秒为单位)。对于每个圆形，调度程序尝试连接渲染的最大次数是在农场中渲染的次数。

如果未显式定义Dispatcher `"1"` ， `/retryDelay` 则调度程序使用值。大多数情况下，默认值是适当的。

```xml
/retryDelay "1"
```

### 配置重试次数 {#configuring-the-number-of-retries}

`/numberOfRetries` 该属性设置Dispatcher对渲染的最大连接尝试次数。如果调度程序在此重试次数后无法成功连接到渲染，调度程序将返回失败的响应。

对于每个圆形，调度程序尝试连接渲染的最大次数是在农场中渲染的次数。因此，调度程序尝试连接的最大次数为( `/numberOfRetries`) x(渲染次数)。

如果未显式定义该值，则默认值为 **5**。

```xml
/numberOfRetries "5"
```

### 使用故障转移机制 {#using-the-failover-mechanism}

在Dispatcher农场上启用故障转移机制，以便在原始请求失败时重新发送对不同渲染的请求。启用故障转移后，调度程序有以下行为：

* 当对渲染的请求返回HTTP状态503(不可用)时，调度程序将请求发送到其他渲染。
* 当对渲染的请求返回HTTP状态50x(非503)时，调度程序会发送对该 `health_check` 属性配置的页面的请求。

   * 如果健康检查返回500(INTERNAL_ SERVER_ ERROR)，Dispatcher将发送原始请求到不同的渲染。
   * 如果修复检查返回HTTP状态200，则调度程序将初始HTTP500错误返回给客户端。

要启用故障转移，请将以下行添加到农场(或网站)：

```xml
/failover "1" 
```

>[!NOTE]
>
>要重试包含正文的HTTP请求，Dispatcher会在后台提取实际内容之前向渲染发送 `Expect: 100-continue` 请求标头。CQ5.5包含CQSE，然后立即回答100(继续)或错误代码。其他servlet容器也应支持此功能。

## 忽略中断错误-/ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此选项。当您看到以下日志消息时，您只需要使用它：
>
>`Error while reading response: Interrupted system call`

如果系统调用的对象位于通过NFS访问的远程系统上，则任何面向文件系统的调用都可能会中断 `EINTR` 。这些系统调用是否可以超时或中断取决于底层文件系统在本地机器上的安装方式。

如果您的实例具有此类配置，并且日志包含以下消息，请使用social参数：

`Error while reading response: Interrupted system call`

内部，调度程序使用可表示为：的循环从远程服务器(即AEM)读取响应：

`while (response not finished) {  
read more data  
}`

此类消息可在部分 `EINTR` 发生时生成 `read more data`，是由接收任何数据之前的信号接收而生成。

要忽略此类中断，您可以将以下参数添加到 `dispatcher.any` (之前 `/farms`)：

`/ignoreEINTR "1"`

设置 `/ignoreEINTR` 为 `"1"` 使Dispatcher继续尝试读取数据，直到读取完整的响应为止。默认值为0，取消激活该选项。

## 为glob属性设计模式 {#designing-patterns-for-glob-properties}

Dispatcher配置文件中的几个部分使用 `glob` 属性作为客户端请求的选择条件。glob属性的值是调度程序与请求的一个方面相比较的模式，如请求的资源的路径或客户端的IP地址。例如 `/filter` ，部分中的项目使用glob模式识别Dispatcher对或拒绝的页面的路径。

glob值可包括通配符字符和字母数字字符，用于定义模式。

| 通配符字符 | 描述 | 示例 |
|--- |--- |--- |
| `*` | 在字符串中匹配任意字符的零个或多个相邻实例。匹配的最终字符由以下任一情况决定： <br/>字符串中的字符与图案中的下一个字符匹配，图案字符具有以下特性：<br/><ul><li>不是*</li><li>不是？</li><li>文本字符(包括空格)或字符类。</li><li>将到达图案的结尾。</li></ul>在字符类中，字符是解释的。 | `*/geo*` 匹配 `/content/geometrixx` 节点和 `/content/geometrixx-outdoors` 节点下的任何页面。以下HTTP请求与glob模式相匹配： <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*`<br/>匹配 `/content/geometrixx-outdoors` 节点下的任何页面。例如，以下HTTP请求与glob模式相匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 匹配任意单个字符。使用外部字符类。在字符类中，此字符经过解释。 | `*outdoors/??/*`<br/> 在geometrixx Outdoors站点中匹配任何语言的页面。例如，以下HTTP请求与glob模式相匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>以下请求与glob模式不匹配： <br/><ul><li>“GET/content/geometrixx-outdoors/en.html”</li></ul> |
| `[ and ]` | 标记字符类的开始和结尾。字符类可以包括一个或多个字符范围和单个字符。<br/>如果目标字符与字符类中的任何字符匹配或在定义的范围内，则会发生匹配。<br/>如果未包含结束括号，则该模式不会产生匹配项。 | `*[o]men.html*`<br/> 匹配以下HTTP请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*`<br/>匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示一系列字符。用于字符类。在字符类之外，此字符是解释的。 | `*[m-p]men.html*` 匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定后面的字符类或字符类。仅用于否定字符类内的字符和字符范围。等效 `^ wildcard`于。<br/>在字符类之外，此字符是解释的。 | `*[!o]men.html*`<br/> 匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>与以下HTTP请求不匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定后面的字符或字符范围。用于仅对字符类内的字符和字符范围进行否定。等效于 `!` 通配符字符。<br/>在字符类之外，此字符是解释的。 | `!` 通配符字符示例应用，用字符替换示例模式中的 `!``^` 字符。 |


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

## 日志记录 {#logging}

在Web服务器配置中，您可以设置：

* 调度程序日志文件的位置。
* 日志级别。

有关详细信息，请参阅Web服务器文档和Dispatcher实例的自述文件。

**Apache旋转/Pied Logs**

如果使用 **Apache** Web服务器，则可以对旋转和/或管道日志使用标准功能。例如，使用管道日志：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

这将自动旋转：

* 调度程序日志文件；在扩展中具有时间戳(logs/dispatcher. log%Y%m%d)。
* (60x60x24x7=604800秒)。

请参阅有关Log Ration和Pied Logs的Apache Web服务器文档；例如 [Apache2.2](https://httpd.apache.org/docs/2.2/logs.html)。

>[!NOTE]
>
>安装时，默认日志级别较高(即级别3=调试)，以便Dispatcher记录所有错误和警告。这在初始阶段非常有用。
>
>但是，这需要额外的资源，因此当Dispatcher根据您的要求顺畅 *运行时*，您可以(应)降低日志级别。

### 跟踪日志记录 {#trace-logging}

在Dispatcher的其他增强功能中，版本4.2.0还引入了跟踪日志记录。

这比调试日志记录更高，在日志中显示其他信息。它添加了以下记录：

* 转发的标题的值；
* 应用于特定操作的规则。

您可以通过在Web服务器 `4` 中设置日志级别来启用跟踪日志记录。

下面是启用跟踪的日志示例：

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

同时请求了与阻止规则匹配的文件时记录的事件：

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## 确认基本操作 {#confirming-basic-operation}

要确认Web服务器、调度程序和AEM实例的基本操作和交互，您可以使用以下步骤：

1. 设置 `loglevel` 为 `3`。

1. 启动Web服务器；这也会启动Dispatcher。
1. 启动AEM实例。
1. 检查Web服务器和Dispatcher的日志和错误文件。\
   根据Web服务器的不同，您应当看到以下消息：\
   `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`\
   和:\
   `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 通过Web服务器冲浪网站。确认根据需要显示内容。\
   例如，在本地安装上，AEM在访问网站控制台上的端口 `4502` 和Web服务器上同时使用两者进行 `80` 访问：\
   ` https://localhost:4502/libs/wcm/core/content/siteadmin.html  
https://localhost:80/libs/wcm/core/content/siteadmin.html  
`结果应相同。使用相同的机制确认对其他页面的访问权限。

1. 检查缓存目录是否已填写。
1. 激活页面以检查是否已正确清除缓存。
1. 如果一切都能正常运行，您可以减少 `loglevel``0`工作量。

## 使用多个调度程序 {#using-multiple-dispatchers}

在复杂的设置中，您可以使用多个调度程序。例如，您可以使用：

* 一个Dispatcher，用于在Intranet上发布网站
* 在不同地址下和具有不同安全性设置的第二个调度程序下发布Internet上的相同内容。

在这种情况下，请确保每个请求只通过一个调度程序。Dispatcher不处理来自其他Dispatcher的请求。因此，请确保两个Dispatcher直接访问AEM网站。

## 调试 {#debugging}

将标题 `X-Dispatcher-Info` 添加到请求时，调度程序回答目标是缓存、返回还是根本无法缓存。响应头 `X-Cache-Info` 包含可读表单中的此信息。您可以使用这些响应标题调试涉及Dispatcher缓存的响应的问题。

默认情况下未启用此功能，因此，要包括响应标题 `X-Cache-Info` ，农场必须包含以下条目：

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

`X-Dispatcher-Info` 此外，标题不需要值，但如果用于测试， `curl` 则必须提供值才能发送标题，如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/we-retail/us/en.html
```

下面是一个包含 `X-Dispatcher-Info` 将返回的响应标题的列表：

* **缓存**\
   目标文件包含在缓存中，调度程序已确定交付它是有效的。
* **缓存**\
   目标文件未包含在缓存中，调度程序已确定缓存输出并交付它。
* **缓存：stat file is edirectly**
the target file is contains in the cache，it is more by a new stat file.调度程序将删除目标文件，从输出中重新创建它并提供它。
* **不可缓存：没有文档根目录**该农场的配置不包含文档根目录(配置元素 `cache.docroot`)。
* **不可缓存：缓存文件路径过长**\
   目标文件-文档根和URL文件的连接超过系统上最长的可能文件名。
* **不可缓存：临时文件路径过长**\
   临时文件名模板超过系统上最长可能的文件名。调度程序首先创建一个临时文件，然后实际创建或覆盖缓存的文件。临时文件名是目标文件名，后面附加有字符 `_YYYYXXXXXX` ，并将替换 `Y` 为 `X` 创建唯一名称。
* **不可缓存：请求URL没有扩展**\
   请求URL没有扩展，或者文件扩展名之后存在路径，例如： `/test.html/a/path`。
* **不可缓存：请求不是GET或HEAD**
HTTP方法既不是GET也不是HEAD。调度程序假定输出将包含不应缓存的动态数据。
* **不可缓存：请求包含一个查询字符串**\
   请求包含一个查询字符串。调度程序假定输出取决于给定的查询字符串，因此不缓存。
* **不可缓存：会话管理器未身份验证**\
   农场缓存由会话管理器控制(配置包含 `sessionmanagement` 节点)，请求不包含相应的身份验证信息。
* **不可缓存：请求包含授权**\
   不允许农场缓存输出( `allowAuthorized 0`)，请求包含身份验证信息。
* **不可缓存：target is a directory**\
   目标文件是目录。这可能会指向一些概念错误，其中URL和某些子URL都包含可缓存输出，例如，如果请求第一次 `/test.html/a/file.ext` 出现并包含可缓存输出，调度程序将无法缓存后续请求的输出 `/test.html`。
* **不可缓存：请求URL具有尾部斜杠**\
   请求URL有一个尾部斜杠。
* **不可缓存：请求URL不在缓存规则中**\
   农场的缓存规则明确拒绝缓存某些请求URL的输出。
* **不可缓存：授权检查器拒绝访问**\
   农场的授权检查器拒绝访问缓存的文件。
* **不可缓存：会话无效**该农场的缓存由会话管理器控制(配置包含一个 `sessionmanagement` 节点)，并且用户的会话不再有效。
* **不可缓存：响应包含`no_cache `**返回的服务器返回 `Dispatcher: no_cache` 标题，禁止调度程序缓存输出。
* **不可缓存：响应内容长度为零**，响应的内容长度为零；调度程序不会创建零长度文件。
