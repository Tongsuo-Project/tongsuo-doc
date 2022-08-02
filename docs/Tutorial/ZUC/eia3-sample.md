# ZUC 128-EIA3 例子

## 构建
构建Tongsuo时需要开启ZUC算法，例如：
```
./Configure enable-zuc
```

## 示例程序
使用 EVP 接口的代码示例如下：
```
#include <stdio.h>
#include <string.h>
#include <openssl/crypto.h>
#include <openssl/evp.h>

int main(int argc, char *argv[])
{
    EVP_MD_CTX *mctx = NULL;
    EVP_PKEY_CTX *pctx = NULL;
    EVP_PKEY *pkey = NULL;
    unsigned char *got = NULL;
    size_t got_len;
    char *mac;
    int rv;
    const char key[] = {
        0xc9, 0xe6, 0xce, 0xc4, 0x60, 0x7c, 0x72, 0xdb,
        0x00, 0x0a, 0xef, 0xa8, 0x83, 0x85, 0xab, 0x0a
    };
    /*
     * EIA3 的 iv 只有5个字节(只用到38bit)，不是随机构造，构造方法如下：
     * |----------32bit----------|-----5bit-----|---1bit---|
     * |          count          |    bearer    | direction|
     */
    const char *iv = "a94059da54";
    const char msg[] = {
        0x98, 0x3b, 0x41, 0xd4, 0x7d, 0x78, 0x0c, 0x9e,
        0x1a, 0xd1, 0x1d, 0x7e, 0xb7, 0x03, 0x91, 0xb1,
        0xde, 0x0b, 0x35, 0xda, 0x2d, 0xc6, 0x2f, 0x83,
        0xe7, 0xb7, 0x8d, 0x63, 0x06, 0xca, 0x0e, 0xa0,
        0x7e, 0x94, 0x1b, 0x7b, 0xe9, 0x13, 0x48, 0xf9,
        0xfc, 0xb1, 0x70, 0xe2, 0x21, 0x7f, 0xec, 0xd9,
        0x7f, 0x9f, 0x68, 0xad, 0xb1, 0x6e, 0x5d, 0x7d,
        0x21, 0xe5, 0x69, 0xd2, 0x80, 0xed, 0x77, 0x5c,
        0xeb, 0xde, 0x3f, 0x40, 0x93, 0xc5, 0x38, 0x81,
        0x00
    };
    size_t msg_len = 73;

    /*
     * 将 EIA3 的 key 转换成 EVP_PKEY
     */
    pkey = EVP_PKEY_new_raw_private_key(EVP_PKEY_EIA3, NULL, key, 16);
    if (pkey == NULL)
        goto end;

    mctx = EVP_MD_CTX_new();
    if (mctx == NULL)
        goto end;

    if (!EVP_DigestSignInit(mctx, &pctx, NULL, NULL, pkey))
        goto end;

    /*
     * 设置 EIA3 的 iv，其中 iv 是十六进制的字符串形式
     * 也可以设置 hexkey 来修改 key，同样是十六进制的字符吕形式
     */
    rv = EVP_PKEY_CTX_ctrl_str(pctx, "iv", iv);
    if (rv <= 0)
        goto end;

    if (!EVP_DigestSignUpdate(mctx, msg, msg_len))
        goto end;

    if (!EVP_DigestSignFinal(mctx, NULL, &got_len))
        goto end;

    got = OPENSSL_malloc(got_len);
    if (got == NULL)
        goto end;

    if (!EVP_DigestSignFinal(mctx, got, &got_len))
        goto end;

    /*
     * EIA3 的 mac 值为4个字节，这里打印出来
     */
    mac = OPENSSL_buf2hexstr(got, got_len);
    printf("MAC=%s\n", mac);

    OPENSSL_free(mac);

end:
    EVP_MD_CTX_free(mctx);
    OPENSSL_free(got);
    EVP_PKEY_free(pkey);

    return 0;
}
```
输出结果：
```
MAC=24:A8:42:B3
```
