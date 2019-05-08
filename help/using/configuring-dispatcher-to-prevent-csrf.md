---
title: 配置调度程序以防止CSRF攻击
seo-title: 配置Adobe AEM调度程序以防止CSRF攻击
description: 了解如何配置AEM调度程序以防止跨站点请求伪造攻击。
seo-description: 了解如何配置Adobe AEM Dispatcher以防止跨站点请求伪造攻击。
uuid: f290bdeb-54e2-4649-b0 fc-6257b422 af2 d
topic-tags: 调度程序
content-type: 引用
discoiquuid: d61d021e-b338-4a1 d-91ee-55427557e931
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# 配置调度程序以防止CSRF攻击{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM提供了一个旨在防止跨站点请求伪造攻击的框架。为了正确利用此框架，您需要对调度程序配置进行以下更改：

>[!NOTE]
>
>请务必根据现有配置更新以下示例中的规则编号。请记住，调度程序将使用最后一个匹配规则授予允许或拒绝，因此将规则放在现有列表底部附近。

1. 在author-farm. any和publish-farm. any `/clientheaders` 部分中，将以下条目添加到列表底部：\
   `CSRF-Token`
1. 在您 `author-farm.any` 和 `publish-farm.any` 或 `publish-filters.any` 文件的/filters部分中，添加以下行，以允许 `/libs/granite/csrf/token.json` 通过调度程序请求请求。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在您 `/cache /rules``publish-farm.any`的部分下添加一条规则，以阻止调度程序缓存 `token.json` 文件。通常，作者绕过缓存，因此您不需要向您的用户添加规则 `author-farm.any`。\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

要验证配置是否正在工作，请观察dispatcher. log调试模式以验证token. json文件是否未缓存，并且过滤器不会阻止该文件。您应当看到类似于：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您还可以验证请求是否已成功 `access_log`。“/libs/granite/csrf/token. json的请求应返回HTTP200状态代码。
