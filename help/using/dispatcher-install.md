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
source-git-commit: ec5342f5737f54937493edb0384cdc1d91b13d7c

---


# 安装Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

>[!NOTE]
>
>Dispatcher版本独立于AEM。如果您遵循了一个指向Dispatcher文档的链接，则可能已重定向到该页面，该链接嵌入到AEM的先前版本的文档中。

使用 [Dispatcher发行说明](release-notes.md) 页面获取操作系统和Web服务器的最新Dispatcher安装文件。Dispatcher发行版编号与Adobe Experience Manager发布号码无关，与Adobe Experience Manager6.x、5.x和Adobe CQ5.x版本兼容。

使用以下文件命名规范：

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

例如，该 `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` 文件包含用于在Linux i686上运行的Apache2.4Web服务器的Dispatcher4.3.1，并使用 **tar** 格式进行打包。

下表列出了每个Web服务器的文件名中使用的Web服务器标识符：

| Web 服务器 | 安装套件 |
|--- |--- |
| Apache2.4 | dispatcher-apache **** 2.4-&lt;其他参数&gt; |
| Apache2.2 | dispatcher-apache **** 2.2-&lt;其他参数&gt; |
| Microsoft Internet Information Server7.5、8、8.5 | dispatcher-**** iis-&lt;其他参数&gt; |
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

### 必需IIS组件 {#required-iis-components}

IIS版本8.5和10要求安装以下IIS组件：

* ISAPI扩展

此外，您必须添加Web服务器(IIS)角色。使用服务器管理器添加角色和组件。

## Microsoft IIS-安装调度程序模块 {#microsoft-iis-installing-the-dispatcher-module}

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

1. 例如，使用Windows资源管理器创建 `<IIS_INSTALLDIR>/Scripts` 目录 `C:\inetpub\Scripts`。

1. 将以下文件从调度程序包解压到此Scripts目录中：

   * `disp_iis.dll`
   * `disp_iis.ini`
   * 以下文件之一，具体取决于Dispatcher在使用AEM作者实例或发布实例：
      * 作者实例： `author_dispatcher.any`
      * 发布实例： `dispatcher.any`

## Microsoft IIS-配置调度程序尼尼文件 {#microsoft-iis-configure-the-dispatcher-ini-file}

编辑 `disp_iis.ini` 文件以配置Dispatcher安装。`.ini` 文件的基本格式如下所示：

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
| configpath | 本地文件系统 `dispatcher.any` 内的位置(绝对路径)。 |
| logfile | `dispatcher.log` 文件的位置。如果未设置此设置，则日志消息将转到窗口事件日志。 |
| loglevel | 定义用于向活动日志输出消息的日志级别。可以指定以下值：日志文件的日志级别： <br/>-仅限错误消息。<br/>1 - 错误和警告。<br/>2 - 错误、警告和信息性消息 <br/>3-错误、警告、信息性和调试消息。<br/>**注意**：建议在安装和测试过程中将日志级别设置为3，然后在生产环境中运行时将日志级别设置为0。 |
| 替换授权 | 指定如何处理HTTP请求中的授权头。以下值为有效：<br/>-未修改授权标题。<br/>1 - 替换任何名为“Authorization”的标题，但其 `Basic <IIS:LOGON\_USER>` 等效项除外。<br/> |
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

### 配置Microsoft IIS {#configuring-microsoft-iis}

配置IIS以集成Dispatcher ISAPI模块。在IIS中，您使用通配符应用程序映射。

### 配置匿名访问- IIS8.5和10 {#configuring-anonymous-access-iis-and}

对作者实例上的默认刷新复制代理进行配置，以便它不发送具有刷新请求的安全凭据。因此，您使用调度程序缓存的网站必须允许匿名访问。

如果您的网站使用身份验证方法，则必须相应地配置刷新复制代理。

1. 打开IIS Manager，然后选择您用作“清除程序”缓存的网站。
1. 使用功能视图模式，在IIS部分中双击身份验证。
1. 如果未启用匿名身份验证，则选择“匿名身份验证”，在“操作”区域中单击“启用”。

### 集成Dispatcher IAPI模块- IIS8.5和10 {#integrating-the-dispatcher-isapi-module-iis-and}

请按照以下过程将Dispatcher ISAPI模块添加到IIS。

1. 打开IIS Manager。
1. 选择您用作调度程序缓存的网站。
1. 使用功能视图模式，在IIS部分中双击处理程序映射。
1. 在“处理程序映射”页面的“操作”面板中，单击“添加通配符脚本映射”，添加以下属性值，然后单击确定：

   * 请求路径：*
   * 可执行文件：disp_ iis. dll文件的绝对路径 `C:\inetpub\Scripts\disp_iis.dll`。
   * 名称：处理函数映射的描述性名称 `Dispatcher`。

1. 在出现的对话框中，要将disp_ iis. dll库添加到ISAPI和CGI限制列表中，请单击“是”。

   对于IIS7.0和7.5，配置已完成。如果配置IIS8.0，请继续执行其余步骤。

1. (IIS8.0)在处理函数映射列表中，选择刚才创建的，在“操作”区域中单击编辑。
1. (IIS8.0)在“编辑脚本映射”对话框中，单击“请求限制”按钮。
1. (IIS8.0)要确保处理函数用于尚未缓存的文件和文件夹，请取消选择“仅当请求已映射到”，然后单击“确定”即可取消选中“调用处理程序”。
1. (IIS8.0)在“编辑脚本映射”对话框上，单击“确定”。

### 配置对缓存的访问- IIS8.5和10 {#configuring-access-to-the-cache-iis-and}

为默认应用程序池用户提供对用作调度程序缓存的文件夹的写入访问权限。

1. Right-click the root folder of the website that you are using as the Dispatcher cache and click Properties, such as `C:\inetpub\wwwroot`.
1. 在安全选项卡上，单击编辑，然后在“权限”对话框中单击添加。此时将打开一个对话框，用于选择用户帐户。单击“位置”按钮，选择计算机名称，然后单击“确定”。

   完成下一步后，将此对话框保持打开状态。

1. 在IIS Manager中，选择您正在用于调度程序缓存的IIS站点，然后在窗口的右侧单击高级设置。
1. 选择“应用程序池”属性的值并将其复制到剪贴板。
1. 返回打开对话框。在“输入对象名称要选择”框中，键入 `IIS AppPool\` 并粘贴剪贴板的内容。该值应类似于以下示例：

   `IIS AppPool\DefaultAppPool`

1. 单击“复选名称”按钮。当Windows解析用户帐户时，单击“确定”。
1. 在调度程序文件夹的“权限”对话框中，选择刚刚添加的帐户，启用帐户的所有权限 **，除非完全控制** ，然后单击确定。单击确定以关闭文件夹属性对话框。

### 注册JSON MIME类型- IIS8.5和10 {#registering-the-json-mime-type-iis-and}

当您希望Dispatcher允许JSON调用时，请使用以下过程注册JSON MIME类型。

1. 在IIS Manager中，选择您的网站并使用功能视图，双击MIME类型。
1. 如果JSON扩展不在列表中，请在“操作”面板中单击“添加”，输入以下属性值，然后单击确定：

   * 文件名扩展： `.json`
   * MIME类型： `application/json`

### 删除bin隐藏区段- IIS8.5和10 {#removing-the-bin-hidden-segment-iis-and}

请按照以下过程删除 `bin` 隐藏区段。不是新的Web站点可以包含此隐藏区段。

1. 在IIS Manager中，选择您的网站并使用功能视图，双击请求过滤。
1. 选择 `bin` 区段，单击“删除”，在确认对话框中单击“是”。

### 将IIS消息记录到文件- IIS8.5和10 {#logging-iis-messages-to-a-file-iis-and}

请按照以下过程将调度程序日志消息写入日志文件而非Windows活动日志。您需要配置Dispatcher以使用日志文件，并提供对文件的写访问权限。

1. 使用Windows资源管理器创建一个名称为 `dispatcher` IIS安装的日志文件夹下方的文件夹。此文件夹的路径为典型安装 `C:\inetpub\logs\dispatcher`。

1. 右键单击调度程序文件夹，然后单击“属性”。
1. 在安全选项卡上，单击编辑，然后在“权限”对话框中单击添加。此时将打开一个对话框，用于选择用户帐户。单击“位置”按钮，选择计算机名称，然后单击“确定”。

   完成下一步后，将此对话框保持打开状态。

1. 在IIS Manager中，选择您正在用于调度程序缓存的IIS站点，然后在窗口的右侧单击高级设置。
1. 选择“应用程序池”属性的值并将其复制到剪贴板。
1. 返回打开对话框。在“输入对象名称要选择”框中，键入 `IIS AppPool\` 并粘贴剪贴板的内容。该值应类似于以下示例：

   `IIS AppPool\DefaultAppPool`

1. 单击“复选名称”按钮。当Windows解析用户帐户时，单击“确定”。
1. 在调度程序文件夹的“权限”对话框中，选择刚刚添加的帐户，支持帐户**的所有权限**但完全控制，**并单击确定。单击确定以关闭文件夹属性对话框。
1. 使用文本编辑器打开 `disp_iis.ini` 文件。
1. 添加类似于以下示例的文本行，以配置日志文件的位置，然后保存文件：

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### 后续步骤 {#next-steps}

在开始使用Dispatcher之前，您必须了解：

* [配置](dispatcher-configuration.md) 调度程序
* [混淆AEM](page-invalidate.md) 以使用Dispatcher。

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>此处介绍 **了有关Windows** 和 **Unix** 下安装的说明。执行步骤时请务必小心。

### 安装Apache Web Server {#installing-apache-web-server}

有关如何安装Apache Web Server的信息，请阅读安装手册- [联机](https://httpd.apache.org/) 或分发。

>[!CAUTION]
>
>如果要通过编译源文件创建Apache二进制文件，请确保打开 **动态模块支持**。This can can be done by use of **—启用共享** 选项。至少包括 `mod_so` 模块。
>
>有关更多信息，请参阅Apache Web Server安装手册。

另请参阅Apache HTTP Server [安全提示](https://httpd.apache.org/docs/2.2/misc/security_tips.html) 和 [安全报告](https://httpd.apache.org/security_report.html)。

### Apache Web Server-添加调度程序模块 {#apache-web-server-add-the-dispatcher-module}

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
      复制 `dispatcher-apache<options>.so` 到此目录中。\
      为了简化长期维护，您还可以创建一个名为 `mod_dispatcher.so` Dispatcher的符号链接：\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. 将调度程序复制到 `<APACHE_ROOT>/conf` 目录中。

   **注意：** 只要相应地配置Dispatcher模块的dispatcherLog属性，您可以将此文件放置在其他位置。(请参阅下面的调度程序特定配置条目。)

### Apache Web Server-配置SERLinux属性 {#apache-web-server-configure-selinux-properties}

如果您在启用SERINX的RedHat Linux内核2.6上运行Dispatcher，您可能会在调度程序日志文件中运行错误消息。

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

这可能是由于启用了SERInX安全性的。然后您需要执行以下任务：

* 配置调度程序模块文件的SerialUX上下文。
* 启用HTPD脚本和模块以建立网络连接。
* 配置docroot的SerialUX上下文，其中存储缓存的文件。

在终端窗口中输入以下命令，用 `[path to the dispatcher.so file]` 您安装到Apache Web *`path to the docroot`* Server的Dispatcher模块的路径以及docroot所在的路径(例如， `/opt/cq/cache`)进行替换：

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server-配置Apache Web Server for Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

需要配置Apache Web Server `httpd.conf`。在Dispatcher安装包中，您将找到一个名为配置文件的示例配置文件 `httpd.conf.disp<x>`。

这些步骤是强制性的：

1. 导航至 `<APACHE_ROOT>/conf`.
1. 打开 `httpd.conf`以进行编辑。
1. 必须按列出的顺序添加以下配置条目：

   * **加载模块** 以在启动时加载模块。
   * 调度程序特定的配置条目，包括 **DispatcherConfig、dispatcherLog** 和 **dispatcherLogLevel**。
   * **setHandler** 以激活Dispatcher。**LoadModule**.
   * **ModmMeusePateInfo** 配置 **mod_ mime的行为**。

1. (可选)建议您更改htdocs目录的所有者：

   * apache服务器以root开头，但子进程作为守护程序开始(出于安全目的)。documentRoot(`<APACHE_ROOT>/htdocs`)必须属于用户守护程序：

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

下表列出了可使用的示例；具体条目取决于特定Apache Web Server：

|  |
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
| DispatcherLogLevel | 日志文件的日志级别： <br/>0-错误 <br/>-警告 <br/>-信息说明 <br/>-调试 <br/>**注意**事项：建议在安装和测试过程中将日志级别设置为3，然后在生产环境中运行时将日志级别设置为0。 |
| DispatcherNoServerHeader | *此参数已弃用，不再具有任何效果。*<br/><br/> 定义要使用的服务器头： <br/><ul><li>未定义或0- HTTP服务器头包含AEM版本。 </li><li>1 - 使用Apache服务器头。</li></ul> |
| DispatcherDeclineRoot | 定义是否拒绝对根“/”的请求： <br/>**0** -接受请求到/ <br/>**1** -请求不由调度程序处理；使用mod_ alias实现正确映射。 |
| DispatcherUseProcessSubURL | 定义是否使用预先处理的URL进行Dispatcher进一步处理： <br/>**-** 使用传递给Web服务器的原始URL。<br/>**** -调度程序使用已经由调度程序处理的URL(即调度 `mod_rewrite`程序)之前的URL，而不是传递给Web服务器的原始URL。例如，原始或处理的URL与调度程序过滤器相匹配。URL还用作缓存文件结构的基础。有关mod_ rewrite的信息，请参阅Apache网站文档；例如Apache2.2。使用mod_ rewrite时，使用标记“passhrough”是建议使用 | PT&#39;(传递到下一个处理函数)以强制重写引擎将internal request_ rec结构的uri字段设置为文件名字段的值。 |
| DispatcherPassError | 定义如何为ErdRockocument处理支持错误代码： <br/>**** -调度程序将所有错误答复发送给客户端。<br/>**** -调度程序不会向客户端发送错误响应(状态代码大于或等于400)，但将状态代码传递给Apache，如允许ErdRockocument指令处理此类状态代码。<br/>**代码范围** -指定将响应传递给Apache的一系列错误代码。其他错误代码将传递给客户端。例如，以下配置将错误412的响应传递给客户端，所有其他错误将传递给Apache：DispatcherPassServ400-411,413-417 |
| DispatcherVillAliveTimeout | 指定持续时间超时，以秒为单位。从调度程序4.2.0开始，默认的保留活动量为60。值为可禁用保持活动量。 |
| dispatcherClyanOnURL | 将此参数设置为On会将原始URL传递给后端而不是画布图标，将覆盖DispatcherUseProcessSubeURL的设置。默认值为关闭。<br/>**注意**：Dispatcher配置中的过滤器规则始终会根据不是原始URL的清理URL进行评估。 |




>[!NOTE]
>
>路径条目相对于Apache Web Server的根目录。

>[!NOTE]
>
>服务器头的默认设置为： `  
ServerTokens Full``  
DispatcherNoServerHeader 0`\
其中显示AEM版本(用于统计目的)。如果您希望禁用标题中可用的此类信息，则可以设置： `  
ServerTokens Prod`\
有关详细信息，请参阅有关ServerTokens指令 [的Apache文档(例如，Apache2.2)](https://httpd.apache.org/docs/2.2/mod/core.html) 。

**SetHandler**

这些条目完成后，您必须将 **setHandler** 语句添加到您的配置( `<Directory>`， `<Location>`)的上下文中以处理传入请求。以下示例配置Dispatcher以处理完整网站的请求：

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

在 **setHandler** 语句之后，您还应添加 **ModmMeuseUseInfo** 定义。

>[!NOTE]
`ModMimeUsePathInfo` 仅当您使用Dispatcher4.0.9或更高版本时，才应使用和配置参数。
(注意，Dispatcher版本4.0.9已于2011年发布。如果您使用的是旧版本，则升级到最近的Dispatcher版本将非常合适。

应该为所有Apache配置设置 **modmimeUsePatInfo**`On` 参数：

`ModMimeUsePathInfo On`

mod_ mime模块(请参阅 [Apache Module mod_ mime](https://httpd.apache.org/docs/2.2/mod/mod_mime.html))用于将内容元数据分配给为HTTP响应选择的内容。默认设置意味着当mod_ mime确定内容类型时，只会考虑映射到文件或目录的URL部分。

参数时 `On`， `ModMimeUsePathInfo` 参数指定 `mod_mime` 基于 *完整* URL确定内容类型；这意味着虚拟资源将根据其扩展名应用布局。

以下示例激活 **ModmMeuseUseInfo**：

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

### 支持HTTPS支持(UNIX和Linux) {#enable-support-for-https-unix-and-linux}

Dispatcher使用OpenSSL实现通过HTTP实现安全通信。支持从调度程序 **4.2.0**、OpenSSL1.0.0和OpenSSL1.0.1开始。默认情况下，Dispatcher使用OpenSSL1.0.0。要使用OpenSSL1.0.1，请使用以下过程创建符号链接，以便Dispatcher使用已安装的OpenSSL库。

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
如果您使用的是Apache的自定义版本，请确保Apache和Dispatcher使用相同版本 [的OpenSSL进行编译](https://www.openssl.org/source/)。

### 后续步骤 {#next-steps-1}

在开始使用调度程序之前，您必须现在：

* [配置](dispatcher-configuration.md) 调度程序
* [混淆AEM](page-invalidate.md) 以使用Dispatcher。

## Sun Java System Web Server/iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
此处介绍了适用于Windows和Unix环境的说明。
请谨慎选择要执行的操作。

### Sun Java System Web Server/iPlanet-安装Web服务器 {#sun-java-system-web-server-iplanet-installing-your-web-server}

有关如何安装这些Web服务器的完整信息，请参阅其各自的文档：

* Sun Java System Web Server
* iPlanet Web Server

### Sun Java System Web Server/iPlanet-添加调度程序模块 {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

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

1. 将Dispatcher文件放入Web服务器 `plugin` 的目录中：

### Sun Java System Web Server/iPlanet-配置Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

需要使用Web服务器进行配置 `obj.conf`。在Dispatcher安装包中，您将找到一个名为配置文件的示例配置文件 `obj.conf.disp`。

1. 导航至 `<WEBSERVER_ROOT>/config`.
1. 打开 `obj.conf`以进行编辑。
1. 复制开始的行：\
   `Service fn="dispService"`\
   `obj.conf.disp` 从到的初始化部分 `obj.conf`。

1. 保存更改。
1. 打开 `magnus.conf` 以进行编辑。
1. 复制开始的两行：\
   `Init funcs="dispService, dispInit"`\
   和\
   `Init fn="dispInit"`\
   `obj.conf.disp` 从到的初始化部分 `magnus.conf`。

1. 保存更改。

>[!NOTE]
以下配置应全部位于一行和一行中，并且 `$(SERVER_ROOT)``$(PRODUCT_SUBDIR)` 必须由相应的值替换。

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
| config | 配置文件的位置和名称 `dispatcher.any.` |
| logfile | 日志文件的位置和名称。 |
| loglevel | 将消息写入日志文件时的日志级别： <br/>**** 0Errors <br/>**** Warnings <br/>**** Infos <br/>**** 3调试 <br/>**注意事项：** 建议在安装和测试过程中将日志级别设置为3，在生产环境中运行时将日志级别设置为0。 |
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

### 后续步骤 {#next-steps-2}

在开始使用调度程序之前，您必须现在：

* [配置](dispatcher-configuration.md) 调度程序
* [混淆AEM](page-invalidate.md) 以使用Dispatcher。
