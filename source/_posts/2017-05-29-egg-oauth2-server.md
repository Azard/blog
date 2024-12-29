---
title: 使用 Egg 快速开发 OAuth 2.0 授权服务
date: 2017-05-29 03:38:00
categories: 后端
tags: [Node.js,后端,OAuth2,Egg.js]
---
# 前言
随着移动互联网的发展，授权协议从 OAuth 1.0 过渡到了 OAuth 2.0，新版授权协议的草案早在 2011 年就已公布，现在已经广泛应用于移动客户端的登录和网站、客户端的第三方授权。相比于会话（session），OAuth 2.0 不关注用户状态，主要用于无状态的 API 和非浏览器的移动客户端。

本文通过实例介绍如何使用 [Egg.js](https://eggjs.org/) 框架和相关插件 [egg-oauth2-server](https://github.com/Azard/egg-oauth2-server)，快速开发 OAuth 2.0 协议的授权服务。

<!--more-->

# 用法
OAuth 2.0 的完整定义较为复杂，协议名为 [RFC 6749](http://www.rfcreader.com/#rfc6749)。

简单来说，就是客户端通过账号密码或者某个万能密码，请求服务端，得到一个在一段时间内有效的令牌（token），之后客户端可以通过在 HTTP 请求的 header 上添加令牌，访问服务端需要授权验证的 API。

再简单点说，就是一次登录，一段时间内有效，而且服务端不需要保存客户端已登录的状态。

如果使用过 Express 或者 Koa 框架，一定接触过在路由（Route）中加入中间件（Middleware）。如果开发好了 OAuth2.0 授权服务，使用中间件在 Egg.js 框架路由中对 API 要求授权验证会非常简单。

```javascript
// {app_root}/app/route.js
app.get('/public/get', 'gold.get');

app.post('/oauth2/access_token', app.oauth.grant());
app.get('/private/get', app.oauth.authorise(), 'gold.get');
```

第一行路由定义的 API `/public/get` 未作任何权限保护，一个不带任何 header 和参数的 GET 请求就能得到 `gold.get` 的返回结果。

第二行路由定义了 API `/oauth2/access_token` ，在 header 里添加相关授权登录信息请求该 API，服务器会返回 token，客户端保存该 token 用于请求需要授权验证的 API。

第三行路由定义的 API `/private/get` 由于添加了中间件 `app.oauth.authorise()` ，会在执行 `gold.get` 前进行 OAuth 2.0 权限验证，如果不是在 GET 请求的 header 里没带正确且在合法时间的令牌（token），则会提前返回 `400` 或者 `401` 错误码。

# 实现 OAuth 2.0 授权服务

独立完整的实现 OAuth 2.0 授权协议步骤繁杂，通过 `egg-oauth2-server` 提供的简化 API，只需要实现关键业务功能，就能开发多种模式的 OAuth 2.0 授权服务。

## 配置
首先在 Egg.js 项目目录里安装 `egg-oauth2-server` 插件。
``` shell
$ npm i egg-oauth2-server --save
```

在项目插件配置文件里开启插件。
``` javascript
// {app_root}/config/plugin.js
exports.oauth2Server = {
  enable: true,
  package: 'egg-oauth2-server',
};
```

然后再在项目的配置文件里，选择需要启用的 OAuth 2.0 模式。其他相关配置，例如 token 有效时长，debug 信息，参考[插件项目说明](https://github.com/Azard/egg-oauth2-server)。
``` javascript
// {app_root}/config/config.default.js
exports.oauth2Server = {
  grants: [ 'password', 'client_credentials' ],
};
```

在这里我们开启了 `password` 模式和 `client_credentials` 模式，OAuth 2.0 协议一共定义了4种模式，这里我们只介绍这两种授权模式的实现。

## password 模式实现
password 模式需要客户端在获取 token 时在 POST 请求的 body 里提供用户名和密码，服务端进行传统的账号正确性验证。

在 `{app_root}/app/extend/` 目录下创建 `oauth.js` 文件，实现如下5个 API 就完成了一个完整的 password 模式的 OAuth 2.0 服务，就能够在 route.js 里使用 `app.oauth.grant()` 和 `app.oauth.authorise()` 。


``` javascript
// {app_root}/app/extend/oauth.js
'use strict';

module.exports = () => {
  const model = {};
  model.getClient = (clientId, clientSecret, callback) => {};
  model.grantTypeAllowed = (clientId, grantType, callback) => {};
  model.getUser = (username, password, callback) => {};
  model.saveAccessToken = (accessToken, clientId, expires, user, callback) => {};
  model.getAccessToken = (bearerToken, callback) => {};
  return model;
};
```

其中获取 token 的 API `app.oauth.grant()` 依次调用上述 API 的顺序是：
`getClient` --> `grantTypeAllowed` --> `getUser` --> `saveAccessToken`

验证 token 正确性的中间件 `app.oauth.authorise()` 只调用 `getAccessToken` 。

第一步，插件库会自动解析 token 请求头里的 `clientId` 和 `clienSecret` ，进行第一步验证，只有当是可被授权的客户端并且密码正确再执行下一步。在这里，我的客户端 `my_app` 的授权密码 `my_secret` 是提前与服务端约定好，或者服务端给客户端的一个口令，服务端可以硬编码或者从数据库里进行查询。

``` javascript
model.getClient = async (clientId, clientSecret, callback) => {
  if (clientId === 'my_app' && clientSecret === 'my_secret') {
    callback(null, { clientId, clientSecret });
    return;
  }
  callback(null, null);
};
```

第二步，插件库自动解析 token 请求头里的 `grantType` ，只有当该客户端满足请求的授权模式时，才执行下一步。

``` javascript
model.grantTypeAllowed = (clientId, grantType, callback) => {
  let allowed = false;
  if (grantType === 'password' && clientId === 'my_app') {
    allowed = true;
  }
  callback(null, allowed);
};
```

第三步，password 模式独有的步骤，插件库自动解析 token 请求 body 里的 `username` 和 `password` 字段传到改 API 里，这里使用了 `egg-mongoose` 插件从数据库里查询用户名，并通过 `bcrypt` 加密库验证密码的正确性。这里会把 `user._id` 作为用户验证正确的回调参数传给下一步进行保存，可以对应每个 token 和具体用户。

``` javascript
model.getUser = async (username, password, callback) => {
  const user = await app.model.User.findOne({ $or: [
      { email: username },
      { name: username },
  ] });
  if (!user) {
    callback(null, null);
    return;
  }
  const result = await bcrypt.compare(password, user.password);
  if (!result) {
    callback(null, null);
  } else {
    callback(null, { id: user._id });
  }
};
```

第四步也就是最后一步，非常简单，将返回给用户的 token 进行保存入库，供之后服务端查询 token 的有效性和对应的用户。

``` javascript
model.saveAccessToken = async (accessToken, clientId, expires, user, callback) => {
  await app.model.OauthToken({ accessToken, expires, clientId, user }).save();
  callback(null);
};
```

以上四步任何一步的回调传 Error，客户端都会得到 400 或者 401 错误码，客户端请求 token 失败。

验证 token 正确性的中间件 `app.oauth.authorise()` 调用的唯一函数 `getAccessToken` 非常简单，直接从数据库中查询该 token 并进行回调，插件库会自动比对有效期。

``` javascript
model.getAccessToken = async (bearerToken, callback) => {
  const token = await app.model.OauthToken.findOne({ accessToken: bearerToken });
  callback(null, token);
};
```

## client_credentials 模式实现

client_credentials 模式顾名思义，就是服务端完全信任客户端，只通过 `clientId` 和 `clientSecret` 验证请求 token 的有效性，和 password 模式的 API 生命周期相比，就是使用 `getUserFromClient` 替代 `getUser` ，这个用户信息可以藏在 `clientSecret` 里或者通过其他调用得到，具体业务的实现比较灵活。

``` javascript
model.getUserFromClient = async (clientId, clientSecret, callback) => {
  callback(null, { id: clientSecret });
};
```

# 具体验证方法
现在可以使用开发好的 OAuth 2.0 授权服务了，首先请求得到 token。

``` javascript
const response = await app.httpRequest()
  .post('/oauth2/access_token')
  .set('Content-Type', 'application/x-www-form-urlencoded')
  .set('Authorization', 'Basic bXlfYXBwOm15X3NlY3JldA==')
  .send({
    grant_type: 'password',
    username: 'test',
    password: '123456',
  })
  .expect(200);
assert(response.body.access_token);
```

其中 `Authorization` 字段的内容是 `Basic` 加上 base64 编码后的 `{clientId:clientSecret}` ，例如事例代码的内容 `bXlfYXBwOm15X3NlY3JldA==` 就是 base64 编码的 `my_app:my_secret` 。

客户端需要保存好 `response.body.access_token` ，用在之后的请求头的 `Authorization` 字段中。

```javascript
await app.httpRequest()
  .get('/private/get')
  .set('Authorization', 'Bearer ' + response.body.access_token)
  .expect(200);
```

这样，使用 Egg 开发的 OAuth 2.0 服务就算开发完成了。

# 结尾
Egg.js 对 Koa 进行了人性化的封装，提供了一个规约完善的脚手架，适合团队快速上手开发。 `egg-oauth2-server` 插件对开发者屏蔽了 OAuth 2.0 协议的细枝末节，开发者只需对关键的业务进行实现，就能开发出一个完全符合协议的授权服务。