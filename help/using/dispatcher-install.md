---
title: 安装Dispatcher
seo-title: 安装AEM Dispatcher
description: 了解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安装Dispatcher模块。
seo-description: 了解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安装AEM调度程序模块。
uuid: 2384b907-1042-4707-b02 f-fba2125618 cf
contentOwner: 用户
converted: 'true'
topic-tags: 调度程序
content-type: 引用
discoiquuid: f00ad751-6b95-4365-8500-e1 e0108 d9536
translation-type: tm+mt
source-git-commit: 6d3ff696780ce55c077a1d14d01efeaebcb8db28

---


# Installing Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

>[!NOTE]
>
>Dispatcher版本独立于AEM。如果您遵循了一个指向Dispatcher文档的链接，则可能已重定向到该页面，该链接嵌入到AEM的先前版本的文档中。

Use the [Dispatcher Release Notes](release-notes.md) page to obtain the latest Dispatcher installation file for your operating system and web server. Dispatcher发行版编号与Adobe Experience Manager发布号码无关，与Adobe Experience Manager6.x、5.x和Adobe CQ5.x版本兼容。

使用以下文件命名规范：

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

For example, the `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` file contains Dispatcher version 4.3.1 for an Apache 2.4 web server that runs on Linux i686 and is packaged using the **tar** format.

下表列出了每个Web服务器的文件名中使用的Web服务器标识符：

| Web 服务器 | 安装套件 |
|--- |--- |
| Apache2.4 | dispatcher-apache **2.4**-&lt;other parameters&gt; |
| Microsoft Internet Information Server7.5、8、8.5 | dispatcher-**iis**-&lt;other parameters&gt; |
| Sun Java Web Server iPlanet | **调度程序-ns-**&lt;其他参数&gt; |

>[!NOTE]
>
>您应该安装可用于平台的最新版Dispatcher。您应当每年升级Dispatcher实例以使用最新版本以利用产品改进。

每个归档文件都包含以下文件：

* Dispatcher模块
* 示例配置文件
* 包含安装说明和最后一分钟信息的自述文件
* 列出在当前版本和过去版本中修复的问题的更改文件

>[!NOTE]
>
>请在开始安装之前检查README文件以获取任何最新更改/平台特定备注。

<!-- 

Comment Type: draft

<h3>Supported Web Servers</h3>

 -->

<!-- 

Comment Type: draft

<p>The following web servers are supported for use with Dispatcher version 4.1.12:</p>

 -->

<!-- 

Comment Type: draft

<p>The following sections detail the specific web server installation procedures.</p>

 -->

## Microsoft Internet Information Server {#microsoft-internet-information-server}

有关如何安装此Web服务器的信息，请参阅以下资源：

* Microsoft在Internet Information Server上自己的文档
* [“官方Microsoft IIS站点”](https://www.iis.net/)

### Required IIS Components {#required-iis-components}

IIS版本8.5和10要求安装以下IIS组件：

* ISAPI扩展

此外，您必须添加Web服务器(IIS)角色。使用服务器管理器添加角色和组件。

## Microsoft IIS - Installing the Dispatcher module {#microsoft-iis-installing-the-dispatcher-module}

Microsoft Internet信息系统所需的存档为：

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

ZIP文件包含以下文件：

| 文件 | 描述 |
|--- |--- |
| `disp_iis.dll` | Dispatcher动态链接库文件。 |
| `disp_iis.ini` | IIS的配置文件。此示例可根据您的要求进行更新。**注意**：ini文件必须具有与dll相同的名称根目录。 |
| `dispatcher.any` | Dispatcher的示例配置文件。 |
| `author_dispatcher.any` | 用于使用创作实例的Dispatcher的示例配置文件。 |
| 自述 | 包含安装说明和最后一分钟信息的自述文件。**注意**：请在开始安装之前检查此文件。 |
| 更改 | 更改了在当前版本和过去版本中修复的问题的文件。 |

请按照以下过程将Dispatcher文件复制到正确的位置。

1. Use Windows Explorer to create the `<IIS_INSTALLDIR>/Scripts` directory, for example, `C:\inetpub\Scripts`.

1. 将以下文件从调度程序包解压到此Scripts目录中：

   * `disp_iis.dll`
   * `disp_iis.ini`
   * 以下文件之一，具体取决于Dispatcher在使用AEM作者实例或发布实例：
      * Author instance: `author_dispatcher.any`
      * Publish instance: `dispatcher.any`

## Microsoft IIS - Configure the Dispatcher INI File {#microsoft-iis-configure-the-dispatcher-ini-file}

Edit the `disp_iis.ini` file to configure the Dispatcher installation. `.ini` 文件的基本格式如下所示：

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

下表介绍了每个属性。

| 参数 | 描述 |
|--- |--- |
| configpath | The location of `dispatcher.any` within the local file system (absolute path). |
| logfile | `dispatcher.log` 文件的位置。如果未设置此设置，则日志消息将转到窗口事件日志。 |
| loglevel | 定义用于向活动日志输出消息的日志级别。The following values may be specified:Log level for the log file: <br/>0 - error messages only. <br/>1 - 错误和警告。<br/>2 - 错误、警告和信息性消息 <br/>3-错误、警告、信息性和调试消息。<br/>**注意**：建议在安装和测试过程中将日志级别设置为3，然后在生产环境中运行时将日志级别设置为0。 |
| 替换授权 | 指定如何处理HTTP请求中的授权头。The following values are valid:<br/>0 - Authorization headers are not modified. <br/>1 - 替换任何名为“Authorization”的标题，但其 `Basic <IIS:LOGON\_USER>` 等效项除外。<br/> |
| servervariables变量 | 定义服务器变量的处理方式。<br/>0 - IIS服务器变量发送到调度程序和AEM。<br/>1 - 所有IIS服务器变量(如 `LOGON\_USER, QUERY\_STRING, ...`)都发送到Dispatcher，以及请求标头(如果未缓存，则还会发送到AEM实例)。<br/>服务器变量包括 `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` 和许多其他变量。有关变量的完整列表，请参阅IIS文档，其中包含详细信息。 |
| enable_ chonged_ transfer | 定义是否为客户端响应启用(1)或禁用(0)分块传输。默认值为 0。 |

示例配置：

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### Configuring Microsoft IIS {#configuring-microsoft-iis}

配置IIS以集成Dispatcher ISAPI模块。在IIS中，您使用通配符应用程序映射。

### Configuring Anonymous Access - IIS 8.5 and 10 {#configuring-anonymous-access-iis-and}

对作者实例上的默认刷新复制代理进行配置，以便它不发送具有刷新请求的安全凭据。因此，您使用调度程序缓存的网站必须允许匿名访问。

如果您的网站使用身份验证方法，则必须相应地配置刷新复制代理。

1. 打开IIS Manager，然后选择您用作“清除程序”缓存的网站。
1. 使用功能视图模式，在IIS部分中双击身份验证。
1. 如果未启用匿名身份验证，则选择“匿名身份验证”，在“操作”区域中单击“启用”。

### Integrating the Dispatcher ISAPI Module - IIS 8.5 and 10 {#integrating-the-dispatcher-isapi-module-iis-and}

请按照以下过程将Dispatcher ISAPI模块添加到IIS。

1. 打开IIS Manager。
1. 选择您用作调度程序缓存的网站。
1. 使用功能视图模式，在IIS部分中双击处理程序映射。
1. 在“处理程序映射”页面的“操作”面板中，单击“添加通配符脚本映射”，添加以下属性值，然后单击确定：

   * 请求路径：*
   * Executable: The absolute path of the disp_iis.dll file, for example `C:\inetpub\Scripts\disp_iis.dll`.
   * Name: A descriptive name for the handler mapping, for example `Dispatcher`.

1. 在出现的对话框中，要将disp_ iis. dll库添加到ISAPI和CGI限制列表中，请单击“是”。

   对于IIS7.0和7.5，配置已完成。如果配置IIS8.0，请继续执行其余步骤。

1. (IIS8.0)在处理函数映射列表中，选择刚才创建的，在“操作”区域中单击编辑。
1. (IIS8.0)在“编辑脚本映射”对话框中，单击“请求限制”按钮。
1. (IIS8.0)要确保处理函数用于尚未缓存的文件和文件夹，请取消选择“仅当请求已映射到”，然后单击“确定”即可取消选中“调用处理程序”。
1. (IIS8.0)在“编辑脚本映射”对话框上，单击“确定”。

### Configuring Access to the Cache - IIS 8.5 and 10 {#configuring-access-to-the-cache-iis-and}

为默认应用程序池用户提供对用作调度程序缓存的文件夹的写入访问权限。

1. Right-click the root folder of the website that you are using as the Dispatcher cache and click Properties, such as `C:\inetpub\wwwroot`.
1. 在安全选项卡上，单击编辑，然后在“权限”对话框中单击添加。此时将打开一个对话框，用于选择用户帐户。单击“位置”按钮，选择计算机名称，然后单击“确定”。

   完成下一步后，将此对话框保持打开状态。

1. 在IIS Manager中，选择您正在用于调度程序缓存的IIS站点，然后在窗口的右侧单击高级设置。
1. 选择“应用程序池”属性的值并将其复制到剪贴板。
1. 返回打开对话框。In the Enter The Object Names To Select box, type `IIS AppPool\` and then paste the contents of your clipboard. 该值应类似于以下示例：

   `IIS AppPool\DefaultAppPool`

1. 单击“复选名称”按钮。当Windows解析用户帐户时，单击“确定”。
1. In the Permissions dialog box for the dispatcher folder, select the account that you just added, enable all of the permissions for the account  **except for Full Control** and click OK. 单击确定以关闭文件夹属性对话框。

### Registering the JSON Mime Type - IIS 8.5 and 10 {#registering-the-json-mime-type-iis-and}

当您希望Dispatcher允许JSON调用时，请使用以下过程注册JSON MIME类型。

1. 在IIS Manager中，选择您的网站并使用功能视图，双击MIME类型。
1. 如果JSON扩展不在列表中，请在“操作”面板中单击“添加”，输入以下属性值，然后单击确定：

   * File Name Extension: `.json`
   * MIME Type: `application/json`

### Removing the bin Hidden Segment - IIS 8.5 and 10 {#removing-the-bin-hidden-segment-iis-and}

Use the following procedure to remove the `bin` hidden segment. 不是新的Web站点可以包含此隐藏区段。

1. 在IIS Manager中，选择您的网站并使用功能视图，双击请求过滤。
1. Select the `bin` segment, click Remove, and in the confirmation dialog box click Yes.

### Logging IIS Messages to a File - IIS 8.5 and 10 {#logging-iis-messages-to-a-file-iis-and}

请按照以下过程将调度程序日志消息写入日志文件而非Windows活动日志。您需要配置Dispatcher以使用日志文件，并提供对文件的写访问权限。

1. Use Windows Explorer to create a folder named `dispatcher` below the logs folder of the IIS installation. The path of this folder for a typical installation is `C:\inetpub\logs\dispatcher`.

1. 右键单击调度程序文件夹，然后单击“属性”。
1. 在安全选项卡上，单击编辑，然后在“权限”对话框中单击添加。此时将打开一个对话框，用于选择用户帐户。单击“位置”按钮，选择计算机名称，然后单击“确定”。

   完成下一步后，将此对话框保持打开状态。

1. 在IIS Manager中，选择您正在用于调度程序缓存的IIS站点，然后在窗口的右侧单击高级设置。
1. 选择“应用程序池”属性的值并将其复制到剪贴板。
1. 返回打开对话框。In the Enter The Object Names To Select box, type `IIS AppPool\` and then paste the contents of your clipboard. 该值应类似于以下示例：

   `IIS AppPool\DefaultAppPool`

1. 单击“复选名称”按钮。当Windows解析用户帐户时，单击“确定”。
1. 在调度程序文件夹的“权限”对话框中，选择刚刚添加的帐户，支持帐户**的所有权限**但完全控制，**并单击确定。单击确定以关闭文件夹属性对话框。
1. Use a text editor to open the `disp_iis.ini` file.
1. 添加类似于以下示例的文本行，以配置日志文件的位置，然后保存文件：

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Next Steps {#next-steps}

在开始使用Dispatcher之前，您必须了解：

* [配置](dispatcher-configuration.md) 调度程序
* [混淆AEM](page-invalidate.md) 以使用Dispatcher。

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>Instructions for installation under both **Windows** and **Unix** are covered here. 执行步骤时请务必小心。

### Installing Apache Web Server {#installing-apache-web-server}

For Information about how to install an Apache Web Server read the installation manual - either [online](https://httpd.apache.org/) or in the distribution.

>[!CAUTION]
>
>If you are creating an Apache binary by compiling the source files, make sure that you turn on **dynamic modules support**. This can be done by using any of the **--enable-shared** options. At a minimum include the `mod_so` module.
>
>有关更多信息，请参阅Apache Web Server安装手册。

Also see the Apache HTTP Server [Security Tips](https://httpd.apache.org/docs/2.4/misc/security_tips.html) and [Security Reports](https://httpd.apache.org/security_report.html).

### Apache Web Server - Add the Dispatcher Module {#apache-web-server-add-the-dispatcher-module}

Dispatcher为：

* **Windows**：Dynamic Link Library(DLL)
* **Unix**：动态共享对象(DSO)

安装存档文件包含以下文件-具体取决于您是否选择了Windows或Unix：

| 文件 | 描述 |
|--- |--- |
| disp_ apache&lt; x. y&gt;. dll | Windows：Dispatcher动态链接库文件。 |
| dispatcher-apache&lt; x. y&gt;-&lt; rel-loc&gt;. so | Unix：调度程序共享对象库文件。 |
| mod_ dispatcher. so | Unix：示例链接。 |
| http. conf. disp&lt; x&gt; | Apache服务器的示例配置文件。 |
| 调度程序 | Dispatcher的示例配置文件。 |
| 自述 | 包含安装说明和最后一分钟信息的自述文件。**注意**：请在开始安装之前检查此文件。 |
| 更改 | 更改了在当前版本和过去版本中修复的问题的文件。 |

请按照以下步骤将Dispatcher添加到Apache Web Server：

1. 将Dispatcher文件放入适当的Apache模块目录中：

   * **Windows**：Place `disp_apache<x.y>.dll``<APACHE_ROOT>/modules`
   * **Unix**：根据安装查找或 `<APACHE_ROOT>/libexec``<APACHE_ROOT>/modules`目录。\
      Copy `dispatcher-apache<options>.so` into this directory.\
      To simplify long-term maintenance you can also create a symbolic link named `mod_dispatcher.so` to the Dispatcher:\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Copy the dispatcher.any file to the `<APACHE_ROOT>/conf` directory.

   **注意：** 只要相应地配置Dispatcher模块的dispatcherLog属性，您可以将此文件放置在其他位置。(请参阅下面的调度程序特定配置条目。)

### Apache Web Server - Configure SELinux Properties {#apache-web-server-configure-selinux-properties}

如果您在启用SERINX的RedHat Linux内核2.6上运行Dispatcher，您可能会在调度程序日志文件中运行错误消息。

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

这可能是由于启用了SERInX安全性的。然后您需要执行以下任务：

* 配置调度程序模块文件的SerialUX上下文。
* 启用HTPD脚本和模块以建立网络连接。
* 配置docroot的SerialUX上下文，其中存储缓存的文件。

Enter the following commands in a terminal window, replacing `[path to the dispatcher.so file]` with the path to the Dispatcher module that you installed to Apache Web Server, and *`path to the docroot`* with the path where the docroot is located (e.g. `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server - Configure Apache Web Server for Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

The Apache Web Server needs to be configured, using `httpd.conf`. In the Dispatcher installation kit you will find an example configuration file named `httpd.conf.disp<x>`.

这些步骤是强制性的：

1. 导航至 `<APACHE_ROOT>/conf`.
1. Open `httpd.conf`for editing.
1. 必须按列出的顺序添加以下配置条目：

   * **加载模块** 以在启动时加载模块。
   * Dispatcher-specific configuration entries, including **DispatcherConfig, DispatcherLog** and **DispatcherLogLevel**.
   * **setHandler** 以激活Dispatcher。**LoadModule**.
   * **ModmMeusePateInfo** 配置 **mod_ mime的行为**。

1. (可选)建议您更改htdocs目录的所有者：

   * apache服务器以root开头，但子进程作为守护程序开始(出于安全目的)。The DocumentRoot (`<APACHE_ROOT>/htdocs`) must belong to the user daemon:

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

下表列出了可使用的示例；具体条目取决于特定Apache Web Server：

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix(假定符号链接) | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>每个语句的第一个参数必须像上面的示例一样编写。
>
>请参阅提供的示例配置文件和Apache Web Server文档，了解有关此命令的完整详细信息。

**调度程序特定的配置条目**

Dispatcher特定的配置条目放在LoadModule条目之后。下表列出了适用于Unix和Windows的示例配置：

**Windows和Unix**

```
...
<fModule disp_apache2.c>
DispatcherConfig conf/dispatcher.any
DispatcherLog logs/dispatcher.log DispatcherLogLevel 3
DispatcherNoServerHeader 0 DispatcherDeclineRoot 0
DispatcherUseProcessedURL 0
DispatcherPassError 0
DispatcherKeepAliveTimeout 60
</IfModule>
...
```

单个配置参数：

| 参数 | 描述 |
|--- |--- |
| DispatcherConfig | Dispatcher配置文件的位置和名称。<br/>当此属性位于主服务器配置中时，所有虚拟主机都会继承该属性值。但是，虚拟主机可以包括一个调度程序配置属性以覆盖主服务器配置。 |
| DispatcherLog | 日志文件的位置和名称。 |
| DispatcherLogLevel | Log level for the log file: <br/>0 - Errors <br/>1 - Warnings <br/>2 - Infos <br/>3 - Debug <br/>**Note**: It is recommended to set the log level to 3 during installation and testing, then to 0 when running in a production environment. |
| DispatcherNoServerHeader | *此参数已弃用，不再具有任何效果。*<br/><br/> 定义要使用的服务器头： <br/><ul><li>未定义或0- HTTP服务器头包含AEM版本。 </li><li>1 - 使用Apache服务器头。</li></ul> |
| DispatcherDeclineRoot | Defines whether to decline requests to the root &quot;/&quot;: <br/>**0** - accept requests to / <br/>**1** - requests to / are not handled by the dispatcher; use mod_alias for the correct mapping. |
| DispatcherUseProcessSubURL | Defines whether to use pre-processed URLs for all further processing by Dispatcher: <br/>**0** - use the original URL passed to the web server. <br/>**** -调度程序使用已经由调度程序处理的URL(即调度 `mod_rewrite`程序)之前的URL，而不是传递给Web服务器的原始URL。例如，原始或处理的URL与调度程序过滤器相匹配。URL还用作缓存文件结构的基础。有关mod_ rewrite的信息，请参阅Apache网站文档；例如Apache2.4。使用mod_ rewrite时，使用标记“passhrough”是建议使用 | PT&#39;(传递到下一个处理函数)以强制重写引擎将internal request_ rec结构的uri字段设置为文件名字段的值。 |
| DispatcherPassError | Defines how to support error codes for ErrorDocument handling: <br/>**0** - Dispatcher spools all error responses to the client. <br/>**** -调度程序不会向客户端发送错误响应(状态代码大于或等于400)，但将状态代码传递给Apache，如允许ErdRockocument指令处理此类状态代码。<br/>**代码范围** -指定将响应传递给Apache的一系列错误代码。其他错误代码将传递给客户端。例如，以下配置将错误412的响应传递给客户端，所有其他错误将传递给Apache：DispatcherPassServ400-411,413-417 |
| DispatcherVillAliveTimeout | 指定持续时间超时，以秒为单位。从调度程序4.2.0开始，默认的保留活动量为60。值为可禁用保持活动量。 |
| dispatcherClyanOnURL | 将此参数设置为On会将原始URL传递给后端而不是画布图标，将覆盖DispatcherUseProcessSubeURL的设置。默认值为关闭。<br/>**注意**：Dispatcher配置中的过滤器规则始终会根据不是原始URL的清理URL进行评估。 |




>[!NOTE]
>
>路径条目相对于Apache Web Server的根目录。

>[!NOTE]
>
>The default settings for the Server Header are: `  
ServerTokens Full` `  
DispatcherNoServerHeader 0`\
其中显示AEM版本(用于统计目的)。If you want to disable such information being available in the header you can set: `  
ServerTokens Prod`\
See the [Apache Documentation about ServerTokens Directive (for example, for Apache 2.4)](https://httpd.apache.org/docs/2.4/mod/core.html) for more information.

**SetHandler**

After these entries you must add a **SetHandler** statement to the context of your configuration ( `<Directory>`, `<Location>`) for the Dispatcher to handle the incoming requests. 以下示例配置Dispatcher以处理完整网站的请求：

**Windows和Unix**

```
...  
<Directory />  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

以下示例配置Dispatcher以处理虚拟域请求：

**Windows**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot _\[cache-path\]_\\docs  
<Directory _\[cache-path\]_\\docs>  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

**Unix**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot /usr/apachecache/docs  
<Directory /usr/apachecache/docs>  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

>[!NOTE]
**setHandler** 语句的参数必须 *像上面的示例*一样编写，因为这是模块中定义的处理函数的名称。
请参阅提供的示例配置文件和Apache Web Server文档，了解有关此命令的完整详细信息。

**ModmMeusePathInfo**

After the **SetHandler** statement you should also add the **ModMimeUsePathInfo** definition.

>[!NOTE]
`ModMimeUsePathInfo` 仅当您使用Dispatcher4.0.9或更高版本时，才应使用和配置参数。
(注意，Dispatcher版本4.0.9已于2011年发布。如果您使用的是旧版本，则升级到最近的Dispatcher版本将非常合适。

The **ModMimeUsePathInfo** parameter should be set `On` for all Apache configurations:

`ModMimeUsePathInfo On`

The mod_mime module (see for example, [Apache Module mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) is used to assign content metadata to the content selected for an HTTP response. 默认设置意味着当mod_ mime确定内容类型时，只会考虑映射到文件或目录的URL部分。

When `On`, the `ModMimeUsePathInfo` parameter specifies that `mod_mime` is to determine the content type based on the *complete* URL; this means that virtual resources will have metainformation applied based on their extension.

The following example activates **ModMimeUsePathInfo**:

**Windows和Unix**

```
...  
<Directory />  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
ModMimeUsePathInfo On  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

### Enable Support for HTTPS (Unix and Linux) {#enable-support-for-https-unix-and-linux}

Dispatcher使用OpenSSL实现通过HTTP实现安全通信。Starting from Dispatcher version **4.2.0**, OpenSSL 1.0.0 and OpenSSL 1.0.1 are supported. 默认情况下，Dispatcher使用OpenSSL1.0.0。要使用OpenSSL1.0.1，请使用以下过程创建符号链接，以便Dispatcher使用已安装的OpenSSL库。

1. 打开终端并将当前目录更改为安装OpenSSL库的目录，例如：

   ```shell
   cd /usr/lib64
   ```

1. 要创建符号链接，请输入以下命令：

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
If you are using a customized version of Apache, make sure Apache and Dispatcher are compiled using the same version of [OpenSSL](https://www.openssl.org/source/).

### Next Steps {#next-steps-1}

在开始使用调度程序之前，您必须现在：

* [配置](dispatcher-configuration.md) 调度程序
* [混淆AEM](page-invalidate.md) 以使用Dispatcher。

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
此处介绍了适用于Windows和Unix环境的说明。
请谨慎选择要执行的操作。

### Sun Java System Web Server / iPlanet - Installing your Web Server {#sun-java-system-web-server-iplanet-installing-your-web-server}

有关如何安装这些Web服务器的完整信息，请参阅其各自的文档：

* Sun Java System Web Server
* iPlanet Web Server

### Sun Java System Web Server / iPlanet - Add the Dispatcher Module {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher为：

* **Windows**：Dynamic Link Library(DLL)
* **Unix**：动态共享对象(DSO)

安装存档文件包含以下文件-具体取决于您是否选择了Windows或Unix：

| 文件 | 描述 |
|---|---|
| `disp_ns.dll` | Windows：Dispatcher动态链接库文件。 |
| `dispatcher.so` | Unix：调度程序共享对象库文件。 |
| `dispatcher.so` | Unix：示例链接。 |
| `obj.conf.disp` | Ilanet/Sun Java System Web服务器的示例配置文件。 |
| `dispatcher.any` | Dispatcher的示例配置文件。 |
| 自述 | 包含安装说明和最后一分钟信息的自述文件。注意：请在开始安装之前检查此文件。 |
| 更改 | 更改了在当前版本和过去版本中修复的问题的文件。 |

请按照以下步骤将Dispatcher添加到Web服务器：

1. Place the Dispatcher file in the web server&#39;s `plugin` directory:

### Sun Java System Web Server / iPlanet - Configure for the Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

The web server needs to be configured, using `obj.conf`. In the Dispatcher installation kit you will find an example configuration file named `obj.conf.disp`.

1. 导航至 `<WEBSERVER_ROOT>/config`.
1. Open `obj.conf`for editing.
1. 复制开始的行：\
   `Service fn="dispService"`\
   `obj.conf.disp` 从到的初始化部分 `obj.conf`。

1. 保存更改。
1. Open `magnus.conf` for editing.
1. 复制开始的两行：\
   `Init funcs="dispService, dispInit"`\
   和\
   `Init fn="dispInit"`\
   `obj.conf.disp` 从到的初始化部分 `magnus.conf`。

1. 保存更改。

>[!NOTE]
The following configurations should all be on one line and the `$(SERVER_ROOT)` and `$(PRODUCT_SUBDIR)` must be replaced by the respective values.

**Init**

下表列出了可使用的示例；具体条目取决于特定的Web服务器：

**Windows和Unix**

```
...  
Init funcs="dispService,dispInit" fn="load-modules" shlib="$(SERVER\_ROOT)/plugins/dispatcher.so"  
Init fn="dispInit" config="$(PRODUCT\_SUBDIR)/dispatcher.any" loglevel="1" logfile="$(PRODUCT\_SUBDIR)/logs/dispatcher.log"  
keepalivetimeout="60"  
...
```

其中：

| 参数 | 描述 |
|--- |--- |
| config | Location and name of the configuration file `dispatcher.any.` |
| logfile | 日志文件的位置和名称。 |
| loglevel | Log level for when writing messages to the log file: <br/>**0** Errors <br/>**1** Warnings <br/>**2** Infos <br/>**3** Debug <br/>**Note:** It is recommended to set the log level to 3 during installation and testing and to 0 when running in a production environment. |
| keepalivetime超时 | 指定持续时间超时，以秒为单位。从调度程序4.2.0开始，默认的保留活动量为60。值为可禁用保持活动量。 |

根据您的要求，您可以将调度程序定义为对象的服务。为整个网站配置Dispatcher修改默认对象：


**Windows**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)\\dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*\\\*"  
...
```

**Unix**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)/dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*/\*"  
...
```

### Next Steps {#next-steps-2}

在开始使用调度程序之前，您必须现在：

* [配置](dispatcher-configuration.md) 调度程序
* [混淆AEM](page-invalidate.md) 以使用Dispatcher。
