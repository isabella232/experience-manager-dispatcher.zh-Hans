---
title: 配置 Dispatcher 以防御 CSRF 攻击
seo-title: 配置 Adobe AEM Dispatcher 以防御 CSRF 攻击
description: 了解如何配置 AEM Dispatcher 以防御跨站点请求伪造攻击。
seo-description: 了解如何配置 Adobe AEM Dispatcher 以防御跨站点请求伪造攻击。
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: dispatcher
content-type: reference
discoiquuid: d61d021e-b338-4a1d-91ee-55427557e931
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: ht
source-wordcount: '246'
ht-degree: 100%

---

# 配置 Dispatcher 以防御 CSRF 攻击{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM 提供了一个用于防御跨站点请求伪造攻击的框架。要正确使用此框架，您需要对 Dispatcher 配置进行以下更改：

>[!NOTE]
>
>请务必根据您的现有配置更新以下示例中的规则编号。请记住，Dispatcher 将使用最后一个匹配规则来授予允许或拒绝权限，因此请将规则放置在现有列表的底部附近。

1. 在 author-farm.any 和 publish-farm.any 的 `/clientheaders` 部分中，将以下条目添加到列表的底部：\
   `CSRF-Token`
1. 在 `author-farm.any` 和 `publish-farm.any` 或 `publish-filters.any` 文件的 /filters 部分中，添加以下行以允许通过 Dispatcher 请求 `/libs/granite/csrf/token.json`。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在 `publish-farm.any` 的 `/cache /rules` 部分下，添加规则以阻止 Dispatcher 缓存 `token.json` 文件。通常创作实例会绕过缓存，因此您无需将规则添加到 `author-farm.any` 中。\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

要验证配置是否有效，请在调试模式下查看 dispatcher.log 以验证 token.json 文件是否未缓存且未被过滤器阻止。您应看到与以下内容类似的消息：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您也可以在 Apache `access_log` 中验证请求是否成功。对 ``/libs/granite/csrf/token.json 的请求应返回 HTTP 200 状态代码。
