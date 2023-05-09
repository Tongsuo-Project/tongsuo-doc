<a name="kVxml"></a>
## 实战SM4加解密
```bash
echo "hello tongsuo" > msg.bin

# SM4-CBC加密
/opt/tongsuo/bin/tongsuo enc -K "3f342e9d67d6ce7be701756af7bac8f2" -e -sm4-cbc -in msg.bin -iv "1fb2d42fb36e2e88a220b04f2e49aa13" -nosalt -out cipher.bin

# SM4-CBC解密
/opt/tongsuo/bin/tongsuo enc -K "3f342e9d67d6ce7be701756af7bac8f2" -d -sm4-cbc -in cipher.bin -iv "1fb2d42fb36e2e88a220b04f2e49aa13" -nosalt -out msg2.bin

# 比较解密的明文和原来的消息是否一样
diff msg.bin msg2.bin
```
<a name="xkbsv"></a>
## 实战SM3杂凑
```bash
echo -n "hello tongsuo" | /opt/tongsuo/bin/tongsuo dgst -sm3
```
结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/21453368/1683597693036-fd7d5bc6-de08-4334-8be7-501926bf3bfc.png#averageHue=%23141414&clientId=u092179e0-ef88-4&from=paste&height=68&id=W6uV6&originHeight=136&originWidth=1484&originalType=binary&ratio=2&rotation=0&showTitle=false&size=36947&status=done&style=none&taskId=u266d0894-71d7-48f3-ba81-bd83e3a8640&title=&width=742)
<a name="rehkK"></a>
## 实战SM2签名和验签
```bash
# 生成一个随机内容文件
dd if=/dev/urandom of=msg.bin bs=1024 count=1

# SM2私钥签名，签名算法为SM2withSM3，Tongsuo/test/certs/sm2.key来自Tongsuo源代码仓库
/opt/tongsuo/bin/tongsuo dgst -sm3 -sign Tongsuo/test/certs/sm2.key -out sigfile msg.bin

# SM2公钥验签，Tongsuo/test/certs/sm2pub.key来自Tongsuo源代码仓库
/opt/tongsuo/bin/tongsuo dgst -sm3 -verify Tongsuo/test/certs/sm2pub.key -signature sigfile msg.bin

```
签名正确时，验证成功可以看到：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/21453368/1683597693031-4ac100d4-978f-479b-87c8-72be154ba563.png#averageHue=%23090909&clientId=u092179e0-ef88-4&from=paste&height=78&id=TczLA&originHeight=156&originWidth=2180&originalType=binary&ratio=2&rotation=0&showTitle=false&size=36325&status=done&style=none&taskId=u39a4fddd-70cf-4d20-9f25-5de50d189a7&title=&width=1090)
<a name="O2uX6"></a>
## 实战SM2加密和解密
```bash
echo "hello tongsuo" > msg.bin

# 使用SM2公钥加密，Tongsuo/test/certs/sm2pub.key来自Tongsuo源代码仓库
/opt/tongsuo/bin/tongsuo pkeyutl -inkey Tongsuo/test/certs/sm2pub.key -pubin -encrypt -in msg.bin -out cipher.bin

# 使用SM2私钥解密，Tongsuo/test/certs/sm2.key来自Tongsuo源代码仓库
/opt/tongsuo/bin/tongsuo pkeyutl -inkey Tongsuo/test/certs/sm2.key -decrypt -in cipher.bin -out msg2.bin

# 比较明文和解密后是否一致
diff msg.bin msg2.bin
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21453368/1683598986788-cab25820-be36-4e5e-bf20-51162ec24990.png#averageHue=%230f0f0f&clientId=ud06b6b9c-65ff-4&from=paste&height=80&id=u45c32ea4&originHeight=160&originWidth=1262&originalType=binary&ratio=2&rotation=0&showTitle=false&size=35467&status=done&style=none&taskId=ucbf2d2cb-5f6a-4d16-b592-aadd89169d8&title=&width=631)
