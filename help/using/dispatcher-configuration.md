---
title: 配置Dispatcher
seo-title: 配置Dispatcher
description: 了解如何配置Dispatcher。
seo-description: 了解如何配置Dispatcher。
uuid: 253ef0f7-2491-4cec-ab22-97439df29fd6
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: '1193211344162'
topic-tags: 调度程序
content-type: 引用
discoiquuid: aeffee8e-bb34-42a7-9a5e-b7d0e848391a
translation-type: tm+mt
source-git-commit: 119f952439a59e51f769f285c79543aec8fdda37

---


# 配置Dispatcher{#configuring-dispatcher}

>[!NOTE]
>
>调度程序版本独立于AEM。 如果您遵循了指向Dispatcher文档的链接（该链接嵌入在AEM先前版本的文档中），则您可能已被重定向到此页。

以下各节介绍了如何配置Dispatcher的各个方面。

## 支持IPv4和IPv6 {#support-for-ipv-and-ipv}

AEM和Dispatcher的所有元素都可以安装在IPv4和IPv6网络中。 请参 [阅IPV4和IPV6](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html#AdditionalPlatformNotes)。

## 调度程序配置文件 {#dispatcher-configuration-files}

默认情况下，调度程序配置存储在文 `dispatcher.any` 本文件中，但您可以在安装过程中更改此文件的名称和位置。

配置文件包含一系列单值或多值属性，这些属性控制Dispatcher的行为：

* 属性名称前缀有正斜杠 `/`。
* 多值属性使用大括号将子项圈起 `{ }`。

示例配置的结构如下：

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

您可以包括其他对配置有贡献的文件：

* 如果配置文件很大，您可以将其拆分为多个较小的文件（更易于管理），然后包括这些文件。
* 包含自动生成的文件。

例如，要在/farms配置中包含文件myFarm.any，请使用以下代码：

```xml
/farms
  {
  $include "myFarm.any"
  }
```

使用星号("*")作为通配符，指定要包含的文件范围。

例如，如果文件包含 `farm_1.any` 一到五 `farm_5.any` 个农场的配置，则可以按如下方式包括这些文件：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用环境变量 {#using-environment-variables}

您可以在dispatcher.any文件中的字符串值属性中使用环境变量，而不是硬编码这些值。 要包含环境变量的值，请使用格式 `${variable_name}`。

例如，如果dispatcher.any文件与缓存目录位于同一目录中，则可以使用 [docroot](dispatcher-configuration.md#main-pars-title-30) 属性的以下值：

```xml
/docroot "${PWD}/cache"
```

另一个示例是，如果您创建了名为的环境变量，该变量 `PUBLISH_IP` 存储了AEM发布实例的主机名，则可以使用 [](dispatcher-configuration.md#main-pars-127-25-0008) /renders属性的以下配置：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 命名调度程序实例 {#naming-the-dispatcher-instance-name}

使用该 `/name` 属性指定唯一名称以标识您的Dispatcher实例。 该 `/name` 属性是配置结构中的顶级属性。

## 定义农场 {#defining-farms-farms}

该属 `/farms` 性定义一组或多组Dispatcher行为，其中每组行为都与不同的网站或URL关联。 酒店 `/farms` 可以包括一个农场或多个农场：

* 当您希望Dispatcher以相同方式处理所有网页或网站时，请使用单一农场。
* 当网站的不同区域或不同网站需要不同的调度程序行为时，创建多个农场。

该 `/farms` 属性是配置结构中的顶级属性。 要定义农场，请向该属性添加子属 `/farms` 性。 使用属性名称可唯一标识Dispatcher实例中的农场。

该属 `/farmname` 性是多值的，并且包含定义调度程序行为的其他属性：

* 农场应用的页面的URL。
* 用于渲染文档的一个或多个服务URL（通常为AEM发布实例）。
* 用于负载平衡多个文档渲染器的统计信息。
* 其他几种行为，如要缓存的文件和位置。

该值可以包含任何字母数字(a-z, 0-9)字符。 以下示例显示了两个名为和的农场的骨架定 `/daycom` 义 `/docsdaycom`:

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
>如果您使用多个渲染场，则从下而上计算列表。 这在为网站定义虚拟主 [机时尤其重](dispatcher-configuration.md#main-pars-117-15-0006) 要。

每个农场财产都可以包含以下子财产：

| 属性名称 | 描述 |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | 默认主页（可选）（仅限IIS） |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | 要传递的客户端HTTP请求的标头。 |
| [/virtualhosts](#identifying-virtual-hosts-virtual-hosts) | 此农场的虚拟主机。 |
| [/会话管理](#enabling-secure-sessions-session-management) | 支持会话管理和身份验证。 |
| [/renders](#defining-page-renderers-renders) | 提供呈现页面的服务器（通常为AEM发布实例）。 |
| [/filter](#configuring-access-to-content-filter) | 定义Dispatcher启用访问的URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 配置对虚URL的访问。 |
| [/propagateSyndPost](#forwarding-syndication-requests-propagate-syndpost) | 支持转发联合请求。 |
| [/cache](#configuring-the-dispatcher-cache-cache) | 配置缓存行为。 |
| [/statistics](#configuring-load-balancing-statistics) | 为负载平衡计算定义统计类别。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-sticky-connections-for) | 包含粘性文档的文件夹。 |
| [/health_check](#specifying-a-health-check-page) | 用于确定服务器可用性的URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重试失败的连接之前的延迟。 |
| [/unavailableDestamy](#reflecting-server-unavailability-in-dispatcher-statistics) | 影响负载平衡计算统计的惩罚。 |
| [/failover](#using-the-fail-over-mechanism) | 当原始请求失败时，向不同渲染重新发送请求。 |
| [/auth_checker](permissions-cache.md) | 有关权限敏感型缓存，请参阅 [缓存安全内容](permissions-cache.md)。 |

## 指定默认页面（仅限IIS）- /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>参 `/homepage`数（仅限IIS）不再有效。 而应使用 [IIS URL重写模块](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)。
>
>如果您使用的是Apache，则应使用该 `mod_rewrite` 模块。 有关Apache 2.4的信息，请参 `mod_rewrite` 阅Apache网站 [文档](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)。 使用时， `mod_rewrite`建议使用标志** ['passthrough|PT'（传递到下一个处理函数）](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)**，以强制重写引擎将内部结构的字段设置为字段 `uri` 的值 `request_rec``filename` 。

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

## 指定要传递的HTTP头 {#specifying-the-http-headers-to-pass-through-clientheaders}

该属 `/clientheaders` 性定义一个HTTP头列表，调度程序从客户端HTTP请求传递给渲染器（AEM实例）。

默认情况下，Dispatcher将标准HTTP头转发到AEM实例。 在某些情况下，您可能希望转发其他标题或删除特定的标题：

* 添加AEM实例在HTTP请求中需要的标题（如自定义标题）。
* 删除仅与Web服务器相关的头，如身份验证头。

如果自定义要传递的标题集，则必须指定完整的标题列表，包括通常默认包含的标题。

例如，处理发布实例的页面激活请求的Dispatcher实例需要部 `PATH` 分中的标 `/clientheaders` 题。 该头 `PATH` 支持复制代理与调度程序之间的通信。

以下代码是以下配置的示例 `/clientheaders`:

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

该属 `/virtualhosts` 性定义Dispatcher接受的此农场的所有主机名/URI组合的列表。 可以使用星号("*")作为通配符。 /属性的值 `virtualhosts` 使用以下格式：

```xml
[scheme]host[uri][*]
```

* `scheme`:（可选） `https://` 或 `https://.`
* `host`:主机的名称或IP地址以及端口号（如有必要）。 (请参 [阅https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`:（可选）资源的路径。

以下示例配置处理myCompany的。com和。ch域以及mySubDivision的所有域的请求：

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

以下配置处理所 *有请求* :

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解析虚拟主机 {#resolving-the-virtual-host}

当Dispatcher收到HTTP或HTTPS请求时，它会查找最匹配请求的虚拟主 `host,` 机 `uri`值 `scheme` 和头。 调度程序按以下顺序计 `virtualhosts` 算属性中的值：

* 调度程序从最低的农场开始，并在dispatcher.any文件中向上进行。
* 对于每个农场，Dispatcher以属性中最顶部的值开 `virtualhosts` 头，并向下进入值列表。

调度程序通过以下方式查找最匹配的虚拟主机值：

* 使用与请求的所有三个、和 `host`匹配的 `scheme`第一个遇 `uri` 到的虚拟主机。
* 如果没 `virtualhosts` 有与请求和 `scheme` 请求的部分相匹配的值， `uri` 则使用与请求 `scheme` 相匹配的第一个遇到的虚拟主 `uri``host` 机。
* 如果没 `virtualhosts` 有值具有与请求主机匹配的主机部分，则使用最顶端农场的最顶端虚拟主机。

因此，您应将默认虚拟主机放在dispatcher.any文件最上 `virtualhosts` 方的农场中属性的顶部。

### 虚拟主机分辨率示例 {#example-virtual-host-resolution}

以下示例代表一个dispatcher.any文件的代码片断，该文件定义了两个调度程序群，每个群定义一个 `virtualhosts` 属性。

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

在此示例中，下表显示了为给定HTTP请求解析的虚拟主机：

| 请求URL | 已解析的虚拟主机 |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/*;` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 启用安全会话- /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` 必 **须在部**`"0"``/cache` 分中设置为才能启用此功能。

创建安全会话以访问渲染场，以便用户需要登录才能访问场中的任何页面。 登录后，用户可以访问农场中的页面。 有关 [将此功能用于CUG的信息](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/cug.html#CreatingTheUserGroupToBeUsed) ，请参阅创建已关闭的用户组。 此外，请在上线前参阅 [调度程序安全核对清单](/help/using/security-checklist.md) 。

该 `/sessionmanagement` 属性是的子属性 `/farms`。

>[!CAUTION]
>
>如果网站的各个部分使用不同的访问要求，您需要定义多个场。

**/sessionmanagement** 有几个子参数：

**/directory** （必填）

存储会话信息的目录。 如果目录不存在，则创建该目录。

>[!CAUTION]
>
> 配置目录子参数时 **不要指向根文件夹** (`/directory "/"`)，因为它可能导致严重问题。 您应始终指定存储会话信息的文件夹的路径。 例如：

```xml
/sessionmanagement 
  { 
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** （可选）

会话信息的编码方式。 使用“md5”使用md5算法进行加密，或使用“hex”使用十六进制编码。 如果加密会话数据，则有权访问文件系统的用户无法读取会话内容。 默认值为“md5”。

**/header** （可选）

存储授权信息的HTTP头或cookie的名称。 如果将信息存储在http头中，请使用 `HTTP:<*header-name*>`。 要将信息存储在cookie中，请使用 `Cookie:<header-name>`。 如果不指定值，则 `HTTP:authorization` 使用。

**/timeout** （可选）

会话在上次使用后超时的秒数。 如果未指定，则使用“800”，因此会话在用户最后一个请求后的13分钟多一点时间超时。

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

/renders属性定义Dispatcher向其发送请求以呈现文档的URL。 以下示例部 `/renders` 分标识要渲染的单个AEM实例：

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

指定访问AEM实例的连接超时（以毫秒为单位）。 默认值为“0”，导致调度程序无限期等待。

**/receiveTimeout**

指定允许响应花费的时间（以毫秒为单位）。 默认值为“600000”，导致Dispatcher等待10分钟。 设置为“0”将完全消除超时。\
如果在解析响应头时到达超时，则返回504的HTTP状态（坏网关）。 如果在读取响应主体时达到超时，调度程序将向客户端返回不完整的响应，但删除可能已写入的任何缓存文件。

**/ipv4**

指定Dispatcher是使用函 `getaddrinfo` 数（对于IPv6）还是函数 `gethostbyname` （对于IPv4）来获取渲染的IP地址。 值为0将 `getaddrinfo` 使用。 值1表示 `gethostbyname` 使用。 默认值为 0。

getaddrinfo函数返回IP地址列表。 调度程序会重复该地址列表，直到它建立TCP/IP连接。 因此，当呈现主机名与多个IP地址关联时，ipv4属性很重要，而主机响应getaddrinfo函数返回始终按相同顺序排列的IP地址列表。 在这种情况下，您应使用gethostbyname函数，以便Dispatcher连接的IP地址是随机的。

Amazon Elastic Load Balancing(ELB)是一项服务，它通过一个可能按相同顺序排列的IP地址列表响应getaddrinfo。

**/secure**

如果 `/secure` 该属性的值为“1”，则调度程序使用HTTPS与AEM实例通信。 有关其他详细信息，另请参 [阅将Dispatcher配置为使用SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl)。

**/always-resolve**

在Dispatcher版 **本4.1.6中**，您可以按如下方式配 `/always-resolve` 置该属性：

* 设置为“1”时，它将解析每个请求的主机名（调度程序将永远不缓存任何IP地址）。 由于需要额外的呼叫来获取每个请求的主机信息，因此可能会对性能产生轻微的影响。
* 如果未设置该属性，则默认情况下将缓存IP地址。

此外，当您遇到动态IP解决问题时，可以使用此属性，如下例所示：

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

## 配置对内容的访问 {#configuring-access-to-content-filter}

使用该 `/filter` 部分指定Dispatcher接受的HTTP请求。 所有其他请求都会以404错误代码（找不到页面）发送回Web服务器。 如果不存 `/filter` 在任何部分，则接受所有请求。

**** 注意：始终拒绝 [对statfile](dispatcher-configuration.md#main-pars-title-28) 的请求。

>[!CAUTION]
>
>有关使用Dispatcher限 [制访问时的进一步注意事项，请参阅Dispatcher安全核对清单](security-checklist.md) 。 另外，请阅读 [AEM安全检查列表](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html) ，以了解有关AEM安装的其他安全详细信息。

/filter部分由一系列规则组成，这些规则根据HTTP请求的请求行部分的模式拒绝或允许访问内容。 您应该为/filter部分使用whilelist策略：

* 首先，拒绝访问所有内容。
* 允许根据需要访问内容。

### 定义过滤器 {#defining-a-filter}

该部分中的每 `/filter` 个项目包括类型和与请求行或整个请求行的特定元素相匹配的模式。 每个过滤器可以包含以下项目：

* **类型**:指示 `/type` 是允许还是拒绝对与模式匹配的请求的访问。 该值可以是 `allow` 或 `deny`。

* **** 请求行的要素：包 `/method`括、 `/url`、 `/query``/protocol` 或，以及根据HTTP请求的请求行部分的这些特定部分过滤请求的模式。 对请求行的元素（而不是整个请求行）进行筛选是首选的筛选方法。

* **** 请求行的高级要素：从Dispatcher 4.2.0开始，有四个新的过滤器元素可供使用。 这些新元素分 `/path`别是 `/selectors`、 `/extension` 和 `/suffix` 。 包括一个或多个这些项目以进一步控制URL模式。

>[!NOTE]
>
>有关每个元素引用的请求行的哪一部分的详细信息，请参阅 [Sling URL分解](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) Wiki页面。

* **glob属性**:该 `/glob` 属性用于与HTTP请求的整个请求行匹配。

>[!CAUTION]
>
>使用globs进行过滤在Dispatcher中已弃用。 因此，应避免在部分中使用glob, `/filter` 因为这可能导致安全问题。 因此，不应该：

`/glob "* *.css *"`

您应使用

`/url "*.css"`

#### HTTP请求的请求行部分 {#the-request-line-part-of-http-requests}

HTTP/1.1定义请 [求行](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) ，如下所示：

*方法Request-URI HTTP-Version*&lt;CRLF&gt;

&lt;CRLF&gt;字符表示回车符，后跟换行符。 以下示例是当客户端请求Geometrixx-Outoors站点的完整页面时收到的请求行：

获取/content/geometrixx-outdoors/en.html HTTP.1.1&lt;CRLF&gt;

您的模式必须考虑请求行中的空格字符和&lt;CRLF&gt;字符。

#### 双引号与单引号 {#double-quotes-vs-single-quotes}

创建过滤器规则时，请对简单模式使用 `"pattern"` 双引号。 如果您使用的是Dispatcher 4.2.0或更高版本，并且您的模式包含正则表达式，则必须将正则表达式模式用单引 `'(pattern1|pattern2)'` 号括起来。

#### Regular Expressions {#regular-expressions}

在Dispatcher 4.2.0之后，您可以在过滤器模式中包含POSIX Extended正则表达式。

#### 过滤器疑难解答 {#troubleshooting-filters}

如果您的过滤器没有按照您期望的方式触发，请在调度程序上启用 [Trace Logging](#trace-logging) （跟踪日志记录），以便您能够看到哪个过滤器正在拦截请求。

#### 示例过滤器：全部拒绝 {#example-filter-deny-all}

以下示例过滤器部分使Dispatcher拒绝所有文件的请求。 您应拒绝访问所有文件，然后允许访问特定区域。

```xml
  /0001  { /glob "*" /type "deny" }
```

对明确拒绝的区域的请求导致返回404错误代码（找不到页面）。

#### 示例过滤器：拒绝对特定区域的访问 {#example-filter-deny-acess-to-specific-areas}

过滤器还允许您拒绝访问各种元素，例如ASP页面和发布实例中的敏感区域。 以下筛选器拒绝访问ASP页面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 示例过滤器：启用POST请求 {#example-filter-enable-post-requests}

以下示例过滤器允许通过POST方法提交表单数据：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 示例过滤器：允许访问工作流控制台 {#example-filter-allow-access-to-the-workflow-console}

以下示例显示了一个用于拒绝外部访问工作流控制台的过滤器：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

如果您的发布实例使用Web应用程序上下文（例如发布），也可以将其添加到筛选器定义中。

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

如果您仍需要访问受限区域内的单个页面，则可以允许访问这些页面。 例如，要允许访问“工作流”控制台中的“存档”选项卡，请添加以下部分：

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>当多个过滤器模式应用于请求时，应用的最后一个过滤器模式是有效的。

#### 示例过滤器：使用正则表达式 {#example-filter-using-regular-expressions}

此过滤器使用正则表达式在非公共内容目录中启用扩展，该表达式在单引号之间定义：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 示例过滤器：筛选请求URL的其他元素 {#example-filter-filter-additional-elements-of-a-request-url}

下面是一个规则示例，它使用路径、选择器和扩展的 `/content` 过滤器阻止从路径及其子树捕获的内容：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### 示例/filter部分 {#example-filter-section}

配置Dispatcher时，应尽可能限制外部访问。 以下示例为外部访客提供了最低访问量：

* `/content`
* 设计和客户端库等杂项内容；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

在创建过滤器后，测 [试页面访问](dispatcher-configuration.md#main-pars-title-19) ，以确保AEM实例的安全。

dispatcher.any文件的以下/filter部分可用作 [Dispatcher配置文件的基础](dispatcher-configuration.md) 。

此示例基于随Dispatcher提供的默认配置文件，它打算作为在生产环境中使用的示例。 前缀为#的项目将停用（已注释掉），如果您决定激活其中的任何项目（通过删除该行上的#），则应当小心，因为这会对安全造成影响。

您应拒绝访问所有内容，然后允许访问特定（受限）元素：

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
>当与Apache一起使用时，请根据Dispatcher模块的DispatcherUseProcessedURL属性设计过滤器URL模式。 (请参 [阅Apache Web Server —— 配置Apache Web Server for Dispatcher](dispatcher-install.md#main-pars-55-35-1022)。)

>[!NOTE]
>
>有关Dynamic media的过滤器0030和0031适用于AEM 6.0及更高版本。

如果您选择扩展访问权限，请考虑以下建议：

* 如果您使 `/admin` 用的是CQ *版本5.4或更早版本* ，则应始终完全禁用对外部的访问。

* 在允许访问中的文件时，必须小心 `/libs`。 应允许个人访问。
* 拒绝对复制配置的访问，因此无法查看：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒绝访问Google Gadgets反向代理：

   * `/libs/opensocial/proxy*`

根据您的安装，可能会在或其他位置下 `/libs`存 `/apps` 在其他必须提供的资源。 您可以将文 `access.log` 件用作确定外部访问的资源的一种方法。

>[!CAUTION]
>
>对控制台和目录的访问可能会对生产环境带来安全风险。 除非您有明确的理由，否则它们应保持停用（已注释）。

>[!CAUTION]
>
>如果您在发 [布环境中使用报告](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/reporting.html#UsingReportsinaPublishEnvironment) ，应将Dispatcher配置为拒绝外部访客 `/etc/reports` 的访问。

### 限制查询字符串 {#restricting-query-strings}

自Dispatcher版本4.1.5起，使用该部 `/filter` 分限制查询字符串。 强烈建议显式允许查询字符串并通过筛选器元素排除通 `allow` 用允许。

单个条目可以有 *glob* ，也可以有方 *法*,url,***url* , *queryVersion和* NotBloy。 以下示例允许查询 `a=*` 字符串并拒绝解析到节点的URL的所有其他查询 `/etc` 字符串：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果规则包含一个 `/query`规则，则它将仅匹配包含查询字符串的请求并匹配提供的查询模式。
>
>在上例中，如果对没有查 `/etc` 询字符串的请求也应允许，则需要以下规则：


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 测试调度程序安全性 {#testing-dispatcher-security}

调度程序过滤器应阻止对AEM发布实例上的以下页面和脚本的访问。 使用Web浏览器尝试像站点访问者那样打开以下页面并验证是否返回代码404。 如果获得了任何其他结果，请调整滤镜。

请注意，您应该看到/content/add_valid_page.html?debug=layout的普通页面呈现。


* /admin
* /system/console
* /dav/crx.default
* /crx
* /bin/crxde/logs
* /jcr:system/jcr:versionStorage.json
* /_jcr_system/_jcr_versionStorage.json
* /libs/wcm/core/content/siteadmin.html
* /libs/collab/core/content/admin.html
* /libs/cq/ui/content/dumplibs.html
* /var/linkchecker.html
* /etc/linkchecker.html
* /home/users/a/admin/profile.json
* /home/users/a/admin/profile.xml
* /libs/cq/core/content/login.json
* ../libs/foundation/components/text/text.jsp
* /content/.{.}/libs/foundation/components/text/text.jsp
* /apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata
* /libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet
* /content.pages.json
* /content.languages.json
* /content.blueprint.json
* /content.-1.json
* /content.10.json
* /content.infinity.json
* /content.tidy.json
* /content.tidy。-1.blubber.json
* /content/dam.tidy。-100.json
* /content/content/geometrixx.sitemap.txt
* /content/add_valid_page.query.json?statement=//*
* /content/add_valid_page.qu%65ry.js%6Fn?statement=//*
* /content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)
* /content/add_valid_path_to_a_page/_jcr_content.json
* /content/add_valid_path_to_a_page/jcr:content_json
* /content/add_valid_path_to_a_page/_jcr_content.feed
* /content/add_valid_path_to_a_page/jcr:content_feed
* /content/add_valid_path_to_a_page/pagename。_jcr_content.feed
* /content/add_valid_path_to_a_page/pagename.jcr
* /content/add_valid_path_to_a_page/pagename.docview.xml
* /content/add_valid_path_to_a_page/pagename.docview.json
* /content/add_valid_path_to_a_page/pagename.sysview.xml
* /etc.xml
* /content.feed.xml
* /content.rss.xml
* /content.feed.html
* /content/add_valid_page.html?debug=layout
* /项目
* /标记
* /etc/replication.html
* /etc/cloudservices.html
* /欢迎

在终端或命令提示符下发出以下命令以确定是否启用了匿名写入访问。 您不应该能够将数据写入节点。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

在终端或命令提示符中发出以下命令以尝试使调度程序缓存失效，并确保收到代码404响应：

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## 启用对虚URL的访问 {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

配置Dispatcher以启用对为您的CQ或AEM页面配置的虚URL的访问。

启用对虚URL的访问后，Dispatcher会定期调用在渲染实例上运行的服务以获取虚URL列表。 调度程序将此列表存储在本地文件中。 当由于部分中的过滤器而拒绝页面请求时，调度程 `/filter` 序会咨询虚URL列表。 如果已拒绝的URL在列表中，则Dispatcher允许访问虚URL。

要启用对虚URL的访问，请向该 `/vanity_urls` 部分添加一 `/farms` 个部分，如下例所示：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

该 `/vanity_urls` 部分包含以下属性：

* `/url`:在渲染实例上运行的虚URL服务的路径。 此属性的值必须为 `"/libs/granite/dispatcher/content/vanityUrls.html"`。

* `/file`:Dispatcher存储虚URL列表的本地文件的路径。 确保Dispatcher具有对此文件的写访问权限。
* `/delay`:（秒）对虚URL服务的调用之间的时间。

>[!NOTE]
>
>如果您的渲染是AEM的实例，则必须安装 [VanityURLS-Components](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq600/component/vanityurls-components) 包才能安装虚URL服务。 (请参 [阅登录到包共享](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/package-manager.html#SigningIntoPackageShare)。)

请按照以下过程启用对虚URL的访问。

1. 如果您的渲染服务是AEM实例，请在发布实例上安装com.adobe.granite.dispatcher.vanityurl.content包（请参阅上面的说明）。
1. 对于您为AEM或CQ页面配置的每个虚URL，请确保该配置 ` [/filter](dispatcher-configuration.md#main-pars_134_32_0009)` 拒绝该URL。 如有必要，请添加一个拒绝该URL的过滤器。
1. 添加下 `/vanity_urls` 面的部分 `/farms`。
1. 重新启动Apache web服务器。

## 转发联合请求- /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

联合请求通常仅适用于Dispatcher，因此默认情况下，它们不会发送到呈示器（例如，AEM实例）。

如有必要，将/propagateSyndPost属性设置为“1”以将联合请求转发到Dispatcher。 如果设置了此选项，则必须确保过滤器部分不拒绝POST请求。

## 配置调度程序缓存- /cache {#configuring-the-dispatcher-cache-cache}

该部 `/cache` 分控制Dispatcher如何缓存文档。 配置多个子属性以实施缓存策略：


* /docroot
* /statfile
* /serveStaleOnError
* /allowAuthorized
* /规则
* /statfilelevel
* /invalidate
* /invalidateHandler
* /allowedClients
* /ignoreUrlParams
* /headers
* /mode
* /gracePeriod
* /enableTTL


缓存部分示例如下所示：

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
>对于权限敏感型缓存，请阅读“缓存 [受保护的内容”](permissions-cache.md)。

### 指定缓存目录 {#specifying-the-cache-directory}

该属 `/docroot` 性标识存储缓存文件的目录。

>[!NOTE]
>
>该值必须与Web服务器的文档根路径完全相同，这样调度程序和Web服务器才能处理相同的文件。\
>Web服务器负责在使用调度程序缓存文件时提供正确的状态代码，因此，它也能找到它很重要。

如果您使用多个农场，则每个农场必须使用不同的文档根。

### 命名Statfile {#naming-the-statfile}

该 `/statfile` 属性标识要用作statfile的文件。 调度程序使用此文件注册最新内容更新的时间。 statfile可以是Web服务器上的任何文件。

statfile没有内容。 更新内容时，Dispatcher会更新时间戳。 默认的stat文件名为。stat，存储在docroot中。 调度程序阻止对statfile的访问。

>[!NOTE]
>
>如果 `/statfileslevel` 已配置，则Dispatcher将忽略该 `/statfile` 属性，并使用。stat作为名称。

### 出错时提供过时文档 {#serving-stale-documents-when-errors-occur}

该属 `/serveStaleOnError` 性控制当渲染服务器返回错误时，Dispatcher是否返回无效文档。 默认情况下，当触及statfile并使缓存内容无效时，Dispatcher会在下次请求缓存内容时删除该内容。

如果 `/serveStaleOnError` 设置为“1”，则除非渲染服务器返回成功响应，否则Dispatcher不会从缓存中删除无效内容。 AEM的5xx响应或连接超时导致Dispatcher提供过时的内容，并响应HTTP状态为111（重新验证失败）。

### 使用身份验证时缓存 {#caching-when-authentication-is-used}

属性 `/allowAuthorized` 控制是否缓存包含以下任意身份验证信息的请求：

* The `authorization` header.
* 名为的Cookie `authorization`。
* 名为的Cookie `login-token`。

默认情况下，包含此身份验证信息的请求不会缓存，因为当缓存的文档返回到客户端时，不会执行身份验证。 此配置可阻止Dispatcher向没有必要权限的用户提供缓存的文档。

但是，如果您的要求允许缓存已验证的文档，请将/allowAuthorized设置为：

`/allowAuthorized "1"`

>[!NOTE]
>
>要启用会话管理(使 `/sessionmanagement` 用属性)，必 `/allowAuthorized` 须将属性设置为 `"0"`。

### 指定要缓存的文档 {#specifying-the-documents-to-cache}

属性 `/rules` 控制根据文档路径缓存哪些文档。 无论/rules属性如何，在以下情况下，Dispatcher都不会缓存文档：

* 如果请求URI包含问号("?")。\
   这通常指示动态页面，如无需缓存的搜索结果。
* 缺少文件扩展名。\
   Web服务器需要扩展来确定文档类型（MIME类型）。
* 身份验证头已设置（可以配置）
* 如果AEM实例使用以下标题做出响应：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>GET或HEAD（对于HTTP头）方法可由调度程序缓存。 有关响应头缓存的其他信息，请参阅 [缓存HTTP响应头部分](dispatcher-configuration.md#caching-http-response-headers) 。

/rules属性中的每个项目都包括 [全局](#designing-patterns-for-glob-properties) 模式和类型：

* 全局模式用于匹配文档的路径。
* 类型指示是否缓存与全局模式匹配的文档。 该值可以是允许（缓存文档）或拒绝（始终渲染文档）。

如果您没有动态页（超出上述规则已排除的页面），则可以配置Dispatcher以缓存所有内容。 此部分的规则部分如下所示：

```xml
/rules
  { 
    /0000  {  /glob "*"   /type "allow" }
  }
```

有关全局属性的信息，请参阅 [设计全局属性的模式](#designing-patterns-for-glob-properties)。

如果页面中有一些部分是动态的（例如新闻应用程序），或在已关闭的用户组中，您可以定义例外：

>[!NOTE]
>
>不得缓存已关闭的用户组，因为未检查缓存的页面的用户权限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }   
  }
```

**压缩**

在Apache web服务器上，您可以压缩缓存的文档。 如果客户端要求Apache以压缩形式返回文档，则使用压缩。 通过启用Apache模块自动进行压 `mod_deflate`缩，例如：

```xml
AddOutputFilterByType DEFLATE text/plain
```

默认情况下，该模块与Apache 2.x一起安装。

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

### 按文件夹级别验证文件 {#invalidating-files-by-folder-level}

使用属 `/statfileslevel` 性根据缓存文件的路径使其无效：

* 调度程 `.stat`序在每个文件夹中从Docroot文件夹创建到您指定的级别的文件。 docroot文件夹为0级。
* 通过触摸文件使文件失 `.stat` 效。 将文 `.stat` 件的上次修改日期与缓存文档的上次修改日期进行比较。 如果文件较新，则重新获取 `.stat` 文档。

* 当某个级别的文件失效时，从 **Docroot到**`.stat` Willever的所有文件 ****`statsfilevel` （以小为准）都将失效。

   * 例如：如果将属性设 `statfileslevel` 置为6，并且某个文件在级别5处无效，则将触及从 `.stat` docroot到5的每个文件。 继续此示例，如果文件在级别7时失效，则每次失效。 `stat` 文件（从docroot到6）将被触碰(自 `/statfileslevel = "6"`此)。

仅无效文件路径**的资源**受到影响。 请考虑以下示例：网站使用结构如果 `/content/myWebsite/xx/.` 您将其设置为 `statfileslevel` 3，则会创 `.stat`建如下文件：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

当中的文件失 `/content/myWebsite/xx` 效时，将触 `.stat` 及从Docroot到Docroot的每 `/content/myWebsite/xx`个文件。 这只适用于，而不 `/content/myWebsite/xx` 适用于 `/content/myWebsite/yy` 或 `/content/anotherWebSite`。

>[!NOTE]
>
>可以通过发送其他标题来防止失效 `CQ-Action-Scope:ResourceOnly`。 这可用于刷新特定资源而不使缓存的其他部分失效。 有关其 [他详细信息](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) ，请参 [阅此页和手动使调度程序缓存失效](https://helpx.adobe.com/experience-manager/dispatcher/using/page-invalidate.html) 。

>[!NOTE]
>
>如果为属性指定了值， `/statfileslevel` 则忽略该 `/statfile` 属性。

### 自动使缓存文件失效 {#automatically-invalidating-cached-files}

该属 `/invalidate` 性定义在内容更新时自动失效的文档。

使用自动失效时，Dispatcher不会在内容更新后删除缓存文件，但会在下次请求它们时检查其有效性。 缓存中未自动失效的文档将保留在缓存中，直到内容更新明确删除它们。

HTML页面通常使用自动失效。 HTML页面通常包含指向其他页面的链接，因此很难确定内容更新是否影响页面。 要确保在内容更新时所有相关页面都失效，请自动使所有HTML页面失效。 以下配置使所有HTML页无效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

有关全局属性的信息，请参阅 [设计全局属性的模式](#designing-patterns-for-glob-properties)。

此配置在激活/content/geometrixx/cn时导致以下活动：

* 所有文件均采用模式en。*已从/content/geometrixx/文件夹中删除。
* /content/geometrixx/cn/_jcr_content文件夹已删除。
* 不会立即删除与/invalidate配置匹配的所有其他文件。 下一个请求发生时，这些文件将被删除。 在我们的示例中，/content/geometrixx.html未被删除，将在请求/content/geometrixx.html时将其删除。

如果您提供自动生成的PDF和ZIP文件供下载，则可能也必须自动使这些文件失效。 配置示例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM与Adobe Analytics集成后，可在您网站的analytics.sitecatalyst.js文件中提供配置数据。 随Dispatcher提供的dispatcher.any文件示例包含此文件的以下失效规则：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自定义失效脚本 {#using-custom-invalidation-scripts}

/invalidateHandler属性允许您定义一个脚本，该脚本为Dispatcher收到的每个失效请求调用。

它使用以下参数调用：

* 处理\
   无效的内容路径
* 操作\
   复制操作（例如，激活、取消激活）
* 操作范围\
   复制操作的范围(空，除非发送的头为空，否则请参 `CQ-Action-Scope: ResourceOnly` 阅从AEM [中使缓存页面失效](page-invalidate.md) ，以了解详细信息)

这可用于涵盖许多不同的用例，如使其他应用程序特定的缓存失效，或处理页面的外部化URL及其在Docroot中的位置与内容路径不匹配的情况。

以下示例脚本记录每个对文件的无效请求。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### 示例失效处理程序脚本 {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### 限制可刷新缓存的客户端 {#limiting-the-clients-that-can-flush-the-cache}

/allowedClients属性定义允许刷新缓存的特定客户端。 所述覆盖图案与所述IP匹配。

以下示例：

1. 拒绝访问任何客户端
1. 明确允许访问localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

有关全局属性的信息，请参阅 [设计全局属性的模式](#designing-patterns-for-glob-properties)。

>[!CAUTION]
>
>建议您定义/allowedClients。
>
>如果不这样做，任何客户端都可以发出调用以清除缓存；如果重复执行此操作，可能会严重影响站点性能。

### 忽略URL参数 {#ignoring-url-parameters}

该部 `ignoreUrlParams` 分定义在确定从缓存中缓存还是传送页面时忽略的URL参数：

* 当请求URL包含所有被忽略的参数时，将缓存页面。
* 当请求URL包含一个或多个不被忽略的参数时，不会缓存页面。

当页面的参数被忽略时，在首次请求页面时将缓存该页面。 对页面的后续请求将在缓存的页面中提供，而不管请求中参数的值如何。

要指定忽略哪些参数，请向属性添加全局 `ignoreUrlParams` 规则：

* 要忽略参数，请创建允许该参数的glob属性。
* 要阻止缓存页面，请创建拒绝该参数的glob属性。

以下示例使Dispatcher忽略“q”参数，以便缓存包含q参数的请求URL:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

使用示例 `ignoreUrlParams` 值，以下HTTP请求会导致页面被缓存，因为该参 `q` 数被忽略：

```xml
GET /mypage.html?q=5
```

使用示 `ignoreUrlParams` 例值，以下HTTP请求会导致页面不 **被缓存** ，因为 `p` 参数不会被忽略：

```xml
GET /mypage.html?q=5&p=4
```

有关全局属性的信息，请参阅 [设计全局属性的模式](#designing-patterns-for-glob-properties)。

### 缓存HTTP响应头 {#caching-http-response-headers}

>[!NOTE]
>
>此功能适用于 **Dispatcher版本4.1.11** 。

该 `/headers` 属性允许您定义将由调度程序缓存的HTTP头类型。 在对未缓存资源的第一个请求中，与配置值之一匹配的所有标题（请参见下面的配置示例）存储在缓存文件旁边的单独文件中。 在对缓存资源的后续请求中，存储的标头被添加到响应。

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
>另外，请注意，不允许使用文件格式化字符。 有关详细信息，请参 [阅为全局属性设计模式](#designing-patterns-for-glob-properties)。

>[!NOTE]
>
>如果您需要Dispatcher从AEM存储和交付ETag响应头，请执行以下操作：
>
>* 在部分中添加标题 `/cache/headers`名称。
>* 在与Dispatcher相关的 [部分中添加](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) 以下Apache指令：
>



```xml
FileETag none
```

### 调度程序缓存文件权限 {#dispatcher-cache-file-permissions}

该属 `mode` 性指定将哪些文件权限应用于缓存中的新目录和文件。 此设置受调用进 `umask` 程的限制。 它是由以下一个或多个值之和构建的八进制数：

* 0400允许由所有者读取。
* 0200允许由所有者写入。
* 0100允许所有者在目录中搜索。
* 0040允许由用户组成员读取。
* 0020允许由用户组成员写入。
* 0010允许用户组成员在目录中搜索。
* 0004允许他人阅读。
* 0002允许他人写入。
* 0001允许其他人在目录中搜索。

默认值为0755，它允许所有者读取、写入或搜索，组和其他人读取或搜索。

### 限制。stat文件触控 {#throttling-stat-file-touching}

使用默认属 `/invalidate` 性时，每次激活都会有效地使所 `.html` 有文件失效(当其路径与部分相 `/invalidate` 匹配时)。 在流量较大的网站上，多次后续激活会增加后端的CPU负载。 在这种情况下，最好“限制”文件触摸，以 `.stat` 使网站保持响应。 您可以使用属性执行此操作 `/gracePeriod` 操作。

该属 `/gracePeriod` 性定义在上次激活之后，过时的自动失效的资源仍可从缓存中服务的秒数。 该属性可用于设置中，如果进行批量激活，则会重复使整个缓存失效。 建议的值为2秒。

有关其他详细信息，请阅读上 `/invalidate` 述和 `/statfileslevel`部分。

## 配置基于时间的缓存失效- /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

如果设置， `enableTTL` 该属性将评估来自后端的响应标头，并且如果它们包含 `Cache-Control``Expires` max-age或date，则会在缓存文件旁边创建一个辅助的空文件，修改时间等于到期日期。 当在修改时间之后请求缓存的文件时，会从后端自动重新请求该文件。

可通过向文件添加以下代码行来启用该 `dispatcher.any` 功能：

```xml
/enableTTL "1"
```

>[!NOTE]
>
>此功能适用于 **Dispatcher版本4.1.11** 。

## 配置负载平衡- /statistics {#configuring-load-balancing-statistics}

该部 `/statistics` 分定义了文件的类别，Dispatcher为这些类别对每个渲染的响应性进行评分。 调度程序使用得分确定要发送请求的渲染。

您创建的每个类别都定义全局模式。 调度程序将所请求内容的URI与这些模式进行比较，以确定所请求内容的类别：

* 类别的顺序决定了它们与URI的比较顺序。
* 与URI匹配的第一个类别模式是文件的类别。 不再评估类别模式。

调度程序最多支持8个统计数据类别。 如果定义的类别超过8个，则只使用前8个类别。

**渲染选择**

每次Dispatcher需要渲染的页面时，它都会使用以下算法来选择渲染：

1. 如果请求包含Cookie中的渲染名称， `renderid` 则Dispatcher将使用该渲染。
1. 如果请求不包含 `renderid` cookie，则Dispatcher将比较渲染统计信息：

   1. 调度程序确定请求URI的类别。
   1. 调度程序确定哪个渲染对该类别的响应得分最低，并选择该渲染。

1. 如果尚未选择任何渲染，请使用列表中的第一个渲染。

渲染器类别的得分基于以前的响应时间以及调度程序尝试的先前失败和成功连接。 对于每次尝试，将更新所请求URI类别的得分。

>[!NOTE]
>
>如果不使用负载平衡，可忽略此部分。

### 定义统计信息类别 {#defining-statistics-categories}

为要保留用于渲染选择的统计信息的每种类型的文档定义一个类别。 /statistics部分包含/categories部分。 要定义类别，请在/categories部分下添加一行，其格式如下：

`/name { /glob "pattern"}`

该类别 `name` 必须是农场特有的。 有关 `pattern` 信息，请参阅全局属 [性的设计模式部分](#designing-patterns-for-glob-properties) 。

要确定URI的类别，调度程序会将URI与每个类别模式进行比较，直到找到匹配项。 调度程序从列表中的第一个类别开始，并按顺序继续。 因此，请首先放置具有更具体模式的类别。

例如，默认dispatcher.any文件的Dispatcher定义了HTML类别和其他类别。 HTML类别更具体，因此它首先显示：

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

### 在调度程序统计中反映服务器不可用性 {#reflecting-server-unavailability-in-dispatcher-statistics}

该属 `/unavailablePenalty` 性设置当与渲染器的连接失败时应用于渲染统计信息的时间（以十分之一秒为单位）。 调度程序将时间添加到与请求的URI匹配的统计信息类别。

例如，当由于AEM未运行（且未侦听）或由于网络相关问题而无法建立到指定主机名／端口的TCP/IP连接时，将应用惩罚。

该 `/unavailablePenalty` 属性是该部分的直接子项( `/farm` 该部分的兄弟项 `/statistics` )。

如果不 `/unavailablePenalty` 存在任何属性，则使用值“1”。

```xml
/unavailablePenalty "1"
```

## 识别粘性连接文件夹- /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

该属 `/stickyConnectionsFor` 性定义了一个包含粘性文档的文件夹；这将使用URL进行访问。 调度程序将位于此文件夹中的所有请求从单个用户发送到同一个渲染实例。 粘性连接可确保所有文档都具有一致的会话数据。 此机制使用 `renderid` cookie。

以下示例定义到/products文件夹的粘性连接：

```xml
/stickyConnectionsFor "/products"
```

当页面由来自多个内容节点的内容组成时，请包 `/paths` 含列出内容路径的属性。 例如，页面包含来自、 `/content/image`和 `/content/video`的内容 `/var/files/pdfs`。 以下配置为页面上的所有内容启用了粘性连接：

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

启用粘性连接后，调度程序模块将设置 `renderid` cookie。 此Cookie没有标记， `httponly` 为了增强安全性，应添加该标记。 可以通过在配置文件的节 `httpOnly` 点中设置属 `/stickyConnections` 性来执行 `dispatcher.any` 此操作。 属性的值（0或1）定义 `renderid` cookie是否附加了 `HttpOnly` 属性。 默认值为0，表示不会添加属性。

有关该标志的其 `httponly` 他信息，请阅 [读此页](https://www.owasp.org/index.php/HttpOnly)。

### 安全 {#secure}

启用粘性连接后，调度程序模块将设置 `renderid` cookie。 此Cookie没有安全标 **志** ，为了增强安全性，应添加安全标志。 可以通过在配置文件的节 `secure` 点中设置属 `/stickyConnections` 性来执行 `dispatcher.any` 此操作。 属性的值（0或1）定义 `renderid` cookie是否附加了 `secure` 属性。 默认值为0，这意味着如果传入的请求是安全 **的** ，将添加属性。 如果将该值设置为1，则无论传入的请求是否安全，都将添加安全标志。

## 处理渲染连接错误 {#handling-render-connection-errors}

在渲染服务器返回500错误或不可用时配置Dispatcher行为。

### 指定运行状况检查页面 {#specifying-a-health-check-page}

使用 `/health_check` 属性指定在发生500状态代码时检查的URL。 如果此页还返回500状态代码，则该实例被视为不可用，在重试之前，可配置的时间代价( `/unavailablePenalty`)将应用于渲染。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定页面重试延迟 {#specifying-the-page-retry-delay}

/属性 `retryDelay` 设置调度程序在与农场进行连接尝试的几轮之间等待的时间（以秒为单位）。 对于每轮，调度程序尝试连接到渲染器的最大次数是农场中的渲染数量。

如果未显式定义， `"1"` 则调 `/retryDelay` 度程序将使用值。 在大多数情况下，默认值是合适的。

```xml
/retryDelay "1"
```

### 配置重试次数 {#configuring-the-number-of-retries}

该属 `/numberOfRetries` 性设置调度程序对渲染器执行的最大连接尝试次数。 如果Dispatcher在此次重试次数后无法成功连接到渲染器，则Dispatcher返回失败的响应。

对于每轮，调度程序尝试连接到渲染器的最大次数是农场中的渲染数量。 因此，调度程序尝试连接的最大次数是( `/numberOfRetries`)x（渲染次数）。

如果未显式定义值，则默认值为 **5**。

```xml
/numberOfRetries "5"
```

### 使用故障转移机制 {#using-the-failover-mechanism}

在原始请求失败时，启用Dispatcher群上的故障转移机制以向不同渲染器重新发送请求。 启用故障转移后，Dispatcher具有以下行为：

* 当对渲染器的请求返回HTTP状态503(UNAVAILABLE)时，调度程序会将该请求发送到其他渲染器。
* 当对渲染器的请求返回HTTP状态50x（除503之外）时，调度程序会发送为属性配置的页面的请 `health_check` 求。

   * 如果运行状况检查返回500(INTERNAL_SERVER_ERROR)，则Dispatcher会将原始请求发送到其他渲染器。
   * 如果综合检查返回HTTP状态200，则调度程序将初始HTTP 500错误返回给客户端。

要启用故障转移，请将以下代码行添加到农场（或网站）:

```xml
/failover "1" 
```

>[!NOTE]
>
>要重试包含正文的HTTP请求，Dispatcher在假设实际内容之 `Expect: 100-continue` 前会向渲染器发送请求标头。 带有CQSE的CQ 5.5随后会立即回答100（继续）或错误代码。 其他servlet容器也应支持此功能。

## 忽略中断错误- /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此选项。 您只需在看到以下日志消息时使用它：
>
>`Error while reading response: Interrupted system call`

如果系统调用的对象位于通过NFS访 `EINTR` 问的远程系统上，则任何面向文件系统的系统调用都可以中断。 这些系统调用是否可以超时或中断取决于基础文件系统在本地计算机上的安装方式。

如果实例具有此类配置，并且日志包含以下消息，请使用/ignoreEINTR参数：

`Error while reading response: Interrupted system call`

在内部，Dispatcher使用可表示为：的循环从远程服务器（即AEM）读取响应：

`while (response not finished) {  
read more data  
}`

当在“ `EINTR``read more data`”部分中出现时，可以生成这样的消息，该消息是由在接收任何数据之前接收信号引起的。

要忽略此类中断，可以将以下参数添加到( `dispatcher.any` 之前 `/farms`):

`/ignoreEINTR "1"`

设置为 `/ignoreEINTR` 将 `"1"` 导致Dispatcher继续尝试读取数据，直到读取完整响应。 默认值为0，并停用该选项。

## 设计全局属性的模式 {#designing-patterns-for-glob-properties}

调度程序配置文件中的几个部分使用属 `glob` 性作为客户端请求的选择标准。 全局属性的值是调度程序与请求的一个方面进行比较的模式，例如所请求资源的路径或客户端的IP地址。 例如，部分中的项使 `/filter` 用全局模式来标识Dispatcher处理或拒绝的页面的路径。

全局值可以包括通配符和字母数字字符以定义模式。

| 通配符 | 描述 | 示例 |
|--- |--- |--- |
| `*` | 匹配字符串中任意字符的零个或多个连续实例。 匹配的最终字符由以下任一情况确定：字 <br/>符串中的字符与模式中的下一个字符匹配，并且模式字符具有以下特征：<br/><ul><li>不是*</li><li>不是？</li><li>文本字符（包括空格）或字符类。</li><li>到达图案的末尾。</li></ul>在字符类中，字符将按字面方式解释。 | `*/geo*` 匹配节点和节 `/content/geometrixx` 点下的任何 `/content/geometrixx-outdoors` 页面。 以下HTTP请求与全局模式匹配： <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` 匹 <br/>配节点下的任何 `/content/geometrixx-outdoors` 页面。 例如，以下HTTP请求与glob模式匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 匹配任何单个字符。 使用外部字符类。 在字符类中，将字面解释此字符。 | `*outdoors/??/*`<br/> 匹配geometrixx-outdoors站点中任何语言的页面。 例如，以下HTTP请求与glob模式匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>以下请求与全局模式不匹配： <br/><ul><li>“获取/content/geometrixx-outdoors/en.html”</li></ul> |
| `[ and ]` | 取消标记字符类的开始和结尾。 字符类可以包括一个或多个字符范围和单个字符。<br/>如果目标字符与字符类中的任意字符或在定义的范围内匹配，则会发生匹配。<br/>如果不包括右括号，则图案不产生匹配项。 | `*[o]men.html*`<br/> 匹配以下HTTP请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` 匹 <br/>配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示字符范围。 用于字符类。  在字符类之外，将字面解释此字符。 | `*[m-p]men.html*` 匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定后面的字符或字符类。 仅用于否定字符类中的字符和字符范围。 等效于 `^ wildcard`。 <br/>在字符类之外，将字面解释此字符。 | `*[!o]men.html*`<br/> 匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>与以下HTTP请求不匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定后面的字符或字符范围。 仅用于否定字符类中的字符和字符范围。 等效于通配 `!` 符。 <br/>在字符类之外，将字面解释此字符。 | 通配符的示例 `!` 适用，用字符替 `!` 代示例模式中的字符 `^` 。 |


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

在Web服务器配置中，可以设置：

* 调度程序日志文件的位置。
* 日志级别。

有关详细信息，请参阅Web服务器文档和Dispatcher实例的自述文件。

**Apache旋转／管道日志**

如果使用 **Apache** web服务器，则可以对旋转和／或管道日志使用标准功能。 例如，使用管道日志：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

此操作将自动旋转：

* 调度程序日志文件；带有扩展中的时间戳(logs/dispatcher.log%Y%m%d)。
* 每周（60 x 60 x 24 x 7 = 604800秒）。

请参见Log Rotation和Pinual Logs上的Apache web服务器文档；例如 [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html)。

>[!NOTE]
>
>安装后，默认日志级别为高（即级别3 =调试），这样调度程序将记录所有错误和警告。 这在初始阶段非常有用。
>
>但是，这需要额外的资源，因此当Dispatcher根据您的要求 *顺利地工作时*，您可以（应）降低日志级别。

### 跟踪日志记录 {#trace-logging}

在Dispatcher的其他增强功能中，版本4.2.0还引入了跟踪日志记录。

这比“调试”日志记录级别高，显示日志中的其他信息。 它添加了以下记录：

* 转发标题的值；
* 正应用于某个操作的规则。

您可以通过将日志级别设置为Web服务器中的“跟踪 `4` 日志记录”来启用。

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

并且请求与阻止规则匹配的文件时记录的事件：

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## 确认基本操作 {#confirming-basic-operation}

要确认Web服务器、Dispatcher和AEM实例的基本操作和交互，可以使用以下步骤：

1. 将设置 `loglevel` 为 `3`。

1. 启动Web服务器；这也会启动调度程序。
1. 启动AEM实例。
1. 检查Web服务器和调度程序的日志和错误文件。\
   根据Web服务器的不同，您应当看到如下消息：\
   `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`\
   和:\
   `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 通过Web服务器浏览网站。 确认内容是按需显示的。\
   例如，在本地安装中，AEM在端口上运行，而在访问网站控 `4502` 制台时，Web服 `80` 务器使用以下两种方式：\
   ` https://localhost:4502/libs/wcm/core/content/siteadmin.html  
https://localhost:80/libs/wcm/core/content/siteadmin.html  
`结果应相同。 使用相同的机制确认对其他页面的访问。

1. 检查缓存目录是否已填写。
1. 激活页面以检查缓存是否正在正确刷新。
1. 如果一切正常，您可以将“”减 `loglevel` 少到 `0`。

## 使用多个调度程序 {#using-multiple-dispatchers}

在复杂的设置中，您可以使用多个调度程序。 例如，您可以使用：

* 一个调度程序，用于在Intranet上发布网站
* 另一个调度程序位于不同的地址下，并具有不同的安全设置，用于在Internet上发布相同的内容。

在这种情况下，请确保每个请求只通过一个调度程序。 调度程序不处理来自其他调度程序的请求。 因此，请确保两个Dispatcher都直接访问AEM网站。

## 调试 {#debugging}

在向请求中添 `X-Dispatcher-Info` 加标题时，Dispatcher会回答目标是否已缓存、从缓存中返回还是根本不可缓存。 响应标题以可 `X-Cache-Info` 读形式包含此信息。 您可以使用这些响应标头来调试与由调度程序缓存的响应有关的问题。

默认情况下，此功能未启用，因此要包含响应标 `X-Cache-Info` 题，群必须包含以下条目：

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

此外， `X-Dispatcher-Info` 标题不需要值，但如果您使用测试 `curl` ，则必须提供一个值才能发送标题，例如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/we-retail/us/en.html
```

以下是包含将返回的响应标题 `X-Dispatcher-Info` 的列表：

* **缓存**\
   目标文件包含在缓存中，调度程序已确定传送该文件是有效的。
* **缓存**\
   目标文件不包含在缓存中，调度程序已确定缓存输出并传送输出是有效的。
* **缓存：stat文件是较新的**。目标文件包含在缓存中，但是，由较新的stat文件使其失效。 调度程序将删除目标文件，从输出中重新创建并传送它。
* **不可缓存：无文档根**&#x200B;农场的配置不包含文档根(配置元素 `cache.docroot`)。
* **不可缓存：缓存文件路径过长**\
   目标文件（文档根文件和URL文件的串联）超过系统上最长的可能文件名。
* **不可缓存：临时文件路径过长**\
   临时文件名模板超出系统上最长的可能文件名。 调度程序首先创建临时文件，然后实际创建或覆盖缓存的文件。 临时文件名是目标文件名，并附加字符， `_YYYYXXXXXX` 其中将替换和 `Y``X` 以创建唯一的名称。
* **不可缓存：请求URL没有扩展名**\
   请求URL没有扩展名，或者文件扩展名后面有一个路径，例如： `/test.html/a/path`.
* **不可缓存：请求不是GET或HEAD** HTTP方法既不是GET也不是HEAD。 调度程序假定输出将包含不应缓存的动态数据。
* **不可缓存：包含查询字符串的请求**\
   请求包含查询字符串。 调度程序假定输出取决于给定的查询字符串，因此不进行缓存。
* **不可缓存：会话管理器未验证**\
   农场的缓存由会话管理器（配置包含节点）管 `sessionmanagement` 理，而请求不包含相应的身份验证信息。
* **不可缓存：请求包含授权**\
   不允许群缓存输出( `allowAuthorized 0`)，并且请求包含身份验证信息。
* **不可缓存：target是目录**\
   目标文件是目录。 这可能指出某些概念性错误，即URL和某些子URL都包含可缓存输出，例如，如果请求首先出现并且包含可缓存输出，则调度程序将无法缓存后续请求的输出 `/test.html/a/file.ext``/test.html`。
* **不可缓存：request URL有尾随斜杠**\
   请求URL具有尾随斜杠。
* **不可缓存：请求URL不在缓存规则中**\
   农场的缓存规则明确拒绝缓存某些请求URL的输出。
* **不可缓存：授权检查器被拒绝访问**\
   农场的授权检查器拒绝访问缓存的文件。
* **不可缓存：会话无效**`sessionmanagement` 群的缓存受会话管理器（配置包含节点）的管理，并且用户的会话无效或不再有效。
* **不可缓存：响应包`no_cache `**&#x200B;含远程服务器返回一个 `Dispatcher: no_cache` 头，禁止调度程序缓存输出。
* **不可缓存：响应内容长度为**&#x200B;零响应内容长度为零；调度程序将不创建零长度文件。
