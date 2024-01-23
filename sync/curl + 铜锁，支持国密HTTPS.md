curl命令行工具可以用于发起HTTPS请求，但是不支持国密TLCP协议，[铜锁curl](https://github.com/Tongsuo-Project/curl)项目在curl的基础上，基于Tongsuo支持curl命令行国密HTTPS通信，即HTTPS over TLCP，同时支持libcurl的国密HTTPS通信。
<a name="iUFk0"></a>
## 铜锁官网下载curl
[https://www.tongsuo.net/releases/](https://www.tongsuo.net/releases/)
<a name="fxg05"></a>
## 基于源码构建铜锁curl
```bash
# 基于Tongsuo，需要先构建Tongsuo
git clone https://github.com/Tongsuo-Project/Tongsuo

cd Tongsuo
./config --prefix=/opt/tongsuo enable-ntls
make -j
make install

cd ../

git clone https://github.com/Tongsuo-Project/curl.git

cd curl

git apply tongsuo.patch

# 依赖autoconf, automake, libtool
autoreconf -fi

# 依赖pkg-config，否则可能出现configure: error: --with-openssl was given but OpenSSL could not be detected
# 如果configure失败，可能是curl依赖的库不存在，比如brotli，可以安装依赖库，或者关闭该选项，例如增加--without-brotli
LDFLAGS=-Wl,-rpath=/opt/tongsuo/lib64 ./configure --enable-warnings --enable-werror --with-openssl=/opt/tongsuo

make -j
# 默认curl命令行会安装到/usr/local/bin，libcurl会安装到/usr/local/lib
make install

```
<a name="E7zo2"></a>
## curl命令行使用国密HTTPS
新增命令行选项：

- --tlcp，使用TLCPv1.1协议
- --sign-cert，设置客户端签名证书
- --sign-key，设置客户端签名私钥
- --enc-cert，设置客户端加密证书
- --enc-key，设置客户端加密私钥

```bash
# 使用TLCP通信
curl --tlcp -kv https://127.0.0.1

# 设置密码套件
curl --tlcp -kv --ciphers "ECC-SM2-SM4-GCM-SM3:ECC-SM2-SM4-CBC-SM3" https://127.0.0.1

# 设置客户端证书
curl --tlcp -kv --sign-cert /path/to/sign_cert --sign-key /path/to/sign_key --enc-cert /path/to/enc_cert --enc-key /path/to/enc_key https://127.0.0.1

```
<a name="axHDr"></a>
## 基于libcurl国密通信示例
<a name="QGVK6"></a>
### 示例1：TLCP 1.1，套件ECC-SM2-SM4-CBC-SM3
```c
#include <stdio.h>
#include <curl/curl.h>

int main(int argc, char **argv)
{
    CURL *curl;
    CURLcode res;

    curl = curl_easy_init();
    if (curl) {
        curl_easy_setopt(curl, CURLOPT_URL, "https://127.0.0.1:443");
        curl_easy_setopt(curl, CURLOPT_SSLVERSION, CURL_SSLVERSION_NTLSv1_1);
        curl_easy_setopt(curl, CURLOPT_SSL_CIPHER_LIST, "ECC-SM2-SM4-CBC-SM3");

        /* optional */
        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 0);
        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 0);

        res = curl_easy_perform(curl);

        if(res != CURLE_OK)
            fprintf(stderr, "curl_easy_perform() failed: %s\n",
                    curl_easy_strerror(res));

        curl_easy_cleanup(curl);
    }

    return 0;
}
// gcc https-tlcp.c -o https-tlcp -I/usr/local/curl/include -lcurl -L/usr/local/curl/lib -Wl,-rpath=/usr/local/curl/lib
```
<a name="tK151"></a>
### 示例2：TLCP 1.1，套件ECDHE-SM2-SM4-CBC-SM3，设置客户端双证书
```c
#include <stdio.h>
#include <curl/curl.h>

int main(int argc, char **argv)
{
    CURL *curl;
    CURLcode res;

    curl = curl_easy_init();
    if (curl) {
        curl_easy_setopt(curl, CURLOPT_URL, "https://127.0.0.1:443");
        curl_easy_setopt(curl, CURLOPT_SSLVERSION, CURL_SSLVERSION_NTLSv1_1);
        curl_easy_setopt(curl, CURLOPT_SSL_CIPHER_LIST,
                         "ECDHE-SM2-SM4-CBC-SM3");

        curl_easy_setopt(curl, CURLOPT_SSLSIGNCERT, "sm2_sign.crt");
        curl_easy_setopt(curl, CURLOPT_SSLSIGNKEY, "sm2_sign.key");
        curl_easy_setopt(curl, CURLOPT_SSLENCCERT, "sm2_enc.crt");
        curl_easy_setopt(curl, CURLOPT_SSLENCKEY, "sm2_enc.key");

        /* optional */
        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 0);
        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 0);

        res = curl_easy_perform(curl);

        if(res != CURLE_OK)
            fprintf(stderr, "curl_easy_perform() failed: %s\n",
                    curl_easy_strerror(res));

        curl_easy_cleanup(curl);
    }

    return 0;
}
// gcc https-tlcp-doublecerts.c -o https-tlcp-doublecerts -I/usr/local/curl/include -lcurl -L/usr/local/curl/lib -Wl,-rpath=/usr/local/curl/lib
```
