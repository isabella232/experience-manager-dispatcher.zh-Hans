---
title: 配置调度程序以防止CSRF攻击
seo-title: 配置Adobe AEM Dispatcher以防止CSRF攻击
description: 了解如何配置AEM Dispatcher以防止跨站点请求伪造攻击。
seo-description: 了解如何配置Adobe AEM Dispatcher以防止跨站点请求伪造攻击。
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: 调度程序
content-type: 引用
discoiquuid: d61d021e-b338-4a1d-91ee-55427557e931
translation-type: tm+mt
source-git-commit: 69edbe7608b46c93d238515e4223606eadad0ac4

---


# 配置调度程序以防止CSRF攻击{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM提供了一个旨在防止跨站点请求伪造攻击的框架。 为了正确利用此框架，您需要对调度程序配置进行以下更改：

>[!NOTE]
>
>请务必根据现有配置更新以下示例中的规则编号。 请记住，调度程序将使用最后一个匹配规则授予允许或拒绝，因此将规则放在现有列表底部附近。

1. 在 `/clientheaders` author-farm.any和publish-farm.any的部分，将以下条目添加到列表底部：\
   `CSRF-Token`
1. 在和／或文件的/filters `author-farm.any` 部分 `publish-farm.any` ，添 `publish-filters.any` 加以下行以允许通过调度程序 `/libs/granite/csrf/token.json` 发出请求。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在您的 `/cache /rules` 部分下，添 `publish-farm.any`加一个规则以阻止调度程序缓存文 `token.json` 件。 通常，作者会绕过缓存，因此您不必将规则添加到您的 `author-farm.any`中。\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

要验证配置是否正在工作，请在“调试”模式中观察dispatcher.log以验证token.json文件是否未被缓存且是否未被过滤器阻止。 您应当看到类似以下内容的消息：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您还可以验证请求在apache中是否成功 `access_log`。 “”的请求应返回HTTP 200状态代码。
