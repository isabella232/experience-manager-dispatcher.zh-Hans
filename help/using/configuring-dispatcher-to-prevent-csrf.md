---
title: 配置 Dispatcher 以防御 CSRF 攻击
seo-title: 配置AdobeAEM调度程序以防止CSRF攻击
description: 了解如何配置AEM Dispatcher以防止跨站点请求伪造攻击。
seo-description: 了解如何配置AdobeAEM Dispatcher以防止跨站点请求伪造攻击。
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: dispatcher
content-type: reference
discoiquuid: d61d021e-b338-4a1d-91ee-55427557e931
translation-type: tm+mt
source-git-commit: 69edbe7608b46c93d238515e4223606eadad0ac4
workflow-type: tm+mt
source-wordcount: '246'
ht-degree: 4%

---


# 配置 Dispatcher 以防御 CSRF 攻击{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM提供了一个旨在防止跨站点请求伪造攻击的框架。 为了正确利用此框架，您需要对调度程序配置进行以下更改：

>[!NOTE]
>
>请务必根据现有配置更新以下示例中的规则编号。 请记住，调度程序将使用最后一个匹配规则授予允许或拒绝，因此将规则放在现有列表的底部附近。

1. 在author-farm.any和publish-farm.any的`/clientheaders`部分，在列表底部添加以下条目：\
   `CSRF-Token`
1. 在`author-farm.any`和`publish-farm.any`或`publish-filters.any`文件的/过滤器部分，添加以下行以允许通过调度程序请求`/libs/granite/csrf/token.json`。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在`publish-farm.any`的`/cache /rules`部分下，添加一个规则以阻止调度程序缓存`token.json`文件。 通常作者会绕过缓存，因此您不必将规则添加到`author-farm.any`中。\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

要验证配置是否正在工作，请在DEBUG模式下观察dispatcher.log以验证token.json文件是否未缓存且未被过滤器阻止。 您应当看到类似以下内容的消息：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您还可以验证apache `access_log`中的请求是否成功。 “/libs/granite/csrf/token.json”的请求应返回HTTP 200状态代码。
