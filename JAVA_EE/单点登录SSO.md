## 1 功能定位

XXL-SSO 是一个分布式单点登录框架。只需要登录一次就可以访问所有相互信任的应用系统。

借助 XXL-SSO，可以快速实现分布式系统单点登录。

## 2 核心概念

| 概念          | 说明                                       |
| ------------- | ------------------------------------------ |
| SSO Server    | 中央认证服务，支持集群                     |
| SSO Client    | 接入SSO认证中心的Client应用                |
| SSO SessionId | 登录用户会话ID，SSO 登录成功为用户自动分配 |
| SSO User      | 登录用户信息，与 SSO SessionId 相对应      |

## 3 登录流程剖析（基于cookie）

- (一个处于未登录态的)<span style="color:orange">用户</span>于 <span style="color:blue">Client 端应用</span>访问受限资源时，将会自动 redirect 到 <span style="color:green">SSO Server</span> 进入统一登录界面；
- 用户登录成功之后将会为用户分配 SSO SessionId 并 redirect 返回<span style="color:blue">来源 Client 端应用</span>，同时附带分配的 SSO SessionId

- 在 Client 端的 SSO Filter 里验证 SSO SessionId 无误，将 SSO SessionId 写入到<span style="color:orange">用户浏览器</span> Client 端域名下 cookie 中
(以下流程为，一个已登录态的用户，want to access a limit resource)
- SSO Filter 验证 SSO SessionId 通过，受限资源请求放行

## 4 注销流程剖析

- 用户与 Client 端应用请求注销 Path 时，将会 redirect 到 SSO Server 自动销毁全局 SSO SessionId，实现全局销毁
- 然后，访问接入 SSO 保护的任意 Client 端应用时，SSO Filter 均会拦截请求并 redirect 到 SSO Server 的统一登录界面

## 5 基于Token，相关概念

- 登录凭证存储：登录成功后，获取到登录凭证（xxl_sso_sessionid=xxx），需要主动存储，如存储在 localStorage、Sqlite 中
- Client 端校验登录状态：<span style="color:blue">后端应用</span>会通过校验 request Header 参数中的是否包含用户登录凭证（xxl_sso_sessionid=xxx）判断；因此，<span style="color:orange">前端</span>发送请求时需要在 Header 参数中设置登陆凭证 token
- 系统角色模型：
    - SSO Server：认证中心，提供用户登录、注销以及登录状态校验等功能；
    - Client 应用：受 SSO 保护的 Client 端 Web 应用，为用户请求提供接口服务；
    - 用户：发起请求的用户，如使用 Android、IOS、桌面客户端等请求访问；

## 6 未登录状态请求处理

基于Cookie，未登录状态请求：

- 页面请求：redirect 到SSO Server登录界面
- JSON请求：返回未登录的JSON格式响应数据
    - 数据格式：

```json
    {
	    "code": "501错误码",
	    "msg": "sso not login."
    }
```

基于Token，未登录状态请求：

- 返回未登录的 JSON 格式响应数据
    - 数据格式：

```json
    {
	    "code": "501错误码",
	    "msg": "sso not login."
    }
```

## 7 登录态自动延期

支持自定义登录态有效期窗口，默认24H，当登录态有效期窗口过半时，自动顺延一个周期。

## 8 记住我

未记住密码时，关闭浏览器则登录态失效；记住密码时，登录态自动延期(这个不叫记住密码，应该叫‘记住我’吧。就是下次直接跳进我的邮箱界面，而非登录页面并带密码黑点)，在自定义延期时间的基础上，原则上可以无限延期。

## 9 路径排除

自定义路径排除Path，允许设置多个，且支持Ant表达式。用于排除SSO客户端不需要过滤的路径。