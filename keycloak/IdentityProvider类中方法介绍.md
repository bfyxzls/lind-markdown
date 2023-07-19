# 认证中要说的三的方法
* IdentityProvider/updateBrokeredUser
* AbstractOAuth2IdentityProvider/extractIdentityFromProfile
* WeixinUserAttributeMapper/updateBrokeredUser

# 共同点
* 这三个方法在用户社区登录时，都会被执行
* 都是更新用户信息的方法

# 不同点
* IdentityProvider/updateBrokeredUser和WeixinUserAttributeMapper/updateBrokeredUser是更新已经绑定用户信息的
* 上面两个方法是可以二选一的，根据自己的需求选择，后者强调职责分离
* AbstractOAuth2IdentityProvider/extractIdentityFromProfile是更新未绑定用户信息的（新用户）

相关文章：https://www.cnblogs.com/lori/p/17559814.html
