<a name="T6jez"></a>
## 背景
R1CS 的背景源于零知识证明和可验证计算领域的研究，它是一种用于表示计算约束的形式化模型，可以描述计算中的输入、输出和约束条件之间的关系。R1CS 的设计旨在使计算过程的约束能够以一种有效和紧凑的方式进行表示，并且能够应用于零知识证明的构建。<br />在 Bulletproofs 算法中，R1CS（Rank-1 Constraint System）是一种用于构建约束的模型，用于描述计算过程中的约束关系。R1CS 提供了一种方式来表达计算中的约束，为构建零知识证明和保护隐私提供了基础。Bulletproofs 利用 R1CS 模型来构建零知识证明，从而实现在不泄露任何有关输入和计算过程的信息的情况下验证计算的正确性。<br />Bulletproofs 需要熟悉其基础概念才能更好的掌握和使用，请移步：[https://www.yuque.com/tsdoc/ts/bulletproofs](https://www.yuque.com/tsdoc/ts/bulletproofs)
<a name="dgHOb"></a>
## 命令行
<a name="RoXBu"></a>
### 公共参数（-ppgen/-pp）

- 生成公共参数
```bash
$ tongsuo bulletproofs -ppgen -out ./pp.pem -curve_name sm2 -gens_capacity 64 -party_capacity 1
```
参数说明，其中 bulletproofs 是 tongsuo 的子命令：bulletproofs 功能。不再赘述。

   - -ppgen：是 bulletproofs 的子命令，指公共参数的生成，pp 是 pub_param 的缩写，gen 是 generate 的缩写；
   - -out：输出文件路径；
   - -curve_name：椭圆曲线名称；
   - -gens_capacity：椭圆曲线点生成器的容量，对于 r1cs 来说，该数须大于约束条件表达式中乘法变量的数量；
   - -party_capacity：可以生成聚合证明的最大参与方数量，仅对 range proof 有效，r1cs proof 设置为1即可。
- 文本显示公共参数

-pp 是用来显示公共参数的子命令。
```bash
$ tongsuo bulletproofs -pp -in ./pp.pem -text -noout
Bulletproofs Public Parameter:
curve: SM2 (1172)
gens_capacity: 64
party_capacity: 1
G[n]:
    [0]: 02:a3:df:fe:3f:b6:88:0e:34:79:be:25:71:b2:03:97:0f:86:58:78:57:32:4d:b4:fa:70:0e:2f:0c:3b:d0:ea:85
    [1]: 03:42:ee:45:da:e7:ee:84:78:8f:18:03:42:c4:0f:9e:0e:93:d8:82:54:08:a4:1a:30:38:bc:94:57:e7:9e:8b:61
    [2]: 03:8a:7c:c7:fb:87:07:b0:a9:fe:65:1e:2a:b7:6a:98:02:0e:3d:1a:43:7d:7c:1a:d4:62:60:ce:00:87:e3:ec:04
    [3]: 03:85:1b:e5:2f:be:1a:91:9c:6a:42:db:22:e0:35:7e:b8:61:67:56:f6:f4:04:51:2d:8d:13:82:58:ec:5a:2d:ec
    # 太长了，中间省略了 #
    [60]: 02:21:70:bc:20:33:2e:0b:b3:27:8d:c2:f3:38:f3:a6:8c:8b:05:5d:0a:15:df:ed:5f:ec:2a:3b:bf:d1:4c:fe:eb
    [61]: 03:53:89:6e:6e:8a:9e:1f:b6:54:63:50:c5:59:14:c5:1f:c2:c4:32:35:d4:8f:43:22:25:cd:3d:52:25:ee:ea:d1
    [62]: 03:e4:f5:ff:b3:25:f6:ff:d7:ed:01:db:17:fa:8b:9c:c6:79:84:e3:6a:bf:42:17:b6:7b:9e:f7:d1:3e:7d:41:54
    [63]: 02:e8:de:7e:27:21:0d:95:03:5b:f1:0e:1d:04:cb:f0:cd:1d:96:25:e7:4f:8b:49:0c:fb:08:cb:e6:a9:af:7f:0a
H[n]:
    [0]: 03:e5:df:a2:8d:50:4e:d5:7f:64:a6:e5:88:a9:17:de:ff:d8:92:2b:57:1c:25:90:2e:8e:5d:ae:b2:21:f1:68:b4
    [1]: 02:30:95:72:37:7c:9f:4c:ec:24:0d:fc:ff:08:38:77:73:49:e8:19:a8:84:cb:6f:0a:24:9e:39:31:01:a0:19:91
    [2]: 03:a3:d1:84:c0:6d:c8:fa:1f:35:6c:65:75:e5:ff:63:45:16:29:bc:ee:5e:71:51:97:44:dd:03:15:bd:4f:bf:c3
    [3]: 03:c3:06:44:b3:34:5c:ac:d7:ed:91:aa:24:e1:0a:cb:f7:04:61:12:d9:3d:bf:65:98:8b:f4:87:8d:9e:7f:b7:6e
    # 太长了，中间省略了 #
    [60]: 03:5f:30:c3:c7:29:02:38:18:5e:d2:ae:34:44:7f:44:52:fb:d1:97:24:7e:a6:7c:9e:f8:16:3f:f9:18:f3:7b:69
    [61]: 02:ff:55:39:f5:be:2e:fb:29:b4:36:e6:f5:ce:d4:55:67:e9:59:3b:53:8a:af:ea:5b:6c:c6:db:45:de:5b:8b:71
    [62]: 03:e2:2f:0c:fa:30:d2:ec:33:5e:aa:d0:c4:ba:7e:4e:6c:03:50:0f:26:27:72:ae:f0:d5:d8:ad:09:24:d0:94:de
    [63]: 02:64:af:b1:25:f4:85:c4:6c:da:fe:e3:b6:24:2a:0d:ad:f9:8b:56:96:6c:6b:3e:f1:f5:ef:92:9d:b2:52:8c:40
U: 03:85:44:28:a8:55:f4:db:cd:1b:60:42:d0:21:5b:31:ea:0e:fb:b2:2e:46:3d:a3:f0:40:48:50:05:0a:a3:a9:ec
H: 02:e5:21:75:20:47:fb:c2:76:c2:86:3c:10:e0:4c:1e:72:66:26:7b:3d:5d:ac:5b:a0:de:ab:11:d3:56:bd:fc:f1
```
如上结果所示，G[n] 和 H[n] 就是随机生成椭圆曲线 SM2 上的点，n = gens_capacity * party_capacity，上图是 64，中间一些随机点由于太长太点篇幅省略掉了。
<a name="rkBc9"></a>
### 证据（-witness）

- 生成证据
```bash
$ tongsuo bulletproofs -witness -pp_in ./pp.pem -out witness_r1cs.pem -r1cs a=5 b=25
```
参数说明：

   - -witness 是用来生成/显示证据的子命令
   - -pp_in：指定公共参数文件所在路径
   - -out：生成的证据文件路径
   - -r1cs：是指定生成的是 r1cs 证据，否则生成的是 range proof 的证据
   - a=5 b=25 是有名变量，多个变量用空格隔开，=号前面是变量名，=号后面是变量值，目前仅支持整数（正/负均可）
- 显示证据
```bash
$ tongsuo bulletproofs -witness -in witness_r1cs.pem -text
Witness:
curve: SM2 (1172)
H: 02:e5:21:75:20:47:fb:c2:76:c2:86:3c:10:e0:4c:1e:72:66:26:7b:3d:5d:ac:5b:a0:de:ab:11:d3:56:bd:fc:f1
V[n]:
    [a]: 02:2b:8b:0d:2c:79:f2:d6:8b:cb:11:40:6c:af:ee:f1:3f:32:f9:c9:11:68:92:81:a0:f8:c0:e9:c0:2c:e8:fe:72
    [b]: 03:fb:d3:ce:8d:bb:d0:b3:71:f3:c6:3f:01:35:98:61:97:eb:af:da:08:00:7b:b3:15:35:6c:2a:a9:da:81:0a:06
v[n]:
    [a]: 5 (0x5)
    [b]: 25 (0x19)
r[n]:
    [0]: 00:c7:05:43:9f:4c:72:46:61:60:36:04:5e:40:c7:1a:b2:2e:49:de:f1:b0:d1:9d:26:15:ae:ea:e3:73:a1:12:fc
    [1]: 6f:3c:4c:9c:33:d3:a5:64:8e:19:4b:62:ad:43:81:f7:67:13:8d:47:f6:a7:cb:13:26:c4:03:e7:83:a7:1f:07
-----BEGIN BULLETPROOFS WITNESS-----
AAAElALlIXUgR/vCdsKGPBDgTB5yZiZ7PV2sW6DeqxHTVr388QAAAAICK4sNLHny
1ovLEUBsr+7xPzL5yRFokoGg+MDpwCzo/nJhAAP7086Nu9CzcfPGPwE1mGGX66/a
CAB7sxU1bCqp2oEKBmIAAAAAAivHBUOfTHJGYWA2BF5AxxqyLkne8bDRnSYVrurj
c6ES/CtvPEycM9OlZI4ZS2KtQ4H3ZxONR/anyxMmxAPng6cfBwAAAAIrAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAUrAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAABk=
-----END BULLETPROOFS WITNESS-----
```
如上结果所示，witness 中包含了椭圆曲线名称，H 点（与公共参数一致），密文证据 V，明文证据 v 和随机数 r。
<a name="gDpy7"></a>
### 证明（-prove/-proof）

- 生成 r1cs 证明（-prove）
```bash
$ tongsuo bulletproofs -prove -pp_in ./pp.pem -witness_in witness_r1cs.pem -out proof-r1cs.pem -r1cs_constraint "a*a-b"

$ tongsuo bulletproofs -prove -pp_in ./pp.pem -witness_in witness_r1cs.pem -out proof-r1cs.pem -r1cs_constraint "a*a-c"
May be extra arguments error, please use -help for usage summary.
80C58E0401000000:error:1F800070:lib(63):bp_r1cs_expression_evaluate_variable:reason(112):crypto/zkp/bulletproofs/r1cs_constraint_expression.c:266:
80C58E0401000000:error:1F80006F:lib(63):bp_r1cs_expression_process:reason(111):crypto/zkp/bulletproofs/r1cs_constraint_expression.c:302:
```
参数说明：

   - -pp_in：指定公共参数文件所在路径
   - -witness_in：指定证据文件所在路径
   - -out：指定证明的输出路径，输出的文件包含两段：R1CS PROOF 和 WITNESS，分别为 proof 和 witness
   - -r1cs_constraint：指定约束条件，里面引用的变量必须在生成证据时提交的变量名中，否则报错，可以看出第二条命令报错了
- 显示证明（-proof）
```bash
$ tongsuo bulletproofs -proof -in proof-r1cs.pem -text
R1CS Proof:
AI1: 03:1f:ed:4a:3a:ee:f3:db:a2:6a:b3:b8:96:bc:9c:c4:3d:47:69:55:96:a4:51:58:fe:99:f6:fa:db:d4:8c:42:f1
AO1: 02:c1:96:7b:71:29:61:8a:e1:cf:9b:35:ef:87:da:f0:11:51:58:cd:7e:40:49:7d:48:a2:5d:c0:f3:26:f7:07:51
S1: 03:da:33:9d:60:e5:07:a8:fc:f6:5b:ee:82:81:cf:28:6f:34:3f:46:04:c3:f7:93:e2:8b:0e:7a:c7:c1:05:ed:32
AI2: 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
AO2: 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
S2: 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
T1: 03:77:c3:ac:ef:56:09:18:a5:88:9a:f1:92:7c:f9:9a:ee:71:d2:66:75:5e:9c:08:9e:e1:a1:41:bb:d3:3e:3c:1c
T3: 02:dd:4b:41:66:76:e6:be:59:3a:5b:6e:3e:a5:2a:10:24:b8:9f:b9:70:b0:12:09:a9:0c:72:5d:b8:82:b9:12:cf
T4: 02:f9:f8:ec:7b:36:6a:b4:95:d2:5b:6b:01:ff:6e:bf:9d:74:a9:a0:3d:0d:52:e2:3b:76:70:11:57:99:fc:9e:2a
T5: 03:03:de:a0:ef:5f:9e:dd:17:24:e3:4d:2b:1b:02:fc:ff:2b:fa:13:4a:58:ea:72:26:8f:15:b9:6a:c2:10:bf:1a
T6: 03:dc:0b:69:a2:95:8a:f5:8d:50:5c:4e:c4:82:59:a6:e8:d6:fc:b1:c0:28:c5:27:d2:71:61:07:bf:46:61:f9:32
taux: 00:8a:1e:5c:6d:8d:99:93:53:e9:6c:4c:a8:f5:c4:11:eb:df:51:36:3d:14:84:0a:0e:66:70:76:23:0f:8b:35:96
mu: 00:f1:68:3f:51:06:02:a4:48:3d:06:6e:47:00:2f:39:62:54:c0:7c:88:c1:37:e3:df:8c:01:a2:05:e3:15:d5:bf
tx: 00:dd:60:0f:98:1e:1b:d5:9b:0b:c2:99:b4:98:4d:53:af:d3:1b:d3:31:9f:43:48:a0:90:80:07:68:dd:5b:35:96
inner proof:
    n: 0
    L[n]:
    R[n]:
    a: 00:e6:9f:e7:68:89:5a:0c:74:99:d8:b3:59:51:95:d0:f2:93:5b:66:ca:eb:72:07:57:04:6b:5f:81:3b:32:ed:6a
    b: 44:dd:57:f4:2f:3a:4d:b8:f3:20:93:5b:5d:dc:04:ff:4f:62:f8:26:66:e4:5f:61:28:e5:45:5e:4f:81:72:4d
-----BEGIN BULLETPROOFS R1CS PROOF-----
AAAElAAAAAgDH+1KOu7z26Jqs7iWvJzEPUdpVZakUVj+mfb629SMQvECwZZ7cSlh
iuHPmzXvh9rwEVFYzX5ASX1Iol3A8yb3B1ED2jOdYOUHqPz2W+6Cgc8obzQ/RgTD
95Piiw56x8EF7TIDd8Os71YJGKWImvGSfPma7nHSZnVenAie4aFBu9M+PBwC3UtB
Znbmvlk6W24+pSoQJLifuXCwEgmpDHJduIK5Es8C+fjsezZqtJXSW2sB/26/nXSp
oD0NUuI7dnARV5n8nioDA96g71+e3Rck400rGwL8/yv6E0pY6nImjxW5asIQvxoD
3AtpopWK9Y1QXE7Eglmm6Nb8scAoxSfScWEHv0Zh+TIAAAADK4oeXG2NmZNT6WxM
qPXEEevfUTY9FIQKDmZwdiMPizWWK/FoP1EGAqRIPQZuRwAvOWJUwHyIwTfj34wB
ogXjFdW/K91gD5geG9WbC8KZtJhNU6/TG9Mxn0NIoJCAB2jdWzWWAAAAAivmn+do
iVoMdJnYs1lRldDyk1tmyutyB1cEa1+BOzLtaitE3Vf0LzpNuPMgk1td3AT/T2L4
JmbkX2Eo5UVeT4FyTQAAAAAAAAAA
-----END BULLETPROOFS R1CS PROOF-----
Witness:
curve: SM2 (1172)
H: 02:e5:21:75:20:47:fb:c2:76:c2:86:3c:10:e0:4c:1e:72:66:26:7b:3d:5d:ac:5b:a0:de:ab:11:d3:56:bd:fc:f1
V[n]:
    [a]: 02:2b:8b:0d:2c:79:f2:d6:8b:cb:11:40:6c:af:ee:f1:3f:32:f9:c9:11:68:92:81:a0:f8:c0:e9:c0:2c:e8:fe:72
    [b]: 03:fb:d3:ce:8d:bb:d0:b3:71:f3:c6:3f:01:35:98:61:97:eb:af:da:08:00:7b:b3:15:35:6c:2a:a9:da:81:0a:06
-----BEGIN BULLETPROOFS WITNESS-----
AAAElALlIXUgR/vCdsKGPBDgTB5yZiZ7PV2sW6DeqxHTVr388QAAAAICK4sNLHny
1ovLEUBsr+7xPzL5yRFokoGg+MDpwCzo/nJhAAP7086Nu9CzcfPGPwE1mGGX66/a
CAB7sxU1bCqp2oEKBmIA
-----END BULLETPROOFS WITNESS-----
```
可以看出 proof-range.pem 中包含了两段：R1CS PROOF 和 WITNESS，这里 WITNESS 的 V 就是密文证据。
<a name="mKwbJ"></a>
### 验证（-verify）
```bash
$ tongsuo bulletproofs -verify -pp_in ./pp.pem -in proof-r1cs.pem -r1cs_constraint "a*a-b"
The proof is valid

$ tongsuo bulletproofs -verify -pp_in ./pp.pem -in proof-r1cs.pem -r1cs_constraint "a*a-b+a*a-b"
The proof is invalid

$ tongsuo bulletproofs -verify -pp_in ./pp.pem -in proof-r1cs.pem -r1cs_constraint "a*5-b"
The proof is invalid
```
验证 r1cs 的 proof 时需要指定相同的约束条件参数（-r1cs_constraint），否则验证失败。需要注意的是表达式一定要一致，即使 a 和 b 的其他运算也能为0，但不一致的约束条件会导致内部运算逻辑与 prove 不一致而验证失败。<br />如上结果，可以看出，与 prove 一致的约束条件的验证结果有效，其他无效。
<a name="rppgC"></a>
## Demo
<a name="wpEZB"></a>
### 代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <openssl/bulletproofs.h>

typedef int (*r1cs_logic_func)(BP_R1CS_CTX *ctx,
                               BP_R1CS_LINEAR_COMBINATION *a1,
                               BP_R1CS_LINEAR_COMBINATION *a2,
                               BP_R1CS_LINEAR_COMBINATION *b1,
                               BP_R1CS_LINEAR_COMBINATION *b2,
                               BP_R1CS_LINEAR_COMBINATION *c1,
                               BP_R1CS_LINEAR_COMBINATION *c2);

/*
 * Constrains a1*a2 + b1*b2 = c1*c2
 */
static int r1cs_example_logic(BP_R1CS_CTX *ctx,
                              BP_R1CS_LINEAR_COMBINATION *a1,
                              BP_R1CS_LINEAR_COMBINATION *a2,
                              BP_R1CS_LINEAR_COMBINATION *b1,
                              BP_R1CS_LINEAR_COMBINATION *b2,
                              BP_R1CS_LINEAR_COMBINATION *c1,
                              BP_R1CS_LINEAR_COMBINATION *c2)
{
    int ret = 0;
    BP_R1CS_LINEAR_COMBINATION *a = NULL, *b = NULL, *c = NULL;

    if (!(a = BP_R1CS_LINEAR_COMBINATION_dup(a1))
        || !(b = BP_R1CS_LINEAR_COMBINATION_dup(b1))
        || !(c = BP_R1CS_LINEAR_COMBINATION_dup(c1)))
        return 0;

    /*
	 * 执行线性约束运算：
     * a = a1*a2
	 * b = b1*b2
     * c = c1*c2
	 * a + b - c = 0
	 */
    if (!BP_R1CS_LINEAR_COMBINATION_mul(a, a2, ctx)
        || !BP_R1CS_LINEAR_COMBINATION_mul(b, b2, ctx)
        || !BP_R1CS_LINEAR_COMBINATION_add(a, b)
        || !BP_R1CS_LINEAR_COMBINATION_mul(c, c2, ctx)
        || !BP_R1CS_LINEAR_COMBINATION_sub(a, c)
        || !BP_R1CS_LINEAR_COMBINATION_constrain(a, ctx))
        goto err;

    ret = 1;

err:
    BP_R1CS_LINEAR_COMBINATION_free(a);
    BP_R1CS_LINEAR_COMBINATION_free(b);
    BP_R1CS_LINEAR_COMBINATION_free(c);

    return ret;
}

static BP_R1CS_PROOF *r1cs_example_prove(BP_R1CS_CTX *ctx, BP_WITNESS *witness,
                                         BIGNUM *a1, BIGNUM *a2,
                                         BIGNUM *b1, BIGNUM *b2,
                                         BIGNUM *c1, BIGNUM *c2,
                                         r1cs_logic_func logic_func)
{
    BP_R1CS_LINEAR_COMBINATION *lc_a1 = NULL, *lc_a2 = NULL;
    BP_R1CS_LINEAR_COMBINATION *lc_b1 = NULL, *lc_b2 = NULL;
    BP_R1CS_LINEAR_COMBINATION *lc_c1 = NULL, *lc_c2 = NULL;
    BP_R1CS_PROOF *proof = NULL;

    /*
	 * 往 witness 中提交有名变量：a1、a2、b1、b2、c1、c2
     * 并获取相应的线性约束对象，然后传递给 logic_func 函数中做逻辑运算
	 */
    if (!(lc_a1 = BP_WITNESS_r1cs_linear_combination_commit(witness, "a1", a1))
        || !(lc_a2 = BP_WITNESS_r1cs_linear_combination_commit(witness, "a2", a2))
        || !(lc_b1 = BP_WITNESS_r1cs_linear_combination_commit(witness, "b1", b1))
        || !(lc_b2 = BP_WITNESS_r1cs_linear_combination_commit(witness, "b2", b2))
        || !(lc_c1 = BP_WITNESS_r1cs_linear_combination_commit(witness, "c1", c1))
        || !(lc_c2 = BP_WITNESS_r1cs_linear_combination_commit(witness, "c2", c2)))
        goto err;

    if (!logic_func(ctx, lc_a1, lc_a2, lc_b1, lc_b2, lc_c1, lc_c2))
        goto err;

    /*
	 * 逻辑运算完成后执行 prove 操作获取 proof 对象
	 */
    if (!(proof = BP_R1CS_PROOF_prove(ctx)))
        goto err;

    return proof;

err:
    BP_R1CS_LINEAR_COMBINATION_free(lc_a1);
    BP_R1CS_LINEAR_COMBINATION_free(lc_a2);
    BP_R1CS_LINEAR_COMBINATION_free(lc_b1);
    BP_R1CS_LINEAR_COMBINATION_free(lc_b2);
    BP_R1CS_LINEAR_COMBINATION_free(lc_c1);
    BP_R1CS_LINEAR_COMBINATION_free(lc_c2);
    BP_R1CS_PROOF_free(proof);
    return NULL;
}

static int r1cs_example_verify(BP_R1CS_CTX *ctx, BP_WITNESS *witness,
                               BP_R1CS_PROOF *proof, r1cs_logic_func logic_func)
{
    BP_R1CS_LINEAR_COMBINATION *lc_a1 = NULL, *lc_a2 = NULL;
    BP_R1CS_LINEAR_COMBINATION *lc_b1 = NULL, *lc_b2 = NULL;
    BP_R1CS_LINEAR_COMBINATION *lc_c1 = NULL, *lc_c2 = NULL;

    /*
	 * 从 witness 中获取有名变量（a1、a2、b1、b2、c1、c2）的线性约束，然后传递给 logic_func 函数中做逻辑运算
	 */
    if (!(lc_a1 = BP_WITNESS_r1cs_linear_combination_get(witness, "a1"))
        || !(lc_a2 = BP_WITNESS_r1cs_linear_combination_get(witness, "a2"))
        || !(lc_b1 = BP_WITNESS_r1cs_linear_combination_get(witness, "b1"))
        || !(lc_b2 = BP_WITNESS_r1cs_linear_combination_get(witness, "b2"))
        || !(lc_c1 = BP_WITNESS_r1cs_linear_combination_get(witness, "c1"))
        || !(lc_c2 = BP_WITNESS_r1cs_linear_combination_get(witness, "c2")))
        goto err;

    if (!logic_func(ctx, lc_a1, lc_a2, lc_b1, lc_b2, lc_c1, lc_c2))
        goto err;

    /*
	 * 逻辑运算完成后执行 verify 操作验证 proof 对象是否有效
	 */
    if (!BP_R1CS_PROOF_verify(ctx, proof))
        goto err;

    return 1;

err:
    BP_R1CS_LINEAR_COMBINATION_free(lc_a1);
    BP_R1CS_LINEAR_COMBINATION_free(lc_a2);
    BP_R1CS_LINEAR_COMBINATION_free(lc_b1);
    BP_R1CS_LINEAR_COMBINATION_free(lc_b2);
    BP_R1CS_LINEAR_COMBINATION_free(lc_c1);
    BP_R1CS_LINEAR_COMBINATION_free(lc_c2);
    return 0;
}

static int r1cs_proof_test(BIGNUM *a1, BIGNUM *a2, BIGNUM *b1, BIGNUM *b2, BIGNUM *c1, BIGNUM *c2,
                           r1cs_logic_func logic_func)
{
    int ret = 0;
    ZKP_TRANSCRIPT *transcript = NULL;
    BP_PUB_PARAM *pp = NULL;
    BP_WITNESS *witness = NULL;
    BP_R1CS_CTX *ctx = NULL;
    BP_R1CS_PROOF *proof = NULL;

    /* 创建交互抄本对象，证明者和验证者需要使用相同的方法和标签，否则验证失败 */
    transcript = ZKP_TRANSCRIPT_new(ZKP_TRANSCRIPT_METHOD_sha256(), "r1cs_test");

    /* 创建公共参数对象 */
    pp = BP_PUB_PARAM_new_by_curve_id(NID_secp256k1, 128, 1);

    /* 创建该公共参数下的证据对象 */
    witness = BP_WITNESS_new(pp);
    ctx = BP_R1CS_CTX_new(pp, witness, transcript);
    if (transcript == NULL || pp == NULL || witness == NULL || ctx == NULL)
        goto err;

    proof = r1cs_example_prove(ctx, witness, a1, a2, b1, b2, c1, c2, logic_func);
    if (proof == NULL)
        goto err;

    if (!r1cs_example_verify(ctx, witness, proof, logic_func))
        goto err;

    ret = 1;
err:
    BP_R1CS_PROOF_free(proof);
    BP_R1CS_CTX_free(ctx);
    ZKP_TRANSCRIPT_free(transcript);
    BP_PUB_PARAM_free(pp);

    return ret;

}

int main(int argc, char *argv[])
{
    int i, a1, a2, b1, b2, c1, c2;
    char *name, *value;
    BIGNUM *bn_a1, *bn_a2, *bn_b1, *bn_b2, *bn_c1, *bn_c2;
    BN_CTX *bn_ctx = NULL;

    printf("test r1cs constraint: a1*a2 + b1*b2 = c1*c2\n");

    if (argc != 7) {
        printf("Invalid parameter!\n");
        return -1;
    }

    for (i = 1; i < argc; i++) {
        name = argv[i];
        value = name + 3;
        if (strncmp(name, "a1=", 3) == 0) {
            a1 = atoi(value);
        } else if (strncmp(name, "a2=", 3) == 0) {
            a2 = atoi(value);
        } else if (strncmp(name, "b1=", 3) == 0) {
            b1 = atoi(value);
        } else if (strncmp(name, "b2=", 3) == 0) {
            b2 = atoi(value);
        } else if (strncmp(name, "c1=", 3) == 0) {
            c1 = atoi(value);
        } else if (strncmp(name, "c2=", 3) == 0) {
            c2 = atoi(value);
        } else {
            printf("Unknow parameter: %s\n", name);
            return -1;
        }
    }

    if (!(bn_ctx = BN_CTX_new()))
        goto err;

    bn_a1 = BN_CTX_get(bn_ctx);
    bn_a2 = BN_CTX_get(bn_ctx);
    bn_b1 = BN_CTX_get(bn_ctx);
    bn_b2 = BN_CTX_get(bn_ctx);
    bn_c1 = BN_CTX_get(bn_ctx);
    bn_c2 = BN_CTX_get(bn_ctx);
    if (bn_c2 == NULL)
        goto err;

    BN_set_word(bn_a1, a1);
    BN_set_word(bn_a2, a2);
    BN_set_word(bn_b1, b1);
    BN_set_word(bn_b2, b2);
    BN_set_word(bn_c1, c1);
    BN_set_word(bn_c2, c2);

    if (r1cs_proof_test(bn_a1, bn_a2, bn_b1, bn_b2, bn_c1, bn_c2, r1cs_example_logic))
        printf("test ok, %d*%d + %d*%d = %d*%d\n", a1, a2, b1, b2, c1, c2);
    else
        printf("test failed, %d*%d + %d*%d != %d*%d\n", a1, a2, b1, b2, c1, c2);

err:
    BN_CTX_free(bn_ctx);
    return 0;
}
```
如上代码注释所示，一次完整的 R1CS 调用涉及6个数据结构，分别为：

- `ZKP_TRANSCRIPT`

该数据结构是交互抄本的结构，用来模拟交互式零知识方案中的交互，是 Bulletproofs 为非交互式零知识证明算法的关键，利用了 Fiat-Shamir 变换将其转变为随机预言机模型中的非交互式零知识方案。

   - 创建/释放
```c
/*
 * 创建 ZKP_TRANSCRIPT
 * 参数：
 *    method：目前仅实现了 sha256 的方法，通过调用 ZKP_TRANSCRIPT_METHOD_sha256() 获得，后续可以实现其他更高效的方法
 *    label：证明者和验证者的标签/标识
 */
ZKP_TRANSCRIPT *ZKP_TRANSCRIPT_new(const ZKP_TRANSCRIPT_METHOD *method,
                                   const char *label);

/*
 * 释放 ZKP_TRANSCRIPT
 */
void ZKP_TRANSCRIPT_free(ZKP_TRANSCRIPT *transcript);
```

- `BP_PUB_PARAM`

该数据结构用来保存 Bulletproofs 的公共参数，证明者和验证者需要在同一个公共参数下才有意义，否则证明者生成的证明验证者永远无法验证通过。

   - 创建/释放
```c
/*
 * 创建 BP_PUB_PARAM
 * 参数：
 *    gens_capacity：对于 r1cs proof 来，该数须大于约束条件表达式中乘法变量的数量
 *    party_capacity：仅对 range proof 有效，对 r1cs proof 设置为1即可
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

   - 提交/获取
```c
/*
 * 提交变量到证据中
 * 参数：
 *    witness：证据对象
 * 	  name：变量名称，一般由证明者绑定变量名称和变量值，验证者通过变量名称获取到对应的密文变量值（即密文证据），转换成线性组合后可以进行约束表达式的运算
 * 	  v：变量值，也就是明文证据值
 */
int BP_WITNESS_r1cs_commit(BP_WITNESS *witness, const char *name, BIGNUM *v);

/*
 * 提交变量到证据中，并获取线性组合对象（用于约束表达式的运算），一般由证明者调用
 * 参数：
 *    witness：证据对象
 * 	  name：变量名称
 * 	  v：变量值
 */
BP_R1CS_LINEAR_COMBINATION *BP_WITNESS_r1cs_linear_combination_commit(BP_WITNESS *witness, const char *name, BIGNUM *v);

/*
 * 从 witness 中获取变量名为 name 的变量，并转换成线性组合后返回线性组合的对象指针，一般由验证者调用
 * 参数：
 *    witness：证据对象
 * 	  name：变量名称
 */
BP_R1CS_LINEAR_COMBINATION *BP_WITNESS_r1cs_linear_combination_get(BP_WITNESS *witness, const char *name);
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

- `BP_R1CS_LINEAR_COMBINATION`

R1CS 线性组合数据结构，用来保存各个变量的组合关系。

   - 创建/释放
```c
/*
 * 创建线性组合对象
 */
BP_R1CS_LINEAR_COMBINATION *BP_R1CS_LINEAR_COMBINATION_new(void);

/*
 * 复制线性组合对象
 */
BP_R1CS_LINEAR_COMBINATION *BP_R1CS_LINEAR_COMBINATION_dup(const BP_R1CS_LINEAR_COMBINATION *lc);

/*
 * 释放线性组合对象内存
 */
void BP_R1CS_LINEAR_COMBINATION_free(BP_R1CS_LINEAR_COMBINATION *lc);

/*
 * 清空线性组合对象，复用对象之前需要清空一下
 */
int BP_R1CS_LINEAR_COMBINATION_clean(BP_R1CS_LINEAR_COMBINATION *lc);
```

   - 加/减/乘运算
```c
/*
 * 线性组合底层乘法，将 l 和 r 分别转换成线性组合 left 和 right，并将 l * r 的结果转换成线性组合 output 
 * 参数：
 * 	  output：用来保存 l * r 转换成线性组合的指针
 * 	  left：用来保存 l 转换成线性组合的指针
 * 	  right：用来保存 r 转换成线性组合的指针
 * 	  l：乘法左边参数
 * 	  r：乘法右边参数
 * 	  ctx：上下文对象
 */
int BP_R1CS_LINEAR_COMBINATION_raw_mul(BP_R1CS_LINEAR_COMBINATION **output,
                                       BP_R1CS_LINEAR_COMBINATION **left,
                                       BP_R1CS_LINEAR_COMBINATION **right,
                                       const BIGNUM *l, const BIGNUM *r,
                                       BP_R1CS_CTX *ctx);

/*
 * 线性组合乘法，执行：lc *= other
 * 参数：
 * 	  lc：乘法第一个参数及保存乘法结果：lc *= other
 * 	  other：乘法另一个参数
 * 	  ctx：上下文对象
 */
int BP_R1CS_LINEAR_COMBINATION_mul(BP_R1CS_LINEAR_COMBINATION *lc,
                                   const BP_R1CS_LINEAR_COMBINATION *other,
                                   BP_R1CS_CTX *ctx);

/*
 * 线性组合加法，执行：lc += other
 * 参数：
 * 	  lc：加法第一个参数及保存加法结果：lc += other
 * 	  other：加法另一个参数
 */
int BP_R1CS_LINEAR_COMBINATION_add(BP_R1CS_LINEAR_COMBINATION *lc,
                                   const BP_R1CS_LINEAR_COMBINATION *other);


/*
 * 线性组合减法，执行：lc -= other
 * 参数：
 * 	  lc：减法第一个参数及保存减法结果：lc -= other
 * 	  other：减法另一个参数
 */
int BP_R1CS_LINEAR_COMBINATION_sub(BP_R1CS_LINEAR_COMBINATION *lc,
                                   const BP_R1CS_LINEAR_COMBINATION *other);

/*
 * 线性组合取负数，执行：lc = -lc
 * 参数：
 * 	  lc：线性组合
 */
int BP_R1CS_LINEAR_COMBINATION_neg(BP_R1CS_LINEAR_COMBINATION *lc);

/*
 * 线性组合标量乘法，执行：lc *= value
 * 参数：
 * 	  lc：乘法第一个参数及保存乘法结果：lc *=  value
 * 	  value：常数标量
 */
int BP_R1CS_LINEAR_COMBINATION_mul_bn(BP_R1CS_LINEAR_COMBINATION *lc,
                                      const BIGNUM *value);

/*
 * 线性组合标量加法，执行：lc += value
 * 参数：
 * 	  lc：加油第一个参数及保存加法结果：lc +=  value
 * 	  value：常数标量
 */
int BP_R1CS_LINEAR_COMBINATION_add_bn(BP_R1CS_LINEAR_COMBINATION *lc,
                                      const BIGNUM *value);

/*
 * 线性组合标量减法，执行：lc -= value
 * 参数：
 * 	  lc：减法第一个参数及保存减法结果：lc -=  value
 * 	  value：常数标量
 */
int BP_R1CS_LINEAR_COMBINATION_sub_bn(BP_R1CS_LINEAR_COMBINATION *lc,
                                      const BIGNUM *value);
```

   - 约束
```c
/*
 * 线性组合约束，线性组合运算操作执行结束后需要对最终的线性组合结果进行约束，目前仅支持约束为0
 * 参数：
 * 	  lc：线性组合运算结果的对象
 * 	  ctx：上下文对象
 */
int BP_R1CS_LINEAR_COMBINATION_constrain(BP_R1CS_LINEAR_COMBINATION *lc, BP_R1CS_CTX *ctx);
```

- `BP_R1CS_CTX`

range proof 操作上下文数据结构。

   - 创建/释放
```c
/*
 * 创建 BP_R1CS_CTX
 * 参数：
 *    pp：公共参数对象
 *    witness：证据对象
 * 	  transcript：交互抄本对象
 */
BP_R1CS_CTX *BP_R1CS_CTX_new(BP_PUB_PARAM *pp, BP_WITNESS *witness, ZKP_TRANSCRIPT *transcript);

/*
 * 释放 BP_R1CS_CTX
 */
void BP_R1CS_CTX_free(BP_RANGE_CTX *ctx);
```

- `BP_R1CS_PROOF`

该数据结构用来保存 Bulletproofs 执行生成证明算法的计算结果，称为证明或者证明对象，由证明者生成，编码后发送给验证者来验证其有效性。

   - 创建/释放
```c
/*
 * 创建 BP_R1CS_PROOF
 * 参数：
 *    pp：公共参数对象
 */
BP_R1CS_PROOF *BP_R1CS_PROOF_new(const BP_PUB_PARAM *pp);

/*
 * 释放 BP_R1CS_PROOF
 */
void BP_R1CS_PROOF_free(BP_R1CS_PROOF *proof);
```

   - 编码/解码
```c
/*
 * 编码 BP_R1CS_PROOF 成二进制格式，写入 out 指定的内存区域中
 * 参数：
 *    proof：证明对象
 * 	  out：要写入的内存指针，如果为 NULL 则返回需要的内存大小，用于申请所需的内存
 *    size：out 内存区域大小
 */
size_t BP_R1CS_PROOF_encode(const BP_R1CS_PROOF *proof, unsigned char *out, size_t size);

/*
 * 解码 BP_R1CS_PROOF
 * 参数：
 *    in：已编码的证明内存指针
 *    size：in 指针内存区域大小
 */
BP_R1CS_PROOF *BP_R1CS_PROOF_decode(const unsigned char *in, size_t size);
```

   - 证明/验证
```c
/*
 * 证明，执行证明算法并创建和返回 proof 对象
 * 参数：
 *    ctx：上下文对象
 * 返回值：成功返回 proof 对象，否则返回 NULL
 */
BP_R1CS_PROOF *BP_R1CS_PROOF_prove(BP_R1CS_CTX *ctx);

/*
 * 验证
 * 参数：
 *    ctx：上下文对象
 *    proof： 证明对象，由证明者生成并发送给验证者来验证
 * 返回值：成功返回1，否则返回0
 */
int BP_R1CS_PROOF_verify(BP_R1CS_CTX *ctx, const BP_R1CS_PROOF *proof);
```
<a name="vjsQg"></a>
### 编译
```bash
$ gcc -Wall -g -o r1cs_proof ./r1cs_proof.c -I/opt/tongsuo-bulletproofs/include -L/opt/tongsuo-bulletproofs/lib -lcrypto
```
将 /opt/tongsuo-bulletproofs 替换成你的铜锁安装目录即可
<a name="PLlq4"></a>
### 运行
```bash
$ ./r1cs_proof a1=2 a2=4 b1=4 b2=5 c1=4 c2=7
test r1cs constraint: a1*a2 + b1*b2 = c1*c2
test ok, 2*4 + 4*5 = 4*7

$ ./r1cs_proof a1=2 a2=4 b1=4 b2=5 c1=4 c2=8
test r1cs constraint: a1*a2 + b1*b2 = c1*c2
test failed, 2*4 + 4*5 != 4*8

$ ./r1cs_proof a1=5 a2=6 b1=3 b2=5 c1=5 c2=9
test r1cs constraint: a1*a2 + b1*b2 = c1*c2
test ok, 5*6 + 3*5 = 5*9

$ ./r1cs_proof a1=5 a2=6 b1=3 b2=5 c1=5 c2=10
test r1cs constraint: a1*a2 + b1*b2 = c1*c2
test failed, 5*6 + 3*5 != 5*10
```
如上结果所示，约束条件表达式为：`a1*a2 + b1*b2 = c1*c2`，如果输入的变量满足等式则验证成功，否则验证失败。
