铜锁提供了一系列API来支持应用程序使用SM2，SM3和SM4算法。本文对这些用法进行整理，以方面用户更好的参考使用。<br />为了节省篇幅，头文件统一描述。
```c
#include <openssl/evp.h>
#include <openssl/err.h>
#include <openssl/ec.h>
```
<a name="qsPzF"></a>
# SM2
SM2算法涉及的操作有：密钥生成、加密、解密、签名、验签
<a name="xxzgZ"></a>
## SM2密钥生成
SM2的密钥是一个公私钥对，根据铜锁版本的不同，SM2密钥对的生成也存在区别。铜锁中使用EVP_PKEY数据结构来表示一对SM2密钥，该结构同时存放SM2的私钥和公钥。
<a name="OHRlJ"></a>
### 铜锁8.3.x和8.2.x系列
采用传统的OpenSSL风格的非对称密钥生成方式进行SM2的密钥生成
```c
EVP_PKEY_CTX *sm2_ctx = NULL;
EVP_PKEY *sm2_key = NULL;

sm2_ctx = EVP_PKEY_CTX_new_id(EVP_PKEY_EC, NULL);
if (sm2_ctx == NULL) {
    /* 错误处理 */
}
if (!EVP_PKEY_keygen_init(sm2_ctx)) {
    /* 错误处理 */
}
if (!EVP_PKEY_CTX_set_ec_paramgen_curve_nid(sm2_ctx, OBJ_sn2nid("SM2"))) {
    /* 错误处理 */
}
if (!EVP_PKEY_keygen(sm2_ctx, &sm2_key)) {
    /* 错误处理 */
}
if (!EVP_PKEY_set_alias_type(sm2_key, EVP_PKEY_SM2)) {
    /* 错误处理 */
}
```
<a name="gLWnS"></a>
### 铜锁8.4.x系列
铜锁8.4.0提供了简易的API来实现SM2密钥对的生成
```c
EVP_PKEY *sm2_key = NULL;

sm2_key = EVP_PKEY_Q_keygen(NULL, NULL, "SM2");
if (sm2_key == NULL) {
    /* 错误处理 */
}
```
执行成功之后，就可以获得一个可在后续使用的EVP_PKEY结构。<br />此外，8.4.x版本的铜锁，也兼容8.3.x和8.2.x版本的SM2密钥生成方式。
<a name="lrIrF"></a>
### 导出密钥
如果需要将内存中的密钥（EVP_PKEY）导出到文件，例如常用的PEM格式，则可使用：PEM_write_bio_PrivateKey函数进行导出。
<a name="xEwei"></a>
### 导入密钥
相比于直接生成新的SM2密钥对，一种更加常见的方式可能是从现有的密钥文件中读取密钥（例如公钥或者私钥）。TBD...
<a name="nfisz"></a>
## SM2加密
在拥有SM2密钥（即EVP_PKEY对象）之后，就可以使用该密钥进行一系列的SM2密码学原语操作。现在从SM2加密算法开始，介绍EVP_PKEY对象的使用方式。
```c
EVP_PKEY_CTX *pctx = NULL;
unsigned char msg[] = "Winter Is Coming";
unsigned char *out = NULL;
size_t outlen = 0, inlen = 0;

/* 假设sm2_key已经提前设置好 */

pctx = EVP_PKEY_CTX_new(sm2_key, NULL);
if (pctx == NULL) {
    /* 错误处理 */
}
if (EVP_PKEY_encrypt_init(pctx) <= 0) {
    /* 错误处理 */
}
inlen = strlen((char *)msg);
if (EVP_PKEY_encrypt(pctx, NULL, &outlen, msg, inlen) <= 0) {
    /* 错误处理 */
}
out = OPENSSL_malloc(outlen);
if (out == NULL) {
    /* 错误处理 */
}
if (EVP_PKEY_encrypt(pctx, out, &outlen, msg, inlen) <= 0) {
    /* 错误处理 */
}
EVP_PKEY_CTX_free(pctx);
EVP_PKEY_free(sm2_key);
```
其中，在铜锁中公钥密码学算法的操作和运算，是基于EVP_PKEY_CTX对象展开的。因此首先需要基于EVP_PKEY创建一个EVP_PKEY_CTX，即上例中的pctx变量，然后对pctx进行加密初始化。上例中，明文消息被存放在msg变量中，是一个字符串，当然也可以是任何其他的数据或内容。然后是一个典型的两次调用过程，即对实际的加密函数EVP_PKEY_encrypt进行两次调用，第一次调用时密文buffer为空，旨在获取加密后的密文长度，然后根据此长度进行内存分配，即上例中对out指针分配内存。第二次调用是实际执行的SM2加密操作，即对明文msg，使用sm2_key进行加密，得到密文out。
<a name="YCLL9"></a>
## SM2解密
SM2的解密操作和SM2加密操作类似，几乎就是将encrypt的相关函数替换为解密的decrypt函数。
```c
EVP_PKEY_CTX *pctx = NULL;
unsigned char *out = NULL;
size_t outlen = 0, inlen = 0;

/* 假设sm2_key, 密文cipher_text, 密文长度inlen已经提前准备好 */
pctx = EVP_PKEY_CTX_new(sm2_key, NULL);
if (pctx == NULL) {
    /* 错误处理 */
}
if (EVP_PKEY_decrypt_init(pctx) <= 0) {
    /* 错误处理 */
}
if (EVP_PKEY_decrypt(pctx, NULL, &outlen, cipher_text, inlen) <= 0) {
    /* 错误处理 */
}
out = OPENSSL_malloc(outlen);
if (out == NULL) {
    /* 错误处理 */
}
if (EVP_PKEY_decrypt(pctx, out, &outlen, cipher_text, inlen) <= 0) {
    /* 错误处理 */
}
EVP_PKEY_CTX_free(pctx);
EVP_PKEY_free(sm2_key);
```
由上例可见，SM2解密的流程和加密类似，经过两次调用EVP_PKEY_decrypt实现对密文的解密，解密后的明文存放在out指向的内存中。对于不再需要使用的PKEY和PKEY_CTX，需要显式的调用对应的_free函数进行释放。
<a name="bzXSg"></a>
## SM2签名
SM2签名操作的整体流程类似于加密操作。
```c
EVP_PKEY_CTX *pctx = NULL;
unsigned char md[EVP_MAX_MD_SIZE];
unsigned char *sig = NULL;
size_t mdlen = EVP_MAX_MD_SIZE, siglen = 0;

/* 假设sm2_key, md, mdlen已经提前准备好*/
pctx = EVP_PKEY_CTX_new(sm2_key, NULL);
if (pctx == NULL) {
    /* 错误处理 */
}
if (EVP_PKEY_sign_init(pctx) <= 0) {
    /* 错误处理 */
}
if (EVP_PKEY_sign(pctx, NULL, &siglen, md, mdlen) <= 0) {
    /* 错误处理 */
}
sig = OPENSSL_malloc(siglen);
if (sig == NULL) {
    /* 错误处理 */
}
if (EVP_PKEY_sign(pctx, sig, &siglen, md, mdlen) <= 0) {
    /* 错误处理 */
}
```
整个流程也是调用两次EVP_PKEY_sign，对md中存储的数据进行签名。这里用md，是message digest的意思，即对消息的摘要进行签名，是目前SM2签名算法的典型使用方式。得到的签名值会存储在sig指针所指向的内存中，其长度为siglen。在使用的时候一般会将消息原文和签名值共同发出，供接收方验签使用。
<a name="IzUDK"></a>
## SM2验签
SM2验签是针对SM2签名和原文进行验证的过程，因此输入的数据需要有原文和签名值，返回的结果是成功或失败，其中成功是指经过SM2验签发现原文没有被篡改，签名值OK。失败则代表验签出现了问题，原文可能被篡改、或签名值损坏等原因导致。
```c
EVP_PKEY sm2_key；
EVP_PKEY_CTX *pctx;
unsigned char *md, *sig;
size_t mdlen, siglen;

/* 假设sm2_key, md, mdlen, sig, siglen都已经准备好 */
EVP_PKEY_CTX_free(pctx);
pctx = EVP_PKEY_CTX_new(sm2_key, NULL);
if (pctx == NULL) {
    /* 错误处理 */
}
if (EVP_PKEY_verify_init(pctx) <= 0) {
    /* 错误处理 */
}
ret = EVP_PKEY_verify(pctx, sig, siglen, md, mdlen);
if (ret == 1) {
    /* 验签成功 */
} else if (ret == 0) {
    /* 验签失败 */
} else if (ret < 0) {
    /* 错误处理 */
} else {
    /* will never happen */
}
```
SM2验签的整体流程类似于签名流程，唯一值得注意的事EVP_PKEY_verify函数的返回值。不同于其他函数，EVP_PKEY_verify返回值只有在小于0的时候，才是发生了错误，需要进行错误处理。而返回0的时候，则只是说明验签失败，而非其他常规错误。
<a name="fftwB"></a>
# SM3
SM3是计算消息摘要的商用密码算法，也叫做密码杂凑算法或哈希算法。SM3算法不需要密钥，可直接针对消息进行计算从而得到其摘要值。根据铜锁版本的不同，SM3算法也有不同的使用方式。
<a name="Ll8l5"></a>
## 铜锁8.3.x和8.2.x系列
```c
EVP_MD_CTX *mdctx = NULL;
unsigned int mdlen = 0;

/* 假设msg, inlen, md都已设置好 */
mdctx = EVP_MD_CTX_new();
if (mdctx == NULL) {
    /* 错误处理 */
}
if (!EVP_DigestInit_ex(mdctx, EVP_sm3(), NULL)) {
    /* 错误处理 */
}
if (!EVP_DigestUpdate(mdctx, msg, inlen)) {
    /* 错误处理 */
}
if (!EVP_DigestFinal_ex(mdctx, md, &mdlen)) {
    /* 错误处理 */
}
EVP_MD_CTX_free(mdctd);
```
其中，EVP_DigestUpdate可以调用多次，实现对数据流进行哈希。
<a name="NutfE"></a>
## 铜锁8.4.x系列
```c
unsigned char data[] = "Winter Is Coming", *md;
size_t len = 0, mdlen = 0;

/* 假设各项数据都已经准备好 */
if (!EVP_Q_digest(NULL, "SM3", NULL, data, len, md, &mdlen)) {
    /* 错误处理 */
}
```
在8.4.x的铜锁中也可以时候用老版本的方式进行SM3计算，例如进行流式数据输入的场景下。
<a name="xZ2PF"></a>
# SM4
SM4是分组加密算法，因此存在加密和解密两种计算。但是因为分组加密涉及到加密模式的问题（mode of operation），尤其是对AEAD相关模式的使用，也存在不用的调用方式，因此需要额外注意。
<a name="lOuKr"></a>
## SM4加密
<a name="kA8vV"></a>
### 非AEAD模式
即诸如ECB，CBC等模式的使用方式。
```c
EVP_CIPHER_CTX *sm4_ctx = NULL;
unsigned char key[] = { 0x01, 0x23, 0x45, 0x67, 0x89, 0xAB, 0xCD, 0xEF,
                        0xFE, 0xDC, 0xBA, 0x98, 0x76, 0x54, 0x32, 0x10 };
unsigned char iv[] = { 0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 
                       0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F };

/* 假设msg, msglen, out, outlen均定义完毕 */
sm4_ctx = EVP_CIPHER_CTX_new();
if (sm4_ctx == NULL) {
    /* 错误处理 */
}
if (!EVP_EncryptInit_ex(sm4_ctx, EVP_sm4_cbc(), NULL, key, iv)) {
    /* 错误处理 */
}
if (!EVP_EncryptUpdate(sm4_ctx, out, (int *)&outlen, msg, msglen)) {
    /* 错误处理 */
}
if (!EVP_EncryptFinal_ex(sm4_ctx, out + outlen, (int *)&tmplen)) {
    /* 错误处理 */
}
EVP_CIPHER_CTX_free(sm4_ctx);
```
SM4算法的密钥是16个字节长，即128位。在上例中，因为使用了CBC模式，因此还需要额外准备IV值。SM4加密的整体流程也是Init-Update-Final的形式。如果需要使用其他加密模式，则需对应调整EVP_sm4_cbc()函数为其他模式，例如EVP_sm4_ecb等。
<a name="XIs8E"></a>
### AEAD模式
铜锁支持的SM4的AEAD模式有GCM和CCM。这两种模式在铜锁中的加密方式基本类似于非AEAD模式，但是由于AEAD增加了额外的认证信息，因此需要增加新的API。
```c
unsigned char key[] = { 0x01, 0x23, 0x45, 0x67, 0x89, 0xAB, 0xCD, 0xEF,
                        0xFE, 0xDC, 0xBA, 0x98, 0x76, 0x54, 0x32, 0x10 };
unsigned char iv_gcm[] = { 0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 
                           0x08, 0x09, 0x0A, 0x0B };
unsigned char aad[] = "Winter has come";
size_t aadlen = strlen((char *)aad);
unsigned char tag[16];

sm4_ctx = EVP_CIPHER_CTX_new();
if (sm4_ctx == NULL) {
    /* 错误处理 */
}
if (!EVP_EncryptInit_ex(sm4_ctx, EVP_sm4_gcm(), NULL, key, iv)) {
    /* 错误处理 */
}
if (!EVP_EncryptUpdate(sm4_ctx, NULL, (int *)&outlen, aad, aadlen)) {
    /* 错误处理 */
}
if (!EVP_EncryptUpdate(sm4_ctx, out, (int *)&outlen, msg, msglen)) {
    /* 错误处理 */
}
if (!EVP_EncryptFinal_ex(sm4_ctx, out + outlen, (int *)&tmplen)) {
    /* 错误处理 */
}
if (!EVP_CIPHER_CTX_ctrl(sm4_ctx, EVP_CTRL_GCM_GET_TAG, 16, tag)) {
    /* 错误处理 */
}
EVP_CIPHER_CTX_free(sm4_ctx);
```
以SM4 GCM为例，在上例中，主要有两处需要注意之处。第一点是在out设置为NULL的情况下调用EVP_EncryptUpdate函数设置AAD（即Additional Authenticated Data），此步骤为可选，即可以不提供AAD。第二点就是需要使用EVP_CIPHER_CTX_ctrl获取tag。这个tag将在解密的时候来判断密文是否被篡改。
<a name="kGQy1"></a>
## SM4解密
<a name="D7hpD"></a>
### 非AEAD模式
非AEAD模式下的SM4算法的解密流程和加密一致，只不过把encrypt函数替换成对应的decrypt函数即可。为了节省篇幅，这里就不赘述。
<a name="lHenz"></a>
### AEAD模式
因为涉及到tag的使用，因此AEAD模式的解密，例如GCM，需要额外对tag进行处理。
```c
unsigned char key[] = { 0x01, 0x23, 0x45, 0x67, 0x89, 0xAB, 0xCD, 0xEF,
                        0xFE, 0xDC, 0xBA, 0x98, 0x76, 0x54, 0x32, 0x10 };
unsigned char iv_gcm[] = { 0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 
                           0x08, 0x09, 0x0A, 0x0B };
unsigned char aad[] = "Winter has come";
size_t aadlen = strlen((char *)aad);

/* 假设tag已从额外渠道获取 */
sm4_ctx = EVP_CIPHER_CTX_new();
if (sm4_ctx == NULL) {
    /* 错误处理 */
}
if (!EVP_DecryptInit_ex(sm4_ctx, EVP_sm4_gcm(), NULL, key, iv)) {
    /* 错误处理 */
}
if (!EVP_DecryptUpdate(sm4_ctx, NULL, (int *)&outlen, aad, aadlen)) {
    /* 错误处理 */
}
if (!EVP_DecryptUpdate(sm4_ctx, out, (int *)&outlen, msg, msglen)) {
    /* 错误处理 */
}
if (!EVP_CIPHER_CTX_ctrl(sm4_ctx, EVP_CTRL_GCM_SET_TAG, 16, tag)) {
    /* 错误处理 */
}
if (!EVP_DecryptFinal_ex(sm4_ctx, out + outlen, (int *)&tmplen)) {
    /* 错误处理 */
}
EVP_CIPHER_CTX_free(sm4_ctx);
```
和GCM模式的加密相比，解密操作最大不同就是需要在EVP_DecryptFinal_ex之前，进行tag的设置，从而使得铜锁可以判断信息是否遭到了篡改。
