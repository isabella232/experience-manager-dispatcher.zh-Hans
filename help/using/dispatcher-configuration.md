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
source-git-commit: 4f1e3740c7eb91023b819ffed0bb5d0b432002be

---


# Configuring Dispatcher{#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher版本独立于AEM。如果您遵循了一个指向Dispatcher文档的链接，则可能已重定向到该页面，该链接嵌入到AEM的先前版本的文档中。

以下部分介绍了如何配置Dispatcher的各个方面。

## Support for IPv4 and IPv6 {#support-for-ipv-and-ipv}

AEM和Dispatcher的所有元素都可以安装在IPv和IPv网络中。See [IPV4 and IPV6](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html#AdditionalPlatformNotes).

## Dispatcher Configuration Files {#dispatcher-configuration-files}

By default the Dispatcher configuration is stored in the `dispatcher.any` text file, though you can change the name and location of this file during installation.

配置文件包含一系列用于控制Dispatcher行为的单值或多值属性：

* Property names are prefixed with a forward slash `/`.
* Multi-valued properties enclose child items using braces `{ }`.

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

For example, if the files `farm_1.any` through to `farm_5.any` contain the configuration of farms one to five, you can include them as follows:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Using Environment Variables {#using-environment-variables}

您可以在调度程序.任何文件中使用字符串值属性中的环境变量，而不是对这些值进行硬编码。To include the value of an environment variable, use the format `${variable_name}`.

For example, if the dispatcher.any file is located in the same directory as the cache directory, the following value for the [docroot](dispatcher-configuration.md#main-pars-title-30) property can be used:

```xml
/docroot "${PWD}/cache"
```

As another example, if you create an environment variable named `PUBLISH_IP` that stores the hostname of the AEM publish instance, the following configuration of the [/renders](dispatcher-configuration.md#main-pars-127-25-0008) property can be used:

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## Naming the Dispatcher Instance {#naming-the-dispatcher-instance-name}

Use the `/name` property to specify a unique name to identify your Dispatcher instance. `/name` 该属性是配置结构中的顶级属性。

## Defining Farms {#defining-farms-farms}

`/farms` 该属性定义一组或多组调度程序行为，其中每个集合都与不同的网站或URL相关联。`/farms` 该属性可包括一个农场或多个农场：

* 当您希望Dispatcher以相同的方式处理所有网页或网站时使用单一农场。
* 当网站或不同网站的不同区域需要不同的调度程序行为时，创建多个农场。

`/farms` 该属性是配置结构中的顶级属性。To define a farm, add a child property to the `/farms` property. 使用可唯一标识调度程序实例中的农场的属性名称。

`/*farmname*` 该属性具有多个值，并且包含定义Dispatcher行为的其他属性：

* 农场应用的页面URL。
* 用于渲染文档的一个或多个服务URL(通常是AEM发布实例)。
* 用于对多个文档渲染器进行负载平衡的统计数据。
* 其他一些行为，如要缓存的文件和位置。

该值可能包含任何字母数字(a-z、0-9)字符。The following example shows the skeleton definition for two farms named `/daycom` and `/docsdaycom`:

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
>如果您使用多个渲染农场，则会对列表进行底部评估。This is particularly relevant when defining [Virtual Hosts](dispatcher-configuration.md#main-pars-117-15-0006) for your websites.

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

## Specify a Default Page (IIS Only) - /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>`/homepage`参数(仅IIS)不再有效。Instead, you should use the [IIS URL Rewrite Module](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>If you are using Apache, you should use the `mod_rewrite` module. See the Apache web site documentation for information about `mod_rewrite` (for example, [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). When using `mod_rewrite`, it is advisable to use the flag ** [&#39;passthrough|PT&#39; (pass through to next handler)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** to force the rewrite engine to set the `uri` field of the internal `request_rec` structure to the value of the `filename` field.

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

## Specifying the HTTP Headers to Pass Through {#specifying-the-http-headers-to-pass-through-clientheaders}

`/clientheaders` 该属性定义Dispatcher从客户端HTTP请求传递给渲染器(AEM实例)的HTTP头列表。

默认情况下，Dispatcher将标准HTTP头转发到AEM实例。在某些情况下，您可能希望转发其他标题或删除特定标题：

* 添加AEM实例在HTTP请求中所需的标题，如自定义标题。
* 删除仅与Web服务器相关的标题，如身份验证标题。

如果您自定义要传递的标题集，则必须指定一个完整的标题列表，包括通常包含的标题。

For example, a Dispatcher instance that handles page activation requests for publish instances requires the `PATH` header in the `/clientheaders` section. `PATH` 标题支持复制代理与调度程序之间的通信。

The following code is an example configuration for `/clientheaders`:

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

## Identifying Virtual Hosts {#identifying-virtual-hosts-virtualhosts}

`/virtualhosts` 该属性定义Dispatcher接受此农场的所有主机名/URI组合的列表。您可以使用星号(“*”)作为通配符。Values for the / `virtualhosts` property use the following format:

```xml
[scheme]host[uri][*]
```

* `scheme`：(可选)或 `https://``https://.`
* `host`：主机的名称或IP地址(如有必要)。(See [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
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

The following configuration handles *all* requests:

```xml
   /virtualhosts
    {
    "*"
    }
```

### Resolving the Virtual Host {#resolving-the-virtual-host}

When Dispatcher receives an HTTP or HTTPS request, it finds the virtual host value that best-matches the `host,` `uri`, and `scheme` headers of the request. Dispatcher evaluates the values in the `virtualhosts` properties in the following order:

* 调度程序从最低农场开始，并在调度程序.任何文件中向上行进。
* For each farm, Dispatcher begins with the topmost value in the `virtualhosts` property and progresses down the list of values.

Dispatcher以以下方式查找匹配的虚拟主机值：

* The first-encountered virtual host that matches all three of the `host`, the `scheme`, and the `uri` of the request is used.
* If no `virtualhosts` values has `scheme` and `uri` parts that both match the `scheme` and `uri` of the request, the first-encountered virtual host that matches the `host` of the request is used.
* If no `virtualhosts` values have a host part that matches the host of the request, the topmost virtual host of the topmost farm is used.

Therefore, you should place your default virtual host at the top of the `virtualhosts` property in the topmost farm of your dispatcher.any file.

### Example Virtual Host Resolution {#example-virtual-host-resolution}

The following example represents a snippet from a dispatcher.any file that defines two Dispatcher farms, and each farm defines a `virtualhosts` property.

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

## Enabling Secure Sessions - /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized`**必须** 在 `"0"``/cache` 部分中设置才能启用此功能。

创建一个安全会话以访问渲染农场，以便用户需要登录才能访问农场中的任何页面。登录后，用户可以访问农场中的所有页面。See [Creating a Closed User Group](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/cug.html#CreatingTheUserGroupToBeUsed) for information about using this feature with CUGs.

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

存储授权信息的HTTP头或cookie的名称。If you store the information in the http header, use `HTTP:<*header-name*>`. To store the information in a cookie, use `Cookie:<header-name>`. If you do not specify a value `HTTP:authorization` is used.

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

## Defining Page Renderers {#defining-page-renderers-renders}

/renders属性定义Dispatcher发送文档请求请求的URL。The following example `/renders` section identifies a single AEM instance for rendering:

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

### Renders options {#renders-options}

**/timeout**

指定访问AEM实例(以毫秒为单位)的连接超时。默认值为“0”，导致调度程序等待失败。

**/receiveTimeout**

指定允许响应的时间(以毫秒为单位)。默认值为“6000”，导致Dispatcher等待10分钟。设置为“0”可完全消除超时。\
如果在解析响应头时到达超时，则返回HTTP状态504(不良网关)。如果在读取响应主体时达到超时，Dispatcher将返回对客户端的不完整响应，但删除可能已编写的任何缓存文件。

**/ipv4**

Specifies whether Dispatcher uses the `getaddrinfo` function (for IPv6) or the `gethostbyname` function (for IPv4) for obtaining the IP address of the render. A value of 0 causes `getaddrinfo` to be used. A value of 1 causes `gethostbyname` to be used. 默认值为 0。

getaddrinfo函数返回IP地址列表。Dispatcher迭代地址列表，直到其建立TCP/IP连接。因此，当渲染主机名与多个IP地址关联时，如果主机为getaddrinfo函数返回一个始终处于同一顺序的IP地址列表，则IPv属性很重要。在这种情况下，您应使用gethostbyname函数，以便调度程序与调度程序连接的IP地址为randomized。

Amazon Elastic Load Dialization(ELB)是一种可响应getaddrinfo的服务，它具有一个潜在的IP地址有序列表。

**/secure**

If the `/secure` property has a value of &quot;1&quot; Dispatcher uses HTTPS to communicate with the AEM instance. For additional details, also see [Configuring Dispatcher to Use SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

With Dispatcher version **4.1.6**, you can configure the `/always-resolve` property as follows:

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

## Configuring Access to Content {#configuring-access-to-content-filter}

Use the `/filter` section to specify the HTTP requests that Dispatcher accepts. 所有其他请求都将返回到带有404错误代码的Web服务器(找不到页面)。If no `/filter` section exists, all requests are accepted.

**注意：** 始终拒绝 [对statfile](dispatcher-configuration.md#main-pars-title-28) 的请求。

>[!CAUTION]
>
>See the [Dispatcher Security Checklist](security-checklist.md) for further considerations when restricting access using Dispatcher. Also, read the [AEM Security Cheklist](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html) for additional security details regarding your AEM installation.

/filter部分由拒绝或允许根据HTTP请求的请求行部分中的模式访问内容的一系列规则组成。您应该为/filter部分使用一个有趣的策略：

* 首先，拒绝访问所有内容。
* 允许根据需要访问内容。

### Defining a Filter {#defining-a-filter}

`/filter` 部分中的每个项目都包含一个类型和一个模式，该模式与请求行或整个请求行的特定元素相匹配。每个过滤器都可以包含以下项目：

* **类型**：它 `/type` 指示是否允许或拒绝对与模式匹配的请求的访问。The value can be either `allow` or `deny`.

* **请求行的元素：** 包括 `/method``/url`、或 `/query``/protocol` 一个模式，用于根据HTTP请求的请求线部分的特定部分过滤请求。过滤请求行(而不是整个请求行上的元素)的元素是首选过滤器方法。

* **请求行的高级元素：** 从调度程序4.2.0开始，可使用四个新的过滤元素。These new elements are `/path`, `/selectors`, `/extension` and `/suffix` respectively. 包括一个或多个项目以进一步控制URL模式。

>[!NOTE]
>
>For more information about what part of the request line each of these elements references, see the [Sling URL Decomposition](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki page.

* **glob属性**： `/glob` 此属性用于与HTTP请求的整个请求行匹配。

>[!CAUTION]
>
>使用globs过滤已在Dispatcher中弃用。As such, you should avoid using globs in the `/filter` sections since it may lead to security issues. 因此，不是：

`/glob "* *.css *"`

您应使用

`/url "*.css"`

#### The request-line Part of HTTP Requests {#the-request-line-part-of-http-requests}

HTTP/1.1 defines the [request-line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) as follows:

*方法请求-URI HTTP-Version*&lt; CRLF&gt;

&lt; CRLF&gt;字符返回回车返回后跟换行源。以下示例是当客户端请求Geometrixx-Othors站点的en页面时接收的请求线：

GET/content/geometrixx-outdoors/en.htmlHTTP.1&lt; CRLF&gt;

您的模式必须考虑请求行和&lt; CRLF&gt;字符中的空格字符。

#### Double-quotes vs Single-quotes {#double-quotes-vs-single-quotes}

When creating your filter rules, use double quotation marks `"pattern"` for simple patterns. If you are using Dispatcher 4.2.0 or later and your pattern includes a regular expression, you must enclose the regex pattern `'(pattern1|pattern2)'` within single quotation marks.

#### Regular Expressions {#regular-expressions}

调度程序4.2.0之后，您可以在筛选模式中包含POSIX Extended正则表达式。

#### Troubleshooting Filters {#troubleshooting-filters}

If your filters are not triggering in the way you would expect, enable [Trace Logging](#trace-logging) on dispatcher so you can see which filter is intercepting the request.

#### Example Filter: Deny All {#example-filter-deny-all}

以下示例过滤器部分将导致Dispatcher拒绝所有文件的请求。您应拒绝访问所有文件，然后允许访问特定区域。

```xml
  /0001  { /glob "*" /type "deny" }
```

请求显式拒绝区域导致返回404错误代码(找不到页面)。

#### Example Filter: Deny Acess to Specific Areas {#example-filter-deny-acess-to-specific-areas}

过滤器还允许您拒绝访问各种元素(例如ASP页面和发布实例中的敏感区域)。以下过滤器拒绝访问ASP页面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Example Filter: Enable POST Requests {#example-filter-enable-post-requests}

以下示例过滤器允许POST方法提交表单数据：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### Example Filter: Allow Access to the Workflow Console {#example-filter-allow-access-to-the-workflow-console}

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

#### Example filter: Using Regular Expressions {#example-filter-using-regular-expressions}

此过滤器使用正则表达式在非公开内容目录中启用扩展，在单引号之间定义此过滤器：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Example filter: Filter Additional Elements of a Request URL {#example-filter-filter-additional-elements-of-a-request-url}

Below is a rule example that blocks content grabbing from the `/content` path and its subtree, using filters for path, selectors and extensions:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### Example /filter section {#example-filter-section}

配置Dispatcher时，您应尽可能限制外部访问。以下示例为外部访客提供了最小的访问权限：

* `/content`
* 各种内容，如设计和客户端库；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

After you create filters, [test page access](dispatcher-configuration.md#main-pars-title-19) to ensure your AEM instance is secure.

The following /filter section of the dispatcher.any file can be used as a basis in your [Dispatcher configuration](dispatcher-configuration.md) file.

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
>与Apache一起使用时，根据调度程序模块的dispatcherUseProcesseDurl属性设计过滤器URL模式。(See [Apache Web Server - Configure your Apache Web Server for Dispatcher](dispatcher-install.md#main-pars-55-35-1022).)

>[!NOTE]
>
>关于Dynamic Media的过滤器0030和0031适用于AEM6.0及更高版本。

如果您选择扩展访问权限，请考虑以下建议：

* External access to `/admin` should always be *completely* disabled if you are using CQ version 5.4 or an earlier version.

* Care must be taken when allowing access to files in `/libs`. 可以单独访问。
* 拒绝访问复制配置，因此无法看到该配置：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒绝访问Google Gadgets反向代理：

   * `/libs/opensocial/proxy*`

Depending on your installation, there might be additional resources under `/libs`, `/apps` or elsewhere, that must be made available. You can use the `access.log` file as one method of determining resources that are being accessed externally.

>[!CAUTION]
>
>访问控制台和目录可能会对生产环境带来安全风险。除非您有明确的选择，否则应取消激活(注释掉)。

>[!CAUTION]
>
>If you are [using reports in a publish environment](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/reporting.html#UsingReportsinaPublishEnvironment) you should configure Dispatcher to deny access to `/etc/reports` for external visitors.

### Restricting Query Strings {#restricting-query-strings}

Since Dispatcher version 4.1.5, use the `/filter` section to restrict query strings. It is highly recommended to explicitly allow query strings and exclude generic allowance through `allow` filter elements.

A single entry can have either *glob* or some combination of *method*,*url*,*query* and *version* but not both. The following example allows the `a=*` query string and denies all other query strings for URLs that resolve to the `/etc` node:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>If a rule contains a `/query`, it will only match requests that contain a query string and match the provided query pattern.
>
>In above example, if requests to `/etc` that have no query string should be allowed as well, the following rules would be required:


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Testing Dispatcher Security {#testing-dispatcher-security}

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
* /content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)
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

## Enabling Access to Vanity URLs {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

配置Dispatcher以允许访问为CQ或AEM页面配置的虚URL。

启用虚URL访问时，调度程序定期调用渲染实例上运行的服务以获取虚URL列表。调度程序将此列表存储在本地文件中。`/filter` 当由于章节中的过滤器而拒绝页面请求被拒绝时，Dispatcher会查阅虚URL列表。如果列表中有被拒绝的URL，Dispatcher允许访问虚URL。

To enable access to vanity URLs, add a `/vanity_urls` section to the `/farms` section, similar to the following example:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls` 该部分包含以下属性：

* `/url`：在渲染实例上运行的虚URL服务的路径。The value of this property must be `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`：Dispatcher存储虚URL列表的本地文件的路径。确保Dispatcher具有对此文件的写入权限。
* `/delay`：(秒)调用虚URL服务之间的时间。

>[!NOTE]
>
>If your render is an instance of AEM you must install the [VanityURLS-Components](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq600/component/vanityurls-components) package to install the vanity URL service. (See [Signing In to Package Share](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/package-manager.html#SigningIntoPackageShare).)

使用以下过程可访问虚URL。

1. 如果渲染服务是AEM实例，请在发布实例上安装com. adobe. granite. dispatcher. vanityurl. content包(请参阅上面的备注)。
1. For each vanity URL that you have configured for an AEM or CQ page, ensure that the ` [/filter](dispatcher-configuration.md#main-pars_134_32_0009)` configuration denies the URL. 如有必要，请添加一个拒绝URL的过滤器。
1. Add the `/vanity_urls` section below `/farms`.
1. 重新启动Apache Web服务器。

## Forwarding Syndication Requests - /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

聚合请求通常仅适用于调度程序，因此默认情况下，它们不会发送到呈示器(例如AEM实例)。

如有必要，请将“/propagateSyndPost”属性设置为“1”以转发到调度程序的请求请求。如果设置了此设置，则必须确保在筛选器部分中不拒绝POST请求。

## Configuring the Dispatcher Cache - /cache {#configuring-the-dispatcher-cache-cache}

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
>For permission-sensitive caching, read [Caching Secured Content](permissions-cache.md).

### Specifying the Cache Directory {#specifying-the-cache-directory}

`/docroot` 该属性标识缓存缓存文件的目录。

>[!NOTE]
>
>该值必须与Web服务器的文档根完全相同，以便调度程序和Web服务器处理相同的文件。\
>Web服务器负责在使用调度程序缓存文件时提供正确的状态代码，这也说明它也很重要。

如果您使用多个农场，每个农场必须使用不同的文档根。

### Naming the Statfile {#naming-the-statfile}

`/statfile` 该属性标识要用作statfile的文件。Dispatcher使用此文件注册最新内容更新的时间。statfile可以是Web服务器上的任何文件。

statfile没有内容。更新内容时，Dispatcher会更新时间戳。默认的stalfile名为. stat并存储在docroot中。调度程序阻止对statfile的访问。

>[!NOTE]
>
>If `/statfileslevel` is configured, Dispatcher ignores the `/statfile` property and uses .stat as the name.

### Serving Stale Documents When Errors Occur {#serving-stale-documents-when-errors-occur}

`/serveStaleOnError` 该属性控制当渲染服务器返回错误时，Dispatcher是否返回失效的文档。默认情况下，在触摸缓存的内容并使缓存的内容失效时，调度程序在下次请求缓存的内容时删除缓存内容。

If `/serveStaleOnError` is set to &quot;1&quot;, Dispatcher does not delete invalidated content from the cache unless the render server returns a successful response. AEM或连接超时中的xx响应导致Dispatcher服务于过期的内容并响应，而HTTP状态为111(重新验证失败)。

### Caching When Authentication is Used {#caching-when-authentication-is-used}

`/allowAuthorized` 该属性控制是否缓存包含下列任何身份验证信息的请求：

* `authorization` 标题。
* A cookie named `authorization`.
* A cookie named `login-token`.

默认情况下，不会缓存包含此身份验证信息的请求，因为在将缓存的文档返回给客户端时，不会执行身份验证。此配置可防止Dispatcher向没有必要权限的用户提供缓存文档。

但是，如果您的要求允许缓存已验证的文档，请将其设置为一个：

`/allowAuthorized "1"`

>[!NOTE]
>
>To enable session management (using the `/sessionmanagement` property), the `/allowAuthorized` property must be set to `"0"`.

### Specifying the Documents to Cache {#specifying-the-documents-to-cache}

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
>GET或HEAD(对于HTTP头)方法由Dispatcher进行缓存。For additional information on response header caching, see the [Caching HTTP Response Headers](dispatcher-configuration.md#caching-http-response-headers) section.

Each item in the /rules property includes a [glob](#designing-patterns-for-glob-properties) pattern and a type:

* glob模式用于匹配文档的路径。
* 类型指示是否缓存与glob模式匹配的文档。该值可以是允许的(用于缓存文档)或拒绝(以始终呈现文档)。

如果没有动态页面(超出以上规则已排除的页面除外)，则可以配置Dispatcher以缓存所有内容。此处的规则部分如下所示：

```xml
/rules
  { 
    /0000  {  /glob "*"   /type "allow" }
  }
```

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

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

在Apache Web服务器上，您可以压缩缓存的文档。压缩允许Apache在客户端请求时以压缩形式返回文档。Compression is done automatically by enabling the Apache module `mod_deflate`, for example:

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

### Invalidating Files by Folder Level {#invalidating-files-by-folder-level}

Use the `/statfileslevel` property to invalidate cached files according to their path:

* Dispatcher creates `.stat`files in each folder from the docroot folder to the level that you specify. docroot文件夹为级别0。
* Files are invalidated by touching the `.stat` file. `.stat` 将文件的上次修改日期与缓存文档的上次修改日期进行比较。`.stat` 如果文件较新，则重新访存文档。

* When a file located at a certain level is invalidated then **all** `.stat` files from the docroot **to** the level of the invalidated file or the configured `statsfilevel` (whichever is smaller) will be touched.

   * For example: if you set the `statfileslevel` property to 6 and a file is invalidated at level 5 then every `.stat` file from docroot to 5 will be touched. 继续执行此示例，如果文件在级别为7，则为无效。`stat` 从docroot到6的文件(因为 `/statfileslevel = "6"`)。

只有资源**沿着路径**才会影响无效文件。Consider the following example: a website uses the structure `/content/myWebsite/xx/.` If you set `statfileslevel` as 3, a `.stat`file is created as follows:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

When a file in `/content/myWebsite/xx` is invalidated then every `.stat` file from docroot down to `/content/myWebsite/xx`is touched. This would be the case only for `/content/myWebsite/xx` and not for example `/content/myWebsite/yy` or `/content/anotherWebSite`.

>[!NOTE]
>
>Invalidation can be prevented by sending an additional Header `CQ-Action-Scope:ResourceOnly`. 这可用于刷新特定资源，而不会使缓存的其他部分失效。See [this page](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) and [Manually Invalidating the Dispatcher Cache](https://helpx.adobe.com/experience-manager/dispatcher/using/page-invalidate.html) for additional details.

>[!NOTE]
>
>If you specify a value for the `/statfileslevel` property, the `/statfile` property is ignored.

### Automatically Invalidating Cached Files {#automatically-invalidating-cached-files}

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

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

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

### Using custom invalidation scripts {#using-custom-invalidation-scripts}

使用Details属性可定义为Dispatcher接收的每个无效请求调用的脚本。

使用以下参数调用它：

* 处理\
   无效的内容路径
* 操作\
   复制操作(例如激活、取消激活)
* 操作范围\
   The replication Action&#39;s Scope (empty, unless a header of `CQ-Action-Scope: ResourceOnly` is sent, see [Invalidating Cached Pages from AEM](page-invalidate.md) for details)

这可用于覆盖许多不同的用例，如使其他应用程序特定缓存失效，或处理页面的外部URL和其在docroot中的外部URL与内容路径不匹配的情况。

下面的示例脚本将每个无效请求记录到一个文件。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### sample invalidation handler script {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### Limiting the Clients That Can Flush the Cache {#limiting-the-clients-that-can-flush-the-cache}

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

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>建议您定义/allowedClients。
>
>如果未完成此操作，则任何客户端都可以发出清除缓存的调用；如果重复执行此操作，则会严重影响网站性能。

### Ignoring URL Parameters {#ignoring-url-parameters}

`ignoreUrlParams` 此部分定义在确定页面是否缓存或从缓存中传递页面时忽略哪些URL参数：

* 当请求URL包含所有被忽略的参数时，页面将缓存。
* 当请求URL包含一个或多个不被忽略的参数时，页面不会缓存。

当页面被忽略某个参数时，第一次请求该页面时将缓存该页面。对页面的后续请求将被缓存，而不管请求中参数的值如何。

To specify which parameters are ignored, add glob rules to the `ignoreUrlParams` property:

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

Using the example `ignoreUrlParams` value, the following HTTP request causes the page to be cached because the `q` parameter is ignored:

```xml
GET /mypage.html?q=5
```

Using the example `ignoreUrlParams` value, the following HTTP request causes the page to **not** be cached because the `p` parameter is not ignored:

```xml
GET /mypage.html?q=5&p=4
```

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

### Caching HTTP Response Headers {#caching-http-response-headers}

>[!NOTE]
>
>This feature is avaiable with version **4.1.11** of the Dispatcher.

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
>另外，请注意，不允许文件全局字符。For more details, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>如果您需要Dispatcher来存储和提供AEM中的eTag响应标题，请执行以下操作：
>
>* Add the header name in the `/cache/headers`section.
>* Add the following [Apache directive](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) in the Dispatcher related section:
>



```xml
FileETag none
```

### Dispatcher Cache File Permissions {#dispatcher-cache-file-permissions}

`mode` 该属性指定了哪些文件权限应用于缓存中的新目录和文件。This setting is restricted by the `umask` of the calling process. 它是一个由以下一个或多个值的总和构成的八进制数字：

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

### Throttling .stat file touching {#throttling-stat-file-touching}

With the default `/invalidate` property, every activation effectively invalidates all `.html` files (when their path matches the `/invalidate` section). 在具有大量流量的网站上，后续激活将增加后端的CPU负载。In such a scenario, it would be desirable to &quot;throttle&quot; `.stat` file touching to keep the website responsive. You can do this by using the `/gracePeriod` property.

`/gracePeriod` 此属性定义了陈旧的自动失效资源的秒数，在最后一个发生激活后，该资源仍可能从缓存提供。该属性可在设置中使用，在此设置中，成批激活会以其他方式使整个缓存失效。建议的值为秒。

For additional details, also read the `/invalidate` and `/statfileslevel`sections above.

## Configuring Time Based Cache Invalidation - /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

If set, the `enableTTL` property will evaluate the response headers from the backend, and if they contain a `Cache-Control` max-age or `Expires` date, an auxiliary, empty file next to the cache file is created, with modification time equal to the expiry date. 当缓存缓存的文件时，会在修改时自动请求该缓存的文件。

You can enable the feature by adding this line to the `dispatcher.any` file:

```xml
/enableTTL "1"
```

>[!NOTE]
>
>This feature is avaiable with version **4.1.11** of the Dispatcher.

## Configuring Load Balancing - /statistics {#configuring-load-balancing-statistics}

`/statistics` 该部分定义了Dispatcher为每个渲染的响应速度提供的文件类别。Dispatcher使用分数确定要发送请求的渲染。

您创建的每个类别都定义了一个glob图案。调度程序将请求内容的URI与这些模式进行比较，以确定所请求内容的类别：

* 类别的顺序决定了与URI相比的顺序。
* 与URI匹配的第一个类别模式是文件的类别。不会评估其他类别模式。

Dispatcher最多支持个统计类别。如果定义超过个类别，则仅使用前八个类别。

**渲染选择**

每次调度程序需要渲染的页面时，它都使用以下算法来选择渲染：

1. If the request contains the render name in a `renderid` cookie, Dispatcher uses that render.
1. If the request includes no `renderid` cookie, Dispatcher compares the render statistics:

   1. 调度程序确定请求URI的冒号。
   1. 调度程序确定哪个渲染具有该类别的最低响应得分，并选择该渲染。

1. 如果尚未选择渲染，请使用列表中的第一个渲染。

渲染的类别得分基于之前的响应时间以及调度程序尝试的先前失败和成功的连接。对于每个尝试，将更新所请求URI类别的分数。

>[!NOTE]
>
>如果您不使用负载平衡，则可忽略此部分。

### Defining Statistics Categories {#defining-statistics-categories}

为要为渲染选择保留统计数据的每种类型的文档定义一个类别。/statistics部分包含一个/categories部分。要定义某个类别，请在/categories部分下面添加一行，其格式如下：

`/name { /glob "pattern"}`

The category `name` must be unique to the farm. `pattern`[designing属性](#designing-patterns-for-glob-properties) 部分的设计模式中介绍了这些功能。

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

### Reflecting Server Unavailability in Dispatcher Statistics {#reflecting-server-unavailability-in-dispatcher-statistics}

`/unavailablePenalty` 属性设置当连接到渲染失败时应用于渲染统计数据的时间(以秒为单位)。调度程序将时间添加到与所请求的URI匹配的统计数据类别。

例如，当无法建立指向指定主机名/端口的TCP/IP连接时，将应用罚款，这是因为AEM没有运行(并且不会侦听)或由于网络相关问题而导致。

`/unavailablePenalty` 该属性是章节 `/farm` 的直接子项( `/statistics` 章节的兄弟姐妹)。

If no `/unavailablePenalty` property exists, a value of &quot;1&quot; is used.

```xml
/unavailablePenalty "1"
```

## Identifying a Sticky Connection Folder - /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

`/stickyConnectionsFor` 该属性定义一个包含粘性文档的文件夹；这将使用URL进行访问。调度程序将所有请求从一个用户发送到同一个渲染实例中。粘性连接确保了会话数据对所有文档都具有一致性和一致性。This mechanism uses the `renderid` cookie.

以下示例定义了与/products文件夹的粘性连接：

```xml
/stickyConnectionsFor "/products"
```

When a page is composed of content from several content nodes, include the `/paths` property that lists the paths to the content. For example, a page contains content from `/content/image`, `/content/video`, and `/var/files/pdfs`. 以下配置可为页面上的所有内容启用粘性连接：

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

When sticky connections are enabled, the dispatcher module sets the `renderid` cookie. This cookie doesn&#39;t have the `httponly` flag, which should be added in order to enhance security. You can do this by setting the `httpOnly` property in the `/stickyConnections` node of a `dispatcher.any` configuration file. The property&#39;s value (either 0 or 1) defines whether the `renderid` cookie has the `HttpOnly` attribute appended. 默认值为0，这表示不会添加属性。

For additional information about the `httponly` flag, read [this page](https://www.owasp.org/index.php/HttpOnly).

### secure {#secure}

When sticky connections are enabled, the dispatcher module sets the `renderid` cookie. This cookie doesn&#39;t have the **secure** flag, which should be added in order to enhance security. You can do this by setting the `secure` property in the `/stickyConnections` node of a `dispatcher.any` configuration file. The property&#39;s value (either 0 or 1) defines whether the `renderid` cookie has the `secure` attribute appended. 默认值为0，这意味着将添加属性**传入请求是安全的。如果该值设置为1，则无论传入请求是否安全，都将添加安全标志。

## Handling Render Connection Errors {#handling-render-connection-errors}

在渲染服务器返回500错误或不可用时配置调度程序行为。

### Specifying a Health Check Page {#specifying-a-health-check-page}

Use the `/health_check` property to specify a URL that is checked when a 500 status code occurs. If this page also returns a 500 status code the instance is considered to be unavailable and a configurable time penalty ( `/unavailablePenalty`) is applied to the render before retrying.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Specifying the Page Retry Delay {#specifying-the-page-retry-delay}

The / `retryDelay` property sets the time (in seconds) that Dispatcher waits between rounds of connection attempts with the farm renders. 对于每个圆形，调度程序尝试连接渲染的最大次数是在农场中渲染的次数。

Dispatcher uses a value of `"1"` if `/retryDelay` is not explicitly defined. 大多数情况下，默认值是适当的。

```xml
/retryDelay "1"
```

### Configuring the Number of Retries {#configuring-the-number-of-retries}

`/numberOfRetries` 该属性设置Dispatcher对渲染的最大连接尝试次数。如果调度程序在此重试次数后无法成功连接到渲染，调度程序将返回失败的响应。

对于每个圆形，调度程序尝试连接渲染的最大次数是在农场中渲染的次数。Therefore, the maximum number of times that Dispatcher attempts a connection is ( `/numberOfRetries`) x (the number of renders).

If the value is not explicitly defined, the default value is **5**.

```xml
/numberOfRetries "5"
```

### Using the Failover Mechanism {#using-the-failover-mechanism}

在Dispatcher农场上启用故障转移机制，以便在原始请求失败时重新发送对不同渲染的请求。启用故障转移后，调度程序有以下行为：

* 当对渲染的请求返回HTTP状态503(不可用)时，调度程序将请求发送到其他渲染。
* When a request to a render returns HTTP status 50x (other than 503), Dispatcher sends a request for the page that is configured for the `health_check` property.

   * 如果健康检查返回500(INTERNAL_ SERVER_ ERROR)，Dispatcher将发送原始请求到不同的渲染。
   * 如果修复检查返回HTTP状态200，则调度程序将初始HTTP500错误返回给客户端。

要启用故障转移，请将以下行添加到农场(或网站)：

```xml
/failover "1" 
```

>[!NOTE]
>
>To retry HTTP requests that contain a body, Dispatcher sends a `Expect: 100-continue` request header to the render before spooling the actual contents. CQ5.5包含CQSE，然后立即回答100(继续)或错误代码。其他servlet容器也应支持此功能。

## Ignoring Interruption Errors - /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此选项。当您看到以下日志消息时，您只需要使用它：
>
>`Error while reading response: Interrupted system call`

Any file system oriented system call can be interrupted `EINTR` if the object of the system call is located on a remote system accessed via NFS. 这些系统调用是否可以超时或中断取决于底层文件系统在本地机器上的安装方式。

如果您的实例具有此类配置，并且日志包含以下消息，请使用social参数：

`Error while reading response: Interrupted system call`

内部，调度程序使用可表示为：的循环从远程服务器(即AEM)读取响应：

`while (response not finished) {  
read more data  
}`

Such messages can be generated when the `EINTR` occurs in the &quot; `read more data`&quot; section and are caused by the reception of a signal before any data was received.

To ignore such interrupts you can add the following parameter to `dispatcher.any` (before `/farms`):

`/ignoreEINTR "1"`

Setting `/ignoreEINTR` to `"1"` causes Dispatcher to continue to attempt to read data until the complete response is read. 默认值为0，取消激活该选项。

## Designing Patterns for glob Properties {#designing-patterns-for-glob-properties}

Several sections in the Dispatcher configuration file use `glob` properties as selection criteria for client requests. glob属性的值是调度程序与请求的一个方面相比较的模式，如请求的资源的路径或客户端的IP地址。For example, the items in the `/filter` section use glob patterns to identify the paths of the pages that Dispatcher acts on or rejects.

glob值可包括通配符字符和字母数字字符，用于定义模式。

| 通配符字符 | 描述 | 示例 |
|--- |--- |--- |
| `*` | 在字符串中匹配任意字符的零个或多个相邻实例。The final character of the match is determined by either of the following situations: <br/>A character in the string matches the next character in the pattern, and the pattern character has the following characteristics:<br/><ul><li>不是*</li><li>不是？</li><li>文本字符(包括空格)或字符类。</li><li>将到达图案的结尾。</li></ul>在字符类中，字符是解释的。 | `*/geo*` 匹配 `/content/geometrixx` 节点和 `/content/geometrixx-outdoors` 节点下的任何页面。The following HTTP requests match the glob pattern: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*`<br/>匹配 `/content/geometrixx-outdoors` 节点下的任何页面。For example, the following HTTP request matches the glob pattern: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 匹配任意单个字符。使用外部字符类。在字符类中，此字符经过解释。 | `*outdoors/??/*`<br/> 在geometrixx Outdoors站点中匹配任何语言的页面。For example, the following HTTP request matches the glob pattern: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>以下请求与glob模式不匹配： <br/><ul><li>“GET/content/geometrixx-outdoors/en.html”</li></ul> |
| `[ and ]` | 标记字符类的开始和结尾。字符类可以包括一个或多个字符范围和单个字符。<br/>如果目标字符与字符类中的任何字符匹配或在定义的范围内，则会发生匹配。<br/>如果未包含结束括号，则该模式不会产生匹配项。 | `*[o]men.html*`<br/> 匹配以下HTTP请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*`<br/>匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示一系列字符。用于字符类。在字符类之外，此字符是解释的。 | `*[m-p]men.html*` 匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Does not match the following HTTP request:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定后面的字符类或字符类。仅用于否定字符类内的字符和字符范围。Equivalent to the `^ wildcard`. <br/>在字符类之外，此字符是解释的。 | `*[!o]men.html*`<br/> 匹配以下HTTP请求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>与以下HTTP请求不匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 与以下HTTP请求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定后面的字符或字符范围。用于仅对字符类内的字符和字符范围进行否定。Equivalent to the `!` wildcard character. <br/>在字符类之外，此字符是解释的。 | `!` 通配符字符示例应用，用字符替换示例模式中的 `!``^` 字符。 |


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

## Logging {#logging}

在Web服务器配置中，您可以设置：

* 调度程序日志文件的位置。
* 日志级别。

有关详细信息，请参阅Web服务器文档和Dispatcher实例的自述文件。

**Apache旋转/Pied Logs**

If using an **Apache** web server you can use the standard functionality for rotated and/or piped logs. 例如，使用管道日志：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

这将自动旋转：

* 调度程序日志文件；在扩展中具有时间戳(logs/dispatcher. log%Y%m%d)。
* (60x60x24x7=604800秒)。

Please see the Apache web server documentation on Log Rotation and Piped Logs; for example [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>安装时，默认日志级别较高(即级别3=调试)，以便Dispatcher记录所有错误和警告。这在初始阶段非常有用。
>
>However, this requires additional resources, so when the Dispatcher is working smoothly *according to your requirements*, you can(should) lower the log level.

### Trace Logging {#trace-logging}

在Dispatcher的其他增强功能中，版本4.2.0还引入了跟踪日志记录。

这比调试日志记录更高，在日志中显示其他信息。它添加了以下记录：

* 转发的标题的值；
* 应用于特定操作的规则。

You can enable Trace Logging by setting the log level to `4` in your web server.

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

## Confirming Basic Operation {#confirming-basic-operation}

要确认Web服务器、调度程序和AEM实例的基本操作和交互，您可以使用以下步骤：

1. Set the `loglevel` to `3`.

1. 启动Web服务器；这也会启动Dispatcher。
1. 启动AEM实例。
1. 检查Web服务器和Dispatcher的日志和错误文件。\
   根据Web服务器的不同，您应当看到以下消息：\
   `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`\
   和:\
   `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 通过Web服务器冲浪网站。确认根据需要显示内容。\
   For example, on a local installation where AEM runs on port `4502` and the web server on `80` access the Websites console using both:\
   ` https://localhost:4502/libs/wcm/core/content/siteadmin.html  
https://localhost:80/libs/wcm/core/content/siteadmin.html  
`结果应相同。使用相同的机制确认对其他页面的访问权限。

1. 检查缓存目录是否已填写。
1. 激活页面以检查是否已正确清除缓存。
1. If everything is operating correctly you can reduce the `loglevel` to `0`.

## Using Multiple Dispatchers {#using-multiple-dispatchers}

在复杂的设置中，您可以使用多个调度程序。例如，您可以使用：

* 一个Dispatcher，用于在Intranet上发布网站
* 在不同地址下和具有不同安全性设置的第二个调度程序下发布Internet上的相同内容。

在这种情况下，请确保每个请求只通过一个调度程序。Dispatcher不处理来自其他Dispatcher的请求。因此，请确保两个Dispatcher直接访问AEM网站。

## Debugging {#debugging}

When adding the header `X-Dispatcher-Info` to a request, Dispatcher answers whether the target was cached, returned from cached or not cacheable at all. The response header `X-Cache-Info` contains this information in a readable form. 您可以使用这些响应标题调试涉及Dispatcher缓存的响应的问题。

This functionality is not enabled by default, so in order for the response header `X-Cache-Info` to be included, the farm must contain the following entry:

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

Below is a list containing the response headers that `X-Dispatcher-Info` will return:

* **缓存**\
   目标文件包含在缓存中，调度程序已确定交付它是有效的。
* **缓存**\
   目标文件未包含在缓存中，调度程序已确定缓存输出并交付它。
* **缓存：stat file is edirectly**
the target file is contains in the cache，it is more by a new stat file.调度程序将删除目标文件，从输出中重新创建它并提供它。
* **不可缓存：没有文档根目录** 该农场的配置不包含文档根目录(配置元素 `cache.docroot`)。
* **不可缓存：缓存文件路径过长**\
   目标文件-文档根和URL文件的连接超过系统上最长的可能文件名。
* **不可缓存：临时文件路径过长**\
   临时文件名模板超过系统上最长可能的文件名。调度程序首先创建一个临时文件，然后实际创建或覆盖缓存的文件。The temporary file name is the target file name with the characters `_YYYYXXXXXX` appended to it, where the `Y` and `X` will be replaced to create a unique name.
* **不可缓存：请求URL没有扩展**\
   The request URL has no extension, or there is a path following the file extension, for example: `/test.html/a/path`.
* **不可缓存：请求不是GET或HEAD**
HTTP方法既不是GET也不是HEAD。调度程序假定输出将包含不应缓存的动态数据。
* **不可缓存：请求包含一个查询字符串**\
   请求包含一个查询字符串。调度程序假定输出取决于给定的查询字符串，因此不缓存。
* **不可缓存：会话管理器未身份验证**\
   The farm&#39;s cache is governed by a session manager (the configuration contains a `sessionmanagement` node) and the request didn&#39;t contain the appropriate authentication information.
* **不可缓存：请求包含授权**\
   The farm is not allowed to cache output ( `allowAuthorized 0`) and the request contains authentication information.
* **不可缓存：target is a directory**\
   目标文件是目录。This might point to some conceptual mistake, where a URL and some sub-URL both contain cacheable output, for example if a request to `/test.html/a/file.ext` comes first and contains cacheable output, the dispatcher will not be able to cache the output of a subsequent request to `/test.html`.
* **不可缓存：请求URL具有尾部斜杠**\
   请求URL有一个尾部斜杠。
* **不可缓存：请求URL不在缓存规则中**\
   农场的缓存规则明确拒绝缓存某些请求URL的输出。
* **不可缓存：授权检查器拒绝访问**\
   农场的授权检查器拒绝访问缓存的文件。
* **不可缓存：会话无效** 该农场的缓存由会话管理器控制(配置包含一个 `sessionmanagement` 节点)，并且用户的会话不再有效。
* **不可缓存：响应包含`no_cache `** 返回的服务器返回 `Dispatcher: no_cache` 标题，禁止调度程序缓存输出。
* **不可缓存：响应内容长度为零**，响应的内容长度为零；调度程序不会创建零长度文件。
