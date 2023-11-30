<a name="dzfFU"></a>
## 编译 NTLS 功能
NTLS 在 Tongsuo 的术语中代指符合 GM/T 0024 SSL VPN 和 TLCP 协议的安全通信协议，其特点是采用加密证书/私钥和签名证书/私钥相分离的方式。<br />在编译 Tongsuo 的时候，需要显式的指定编译参数方可开启 NTLS 的支持：
```bash
./config enable-ntls --prefix=/path/to/tongsuo
make -j
make install
```
<a name="54bbd597"></a>
## 特性使用（s_server/s_client工具验证）
测试用证书在`test/certs/sm2`目录下<br />server 端：命令行输入
```bash
openssl s_server -accept 127.0.0.1:4433 \
-enc_cert test/certs/sm2/server_enc.crt \
-enc_key test/certs/sm2/server_enc.key \
-sign_cert test/certs/sm2/server_sign.crt \
-sign_key test/certs/sm2/server_sign.key \
-enable_ntls
```
client 端(测试 ECC-SM2-WITH-SM4-SM3 套件)：命令行输入
```bash
openssl s_client -connect 127.0.0.1:4433 -cipher ECC-SM2-WITH-SM4-SM3 -enable_ntls -ntls
```
client 端(测试 ECDHE-SM2-WITH-SM4-SM3 套件)：命令行输入
```bash
openssl s_client -connect 127.0.0.1:4433 -cipher ECDHE-SM2-WITH-SM4-SM3 \
-sign_cert test/certs/sm2/client_sign.crt \
-sign_key test/certs/sm2/client_sign.key \
-enc_cert test/certs/sm2/client_enc.crt \
-enc_key test/certs/sm2/client_enc.key \
-enable_ntls -ntls
```
<a name="041f0a9c"></a>
## 客户端和服务端集成TLCP
服务端代码片段如下所示，可编译的完整服务端代码可以参考[https://www.yuque.com/tsdoc/ts/gdwdfuoih2mxxotf#L7Ntj](https://www.yuque.com/tsdoc/ts/gdwdfuoih2mxxotf#L7Ntj)
```c
int main() {
    //变量定义
    const SSL_METHOD *meth = NULL;
    SSL_CTX *ctx = NULL;
    const char *sign_key_file = "/path/to/sign_key_file";
    const char *sign_cert_file = "/path/to/sign_cert_file";
    const char *enc_key_file = "/path/to/enc_key_file";
    const char *enc_cert_file = "/path/to/enc_cert_file";

    //双证书相关server的各种定义
    meth = NTLS_server_method();
    //生成上下文
    ctx = SSL_CTX_new(meth);
    //允许使用国密双证书功能
    SSL_CTX_enable_ntls(ctx);

    //加载签名证书，加密证书
    if (sign_key_file) {
        if (!SSL_CTX_use_sign_PrivateKey_file(cctx->ctx, sign_key_file,
                                              SSL_FILETYPE_PEM))
            goto err;
    }

    if (sign_cert_file) {
        if (!SSL_CTX_use_sign_certificate_file(cctx->ctx, sign_cert_file,
                                               SSL_FILETYPE_PEM))
            goto err;
    }

    if (enc_key_file) {
        if (!SSL_CTX_use_enc_PrivateKey_file(cctx->ctx, enc_key_file,
                                             SSL_FILETYPE_PEM))
            goto err;
    }

    if (enc_cert_file) {
        if (!SSL_CTX_use_enc_certificate_file(cctx->ctx, enc_cert_file,
                                              SSL_FILETYPE_PEM))
            goto err;
    }

    //...后续同标准tls流程
    con = SSL_new(ctx);
}
```
客户端代码片段如下所示，可编译的完整客户端可以参考[https://www.yuque.com/tsdoc/ts/gdwdfuoih2mxxotf#Zhkwt](https://www.yuque.com/tsdoc/ts/gdwdfuoih2mxxotf#Zhkwt)
```c
int main() {
    //变量定义
    const SSL_METHOD *meth = NULL;
    SSL_CTX *ctx = NULL;
    const char *sign_key_file = "/path/to/sign_key_file";
    const char *sign_cert_file = "/path/to/sign_cert_file";
    const char *enc_key_file = "/path/to/enc_key_file";
    const char *enc_cert_file = "/path/to/enc_cert_file";

    //双证书相关client的各种定义
    meth = NTLS_client_method();
    //生成上下文
    ctx = SSL_CTX_new(meth);
    //允许使用国密双证书功能
    SSL_CTX_enable_ntls(ctx);

    //设置算法套件为ECC-SM2-WITH-SM4-SM3或者ECDHE-SM2-WITH-SM4-SM3
    //这一步并不强制编写，默认ECC-SM2-WITH-SM4-SM3优先
    if(SSL_CTX_set_cipher_list(ctx, "ECC-SM2-WITH-SM4-SM3") <= 0)
        goto err;

    //加载签名证书，加密证书，仅ECDHE-SM2-WITH-SM4-SM3套件需要这一步,
    //该部分流程用...begin...和...end...注明
    // ...begin...
    if (sign_key_file) {
        if (!SSL_CTX_use_sign_PrivateKey_file(cctx->ctx, sign_key_file,
                                              SSL_FILETYPE_PEM))
            goto err;
    }

    if (sign_cert_file) {
        if (!SSL_CTX_use_sign_certificate_file(cctx->ctx, sign_cert_file,
                                               SSL_FILETYPE_PEM))
            goto err;
    }

    if (enc_key_file) {
        if (!SSL_CTX_use_enc_PrivateKey_file(cctx->ctx, enc_key_file,
                                             SSL_FILETYPE_PEM))
            goto err;
    }

    if (enc_cert_file) {
        if (!SSL_CTX_use_enc_certificate_file(cctx->ctx, enc_cert_file,
                                              SSL_FILETYPE_PEM))
            goto err;
    }
    // ...end...

    //...后续同标准tls流程
    con = SSL_new(ctx);
}
```
<a name="ac8549da"></a>
## 说明
由于国密双证书的握手流程和协议版本号与标准tls流程存在一定的不同，因此我们选择将双证书的实现(代码里命名为 ntls )同现有的 tls 状态机拆分开来，然后在入口处通过对请求的版本号进行识别，然后使其进入正确的状态机。
