<a name="kyWSa"></a>
## 背景
近半个世纪, 随着网络技术的飞速发展，计算模式逐渐由集中式迁移分布式，新型计算模式对加密方案的需求也从单一的机密性保护扩展到对隐私计算的支持，半同态加密算法 ElGamal 支持整数上的模加运算 “+” 同态, 适用于密态计算场景，在区块链和机器学习等涉及恶意敌手的计算场景中，还常需要**以隐私保护的方式证明密文加密的明文满足声称的约束关系**，例如，可以在加密状态下验证用户是否拥有足够的资金，而无需透露具体的账户余额，这就需要结合零知识区间范围证明来实现。<br />零知识密态区间范围证明可以根据证明者的角色分为两类场景:

1. 证明者为密文生成方：证明者知晓加密随机数和明文；
2. 证明者为密文接收方：证明者知晓解密私钥和明文；

我们称上面两种情形下完成密态证明的组件为 Gadget-1 和 Gadget-2。下面详细讨论 Gadget-1 的构造，Gadget-2 的设计可以通过重加密技术归结为 Gadget-2。
<a name="BmTfs"></a>
## 原理
目前比较高效的半同态加密算法和范围证明协议分别为EC Exponential ElGamal PKE(椭圆曲线群上的指数形式ElGamal公钥加密, 以下简称为ElGamal PKE) 和 Bulletproof，其中Bulletproof 接收的断言形式为Pedersen承诺。

**ElGamal PKE与Bulletproof的组合应用.** 密文的生成方如何为密文加密的明文生成范围证明呢? 注意到尽管ElGamal密文的第二项也是 Pedersen 承诺的形式, 但当证明者为密文生成方时, 其知晓承诺密钥![](https://cdn.nlark.com/yuque/__latex/25bf49115e581d9ceb645eb1809527a2.svg#card=math&code=%28pk%2C%20g%29&id=bEyV0)之间的离散对数![](https://cdn.nlark.com/yuque/__latex/c64a8ddee3348e01e58dd60cfd76abca.svg#card=math&code=sk&id=tVoxC), 因此无法调用 Bulletproof 完成证明 (合理性得不到保证)，如图1所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704380837999-ec3055f3-e83c-4287-8204-e2878b2edc77.png#averageHue=%23faf9f9&clientId=uaf827862-39dc-4&from=paste&height=145&id=u7675f9d9&originHeight=290&originWidth=1402&originalType=binary&ratio=2&rotation=0&showTitle=false&size=38078&status=done&style=none&taskId=u862f9371-99a8-47a6-a486-5953c2725c6&title=&width=701)<br />（图1：ElGamal PKE无法与 Bulletproof 直接对接）<br />解决该问题有两种技术手段：

1. 证明者首先使用 NIZKPoK 证明其知晓密文的随机数和消息, 再引入新的 Pedersen 承诺作为桥接, 并设计 Σ 协议证明新承诺的消息与明文的一致性 (注: 该 Sigma 协议可与第一步的 NIZKPoK 协议合并设计), 再调用 Bulletproof 对桥接承诺进行证明, 如图2所示. 该方法的缺点是需要引入**桥接承诺和额外的 Σ 协议**, 增大了证明和验证的开销。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704381165915-6035b4b5-5f22-4d4b-810c-214f590e2995.png#averageHue=%23f5f3f1&clientId=uaf827862-39dc-4&from=paste&height=179&id=ua334da78&originHeight=358&originWidth=1420&originalType=binary&ratio=2&rotation=0&showTitle=false&size=56532&status=done&style=none&taskId=ua83f5a24-017e-4e1c-8a03-e32990775f5&title=&width=710)<br />（图2：ElGamal PKE的密态范围证明组件Gadget-1之设计方法一）

2. 结合待证明的 ElGamal PKE 密文结构对 Bulletproof 进行重新设计, 使用量身定制的 Σ-Bulletproof 完成证明, 如图3所示. 该方法的缺点是需要对 Bulletproof 进行定制化拆解重构, 不具备模块化特性。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704381224560-8558f219-9b9b-47db-9da9-bedc1e9758b7.png#averageHue=%23f1ebf3&clientId=uaf827862-39dc-4&from=paste&height=150&id=u7c9b345a&originHeight=300&originWidth=1402&originalType=binary&ratio=2&rotation=0&showTitle=false&size=31739&status=done&style=none&taskId=u765c108b-7cee-4fd7-ac19-9dc34d748aa&title=&width=701)<br />（图3：ElGamal PKE的密态范围证明组件Gadget-1之设计方法二）<br />上述两种技术手段均存在不足. 为了解决这一问题, 山东大学网络空间安全学院陈宇教授对标准的ElGamal PKE进行变形扭转, 将封装密文与会话密钥的位置对调, 同时更改明文同构映射编码的基底, 得到 twisted ElGamal PKE，以下是陈宇教授对 Twisted ElGamal PKE 的定义：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704382121599-6d321565-4915-4cfd-81b6-569ac9fb6dd5.png#averageHue=%23e1f0e9&clientId=ud839c85a-96bd-4&from=paste&height=226&id=uf87205ac&originHeight=452&originWidth=1748&originalType=binary&ratio=2&rotation=0&showTitle=false&size=191239&status=done&style=none&taskId=u1b4c538a-67fd-40c9-abf4-e5a4c21f4cb&title=&width=874)
<a name="sxzMz"></a>
### 零知识证明友好特性
Twisted ElGamal PKE与标准ElGamal PKE 的性能和安全性相当, 同样满足整数上的模加同态. 特别的, 密文的第二部分恰好是标准的 Pedersen承诺形态，且承诺密钥陷门未知, 因此可**无缝对接 **Bulletproof 等一系列断言类型为 Pedersen承诺的区间范围证明, 无须进行桥接即可以黑盒的方式调用现有的区间范围证明协议, 如图4所示. 我们称公钥加密方案的这种性质为零知识证明友好。<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704381801792-d3575948-1684-42aa-b004-eaa352c4f0c6.png#averageHue=%23f9f9f9&clientId=ud839c85a-96bd-4&from=paste&height=220&id=XHsJD&originHeight=440&originWidth=1446&originalType=binary&ratio=2&rotation=0&showTitle=false&size=60594&status=done&style=none&taskId=u717e1632-658e-4089-83d1-f9c04d469ec&title=&width=723)<br />               （图4：Twisted ElGamal PKE的密态范围证明组件Gadget-1）

Twisted ElGamal PKE的密态证明组件Gadget-2的设计可以通过如下步骤完成: <br />1、证明者使用私钥对密文进行部分解密; <br />2、证明者选取新的随机数对部分解密结果进行重加密得到新密文; <br />3、证明者设计NIZK协议证明新旧密文的一致性, 即均是对同一个消息的加密(具体可通过证明DDH元组的Sigma协议实现); <br />4、证明者调用Gadget-1对新密文完成密态证明. 

**比较.** 相比标准的ElGamal PKE, twisted ElGamal PKE的显著优势在于**零知识证明友好**。下表对比了两者的密态证明组件的效率：<br />（表1：标准ElGamal PKE和twisted ElGamal PKE与Bulletproof 对接开销的对比）<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704383226423-b11446ae-f5a9-44c6-8b3f-21e714061f42.png#averageHue=%23eaeaea&clientId=ud839c85a-96bd-4&from=paste&height=103&id=u7b69368f&originHeight=206&originWidth=1300&originalType=binary&ratio=2&rotation=0&showTitle=false&size=42602&status=done&style=none&taskId=u1b1abe80-0d5e-4f3c-8483-fd8143e14a3&title=&width=650)<br />上表以twisted ElGamal PKE的对接开销为比较基准。在统计证明者和验证者开销时, 省去了数域上的操作, 因其代价相比椭圆曲线群上的操作相比可忽略，n是需要证明的密文数，在很多实际应用中, 单个证明者需要对多个密文通过 Gadgets-1/2 进行区间范围证明，当密文数量为百万量级时, 使用 twisted ElGamal PKE带来的性能提升是相当可观的。<br />以上内容参考了陈宇教授的《公钥加密设计方法》一书手稿。更详细的原理和C++实现请查阅陈宇教授的论文：[https://eprint.iacr.org/2019/319](https://eprint.iacr.org/2019/319)<br />铜锁开源密码学算法库对Bulletproofs和twisted ElGamal PKE进行了完善的支持，从而显著增强了铜锁库的隐私计算能力、赋能更多的隐私计算应用。铜锁公众号中的使用教程文档如下：<br />铜锁支持 Bulletproofs 算法：[https://mp.weixin.qq.com/s/eI0Dky-cG5biKjT-EelnHQ](https://mp.weixin.qq.com/s/eI0Dky-cG5biKjT-EelnHQ)<br />铜锁 Bulletproofs Range 使用教程：[https://mp.weixin.qq.com/s/oA2L5TduOti1R2WnqOwapQ](https://mp.weixin.qq.com/s/oA2L5TduOti1R2WnqOwapQ) <br />铜锁 Bulletproofs R1CS 使用教程：[https://mp.weixin.qq.com/s/vKRl4fLyEuNF78JcqYYVfw](https://mp.weixin.qq.com/s/vKRl4fLyEuNF78JcqYYVfw) <br />此外，如果对代码实现有兴趣也请直接查阅铜锁 zkp 算法的实现：[https://github.com/Tongsuo-Project/Tongsuo/tree/master/crypto/zkp](https://github.com/Tongsuo-Project/Tongsuo/tree/master/crypto/zkp)

