## 登录商户平台

[https://pay.weixin.qq.com/apply/applyment4normal/detail][1]

```
1. 一定先不要申请小程序。
2. 先申请公众号，然后300块钱认证。
3. 公众号认证后，从公众号->关联小程序，->复用公众号认证
4. 申请商户号
5. 商户号关联小程序
```

### 微信支付 部署上的细节
```
# 1. ASE加密位数限制， 可以用如下方法解决
cd /usr/java/jdk1.8.0_131/jre/lib/security
替换 
local_policy.jar  US_export_policy.jar


```
[1]: https://pay.weixin.qq.com/apply/applyment4normal/detail