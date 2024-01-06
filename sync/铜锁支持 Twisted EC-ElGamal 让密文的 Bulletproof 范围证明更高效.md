<a name="kyWSa"></a>
## 背景
<a name="sQNJw"></a>
### 关于零知识证明
零知识证明是一种能够证明某个主张为真，却不泄露有关这个主张的详细信息的密码学技术。<br />举例来说，假设你有一袋红、蓝、绿三种颜色的球，你想证明给别人看，你能够分辨这些颜色，但又不想透露每种颜色的具体数量。零知识证明的方式是这样的：

1. 你选取其中两种颜色，比如红和绿。
2. 别人随机选一种颜色，比如红。
3. 你将两种颜色的球混在一起给对方。
4. 对方从中随机拿出一种颜色，让你猜是哪一种。
5. 如果你正确猜出，就证明了你知道颜色分布，但对方并不知道每种颜色的具体数量。

在实际应用中，零知识证明可用于隐私保护、身份验证等领域，具体的实现方式会依赖于具体的协议和工具。

零知识证明大家都知道, 就不用说了
<a name="o61gW"></a>
### 关于范围证明
范围证明是零知识证明的一种特定形式，通常用于证明某个秘密值位于某个特定范围内，而无需透露实际的值。下面是一个简化的范围证明的例子：<br />假设有一个秘密数字 x，范围在 1 到 100 之间。Alice 想向 Bob 证明她知道 x 的值，但不想透露具体的数字。

1. Alice 选择承诺阶段：
   - Alice 首先生成一个“承诺”（commitment），这是一个隐藏了 x 的哈希值的数字。Bob 只能看到这个承诺，但无法从中推断出 x 的实际值。
2. Bob 选择挑战阶段：
   - Bob 随机选择一个挑战，比如要求 Alice 证明 x 在 50 到 75 的范围内。
3. Alice 回应挑战，生成证据：
   - Alice 通过使用零知识证明的协议，证明她知道 x 的确在 50 到 75 之间，而不需要透露 x 的具体值。
   - 这可能涉及 Alice 生成一些零知识证明的证据，例如证明她知道一些在这个范围内的随机数，以此间接证明 x 的范围。
4. Bob 验证证据：
   - Bob 检查 Alice 提供的证据，确保它满足挑战的条件，但并不知道 x 的确切值。

通过这个过程，Alice 成功地证明了她知道 x 在指定的范围内，而不需要透露 x 的实际值。这是一个简单的范围证明的零知识证明例子。在实际的密码学协议中，会使用更复杂的数学和密码学技术来确保安全性和隐私。

区间范围证明也不用说了
<a name="i2xZX"></a>
### 关于半同态加密
半同态加密是一类密码学算法，它允许对加密数据进行特定类型的计算，而不需要先解密数据。在半同态加密中，虽然无法在加密状态下执行所有计算，但可以执行一些基本的数学运算，例如加法或乘法，而不破坏加密的状态。这使得在不暴露敏感信息的情况下进行一定程度的计算成为可能。<br />半同态加密可用于在区块链上实现零知识证明，同时保护用户身份和数据。例如，可以在加密状态下验证用户是否拥有足够的资金，而无需透露具体的账户余额，这就需要结合上述提到的范围证明来实现。

我觉得不必展开写半同态加密的区块链应用, 直接照我书里的内容说即可
<a name="BmTfs"></a>
## 原理
目前比较高效的半同态加密算法和范围证明算法分别为 EC-ElGamal 和 Bulletproof。<br />EC-ElGamal 的加密公式为：![](https://cdn.nlark.com/yuque/__latex/94bf3aa33dc208e1133461bc982502f9.svg#card=math&code=C%3D%28g%5Er%2C%20g%5Empk%5Er%29%3D%28C_1%2CC_2%29&id=rWQpJ)，其中 pk 为公钥，r 为随机数，m 为明文（下同），加密结果为C，![](https://cdn.nlark.com/yuque/__latex/3484a0fb89180e2a2896a7689346ed75.svg#card=math&code=C_1&id=aVyHu)和![](https://cdn.nlark.com/yuque/__latex/23c4cd950cf3916a8a78af54b2decd2d.svg#card=math&code=C_2&id=q1H7K)为椭圆曲线上的两个点（![](https://cdn.nlark.com/yuque/__latex/2f4d472220b00b514539abe2b9f4f12b.svg#card=math&code=C_1%3Dg%5Er&id=hA2Vw)，![](https://cdn.nlark.com/yuque/__latex/85bb69d4fad27198c97cbe1e9340076f.svg#card=math&code=C_2%3Dg%5Empk%5Er&id=QetoE)）。<br />Bulletproof 的 Pedersen 承诺公式为： ![](https://cdn.nlark.com/yuque/__latex/2388b3789835aedacd389ca5fa9f3567.svg#card=math&code=V%3Dg%5Erh%5Em&id=mfYCS)。![](https://cdn.nlark.com/yuque/__latex/3a8f2dd249e4477d46eb555ea390d7cf.svg#card=math&code=C_1%3Dg%5Er%2C%20C_2%3Dg%5Empk%5Er&id=rBDEk)

排版还是建议严谨一点, 很多地方该用数学模式应该用数学模式. <br />在适当的地方得提到Gadget-1/2, 不然后面的表格没法衔接


在区块链的场景中，链上的数据为半同态加密的结果 ![](https://cdn.nlark.com/yuque/__latex/3484a0fb89180e2a2896a7689346ed75.svg#card=math&code=C_1&id=gAPrF)和![](https://cdn.nlark.com/yuque/__latex/23c4cd950cf3916a8a78af54b2decd2d.svg#card=math&code=C_2&id=XbVSO)，数据的生成者为 A，数据的使用者为 B， 如下图：<br />![](https://cdn.nlark.com/yuque/0/2024/jpeg/26770235/1704379672378-59d9a5e0-d66d-47a7-853a-313e45738ab7.jpeg)<br />(图1：区块链场景示例)<br />假如链上加密数据就是 A 的账户余额，A 向 B 转账时，B 如何来验证 A 的余额是否足够呢，也就是说如何使用 Bulletproof 算法来做密文的范围证明断言。观察上述 ElGamal 公式可发现，密文 ![](https://cdn.nlark.com/yuque/__latex/85bb69d4fad27198c97cbe1e9340076f.svg#card=math&code=C_2%3Dg%5Empk%5Er&id=DudWp)也是 Pedersen 承诺的形式, 但是若证明者为密文生成方, 则其知晓承诺密钥 (pk, g) 之间的离散对数关系, 因此无法调用 Bulletproof 完成证明 (合理性得不到保证)，如图2所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704380837999-ec3055f3-e83c-4287-8204-e2878b2edc77.png#averageHue=%23faf9f9&clientId=uaf827862-39dc-4&from=paste&height=145&id=u7675f9d9&originHeight=290&originWidth=1402&originalType=binary&ratio=2&rotation=0&showTitle=false&size=38078&status=done&style=none&taskId=u862f9371-99a8-47a6-a486-5953c2725c6&title=&width=701)<br />（图2：ElGamal 无法与 Bulletproof 直接对接）<br />解决该问题有两种技术手段：

1. 证明者首先使用 NIZKPoK 证明其知晓密文的随机数和消息, 再引入新的 Pedersen 承诺作为桥接, 并设计 Σ 协议证明新承诺的消息与明文的一致性 (注: 该 Sigma 协议可与第一步的 NIZKPoK 协议合并设计), 再调用 Bulletproof 对桥接承诺进行证明, 如图3所示. 该方法的缺点是需要引入桥接承诺的额外的 Σ 协议, 增大了证明和验证的开销。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704381165915-6035b4b5-5f22-4d4b-810c-214f590e2995.png#averageHue=%23f5f3f1&clientId=uaf827862-39dc-4&from=paste&height=179&id=ua334da78&originHeight=358&originWidth=1420&originalType=binary&ratio=2&rotation=0&showTitle=false&size=56532&status=done&style=none&taskId=ua83f5a24-017e-4e1c-8a03-e32990775f5&title=&width=710)<br />（图3：ElGamal PKE 的密态区间范围证明设计方法一）

2. 结合待证明的 ElGamal PKE 密文对 Bulletproof 进行重新设计, 使用量身定制的 Σ-Bulletproof 完成证明, 如图4所示. 该方法的缺点是需要对 Bulletproof 进行定制化的改动, 不具备模块化特性。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704381224560-8558f219-9b9b-47db-9da9-bedc1e9758b7.png#averageHue=%23f1ebf3&clientId=uaf827862-39dc-4&from=paste&height=150&id=u7c9b345a&originHeight=300&originWidth=1402&originalType=binary&ratio=2&rotation=0&showTitle=false&size=31739&status=done&style=none&taskId=u765c108b-7cee-4fd7-ac19-9dc34d748aa&title=&width=701)<br />（图4：ElGamal PKE 的密态区间范围证明设计方法二）<br />上述两种技术手段均存在不足. 为了解决这一问题, 山东大学网络空间安全学院陈宇教授对 exponential ElGamal PKE 进行变形扭转, 将封装密文![](https://cdn.nlark.com/yuque/__latex/61ee3d340fef1850d69757325080074a.svg#card=math&code=g%5Er&id=XKbYB)与会话密钥![](https://cdn.nlark.com/yuque/__latex/9c3df2d0f14af08a21a6d3197c1c3c57.svg#card=math&code=pk%5Er&id=VWqXr)的位置对调, 同时更改同构映射编码的基底, 得到 twisted ElGamal PKE，如图5所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704381801792-d3575948-1684-42aa-b004-eaa352c4f0c6.png#averageHue=%23f9f9f9&clientId=ud839c85a-96bd-4&from=paste&height=220&id=ucb211152&originHeight=440&originWidth=1446&originalType=binary&ratio=2&rotation=0&showTitle=false&size=60594&status=done&style=none&taskId=u717e1632-658e-4089-83d1-f9c04d469ec&title=&width=723)<br />（图5：Twisted ElGamal PKE 的密态区间范围证明）<br />以下是陈宇教授对 Twisted ElGamal PKE 的定义：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704382121599-6d321565-4915-4cfd-81b6-569ac9fb6dd5.png#averageHue=%23e1f0e9&clientId=ud839c85a-96bd-4&from=paste&height=226&id=uf87205ac&originHeight=452&originWidth=1748&originalType=binary&ratio=2&rotation=0&showTitle=false&size=191239&status=done&style=none&taskId=u1b4c538a-67fd-40c9-abf4-e5a4c21f4cb&title=&width=874)
<a name="x0Cy4"></a>
### 正确性
以下公式说明方案具有完美正确性：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704382215422-ee2b9bf7-4631-42ff-a2ab-1adc5d6e3239.png#averageHue=%23f4f4f4&clientId=ud839c85a-96bd-4&from=paste&height=44&id=u9979832e&originHeight=88&originWidth=606&originalType=binary&ratio=2&rotation=0&showTitle=false&size=9010&status=done&style=none&taskId=ub5706386-7c81-4298-b446-fe0e32870f9&title=&width=303)<br />（证明与标准的 ElGamal PKE 证明类似）
<a name="sxzMz"></a>
### 零知识证明友好特性
新的加密方案与 exponential ElGamal PKE 的性能和安全性相当, 同样满足![](https://cdn.nlark.com/yuque/__latex/5655300ec00f47a4e526d67d797b5cdd.svg#card=math&code=%5Cmathbb%7BZ%7D_q&id=qnpNQ)上的模加同态. 特别的, 密文的第二部分恰好是标准的 Pedersen 承诺形态 (承诺密钥陷门未知), 可无缝对接 Bulletproof 等一系列断言类型为 Pedersen commitment 的区间范围证明, 如图5所示. 我们称公钥加密方案的这种性质为零知识证明友好。<br />下表对比了两者的密态证明组件的效率：<br />（表1：标准 ElGamal 和 twisted ElGamal 与 Bulletproof 对接开销的对比）<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/26770235/1704383226423-b11446ae-f5a9-44c6-8b3f-21e714061f42.png#averageHue=%23eaeaea&clientId=ud839c85a-96bd-4&from=paste&height=103&id=u7b69368f&originHeight=206&originWidth=1300&originalType=binary&ratio=2&rotation=0&showTitle=false&size=42602&status=done&style=none&taskId=u1b1abe80-0d5e-4f3c-8483-fd8143e14a3&title=&width=650)<br />在统计证明者和验证者开销时, 省去了数域上的操作, 因其代价相比椭圆曲线群上的操作相比可忽略，n 是需要证明的密文数，在很多实际应用中, 单个证明者需要对多个密文通过 gadgets-1/2 进行区间范围证明，当密文数量为百万量级时, 使用 twisted ElGamal 带来的性能提升是相当可观的。关于更详细的原理请查阅陈宇教授的论文：[https://pdfs.semanticscholar.org/de6b/2e11feaa28addb12fa4bfa72cf2a619cb03b.pdf](https://pdfs.semanticscholar.org/de6b/2e11feaa28addb12fa4bfa72cf2a619cb03b.pdf)<br />铜锁开源密码学算法库对 Bulletproofs 和 twisted ElGamal 进行了完善的支持，在铜锁公众号中也发表过相关的使用教程文档，例如：<br />铜锁支持 Bulletproofs 算法：[https://mp.weixin.qq.com/s/eI0Dky-cG5biKjT-EelnHQ](https://mp.weixin.qq.com/s/eI0Dky-cG5biKjT-EelnHQ)<br />铜锁 Bulletproofs Range 使用教程：[https://mp.weixin.qq.com/s/oA2L5TduOti1R2WnqOwapQ](https://mp.weixin.qq.com/s/oA2L5TduOti1R2WnqOwapQ) <br />铜锁 Bulletproofs R1CS 使用教程：[https://mp.weixin.qq.com/s/vKRl4fLyEuNF78JcqYYVfw](https://mp.weixin.qq.com/s/vKRl4fLyEuNF78JcqYYVfw) <br />此外，如果对代码实现有兴趣也请直接查阅铜锁 zkp 算法的实现：[https://github.com/Tongsuo-Project/Tongsuo/tree/master/crypto/zkp](https://github.com/Tongsuo-Project/Tongsuo/tree/master/crypto/zkp)

