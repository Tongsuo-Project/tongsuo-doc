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
<a name="RdN2W"></a>
### 修改构建脚本
修改多个构建脚本，使用铜锁密码库，生成以下diff文件。<br />[tongsuo.diff](https://www.yuque.com/attachments/yuque/0/2023/diff/21453368/1686887247326-7b9518d1-6784-410a-896e-fc4708f6b851.diff?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2023%2Fdiff%2F21453368%2F1686887247326-7b9518d1-6784-410a-896e-fc4708f6b851.diff%22%2C%22name%22%3A%22tongsuo.diff%22%2C%22size%22%3A111566%2C%22ext%22%3A%22diff%22%2C%22source%22%3A%22%22%2C%22status%22%3A%22done%22%2C%22download%22%3Atrue%2C%22taskId%22%3A%22ua53bd94e-726c-435d-b588-03157128722%22%2C%22taskType%22%3A%22upload%22%2C%22type%22%3A%22%22%2C%22__spacing%22%3A%22both%22%2C%22mode%22%3A%22title%22%2C%22id%22%3A%22u0d3db4a5%22%2C%22margin%22%3A%7B%22top%22%3Atrue%2C%22bottom%22%3Atrue%7D%2C%22card%22%3A%22file%22%7D)
```bash
patch -p1 < tongsuo.diff
```
<a name="Gxif7"></a>
### 构建OpenHarmony
```bash
# 以RK3568开发板为例，构建其他产品需要使用对应的名字
./build.sh --product-name rk3568 --ccache
```
