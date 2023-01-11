在GNU wget项目的基础上，基于[Tongsuo](https://github.com/Tongsuo-Project/Tongsuo)项目支持TLCP协议，即可以使用wget从国密web服务器下载文件。
<a name="WccVJ"></a>
### 构建wget
基于源码构建wget，以Ubuntu操作系统为例：
```bash
# 基于Tongsuo 8.3，需要先构建Tongsuo
git clone https://github.com/Tongsuo-Project/Tongsuo

cd Tongsuo
git checkout 8.3-stable
./config --prefix=/opt/tongsuo enable-ntls
make -j
make install

cd ..

# 安装依赖
sudo apt-get install -y automake autoconf autoconf-archive autopoint \
            flex texinfo gperf pkg-config make libhttp-daemon-perl \
            libio-socket-ssl-perl libidn2-dev gettext texlive python3 valgrind \
            language-pack-tr language-pack-ru

git clone https://github.com/Tongsuo-Project/wget

cd wget
./bootstrap
autoreconf -fi
./configure --with-ssl=openssl --with-libssl-prefix=/opt/tongsuo --disable-ntlm
make -j
make install


```
<a name="Y2vYy"></a>
### wget下载文件
增加命令行选项：

- --secure-protocol=tlcp,	使用TLCP协议
- --sign-cert，			客户端签名证书
- --sign-key，			客户端签名私钥
- --enc-cert，			客户端加密证书
- --enc-key，			客户端加密私钥
```bash
# TLCP
wget --secure-protocol=tlcp https://127.0.0.1 --no-check-certificate

# TLCP，使用套件ECDHE-SM2-SM4-CBC-SM3，需要配置客户端双证书
wget --secure-protocol=tlcp --sign-cert=./sign.crt --sign-key=./sign.key --enc-cert=./enc.crt --enc-key=./enc.key https://127.0.0.1 --no-check-certificate --ciphers="ECDHE-SM2-SM4-CBC-SM3"
```
