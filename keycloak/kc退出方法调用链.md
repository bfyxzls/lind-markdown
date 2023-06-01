# kc的退出
> 注意，这个方法是同步方法，如果你的回调方法延时严重，会影响KC的退出速度
 * 浏览器退出
 * 后台rest接口退出 [依赖客户端配置项:Backchannel Logout URL]
   * AuthenticationManager.backchannelLogoutClientSession()
     * protocol.backchannelLogout(userSession, clientSession)
       * OIDCLoginProtocol.backchannelLogout()
         * ResourceAdminManager(session).logoutClientSessionWithBackchannelLogoutUrl()
# 退出方法
* 基于session_state [session_id]
```
TYPE: GET
URL: /auth/realms/fabao/sms/remove-sessions/{session_state}?redirect_uri={redirectUrl}
```
* 基于refresh_token
```
type: POST
enctype: x-www-form-urlencoded
body: 
refresh_token: 刷新token
client_id: 客户端
client_secret: 客户端密钥
```
