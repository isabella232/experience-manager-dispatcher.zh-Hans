---
title: 配置 Dispatcher 以防御 CSRF 攻击
seo-title: 配置AdobeAEM Dispatcher以防止CSRF攻击
description: 了解如何配置AEM Dispatcher以防止跨站点请求伪造攻击。
seo-description: 了解如何配置AdobeAEM Dispatcher以防止跨站点请求伪造攻击。
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: dispatcher
content-type: reference
discoiquuid: d61d021e-b338-4a1d-91ee-55427557e931
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '246'
ht-degree: 4%

---

# 配置 Dispatcher 以防御 CSRF 攻击{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM提供了一个旨在防止跨站点请求伪造攻击的框架。 要正确使用此框架，您需要对Dispatcher配置进行以下更改：

>[!NOTE]
>
>请务必根据您的现有配置更新以下示例中的规则号。 请记住，调度程序将使用最后一个匹配规则来授予允许或拒绝，因此请将规则放在现有列表底部附近。

1. 在author-farm.any和publish-farm.any的`/clientheaders`部分中，将以下条目添加到列表底部：\
   `CSRF-Token`
1. 在`author-farm.any`和`publish-farm.any`或`publish-filters.any`文件的/filters部分中，添加以下行以允许通过调度程序发出`/libs/granite/csrf/token.json`请求。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在`publish-farm.any`的`/cache /rules`部分下，添加规则以阻止调度程序缓存`token.json`文件。 通常，作者会绕过缓存，因此您不应该将规则添加到`author-farm.any`中。\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

要验证配置是否正常工作，请在调试模式下观看dispatcher.log ，以验证token.json文件是否未缓存且过滤器未阻止。 您应会看到类似以下内容的消息：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您还可以验证请求是否在Apache `access_log`中成功。 对“/libs/granite/csrf/token.json”的请求应返回一个HTTP 200状态代码。
