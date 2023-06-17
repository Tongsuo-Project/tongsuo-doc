OpenHarmony操作系统可以基于铜锁密码库构建。例如，Huks模块可以使用铜锁作为底层的Engine，提供密钥管理等能力。同理，对于其他模块，包括certificate_manager等，以及第三方模块都可以依赖铜锁提供的密码学原语和安全传输协议。<br />OpenHarmony基于铜锁构建，需要修改各个模块的构建脚本，核心修改包括：

- 使用Tongsuo头文件，//third_party/Tongsuo/include
- 依赖libcrypto动态库，//third_party/Tongsuo:tongsuo_libcrypto_shared
- 依赖libssl动态库，//third_party/Tongsuo:tongsuo_libssl_shared

详细适配步骤请参考以下说明。
<a name="PY2gk"></a>
## 适配OpenHarmony 3.2
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

cd Tongsuo

# 切换到8.3-stable分支
git checkout 8.3-stable

cd ..
# 为确保基于铜锁构建而不使用OpenSSL，可以将openssl文件夹改名字或删除
mv openssl openssl.orig
```
<a name="RdN2W"></a>
### 修改构建脚本
修改多个构建脚本，使用铜锁密码库，生成以下diff文件。<br />[tongsuo.diff](https://www.yuque.com/attachments/yuque/0/2023/diff/21453368/1686895980002-cba02dcd-5156-482a-8a36-829f8dcb1d2a.diff?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2023%2Fdiff%2F21453368%2F1686895980002-cba02dcd-5156-482a-8a36-829f8dcb1d2a.diff%22%2C%22name%22%3A%22tongsuo.diff%22%2C%22size%22%3A125631%2C%22ext%22%3A%22diff%22%2C%22source%22%3A%22%22%2C%22status%22%3A%22done%22%2C%22download%22%3Atrue%2C%22taskId%22%3A%22ufb0e1693-f75e-472d-9a40-0860b2bb019%22%2C%22taskType%22%3A%22upload%22%2C%22type%22%3A%22%22%2C%22__spacing%22%3A%22both%22%2C%22mode%22%3A%22title%22%2C%22id%22%3A%22u347d043b%22%2C%22margin%22%3A%7B%22top%22%3Atrue%2C%22bottom%22%3Atrue%7D%2C%22card%22%3A%22file%22%7D)
```bash
patch -p1 < tongsuo.diff
```
<a name="Gxif7"></a>
### 构建OpenHarmony
```bash
# 以RK3568开发板为例，构建其他产品需要使用对应的名字
./build.sh --product-name rk3568 --ccache
```
