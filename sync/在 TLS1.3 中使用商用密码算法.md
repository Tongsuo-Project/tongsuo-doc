<a name="lrjx4"></a>
## 背景文档
RFC8998: [https://datatracker.ietf.org/doc/html/rfc8998](https://datatracker.ietf.org/doc/html/rfc8998)
<a name="ac8549da"></a>
## 重要说明
在 tls1.3 的标准中，现有的任意算法套件均不强制使用某一个签名算法，而为了国密算法的统一性，我们在RFC8998 的定义中明确要求了 tls1.3+国密的两个套件: TLS_SM4_GCM_SM3/TLS_SM4_CCM_SM3 需要强制使用 SM2 作为签名算法(即 server 端需要配置国密证书)。 然而出于兼容性考虑，现有的大量软件和设备并没有部署国密证书，为保证大家能尽快体验到更快更便捷的国密流程，我们的实现中并不完全强制签名算法为 SM2，而是提供了一个开关来由用户选择是否要开启这种强制措施(默认关闭)，在本手册后续会介绍相关用法。
<a name="0344679d"></a>
## 特性使用（s_client/s_server 工具验证）
server 端：命令行输入
```bash
openssl s_server -cert test/certs/sm2.pem -key test/certs/sm2.key -accept 127.0.0.1:4433
```
test/certs/sm2.pem -> [https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2.pem](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2.pem)<br />test/certs/sm2.key -> [https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2.key](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2.key)<br />client 端：命令行输入
```bash
openssl s_client -connect 127.0.0.1:4433 -tls1_3 -ciphersuites TLS_SM4_GCM_SM3
```
<a name="041f0a9c"></a>
## 在你的 client/server 中使用（相关 api 使用）
server 端：和标准 TLS 的 server 一样，无需做任何修改，仅需要注意是否强制签名算法使用 SM2 (虽然我们在标准中定义了 tls1.3+国密的算法套件必须强制使用 sm2 签名算法，但由于国密证书还不够普及，所以我们在实现上暂时放宽了这个限制，而是提供了一个开关)，通过下面的代码开启/关闭：
```c
//开启
SSL_CTX_enable_sm_tls13_strict(ctx);
//关闭
SSL_CTX_disable_sm_tls13_strict(ctx);
```
client 端：client 无需做修改，只需要强制指定算法套件为 TLS_SM4_GCM_SM3/TLS_SM4_CCM_SM3 (国密套件的默认优先级低于现有的 tls1.3 算法套件)，通过这种方式设定：
```c
SSL_CTX_set_ciphersuites(ctx, "TLS_SM4_GCM_SM3");
```

设置 Key Share extension 中的曲线名称方法为（也可以不设置使用默认值即可）：
```bash
SSL_CTX_set1_curves_list(ctx, "SM2:X25519:prime256v1")
```
