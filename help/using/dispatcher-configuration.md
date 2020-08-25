---
title: 配置 Dispatcher
seo-title: 配置 Dispatcher
description: 了解如何配置Dispatcher。
seo-description: 了解如何配置Dispatcher。
uuid: 253ef0f7-2491-4cec-ab22-97439df29fd6
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
topic-tags: dispatcher
content-type: reference
discoiquuid: aeffee8e-bb34-42a7-9a5e-b7d0e848391a
translation-type: tm+mt
source-git-commit: 31dc02adc1d16ec61d5191b86d0d1276c2e1da46
workflow-type: tm+mt
source-wordcount: '8582'
ht-degree: 2%

---


# 配置 Dispatcher{#configuring-dispatcher}

>[!NOTE]
>
>各个 Dispatcher 版本与 AEM 相互独立。如果单击以前版本 AEM 文档中嵌入的 Dispatcher 文档链接，可能会重定向到此页面。

以下各节介绍如何配置调度程序的各个方面。

## 支持IPv4和IPv6 {#support-for-ipv-and-ipv}

AEM和Dispatcher的所有元素都可以安装在IPv4和IPv6网络中。 请参 [阅IPV4和IPV6](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html#AdditionalPlatformNotes)。

## 调度程序配置文件 {#dispatcher-configuration-files}

默认情况下，调度程序配置存储在 `dispatcher.any` 文本文件中，但您可以在安装过程中更改此文件的名称和位置。

配置文件包含一系列单值或多值属性，这些属性控制Dispatcher的行为：

* 属性名称前缀有正斜杠 `/`。
* 多值属性使用大括号将子项括起来 `{ }`。

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

使用星号(“*”)作为通配符，指定要包含的文件范围。

例如，如果文件包含 `farm_1.any` 1到 `farm_5.any` 5个农场的配置，则可以按如下方式包括它们：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用环境变量 {#using-environment-variables}

您可以在dispatcher.any文件中的字符串值属性中使用环境变量，而不是硬编码这些值。 要包含环境变量的值，请使用格式 `${variable_name}`。

例如，如果dispatcher.any文件与缓存目录位于同一目录中，则可以使用docroot属 [性的](dispatcher-configuration.md#main-pars-title-30) 以下值：

```xml
/docroot "${PWD}/cache"
```

另一个示例是，如果创建一个名为的环境变 `PUBLISH_IP` 量来存储AEM发布实例的主机名，则可以使用 [/renders属性的](dispatcher-configuration.md#main-pars-127-25-0008) 以下配置：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 命名调度程序实例 {#naming-the-dispatcher-instance-name}

使用属 `/name` 性指定唯一名称以标识Dispatcher实例。 该 `/name` 属性是配置结构中的顶级属性。

## 定义场 {#defining-farms-farms}

该属 `/farms` 性定义一组或多组Dispatcher行为，其中每组行为都与不同的网站或URL关联。 酒店 `/farms` 可以包括一个农场或多个农场：

* 当您希望Dispatcher以相同方式处理所有网页或网站时，请使用单个场。
* 当网站的不同区域或不同网站需要不同的调度程序行为时，创建多个场。

该 `/farms` 属性是配置结构中的顶级属性。 要定义场，请向属性添加子属 `/farms` 性。 使用属性名称，该属性名称可唯一标识调度程序实例中的场。

该属 `/farmname` 性是多值的，并且包含定义调度程序行为的其他属性：

* 场应用的页面的URL。
* 一个或多个用于渲染文档的服务URL(通常为AEM发布实例)。
* 用于负载平衡多个文档呈示器的统计信息。
* 其他几种行为，如要缓存的文件和位置。

该值可以包含任何字母数字(a-z, 0-9)字符。 以下示例显示了名为和的两个农场的骨架 `/daycom` 定义 `/docsdaycom`:

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
>如果您使用多个渲染场，则从下到上计算列表。 这在为网站定义虚 [拟主机](dispatcher-configuration.md#main-pars-117-15-0006) 时尤其重要。

每个农场属性都可以包含以下子属性：

| 属性名称 | 描述 |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | 默认主页（可选）（仅限IIS） |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | 要传递的客户端HTTP请求的标头。 |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | 此场的虚拟主机。 |
| [/会话管理](#enabling-secure-sessions-sessionmanagement) | 支持会话管理和身份验证。 |
| [/renders](#defining-page-renderers-renders) | 提供呈现页面的服务器(通常为AEM发布实例)。 |
| [/filter](#configuring-access-to-content-filter) | 定义Dispatcher启用访问的URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 配置对虚URL的访问。 |
| [/propagateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | 支持转发联合请求。 |
| [/cache](#configuring-the-dispatcher-cache-cache) | 配置缓存行为。 |
| [/statistics](#configuring-load-balancing-statistics) | 为负载平衡计算定义统计类别。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | 包含粘滞文档的文件夹。 |
| [/health_check](#specifying-a-health-check-page) | 用于确定服务器可用性的URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重试失败的连接之前的延迟。 |
| [/unavailableDescument](#reflecting-server-unavailability-in-dispatcher-statistics) | 影响负载平衡计算统计的惩罚。 |
| [/failover](#using-the-failover-mechanism) | 在原始请求失败时向不同呈现重新发送请求。 |
| [/auth_checker](permissions-cache.md) | 有关对权限敏感的缓存，请参阅 [缓存安全内容](permissions-cache.md)。 |

## 指定默认页面（仅限IIS）- /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>参 `/homepage`数（仅限IIS）不再工作。 而应使用IIS URL [重写模块](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)。
>
>如果您使用的是Apache，则应使用 `mod_rewrite` 模块。 有关Apache 2.4的信息，请 `mod_rewrite` 参阅Apache网 [站文档](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)。 使用 `mod_rewrite`时，建议使用标志“passthrough|PT”( **[传递到下一个处理程序)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** ，以强制重写引擎将内部结构的字段 `uri` 设置为 `request_rec` 字段的值 `filename` 。

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

该 `/clientheaders` 属性定义一列表HTTP头，调度程序将其从客户端HTTP请求传递给渲染器(AEM实例)。

默认情况下，Dispatcher将标准HTTP头转发到AEM实例。 在某些情况下，您可能希望转发其他标题或删除特定标题：

* 添加AEM实例在HTTP请求中需要的标头（如自定义标头）。
* 删除仅与Web服务器相关的头，如身份验证头。

如果要自定义要传递的标题集，则必须指定完整的标题列表，包括通常默认包含的标题。

例如，处理发布实例的页面激活请求的调度程序实例需 `PATH` 要部分中的 `/clientheaders` 标题。 该头 `PATH` 允许复制代理与调度程序之间进行通信。

以下代码是配置示例 `/clientheaders`:

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

该属 `/virtualhosts` 性定义调度程序为此场接受的所有主机名/URI组合的列表。 可以使用星号(“*”)字符作为通配符。 /属性的 `virtualhosts` 值使用以下格式：

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

以下配置处理所 *有请* 求：

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解析虚拟主机 {#resolving-the-virtual-host}

当Dispatcher收到HTTP或HTTPS请求时，它会找到与请求的头和头 `host,` 最 `uri`匹配的 `scheme` 虚拟主机值。 调度程序按以下顺序 `virtualhosts` 计算属性中的值：

* 调度程序从最低的场开始，并在dispatcher.any文件中向上进行。
* 对于每个场，Dispatcher以属性中最顶部的值开 `virtualhosts` 头，并向下显示值列表。

调度程序通过以下方式查找最佳匹配的虚拟主机值：

* 使用与请求的所有三个、和 `host``scheme`都匹配 `uri` 的第一个虚拟主机。
* 如果没 `virtualhosts` 有与请 `scheme` 求和 `uri` 请求匹配的值和部分，则使用与请求匹配的第一个遇 `scheme``uri``host` 到的虚拟主机。
* 如果没 `virtualhosts` 有与请求主机匹配的主机部分，则使用最顶端场的最顶端虚拟主机。

因此，您应将默认虚拟主机放在dispatcher. `virtualhosts` any文件最上方的场中属性的顶部。

### 虚拟主机解析示例 {#example-virtual-host-resolution}

以下示例代表一个来自定义两个调度程序群的dispatcher.any文件的代码片断，每个群都定义一个 `virtualhosts` 属性。

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
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 启用安全会话- /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` **必须** 在部分中 `"0"` 设置 `/cache` 为才能启用此功能。

创建安全会话以访问渲染场，这样用户需要登录才能访问场中的任何页面。 登录后，用户可以访问场中的页面。 有 [关将此功能与CUG结合使用](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/cug.html#CreatingTheUserGroupToBeUsed) ，请参阅创建已关闭的用户组。 另外，上线前请参 [阅调度程序安](/help/using/security-checklist.md) 全核对清单。

属 `/sessionmanagement` 性是的子属性 `/farms`。

>[!CAUTION]
>
>如果网站的各个部分使用不同的访问要求，您需要定义多个场。

**/sessionmanagement** 有几个子参数：

**/directory** （必填）

存储会话信息的目录。 如果目录不存在，则创建该目录。

>[!CAUTION]
>
> 配置目录子参数时 **不指向** 根文件夹(`/directory "/"`)，因为它可能导致严重问题。 您应始终指定存储会话信息的文件夹的路径。 例如：

```xml
/sessionmanagement 
  { 
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** （可选）

会话信息的编码方式。 使用“md5”表示使用md5算法进行加密，或使用“hex”表示十六进制编码。 如果加密会话数据，则具有文件系统访问权限的用户无法读取会话内容。 默认值为“md5”。

**/header** （可选）

存储授权信息的HTTP头或cookie的名称。 如果将信息存储在http头中，请使用 `HTTP:<*header-name*>`。 要将信息存储在cookie中，请使用 `Cookie:<header-name>`。 如果不指定值，则 `HTTP:authorization` 使用。

**/timeout** （可选）

会话在上次使用后超时的秒数。 如果未指定“800”，则会话在用户最后请求后13分钟多一点超时。

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

## 定义页面呈示器 {#defining-page-renderers-renders}

/renders属性定义Dispatcher向其发送请求以呈现文档的URL。 以下示例部 `/renders` 分标识用于渲染的单个AEM实例：

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

以下示例/renders部分在两个AEM实例之间平均分发呈现请求：

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

指定允许响应花费的时间（以毫秒为单位）。 默认值为“600000”，导致调度程序等待10分钟。 设置为“0”将完全消除超时。\
如果解析响应头时达到超时，则返回504的HTTP状态（错误网关）。 如果在读取响应正文时达到超时，调度程序将向客户端返回不完整的响应，但删除可能已写入的任何缓存文件。

**/ipv4**

指定调度程序是 `getaddrinfo` 使用函数（对于IPv6）还是 `gethostbyname` 函数（对于IPv4）来获取渲染器的IP地址。 值为0将 `getaddrinfo` 导致使用。 值为1将 `gethostbyname` 导致使用。 默认值为 0。

getaddrinfo函数返回IP地址列表。 调度程序重复列表地址，直到它建立TCP/IP连接。 因此，当呈现主机名与多个IP地址关联时，ipv4属性很重要，主机响应getaddrinfo函数返回始终按相同顺序排列的IP地址列表。 在这种情况下，您应使用gethostbyname函数，以便Dispatcher连接的IP地址是随机的。

Amazon弹性负载平衡(ELB)是一种对getaddrinfo做出响应的服务，其IP地址的列表可能按相同顺序排列。

**/secure**

如果该 `/secure` 属性的值为“1”，则调度程序使用HTTPS与AEM实例通信。 有关其他详细信息，另请参 [阅将调度程序配置为使用SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl)。

**/always-resolve**

在Dispatcher版 **本4.1.6中**，您可以按如 `/always-resolve` 下方式配置属性：

* 如果设置为“1”，则将解析每个请求的主机名（调度程序将不会缓存任何IP地址）。 由于需要额外的呼叫来获取每个请求的主机信息，因此可能会对性能产生轻微影响。
* 如果未设置属性，则默认情况下将缓存IP地址。

此外，当您遇到动态IP解决问题时，可使用此属性，如以下示例所示：

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

使用该 `/filter` 部分指定调度程序接受的HTTP请求。 所有其他请求都会以404错误代码（找不到页面）发回到Web服务器。 如果不存在 `/filter` 任何部分，则接受所有请求。

**注意：** 始终拒 [绝](dispatcher-configuration.md#main-pars-title-28) statfile请求。

>[!CAUTION]
>
>有关使用Dispatcher [限制访问时的进一步注意事项](security-checklist.md) ，请参阅Dispatcher安全核对清单。 另外，请阅读AEM安 [全检查列表](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html) ，了解有关AEM安装的其他安全详细信息。

/filter部分由一系列规则组成，这些规则根据HTTP请求的请求行部分的模式拒绝或允许访问内容。 您应对/filter部分使用whilelist策略：

* 首先，拒绝访问所有内容。
* 允许根据需要访问内容。

### 定义筛选器 {#defining-a-filter}

该部分中的每 `/filter` 个项目都包括类型和与请求行或整个请求行的特定元素匹配的模式。 每个筛选器都可包含以下项：

* **类型**:指示 `/type` 是否允许或拒绝对与模式匹配的请求进行访问。 该值可以是 `allow` 或 `deny`。

* **请求行的要素：** 包 `/method`括 `/url`、 `/query``/protocol` 或，以及根据HTTP请求的请求行部分的这些特定部分过滤请求的模式。 首选的筛选方法是筛选请求行的元素（而不是整个请求行）。

* **请求行的高级元素：** 从Dispatcher 4.2.0开始，有四个新的过滤器元素可供使用。 这些新元素 `/path`分 `/selectors`别 `/extension` 是 `/suffix` 和。 包括一个或多个这些项目以进一步控制URL模式。

>[!NOTE]
>
>有关每个元素引用的请求行的哪一部分的详细信息，请参 [阅Sling URL分解](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki页面。

* **全局属性**:该 `/glob` 属性用于与HTTP请求的整个请求行匹配。

>[!CAUTION]
>
>在Dispatcher中，已弃用使用globs进行过滤。 因此，应避免在部分中使用 `/filter` glob，因为这可能导致安全问题。 因此，不是：

`/glob "* *.css *"`

您应使用

`/url "*.css"`

#### HTTP请求的请求行部分 {#the-request-line-part-of-http-requests}

HTTP/1.1定义请 [求行](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) ，如下所示：

*方法Request-URI HTTP-Version*&lt;CRLF>

&lt;CRLF>字符表示回车后跟换行符。 以下示例是客户端请求Geometrixx输出站点的整个页面时收到的请求行：

GET/content/geometrixx-outdoors/en.html HTTP.1.1&lt;CRLF>

您的模式必须考虑请求行中的空格字符和&lt;CRLF>字符。

#### 多次报价与单报价 {#double-quotes-vs-single-quotes}

在创建筛选规则时，请对简单模式使用 `"pattern"` 多次引号。 如果您使用的是Dispatcher 4.2.0或更高版本，并且您的模式包含常规表达式，则必须将正则表达式模式 `'(pattern1|pattern2)'` 用单引号括起来。

#### Regular Expressions {#regular-expressions}

在Dispatcher 4.2.0之后，您可以在滤镜模式中包含POSIX Extended Regular表达式。

#### 疑难解答过滤器 {#troubleshooting-filters}

如果您的过滤器没有按您期望的方式触发，请在调 [度程序上启用](#trace-logging) “跟踪记录”，以便您能够看到哪个过滤器正在拦截请求。

#### 示例筛选器：全部拒绝 {#example-filter-deny-all}

以下示例过滤器部分导致调度程序拒绝所有文件的请求。 您应拒绝访问所有文件，然后允许访问特定区域。

```xml
  /0001  { /glob "*" /type "deny" }
```

对明确拒绝的区域的请求导致返回404错误代码（找不到页面）。

#### 示例筛选器：拒绝对特定区域的访问 {#example-filter-deny-acess-to-specific-areas}

过滤器还允许您拒绝访问各种元素，例如ASP页面和发布实例中的敏感区域。 以下筛选器拒绝访问ASP页面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 示例筛选器：启用POST请求 {#example-filter-enable-post-requests}

以下示例筛选器允许使用POST方法提交表单数据：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 示例筛选器：允许访问工作流控制台 {#example-filter-allow-access-to-the-workflow-console}

以下示例显示了用于拒绝对工作流控制台进行外部访问的筛选器：

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
>当多个过滤器模式应用于请求时，应用的最后一个过滤模式将有效。

#### 示例过滤器：使用常规表达式 {#example-filter-using-regular-expressions}

此过滤器使用常规表达式启用非公共内容目录中的扩展，在单引号之间定义：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 示例过滤器：筛选请求URL的其他元素 {#example-filter-filter-additional-elements-of-a-request-url}

下面是一个规则示例，它使用路径、选 `/content` 择器和扩展的过滤器阻止从路径及其子树捕获的内容：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### 示例/filter部分 {#example-filter-section}

配置Dispatcher时，应尽可能限制外部访问。 以下示例为外部访客提供了最低访问权限：

* `/content`
* 设计和客户端库等杂项内容；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

创建过滤器后， [测试页面访问](dispatcher-configuration.md#main-pars-title-19) ，以确保AEM实例的安全。

调度程序。any文件的以下/filter部分可用作Dispatcher配置文 [件的基础](dispatcher-configuration.md) 。

此示例基于随Dispatcher提供的默认配置文件，它打算作为在生产环境中使用的示例。 #前缀的项目将停用（已注释），如果您决定激活其中的任何项目（通过删除该行上的#），应当小心，因为这可能会对安全产生影响。

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
>当与Apache一起使用时，请根据Dispatcher模块的DispatcherUseProcessedURL属性设计您的过滤器URL模式。 (请参 [阅Apache Web Server —— 为Dispatcher配置Apache Web Server](dispatcher-install.md#main-pars-55-35-1022)。)

>[!NOTE]
>
>有关Dynamic Media的过滤器0030和0031适用于AEM 6.0及更高版本。

如果您选择扩展访问权限，请考虑以下建议：

* 如果您使 `/admin` 用的是 *CQ 5.4或* 更早版本，则应始终完全禁用对外部的访问。

* 允许访问中的文件时，必须小心 `/libs`。 应允许个人访问。
* 拒绝对复制配置的访问，因此无法看到：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒绝访问Google小工具反向代理：

   * `/libs/opensocial/proxy*`

根据您的安装，可能在或其他位置下 `/libs`存 `/apps` 在其他必须提供的资源。 您可以将文 `access.log` 件用作确定外部访问的资源的一种方法。

>[!CAUTION]
>
>对控制台和目录的访问可能会对生产环境带来安全风险。 除非您有明确的理由，否则它们应保持停用（注释掉）。

>[!CAUTION]
>
>如果在发 [布环境中使用报告](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/reporting.html#UsingReportsinaPublishEnvironment) ，应将Dispatcher配置为拒绝对外 `/etc/reports` 部访客的访问。

### 限制查询字符串 {#restricting-query-strings}

自Dispatcher 4.1.5版起，请使用该部 `/filter` 分限制查询字符串。 强烈建议显式允许查询字符串并通过筛选器元素排除通 `allow` 用允许。

单个条目可以 *具有* glob或方法、url *、*&#x200B;查询和版的某&#x200B;*些组合，***** 但两者都不是。 以下示例允许查询 `a=*` 字符串并拒绝解析到该节点的URL的所有其他查询 `/etc` 字符串：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果规则包含查询 `/query`字符串，则它将仅匹配包含查询字符串的请求，并匹配提供的模式。
>
>在上例中，如果对没有 `/etc` 查询字符串的请求也应允许，则需要以下规则：


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 测试调度程序安全性 {#testing-dispatcher-security}

调度程序过滤器应阻止访问AEM发布实例上的以下页面和脚本。 使用Web浏览器尝试像站点访客一样打开以下页面并验证是否返回代码404。 如果获得了任何其他结果，请调整过滤器。

请注意，您应当看到的是的普通页面呈 `/content/add_valid_page.html?debug=layout`现。


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

在终端或命令提示符下发出以下命令以确定是否启用了匿名写入访问。 您不应能够将数据写入节点。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

在终端或命令提示符下发出以下命令，尝试使调度程序缓存失效，并确保收到代码404响应：

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

启用对虚URL的访问时，Dispatcher会定期调用在渲染实例上运行的服务以获取虚URL的列表。 调度程序将此列表存储在本地文件中。 当由于部分中的筛选器而拒绝对页面的请 `/filter` 求时，调度程序将咨询虚URL的列表。 如果拒绝的URL在列表上，则Dispatcher允许访问虚URL。

要启用对虚URL的访问，请 `/vanity_urls` 向节添加 `/farms` 一个节，与以下示例类似：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

该 `/vanity_urls` 部分包含以下属性：

* `/url`:在渲染实例上运行的虚URL服务的路径。 此属性的值必须为 `"/libs/granite/dispatcher/content/vanityUrls.html"`。

* `/file`:本地文件的路径，调度程序存储虚URL的列表。 确保Dispatcher具有对此文件的写访问权限。
* `/delay`:（秒）调用虚URL服务之间的时间。

>[!NOTE]
>
>如果您的渲染是AEM的实例，则必须安 [装VanityURLS](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq600/component/vanityurls-components) -Components包才能安装虚URL服务。 (请参 [阅登录到包共享](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/package-manager.html#SigningIntoPackageShare)。)

请按照以下过程启用对虚URL的访问。

1. 如果您的渲染服务是AEM实例，请在发布实例上安装com.adobe.granite.dispatcher.vanityurl.content包（请参阅上面的说明）。
1. 对于您为AEM或CQ页面配置的每个虚URL，请确保配置 ` [/filter](dispatcher-configuration.md#main-pars_134_32_0009)` 拒绝该URL。 如有必要，请添加一个拒绝该URL的过滤器。
1. 添加下 `/vanity_urls` 面的部 `/farms`分。
1. 重新启动Apache Web服务器。

## 转发联合请求- /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

联合请求通常仅适用于Dispatcher，因此默认情况下，它们不会发送到呈示器(例如AEM实例)。

如有必要，将/propagateSyndPost属性设置为“1”以将联合请求转发给调度程序。 如果设置了此选项，则必须确保筛选器部分不拒绝POST请求。

## 配置调度程序缓存- /cache {#configuring-the-dispatcher-cache-cache}

该部 `/cache` 分控制调度程序如何缓存文档。 配置多个子属性以实施缓存策略：


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
>对于权限敏感型缓存，请阅读“缓存 [受保护的内容](permissions-cache.md)”。

### 指定缓存目录 {#specifying-the-cache-directory}

该属 `/docroot` 性标识存储缓存文件的目录。

>[!NOTE]
>
>该值必须与Web服务器的文档根路径完全相同，这样调度程序和Web服务器就可以处理相同的文件。\
>Web服务器负责在使用调度程序缓存文件时提供正确的状态代码，这就是它能够找到它的重要原因。

如果您使用多个场，则每个场必须使用不同的文档根。

### 命名Statfile {#naming-the-statfile}

该 `/statfile` 属性标识要用作statfile的文件。 调度程序使用此文件注册最新内容更新的时间。 statfile可以是Web服务器上的任何文件。

statfile没有内容。 更新内容时，Dispatcher会更新时间戳。 默认的stat文件名为。stat，存储在docroot中。 调度程序阻止对statfile的访问。

>[!NOTE]
>
>如果 `/statfileslevel` 已配置，则调度程序将忽 `/statfile` 略该属性，并使用。stat作为名称。

### 出错时提供过时文档 {#serving-stale-documents-when-errors-occur}

当渲染 `/serveStaleOnError` 服务器返回错误时，该属性控制Dispatcher是否返回无效文档。 默认情况下，当触及静态文件并使缓存内容无效时，Dispatcher会在下次请求缓存内容时删除该内容。

如果 `/serveStaleOnError` 设置为“1”，则除非渲染服务器返回成功响应，否则调度程序不会从缓存中删除无效内容。 AEM的5xx响应或连接超时导致Dispatcher提供过时的内容，并响应HTTP状态111（重新验证失败）。

### 使用身份验证时缓存 {#caching-when-authentication-is-used}

属性 `/allowAuthorized` 控制是否缓存包含以下任意身份验证信息的请求：

* The `authorization` header.
* 名为的cookie `authorization`。
* 名为的cookie `login-token`。

默认情况下，包含此身份验证信息的请求不会缓存，因为当缓存文档返回到客户端时，不会执行身份验证。 此配置可阻止Dispatcher向没有必要权限的用户提供缓存文档。

但是，如果要求允许缓存已验证的文档，请将/allowAuthorized设置为：

`/allowAuthorized "1"`

>[!NOTE]
>
>要启用会话管理(使 `/sessionmanagement` 用属性), `/allowAuthorized` 必须将属性设置为 `"0"`。

### 指定要缓存的文档 {#specifying-the-documents-to-cache}

属性 `/rules` 控制根据文档路径缓存哪些文档。 无论/rules属性如何，在以下情况下，调度程序都不会缓存文档:

* 如果请求URI包含问号(“?”)。\
   这通常指示动态页面，如无需缓存的搜索结果。
* 缺失文件扩展名。\
   Web 服务器需要扩展名来确定文档类型（比如 MIME 类型）。
* 设置了身份验证标头（此项可进行配置）
* 如果AEM实例使用以下标题做出响应：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>GET 或 HEAD（针对 HTTP 标头）方法可由 Dispatcher 缓存。有关响应头缓存的其他信息，请参 [阅缓存HTTP响应头](dispatcher-configuration.md#caching-http-response-headers) 。

/rules属性中的每个项都包括 [全局](#designing-patterns-for-glob-properties) 模式和类型：

* 全局模式用于匹配文档的路径。
* 类型指示是否缓存与全局模式匹配的文档。 该值可以是允许(缓存文档)或拒绝(始终呈现文档)。

如果您没有动态页面（除上述规则已排除的页面之外），您可以配置Dispatcher以缓存所有内容。 此项的规则部分如下所示：

```xml
/rules
  { 
    /0000  {  /glob "*"   /type "allow" }
  }
```

有关全局属性的信息，请参 [阅设计全局属性的模式](#designing-patterns-for-glob-properties)。

如果页面中有一些部分是动态的（例如，新闻应用程序），或在已关闭的用户组中，您可以定义例外：

>[!NOTE]
>
>关闭的用户组不得缓存，因为未检查缓存页面的用户权限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }   
  }
```

**压缩**

在Apache Web服务器上，您可以压缩缓存的文档。 如果客户端请求压缩，Apache将以压缩形式返回文档。 通过启用Apache模块自动完成压 `mod_deflate`缩，例如：

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

### 按文件夹级别验证文件 {#invalidating-files-by-folder-level}

使用属 `/statfileslevel` 性根据缓存文件的路径使其失效：

* 调度程 `.stat`序在每个文件夹中从Docroot文件夹创建到您指定的级别的文件。 Docroot文件夹为0级。
* 通过触摸文件使文件失 `.stat` 效。 将文 `.stat` 件的上次修改日期与缓存文档的上次修改日期进行比较。 如果文件较新，将重 `.stat` 新获取文档。

* 当某个级别的文件失效 **时** ，将触 `.stat` 及从Docroot **到** 失效文件级别的所 `statsfilevel` 有文件，或者将触碰已配置的文件（较小的）。

   * 例如：如果将属性 `statfileslevel` 设置为6，并且文件在级别5处失效，则将触及从 `.stat` docroot到5的每个文件。 继续此示例，如果文件在级别7时失效，则每次。 `stat` 将触摸从docroot到6的文件(自此 `/statfileslevel = "6"`)。

仅无效文 **件路径中** 的资源受到影响。 请考虑以下示例：网站使用结构如 `/content/myWebsite/xx/.` 果设置 `statfileslevel` 为3，则会 `.stat`创建如下文件：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

当中的文件失 `/content/myWebsite/xx` 效时，将触 `.stat` 及从Docroot到Docroot的 `/content/myWebsite/xx`每个文件。 这种情况只适用于 `/content/myWebsite/xx` 而不适用于 `/content/myWebsite/yy` 或 `/content/anotherWebSite`。

>[!NOTE]
>
>通过发送其他标头可以防止失效 `CQ-Action-Scope:ResourceOnly`。 这可用于刷新特定资源而不使高速缓存的其他部分失效。 有关 [其他详](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) 细信 [息，请参阅此页和手动使调度程序缓存失效](https://helpx.adobe.com/experience-manager/dispatcher/using/page-invalidate.html) 。

>[!NOTE]
>
>如果为属性指定 `/statfileslevel` 值，则忽 `/statfile` 略该属性。

### 自动使缓存文件失效 {#automatically-invalidating-cached-files}

该属 `/invalidate` 性定义在更新内容时自动失效的文档。

如果自动失效，调度程序在内容更新后不会删除缓存文件，但会在下次请求文件时检查其有效性。 缓存中未自动失效的文档将保留在缓存中，直到内容更新显式删除它们。

HTML页面通常使用自动失效。 HTML页面通常包含指向其他页面的链接，因此很难确定内容更新是否影响页面。 要确保更新内容时所有相关页面都失效，请自动使所有HTML页面失效。 以下配置使所有HTML页失效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

有关全局属性的信息，请参 [阅设计全局属性的模式](#designing-patterns-for-glob-properties)。

此配置在激活/content/geometrixx/en时导致以下活动:

* 所有带图案的文件。*已从/content/geometrixx/文件夹中删除。
* 已删除/content/geometrixx/cn/_jcr_content文件夹。
* 不会立即删除与/invalidate配置匹配的所有其他文件。 下一个请求发生时，这些文件将被删除。 在我们的示例中，/content/geometrixx.html未被删除，将在请求/content/geometrixx.html时将其删除。

如果您优惠自动生成的PDF和ZIP文件进行下载，则可能还必须自动使这些文件失效。 配置示例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM与Adobe Analytics的集成在您网站的analytics.sitecatalyst.js文件中提供配置数据。 随Dispatcher提供的dispatcher.any文件示例包含此文件的以下失效规则：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自定义失效脚本 {#using-custom-invalidation-scripts}

/invalidateHandler属性允许您定义一个脚本，该脚本被调用用于Dispatcher接收的每个失效请求。

它使用以下参数调用：

* 处理\
   无效的内容路径
* 操作\
   复制操作（例如，激活、取消激活）
* 操作范围\
   复制操作的范围(空，除非发送标头，否 `CQ-Action-Scope: ResourceOnly` 则请参 [阅AEM的缓存页面失效](page-invalidate.md) ，了解详细信息)

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

有关全局属性的信息，请参 [阅设计全局属性的模式](#designing-patterns-for-glob-properties)。

>[!CAUTION]
>
>建议您定义/allowedClients。
>
>如果不执行此操作，任何客户端都可发出清除缓存的调用；如果重复执行此操作，可能会严重影响站点性能。

### 忽略URL参数 {#ignoring-url-parameters}

该部 `ignoreUrlParams` 分定义在确定页面是缓存还是从缓存传送时忽略的URL参数：

* 当请求URL包含所有被忽略的参数时，将缓存页面。
* 当请求URL包含一个或多个不被忽略的参数时，不会缓存页面。

当忽略某个页面的参数时，将在首次请求该页面时缓存该页面。 对页面的后续请求将提供到缓存的页面，而不管请求中的参数值如何。

要指定忽略哪些参数，请向属性添加全局 `ignoreUrlParams` 规则：

* 要忽略参数，请创建允许该参数的全局属性。
* 要阻止对页面进行缓存，请创建拒绝参数的glob属性。

以下示例导致Dispatcher忽略“q”参数，以便缓存包含q参数的请求URL:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

使用示例 `ignoreUrlParams` 值，以下HTTP请求会导致页面被缓存，因为 `q` 该参数被忽略：

```xml
GET /mypage.html?q=5
```

使用示 `ignoreUrlParams` 例值，以下HTTP请求会导致页面 **不被缓** 存，因 `p` 为参数不会被忽略：

```xml
GET /mypage.html?q=5&p=4
```

有关全局属性的信息，请参 [阅设计全局属性的模式](#designing-patterns-for-glob-properties)。

### 缓存HTTP响应头 {#caching-http-response-headers}

>[!NOTE]
>
>此功能适用于 **4.1.11** 版调度程序。

该 `/headers` 属性允许您定义将由调度程序缓存的HTTP头类型。 在对未缓存资源的第一个请求中，与配置值之一匹配的所有标头（请参见下面的配置示例）都存储在缓存文件旁的单独文件中。 在对缓存资源的后续请求中，存储的标头被添加到响应中。

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
>如果需要Dispatcher从AEM存储和传送ETag响应头，请执行以下操作：
>
>* 在节中添加标题 `/cache/headers`名称。
>* 在与Dispatcher相 [关的部分](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) ，添加以下Apache指令：

>



```xml
FileETag none
```

### 调度程序缓存文件权限 {#dispatcher-cache-file-permissions}

该属 `mode` 性指定将哪些文件权限应用于缓存中的新目录和文件。 此设置受调用过 `umask` 程的限制。 它是从以下一个或多个值之和构建的八进制数：

* 0400允许由所有者读取。
* 0200允许由所有者写入。
* 0100允许所有者在目录中搜索。
* 0040允许由组成员读取。
* 0020允许由组成员写入。
* 0010允许组成员在目录中搜索。
* 0004允许他人阅读。
* 0002允许他人写入。
* 0001允许其他人在目录中搜索。

默认值为0755，它允许所有者读取、写入或搜索，以及组和其他人读取或搜索。

### 限制。stat文件触控 {#throttling-stat-file-touching}

使用默认属 `/invalidate` 性，每个激活都会有效地使所 `.html` 有文件(当其路径与节 `/invalidate` 匹配时)无效。 在具有大量流量的网站上，多个后续激活将增加后端的cpu负载。 在这种情况下，最好“限制”文件触 `.stat` 摸以使网站保持响应。 您可以使用属性来执行此 `/gracePeriod` 操作。

该属 `/gracePeriod` 性定义在上次出现激活后，过时的自动失效资源仍可从缓存服务的秒数。 该属性可用在设置中，在该设置中，批激活会反复使整个缓存失效。 建议的值为2秒。

有关其他详细信息，请阅读 `/invalidate` 上面 `/statfileslevel`的和部分。

### 配置基于时间的缓存失效- /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

如果设置 `/enableTTL` ，属性将评估来自后端的响应头，如果它们包含 `Cache-Control``Expires` max-age或date，则会创建缓存文件旁的辅助空文件，修改时间等于到期日期。 当请求缓存的文件超过修改时间时，会自动从后端重新请求该文件。

>[!NOTE]
>
>此功能在Dispatcher的4. **1.11或更高版** 本中可用。

## 配置负载平衡- /statistics {#configuring-load-balancing-statistics}

该部 `/statistics` 分定义一类别文件，Dispatcher为这些文件对每个渲染的响应性进行评分。 调度程序使用分数来确定发送请求的渲染。

您创建的每个类别定义全局模式。 调度程序将所请求内容的URI与这些模式进行比较，以确定所请求内容的类别:

* 类别的顺序决定了它们与URI比较的顺序。
* 与URI匹配的第一个类别模式是文件的类别。 不再评估类别模式。

调度程序最多支持8个统计类别。 如果定义的类别超过8个，则只使用前8个。

**渲染选择**

每次Dispatcher需要渲染页面时，它都使用以下算法来选择渲染：

1. 如果请求包含Cookie中的渲染名 `renderid` 称，则Dispatcher将使用该渲染。
1. 如果请求中不包 `renderid` 含cookie，则Dispatcher将比较呈现统计信息：

   1. 调度程序确定请求URI的类别。
   1. 调度程序确定哪个渲染具有该类别的最低响应得分，并选择该渲染。

1. 如果尚未选择任何渲染，请在列表中使用第一个渲染。

渲染器类别的得分基于以前的响应时间以及调度程序尝试的以前失败和成功连接。 对于每次尝试，将更新所请求URI类别的得分。

>[!NOTE]
>
>如果不使用负载平衡，可忽略此部分。

### 定义统计类别 {#defining-statistics-categories}

为每种类型的类别定义一个文档，您要为其保留用于渲染选择的统计信息。 /statistics部分包含/类别部分。 要定义类别，请在/类别节下添加一行，其格式如下：

`/name { /glob "pattern"}`

类别 `name` 必须是农场特有的。 有关 `pattern` 信息，请参 [阅全局属性的设计模式](#designing-patterns-for-glob-properties) 。

要确定URI的类别，调度程序会将URI与每个类别模式进行比较，直到找到匹配项。 调度程序从列表中的第一个类别开始，并按顺序继续。 因此，首先放置具有更具体模式的类别。

例如，默认dispatcher.any文件是调度程序，它定义HTML类别和其他类别。 HTML类别更加具体，因此它首先显示：

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

以下示例还包括搜索页面的类别:

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

该属 `/unavailablePenalty` 性设置当与渲染器的连接失败时应用于渲染统计信息的时间（以秒计）。 调度程序将时间添加到与请求的URI匹配的统计类别。

例如，当无法建立到指定主机名／端口的TCP/IP连接时，会应用惩罚，因为AEM未运行（且未侦听），或者由于网络相关问题。

该 `/unavailablePenalty` 属性是该部分的直接子 `/farm` 项(该部分的兄弟 `/statistics` 项)。

如果不 `/unavailablePenalty` 存在任何属性，则使用值“1”。

```xml
/unavailablePenalty "1"
```

## 识别粘滞连接文件夹- /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

该属 `/stickyConnectionsFor` 性定义一个包含粘滞文档的文件夹；这将通过URL进行访问。 调度程序将所有位于此文件夹中的请求从单个用户发送到同一渲染实例。 粘滞连接可确保会话数据对所有文档都存在且一致。 此机制使用 `renderid` cookie。

以下示例定义到/products文件夹的粘滞连接：

```xml
/stickyConnectionsFor "/products"
```

当页面由来自多个内容节点的内容组成时，请包 `/paths` 含列表内容路径的属性。 例如，页面包含来自、 `/content/image`和 `/content/video`的内 `/var/files/pdfs`容。 以下配置为页面上的所有内容启用粘贴连接：

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

启用粘性连接后，调度程序模块将设置 `renderid` cookie。 此Cookie没有标记， `httponly` 应添加该添加该标记以增强安全性。 可通过在配置文件的节 `httpOnly` 点中设 `/stickyConnections` 置属性来 `dispatcher.any` 执行此操作。 属性的值（0或1）定义cookie是否附 `renderid` 加了属 `HttpOnly` 性。 默认值为0，表示不会添加属性。

有关该标志的其 `httponly` 他信息，请 [阅读此页](https://www.owasp.org/index.php/HttpOnly)。

### 安全 {#secure}

启用粘性连接后，调度程序模块将设置 `renderid` cookie。 此Cookie没有安全标 **志** ，应该添加该标志以增强安全性。 可通过在配置文件的节 `secure` 点中设 `/stickyConnections` 置属性来 `dispatcher.any` 执行此操作。 属性的值（0或1）定义cookie是否附 `renderid` 加了属 `secure` 性。 默认值为0，这意味着如果传入的请求是安全 **的** ，将添加属性。 如果该值设置为1，则无论传入请求是否安全，都将添加安全标志。

## 处理渲染连接错误 {#handling-render-connection-errors}

当渲染服务器返回500错误或不可用时配置调度程序行为。

### 指定运行状况检查页 {#specifying-a-health-check-page}

使用属 `/health_check` 性指定在发生500状态代码时检查的URL。 如果此页还返回500个状态代码，则该实例被视为不可用，在重试之前，将对渲染应用可配置的 `/unavailablePenalty`时间惩罚()。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定页面重试延迟 {#specifying-the-page-retry-delay}

/属性 `retryDelay` 设置调度程序在场渲染的连接尝试轮次之间等待的时间（以秒为单位）。 对于每轮，调度程序尝试连接到渲染器的最大次数是场中的渲染次数。

如果未显式定义， `"1"` 调度 `/retryDelay` 程序将使用值。 默认值在大多数情况下都适用。

```xml
/retryDelay "1"
```

### 配置重试数 {#configuring-the-number-of-retries}

该属 `/numberOfRetries` 性设置调度程序在渲染时执行的最大连接尝试轮数。 如果Dispatcher在此数量的重试后无法成功连接到渲染器，则Dispatcher返回失败的响应。

对于每轮，调度程序尝试连接到渲染器的最大次数是场中的渲染次数。 因此，调度程序尝试连接的最大次数 `/numberOfRetries`为()x（渲染次数）。

如果未显式定义该值，则默认值为 **5**。

```xml
/numberOfRetries "5"
```

### 使用故障转移机制 {#using-the-failover-mechanism}

在原始请求失败时，启用调度程序群上的故障转移机制以向不同的渲染器重新发送请求。 启用故障转移后，调度程序具有以下行为：

* 当对渲染器的请求返回HTTP状态503(UNAVAILABLE)时，调度程序会将请求发送到其他渲染器。
* 当对渲染器的请求返回HTTP状态50x（不是503）时，调度程序会发送为该属性配置的页面的请 `health_check` 求。

   * 如果运行状况检查返回500(INTERNAL_SERVER_ERROR)，则调度程序会将原始请求发送到其他渲染器。
   * 如果运行状况检查返回HTTP状态200，则调度程序将初始HTTP 500错误返回给客户端。

要启用故障转移，请将以下行添加到场（或网站）:

```xml
/failover "1" 
```

>[!NOTE]
>
>要重试包含正文的HTTP请求，Dispatcher会先向渲染器发 `Expect: 100-continue` 送请求标头，然后再假脱机实际内容。 带CQSE的CQ 5.5随后立即回答为100（继续）或错误代码。 其他servlet容器也应支持此。

## 忽略中断错误- /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此选项。 您只需在看到以下日志消息时使用它：
>
>`Error while reading response: Interrupted system call`

如果系统调用的对象位于通过NFS `EINTR` 访问的远程系统上，则任何面向文件系统的系统调用都可以中断。 这些系统调用是否可以超时或中断取决于基础文件系统在本地计算机上的安装方式。

如果实例具有此类配置，并且日志包含以下消息，请使用/ignoreEINTR参数：

`Error while reading response: Interrupted system call`

在内部，调度程序使用循环从远程服务器(即AEM)读取响应，该循环可以表示为：

```
while (response not finished) {  
read more data  
}
```

当在“”部分中发 `EINTR` 生时，可 `read more data`以生成这样的消息，该消息是在接收到任何数据之前由信号的接收引起的。

要忽略此类中断，可以将以下参数添加 `dispatcher.any` 到(之 `/farms`前):

`/ignoreEINTR "1"`

设置 `/ignoreEINTR` 为 `"1"` 会导致Dispatcher继续尝试读取数据，直到读取完整响应。 默认值为0，并取消激活该选项。

## 全局属性的设计模式 {#designing-patterns-for-glob-properties}

调度程序配置文件中的几个部分使 `glob` 用属性作为客户端请求的选择标准。 全局属性的值是调度程序与请求的一个方面进行比较的模式，如所请求资源的路径或客户端的IP地址。 例如，部分中的项 `/filter` 使用全局模式来标识Dispatcher处理或拒绝的页面的路径。

全局值可以包括通配符和字母数字字符以定义模式。

| 通配符 | 描述 | 示例 |
|--- |--- |--- |
| `*` | 匹配字符串中任意字符的零个或多个连续实例。 匹配的最终字符由以下任一情况确定： <br/>字符串中的字符与模式中的下一个字符匹配，并且模式字符具有以下特征：<br/><ul><li>不是*</li><li>不是？</li><li>文本字符（包括空格）或字符类。</li><li>到达图案的末尾。</li></ul>在字符类中，字符将字面地解释。 | `*/geo*` 匹配节点和节 `/content/geometrixx` 点下的任何 `/content/geometrixx-outdoors` 页面。 以下HTTP请求与全局模式匹配： <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>匹配节点下的任何 `/content/geometrixx-outdoors` 页面。 例如，以下HTTP请求与全局模式匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 匹配任何单个字符。 使用外部字符类。 在字符类中，该字符将字面解释。 | `*outdoors/??/*`<br/> 匹配geometrixx-outdoors站点中任何语言的页面。 例如，以下HTTP请求与全局模式匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>以下请求与全局模式不匹配： <br/><ul><li>&quot;GET/content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | 取消字符类的开头和结尾的标记。 字符类可以包括一个或多个字符范围和单个字符。<br/>如果目标字符与字符类中的任意字符或在定义的范围内匹配，则会发生匹配。<br/>如果未包括右括号，则图案不产生匹配项。 | `*[o]men.html*`<br/> 匹配以下HTTP请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示字符范围。 用于字符类。  在字符类之外，将字面解释此字符。 | `*[m-p]men.html*` 匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定后面的字符或字符类。 仅用于否定字符类中的字符和字符范围。 相当于 `^ wildcard`。 <br/>在字符类之外，将字面解释此字符。 | `*[!o]men.html*`<br/> 匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>与以下HTTP请求不匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定后面的字符或字符范围。 仅用于否定字符类中的字符和字符范围。 等效于通配 `!` 符字符。 <br/>在字符类之外，将字面解释此字符。 | 通配符的示 `!` 例应用，将示例模式 `!` 中的字符替换为 `^` 字符。 |


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

如果使用 **Apache** Web服务器，则可对旋转和／或管道日志使用标准功能。 例如，使用管道日志：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

这将自动旋转：

* 调度程序日志文件；扩展中包含时间戳(logs/dispatcher.log%Y%m%d)。
* 每周（60 x 60 x 24 x 7 = 604800秒）。

请参见Log Rotation和Pinual Logs上的Apache Web服务器文档；例如 [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html)。

>[!NOTE]
>
>安装后，默认日志级别较高（即级别3 =调试），因此调度程序会记录所有错误和警告。 这在最初阶段非常有用。
>
>但是，这需要额外的资源，因此当Dispatcher根据您的要 *求顺利工作时*，您可以（应）降低日志级别。

### 跟踪日志记录 {#trace-logging}

在对Dispatcher的其他增强功能中，版本4.2.0还引入了跟踪日志记录。

这比调试日志记录更高，日志中显示其他信息。 它添加了以下记录：

* 转发报头的值；
* 正应用于特定操作的规则。

可通过将日志级别设置为Web服务器中，启 `4` 用跟踪日志记录。

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

当请求与阻止规则匹配的文件时记录事件:

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## 确认基本操作 {#confirming-basic-operation}

要确认Web服务器、调度程序和AEM实例的基本操作和交互，可以执行以下步骤：

1. 将设置 `loglevel` 为 `3`。

1. 开始Web服务器；这也会开始调度程序。
1. 开始AEM实例。
1. 检查Web服务器和调度程序的日志和错误文件。\
   根据Web服务器，您应该看到如下消息：\
   `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`\
   和:\
   `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 通过Web服务器浏览网站。 确认内容是按需要显示的。\
   例如，在AEM在端口上运行的本地安装中，以及在访问网 `4502` 站控制台时使 `80` 用以下两种方式运行的Web服务器：\
   ` https://localhost:4502/libs/wcm/core/content/siteadmin.html  
https://localhost:80/libs/wcm/core/content/siteadmin.html  
`结果应一致。 使用相同机制确认对其他页面的访问。

1. 检查是否已填写缓存目录。
1. 激活页面以检查缓存是否正在正确刷新。
1. 如果一切正常运行，您可以将 `loglevel` 调整 `0`为

## 使用多个 Dispatcher {#using-multiple-dispatchers}

在复杂设置中，您可以使用多个 Dispatcher。例如，您可以使用：

* 一个 Dispatcher 用于在内联网上发布网站
* 第二个 Dispatcher，通过不同的地址和不同的安全设置，在内联网上发布相同的内容。

在这种情况下，请确保每个请求只通过一个 Dispatcher。一个 Dispatcher 不能处理来自另一个 Dispatcher 的请求。因此，请确保两个 Dispatcher 都能直接访问 AEM 网站。

## 调试 {#debugging}

在向请求中添 `X-Dispatcher-Info` 加标题时，调度程序会回答目标是已缓存、从缓存中返回还是根本无法缓存。 响应标题 `X-Cache-Info` 以可读形式包含此信息。 您可以使用这些响应标头来调试与由调度程序缓存的响应有关的问题。

默认情况下，此功能未启用，因此要包含响应头， `X-Cache-Info` 场必须包含以下条目：

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

此外， `X-Dispatcher-Info` 标题不需要值，但如果您用 `curl` 于测试，则必须提供值才能发送标题，例如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/we-retail/us/en.html
```

以下是包含将返回的响应标头 `X-Dispatcher-Info` 的列表:

* **缓存**\
   目标文件包含在缓存中，调度程序已确定传送该文件是有效的。
* **缓存**\
   目标文件未包含在缓存中，调度程序已确定缓存输出并传送输出是有效的。
* **缓存：stat文件是较新**&#x200B;的目标文件包含在缓存中，但是，它由较新的stat文件失效。 调度程序将删除目标文件，从输出中重新创建并传送它。
* **无法缓存：无文档**&#x200B;根此场的配置不包含文档根（配置元素） 
`cache.docroot`).
* **无法缓存：缓存文件路径太长**\
   目标文件(文档根文件和URL文件的串连)超过系统上最长的可能文件名。
* **无法缓存：临时文件路径太长**\
   临时文件名模板超出系统上最长的可能文件名。 调度程序首先创建临时文件，然后实际创建或覆盖缓存的文件。 临时文件名是目标文件名，并附加 `_YYYYXXXXXX` 字符，将替换 `Y` 和 `X` 以创建唯一名称。
* **无法缓存：请求URL没有扩展名**\
   请求URL没有扩展名，或者文件扩展名后面有一个路径，例如： `/test.html/a/path`.
* **无法缓存：请求不是GET或HEAD** HTTP方法既不是GET也不是HEAD。 调度程序假定输出将包含不应缓存的动态数据。
* **无法缓存：请求包含查询字符串**\
   该请求包含一个查询字符串。 调度程序假定输出取决于给定的查询字符串，因此不进行缓存。
* **无法缓存：会话管理器未验证**\
   场的缓存受会话管理器(配置包含节 `sessionmanagement` 点)控制，请求不包含相应的身份验证信息。
* **无法缓存：请求包含授权**\
   场不允许缓存输出(), `allowAuthorized 0`并且请求包含身份验证信息。
* **无法缓存：目标是目录**\
   目标文件是目录。 这可能指出某些概念性错误，即URL和某些子URL都包含可缓存输出，例如，如果请求是第一个并包含可缓存输出，调度程序将无法缓存后续请求的输出 `/test.html/a/file.ext``/test.html`。
* **无法缓存：请求URL具有尾随斜杠**\
   请求URL具有尾随斜杠。
* **无法缓存：缓存规则中未包含请求URL**\
   场的缓存规则明确拒绝缓存某些请求URL的输出。
* **无法缓存：授权检查器拒绝访问**\
   场的授权检查器拒绝访问缓存的文件。
* **无法缓存：会话无效**&#x200B;场的缓存受会话管理器（配置包含节点）的管 `sessionmanagement` 理，并且用户的会话无效或不再有效。
* **无法缓存：响应包`no_cache `**&#x200B;含远程服务器返回 
`Dispatcher: no_cache` header，禁止调度程序缓存输出。
* **无法缓存：响应内容长度**&#x200B;为零响应的内容长度为零；调度程序将不创建零长度文件。
