[wg/wrk](https://github.com/wg/wrk)基于OpenSSL，用于测试HTTPS性能。<br />[铜锁wrk](https://github.com/Tongsuo-Project/wrk)基于原有的wrk项目，适配wrk + 铜锁密码库，支持国密HTTPS，即HTTP over TLCP，方便大家用于测试国密HTTPS性能。
<a name="P7dyL"></a>
### 新增命令行参数
    -n, --ntls             Use NTLS (TLCP) instead of TLS，使用国密TLCP通信<br />    -C, --cipher      <S>  Cipher list，设置密码套件，例如ECC-SM2-SM4-CBC-SM3<br />    -S, --sign_cert   <F>  Signature certificate，设置客户端的签名证书<br />    -K, --sign_key    <F>  Signature key，设置客户端的签名私钥<br />    -E, --enc_cert    <F>  Encryption certificate，设置客户端的加密证书<br />    -Y, --enc_key     <F>  Encryption key，设置客户端的加密私钥
<a name="oG2jJ"></a>
### 构建wrk
```bash
# wrk基于铜锁密码库测试国密性能，需要先构建铜锁密码库
git clone https://github.com/Tongsuo-Project/Tongsuo

cd Tongsuo
./config --prefix=/opt/tongsuo enable-ntls
make -j
make install

cd ../

git clone https://github.com/Tongsuo-Project/wrk

cd wrk

WITH_OPENSSL=/opt/tongsuo/ LDFLAGS=-Wl,-rpath=/opt/tongsuo/lib64/ make

```
<a name="EypCS"></a>
### 性能测试
```bash
# 测试ECC-SM2-SM4-CBC-SM3
./wrk -t2 -c100 -d10 -n --cipher "ECC-SM2-SM4-CBC-SM3" https://127.0.0.1


# 测试ECDHE-SM2-SM4-CBC-SM3，需要配置客户端的签名和加密的证书和私钥
./wrk -t2 -c100 -d10 -n --cipher "ECDHE-SM2-SM4-CBC-SM3" --sign_cert /path/to/sm2_sign.crt  --sign_key /path/to/sm2_sign.key --enc_cert /path/to/sm2_enc.crt  --enc_key /path/to/sm2_enc.key https://127.0.0.1 
```
