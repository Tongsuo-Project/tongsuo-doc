<a name="NGaoi"></a>
## 构建
构建 Tongsuo 时需要开启证书压缩功能，例如：
```bash
./Configure enable-cert-compression
```
<a name="3e49d1b2"></a>
## 示例程序
可以在设置 SSL_CTX 时，添加证书压缩算法，代码示例如下：
```c
#include <openssl/ssl.h>
#include <zlib.h>

static int zlib_compress(SSL *s,
                         const unsigned char *in, size_t inlen,
                         unsigned char *out, size_t *outlen)
{

    if (out == NULL) {
        *outlen = compressBound(inlen);
        return 1;
    }

    if (compress2(out, outlen, in, inlen, Z_DEFAULT_COMPRESSION) != Z_OK)
        return 0;

    return 1;
}

static int zlib_decompress(SSL *s,
                           const unsigned char *in, size_t inlen,
                           unsigned char *out, size_t outlen)
{
    size_t len = outlen;

    if (uncompress(out, &len, in, inlen) != Z_OK)
        return 0;

    if (len != outlen)
        return 0;

    return 1;
}

int main() {
    const SSL_METHOD *meth = TLS_client_method();
    SSL_CTX *ctx = SSL_CTX_new(meth);

    /* 配置证书、私钥... */

    /* 例如：设置压缩算法为zlib */
    SSL_CTX_add_cert_compression_alg(ctx, TLSEXT_cert_compression_zlib,
                                     zlib_compress, zlib_decompress);

    SSL *con = SSL_new(ctx);

    /* 握手... */

    return 0;
}
```
<a name="NSkBB"></a>
## 命令行
也可以使用 Tongsuo 提供的 s_client 和 s_server 来使用 TLS 证书压缩功能：
```bash
# 服务端
/opt/babassl/bin/openssl s_server -accept 127.0.0.1:34567 -cert server.crt -key server.key -tls1_3 -cert_comp zlib -www -quiet

# 客户端
/opt/babassl/bin/openssl s_client -connect 127.0.0.1:34567 -tls1_3 -cert_comp zlib -ign_eof -trace
```
