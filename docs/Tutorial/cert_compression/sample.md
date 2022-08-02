# 应用程序使用铜锁证书压缩功能

## 构建
构建铜锁时需要开启证书压缩功能，例如：
```
./Configure enable-cert-compression
```

## 示例程序
可以在设置SSL_CTX时，添加证书压缩算法，代码示例如下：
```
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

## 命令行
也可以使用铜锁提供的s_client和s_server来使用TLS证书压缩功能：
```
# 服务端
/opt/tongsuo/bin/openssl s_server -accept 127.0.0.1:34567 -cert server.crt -key server.key -tls1_3 -cert_comp zlib -www -quiet

# 客户端
/opt/tongsuo/bin/openssl s_client -connect 127.0.0.1:34567 -tls1_3 -cert_comp zlib -ign_eof -trace
```

