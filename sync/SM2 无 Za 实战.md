SM2 规范中定义的 Za 是一个另人讨厌的特性，把哈希计算和签名验签耦合在了一起（国际主流的算法是没有这个"优秀"特性的），这在许多场景下给程序架构带来挑战，容易形成所谓的💩山代码。无论是国密使用者，还是开发者，Za的存在都给我们带来不少的困惑。

Tongsuo在保持前向兼容的前提下，已经支持无 Za 的 SM2 签名验签功能，要禁掉Za，需要明确指定一个特殊的参数。这跟不设置可辨别用户ID是不一样的，默认情况下，不设置可辨别用户ID，仍然会以空ID来计算Za。

<a name="Z3ryZ"></a>
## 普通用户
普通使用 openssl 工具的用户，可以通过向签名和验签功能的命令指定 "sm2-za:no" 的参数来明确禁止Za，以下是几个示例：
```bash
# 生成自签名证书
openssl req -verbose -new -nodes -utf8 -days 10000 -batch -x509 \
    -sm3 -sigopt "sm2-za:no" \
    -config .genkey.conf \
    -copy_extensions copyall \
    -extensions v3_ca \
    -newkey sm2 \
    -out ca.cert \
    -keyout ca.key

# 生成证书签名请求并验证

openssl ecparam -genkey -name SM2 -text -out sm2.key

openssl req -verbose -new \
    -sm3 -sigopt "sm2-za:no" \
    -config .genkey.conf \
    -key sm2.key \
    -out sm2.csr

openssl req -verbose -verify \
    -vfyopt "sm2-za:no" \
    -config .genkey.conf \
    -noout -text \
    -in sm2.csr

# 给CSR签名，生成证书并验证

openssl x509 -req -days 10000 \
    -sm3 \
    -sigopt "sm2-za:no" \
    -vfyopt "sm2-za:no" \
    -extfile .genkey.conf -extensions v3_req \
    -CA ca.cert -CAkey ca.key -CAcreateserial \
    -in sm2.csr \
    -out sm2.cert

openssl verify -verbose \
    -vfyopt "sm2-za:no" \
    -show_chain \
    -x509_strict \
    -CAfile ca.cert \
    sm2.cert

```
其中 .genkey.conf 文件的内容如下：
```c
[ req ]
distinguished_name = req_distinguished_name$
prompt = no
string_mask = utf8only
x509_extensions = v3_ca

[ req_distinguished_name ]
O = SM2-noza-CA
CN = SM2 certificate signing key without Za
emailAddress = ca@sm2-noza

[ v3_ca ]
basicConstraints = critical, CA:TRUE
keyUsage = cRLSign, keyCertSign
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always, issuer

[ v3_req ]
basicConstraints = critical, CA:FALSE
keyUsage = digitalSignature, nonRepudiation
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always, issuer

```
<a name="KmLBb"></a>
## 编程API 
对于开发者来说，可以为 EVP_PKEY_CTX 对象调用如下 API 明确禁止Za：
```c
EVP_PKEY_CTX_ctrl_str(pctx, "sm2-za", "no");
```
