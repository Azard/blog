---
title: 实战：Node后端，从session迁移到OAuth2
date: 2017-03-13 20:59:54
categories: 后端
tags: [Node.js,后端,OAuth2,session]
---
最近一周，在开发 Appetizer 的后端新业务的过程中，需要提供几个开放 API 供自己团队的客户端调用，未来也许会让第三方团队进行调用。Appetizer 后端的 API 层我取名叫 eevee，口袋妖怪里的伊布的英文名，承载了整个账号系统、账号业务页面以及一些账号和图片服务的 API，使用 Express 4.0 和 MongoDB 实现的业务逻辑不是特别复杂的后端应用，主要的接入应用是能够记录 session 的客户端浏览器。

<!--more-->

# 起因
问题在于，我们基于 Electron 实现的 WebApp，服务端不能够很好的通过传统的 session 来保存 WebApp 的状态，因为每次重新启动的 Electron 客户端的 coockie 都会发生变化，导致服务端无法识别对应的 session。因此决定增加一层 OAuth 2.0，浏览器访问的页面使用基于 session 的 API 保存用户登录状态，其他客户端（Electron，Python）访问的 API 使用 OAuth 2.0 权限认证。这么实现有两个好处：
* 加速不需要 session 的 API。
* 客户端自行管理 token，减轻服务端压力，客户端实现根据具体业务变化更方便。
* 便于将来可能的第三方厂商接入。

# 改造方案
我用到了 [node-oauth2-server](https://github.com/oauthjs/node-oauth2-server) ，是一个使用 node 实现的 oauth2 服务端接口规范，具体的存储和规则还需自行根据业务完善。根据阮一峰 OAuth2 介绍博客描述的4种 OAuth2 方案模型，结合 Appetizer 的需求，我进行了如下改造：
* 在 route 上区分使用 OAuth2 验证权限、使用 session 判断状态、不验证任何权限的三种 API，通过 Express 的路由中间件可以很方便的进行扩展区分。
* 当 token 被使用时，自动续期，类似 session 的自动续期机制，同样通过路由中间件实现。
* 提供一个基于 OAuth2 密码模型的 API。在实际业务中，我只将该 API 提供给我们团队开发的 Python 客户端进行使用，在客户端硬编码保存了 clientId 和 clientKey，用户在客户端输入账号密码，客户端作为代理进行请求获取 access_token。这个模型需要用户充分相信客户端不会钓鱼保存密码。
* 提供一个基于 OAuth2 变种客户端模型的 web 页面给 WebApp。在之前的 WebApp 登录过程中，支持 Appetizer 原生账号和 GitHub 第三方账号登录，web 页面相当于我们除了 OAuth2 服务器之外的账号验证服务器的代理页面，首先像浏览器一样实现正常的账号密码验证或者 GitHub 第三方账号登录，然后服务端获取账号的具体信息，代理请求 OAuth2 服务器授权，请求授权的过程是在 clientKey 中保存登录账号的信息，OAuth2 根据该信息检索验证账号的正确性给予授权。

上述的第四个改造，实际上是客户端模型的一种变种，先登录然后再请求 OAuth2，比较魔改。