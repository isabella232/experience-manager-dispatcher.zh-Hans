---
title: 将 SSL 与 Dispatcher 结合使用
seo-title: 将 SSL 与 Dispatcher 结合使用
description: 了解如何配置Dispatcher以使用SSL连接与AEM通信。
seo-description: 了解如何配置Dispatcher以使用SSL连接与AEM通信。
uuid: 1a8f448c-d3d8-4798-a5cb-9579171171ed
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 771cfd85-6c26-4ff2-a3fe-dff8d8f7920b
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: f9fb0e94dbd1c67bf87463570e8b5eddaca11bf3
workflow-type: tm+mt
source-wordcount: '1375'
ht-degree: 0%

---


# 将 SSL 与 Dispatcher 结合使用 {#using-ssl-with-dispatcher}

使用Dispatcher与渲染计算机之间的SSL连接：

* [单向SSL](#use-ssl-when-dispatcher-connects-to-aem)
* [相互SSL](#configuring-mutual-ssl-between-dispatcher-and-aem)

>[!NOTE]
>
>与SSL证书相关的操作绑定到第三方产品。 Adobe白金维护和支持合同未涵盖这些内容。

## 调度程序连接到AEM {#use-ssl-when-dispatcher-connects-to-aem}时使用SSL

将Dispatcher配置为使用SSL连接与AEM或CQ渲染实例通信。

在配置Dispatcher之前，请配置AEM或CQ以使用SSL:

* AEM 6.2:[启用HTTP Over SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1:[启用HTTP Over SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 旧版AEM:请参阅[此页](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)。

### 与SSL相关的请求标头{#ssl-related-request-headers}

当Dispatcher收到HTTPS请求时，Dispatcher在后续请求中包括其发送到AEM或CQ的以下标头：

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

通过`mod_ssl`的Apache-2.4的请求包含的标头与以下示例类似：

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### 将调度程序配置为使用SSL {#configuring-dispatcher-to-use-ssl}

要将Dispatcher配置为通过SSL与AEM或CQ连接，您的[dispatcher.any](dispatcher-configuration.md)文件需要以下属性：

* 处理HTTPS请求的虚拟主机。
* 虚拟主机的`renders`部分包含一个项，它标识使用HTTPS的CQ或AEM实例的主机名和端口。
* `renders`项包含名为`secure`的值`1`的属性。

注意：如果需要，可创建另一个虚拟主机以处理HTTP请求。

以下示例dispatcher.any文件显示了使用HTTPS连接到主机`localhost`和端口`8443`上运行的CQ实例的属性值：

```
/farms
{
   /secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTPS requestss
         "https://*"
      }
      /renders
      {
      /0001
         {
            # hostname or IP of the render
            /hostname "localhost"
            # port of the render
            /port "8443"
            # connect via HTTPS
            /secure "1"
         }
      }
     # the rest of the properties are omitted
   }

   /non-secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTP requests
         "https://*"
      }
      /renders
      {
         /0001
      {
         # hostname or IP of the render
         /hostname "localhost"
         # port of the render
         /port "4503"
      }
   }
    # the rest of the properties are omitted
}
```

## 配置调度程序与AEM {#configuring-mutual-ssl-between-dispatcher-and-aem}之间的相互SSL

配置Dispatcher与渲染计算机(通常为AEM或CQ发布实例)之间的连接，以使用相互SSL:

* 调度程序通过SSL连接到渲染实例。
* 渲染实例验证调度程序证书的有效性。
* 调度程序验证渲染实例的证书的CA是否受信任。
* （可选）调度程序验证渲染实例的证书是否与渲染实例的服务器地址匹配。

要配置相互SSL，您需要由受信任的证书颁发机构(CA)签名的证书。 自签名证书不足。 您可以充当CA，也可以使用第三方CA的服务签署您的证书。 要配置相互SSL，您需要以下项：

* 渲染实例和调度程序的已签名证书
* CA证书（如果您充当CA）
* OpenSSL库，用于生成CA、证书和证书请求。

请执行以下步骤来配置相互SSL:

1. [为](dispatcher-install.md) 您的平台安装最新版本的Dispatcher。使用支持SSL的Dispatcher二进制文件（SSL在文件名中，如dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar）。
1. [为调度程序和渲染实](dispatcher-ssl.md#main-pars-title-3) 例创建或获取CA签名的证书。
1. [创建包含渲染证书](dispatcher-ssl.md#main-pars-title-6) 的密钥库，并配置渲染的HTTP服务以使用它。
1. [配置调度程序Web服务](dispatcher-ssl.md#main-pars-title-4) 器模块以实现相互SSL。

### 创建或获取CA签名证书{#creating-or-obtaining-ca-signed-certificates}

创建或获取CA签名的证书，以验证发布实例和调度程序。

#### 创建CA {#creating-your-ca}

如果您充当CA，请使用[OpenSSL](https://www.openssl.org/)创建对服务器和客户端证书进行签名的证书颁发机构。 （必须安装OpenSSL库。） 如果您使用的是第三方CA，请不要执行此过程。

1. 打开终端，将当前目录更改为包含CA.sh文件的目录，如`/usr/local/ssl/misc`。
1. 要创建CA，请输入以下命令，然后在提示时提供值：

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >openssl.cnf文件中的多个属性控制CA.sh脚本的行为。 在创建CA之前，您应根据需要修改此文件。

#### 创建证书{#creating-the-certificates}

使用OpenSSL创建证书请求以发送到第三方CA或使用CA进行签名。

创建证书时，OpenSSL使用“公用名称”属性来标识证书持有人。 对于渲染实例的证书，如果要将Dispatcher配置为仅在证书与Publish实例的主机名匹配时接受该证书，请使用实例计算机的主机名作为通用名称。 （请参阅[ DispatcherCheckPeerCN](dispatcher-ssl.md#main-pars-title-11)属性。）

1. 打开终端，将当前目录更改为包含OpenSSL库CH.sh文件的目录。
1. 输入以下命令，并在出现提示时提供值。 如果需要，请使用发布实例的主机名作为通用名称。 主机名是渲染器IP地址的DNS可解析名称：

   ```shell
   ./CA.sh -newreq
   ```

   如果您使用第三方CA，请将newreq.pem文件发送给CA进行签名。 如果您是CA，请继续执行步骤3。

1. 输入以下命令，使用CA的证书对证书进行签名：

   ```shell
   ./CA.sh -sign
   ```

   在包含CA管理文件的目录中创建了两个名为newcert.pem和newkey.pem的文件。 这些是渲染计算机的公共证书和私钥。

1. 将newcert.pem重命名为rendercert.pem，将newkey.pem重命名为renderkey.pem。
1. 重复步骤2和3，为调度程序模块创建新证书和新公钥。 确保使用特定于调度程序实例的通用名称。
1. 将newcert.pem重命名为dispcert.pem，将newkey.pem重命名为dispkey.pem。

### 在渲染计算机{#configuring-ssl-on-the-render-computer}上配置SSL

使用rendercert.pem和renderkey.pem文件在render实例上配置SSL。

#### 将渲染证书转换为JKS格式{#converting-the-render-certificate-to-jks-format}

使用以下命令将渲染证书（即PEM文件）转换为PKCS#12文件。 还包括对渲染证书进行签名的CA的证书：

1. 在终端窗口中，将当前目录更改为渲染证书和私钥的位置。
1. 输入以下命令，将渲染证书（PEM文件）转换为PKCS#12文件。 还包括对渲染证书进行签名的CA的证书：

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. 输入以下命令将PKCS#12文件转换为Java KeyStore(JKS)格式：

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. Java密钥库是使用默认别名创建的。 根据需要更改别名：

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### 将CA证书添加到Render的Truststore {#adding-the-ca-cert-to-the-render-s-truststore}

如果您充当CA，请将您的CA证书导入密钥库。 然后，配置运行渲染实例的JVM以信任密钥库。

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2014-08-12T13:11:21.401-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The jetty http service has properties to specify trusted CA certificates for mutual SSL for 6.0. Whether they are operable is undetetermined. See https://issues.adobe.com/browse/DOC-4738.</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;"> </p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For 5.6.1, you would specify the system property javax.net.ssl.trustStore, using the path to cacerts as value.</p>

 -->

1. 使用文本编辑器打开cacert.pem文件并删除下一行之前的所有文本：

   `-----BEGIN CERTIFICATE-----`

1. 使用以下命令将证书导入密钥库：

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. 要配置运行渲染实例的JVM以信任密钥库，请使用以下系统属性：

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   例如，如果使用crx-quickstart/bin/quickstart脚本开始发布实例，则可以修改CQ_JVM_OPTS属性：

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### 配置渲染实例{#configuring-the-render-instance}

使用渲染证书和“在发布实例&#x200B;*上启用SSL”部分中的说明，将渲染实例的HTTP服务配置为使用SSL:*

* AEM 6.2:[启用HTTP Over SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1:[启用HTTP Over SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 旧版AEM:请参阅[此页。](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)

### 为调度程序模块{#configuring-ssl-for-the-dispatcher-module}配置SSL

要将Dispatcher配置为使用相互SSL，请准备Dispatcher证书，然后配置Web服务器模块。

### 创建统一调度程序证书{#creating-a-unified-dispatcher-certificate}

将调度程序证书和未加密的私钥合并到单个PEM文件中。 使用文本编辑器或`cat`命令创建与以下示例类似的文件：

1. 打开终端，将当前目录更改为dispkey.pem文件的位置。
1. 要解密私钥，请输入以下命令：

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. 使用文本编辑器或`cat`命令将未加密的私钥和证书组合到一个文件中，该文件类似于以下示例：

   ```xml
   -----BEGIN RSA PRIVATE KEY-----
   MIICxjBABgkqhkiG9w0B...
   ...M2HWhDn5ywJsX
   -----END RSA PRIVATE KEY-----
   -----BEGIN CERTIFICATE-----
   MIIC3TCCAk...
   ...roZAs=
   -----END CERTIFICATE-----
   ```

### 指定用于调度程序{#specifying-the-certificate-to-use-for-dispatcher}的证书

将以下属性添加到[调度程序模块配置](dispatcher-install.md#main-pars-55-35-1022)（在`httpd.conf`中）:

* `DispatcherCertificateFile`:指向Dispatcher统一证书文件的路径，包含公共证书和未加密的私钥。当SSL服务器请求调度程序客户端证书时，将使用此文件。
* `DispatcherCACertificateFile`:CA证书文件的路径，如果SSL服务器显示的CA不受根颁发机构信任，则使用该路径。
* `DispatcherCheckPeerCN`:是启用() `On`还是禁用() `Off`主机名检查远程服务器证书。

以下代码是一个配置示例：

```xml
<IfModule disp_apache2.c>
  DispatcherConfig conf/dispatcher.any
  DispatcherLog    logs/dispatcher.log
  DispatcherLogLevel 3
  DispatcherNoServerHeader 0
  DispatcherDeclineRoot 0
  DispatcherUseProcessedURL 0
  DispatcherPassError 0
  DispatcherCertificateFile disp_unified.pem
  DispatcherCACertificateFile cacert.pem
  DispatcherCheckPeerCN On
</IfModule>
```

