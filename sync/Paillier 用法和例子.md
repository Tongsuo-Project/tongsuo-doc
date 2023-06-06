<a name="bSTFE"></a>
## demo 程序
```c
#include <stdio.h>
#include <time.h>
#include <openssl/paillier.h>
#include <openssl/pem.h>

#define CLOCKS_PER_MSEC (CLOCKS_PER_SEC/1000)

int main(int argc, char *argv[])
{
    int ret = -1;
    int32_t r;
    clock_t begin, end;
    PAILLIER_KEY *pail_key = NULL, *pail_pub = NULL;
    PAILLIER_CTX *ctx1 = NULL, *ctx2 = NULL;
    PAILLIER_CIPHERTEXT *c1 = NULL, *c2 = NULL, *c3 = NULL;
    FILE *pk_file = fopen("pail-pub.pem", "rb");
    FILE *sk_file = fopen("pail-key.pem", "rb");

    if ((pail_pub = PEM_read_PAILLIER_PublicKey(pk_file, NULL, NULL, NULL)) == NULL)
        goto err;
    if ((pail_key = PEM_read_PAILLIER_PrivateKey(sk_file, NULL, NULL, NULL)) == NULL)
        goto err;

    if ((ctx1 = PAILLIER_CTX_new(pail_pub, PAILLIER_MAX_THRESHOLD)) == NULL)
        goto err;
    if ((ctx2 = PAILLIER_CTX_new(pail_key, PAILLIER_MAX_THRESHOLD)) == NULL)
        goto err;

    if ((c1 = PAILLIER_CIPHERTEXT_new(ctx1)) == NULL)
        goto err;
    if ((c2 = PAILLIER_CIPHERTEXT_new(ctx1)) == NULL)
        goto err;

    begin = clock();
    if (!PAILLIER_encrypt(ctx1, c1, 20000021))
        goto err;
    end = clock();
    printf("PAILLIER_encrypt(20000021) cost: %lfms\n", (double)(end - begin)/CLOCKS_PER_MSEC);

    begin = clock();
    if (!PAILLIER_encrypt(ctx1, c2, 500))
        goto err;
    end = clock();
    printf("PAILLIER_encrypt(500) cost: %lfms\n", (double)(end - begin)/CLOCKS_PER_MSEC);

    if ((c3 = PAILLIER_CIPHERTEXT_new(ctx1)) == NULL)
        goto err;

    begin = clock();
    if (!PAILLIER_add(ctx1, c3, c1, c2))
        goto err;
    end = clock();
    printf("PAILLIER_add(C2000021,C500) cost: %lfms\n", (double)(end - begin)/CLOCKS_PER_MSEC);

    begin = clock();
    if (!(PAILLIER_decrypt(ctx2, &r, c3)))
        goto err;
    end = clock();
    printf("PAILLIER_decrypt(C20000021,C500) result: %d, cost: %lfms\n", r, (double)(end - begin)/CLOCKS_PER_MSEC);

    begin = clock();
    if (!PAILLIER_mul(ctx1, c3, c2, 800))
        goto err;
    end = clock();
    printf("PAILLIER_mul(C500,800) cost: %lfms\n", (double)(end - begin)/CLOCKS_PER_MSEC);

    begin = clock();
    if (!(PAILLIER_decrypt(ctx2, &r, c3)))
        goto err;
    end = clock();
    printf("PAILLIER_decrypt(C500,800) result: %d, cost: %lfms\n", r, (double)(end - begin)/CLOCKS_PER_MSEC);


    printf("PAILLIER_CIPHERTEXT_encode size: %zu\n", PAILLIER_CIPHERTEXT_encode(ctx2, NULL, 0, c3, 1));

    ret = 0;
err:
    PAILLIER_KEY_free(pail_key);
    PAILLIER_KEY_free(pail_pub);
    PAILLIER_CIPHERTEXT_free(c1);
    PAILLIER_CIPHERTEXT_free(c2);
    PAILLIER_CIPHERTEXT_free(c3);
    PAILLIER_CTX_free(ctx1);
    PAILLIER_CTX_free(ctx2);
    fclose(sk_file);
    fclose(pk_file);
    return ret;
}
```
<a name="a2ce7aea"></a>
## 编译和运行
先确保 Tongsuo 开启 paillier，如果是手工编译 Tongsuo，可参考如下编译步骤：
```bash
# 下载代码
git clone git@github.com:Tongsuo-Project/Tongsuo.git


# 编译参数需要加上：enable-paillier
./config  --debug no-shared no-threads enable-paillier --strict-warnings -fPIC --prefix=/opt/tongsuo

# 编译
make -j

# 安装到目录 /opt/tongsuo 
sudo make install
```
<a name="9bc315ad"></a>
## 编译 demo 程序
```bash
gcc -Wall -g -o paillier_test ./paillier_test.c -I/opt/tongsuo/include -L/opt/tongsuo/lib -lssl -lcrypto
```
<a name="a99b5c19"></a>
## 生成 Paillier 公私钥
```bash
# 先生成私钥
/opt/tongsuo/bin/openssl paillier -keygen 2048 -out pail-key.pem
# 用私钥生成公钥
/opt/tongsuo/bin/openssl paillier -pubgen -key_in ./pail-key.pem -out pail-pub.pem
```
<a name="97c957fc"></a>
## 运行结果
```bash
$ ./paillier_test
PAILLIER_encrypt(20000021) cost: 3.202000ms
PAILLIER_encrypt(500) cost: 0.442000ms
PAILLIER_add(C2000021,C500) cost: 0.047000ms
PAILLIER_decrypt(C20000021,C500) result: 20000521, cost: 0.471000ms
PAILLIER_mul(C500,800) cost: 0.056000ms
PAILLIER_decrypt(C500,800) result: 400000, cost: 0.464000ms
PAILLIER_CIPHERTEXT_encode size: 256
```

