<a name="T6jez"></a>
## 背景
在许多应用场景中，需要证明一个值满足特定的范围条件，但又不希望透露该值的具体大小，传统的证明方法可能需要暴露更多的信息，而范围证明可以提供一种更高效、更隐私保护的方式来实现这一目标。Bulletproofs Range Proof（范围证明）就是这样一种高效的范围证明技术，Range Proof 也是 Bulletproofs 算法的一个重要应用，用于证明一个值位于指定的范围内：![](https://cdn.nlark.com/yuque/__latex/fc6d869621c39905cbb2106c5c1f57dc.svg#card=math&code=%5B0%2C%202%5En-1%5D&id=YNLx7)，（n 为指数，是 Range Proof 的重要参数）。<br />在加密货币交易中，范围证明可以用于证明交易金额在特定范围内，以确保交易的合法性和保护用户隐私，这可以防止恶意用户创建无效的交易或泄露敏感的交易金额信息。而 Bulletproofs 具有 proof 较小的特点，可以大大减小交易成本。以下是一些使用 Bulletproofs 的区块链项目：

- Monero（门罗币）：Monero 是一种基于隐私的加密货币，它使用了 Bulletproofs 技术来实现交易的隐私和匿名性。Bulletproofs 可以帮助减小 Monero 交易的大小，提高交易的效率，并保护用户的隐私。
- Grin：Grin 是一种基于 MimbleWimble 协议的加密货币，它使用了 Bulletproofs 技术来实现交易的隐私和可扩展性。Bulletproofs 在 Grin 中被用于构建零知识证明，以验证交易的有效性和保护用户的隐私。
- Zcoin：Zcoin 是一种基于零知识证明的隐私加密货币，它使用了 Bulletproofs 技术来改进其零知识证明方案。Bulletproofs 可以帮助 Zcoin 实现更高效和更紧凑的证明，提高隐私保护的性能。

Bulletproofs 的零知识证明和范围证明特性使其成为提高区块链隐私和性能的有力工具，因此在隐私保护和可扩展性方面具有广泛的应用潜力。
<a name="dgHOb"></a>
## 命令行
<a name="RoXBu"></a>
### 公共参数（-ppgen/-pp）

- 生成公共参数
```bash
$ tongsuo bulletproofs -ppgen -out ./pp.pem -curve_name sm2 -gens_capacity 16 -party_capacity 4
```
参数说明，其中 bulletproofs 是 tongsuo 的子命令：bulletproofs 功能。不再赘述。

   - -ppgen：是 bulletproofs 的子命令，指公共参数的生成，pp 是 pub_param 的缩写，gen 是 generate 的缩写；
   - -out：输出文件路径；
   - -curve_name：椭圆曲线名称；
   - -gens_capacity：椭圆曲线点生成器的容量，对于 range proof 来说，该数是 range 的比特位数。这里是 16 位，也就是证明的范围为：![](https://cdn.nlark.com/yuque/__latex/5b98de5d8e216826c50030217f3e648f.svg#card=math&code=%5B0%2C%202%5E%7B16%7D-1%5D&id=hfLBh)，即：![](https://cdn.nlark.com/yuque/__latex/42e4512b639923eb45d16b850abd953d.svg#card=math&code=%5B0%2C%2065535%5D&id=Id5IO)
   - -party_capacity：可以生成聚合证明的最大参与方数量，仅对 range proof 有效，也就是批量范围证明的最大个数。这里是 4，也就是最多只能批量证明/验证4个数。
- 文本显示公共参数

-pp 是用来显示公共参数的子命令。
```bash
$ tongsuo bulletproofs -pp -in ./pp.pem -text -noout
Bulletproofs Public Parameter:
curve: SM2 (1172)
gens_capacity: 16
party_capacity: 4
G[n]:
    [0]: 02:b2:83:7e:13:99:e9:d7:93:57:8d:11:4a:e7:f1:01:66:0b:b2:7b:d9:2e:a4:c7:8a:bf:a6:10:89:0c:c6:93:e9
    [1]: 02:7e:44:85:96:15:53:db:f2:72:66:c5:af:b6:4d:d2:1b:f0:b7:aa:3a:f2:3a:14:fd:99:2c:e0:5d:56:26:2e:fb
    [2]: 02:81:57:f9:d5:38:f2:11:16:f4:a1:45:60:8e:50:da:31:2d:6a:c8:fa:19:d3:36:90:66:01:14:2c:14:53:0b:f0
    [3]: 03:20:46:61:ea:ff:68:49:a3:e3:d9:d5:3b:6a:a0:22:c4:78:47:c1:d0:1b:08:d7:42:8b:c9:19:d1:a8:f5:e5:18
    # 太长了，中间省略了 #
    [60]: 03:a7:b9:80:6f:ff:7d:30:6e:86:9d:d4:e0:cd:52:ee:cc:10:7e:0d:0e:6e:26:25:ae:bb:5d:48:b3:43:c0:4a:a0
    [61]: 03:4a:b8:07:7f:44:49:b2:0e:b9:26:4e:56:97:76:89:57:22:e7:34:3d:c2:e1:d9:57:a3:73:0d:dd:49:79:19:f0
    [62]: 03:9f:24:5c:30:a8:6d:08:75:8a:4c:79:04:f0:e5:41:b7:3d:41:84:71:51:77:8e:41:36:4a:9b:b6:24:bb:26:73
    [63]: 03:97:c7:a6:45:70:19:a2:ae:ff:52:fa:1c:c7:f2:e6:a0:dd:91:fd:e8:97:d8:c2:9b:4e:41:a4:9c:17:73:63:8b
H[n]:
    [0]: 03:0a:b0:b0:4c:09:7f:ec:e2:a8:41:c0:fc:75:ad:f3:02:cf:af:6a:77:a7:96:35:d8:ce:8b:6a:4b:02:a3:8c:49
    [1]: 02:5e:1d:90:be:23:90:a9:a6:02:b2:9f:b2:07:70:eb:f5:d3:0d:93:65:bd:ac:60:21:50:cc:d2:05:29:29:9d:5d
    [2]: 03:3c:d3:ef:45:77:e1:0d:e4:51:6e:73:ae:61:04:fc:6a:8d:f1:69:a6:9e:ca:d8:c4:bb:63:4c:37:c6:30:c9:05
    [3]: 03:b1:1d:fb:e0:f3:1f:33:a1:12:cf:58:0b:2c:a4:13:68:7f:42:57:98:f4:3d:0f:0c:88:98:f7:31:0e:e7:35:27
    # 太长了，中间省略了 #
    [60]: 02:bc:b6:19:81:7e:53:21:f0:b9:00:fe:b2:77:47:4c:e5:d8:f4:57:43:d0:5a:52:ba:78:59:6c:e6:00:78:23:04
    [61]: 03:cd:c2:e8:fc:27:0b:2c:af:6b:39:d7:c0:7b:2a:58:61:dd:36:8e:72:3d:6c:c1:1f:b0:12:3e:9d:78:c3:7b:cd
    [62]: 03:27:53:ec:07:b8:05:fe:24:5f:77:1d:62:c1:32:8f:75:8b:1e:bc:4f:92:ca:02:c1:ef:26:73:b3:c7:41:83:32
    [63]: 02:f0:7c:60:03:68:41:08:27:6b:d8:0b:f1:37:ba:0f:68:ac:33:f9:2b:4d:b6:74:d7:21:9f:7b:73:4c:8c:2e:90
U: 03:34:aa:13:0f:76:92:8d:33:1f:62:00:70:ff:04:86:08:9c:c7:3b:77:66:af:77:59:ce:d9:69:66:86:60:db:24
H: 02:e5:21:75:20:47:fb:c2:76:c2:86:3c:10:e0:4c:1e:72:66:26:7b:3d:5d:ac:5b:a0:de:ab:11:d3:56:bd:fc:f1
```
如上结果所示，G[n] 和 H[n] 就是随机生成椭圆曲线 SM2 上的点，n = gens_capacity * party_capacity，上图是 64，中间一些随机点由于太长太点篇幅省略掉了。
<a name="rkBc9"></a>
### 证据（-witness）

- 生成证据
```bash
$ tongsuo bulletproofs -witness -pp_in ./pp.pem -out witness_range.pem 11 22 10000

$ tongsuo bulletproofs -witness -pp_in ./pp.pem -out witness_range_capacity_exceeded.pem 11 22 33 44 55

$ tongsuo bulletproofs -witness -pp_in ./pp.pem -out witness_range_exceeded.pem 65536

$ tongsuo bulletproofs -witness -pp_in ./pp.pem -out witness_range_exceeded_one.pem 11 22 65536
```
参数说明：

   - -witness 是用来生成/显示证据的子命令
   - -pp_in：指定公共参数文件所在路径
   - -out：生成的证据文件路径，上述四条命令生成的四个证据文件分别为：
      - witness_range.pem：正常的 range 证据文件
      - witness_range_capacity_exceeded.pem：容量超出的证据文件，因为生成的公共参数支持的批量证明和验证个数为最多4个，这里指定了5个
      - witness_range_exceeded.pem：range 范围超出的证据文件，因为生成的公共参数可证明和验证的二进制比特位为16，验证范围是 ![](https://cdn.nlark.com/yuque/__latex/0e195e326df3cd1a6efeebe87a9eb02e.svg#card=math&code=%5B0%2C%202%5E%7B16%7D%29&id=UGpkj)，也就是 0-65535，提交的 65535 在验证时会失败。
      - witness_range_exceeded_one.pem：指定了3个数：11、22 和 65536，其中 11 和 22 正常，但 65536 超出范围了，这个 witness 文件生成的 proof 在验证时也会失败。
- 显示证据
```bash
$ tongsuo bulletproofs -witness -in witness_range.pem -text
Witness:
curve: SM2 (1172)
H: 02:e5:21:75:20:47:fb:c2:76:c2:86:3c:10:e0:4c:1e:72:66:26:7b:3d:5d:ac:5b:a0:de:ab:11:d3:56:bd:fc:f1
V[n]:
    [0]: 03:a0:80:74:69:3a:9e:ad:d3:c2:f2:8f:90:db:34:11:89:b5:ab:2a:da:e5:e5:45:1c:0a:e3:63:86:f2:0e:92:13
    [1]: 02:1b:1d:6a:c4:e5:ae:80:19:0c:0d:ce:48:bf:7e:19:c1:b3:a2:9a:ce:da:d1:40:dc:bd:de:43:b4:e3:c8:6a:14
    [2]: 02:b3:76:ca:f5:0a:fc:37:9d:dd:55:52:37:f7:2c:4e:a4:9c:91:41:f4:d0:ea:2e:64:05:2e:dd:f7:fb:0a:1b:1d
v[n]:
    [0]: 11 (0xb)
    [1]: 22 (0x16)
    [2]: 10000 (0x2710)
r[n]:
    [0]: 4d:df:5b:b7:fb:ed:73:1a:cc:40:53:19:4f:35:e8:f5:bf:f8:08:30:cc:6d:f7:25:77:87:f1:a4:87:1e:d8:ea
    [1]: 57:cb:b6:2a:db:b5:5c:e9:33:27:73:88:72:b4:18:18:1e:da:41:b1:71:32:8f:29:f4:e7:cc:c7:a4:f3:3b:31
    [2]: 00:88:b0:2c:3f:a0:4d:8a:91:1e:fa:3c:9b:0a:c2:e0:b1:89:7a:b4:b2:2c:e6:50:75:f7:61:60:d3:2c:b9:33:94
-----BEGIN BULLETPROOFS WITNESS-----
AAAElALlIXUgR/vCdsKGPBDgTB5yZiZ7PV2sW6DeqxHTVr388QAAAAMDoIB0aTqe
rdPC8o+Q2zQRibWrKtrl5UUcCuNjhvIOkhMAAhsdasTlroAZDA3OSL9+GcGzoprO
2tFA3L3eQ7TjyGoUAAKzdsr1Cvw3nd1VUjf3LE6knJFB9NDqLmQFLt33+wobHQAA
AAADK03fW7f77XMazEBTGU816PW/+AgwzG33JXeH8aSHHtjqK1fLtirbtVzpMydz
iHK0GBge2kGxcTKPKfTnzMek8zsxK4iwLD+gTYqRHvo8mwrC4LGJerSyLOZQdfdh
YNMsuTOUAAAAAysAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACysAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFisAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAnEA==
-----END BULLETPROOFS WITNESS-----
```
如上结果所示，witness 中包含了椭圆曲线名称，H 点（与公共参数一致），密文证据 V，明文证据 v 和随机数 r。
<a name="gDpy7"></a>
### 证明（-prove/-proof）

- 生成 range 证明（-prove）
```bash
$ tongsuo ./apps/openssl bulletproofs -prove -pp_in ./pp.pem -witness_in witness_range.pem -out proof-range.pem

$ tongsuo bulletproofs -prove -pp_in ./pp.pem -witness_in witness_range_exceeded.pem -out proof-range-exceeded.pem

$ tongsuo bulletproofs -prove -pp_in ./pp.pem -witness_in witness_range_exceeded_one.pem -out proof-range-exceeded-one.pem

$ tongsuo ./apps/openssl bulletproofs -prove -pp_in ./pp.pem -witness_in witness_range_capacity_exceeded.pem -out proof-range-capacity-exceeded.pem
May be extra arguments error, please use -help for usage summary.
80C54E0201000000:error:1F800068:lib(63):BP_RANGE_PROOF_prove:reason(104):crypto/zkp/bulletproofs/range_proof.c:276:

```
参数说明：

   - -pp_in：指定公共参数文件所在路径
   - -witness_in：指定证据文件所在路径
   - -out：指定证明的输出路径，输出的文件包含两段：R1CS PROOF 和 WITNESS，分别为 proof 和 witness

由证据文件 witness_range.pem、proof-range-exceeded.pem、proof-range-exceeded_one.pem 和 proof-range-capacity-exceeded.pem 分别生成相应的证明（proof），可以看出，超出批量容量个数的证据生成证明时报错了。

- 显示证明（-proof）
```bash
$ tongsuo bulletproofs -proof -in proof-range.pem -text
Range Proof:
A: 03:03:29:b3:64:40:02:99:1a:4d:22:4d:af:4e:6c:8d:14:84:e7:6a:94:05:bd:91:57:d0:40:7a:c1:fe:e5:96:94
S: 03:12:46:e0:97:01:a2:d6:bf:ea:90:5c:01:2c:0c:fe:ae:5f:8a:da:38:ce:0e:6c:29:a4:08:46:fb:34:ed:5b:ed
T1: 02:c3:0b:2d:49:8f:b1:e5:bb:2a:d6:40:51:8a:2c:6a:58:d8:59:0c:87:f1:e4:b4:81:f5:b1:a3:2d:d6:90:7e:60
T2: 02:eb:34:90:10:ed:b1:32:99:9f:1f:ff:09:7a:d3:15:4c:be:34:69:59:48:a4:fd:4b:e5:70:2a:d1:7a:30:9b:82
taux: 38:e7:c2:84:3c:16:c3:8e:7d:5e:8d:38:d5:8b:8e:4a:8e:45:95:db:e3:49:87:7f:36:63:b8:fb:6a:e8:0b:c4
mu: 00:b8:29:e5:f8:75:a3:32:f6:38:23:40:2e:90:ec:35:b0:84:2a:00:1d:04:6b:2e:67:53:50:ec:c0:98:ff:79:6b
tx: 00:8f:46:12:73:9d:e6:cb:aa:3e:21:4b:4d:f8:b4:66:91:40:fd:7e:bb:c0:b5:f7:07:ce:a5:f2:ff:98:86:62:40
inner proof:
    n: 6
    L[n]:
        [0]: 02:e7:34:ca:e7:98:a6:52:a6:83:ce:62:a6:c0:50:ec:86:4b:98:e9:19:c7:54:07:3d:d5:e3:f5:8d:fb:50:e0:c5
        [1]: 03:53:38:d4:99:6d:7d:cd:88:88:e5:92:b2:ce:13:f4:ce:3b:fe:e2:5f:12:c5:4b:d0:93:83:b6:35:fd:0d:03:24
        [2]: 03:a6:f9:93:14:b2:46:02:34:96:08:39:d2:28:65:83:1e:27:20:96:b1:e5:e7:f8:1f:a6:7a:9e:e7:df:60:e2:79
        [3]: 02:f4:b5:c4:76:a5:5e:c0:69:c5:a2:4d:24:52:9a:90:87:e3:cc:2a:3d:52:0b:3e:58:24:2e:c7:66:4e:65:b3:cf
        [4]: 03:44:5b:eb:8f:1c:91:a1:7f:9e:a7:9b:ed:fb:52:81:90:3a:4a:dc:1d:ee:dd:6b:66:71:c2:14:3c:7a:d4:c3:03
        [5]: 03:b7:53:59:4d:69:cf:12:85:41:52:60:f9:d2:c2:a1:82:88:6e:af:f3:33:f6:36:55:eb:99:61:ac:37:7d:c4:74
    R[n]:
        [0]: 03:08:3d:7c:3c:21:7f:7f:79:bb:43:05:ec:0b:76:41:9f:54:13:d0:c9:2d:7a:23:67:91:92:7f:74:ce:3e:ef:04
        [1]: 02:a7:f1:f2:de:ad:89:7c:1a:17:06:df:59:81:9f:9e:93:51:22:19:a7:74:9a:30:6c:ca:e8:6b:c5:8b:c2:6f:85
        [2]: 02:fc:0a:45:3b:86:37:a0:fd:6a:8f:92:0a:30:b9:2d:99:c5:2c:89:20:40:23:39:58:b0:da:f4:de:0c:5c:2a:ad
        [3]: 02:da:69:0e:87:f5:57:8d:a1:44:c7:9a:40:d2:d3:f5:aa:c1:e1:08:5c:fd:d4:37:9a:75:8a:9b:4e:2f:fe:56:79
        [4]: 02:35:7c:29:a0:47:17:84:71:ed:9c:bd:ed:9e:fc:9f:83:81:33:9d:c0:30:15:54:6f:53:f0:49:79:6d:a4:60:51
        [5]: 03:a0:88:ec:8a:de:4a:59:78:a7:f4:c9:5d:9a:a0:b9:ab:58:93:61:22:ef:ec:66:5c:1e:53:f1:a3:d6:bb:eb:b3
    a: 00:bb:f6:3c:a4:5c:d8:8e:7c:a6:fc:e1:c6:68:83:b3:67:49:b0:1e:a4:0d:1c:43:8d:a9:21:49:f2:cc:97:70:6e
    b: 00:98:f8:f1:9e:fd:5c:3d:39:2f:cf:4b:f4:78:2d:3d:26:1b:ca:cc:35:25:17:d6:ca:78:65:34:f4:65:81:fa:5b
-----BEGIN BULLETPROOFS RANGE PROOF-----
AAAElAAAAAQDAymzZEACmRpNIk2vTmyNFITnapQFvZFX0EB6wf7llpQDEkbglwGi
1r/qkFwBLAz+rl+K2jjODmwppAhG+zTtW+0CwwstSY+x5bsq1kBRiixqWNhZDIfx
5LSB9bGjLdaQfmAC6zSQEO2xMpmfH/8JetMVTL40aVlIpP1L5XAq0Xowm4IAAAAD
KzjnwoQ8FsOOfV6NONWLjkqORZXb40mHfzZjuPtq6AvEK7gp5fh1ozL2OCNALpDs
NbCEKgAdBGsuZ1NQ7MCY/3lrK49GEnOd5suqPiFLTfi0ZpFA/X67wLX3B86l8v+Y
hmJAAAAAAiu79jykXNiOfKb84cZog7NnSbAepA0cQ42pIUnyzJdwbiuY+PGe/Vw9
OS/PS/R4LT0mG8rMNSUX1sp4ZTT0ZYH6WwAAAAYC5zTK55imUqaDzmKmwFDshkuY
6RnHVAc91eP1jftQ4MUDUzjUmW19zYiI5ZKyzhP0zjv+4l8SxUvQk4O2Nf0NAyQD
pvmTFLJGAjSWCDnSKGWDHicglrHl5/gfpnqe599g4nkC9LXEdqVewGnFok0kUpqQ
h+PMKj1SCz5YJC7HZk5ls88DRFvrjxyRoX+ep5vt+1KBkDpK3B3u3WtmccIUPHrU
wwMDt1NZTWnPEoVBUmD50sKhgohur/Mz9jZV65lhrDd9xHQAAAAGAwg9fDwhf395
u0MF7At2QZ9UE9DJLXojZ5GSf3TOPu8EAqfx8t6tiXwaFwbfWYGfnpNRIhmndJow
bMroa8WLwm+FAvwKRTuGN6D9ao+SCjC5LZnFLIkgQCM5WLDa9N4MXCqtAtppDof1
V42hRMeaQNLT9arB4Qhc/dQ3mnWKm04v/lZ5AjV8KaBHF4Rx7Zy97Z78n4OBM53A
MBVUb1PwSXltpGBRA6CI7IreSll4p/TJXZqguatYk2Ei7+xmXB5T8aPWu+uz
-----END BULLETPROOFS RANGE PROOF-----
Witness:
curve: SM2 (1172)
H: 02:e5:21:75:20:47:fb:c2:76:c2:86:3c:10:e0:4c:1e:72:66:26:7b:3d:5d:ac:5b:a0:de:ab:11:d3:56:bd:fc:f1
V[n]:
    [0]: 03:a0:80:74:69:3a:9e:ad:d3:c2:f2:8f:90:db:34:11:89:b5:ab:2a:da:e5:e5:45:1c:0a:e3:63:86:f2:0e:92:13
    [1]: 02:1b:1d:6a:c4:e5:ae:80:19:0c:0d:ce:48:bf:7e:19:c1:b3:a2:9a:ce:da:d1:40:dc:bd:de:43:b4:e3:c8:6a:14
    [2]: 02:b3:76:ca:f5:0a:fc:37:9d:dd:55:52:37:f7:2c:4e:a4:9c:91:41:f4:d0:ea:2e:64:05:2e:dd:f7:fb:0a:1b:1d
    [3]: 02:0d:30:77:fb:4f:a5:f6:7d:e4:78:f3:84:ca:96:28:c2:46:92:df:2b:dc:57:f7:f8:8b:8a:33:63:c9:6d:91:7b
-----BEGIN BULLETPROOFS WITNESS-----
AAAElALlIXUgR/vCdsKGPBDgTB5yZiZ7PV2sW6DeqxHTVr388QAAAAQDoIB0aTqe
rdPC8o+Q2zQRibWrKtrl5UUcCuNjhvIOkhMAAhsdasTlroAZDA3OSL9+GcGzoprO
2tFA3L3eQ7TjyGoUAAKzdsr1Cvw3nd1VUjf3LE6knJFB9NDqLmQFLt33+wobHQAC
DTB3+0+l9n3kePOEypYowkaS3yvcV/f4i4ozY8ltkXsA
-----END BULLETPROOFS WITNESS-----

```
可以看出 proof-range.pem 中包含了两段：RANGE PROOF 和 WITNESS，这里 WITNESS 的 V 就是密文证据。
<a name="mKwbJ"></a>
### 验证（-verify）
```bash
$ tongsuo bulletproofs -verify -pp_in ./pp.pem -in proof-range.pem
The proof is valid

$ tongsuo bulletproofs -verify -pp_in ./pp.pem -in proof-range-exceeded.pem
The proof is invalid

$ tongsuo bulletproofs -verify -pp_in ./pp.pem -in proof-range-exceeded-one.pem
The proof is invalid
```
如上结果，可以看出，在范围内的 proof 验证结果有效，超出范围的验证无效。
<a name="rppgC"></a>
## Demo
<a name="wpEZB"></a>
### 代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <openssl/bulletproofs.h>

static int range_proofs_test(int bits, int64_t secrets[], int len)
{
    int ret = 0, i;
    BIGNUM *v = NULL;
    BP_TRANSCRIPT *transcript = NULL;
    BP_PUB_PARAM *pp = NULL;
    BP_WITNESS *witness = NULL;
    BP_RANGE_CTX *ctx = NULL;
    BP_RANGE_PROOF *proof = NULL;

    v = BN_new();
    
    /* 创建交互抄本对象，证明者和验证者需要使用相同的方法和标签，否则验证失败 */
    transcript = BP_TRANSCRIPT_new(BP_TRANSCRIPT_METHOD_sha256(), "test");
    
    /* 创建公共参数对象，这里最大的批量验证个数为8 */
    pp = BP_PUB_PARAM_new_by_curve_id(NID_secp256k1, bits, 8);

    if (v == NULL || transcript == NULL || pp == NULL)
        goto err;

    /* 创建该公共参数下的证据对象 */
    witness = BP_WITNESS_new(pp);
    if (witness == NULL)
        goto err;

    for (i = 0; i < len; i++) {
        if (!BN_lebin2bn((const unsigned char *)&secrets[i], sizeof(secrets[i]), v))
            goto err;

        /* 往证据对象中提交明文证据，由于是 range proof，不需要绑定名称，所以名称可以直接传 NULL */
        if (!BP_WITNESS_commit(witness, NULL, v))
            goto err;
    }

    /* 创建 range proof 上下文对象 */
    ctx = BP_RANGE_CTX_new(pp, witness, transcript);
    if (ctx == NULL)
        goto err;

    /* 创建证明对象 */
    proof = BP_RANGE_PROOF_new_prove(ctx);
    if (proof == NULL)
        goto err;

    /* 验证证明对象是否有效 */
    if (!BP_RANGE_PROOF_verify(ctx, proof))
        goto err;

    ret = 1;
err:
    BP_RANGE_PROOF_free(proof);
    BP_RANGE_CTX_free(ctx);
    BP_WITNESS_free(witness);
    BP_PUB_PARAM_free(pp);
    BP_TRANSCRIPT_free(transcript);
    BN_free(v);

    return ret;
}

int main(int argc, char *argv[])
{
    int i, ret;
    int64_t secrets[16];

    if (argc <= 1 || argc >= sizeof(secrets)/sizeof(secrets[0])) {
        printf("Invalid parameter!\n");
        return -1;
    }

    printf("number: ");
    for (i = 1; i < argc; i++) {
        secrets[i-1] = atoi(argv[i]);
        printf("%lld ", secrets[i-1]);
    }

    /* 这里指定的位数为16，即 range 范围为：[0, 65535] */
    ret = range_proofs_test(16, secrets, argc - 1);

    printf("%s range: [0, 65535]\n", ret == 1 ? "in" : "not in");

    return 0;
}
```
如上代码注释所示，一次完整的调用涉及5个数据结构，分别为：

- `BP_TRANSCRIPT`

该数据结构是交互抄本的结构，用来模拟交互式零知识方案中的交互，是 Bulletproofs 为非交互式零知识证明算法的关键，利用了 Fiat-Shamir 变换将其转变为随机预言机模型中的非交互式零知识方案。

   - 创建/释放
```c
/*
 * 创建 BP_TRANSCRIPT
 * 参数：
 *    method：目前仅实现了 sha256 的方法，通过调用 BP_TRANSCRIPT_METHOD_sha256() 获得，后续可以实现其他更高效的方法
 *    label：证明者和验证者的标签/标识
 */
BP_TRANSCRIPT *BP_TRANSCRIPT_new(const BP_TRANSCRIPT_METHOD *method,
                                 const char *label);

/*
 * 释放 BP_TRANSCRIPT
 */
void BP_TRANSCRIPT_free(BP_TRANSCRIPT *transcript);
```

- `BP_PUB_PARAM`

该数据结构用来保存 Bulletproofs 的公共参数，证明者和验证者需要在同一个公共参数下才有意义，否则证明者生成的证明验证者永远无法验证通过。

   - 创建/释放
```c
/*
 * 创建 BP_PUB_PARAM
 * 参数：
 *    gens_capacity：对于 range proof 来说就是 range 的位数
 *    party_capacity：对于 range proof 来说就是批量证明/验证的最大个数
 */
BP_PUB_PARAM *BP_PUB_PARAM_new(const EC_GROUP *group, int gens_capacity,
                               int party_capacity);
BP_PUB_PARAM *BP_PUB_PARAM_new_by_curve_name(const char *curve_name,
                                             int gens_capacity,
                                             int party_capacity);
BP_PUB_PARAM *BP_PUB_PARAM_new_by_curve_id(int curve_id,
                                           int gens_capacity,
                                           int party_capacity);

/* 
 * 释放 BP_PUB_PARAM
 */
void BP_PUB_PARAM_free(BP_PUB_PARAM *pp);
```

   - 编码/解码
```c
/*
 * 编码 BP_PUB_PARAM 成二进制格式，写入 out 指定的内存区域中
 * 参数：
 *    pp：公共参数对象
 * 	  out：要写入的内存指针，如果为 NULL 则返回需要的内存大小，用于申请所需的内存
 *    size：out 内存区域大小
 */
size_t BP_PUB_PARAM_encode(const BP_PUB_PARAM *pp, unsigned char *out, size_t size);

/*
 * 解码 BP_PUB_PARAM
 * 参数：
 *    in：已编码的证明内存指针
 *    size：in 指针内存区域大小
 */
BP_PUB_PARAM *BP_PUB_PARAM_decode(const unsigned char *in, size_t size);
```

- `BP_WITNESS`

证据数据结构，用来保存明文证据和密文证据，其中明文证据用于 prove，密文证据用于 verify。

   - 创建/释放
```c
/*
 * 创建 BP_WITNESS
 * 参数：
 *    pp：公共参数对象
 */
BP_WITNESS *BP_WITNESS_new(const BP_PUB_PARAM *pp);

/*
 * 释放 BP_WITNESS
 */
void BP_WITNESS_free(BP_WITNESS *witness);
```

   - 提交证据
```c
/*
 * 提交变量到证据中
 * 参数：
 *    witness：证据对象
 * 	  name：变量名称，range proof 不需要关注变量名称，设置为 NULL 即可
 * 	  v：变量值，也就是明文证据值
 */
int BP_WITNESS_commit(BP_WITNESS *witness, const char *name, const BIGNUM *v);
```

   - 编码/解码
```c
/*
 * 编码 BP_WITNESS 成二进制格式，写入 out 指定的内存区域中
 * 参数：
 *    witness：证据对象
 * 	  out：要写入的内存指针，如果为 NULL 则返回需要的内存大小，用于申请所需的内存
 *    size：out 内存区域大小
 *    flag：是否编码明文证据的开关，1为编码，0为不编码
 */
size_t BP_WITNESS_encode(const BP_WITNESS *witness, unsigned char *out, size_t size, int flag);

/*
 * 解码 BP_WITNESS
 * 参数：
 *    in：已编码的证据内存指针
 *    size：in 指针内存区域大小
 *    flag：是否解码明文证据的开关，1为解码，0为不解码
 */
BP_WITNESS *BP_WITNESS_decode(const unsigned char *in, size_t size, int flag);
```

- `BP_RANGE_CTX`

range proof 操作上下文数据结构。

   - 创建/释放
```c
/*
 * 创建 BP_RANGE_CTX
 * 参数：
 *    pp：公共参数对象
 *    witness：证据对象
 * 	  transcript：交互抄本对象
 */
BP_RANGE_CTX *BP_RANGE_CTX_new(BP_PUB_PARAM *pp, BP_WITNESS *witness, BP_TRANSCRIPT *transcript);

/*
 * 释放 BP_RANGE_CTX
 */
void BP_RANGE_CTX_free(BP_RANGE_CTX *ctx);
```

- `BP_RANGE_PROOF`

该数据结构用来保存 Bulletproofs 执行生成证明算法的计算结果，称为证明或者证明对象，由证明者生成，编码后发送给验证者来验证其有效性。

   - 创建/释放
```c
/*
 * 创建 BP_RANGE_PROOF
 * 参数：
 *    pp：公共参数对象
 */
BP_RANGE_PROOF *BP_RANGE_PROOF_new(const BP_PUB_PARAM *pp);

/*
 * 证明并创建 BP_RANGE_PROOF
 * 参数：
 *    ctx：上下文对象
 */
BP_RANGE_PROOF *BP_RANGE_PROOF_new_prove(BP_RANGE_CTX *ctx);

/*
 * 释放 BP_RANGE_PROOF
 */
void BP_RANGE_PROOF_free(BP_RANGE_PROOF *proof);
```

   - 编码/解码
```c
/*
 * 编码 BP_RANGE_PROOF 成二进制格式，写入 out 指定的内存区域中
 * 参数：
 *    proof：证明对象
 * 	  out：要写入的内存指针，如果为 NULL 则返回需要的内存大小，用于申请所需的内存
 *    size：out 内存区域大小
 */
size_t BP_RANGE_PROOF_encode(const BP_RANGE_PROOF *proof, unsigned char *out, size_t size);

/*
 * 解码 BP_RANGE_PROOF
 * 参数：
 *    in：已编码的证明内存指针
 *    size：in 指针内存区域大小
 */
BP_RANGE_PROOF *BP_RANGE_PROOF_decode(const unsigned char *in, size_t size);
```

   - 证明/验证
```c
/*
 * 证明，执行证明算法并把结果保存到 proof 对象中
 * 参数：
 *    ctx：上下文对象
 *    proof： 证明对象，一般由 BP_RANGE_PROOF_new 创建
 * 返回值：成功返回1，否则返回0
 */
int BP_RANGE_PROOF_prove(BP_RANGE_CTX *ctx, BP_RANGE_PROOF *proof);

/*
 * 验证
 * 参数：
 *    ctx：上下文对象
 *    proof： 证明对象，由证明者生成并发送给验证者来验证
 * 返回值：成功返回1，否则返回0
 */
int BP_RANGE_PROOF_verify(BP_RANGE_CTX *ctx, const BP_RANGE_PROOF *proof);
```
<a name="vjsQg"></a>
### 编译
```bash
$ gcc -Wall -g -o range_proof ./range_proof.c -I/opt/tongsuo-bulletproofs/include -L/opt/tongsuo-bulletproofs/lib -lcrypto
```
将 /opt/tongsuo-bulletproofs 替换成你的铜锁安装目录即可
<a name="PLlq4"></a>
### 运行
```bash
$ ./range_proof 11
number: 11 in range: [0, 65535]

$ ./range_proof 11 22
number: 11 22 in range: [0, 65535]

$ ./range_proof 11 22 65535
number: 11 22 65535 in range: [0, 65535]

$ ./range_proof 11 22 65536
number: 11 22 65536 not in range: [0, 65535]

$ ./range_proof 0
number: 0 in range: [0, 65535]

$ ./range_proof 65536
number: 65536 not in range: [0, 65535]

$ ./range_proof 11 22 33 44 55 66 77 88
number: 11 22 33 44 55 66 77 88 in range: [0, 65535]

$ ./range_proof 11 22 33 44 55 66 77 88 99
number: 11 22 33 44 55 66 77 88 99 not in range: [0, 65535]
```
如上结果所示，含有超出范围的数（65536）和超出批量验证个数的输入都验证失败了，其他输入验证成功。
