> 本文旨在介绍本系列文章第一篇《综述》中涉及的快速模约减算法的详细推导和实现过程

<a name="u4cRK"></a>
### 1 背景
在SM2数字签名算法中，有限域运算是椭圆曲线点运算和数字签名运算的基础，其运算性能将直接影响整体性能表现。而在有限域运算中，**模约减**运算的整体使用频率最高，在模乘、模平方、模逆等运算中均有运用。以模乘法为例，其定义为：对于素数域参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=fXHNL)和乘数![](https://intranetproxy.alipay.com/skylark/lark/__latex/26fdbf8e53cb0e48da5f4ddd4aaf5a5c.svg#card=math&code=a&id=z1MDa)，模乘运算的结果等于 ![](https://intranetproxy.alipay.com/skylark/lark/__latex/04cc3bc06b9e199ed432aff8e5ce2245.svg#card=math&code=a%20%2A%20a%20%5Cmod%20p&id=Khs2a)。也就是说，对于乘法结果大于![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=Ir6yf)的**中间值**，模乘法需要将其约减到![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=j6elu)以内，此过程因此被称为**模约减**。<br />由于![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=siSMV)是一个非常大的素数，且上述运算中间值最大可达![](https://intranetproxy.alipay.com/skylark/lark/__latex/c5df6dd4c72a34ca9c57a96c0ce3aaea.svg#card=math&code=p%5E2-1&id=T089r)，采用除法来计算模约减的效率将极其低下（在具体实践中，有限域除法一般借助乘法逆元实现，其耗时大约是模乘运算的200-300倍）。目前，铜锁项目中SM2算法使用了通用蒙哥马利约分算法实现模约减过程，暂时没有针对曲线特化的快速模约减算法实现。相应的，部分常用的NIST曲线`nistp224`、`nistp256`和`nistp521`基于广域梅森素数实现了曲线特化的快速模约减算法，并运用于各自的64-bit平台优化中。<br />考虑到SM2曲线同样采用广域梅森素数作为素数域参数，且实现64-bit平台优化也需要使用模约减算法，因此为SM2曲线实现曲线参数特化的快速模约减算法是非常有必要的。同时，为了保证对部分32位平台的兼容性和后续扩展，在功能开发的过程中保留了原有的条件编译模式，同时支持32位和64位平台使用该快速模约减算法。测试结果表明，使用了快速模约减的SM2模乘运算与简单模乘和蒙哥马利模乘相比，时间开销分别降低**81.34%**和**29.91%**。
<a name="fBZNl"></a>
### 2 预备知识
在数论中，**梅森素数（Mersenne Prime) **是形如![](https://intranetproxy.alipay.com/skylark/lark/__latex/2de7e0a6029776d73b108451d825c5e7.svg#card=math&code=2%5En%20%E2%88%92%201&id=SM8vg)的质数，其中![](https://intranetproxy.alipay.com/skylark/lark/__latex/df378375e7693bdcf9535661c023c02e.svg#card=math&code=n&id=Cl1vF)为整数。梅森素数的数量非常稀少，直到今天（2023年7月）仅有51个梅森素数被发现。由于梅森素数的构成与2的指数幂相关，在以二进制为数据表示方式的现代计算机中，它不仅便于计算和存储，也能够加速有限域运算，因此在密码学领域广受青睐。NIST曲线家族中，`nistp521`曲线的素数域参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=EzVJC)就选择了梅森素数![](https://intranetproxy.alipay.com/skylark/lark/__latex/c6d31b3f3aaa549c8126cb5f8fa47ee9.svg#card=math&code=2%5E%7B521%7D%20-1&id=bH5bW)。<br />然而，梅森素数的数量实在是过于稀少，以至于在椭圆曲线素数域参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=lnRx4)的合理取值范围内仅有一个梅森素数![](https://intranetproxy.alipay.com/skylark/lark/__latex/c6d31b3f3aaa549c8126cb5f8fa47ee9.svg#card=math&code=2%5E%7B521%7D%20-1&id=Mi3gB)，其他的梅森素数要么过大，要么太小，不满足使用要求。因此，许多椭圆曲线的素数域参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=a67TP)选用了梅森素数的拓展版本——**广义梅森素数（General Mersenne Prime) **。广义梅森素数是形如 ![](https://intranetproxy.alipay.com/skylark/lark/__latex/78d2ccc2fcf050f2d2fab354b29023ef.svg#card=math&code=f%282%5En%29&id=bQt5t)的素数，其中![](https://intranetproxy.alipay.com/skylark/lark/__latex/3818937ac554603003e6727783932e9f.svg#card=math&code=f%28x%29&id=xorVu)是具有较小整数系数的低次多项式。例如，`nistp256`曲线采用如下所示的广义梅森素数作为素数域参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=iKkzI)：<br />![](https://intranetproxy.alipay.com/skylark/lark/__latex/dfeb442e4f918db9a749affe618ba932.svg#card=math&code=p_%7Bnistp256%7D%20%3D%202%5E%7B256%7D%20%E2%88%92%202%5E%7B224%7D%20%2B%202%5E%7B192%7D%20%2B%202%5E%7B96%7D%20%E2%88%92%201&id=P66R4)
<a name="V8xrT"></a>
### 3 算法原理
在国标GM/T 0003.5-2012中，给出了sm2曲线的推荐素数域参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=MhLU9)。![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=pTbIa)是一个广义梅森素数，表示如下：<br />                                            ![](https://intranetproxy.alipay.com/skylark/lark/__latex/a52d525c1f15fd43e5f0d22ccac3bf83.svg#card=math&code=p_%7Bsm2p256%7D%20%3D%202%5E%7B256%7D%20%E2%88%92%202%5E%7B224%7D%20-%202%5E%7B96%7D%20%2B%202%5E%7B64%7D%20-%201&id=JiUw4)<br />下面以大整数![](https://intranetproxy.alipay.com/skylark/lark/__latex/fe6e0903d0b82e7f2d50cfbb7c02f324.svg#card=math&code=v%28v%20%3C%20p%5E2%20%3C%202%5E%7B512%7D%29&id=pg2IP)和参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/719d3c7a4c6a0ed2b84aaeb7a6572d8c.svg#card=math&code=p_%7Bsm2p256%7D&id=iAG48)为例，推导基于广义梅森素数的快速模约减算法。该算法利用参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/719d3c7a4c6a0ed2b84aaeb7a6572d8c.svg#card=math&code=p_%7Bsm2p256%7D&id=vJepG)各项系数之间最小差为![](https://intranetproxy.alipay.com/skylark/lark/__latex/967952ba4e1b290af0e1170cb711f7f9.svg#card=math&code=2%5E%7B32%7D&id=Au49K)的特点，将表示中间值的大整数按照32位分割，并转化为如下表示形式：<br />     ![](https://intranetproxy.alipay.com/skylark/lark/__latex/53e2c9348043018f708ac9c529d7a70f.svg#card=math&code=%28v%5B0%5D%2A2%5E0%20%2B%20v%5B1%5D%2A2%5E%7B32%7D%20%2B%20v%5B2%5D%2A2%5E%7B64%7D%20%2B%20...%20%2B%20v%5B15%5D%2A2%5E%7B480%7D%29%20%20%5Cmod%20p%0A%2C%E5%85%B6%E4%B8%AD0%20%5Cleq%20v%5Bi%5D%3C%202%5E%7B32%7D%20&id=oawVh)<br />然后将![](https://intranetproxy.alipay.com/skylark/lark/__latex/63504f1711a6fe7021912088ac182e8b.svg#card=math&code=v%5B8%5D%20-%20v%5B15%5D&id=XU2L4)的高位参数约化到![](https://intranetproxy.alipay.com/skylark/lark/__latex/308b899646a87220bfe36fe6a54af302.svg#card=math&code=v%5B0%5D%20-%20v%5B7%5D&id=vaBl4)的低位参数上。在约化之前，我们先根据将参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/719d3c7a4c6a0ed2b84aaeb7a6572d8c.svg#card=math&code=p_%7Bsm2p256%7D&id=z1JzX)转化为按照32位分割的表示形式：

| 2^288 | 2^256 | 2^224 | 2^192 | 2^160 | 2^128 | 2^096 | 2^064 | 2^032 | 2^000 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  | 1 | -1 | 0 | 0 | 0 | -1 | 1 | 0 | -1 |

接下来处理所有高位参数，此处以![](https://intranetproxy.alipay.com/skylark/lark/__latex/0b1b96f21b0f80ddcbc3f56820b9ffe0.svg#card=math&code=v%5B8%5D&id=mDxH3)为例，设![](https://intranetproxy.alipay.com/skylark/lark/__latex/980b58ff55442af881578507d1732ea4.svg#card=math&code=v%5B8%5D%20%3D%20a&id=fV9AU)，那么![](https://intranetproxy.alipay.com/skylark/lark/__latex/0b1b96f21b0f80ddcbc3f56820b9ffe0.svg#card=math&code=v%5B8%5D&id=FzsCN)可以表示为：

| 2^288 | 2^256 | 2^224 | 2^192 | 2^160 | 2^128 | 2^096 | 2^064 | 2^032 | 2^000 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | a | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

	注意到，根据模运算的运算规则，![](https://intranetproxy.alipay.com/skylark/lark/__latex/c2c3e906b673bbb4797c8947cc509f13.svg#card=math&code=a%20%2A%202%5E%7B256%7D%20%5Cmod%20p%20%5Cequiv%20%EF%BC%88a%2A2%5E%7B256%7D%20-%20x%2Ap%29%20%5Cmod%20p&id=xJvpO)，也就是说，在原数的基础上增减模数![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=p51ET)的倍数并不影响模运算的结果。那么，为了消去高位参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/26fdbf8e53cb0e48da5f4ddd4aaf5a5c.svg#card=math&code=a&id=RgEbC)，不妨减去一个![](https://intranetproxy.alipay.com/skylark/lark/__latex/f9c7506bc934b8cf1dedba39cc2d0400.svg#card=math&code=a%20%2Ap&id=eapGM)，此时有：

| 2^288 | 2^256 | 2^224 | 2^192 | 2^160 | 2^128 | 2^096 | 2^064 | 2^032 | 2^000 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | a | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
|  | a | -a | 0 | 0 | 0 | -a | a | 0 | -a |

上下相减，高位参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/0b1b96f21b0f80ddcbc3f56820b9ffe0.svg#card=math&code=v%5B8%5D&id=tf5TZ)被成功消除，低位约化的结果为：![](https://intranetproxy.alipay.com/skylark/lark/__latex/26a0161d43553eaf81191c1700cb1f0a.svg#card=math&code=a%20%2A%202%5E%7B224%7D%20%2B%20a%20%2A%202%5E%7B96%7D%20-%20a%20%2A%202%5E%7B64%7D%20%2B%20a&id=PNJYF)。依次对![](https://intranetproxy.alipay.com/skylark/lark/__latex/63504f1711a6fe7021912088ac182e8b.svg#card=math&code=v%5B8%5D%20-%20v%5B15%5D&id=jubT1)做低位约化即可消除所有高位参数。将所有的低位约化结果合并同类项，可得到如下的sm2快速模约减算法。该算法由[白国强等人](https://ieeexplore.ieee.org/document/7011249)于2014年首次提出，实际运用中也有多种不同的参数组合形式：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/38409340/1690527422905-245bea5a-c6e9-4eea-bdae-71cc7058cdaa.png#averageHue=%23f4f4f4&clientId=ue25066bc-c5f7-4&from=paste&height=449&id=CX4LE&originHeight=812&originWidth=794&originalType=binary&ratio=2&rotation=0&showTitle=false&size=233276&status=done&style=none&taskId=u98c3b141-3bc0-4973-b583-b1456421f62&title=&width=439)<br />图1：基于参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/719d3c7a4c6a0ed2b84aaeb7a6572d8c.svg#card=math&code=p_%7Bsm2p256%7D&id=suuIX)的快速模约减算法<br />最后对约化后的低位参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/308b899646a87220bfe36fe6a54af302.svg#card=math&code=v%5B0%5D%20-%20v%5B7%5D&id=Z3KaN)再做一次约减，旨在将模约减结果规约到![](https://intranetproxy.alipay.com/skylark/lark/__latex/49c75e46fc597079eb8921661d638490.svg#card=math&code=%5B0%2Cp%29&id=ymJrN)范围内。此处最多需要减去![](https://intranetproxy.alipay.com/skylark/lark/__latex/b913de27528f9c642e8bb1f43029baec.svg#card=math&code=13p&id=deYJC)即可得到最终结果，无需使用成本昂贵的模除法。从图1中不难看出，基于广义梅森素数的SM2快速模约减算法仅需要少量的有限域加法和减法即可实现，性能表现明显优于使用模逆元的模约减算法。
<a name="mUu1O"></a>
### 4 算法实现
（本节将介绍所实现算法的核心部分，完整算法实现请参见`crypto/bn/bn_sm2.c`。）<br />前文提到，在高位参数约化后，还需要对低位参数做约化以获得最终结果，也就是减去参数![](https://intranetproxy.alipay.com/skylark/lark/__latex/d4cd21d60552e207f237e82def9029b6.svg#card=math&code=p&id=O3cG3)的某个倍数![](https://intranetproxy.alipay.com/skylark/lark/__latex/0f8a2f05c01c9d853de55e9aaf0a42c3.svg#card=math&code=np%20%5Cquad%20%280%3C%3Dp%3C%3D13%29&id=YmGME)。为了进一步提高性能，在代码实现中我们构造一个预计算表以存储![](https://intranetproxy.alipay.com/skylark/lark/__latex/1c7c4939f0c7c88af4ea00fb49f4af54.svg#card=math&code=1-13p&id=Vctht)，以节约计算![](https://intranetproxy.alipay.com/skylark/lark/__latex/2479d2fb124fa29204536fe26877b873.svg#card=math&code=np&id=sNoiN)的开销。又因为约减结果在![](https://intranetproxy.alipay.com/skylark/lark/__latex/49c75e46fc597079eb8921661d638490.svg#card=math&code=%5B0%2Cp%29&id=LhmFO)范围内，且![](https://intranetproxy.alipay.com/skylark/lark/__latex/3dda6cb899835304408c4b9a15db9936.svg#card=math&code=p%20%3C%202%5E%7B256%7D&id=UZ37N),因此只需要保留预计算参数的低256位：
```c
/*
 * /* Pre-computed tables are "carry-less" values of modulus*(i+1),
 * all values are in little-endian format.
 */
static const BN_ULONG _sm2_p_256[][BN_SM2_256_TOP] = {
    {0xFFFFFFFFFFFFFFFFull, 0xFFFFFFFF00000000ull,
     0xFFFFFFFFFFFFFFFFull, 0xFFFFFFFEFFFFFFFFull},/*p*/
    {0xFFFFFFFFFFFFFFFEull, 0xFFFFFFFE00000001ull,
     0xFFFFFFFFFFFFFFFFull, 0xFFFFFFFDFFFFFFFFull},/*2p*/
    ...
    {0xFFFFFFFFFFFFFFF4ull, 0xFFFFFFF40000000Bull,
     0xFFFFFFFFFFFFFFFFull, 0xFFFFFFF3FFFFFFFFull},/*12p*/
    {0xFFFFFFFFFFFFFFF3ull, 0xFFFFFFF30000000Cull,
     0xFFFFFFFFFFFFFFFFull, 0xFFFFFFF2FFFFFFFFull}/*13p*/
};
```
同时，为了保证被约减数![](https://intranetproxy.alipay.com/skylark/lark/__latex/a770a282bbfa0ae1ec474b7ed311656d.svg#card=math&code=v&id=kaNNx)满足![](https://intranetproxy.alipay.com/skylark/lark/__latex/b81bb0345ae6ca9443932e29b7e10f8c.svg#card=math&code=0%20%3C%3D%20v%20%3C%20p%5E2&id=jWJSN)，在模约减前还需要对输入进行验证。基于同样的原因，我们还需要预计算![](https://intranetproxy.alipay.com/skylark/lark/__latex/d72a175fb2d23aaf0ef84e9a2be3884b.svg#card=math&code=p%5E2&id=JJpUu)的值：
```c
/* pre-compute the value of p^2 check if the input satisfies input < p^2. */
static const BN_ULONG _sm2_p_256_sqr[] = {
    0x0000000000000001ULL, 0x00000001FFFFFFFEULL,
    0xFFFFFFFE00000001ULL, 0x0000000200000000ULL,
    0xFFFFFFFDFFFFFFFEULL, 0xFFFFFFFE00000003ULL,
    0xFFFFFFFFFFFFFFFFULL, 0xFFFFFFFE00000000ULL
};
```
在64位平台和部分支持64位无符号整型的32位平台上，由于![](https://intranetproxy.alipay.com/skylark/lark/__latex/e187b0d138778a4f925ac178c4fbc6a3.svg#card=math&code=v%5Bi%5D&id=zig4n)满足![](https://intranetproxy.alipay.com/skylark/lark/__latex/5fed335646c644579271f301f3a8bb7a.svg#card=math&code=0%20%5Cleq%20v%5Bi%5D%3C%202%5E%7B32%7D&id=Lu5v9)，因此可以使用64位无符号整形变量`accumulator`作为中间值存储累加结果，从而在![](https://intranetproxy.alipay.com/skylark/lark/__latex/308b899646a87220bfe36fe6a54af302.svg#card=math&code=v%5B0%5D%20-%20v%5B7%5D&id=NNnYf)各参数上直接累加高位参数约化结果。以![](https://intranetproxy.alipay.com/skylark/lark/__latex/5fd2dda523635dcc9dddd464e03e167b.svg#card=math&code=v%5B0%5D&id=MyyiW)为例，其相应代码如下：
```c
    	SM2_INT64 acc;         /* accumulator */

        acc = rp[0];
        acc += bp[8 - 8];
        acc += bp[9 - 8];
        acc += bp[10 - 8];
        acc += bp[11 - 8];
        acc += bp[12 - 8];
        acc += bp[13 - 8];
        acc += bp[14 - 8];
        acc += bp[15 - 8];
        acc += bp[13 - 8];
        acc += bp[14 - 8];
        acc += bp[15 - 8];	/*累加高位参数约化结果*/
        rp[0] = (unsigned int)acc; /*获得结果的低32位*/
        acc >>= 32; /*进位部分右移32位，作为v[1]输入的一部分*/
```
	对于部分不支持64位无符号整型的32位平台，32位整形变量由于数据溢出无法作为中间值保留进位结果，上述代码将不能正确运行。此时需要借助现有工具函数和铜锁底层大数运算函数的支持，按照前文展示的算法逐一累加![](https://intranetproxy.alipay.com/skylark/lark/__latex/66205a4b6ae53dc8beb5db97e890e448.svg#card=math&code=s_1%20%EF%BD%9E%20s_%7B14%7D&id=LbFjS)，每次还需要记录进位![](https://intranetproxy.alipay.com/skylark/lark/__latex/b544ac60b7af9d60897b2d132b0c767b.svg#card=math&code=carry&id=E9QF4):
```c
        /*
         * r_d += s2 + s6 + s7 + s8 + s9
         * s2 = (c15, c14, c13, c12, c11, 0, c9, c8)
         */
        sm2_set_256(t_d, buf.bn, 15, 14, 13, 12, 11, 0, 9, 8);
        carry += (int)bn_add_words(r_d, r_d, t_d, BN_SM2_256_TOP);
        /*
         * s6 = (c11, c11, c10, c15, c14, 0, c13, c12)
         */
        sm2_set_256(t_d, buf.bn, 11, 11, 10, 15, 14, 0, 13, 12);
        carry += (int)bn_add_words(r_d, r_d, t_d, BN_SM2_256_TOP);
        /*
         * s7 = (c10, c15, c14, c13, c12, 0, c11, c10)
         */
        sm2_set_256(t_d, buf.bn, 10, 15, 14, 13, 12, 0, 11, 10);
        carry += (int)bn_add_words(r_d, r_d, t_d, BN_SM2_256_TOP);
        /*
         * s8 = (c9, 0, 0, c9, c8, 0, c10, c9)
         */
        sm2_set_256(t_d, buf.bn, 9, 0, 0, 9, 8, 0, 10, 9);
        carry += (int)bn_add_words(r_d, r_d, t_d, BN_SM2_256_TOP);
        /*
         * s9 = (c8, 0, 0, 0, c15, 0, c12, c11)
         */
        sm2_set_256(t_d, buf.bn, 8, 0, 0, 0, 15, 0, 12, 11);
        carry += (int)bn_add_words(r_d, r_d, t_d, BN_SM2_256_TOP);
```
	最后处理进位，得到最终结果，此处变量mask的作用是减少代码中不必要的分支，尽可能提高其抗侧信道攻击强度：
```c
    mask =
        0 - (PTR_SIZE_INT) (*u.f) (c_d, r_d, _sm2_p_256[0], BN_SM2_256_TOP);
    mask &= 0 - (PTR_SIZE_INT) carry;
    res = c_d;
    res = (BN_ULONG *)(((PTR_SIZE_INT) res & ~mask) |
                       ((PTR_SIZE_INT) r_d & mask));
    sm2_cp_bn(r_d, res, BN_SM2_256_TOP);
    r->top = BN_SM2_256_TOP;
    bn_correct_top(r);
```
<a name="k52MK"></a>
### 5 测试
<a name="DOpk0"></a>
#### 5.1 正确性测试
为了验证快速模约减算法的正确性，我们在测试环节设计了相应的单元测试。该测试的核心思路是，随机生成一批有限域大整数![](https://intranetproxy.alipay.com/skylark/lark/__latex/f4b035f65a610c49719892cdc54401ff.svg#card=math&code=a_i%2C%20b_i&id=XHand)，分别使用铜锁提供的默认方法和本文实现的快速模约减方法计算![](https://intranetproxy.alipay.com/skylark/lark/__latex/480943b95852e69f61d9c0b3d29b25e8.svg#card=math&code=a_i%20%2A%20b_i%20%5Cmod%20p&id=XoqBq)，然后比较二者计算结果是否相等：
```c
    /*
     * Randomly generate two numbers in the prime field and multiply them，then compare the results of fast modular reduction and conventional algorithms.
     */
    for (i = 0; i < 100; i++){
        if (!TEST_true(BN_priv_rand_range_ex(a, p, 0, ctx))
            || !TEST_true(BN_priv_rand_range_ex(b, p, 0, ctx))
            || !TEST_true(BN_mul(ab_fast, a, b, ctx))
            || !TEST_true(BN_sm2_mod_256(ab_fast, ab_fast, p, ctx))
            || !TEST_true(BN_mod_mul(ab, a, b, p, ctx))) {
            goto done;
        }

        if (!TEST_int_eq(BN_ucmp(ab, ab_fast), 0)){
            goto done;
        }
    }
```
	可以使用如下命令调用测试：
```bash
make test TESTS="test_mod_sm2"
```
	测试结果：
```bash
➜  Tongsuo git:(sm2p256) ✗ make test TESTS="test_mod_sm2"
/Library/Developer/CommandLineTools/usr/bin/make depend && /Library/Developer/CommandLineTools/usr/bin/make _tests
10-test_mod_sm2.t .. ok   
All tests successful.
Files=1, Tests=1,  1 wallclock secs ( 0.00 usr  0.01 sys +  0.07 cusr  0.02 csys =  0.10 CPU)
Result: PASS
```
<a name="Xnmyf"></a>
#### 5.2 性能测试
为了充分比较快速模约减算法的性能，我们选择比较SM2有限域运算中较常用的模乘法运算作为测试单元，测试使用了SM2快速模约减函数`BN_sm2_mod_256`、简单模约减函数`BN_nnmod`的模乘函数`ossl_ec_GFp_field_mul` 和通用蒙哥马利模乘函数`BN_mod_mul_montgomery`的模乘法运算耗时。测试在`Apple M2 pro`平台上进行，测试数据如下表所示：

|  | SM2快速模约减 | 简单模约减 | 蒙哥马利模乘约减 |
| --- | --- | --- | --- |
| 模乘法耗时（ns/iter） | **35.53** | 190.45 | 50.69 |

测试结果表明，使用SM2快速模约减函数的模乘法耗时最少，相较于简单模约减能够减少**81.34%**的时间开销，即使对使用了汇编优化的蒙哥马利模乘运算，在计算时间上仍有**29.91%**的优势。
