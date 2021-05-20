---
title: 安装 Dispatcher
seo-title: 安装AEM Dispatcher
description: 了解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安装Dispatcher模块。
seo-description: 了解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安装AEM Dispatcher模块。
uuid: 2384b907-1042-4707-b02f-fba2125618cf
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
discoiquuid: f00ad751-6b95-4365-8500-e1e0108d9536
exl-id: 9375d1c0-8d9e-46cb-9810-fa4162a8c1ba
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '3684'
ht-degree: 0%

---

# 安装 Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

使用[Dispatcher发行说明](release-notes.md)页面获取操作系统和Web服务器的最新Dispatcher安装文件。 Dispatcher版本号与Adobe Experience Manager版本号无关，与Adobe Experience Manager 6.x、5.x和Adobe CQ 5.x版本兼容。

>[!NOTE]
>
>请注意，Adobe Experience Manager 6.5需要Dispatcher版本4.3.2或更高版本。 也就是说，Dispatcher版本与AEM无关，例如，Dispatcher版本4.3.2也与Adobe Experience Manager 6.4兼容。

使用以下文件命名约定：

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

例如，`dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz`文件包含适用于在Linux i686上运行的Apache 2.4 Web服务器的Dispatcher版本4.3.1，并使用&#x200B;**tar**&#x200B;格式打包。

下表列出了在每个Web服务器的文件名中使用的Web服务器标识符：

| Web 服务器 | 安装套件 |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;其他参数> |
| Microsoft Internet Information Server 7.5、8、8.5 | dispatcher-**iis**-&lt;其他参数> |
| Sun Java Web Server iPlanet | dispatcher-**ns**-&lt;其他参数> |

>[!CAUTION]
>
>您应该安装适用于您的平台的最新版本的Dispatcher。 您应该每年升级Dispatcher实例，以使用最新版本来利用产品改进。

每个存档都包含以下文件：

* 调度程序模块
* 示例配置文件
* 包含安装说明和最后时刻信息的自述文件
* 列出当前版本和过去版本中已修复问题的CHANGES文件

>[!NOTE]
>
>在开始安装之前，请查看自述文件，了解任何最后时刻的更改/平台特定说明。

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

* Internet Information Server上Microsoft自己的文档
* [“The Formical Microsoft IIS site”（官方的Microsoft IIS站点）](https://www.iis.net/)

### 所需的IIS组件{#required-iis-components}

IIS版本8.5和10要求安装以下IIS组件：

* ISAPI扩展

此外，您还必须添加Web服务器(IIS)角色。 使用服务器管理器添加角色和组件。

## Microsoft IIS — 安装调度程序模块{#microsoft-iis-installing-the-dispatcher-module}

Microsoft Internet Information System所需的存档是：

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

ZIP文件包含以下文件：

| 文件 | 描述 |
|--- |--- |
| `disp_iis.dll` | Dispatcher动态链接库文件。 |
| `disp_iis.ini` | IIS的配置文件。 此示例可根据您的要求进行更新。 **注意**:ini文件必须与dll具有相同的名称根。 |
| `dispatcher.any` | Dispatcher的示例配置文件。 |
| `author_dispatcher.any` | Dispatcher使用创作实例的示例配置文件。 |
| 自述文件 | 自述文件，其中包含安装说明和最后时刻信息。 **注意**:请在开始安装之前检查此文件。 |
| 更改 | 更改文件，其中列出当前版本和过去版本中已修复的问题。 |

请按照以下过程将Dispatcher文件复制到正确的位置。

1. 使用Windows资源管理器创建`<IIS_INSTALLDIR>/Scripts`目录，例如`C:\inetpub\Scripts`。

1. 将以下文件从Dispatcher包解压到此Scripts目录中：

   * `disp_iis.dll`
   * `disp_iis.ini`
   * 根据Dispatcher是使用AEM创作实例还是发布实例，确定以下文件之一：
      * 创作实例：`author_dispatcher.any`
      * 发布实例：`dispatcher.any`

## Microsoft IIS — 配置Dispatcher INI文件{#microsoft-iis-configure-the-dispatcher-ini-file}

编辑`disp_iis.ini`文件以配置Dispatcher安装。 `.ini`文件的基本格式如下：

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

下表描述了每个属性。

| 参数 | 描述 |
|--- |--- |
| configpath | `dispatcher.any`在本地文件系统中的位置（绝对路径）。 |
| 日志文件 | `dispatcher.log`文件的位置。 如果未设置此设置，则日志消息将转到windows事件日志。 |
| loglevel | 定义用于将消息输出到事件日志的日志级别。 可以指定以下值：日志文件的日志级别：<br/>0 — 仅错误消息。 <br/>1 — 错误和警告。<br/>2 — 错误、警告和信息性消 <br/>息3 — 错误、警告、信息性和调试消息。<br/>**注意**:建议在安装和测试期间将日志级别设置为3，在生产环境中运行时将日志级别设置为0。 |
| replaceauthorization | 指定如何处理HTTP请求中的授权标头。 以下值有效：<br/>0 — 未修改授权标头。 <br/>1 — 将名为“Authorization”（“Basic”除外）的任何标头替换为其等 `Basic <IIS:LOGON\_USER>` 效项。<br/> |
| servervariables | 定义如何处理服务器变量。<br/>0 - IIS服务器变量既不会发送到Dispatcher，也不会发送到AEM。<br/>1 — 所有IIS服务器变量(如 `LOGON\_USER, QUERY\_STRING, ...`)连同请求标头一起发送到Dispatcher(如果未缓存，也发送到AEM实例)。<br/>服务器变量包 `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` 括和许多其他变量。有关变量的完整列表（包括详细信息），请参阅IIS文档。 |
| enable_chunked_transfer | 定义是启用(1)还是禁用(0)客户端响应的分块传输。 默认值为 0。 |

示例配置：

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### 配置Microsoft IIS {#configuring-microsoft-iis}

配置IIS以集成Dispatcher ISAPI模块。 在IIS中，使用通配符应用程序映射。

### 配置匿名访问 — IIS 8.5和10 {#configuring-anonymous-access-iis-and}

已配置Author实例上的默认刷新复制代理，以便它不会发送具有刷新请求的安全凭据。 因此，您使用Dispatcher缓存的网站必须允许匿名访问。

如果您的网站使用身份验证方法，则必须相应地配置刷新复制代理。

1. 打开IIS管理器，并选择您用作Disptcher缓存的网站。
1. 使用功能视图模式，在IIS部分中双击身份验证。
1. 如果未启用“匿名身份验证”，请选择“匿名身份验证”，然后在“操作”区域单击“启用”。

### 集成Dispatcher ISAPI模块 — IIS 8.5和10 {#integrating-the-dispatcher-isapi-module-iis-and}

请按照以下过程将Dispatcher ISAPI模块添加到IIS。

1. 打开IIS管理器。
1. 选择您用作调度程序缓存的网站。
1. 使用“功能视图”模式，在IIS部分中双击“处理程序映射”。
1. 在“处理程序映射”页的“操作”面板中，单击添加通配符脚本映射，添加以下属性值，然后单击确定：

   * 请求路径：*
   * 可执行文件：disp_iis.dll文件的绝对路径，例如`C:\inetpub\Scripts\disp_iis.dll`。
   * 名称：处理程序映射的描述性名称，例如`Dispatcher`。

1. 在显示的对话框中，要将disp_iis.dll库添加到ISAPI和CGI限制列表，请单击“是”。

   对于IIS 7.0和7.5，配置已完成。 如果要配置IIS 8.0，请继续执行其余步骤。

1. (IIS 8.0)在处理程序映射列表中，选择刚刚创建的映射，然后在“操作”区域中，单击“编辑”。
1. (IIS 8.0)在“编辑脚本映射”对话框中，单击“请求限制”按钮。
1. (IIS 8.0)要确保处理程序用于尚未缓存的文件和文件夹，请取消选择“仅在请求映射到时调用处理程序”，然后单击“确定”。
1. (IIS 8.0)在“编辑脚本映射”对话框中，单击“确定”。

### 配置对缓存的访问 — IIS 8.5和10 {#configuring-access-to-the-cache-iis-and}

为默认的应用程序池用户提供对用作调度程序缓存的文件夹的写入访问权限。

1. 右键单击用作Dispatcher缓存的网站的根文件夹，然后单击属性，如`C:\inetpub\wwwroot`。
1. 在“安全”选项卡上，单击“编辑”，然后在“权限”对话框中，单击“添加”。 将打开一个用于选择用户帐户的对话框。 单击“位置”按钮，选择您的计算机名称，然后单击“确定”。

   在完成下一步时，保持此对话框处于打开状态。

1. 在IIS管理器中，选择用于Dispatcher缓存的IIS站点，然后在窗口的右侧单击高级设置。
1. 选择“应用程序池”属性的值，并将其复制到剪贴板。
1. 返回打开的对话框。 在输入要选择的对象名称框中，键入`IIS AppPool\`，然后粘贴剪贴板的内容。 该值应类似于以下示例：

   `IIS AppPool\DefaultAppPool`

1. 单击“检查名称”按钮。 Windows解析用户帐户时，单击“确定”。
1. 在Dispatcher文件夹的“权限”对话框中，选择您刚才添加的帐户，启用该帐户&#x200B;**（除“完全控制”**&#x200B;之外）的所有权限，然后单击“确定”。 单击确定以关闭文件夹属性对话框。

### 注册JSON Mime类型 — IIS 8.5和10 {#registering-the-json-mime-type-iis-and}

当您希望Dispatcher允许JSON调用时，请按照以下过程注册JSON MIME类型。

1. 在IIS管理器中，选择您的网站，然后使用“功能视图”，双击“Mime类型”。
1. 如果JSON扩展不在列表中，请在“操作”面板中单击添加，输入以下属性值，然后单击确定：

   * 文件扩展名：`.json`
   * MIME类型：`application/json`

### 删除Bin Hidden Segment - IIS 8.5和10 {#removing-the-bin-hidden-segment-iis-and}

请按照以下过程删除`bin`隐藏的区段。 非新网站可以包含此隐藏区段。

1. 在IIS管理器中，选择您的网站，然后使用“功能视图”，双击“请求过滤”。
1. 选择`bin`区段，单击删除，然后在确认对话框中单击是。

### 将IIS消息记录到文件 — IIS 8.5和10 {#logging-iis-messages-to-a-file-iis-and}

请按照以下过程将Dispatcher日志消息写入日志文件，而不是写入Windows事件日志。 您需要配置Dispatcher以使用日志文件，并为IIS提供对该文件的写访问权限。

1. 使用Windows资源管理器在IIS安装的logs文件夹下创建一个名为`dispatcher`的文件夹。 典型安装的此文件夹路径为`C:\inetpub\logs\dispatcher`。

1. 右键单击调度程序文件夹，然后单击属性。
1. 在“安全”选项卡上，单击“编辑”，然后在“权限”对话框中，单击“添加”。 将打开一个用于选择用户帐户的对话框。 单击“位置”按钮，选择您的计算机名称，然后单击“确定”。

   在完成下一步时，保持此对话框处于打开状态。

1. 在IIS管理器中，选择用于Dispatcher缓存的IIS站点，然后在窗口的右侧单击高级设置。
1. 选择“应用程序池”属性的值，并将其复制到剪贴板。
1. 返回打开的对话框。 在输入要选择的对象名称框中，键入`IIS AppPool\`，然后粘贴剪贴板的内容。 该值应类似于以下示例：

   `IIS AppPool\DefaultAppPool`

1. 单击“检查名称”按钮。 Windows解析用户帐户时，单击“确定”。
1. 在Dispatcher文件夹的“权限”对话框中，选择您刚才添加的帐户，启用该帐户&#x200B;**的所有权限（“完全控制”除外），**&#x200B;并单击“确定”。 单击确定以关闭文件夹属性对话框。
1. 使用文本编辑器打开`disp_iis.ini`文件。
1. 添加一行与以下示例类似的文本以配置日志文件的位置，然后保存文件：

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### 后续步骤{#next-steps}

在开始使用Dispatcher之前，您必须知道：

* [](dispatcher-configuration.md) ConfigureDispatcher
* [配置](page-invalidate.md) AEM以与Dispatcher结合使用。

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>此处介绍了在&#x200B;**Windows**&#x200B;和&#x200B;**Unix**&#x200B;下进行安装的说明。 执行这些步骤时请小心。

### 安装Apache Web Server {#installing-apache-web-server}

有关如何安装Apache Web Server的信息，请阅读安装手册 — [online](https://httpd.apache.org/)或在分发中。

>[!CAUTION]
>
>如果要通过编译源文件来创建Apache二进制文件，请确保打开&#x200B;**动态模块支持**。 可以使用&#x200B;**—enable-shared**&#x200B;选项中的任意选项来完成此操作。 至少包括`mod_so`模块。
>
>有关详细信息，请参阅《Apache Web Server安装手册》。

另请参阅Apache HTTP Server [安全提示](https://httpd.apache.org/docs/2.4/misc/security_tips.html)和[安全报告](https://httpd.apache.org/security_report.html)。

### Apache Web Server — 添加调度程序模块{#apache-web-server-add-the-dispatcher-module}

Dispatcher具有以下任一形式：

* **Windows**:动态链接库(DLL)
* **Unix**:动态共享对象(DSO)

安装存档文件包含以下文件 — 具体取决于您是选择Windows还是Unix:

| 文件 | 描述 |
|--- |--- |
| disp_apache&lt;x.y>.dll | Windows:Dispatcher动态链接库文件。 |
| dispatcher-apache&lt;x.y>-&lt;rel-nr>.so | Unix:Dispatcher共享对象库文件。 |
| mod_dispatcher.so | Unix:示例链接。 |
| http.conf.disp&lt;x> | Apache服务器的示例配置文件。 |
| dispatcher.any | Dispatcher的示例配置文件。 |
| 自述文件 | 自述文件，其中包含安装说明和最后时刻信息。 **注意**:请在开始安装之前检查此文件。 |
| 更改 | 更改文件，其中列出了当前版本和过去版本中修复的问题。 |

按照以下步骤将Dispatcher添加到Apache Web Server:

1. 将Dispatcher文件放在相应的Apache模块目录中：

   * **Windows**:位置  `disp_apache<x.y>.dll` `<APACHE_ROOT>/modules`
   * **Unix**:根据您的 `<APACHE_ROOT>/libexec` 安装 `<APACHE_ROOT>/modules`找到或目录。\
      将`dispatcher-apache<options>.so`复制到此目录中。\
      为简化长期维护，您还可以创建一个名为`mod_dispatcher.so`的符号链接，指向调度程序：\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. 将dispatcher.any文件复制到`<APACHE_ROOT>/conf`目录。

   **注意：** 只要相应地配置了Dispatcher模块的DispatcherLog属性，您就可以将此文件放置到其他位置。（请参阅下面的“特定于调度程序的配置条目”。）

### Apache Web Server — 配置SELinux属性{#apache-web-server-configure-selinux-properties}

如果在启用SELinux的RedHat Linux Kernel 2.6上运行Dispatcher，则可能会在Dispatcher日志文件中遇到类似此类的错误消息。

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

这可能是由于启用了SELinux安全性。 然后，您需要执行以下任务：

* 配置调度程序模块文件的SELinux上下文。
* 启用HTTPD脚本和模块以建立网络连接。
* 配置docroot的SELinux上下文，其中存储了缓存的文件。

在“终端”窗口中输入以下命令，将`[path to the dispatcher.so file]`替换为您安装到Apache Web Server的Dispatcher模块的路径，将&#x200B;*`path to the docroot`*&#x200B;替换为docroot所在的路径(例如，`/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server — 为Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}配置Apache Web Server

需要使用`httpd.conf`配置Apache Web服务器。 在Dispatcher安装工具包中，您将找到一个名为`httpd.conf.disp<x>`的示例配置文件。

这些步骤是强制性的：

1. 导航至 `<APACHE_ROOT>/conf`.
1. 打开`httpd.conf`进行编辑。
1. 必须按所列顺序添加以下配置条目：

   * **** LoadModule在启动时加载模块。
   * 特定于Dispatcher的配置条目，包括&#x200B;**DispatcherConfig、DispatcherLog**&#x200B;和&#x200B;**DispatcherLogLevel**。
   * **** SetHandlerto激活调度程序。**LoadModule**。
   * **** ModMimeUsePathInfo配置mod_mime **的行为**。

1. （可选）建议您更改htdocs目录的所有者：

   * Apache服务器以root身份启动，但子进程以守护程序身份启动（出于安全考虑）。 DocumentRoot(`<APACHE_ROOT>/htdocs`)必须属于用户守护程序：

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

下表列出了可使用的示例；具体条目取决于您特定的Apache Web Server:

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix（假定为符号链接） | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>每个语句的第一个参数必须与上述示例中完全相同地编写。
>
>有关此命令的完整详细信息，请参阅提供的示例配置文件和Apache Web Server文档。

**Dispatcher特定配置条目**

特定于Dispatcher的配置项放在LoadModule项之后。 下表列出了适用于Unix和Windows的示例配置：

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
| DispatcherConfig | Dispatcher配置文件的位置和名称。 <br/>当此属性位于主服务器配置中时，所有虚拟主机都将继承该属性值。但是，虚拟主机可以包含DispatcherConfig属性以覆盖主服务器配置。 |
| DispatcherLog | 日志文件的位置和名称。 |
| DispatcherLogLevel | 日志文件的日志级别：<br/>0 — 错误<br/>1 — 警告<br/>2 — 信息<br/>3 — 调试&#x200B;<br/>**注意**:建议在安装和测试期间将日志级别设置为3，在生产环境中运行时将日志级别设置为0。 |
| DispatcherNoServerHeader | *此参数已弃用，不再有任何效果。*<br/><br/> 定义要使用的服务器标头：  <br/><ul><li>未定义或0 - HTTP服务器标头包含AEM版本。 </li><li>1 — 使用Apache服务器标头。</li></ul> |
| DispatcherDeliseRoot | 定义是否拒绝对根“/”的请求：<br/>**0** — 接受对/ <br/>**1**&#x200B;的请求 — 调度程序处理/未处理的请求；使用mod_alias进行正确映射。 |
| DispatcherUseProcessedURL | 定义是否使用预处理URL进行Dispatcher的所有进一步处理：<br/>**0** — 使用传递到web服务器的原始URL。 <br/>**1**  — 调度程序使用由调度程序之前的处理程序已处理的URL(例如， `mod_rewrite`)，而不是传递到web服务器的原始URL。例如，原始URL或已处理的URL与Dispatcher过滤器相匹配。 URL还用作缓存文件结构的基础。   有关mod_rewrite的信息，请参阅Apache网站文档；例如，Apache 2.4。使用mod_rewrite时，建议使用标志“直通” | PT&#39;（传递给下一个处理程序）强制重写引擎将内部request_rec结构的uri字段设置为filename字段的值。 |
| DispatcherPassError | 定义如何支持ErrorDocument处理的错误代码：<br/>**0** - Dispatcher会将所有错误响应转发到客户端。 <br/>**1**  - Dispatcher不会将错误响应假脱机到客户端（其中状态代码大于或等于400），但会将状态代码传递到Apache，例如，允许ErrorDocument指令处理此类状态代码。<br/>**代码范围**  — 指定将响应传递到Apache的错误代码范围。其他错误代码将传递到客户端。 例如，以下配置将错误412的响应传递给客户端，而所有其他错误都传递给Apache:DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | 指定保持活动状态的超时时间（以秒为单位）。 从Dispatcher版本4.2.0开始，默认的保持活动状态值为60。 值0将禁用保持活动状态。 |
| DispatcherNoCanonURL | 将此参数设置为On会将原始URL传递到后端，而不是规范化的URL，并将覆盖DispatcherUseProcessedURL的设置。 默认值为Off。 <br/>**注意**:Dispatcher配置中的过滤器规则将始终针对已清理的URL而不是原始URL进行评估。 |




>[!NOTE]
>
>路径条目相对于Apache Web Server的根目录。

>[!NOTE]
>
>服务器标题的默认设置为：`  
ServerTokens Full` `  
DispatcherNoServerHeader 0`\
显示AEM版本（用于统计目的）。 如果要禁用此类信息在标题中可用，可以设置：`  
ServerTokens Prod`\
有关更多信息，请参阅[Apache Documentation about ServerTokens Directive（例如，适用于Apache 2.4）](https://httpd.apache.org/docs/2.4/mod/core.html)。

**SetHandler**

在这些条目之后，必须向配置上下文(`<Directory>`、`<Location>`)中添加&#x200B;**SetHandler**&#x200B;语句，以便Dispatcher处理传入请求。 以下示例将Dispatcher配置为处理完整网站的请求：

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

以下示例将Dispatcher配置为处理虚拟域的请求：

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
**SetHandler**&#x200B;语句的参数必须与上述示例&#x200B;*中的参数完全一样编写，因为这是模块中定义的处理程序的名称。*
有关此命令的完整详细信息，请参阅提供的示例配置文件和Apache Web Server文档。

**ModMimeUsePathInfo**

在&#x200B;**SetHandler**&#x200B;语句之后，还应添加&#x200B;**ModMimeUsePathInfo**&#x200B;定义。

>[!NOTE]
仅当您使用的是Dispatcher版本4.0.9或更高版本时，才应使用和配置`ModMimeUsePathInfo`参数。
(请注意，Dispatcher版本4.0.9已于2011年发布。 如果您使用的是旧版本，则升级到最新的Dispatcher版本是合适的)。

对于所有Apache配置，应设置&#x200B;**ModMimeUsePathInfo**&#x200B;参数`On`:

`ModMimeUsePathInfo On`

mod_mime模块（例如， [Apache模块mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)）用于将内容元数据分配给为HTTP响应选择的内容。 默认设置意味着当mod_mime确定内容类型时，将只考虑映射到文件或目录的URL部分。

当`On`时，`ModMimeUsePathInfo`参数指定`mod_mime`将根据&#x200B;*complete* URL确定内容类型；这意味着虚拟资源将根据其扩展应用元信息。

以下示例激活&#x200B;**ModMimeUsePathInfo**:

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

### 启用对HTTPS（Unix和Linux）的支持{#enable-support-for-https-unix-and-linux}

Dispatcher使用OpenSSL通过HTTP实施安全通信。 从Dispatcher版本&#x200B;**4.2.0**&#x200B;开始，支持OpenSSL 1.0.0和OpenSSL 1.0.1。 默认情况下，Dispatcher使用OpenSSL 1.0.0。 要使用OpenSSL 1.0.1，请使用以下过程创建符号链接，以便Dispatcher使用已安装的OpenSSL库。

1. 打开终端，并将当前目录更改为安装OpenSSL库的目录，例如：

   ```shell
   cd /usr/lib64
   ```

1. 要创建符号链接，请输入以下命令：

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
如果您使用的是Apache的自定义版本，请确保Apache和Dispatcher是使用相同版本的[OpenSSL](https://www.openssl.org/source/)编译的。

### 后续步骤{#next-steps-1}

现在，在开始使用Dispatcher之前，您必须：

* [](dispatcher-configuration.md) ConfigureDispatcher
* [配置](page-invalidate.md) AEM以与Dispatcher结合使用。

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
此处介绍了Windows和Unix环境的说明。
选择要执行的时候请务必小心。

### Sun Java System Web Server / iPlanet — 安装Web服务器{#sun-java-system-web-server-iplanet-installing-your-web-server}

有关如何安装这些Web服务器的完整信息，请参阅其各自的文档：

* Sun Java System Web服务器
* iPlanet Web Server

### Sun Java System Web Server / iPlanet — 添加调度程序模块{#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher具有以下任一形式：

* **Windows**:动态链接库(DLL)
* **Unix**:动态共享对象(DSO)

安装存档文件包含以下文件 — 具体取决于您是选择Windows还是Unix:

| 文件 | 描述 |
|---|---|
| `disp_ns.dll` | Windows:Dispatcher动态链接库文件。 |
| `dispatcher.so` | Unix:Dispatcher共享对象库文件。 |
| `dispatcher.so` | Unix:示例链接。 |
| `obj.conf.disp` | iPlanet/Sun Java System Web服务器的示例配置文件。 |
| `dispatcher.any` | Dispatcher的示例配置文件。 |
| 自述文件 | 自述文件，其中包含安装说明和最后时刻信息。 注意：请在开始安装之前检查此文件。 |
| 更改 | 更改文件，其中列出了当前版本和过去版本中修复的问题。 |

请按照以下步骤将Dispatcher添加到Web服务器：

1. 将Dispatcher文件放在Web服务器的`plugin`目录中：

### Sun Java System Web Server / iPlanet — 为调度程序{#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}配置

需要使用`obj.conf`配置Web服务器。 在Dispatcher安装工具包中，您将找到一个名为`obj.conf.disp`的示例配置文件。

1. 导航至 `<WEBSERVER_ROOT>/config`.
1. 打开`obj.conf`进行编辑。
1. 复制开始的行：\
   `Service fn="dispService"`\
   从`obj.conf.disp`到`obj.conf`的初始化部分。

1. 保存更改。
1. 打开`magnus.conf`进行编辑。
1. 复制以下两行开头的代码：\
   `Init funcs="dispService, dispInit"`\
   和\
   `Init fn="dispInit"`\
   从`obj.conf.disp`到`magnus.conf`的初始化部分。

1. 保存更改。

>[!NOTE]
以下配置应全部位于一行中，且`$(SERVER_ROOT)`和`$(PRODUCT_SUBDIR)`必须替换为相应的值。

**初始化**

下表列出了可使用的示例；具体条目是根据您的特定web服务器：

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
| 配置 | 配置文件`dispatcher.any.`的位置和名称 |
| 日志文件 | 日志文件的位置和名称。 |
| loglevel | 将消息写入日志文件时的日志级别：<br/>**0**&#x200B;错误&#x200B;<br/>**1**&#x200B;警告&#x200B;<br/>**2**&#x200B;信息&#x200B;<br/>**3**&#x200B;调试&#x200B;<br/>**注意：**&#x200B;建议在安装和测试期间将日志级别设置为3，在生产环境中运行时将日志级别设置为0。 |
| keepalitimeout | 指定保持活动状态的超时时间（以秒为单位）。 从Dispatcher版本4.2.0开始，默认的保持活动状态值为60。 值0将禁用保持活动状态。 |

根据您的要求，您可以将Dispatcher定义为对象的服务。 要为整个网站配置Dispatcher，请修改默认对象：


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

### 后续步骤{#next-steps-2}

现在，在开始使用Dispatcher之前，您必须：

* [](dispatcher-configuration.md) ConfigureDispatcher
* [配置](page-invalidate.md) AEM以与Dispatcher结合使用。
