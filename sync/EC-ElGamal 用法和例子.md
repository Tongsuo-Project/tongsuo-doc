<a name="bSTFE"></a>
## demo 程序
```c
#include <stdio.h>
#include <time.h>
#include <openssl/ec.h>
#include <openssl/pem.h>

#define CLOCKS_PER_MSEC (CLOCKS_PER_SEC/1000)

int main(int argc, char *argv[])
{
    int ret = -1;
    uint32_t r;
    clock_t begin, end;
    EC_KEY *sk_eckey = NULL, *pk_eckey = NULL;
    EC_ELGAMAL_CTX *ctx1 = NULL, *ctx2 = NULL;
    EC_ELGAMAL_CIPHERTEXT *c1 = NULL, *c2 = NULL, *c3 = NULL;
    EC_ELGAMAL_DECRYPT_TABLE *table = NULL;
    FILE *pk_file = fopen("ec-pk.pem", "rb");
    FILE *sk_file = fopen("ec-sk.pem", "rb");

    if ((pk_eckey = PEM_read_EC_PUBKEY(pk_file, NULL, NULL, NULL)) == NULL)
        goto err;
    if ((sk_eckey = PEM_read_ECPrivateKey(sk_file, NULL, NULL, NULL)) == NULL)
        goto err;

    if ((ctx1 = EC_ELGAMAL_CTX_new(pk_eckey)) == NULL)
        goto err;
    if ((ctx2 = EC_ELGAMAL_CTX_new(sk_eckey)) == NULL)
        goto err;

    begin = clock();
    if ((table = EC_ELGAMAL_DECRYPT_TABLE_new(ctx2, 0)) == NULL)
        goto err;

    EC_ELGAMAL_CTX_set_decrypt_table(ctx2, table);
    end = clock();
    printf("EC_ELGAMAL_DECRYPT_TABLE_new(1) cost: %lfms\n", (double)(end - begin)/CLOCKS_PER_MSEC);

    if ((c1 = EC_ELGAMAL_CIPHERTEXT_new(ctx1)) == NULL)
        goto err;
    if ((c2 = EC_ELGAMAL_CIPHERTEXT_new(ctx1)) == NULL)
        goto err;

    begin = clock();
    if (!EC_ELGAMAL_encrypt(ctx1, c1, 20000021))
        goto err;
    end = clock();
    printf("EC_ELGAMAL_encrypt(20000021) cost: %lfms\n", (double)(end - begin)/CLOCKS_PER_MSEC);

    begin = clock();
    if (!EC_ELGAMAL_encrypt(ctx1, c2, 500))
        goto err;
    end = clock();
    printf("EC_ELGAMAL_encrypt(500) cost: %lfms\n", (double)(end - begin)/CLOCKS_PER_MSEC);

    if ((c3 = EC_ELGAMAL_CIPHERTEXT_new(ctx1)) == NULL)
        goto err;

    begin = clock();
    if (!EC_ELGAMAL_add(ctx1, c3, c1, c2))
        goto err;
    end = clock();
    printf("EC_ELGAMAL_add(C2000021,C500) cost: %lfms\n", (double)(end - begin)/CLOCKS_PER_MSEC);

    begin = clock();
    if (!(EC_ELGAMAL_decrypt(ctx2, &r, c3)))
        goto err;
    end = clock();
    printf("EC_ELGAMAL_decrypt(C20000021,C500) result: %d, cost: %lfms\n", r, (double)(end - begin)/CLOCKS_PER_MSEC);

    begin = clock();
    if (!EC_ELGAMAL_mul(ctx1, c3, c2, 800))
        goto err;
    end = clock();
    printf("EC_ELGAMAL_mul(C500,800) cost: %lfms\n", (double)(end - begin)/CLOCKS_PER_MSEC);

    begin = clock();
    if (!(EC_ELGAMAL_decrypt(ctx2, &r, c3)))
        goto err;
    end = clock();
    printf("EC_ELGAMAL_decrypt(C500,800) result: %d, cost: %lfms\n", r, (double)(end - begin)/CLOCKS_PER_MSEC);


    printf("EC_ELGAMAL_CIPHERTEXT_encode size: %zu\n", EC_ELGAMAL_CIPHERTEXT_encode(ctx2, NULL, 0, NULL, 1));

    ret = 0;
err:
    EC_KEY_free(sk_eckey);
    EC_KEY_free(pk_eckey);
    EC_ELGAMAL_DECRYPT_TABLE_free(table);
    EC_ELGAMAL_CIPHERTEXT_free(c1);
    EC_ELGAMAL_CIPHERTEXT_free(c2);
    EC_ELGAMAL_CIPHERTEXT_free(c3);
    EC_ELGAMAL_CTX_free(ctx1);
    EC_ELGAMAL_CTX_free(ctx2);
    fclose(sk_file);
    fclose(pk_file);
    return ret;
}
```
<a name="a2ce7aea"></a>
## 编译和运行
先确保 Tongsuo 开启 ec_elgamal，如果是手工编译 Tongsuo，可参考如下编译步骤：
```bash
# 下载代码
git clone git@github.com:Tongsuo-Project/Tongsuo.git


# 编译参数需要加上：enable-ec_elgamal
./config  --debug no-shared no-threads enable-ec_elgamal --strict-warnings -fPIC --prefix=/opt/tongsuo

# 编译
make -j

# 安装到目录 /opt/tongsuo 
sudo make install
```
<a name="9bc315ad"></a>
## 编译 demo 程序
```bash
gcc -Wall -g -o ec_elgamal_test ./ec_elgamal_test.c -I/opt/tongsuo/include -L/opt/tongsuo/lib -lssl -lcrypto
```
<a name="a99b5c19"></a>
## 生成 ECC 公私钥
```bash
# 先生成私钥，这里生成的是 SM2 曲线的私钥
/opt/tongsuo/bin/openssl ecparam -genkey -name SM2 -out ec-sk.pem
# 用私钥生成公钥
/opt/tongsuo/bin/openssl ec -in ./ec-sk.pem -pubout -out ec-pk.pem
```
<a name="97c957fc"></a>
## 运行结果
```bash
$ ./ec_elgamal_test
EC_ELGAMAL_DECRYPT_TABLE_new(0) cost: 2557.715000ms
EC_ELGAMAL_encrypt(20000021) cost: 1.448000ms
EC_ELGAMAL_encrypt(500) cost: 1.605000ms
EC_ELGAMAL_add(C2000021,C500) cost: 0.014000ms
EC_ELGAMAL_decrypt(C20000021,C500) result: 20000521, cost: 12.748000ms
EC_ELGAMAL_mul(C500,800) cost: 1.562000ms
EC_ELGAMAL_decrypt(C500,800) result: 400000, cost: 1.137000ms
EC_ELGAMAL_CIPHERTEXT_encode size: 66
```
<a name="1bbbb204"></a>
## 注意事项
`EC_ELGAMAL_DECRYPT_TABLE_new` 函数第二个参数指定是否支持负数解密，如果支持负数解密会影响解密性能，因为查询解密表的次数变多了，以及点的运算变多了，同时构造支持负数的解密表需要的时间也比较长，如下是支持负数解密的运行结果：
```bash
$ ./ec_elgamal_test
EC_ELGAMAL_DECRYPT_TABLE_new(1) cost: 99023.542000ms
EC_ELGAMAL_encrypt(20000021) cost: 1.451000ms
EC_ELGAMAL_encrypt(500) cost: 1.426000ms
EC_ELGAMAL_add(C2000021,C500) cost: 0.021000ms
EC_ELGAMAL_decrypt(C20000021,C500) result: 20000521, cost: 461.471000ms
EC_ELGAMAL_sub(C500,C2000021) cost: 0.025000ms
EC_ELGAMAL_decrypt(C500,C20000021) result: -19999521, cost: 830.430000ms
EC_ELGAMAL_mul(C500,800) cost: 1.568000ms
EC_ELGAMAL_decrypt(C500,800) result: 400000, cost: 261.117000ms
EC_ELGAMAL_CIPHERTEXT_encode size: 66
```
