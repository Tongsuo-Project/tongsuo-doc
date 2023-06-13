<a name="PY2gk"></a>
## 铜锁适配OpenHarmony 3.2，替换openssl
<a name="HiylI"></a>
### 下载OpenHarmony 3.2 release源码
```bash
# 需要上传ssh公钥到gitee账户
repo init -u git@gitee.com:openharmony/manifest.git -b OpenHarmony-3.2-Release --no-repo-verify
repo sync -c
repo forall -c 'git lfs pull'


# 安装编译器及二进制工具
bash build/prebuilts_download.sh
```
<a name="FGBfT"></a>
### 下载铜锁源代码
```bash
cd third_party

# 将铜锁源代码下载到third_party下面
git clone https://github.com/Tongsuo-Project/Tongsuo

# 切换到8.3-stable分支
git checkout 8.3-stable

# 为确保基于铜锁构建而不使用OpenSSL，可以将openssl文件夹改名字或删除
mv openssl openssl.orig
```
<a name="Gxif7"></a>
### 构建OpenHarmony
```bash
# 以RK3568开发板为例，构建其他产品需要使用对应的名字
./build.sh --product-name rk3568 --ccache
```
