本文翻译 OpenSSL 官网文档：[https://www.openssl.org/docs/OpenSSL300Design.html](https://www.openssl.org/docs/OpenSSL300Design.html)<br />Tongsuo-8.4.0 是基于 OpenSSL-3.0.3 开发，所以本文对 Tongsuo 开发者同样适用，内容丰富，值得一读！
<a name="AP9Lt"></a>
## 介绍
本文概述了 OpenSSL 3.0 的设计，这是在 1.1.1 版本之后的 OpenSSL 的下一个版本。假设读者熟悉名为 "OpenSSL 战略架构" 的文档，并对 OpenSSL 1.1.x 有一定的实际操作经验。<br />OpenSSL 3.0 版本对大多数现有应用程序影响很小；几乎所有良好行为的应用程序只需重新编译即可。<br />OpenSSL 3.0 中的大多数变更是为了进行内部架构重组，以便实现一个长期可支持的密码学框架，从而更好地将算法实现与算法 API 分离。这些结构性变更还支持更易维护的 OpenSSL FIPS 密码模块 3.0。<br />在 OpenSSL 3.0 中，目前标记为弃用（deprecated）的 API 不会被移除。<br />在 OpenSSL 3.0 中，许多额外的低级函数将被标记为弃用（deprecated）的 API。<br />OpenSSL 3.0 将同时支持应用程序同时具有处于 FIPS 模式（使用 OpenSSL FIPS 密码模块 3.0）和非 FIPS 模式的 TLS 连接。<br />有关 3.0 版本的更多最新信息，请参考 [openssl.org/docs] ([https://www.openssl.org/docs)](https://www.openssl.org/docs)) 上的链接。
<a name="IIL91"></a>
## 术语
以下术语按字母顺序在本文中使用，简要定义如下：

- 算法（Algorithm），有时称为密码算法，是执行一组操作（如加密或解密）的方法。我们使用这个术语时是抽象的，通常通过名称（例如“aes-128-cbc”）来表示一个算法。
- 算法实现（Algorithm implementation），有时简称为实现（implementation），是算法的具体实现。这主要以代码形式表示为一组函数。
- CAVS 是密码算法验证系统。这是一种工具，用于测试密码实现是否符合 FIPS 标准。
- CMVP 是密码模块验证程序。这个过程验证密码实现是否符合 FIPS 标准。
- EVP 是由 libcrypto 实现的一系列 API，使应用程序能够执行密码操作。EVP API 的实现使用 Core 和 Provider 组件。
- Core 是 libcrypto 中的一个组件，使应用程序能够访问 Provider 提供的算法实现。
- CSP 是关键安全参数（Critical Security Parameters）。这包括在未经授权的披露或修改的情况下可能损害模块安全性的任何信息（例如私钥、密码、PIN 码等）。
- 显式获取（Explicit Fetch）是一种查找算法实现的方法，应用程序通过显式调用来定位实现并提供搜索条件。
- FIPS 是联邦信息处理标准（Federal Information Processing Standards）。这是由美国政府定义的一组标准。特别是 FIPS 140-2 标准适用于密码软件。
- FIPS 模块是经过 CMVP 验证符合 FIPS 标准的密码算法实现。在 OpenSSL 中，FIPS 模块以 Provider 的形式实现，并以可动态加载模块的形式提供。
- 隐式获取（Implicit Fetch）是一种查找算法实现的方法，应用程序不会显式调用来定位实现，因此会使用默认的搜索条件。
- 完整性检查（Integrity Check）是在加载 FIPS 模块时自动运行的测试。模块会对自身进行校验，并验证是否被恶意修改。
- KAS 是密钥协商方案（Key Agreement Scheme）。它是两个通信方协商共享密钥的方法。
- KAT 是已知答案测试（Known Answer Tests）。它是用于对 FIPS 模块进行健康检查的一组测试。
- libcrypto 是 OpenSSL 实现的一个共享库，为应用程序提供各种与密码学相关的功能。
- libssl 是 OpenSSL 实现的一个共享库，为应用程序提供创建 SSL/TLS 连接的能力，可以作为客户端或服务器。
- 库上下文（Library Context）是一个不透明结构，保存库的“全局”数据。
- 操作（Operation）是对数据执行的一类函数，如计算摘要、加密、解密等。一个算法可以提供一个或多个操作。例如，RSA 提供非对称加密、非对称解密、签名、验证等。
- 参数（Parameters）是一组与实现无关的键值对，用于在 Core 和 Provider 之间传递对象数据。例如，它们可以用于传输私钥数据。
- POST 指的是 FIPS 模块的上电自检（也称为开机自检），在安装时、上电（即每次为应用程序加载 FIPS 模块时）或按需运行。这些测试包括完整性检查和 KAT。如果 KAT 在安装时成功运行，则在上电时不需要再次运行，但始终执行完整性检查。
- 属性（Properties）用于 Provider 描述其算法实现的特性。它们还用于应用程序查询以查找特定实现。
- Provider 是提供一个或多个算法实现的单元。
- Provider 模块是以可动态加载模块形式的 Provider。
<a name="mfAOu"></a>
## 架构
架构应具备以下特性：

- 共享服务（Common Services）是应用程序和 Provider 可用的构建块，例如 BIO、X509、SECMEM、ASN.1 等。
- Provider 实现密码算法和支持服务。一个算法可能由多个操作组成（例如 RSA 可能有“加密”、“解密”、“签名”、“验证”等）。同样，一个操作（例如“签名”）可以由多个算法实现，比如 RSA 和 ECDSA。Provider 包含了算法的密码原语实现。此版本将包括以下 Provider：
   1. 默认 Provider（Default），包含当前非遗留（non-legacy）的 OpenSSL 密码算法；这将作为内置部分（即 libcrypto 的一部分）。
   2. 遗留 Provider（Legacy），包含旧算法的实现（例如 DES、MDC2、MD2、Blowfish、CAST）。
   3. FIPS Provider，实现 OpenSSL FIPS 密码模块 3.0；可以在运行时动态加载。
- Core 使应用程序（和其他 Provider）能够访问 Provider 提供的操作。Core 是定位操作的具体实现的机制。
- 协议实现，例如 TLS、DTLS。

本文档中有许多关于 "EVP API" 的引用。这指的是 "应用级别" 的操作，例如公钥签名、生成摘要等。这些函数包括 EVP_DigestSign、EVP_Digest、EVP_MAC_init 等。EVP API 还封装了执行这些服务所使用的密码对象，例如 EVP_PKEY、EVP_CIPHER、EVP_MD、EVP_MAC 等等。Provider 为后者集合实现了后端功能。这些对象的实例可以根据应用程序的需求隐式或显式地绑定到 Provider 上。这在下面的 Provider 设计部分将详细讨论。

架构具有以下特点：

- EVP 层是对 Provider 中实现的操作的薄封装，大多数调用会直接传递，几乎没有预处理或后处理过程。
- 将提供新的 EVP API，以影响 Core 如何选择（或查找）要在任何给定 EVP 调用中使用的操作的实现方式。
- 以与实现无关的方式在 libcrypto 和 Provider 之间传递信息。
- 将弃用遗留 API（例如，不通过 EVP 层的低级密码 API）。请注意，存在针对非遗留算法的遗留 API（例如，AES 不是遗留算法，但 AES_encrypt 是遗留 API）。
- OpenSSL FIPS 密码模块将作为动态加载的 Provider 实现，它将是自包含的（即只能依赖于系统运行时库和核心提供的服务）。

<a name="hXr6f"></a>
### 概念组件视图
OpenSSL 架构中的概念组件概述如下图所示。请注意，图中组件的存在并不意味着该组件是公共 API 或用于直接访问或使用的最终用户的组件。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1686722022395-3175d954-f3f8-4149-a858-b3199413b85a.png#averageHue=%23c49a9a&clientId=ueb21923b-dd6e-4&from=paste&id=u082f21c2&originHeight=527&originWidth=588&originalType=binary&ratio=2&rotation=0&showTitle=false&size=88992&status=done&style=none&taskId=u996fcdcd-c6d5-4435-ad99-7ad75e64913&title=)<br />新的组件（在先前架构中不存在）如下所示：

- Core：这是一个基本组件，将对操作（如加密）的请求连接到提供该操作的 Provider。它提供了定位实现指定操作的算法的能力，给定一组实现必须满足的属性。例如，加密算法的属性至少包括 "fips"。
- 默认 Provider（Default Provider）：实现了一组默认算法。
- FIPS Provider：实现了经过 FIPS 验证的一组算法，并通过核心提供访问。这包括以下支持服务：
   - POST：上电自检，包括：
      - KAT：已知答案测试
      - 完整性检查
   - 低级实现（Low Level Implementations）：实际实现密码原语的一组组件，以满足 FIPS 规定的自包含要求。
- 遗留 Provider（Legacy Provider）：提供了旧算法的实现，将通过 EVP 级别的 API 暴露出来。
- 第三方 Provider（3rd Party Providers）：最终，第三方可能会提供自己的 Provider。第三方 Provider 与任何其他 Provider 一样，实现了一组算法，可以通过 Core 访问，并对应用程序和其他 Provider 可见。
- 空 Provider（Null Provider）：一个什么都不做的 Provider，这对于测试正确使用库上下文非常有用。
- 基础 Provider（Base Provider）：用于密钥序列化的 Provider。FIPS Provider 需要它，因为它本身不包含加载密钥的方法。基础 Provider 也嵌入在默认 Provider 中。
<a name="mXv6y"></a>
### 打包视图
上述概念组件视图中描述的各个组件在物理上打包为：

- 可由用户使用的可执行应用程序
- 供应用程序使用的库
- 供 Core 使用的可动态加载模块

OpenSSL 3.0 将提供多种不同的打包选项（例如一个名为 libcrypto 的单一库，包含除 FIPS Provider 之外的所有内容，以及所有 Provider 作为单独的可动态加载模块）。<br />哪些可动态加载模块被注册、使用或可用可以在运行时进行配置，以下图示根据物理打包描述了架构：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1686723109327-e7049e58-b501-4e74-9d17-362232c69170.png#averageHue=%235e5050&clientId=ueb21923b-dd6e-4&from=paste&id=u317c0ed4&originHeight=573&originWidth=688&originalType=binary&ratio=2&rotation=0&showTitle=false&size=77692&status=done&style=none&taskId=uba414747-45a6-4665-8b6b-349672959e2&title=)<br />本版本引入的物理打包如下：

- FIPS 模块。该模块包含了实现了一组经过 FIPS 验证的算法，并通过 Core 可访问的 FIPS Provider。FIPS Provider 即 OpenSSL FIPS Cryptographic Module 3.0。

我们不会试图阻止用户出错，但我们会考虑典型用户的使用情况和“安全性”。默认情况下，将构建并安装 FIPS Provider。

我们将能够执行安全检查，以检测用户是否以对 FIPS 有影响的方式修改了源代码，并在未提供覆盖选项的情况下（尽力而为）阻止构建 FIPS Provider。

我们需要确保存在一种机制，使最终用户能够确定他们对 FIPS 模块的使用是否符合正式验证的允许用途。

FIPS 模块的版本将与基础 OpenSSL 版本号对齐，验证时的版本号取决于该版本。并非所有 OpenSSL 发布版本都需要更新 FIPS 模块。因此，当发布新的 FIPS 模块版本时，其版本号可能存在间隔或跳跃，与之前版本相比。

- Legacy 模块。该模块包含了旧版算法的实现。

最初的计划是使用 Provider 的封装来构建引擎 (Engines)，以便在将 ENGINE 指针传递给某些函数时，使其像往常一样工作，并在充当默认实现时作为 Provider。在开发过程中的调查显示，这种方法存在问题的边缘情况。目前的解决方法是，当进行 EVP 调用时，目前存在两个代码路径。对于引擎（engine）的支持，使用“遗留密钥”的旧代码。长期计划是从代码库中删除引擎（engine）和遗留代码路径。一旦删除引擎（engine），任何作为引擎（engine）编写的内容都需要重新编写为 Provider。
<a name="IhkOR"></a>
## Core 和 Provider 设计
下图显示了与 Core 和 Provider 设计相关的交互，有四个主要组件：用户应用程序、EVP 组件、Core 和密码Provider（可能有多个 Provider，但在此不相关）。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1686723710026-a7e719fe-2528-49ec-a24f-24d73b906bc3.png#averageHue=%2335305b&clientId=ueb21923b-dd6e-4&from=paste&id=u9cd453eb&originHeight=376&originWidth=724&originalType=binary&ratio=2&rotation=0&showTitle=false&size=53520&status=done&style=none&taskId=ua654a17e-2f6c-4e37-bee7-93848eda5e8&title=)<br />Core 具有以下特点：

- 实现了 Provider 的发现、加载、初始化和卸载功能；
- 支持基于属性的算法查询；
- 实现了算法查询和实现细节的缓存；
- 在库上下文中运行，其中包含全局属性、搜索缓存和分派表等数据。

Provider 具有以下特点：

- 提供对特定算法实现的访问；
- 将算法实现与一组明确定义的属性相关联；
- 以一种与具体实现无关的方式支持参数传递；
- 可以在任何时间点进行加载；
- 具有众所周知的模块入口点。

接下来的小节描述了应用程序使用的流程，以加载 Provider、获取算法实现并使用它为例。此外，本节详细描述了算法、属性和参数的命名方式，以及如何处理算法查询、注册和初始化算法，以及如何加载 Provider。<br />为了使应用程序能够使用算法，首先必须通过算法查询来“获取（fetch）”其实现。我们的设计目标是能够支持显式（事先）获取算法和在使用时获取算法的方式。默认情况下，我们希望在使用时进行获取（例如使用EVP_sha256()），这样算法通常会在“init”函数期间进行获取，并绑定到上下文对象（通常命名为ctx）。显式获取选项将通过新的API调用实现（例如 EVP_MD_fetch()）。<br />上述图示展示了显式获取算法的方法。具体步骤如下：

1. 需要加载每个 Provider，这将隐式发生（默认 Provider 或通过配置指定），也可以由应用程序显式请求加载。加载过程包括动态共享对象的加载（根据需要）和初始化。
   1. Core 组件将模块物理加载到内存中（如果默认 Provider 已经在内存中，则无需加载）。
   2. Core 组件调用 Provider 的入口点，以便 Provider 对自身进行初始化。
      1. 在入口点函数中，Provider 使用从 Core 组件传入的值初始化一些 Provider 变量。如果初始化成功，Provider 将返回一个用于 Provider 算法实现查询的回调函数给 Core 组件。
2. 用户应用程序通过调用获取例程请求算法。
   1. EVP 将全局属性与调用特定属性以及算法标识相结合，以找到相应的算法实现，然后创建并返回一个库句柄（例如EVP_MD，EVP_CIPHER）给应用程序。
      1. 在内部缓存中进行第一次实现调度表的搜索。
      2. 如果第一次搜索失败，则通过询问 Provider 是否具有符合查询属性的算法实现来进行第二次搜索，当完成此搜索时，除非 Provider 选择不进行缓存（用于第一次搜索2.1.1），否则结果数据将被缓存。例如，PKCS＃11 Provider 可能选择不进行缓存，因为其算法可能随时间可用和不可用。
3. 然后，用户应用程序通过 EVP API（例如 EVP_DigestInit()，EVP_DigestUpdate()，EVP_DigestFinal() 等）使用算法。
   1. 函数指针被调用，并最终进入 Provider 的实现，执行请求的密码算法。

对于现有的 EVP_{algorithm}() 函数（例如 EVP_sha256() 等），大部分情况下保持不变。特别是，当 EVP_{algorithm}() 调用返回时，并不会立即执行获取算法的操作，而是在将上下文对象（例如 EVP_MD_CTX）绑定到相应的 EVP 初始化函数内部时隐式地进行。具体来说，步骤 2.1 发生在步骤 3.1 之前，这被称为 "隐式获取"，隐式获取总是在默认的库上下文中进行操作（详见下文的[库上下文](#lquJj)）。<br />方法调度表是一个由 <函数 ID, 函数指针> 对组成的列表，其中函数 ID 是 OpenSSL 公开定义并已知的，同时还包括一组用于标识每个特定实现的属性。Core 可以根据属性查询找到相应的调度表，以供适用的操作使用。这种方法允许 Provider 灵活地传递函数引用，以便 OpenSSL 代码可以动态创建其方法结构。<br />Provider 可以在任何时间点加载，也可以在任何时间点请求卸载。在卸载 Provider 时，应用程序需要确保该 Provider 当前未被使用或引用，如果尝试使用不再可用的实现，则会返回错误信息。<br />关于 EVP_{algorithm}() 函数的返回值，目前应用程序可以做出的假设是：

- 常量指针
- 不需要由应用程序释放
- 可以安全地进行比较，用于检查算法是否相同（即特定比较 EVP_CIPHER、EVP_MD 等指针）

对于应用程序直接使用显式获取（而不是使用现有的 EVP_{algorithm}() 函数）的情况，语义将有所不同：

- 非常量指针
- 需要由应用程序释放
- 指针之间不能安全地进行比较（后文将详细说明）

将提供新的 API 来测试可以用于显式获取对象和静态变体对象的相等性，这些 API 将使得可以比较算法标识本身或具体的算法实现。
<a name="lquJj"></a>
### 库上下文
库上下文是一个不透明的结构，用于保存库的“全局”数据，OpenSSL 将提供这样的结构，仅限于 Core 必须保留的全局数据，未来的扩展可能会包括其他现有的全局数据，应用程序可以创建和销毁一个或多个库上下文，所有后续与 Core 的交互都将在其中进行，如果应用程序不创建并提供自己的库上下文，则将使用内部的默认上下文。
```c
OPENSSL_CTX *OPENSSL_CTX_new();
void OPENSSL_CTX_free(OPENSSL_CTX *ctx);
```

库上下文可以传递给显式获取函数。如果将 NULL 传递给它们，将使用内部默认上下文。<br />可以分配多个库上下文，这意味着任何 Provider 模块可能会被初始化多次，这使得应用程序既可以直接链接到 libcrypto 并加载所需的 Provider，又可以链接到使用其自己 Provider 模块的其他库，而二者是相互独立的。
<a name="zI9G6"></a>
### 命名
算法、参数和属性需要命名，为了确保一致性，并使外部 Provider 实现者能够以一致的方式定义新名称，将建立一个推荐或已使用名称的注册表。它将与源代码分开维护。<br />需要能够定义名称的别名，因为在某些情况下，对同一事物存在多个名称（例如，对于具有通用名称和NIST名称的椭圆曲线）的上下文。
<a name="v8Zux"></a>
### 算法实现选择属性
算法实现（包括加密和非加密）将具有一些属性，用于从可用的实现中选择一个实现。在3.0版本中，定义了两个属性：

- 该实现是否为默认实现？
- 该实现是否经过 FIPS 验证？

有效的输入及其含义如下：

| **属性字符串** | **定义中的含义** | **查询中的含义** |
| --- | --- | --- |
| default | 是默认实现 | 请求默认实现 |
| default=yes | 是默认实现 | 请求默认实现 |
| default=no | 不是默认实现 | 请求非默认实现 |
| fips | 此实现已通过 FIPS 验证 | 请求经过 FIPS 验证的实现 |
| fips=yes | 此实现已通过 FIPS 验证 | 请求经过 FIPS 验证的实现 |
| fips=no | 此实现未通过 FIPS 验证 | 请求未经过 FIPS 验证的实现 |

在所有情况下，属性名称将被定义为可打印的 ASCII 字符，并且不区分大小写，属性值可以带引号或不带引号，不带引号的值也必须是可打印的 ASCII 字符，并且不区分大小写，引号中的值仅以原始字节比较的方式进行相等性测试。<br />Provider 将能够提供自己的名称或值，属性定义和查询的完整语法见[附录1 - 属性语法](#NDwuz)。<br />OpenSSL 保留所有没有句点的属性名称；供应商提供的属性名称必须在名称中包含句点。预期（但不强制要求）属性名称中第一个句点之前的部分是供应商的名称或与之相关的内容，以通过命名空间提供一定程度的冲突避免。<br />在开发此版本的过程中，可能会定义其他属性，一个可能的候选是 provider，表示提供实现的 Provider 名称。另一个可能性是 engine，表示此算法由伪装为 Provider 的 OpenSSL 1.1.1 动态加载的引擎实现。<br />将有一个内置的全局属性查询字符串，其值为"default"。
<a name="cIR0z"></a>
#### 属性选择算法
算法实现的选择基于属性。<br />Provider 在其提供的算法上设置属性，应用程序在算法选择过程中设置要用作筛选条件的属性查询。<br />可以在以下位置指定获取算法实现所需的属性：

1. 全局配置文件中的全局设置；
2. 基于 API 调用的全局设置；
3. 针对特定对象的每个对象的属性设置。例如，SSL_CTX，SSL。

属性将在算法查找过程中使用（参数规范的属性值）。

属性集将以解析为每个指定属性（关键字）的属性的单个值的方式进行评估。关键字评估的优先顺序如下：

1. 获取的每个对象或直接指定的 API 参数
2. 通过 API 调用设置的全局（默认）属性
3. 在配置文件中设置的全局（默认）属性

在开发过程中，可能会定义其他属性设置方法和评估方法。<br />默认情况下，OpenSSL 3.0 将自动加载配置文件（其中包含全局属性和其他设置），而无需显式的应用程序 API 调用，这将在 libcrypto 中发生。请注意，在 OpenSSL 1.1.1 中，配置文件仅在默认（自动）初始化 libssl 时自动加载。
<a name="N4Ek5"></a>
### 参数定义
OpenSSL Core 和 Provider 在保持 OpenSSL 和 Provider 结构不透明的同时需要交换数据，所有复合值将作为项目数组传递，使用 [附录2 - 参数传递](#Q6UPj) 中定义的公共数据结构，参数将使用它们的名称（作为字符串）进行标识，每个参数包含自己的类型和大小信息。<br />Core 将定义一个 API，用于将参数值数组或值请求传递给 Provider 或特定的算法实现，对于后者，还有由该实现处理的相关对象，对于基本机器类型，可以开发宏来辅助构建和提取值。
<a name="Bee2E"></a>
### 操作和操作函数定义
虽然算法和参数名称基本上由 Provider 控制和分配，但由 libcrypto 调用的操作和相关函数基本上由 Core 控制和分配。<br />对于仅由 Core 控制的内容，我们将使用宏来命名它们，使用数字作为索引值，分配的索引值是递增的，即对于任何新的操作或函数，将选择下一个可用的数字。
<a name="Cxm8T"></a>
### 算法查询
每种算法类型（例如 EVP_MD、EVP_CIPHER 等）都有一个可用的“fetch”函数（例如 EVP_MD_fetch()、EVP_CIPHER_fetch()），算法实现是通过其名称和属性来识别的。<br />如 [Core 和 Provider 设计](#IhkOR) 中所述，每个 fetch 函数将使用 Core 提供的服务来找到适合的实现，如果找到适当的实现，它将被构造成适当的算法结构（例如 EVP_MD、EVP_CIPHER）并返回给调用应用程序。<br />如果多个实现与传递的名称和属性完全匹配，其中之一将在检索时返回，但具体返回哪个实现是不确定的，此外，并不能保证每次都返回相同的匹配实现。
<a name="Jg1AX"></a>
### 算法查询缓存
算法查询将与其结果一起被缓存。<br />下列这些算法查询缓存都可以清除：

- 返回特定算法实现的所有查询
- 来自特定 Provider 的所有算法实现
- 所有算法实现
<a name="LLxYH"></a>
### 多级查询
为了处理全局属性和传递给特定调用（例如获取调用）的属性，全局属性查询设置将与传递的属性设置合并，除非存在冲突，具体规则如下：

| **全局设置** | **传递的设置** | **最终查询结果** |
| --- | --- | --- |
| fips=yes | fips=yes | fips=yes |
| fips=yes | fips=no | fips=no |
| fips=yes | -fips | fips is not specified |
| fips=yes | fips is not specified | fips=yes |
| fips=no | fips=yes | fips=yes |
| fips=no | fips=no | fips=no |
| fips=no | -fips | fips is not specified |
| fips=no | fips is not specified | fips=no |
| fips is not specified | fips=yes | fips=yes |
| fips is not specified | fips=no | fips=no |
| fips is not specified | -fips | fips is not specified |
| fips is not specified | fips is not specified | fips is not specified |

<a name="dXR94"></a>
### Provider 模块加载
Provider 可以是内置的或可动态加载的模块。<br />所有算法都是由 Provider 实现的，OpenSSL Core 最初未加载任何 Provider，因此没有可用的算法，需要查找和加载 Provider，随后，Core 可以在稍后的时间查询其中包含的算法实现，这些查询可能会被缓存。<br />如果在第一次获取（隐式或显式）时尚未加载任何 Provider，则会自动加载内置的默认 Provider。<br />请注意，Provider 可能针对 libcrypto 当前版本之前的旧版本 Core API 进行编写，例如，用户可以运行与 OpenSSL 主版本不同的 FIPS Provider 模块版本，这意味着 Core API 必须保持稳定和向后兼容（就像任何其他公共 API 一样）。<br />OpenSSL 构建的所有命令行应用程序都将获得一个 -provider xxx 选项，用于加载 Provider，该选项可以在命令行上多次指定（可以始终加载多个 Provider），并且如果 Provider 在特定操作中未使用（例如，在进行 SHA256 摘要时加载仅提供 AES 的 Provider），并不会导致错误。
<a name="gwIAU"></a>
#### 查找和加载动态 Provider 模块
动态 Provider 模块在 UNIX 类型操作系统上是 .so 文件，在 Windows 类型操作系统上是 .dll 文件，或者在其他操作系统上对应的文件类型。默认情况下，它们将被安装在一个众所周知的目录中。<br />Provider 模块的加载可以通过以下几种方式进行：

- 按需加载，应用程序必须明确指定要加载的 Provider 模块。
- 通过配置文件加载，加载的 Provider 模块集合将在配置文件中指定。

其中一些方法可以进行组合使用。<br />Provider 模块可以通过完整路径指定，因此即使它不位于众所周知的目录中，也可以加载。<br />Core 加载 Provider 模块后，会调用 Provider 模块的入口点函数。
<a name="awb4A"></a>
#### Provider 模块入口点
一个 Provider 模块必须具有以下众所周知的入口点函数：
```c
int OSSL_provider_init(const OSSL_PROVIDER *provider,
                       const OSSL_DISPATCH *in,
                       const OSSL_DISPATCH **out
                       void **provider_ctx);
```
如果动态加载的对象中不存在该入口点，则它不是一个有效的模块，加载会失败。<br />`in`是核心传递给 Provider 的函数数组。<br />`out`是 Provider 传递回 Core 的 Provider 函数数组。<br />`provider_ctx`（在本文档的其他地方可能会缩写为provctx）是 Provider 可选创建的对象，用于自身使用（存储它需要安全保留的数据），这个指针将传递回适当的 Provider 函数。<br />`provider`是指向 Core 所属 Provider 对象的句柄，它可以作为唯一的 Provider 标识，在某些 API 调用中可能需要，该对象还将填充各种数据，如模块路径、Provider 的 NCONF 配置结构（了解如何实现可参见下面的[CONF / NCONF值作为参数](#xk6rY)的示例），Provider 可以使用 Core 提供的参数获取回调来检索这些各种值，类型 `OSSL_PROVIDER`是不透明的。<br />`OSSL_DISPATCH`是一个开放结构，实现了 [**Core and Provider 设计**](#IhkOR)** **介绍中提到的 <函数 ID, 函数指针> 元组。
```c
typedef struct ossl_dispatch_st {
    int function_id;
    void *(*function)();
} OSSL_DISPATCH;
```
`function_id`标识特定的函数，`function`是指向该函数的指针。这些函数的数组以`function_id`设置为 0 来终止。<br />Provider 模块可以链接或者不链接到 libcrypto，如果没有链接，则它将无法直接访问任何 libcrypto 函数，所有与 libcrypto 的基本通信将通过 Core 提供的回调函数进行。重要的是，由特定 Provider 分配的内存应由相同的 Provider 来释放，同样，libcrypto 中分配的内存应由 libcrypto 释放。<br />API 将指定一组众所周知的回调函数编号，在后续发布中，可以根据需要添加更多的函数编号，而不会破坏向后兼容性。
```c
/* Functions provided by the Core to the provider */
#define OSSL_FUNC_ERR_PUT_ERROR                        1
#define OSSL_FUNC_GET_PARAMS                           2
/* Functions provided by the provider to the Core */
#define OSSL_FUNC_PROVIDER_QUERY_OPERATION             3
#define OSSL_FUNC_PROVIDER_TEARDOWN                    4
```
Core 将设置一个众所周知的回调函数数组：
```c
static OSSL_DISPATCH core_callbacks[] = {
    { OSSL_FUNC_ERR_PUT_ERROR, ERR_put_error },
    /* int ossl_get_params(OSSL_PROVIDER *prov, OSSL_PARAM params[]); */
    { OSSL_FUNC_GET_PARAMS, ossl_get_params, }
    /* ... and more */
};
```
这只是核心可能决定传递给 Provider 的一些函数之一。根据需要，我们还可以传递用于日志记录、测试、仪表等方面的函数。<br />一旦模块加载完成并找到了众所周知的入口点，Core 就可以调用初始化入口点：
```c
/*
 * NOTE: this code is meant as a simple demonstration of what could happen
 * in the core.  This is an area where the OSSL_PROVIDER type is not opaque.
 */
OSSL_PROVIDER *provider = OSSL_PROVIDER_new();
const OSSL_DISPATCH *provider_callbacks;
/*
 * The following are diverse parameters that the provider can get the values
 * of with ossl_get_params.
 */
/* reference to the loaded module, or NULL if built in */
provider->module = dso;
/* reference to the path of the loaded module */
provider->module_path = dso_path;
/* reference to the NCONF structure used for this provider */
provider->conf_module = conf_module;

if (!OSSL_provider_init(provider, core_callbacks, &provider_callbacks))
    goto err;

/* populate |provider| with functions passed by the provider */
while (provider_callbacks->func_num > 0) {
    switch (provider_callbacks->func_num) {
    case OSSL_FUNC_PROVIDER_QUERY_OPERATION:
        provider->query_operation = provider_callbacks->func;
        break;
    case OSSL_FUNC_PROVIDER_TEARDOWN:
        provider->teardown = provider_callbacks->func;
        break;
    }
    provider_callbacks++;
}
```
`OSSL_provider_init`入口点不会注册任何需要的算法，但它将返回至少这两个回调函数以启用这个过程：

- `OSSL_FUNC_QUERY_OPERATION`，用于查找可用的操作实现。它必须返回一个`OSSL_ALGORITHM`数组（见下文），将算法名称和属性定义字符串映射到实现调度表，该函数还必须能够指示结果数组是否可以被 Core 缓存，下面将详细解释这一点。
- `OSSL_FUNC_TEARDOWN`，在 Provider 被卸载时使用。

Provider 注册回调只能在 `OSSL_provider_init()`调用成功后执行。
<a name="KSjTa"></a>
#### Provider 初始化和算法注册
一个算法提供一组操作（功能、特性等），这些操作通过函数调用，例如，RSA 算法提供签名和加密（两个操作），通过`init`、`update`、`final`函数进行签名，以及`init`、`update`、`final`函数进行加密，函数集由上层 EVP 代码的实现确定。<br />操作通过唯一的编号进行标识，例如：
```c
#define OSSL_OP_DIGEST                     1
#define OSSL_OP_SYM_ENCRYPT                2
#define OSSL_OP_SEAL                       3
#define OSSL_OP_DIGEST_SIGN                4
#define OSSL_OP_SIGN                       5
#define OSSL_OP_ASYM_KEYGEN                6
#define OSSL_OP_ASYM_PARAMGEN              7
#define OSSL_OP_ASYM_ENCRYPT               8
#define OSSL_OP_ASYM_SIGN                  9
#define OSSL_OP_ASYM_DERIVE               10
```
要使 Provider 中的算法可供 libcrypto 使用，它必须注册一个操作查询回调函数，该函数根据操作标识返回一个实现描述符数组：<br /><算法名称，属性定义字符串，实现的`OSSL_DISPATCH*`><br />因此，例如，如果给定的操作是`OSSL_OP_DIGEST`，此查询回调将返回其所有摘要的列表。<br />算法通过字符串进行标识。<br />Core 库以函数表的形式提供了一组服务供 Provider 使用。<br />Provider 还将通过提供的回调函数提供返回信息的服务（以[附录2 - 参数传递](#Q6UPj)中指定的参数形式），例如：

- 版本号
- 构建字符串 - 根据当前OpenSSL相关的构建信息（仅在 Provider 级别）
- Provider 名称

为了实现一个操作，可能需要定义多个函数回调，每个函数将通过数字函数标识进行标识，对于操作和函数的组合，每个标识都是唯一的，即为摘要操作的 init 函数分配的编号不能用于其他操作的 init 函数，它们将有自己的唯一编号。例如，对于摘要操作，需要以下这些函数：
```c
#define OSSL_OP_DIGEST_NEWCTX_FUNC         1
#define OSSL_OP_DIGEST_INIT_FUNC           2
#define OSSL_OP_DIGEST_UPDATE_FUNC         3
#define OSSL_OP_DIGEST_FINAL_FUNC          4
#define OSSL_OP_DIGEST_FREECTX_FUNC        5
typedef void *(*OSSL_OP_digest_newctx_fn)(void *provctx);
typedef int (*OSSL_OP_digest_init_fn)(void *ctx);
typedef int (*OSSL_OP_digest_update_fn)(void *ctx, void *data, size_t len);
typedef int (*OSSL_OP_digest_final_fn)(void *ctx, void *md, size_t mdsize,
                                       size_t *outlen);
typedef void (*OSSL_OP_digest_freectx_fn)(void *ctx);
```
对于无法处理多部分操作的设备，还建议使用多合一版本：
```c
#define OSSL_OP_DIGEST_FUNC                6
typedef int (*OSSL_OP_digest)(void *provctx,
                              const void *data, size_t len,
                              unsigned char *md, size_t mdsize,
                              size_t *outlen);
```
然后，Provider 定义包含每个算法实现的函数集的数组，并为每个操作定义一个算法描述符数组，算法描述符在前面提到过，并且可以公开定义如下：
```c
typedef struct ossl_algorithm_st {
    const char *name;
    const char *properties;
    OSSL_DISPATCH *impl;
} OSSL_ALGORITHM;

```
例如（这只是一个示例，Provider 可以按照自己的方式组织这些内容，重要的是算法查询函数（如下面的`fips_query_operation`）返回的内容），FIPS 模块可以定义如下数组来表示 SHA1 算法：
```c
static OSSL_DISPATCH fips_sha1_callbacks[] = {
    { OSSL_OP_DIGEST_NEWCTX_FUNC, fips_sha1_newctx },
    { OSSL_OP_DIGEST_INIT_FUNC, fips_sha1_init },
    { OSSL_OP_DIGEST_UPDATE_FUNC, fips_sha1_update },
    { OSSL_OP_DIGEST_FINAL_FUNC, fips_sha1_final },
    { OSSL_OP_DIGEST_FUNC, fips_sha1_digest },
    { OSSL_OP_DIGEST_FREECTX_FUNC, fips_sha1_freectx },
    { 0, NULL }
};
static const char prop_fips[] = "fips";
static const OSSL_ALGORITHM fips_digests[] = {
    { "sha1", prop_fips, fips_sha1_callbacks },
    { "SHA-1", prop_fips, fips_sha1_callbacks }, /* alias for "sha1" */
    { NULL, NULL, NULL }
};
```
FIPS Provider 初始化模块入口点函数可能如下所示：
```c
static int fips_query_operation(void *provctx, int op_id,
                                const OSSL_ALGORITHM **map)
{
    *map = NULL;
    switch (op_id) {
    case OSSL_OP_DIGEST:
        *map = fips_digests;
        break;
    }
    return *map != NULL;
}

#define param_set_string(o,s) do {                                  \
    (o)->buffer = (s);                                              \
    (o)->data_type = OSSL_PARAM_UTF8_STRING_PTR;                    \
    if ((o)->result_size != NULL) *(o)->result_size = sizeof(s);    \
} while(0)
static int fips_get_params(void *provctx, OSSL_PARAM *outparams)
{
    while (outparams->key != NULL) {
        if (strcmp(outparams->key, "provider.name") == 0) {
            param_set_string(outparams, "OPENSSL_FIPS");
        } else if if (strcmp(outparams->key, "provider.build") == 0) {
            param_set_string(outparams, OSSL_FIPS_PROV_BUILD_STRING);
        }
    }
    return 1;
}

OSSL_DISPATCH provider_dispatch[] = {
    { OSSL_FUNC_PROVIDER_QUERY_OPERATION, fips_query_operation },
    { OSSL_FUNC_PROVIDER_GET_PARAMS, fips_get_params },
    { OSSL_FUNC_PROVIDER_STATUS, fips_get_status },
    { OSSL_FUNC_PROVIDER_TEARDOWN, fips_teardown },
    { 0, NULL }
};
static core_put_error_fn *core_put_error = NULL;
static core_get_params_fn *core_get_params = NULL;

int OSSL_provider_init(const OSSL_PROVIDER *provider,
                       const OSSL_DISPATCH *in,
                       const OSSL_DISPATCH **out
                       void **provider_ctx)
{
    int ret = 0;

    /*
     * Start with collecting the functions provided by the core
     * (we could write it more elegantly, but ...)
     */
    while (in->func_num > 0) {
        switch (in->func_num) {
        case OSSL_FUNC_ERR_PUT_ERROR:
            core_put_error = in->func;
            break;
        case OSSL_FUNC_GET_PARAMS:
            core_get_params = in->func;
            Break;
        }
        in++;
    }

    /* Get all parameters required for self tests */
    {
        /*
         * All these parameters come from a configuration saying this:
         *
         * [provider]
         * selftest_i = 4
         * selftest_path = "foo"
         * selftest_bool = true
         * selftest_name = "bar"
         */
        OSSL_PARAM selftest_params[] = {
            { "provider.selftest_i", OSSL_PARAM_NUMBER,
              &selftest_i, sizeof(selftest_i), NULL },
            { "provider.selftest_path", OSSL_PARAM_STRING,
              &selftest_path, sizeof(selftest_path), &selftest_path_ln },
            { "provider.selftest_bool", OSSL_PARAM_BOOLEAN,
              &selftest_bool, sizeof(selftest_bool), NULL },
            { "provider.selftest_name", OSSL_PARAM_STRING,
              &selftest_name, sizeof(selftest_name), &selftest_name_ln },
            { NULL, 0, NULL, 0, NULL }
        }
        core_get_params(provider, selftest_params);
    }

    /* Perform the FIPS self test - only return params if it succeeds. */
    if (OSSL_FIPS_self_test()) {
        *out = provider_dispatch;
        return 1;
    }
    return 0;
}
```
<a name="tRc3W"></a>
### 算法选择
同时可能存在多个 Provider，重新编译为此版本的现有应用程序代码应该可以继续工作。与此同时，通过进行轻微的代码调整，应该能够使用基于属性的新算法查找功能来查找和使用算法。<br />为了说明这个过程是如何工作的，下面的代码是使用 OpenSSL 1.1.1 进行简单的 AES-CBC-128 加密的示例。为简洁起见，所有的错误处理都已被剥离。
```c
EVP_CIPHER_CTX *ctx;
EVP_CIPHER *ciph;

ctx = EVP_CIPHER_CTX_new();
ciph = EVP_aes_128_cbc();
EVP_EncryptInit_ex(ctx, ciph, NULL, key, iv);
EVP_EncryptUpdate(ctx, ciphertext, &clen, plaintext, plen);
EVP_EncryptFinal_ex(ctx, ciphertext + clen, &clentmp);
clen += clentmp;

EVP_CIPHER_CTX_free(ctx);
```
在 OpenSSL 3.0 中，这样的代码仍然可以正常工作，并且将使用来自 Provider 的算法（假设没有进行其他配置，将使用默认 Provider），它也可以通过显式获取进行重写，如下所示。显式获取还可以使应用程序在需要时指定非默认的库上下文（在此示例中为osslctx）：
```c
EVP_CIPHER_CTX *ctx;
EVP_CIPHER *ciph;

ctx = EVP_CIPHER_CTX_new();
ciph = EVP_CIPHER_fetch(osslctx, "aes-128-cbc", NULL);                /* <=== */
EVP_EncryptInit_ex(ctx, ciph, NULL, key, iv);
EVP_EncryptUpdate(ctx, ciphertext, &clen, plaintext, plen);
EVP_EncryptFinal_ex(ctx, ciphertext + clen, &clentmp);
clen += clentmp;

EVP_CIPHER_CTX_free(ctx);
EVP_CIPHER_free(ciph);                                                /* <=== */
```
应用程序可能希望使用来自不同 Provider 的算法。<br />例如，考虑这样的情况：应用程序希望使用 FIPS Provider 的某些算法，但在某些情况下仍然使用默认算法。可以以不同的方式实现，例如：

1. 只使用 FIPS 算法；
2. 默认使用 FIPS 算法，但能够在需要时进行覆盖，以获得对非 FIPS 算法的访问；
3. 默认不关心 FIPS 算法，但能够在需要时进行覆盖，以获得 FIPS 算法。
<a name="CckIP"></a>
#### 只使用 FIPS 算法
与 OpenSSL 3.0.0 之前版本编写的代码相比，如果您只需要 FIPS 实现，您只需要像这样进行一些更改：
```c
int main(void)
{
    EVP_set_default_alg_properties(NULL, "fips=yes");                 /* <=== */
    ...
}
```
然后，使用`EVP_aes_128_cbc()`的上述加密代码将继续像以前一样工作，`EVP_EncryptInit_ex()`调用将使用默认的算法属性，并通过 Core 查找以获取与 FIPS 实现关联的句柄，然后，该实现将与 `EVP_CIPHER_CTX`对象关联起来，如果没有适用的算法实现可用，`EVP_Encrypt_init_ex()`调用将失败。<br />`EVP_set_default_alg_properties`的第一个参数是库上下文，NULL 表示默认的内部上下文。
<a name="bYRIH"></a>
#### 默认使用 FIPS 算法，但允许覆盖
要将默认设置为使用 FIPS 算法，但根据需要覆盖为非 FIPS 算法，与 pre-3.0.0 OpenSSL 的代码相比，应用程序可能会进行以下更改：
```c
int main(void)
{
    EVP_set_default_alg_properties(osslctx, "fips=yes");              /* <=== */
    ...
}

EVP_CIPHER_CTX *ctx;
EVP_CIPHER *ciph;

ctx = EVP_CIPHER_CTX_new();
ciph = EVP_CIPHER_fetch(osslctx, "aes-128-cbc", "fips!=yes");         /* <=== */
EVP_EncryptInit_ex(ctx, ciph, NULL, key, iv);
EVP_EncryptUpdate(ctx, ciphertext, &clen, plaintext, plen);
EVP_EncryptFinal_ex(ctx, ciphertext + clen, &clentmp);
clen += clentmp;

EVP_CIPHER_CTX_free(ctx);
EVP_CIPHER_free(ciph);                                                /* <=== */

```
这里的`EVP_CIPHER_fetch()`调用会结合以下属性：

1. 默认的算法属性
2. 作为参数传入的属性（传入的属性优先级更高）。

因为`EVP_CIPHER_fetch()`调用覆盖了默认的“fips”属性，它将寻找一个不是“fips”的`AES-CBC-128`的实现。<br />在这个例子中，我们看到使用了非默认的库上下文，这只有在明确获取实现的情况下才可能发生。<br />（注意：对于细心的读者，“fips!=yes”也可以写为“fips=no”，但这里提供的是“不等于”运算符的一个示例）
<a name="bBHem"></a>
#### 默认不关注 FIPS 算法，并允许覆盖 FIPS
为了默认不使用 FIPS 算法，但可以根据需要覆盖为使用 FIPS 算法，应用程序代码可能如下所示（与3.0.0之前版本的OpenSSL代码相比）：
```c
EVP_CIPHER_CTX *ctx;
EVP_CIPHER *ciph;

ctx = EVP_CIPHER_CTX_new();
ciph = EVP_CIPHER_fetch(osslctx, "aes-128-cbc", "fips=yes");          /* <=== */
EVP_EncryptInit_ex(ctx, ciph, NULL, key, iv);
EVP_EncryptUpdate(ctx, ciphertext, &clen, plaintext, plen);
EVP_EncryptFinal_ex(ctx, ciphertext + clen, &clentmp);
clen += clentmp;

EVP_CIPHER_CTX_free(ctx);
EVP_CIPHER_free(ciph);                                                /* <=== */
```
在这个版本中，我们没有在“main”中覆盖默认的算法属性，因此你将获得默认的开箱即用设置，即不要求使用 FIPS 算法。然而，我们在`EVP_CIPHER_fetch()`级别上明确设置了“fips”属性，因此它覆盖了默认设置。当`EVP_CIPHER_fetch()`使用 Core 查找算法时，它将获得对 FIPS 算法的引用（如果没有这样的算法，则失败）。
<a name="nQFvW"></a>
#### 非对称算法选择
请注意，对于对称加密/解密和消息摘要，存在现有的 OpenSSL 对象可用于表示算法，即`EVP_CIPHER`和`EVP_MD`。对于非对称算法，没有等效的对象，使用的算法从`EVP_PKEY`的类型隐式推断出来。<br />为了解决这个问题，将引入一个新的非对称算法对象。在下面的示例中，执行了一个 ECDH 密钥派生操作，我们使用一个新的算法对象`EVP_ASYM`来查找 FIPS 的 ECDH 实现（当然，假设我们知道给定的私钥是 ECC 私钥）：
```c
EVP_PKEY_CTX *pctx = EVP_PKEY_CTX_new(privkey, NULL);
EVP_ASYM *asym = EVP_ASYM_fetch(osslctx, EVP_PKEY_EC, "fips=yes");
EVP_PKEY_CTX_set_alg(pctx, asym));
EVP_PKEY_derive_init(pctx);
EVP_PKEY_derive_set_peer(pctx, pubkey);
EVP_PKEY_derive(pctx, out, &outlen);
EVP_PKEY_CTX_free(pctx);
```
<a name="x4rrZ"></a>
### 算法选择动态视图示例
下面的时序图展示了如何从默认 Provider 中选择和调用 SHA256 算法的示例。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1686820699717-7cfe5384-9c7d-41f2-bd82-87cc6dfdb19d.png#averageHue=%23fafafa&clientId=ub20302ed-668f-4&from=paste&id=u18fd0e15&originHeight=695&originWidth=1333&originalType=binary&ratio=2&rotation=0&showTitle=false&size=70521&status=done&style=none&taskId=u07444bde-0af3-4f14-b8e7-b74a80557d2&title=)<br />请注意，EVP 层的每个调用都由 EVP 层中的薄封装器实现，这些封装器按照算法的方式在 Provider 中调用同名函数，要使用的特定 Provider 函数将通过显式的`EVP_MD_fetch()`调用在 Core 调度表中查找，该调用指定了消息摘要名称作为字符串以及其他相关属性，返回的 "md" 对象包含对所选 Provider 中算法实现的函数指针。<br />`EVP_MD_CTX`对象没有传递给 Provider，因为我们不知道任何特定的 Provider 模块是否与 libcrypto 链接，相反，我们只是传递一个黑盒句柄（void * 指针），Provider 将与其所需的任何结构相关联。在操作开始时，通过对 Provider 进行明确的`digestNewCtx()`调用来分配此句柄，并在结束时通过`digestFreeCtx()`调用来释放。<br />下一个图示展示了稍微复杂一些的情景，即使用 RSA 和 SHA256 的`EVP_DigestSign*`操作。该图示从 libcrypto 的角度绘制，其中算法由 FIPS 模块提供，稍后的章节将从 FIPS 模块的角度考察这个情景。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1686820946886-282bb581-7d0c-4ed4-898b-65ba5a5096be.png#averageHue=%23fbfbfb&clientId=ub20302ed-668f-4&from=paste&id=u78a64063&originHeight=1197&originWidth=2189&originalType=binary&ratio=2&rotation=0&showTitle=false&size=149119&status=done&style=none&taskId=ud682b0de-4fcf-4a08-82f0-344aa576b32&title=)<br />`EVP_DigestSign*`操作更加复杂，因为它涉及两个算法：签名算法和摘要算法。通常情况下，这两个算法可能来自不同的 Provider，也可能来自同一个 Provider。在使用 FIPS 模块的情况下，这两个算法必须来自同一个 FIPS 模块 Provider，如果尝试违反这个规则，操作将失败。<br />尽管有两个算法的额外复杂性，但与之前图示中展示的简单的`EVP_Digest*`操作相同的概念仍然适用。生成了两个上下文：`EVP_MD_CTX`和`EVP_PKEY_CTX`。这两个上下文都不会传递给 Provider。相反，通过显式的 "newCtx" Provider 调用创建黑盒（void *）句柄，然后在后续的 "init"、"update" 和 "final" 操作中传递这些句柄。<br />算法是通过提前使用显式的`EVP_MD_fetch()`和`EVP_ASYM_fetch()`调用在 Core 调度表中查找的。
<a name="NXJtN"></a>
## FIPS 模块
这是一个经过 [FIPS 140-2](https://csrc.nist.gov/publications/detail/fips/140/2/final) 验证的加密模块，它是一个只包含经过 FIPS 验证/批准的加密算法的 Provider，非 FIPS 算法将由默认 Provider（而不是 FIPS 模块）提供。<br />该模块是可以动态加载的，不支持静态链接。<br />FIPS 模块本身不会有 "FIPS 模式"，可以使用 FIPS Provider 的 OpenSSL 将具有与 FIPS-Module-2.0.0 兼容的"模式"概念。
<a name="NWJrH"></a>
### FIPS 模块版本编号
版本将为 FIPS-Module-3.0。<br />任何后续的修订版本将以与先前发布类似的方式进行标记，例如 3.0.x。<br />对于更改通知或重新验证，FIPS 模块的版本号将更新以匹配当前的 OpenSSL 库版本。
<a name="OavX2"></a>
### 检测 FIPS 边界内的变更
为了进行验证，我们需要检测是否有任何相关的源代码发生了变化。<br />可以使用一个脚本来对 C 源代码进行标记化处理，就像 C 预处理器一样，但还要教会它忽略源代码中的某些部分：

- 系统的`#include`指令。
- 在 FIPS 模式下被条件排除的代码（如下文中所述的[条件代码](#chOsw)）。

（提醒：C 预处理器可以将所有非换行空白字符合并，并在每个标记之间留下标准的单个空格，对于此目的，注释被视为空白字符）<br />标记化处理的结果可以通过校验和进行处理，该校验和存储在与源代码文件相对应的文件中，并最终进行版本控制。<br />该过程大致如下（并非完全相同，这只是一个代码示例，用于展示整个过程）：
```c
    for f in $(FIPS_SOURCES); do
        perl ./util/fips-tokenize $f | openssl sha256 -r
    done | openssl sha256 -hex -out fips.checksum
```
还会有一些机制来提醒我们有关变化的信息，以便我们可以采取适当的措施。例如：
```c
    git diff --quiet fips.checksum || \
        (git rev-parse HEAD > fips.commit; scream)
```
关于 scream 的具体操作尚待确定。<br />更新 fips.checksum 应该作为正常的 make update 的一部分进行，这是通常用于更改和检查版本控制文件的方法，OpenSSL 的持续集成已经运行了此命令，以确保没有遗漏任何内容，并且如果有内容被更改，将中断构建过程，运行`make update`也是正常的 OpenSSL 发布流程的一部分。
<a name="tr7Ws"></a>
#### 如何对已签名校验和的更改做出反应
尽管发生了变更，但我们仓库中的校验和更改本身并没有什么大不了的，它只是提醒我们需要额外关注 FIPS 源代码。<br />有两种可能的情况：

1. 当即将发布新版本，并且 fips.checksum 不再包含上一个经过验证的源代码的校验和时，将 FIPS 源代码发送给实验室，并开始更新验证流程。
2. 在发布新版本的同时，fips.checksum 不再包含上一个经过验证的源代码的校验和时，将 FIPS 源代码（包括 diff 文件和变更列表）发送给实验室，并启动相应的更新验证流程。

已验证的校验和列表将在其他地方列出（稍后确定具体位置）。
<a name="SUZeS"></a>
#### 编译
对于每个 FIPS Provider 的源文件，我们计算该文件的校验和，并将其与 fips.checksum 中收集的校验和进行比对，如果存在不匹配，将拒绝编译。
<a name="tWICZ"></a>
### FIPS 模式
FIPS 模块仅包含经过 FIPS 验证的密码算法，任何 FIPS 模式的“切换逻辑”将位于 FIPS 模块边界之外 - 这将由“fips”属性处理。<br />与 FIPS 模式相关的[条件代码](#chOsw)在单独的部分中讨论。<br />以下的 FIPS API 将继续可供应用程序使用（为了保持一致性，使用了与 1.1.1 版本中相同的名称）：

- `int FIPS_mode_set(int on)`

确保当前全局属性设置中设置了“fips=yes”（当 on != 0 时），或者未设置“fips”（当 on == 0 时）。这还将尝试使用属性“fips=yes”获取 HMAC-SHA256 算法，并确保它成功返回。

- `int FIPS_mode(void)`

如果当前的全局属性字符串包含属性“fips=yes”（或“fips”），则返回1，否则返回0。<br />我们可以检查当前是否有提供 FIPS 算法的 Provider 可用，并稍微以不同的方式处理。

- `int FIPS_self_test(void)`

如果 FIPS_mode() 返回true，则运行 KATs。<br />完整性测试将不在此处涵盖，如果我们决定提供它，它将是一个单独的函数。<br />成功时返回1，失败或没有 OpenSSL FIPS Provider 时返回0。<br />注意：这些函数只能在 OpenSSL FIPS Provider 的上下文中运行，而不能在其他任何 FIPS Provider的上下文中运行。这些是过时的遗留接口，应使用`EVP_set_default_alg_properties()`函数进行非遗留配置。
<a name="y9yMY"></a>
### 角色和身份认证
有两个隐含的角色 - 密码官（CO）和用户。这两个角色都支持相同的服务，唯一的区别是 CO 负责安装软件，该模块不应支持用户身份验证（对于1级而言不是必需的），所有这些都可以在安全策略中解释，而无需编写具体的代码。
<a name="ingQX"></a>
### 有限状态模型（FIPS 140-2第4.4节）
需要定义一个状态机。<br />我们将需要以下状态：

- 自检状态 - 初始化、运行、自检、错误、关闭（可能还包括后触发状态）
- 错误状态 - 如果自检失败，模块应返回该操作的错误。可以尝试清除错误并重复操作，如果失败仍然存在，模块应进入错误状态，这可以是一个硬错误状态，其中所有加密操作都失败，或者是一个功能降级状态，其中失败的组件仅在使用时返回错误。

触发自检失败的方式包括：

   1. 连续测试（生成密钥对的成对测试（签名/验证）和从熵源比较测试中验证输入到 DRBG 的随机数不相同）。
   2. DRBG 健康测试 - 这可以始终在 RNG 中引发错误（而不是设置全局错误状态）。
   3. 安装、启动或按需的 POST 完整性测试失败。
   4. 启动或按需的 POST KAT 失败。

将提供一个内部 API 来设置上述情况的失败状态。
<a name="dY9Z3"></a>
#### 状态机
在状态机中未显示的状态以虚线表示。进入和离开错误状态的边缘以虚线表示，以表明不希望遍历这些边缘。

| 状态模型由这些状态组成：<br />1. 电源关闭（Power Off）：FIPS 模块未加载到应用程序中，共享库不在内存中。<br />2. 电源开启（Power On）：FIPS 模块已被应用程序加载，并且共享库在内存中。默认入口点构造函数将被启动。<br />3. 初始化（Initialisation）：调用OSSL_provider_init。<br />4. 完整性检查（Integrity Check）（POST 完整性）：模块对自身进行校验，并验证其是否被恶意更改。（在 FIPS 提供程序的 OSSL_provider_init() 期间运行）。<br />5. 自检（Self Test）（POST KAT）：FIPS 模块在安装期间执行其自检，或者在通过 API 调用进行按需自检。<br />6. 运行（Running）：FIPS 模块处于正常运行状态，可以使用所有 API，并进行连续测试。<br />7. 错误（Error）：FIPS 模块进入错误状态，调用所有加密 API 将返回错误。<br />8. 关闭（Shutdown）：FIPS 模块正在被终止并从使用应用程序中卸载。<br /> | ![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687098629430-d1db5184-75b0-4088-8038-8f2e000cab4d.png#averageHue=%23f7f7f7&clientId=ud9c33b12-ba37-4&from=paste&id=ua31b0b8b&originHeight=871&originWidth=411&originalType=binary&ratio=2&rotation=0&showTitle=false&size=64011&status=done&style=none&taskId=u79546802-3307-4a9e-a937-ce51c7fcd57&title=) |
| --- | --- |


状态之间的边缘关系如下：

1. 从电源关闭（Power Off）到电源开启（Power On）：此转换由操作系统在将共享库加载到应用程序时执行。
2. 从电源开启（Power On）到初始化（Initialisation）：当调用共享库的构造函数时发生此转换。
3. 从电源开启（Power On）到关闭（Shutdown）：如果无法调用构造函数或构造函数失败，将触发此转换。
4. 从初始化（Initialisation）到完整性检查（Integrity Check）：此转换在初始化代码完成后发生，计算模块完整性校验和，并与预期值进行比较。
5. 从初始化（Initialisation）到错误（Error）：如果初始化代码在启动自测之前遇到错误，将触发此转换。
6. 从完整性检查（Integrity Check）到运行（Running）：对于所有启动过程中完整性检查成功的情况，发生此转换。
7. 从完整性检查（Integrity Check）到自测（Self Test）：在安装过程中，如果完整性检查成功，发生此转换。
8. 从完整性检查（Integrity Check）到错误（Error）：如果完整性检查失败，将触发此转换。
9. 从运行（Running）到关闭（Shutdown）：当 FIPS 模块最终化时发生此转换。
10. 从运行（Running）到错误（Error）：如果连续测试中的任何一个失败，将触发此转换。
11. 从运行（Running）到自测（Self Test）：应用程序手动启动自测时触发此转换。不重新运行完整性检查。
12. 从自测（Self Test）到运行（Running）：自测通过时发生此转换。
13. 从自测（Self Test）到错误（Error）：如果自测失败，将触发此转换。
14. 从关闭（Shutdown）到电源关闭（Power Off）：当 FIPS 模块从应用程序的内存中卸载时发生此转换。
15. 从错误（Error）到关闭（Shutdown）：当 FIPS 模块最终化时发生此转换。

如果可能的话，我们应该尽量在运行状态下注册算法，任何进入运行状态的转换都应该允许注册/缓存密码算法，而任何进入错误或关闭状态的转换都应该清除 libcrypto 中的所有缓存算法，通过采用这种方法，我们可以避免在所有密码工厂函数中检查状态，这样可以避免为自我测试（手动启动时）提供特殊情况访问权限，同时阻止外部调用者的访问。
<a name="j1wZo"></a>
### 服务
FIPS模块提供以下服务。

- 显示状态。如果“运行”状态处于活动状态，则返回1，否则返回0。
- 密码服务，如HMAC、SHS、加密。请参阅[附录3-算法](#CCzK4)。
- 自检（按需执行）- libcrypto 中提供了一个名为`FIPS_self_test()`的公共 API来访问此方法。所使用的方法必须与初始化时触发的方法相同，安全策略将指明只有在没有其他密码服务运行时才能访问此方法。
- 密钥清零。请参阅 [CSP/Key 清零](#TCvUp)。

这些服务仅在运行状态下运行，在任何其他状态下尝试访问服务将返回错误，如果自检失败，则任何尝试访问任何服务的操作都应返回错误。
<a name="GZQLw"></a>
### 自测试
自测试包括上电自测试（POST）和运行时测试（例如，确保熵不会重复作为 RNG 的输入）。<br />POST 包括模块完整性检查（每次运行使用 FIPS 的应用程序时运行）以及算法 KAT（可以在安装时运行一次）。<br />POST 测试在调用 FIPS 模块的`OSSL_provider_init()`入口点时运行。<br />为了按照正确的顺序实现完整性测试和 KAT，模块需要访问以下数据项：

1. 库的路径；
2. 库内容的 HMAC-SHA256（或包含该值的文件的路径）；
3. 指示库已安装并通过 KAT 的指示器；
4. 该指示器的 HMAC-SHA256。

这些值将成为可以通过`OSSL_PROVIDER`对象和相关的`OSSL_PARAM getter`获取的参数的一部分。将使用一个“更安全”的获取值函数来获取这些值，该函数不会展开环境变量等。此外，还需要传递用于访问和返回库内容的函数（可能是基于 BIO 的，通过将 Core 传递给模块的调度表中的少量 BIO 函数来实现），以便模块可以生成自己的库摘要。<br />新的 OpenSSL“fips” 应用程序将提供安装（运行 KAT 并输出配置文件数据）和检查（检查配置文件中的值是否有效）的功能。<br />模块的默认入口点（DEP），即 Linux 库中的 “.init” 函数，将设置一个模块变量（可能是状态变量），在`OSSL_provider_init()`中将检查此变量，并且如果设置了（通常会设置），则验证文件中的值。这个两步过程满足了 FIPS 要求，即 DEP 确保测试运行，但允许我们在正常运行时初始化模块的其余部分时实现这些测试。<br />作为构建过程的一部分，必须将 FIPS 模块的完整性校验和保存到文件中，这可以通过脚本完成，它只是使用已知固定密钥对整个 FIPS 模块文件进行 HMAC_SHA256 运算的结果，如果库已签名，则必须在应用签名之后计算校验和。<br />至少具有112位的固定密钥将嵌入到 FIPS 模块中，用于所有 HMAC 完整性操作，这个密钥也将提供给外部构建脚本。<br />出于测试目的，即使其中一个或多个测试失败，所有活动的 POST 测试也会运行。
<a name="iP8nz"></a>
#### 完整性校验和的位置
完整性校验和将在安装过程中保存到一个单独的文件中，默认情况下，该文件将与 FIPS 模块本身位于相同的位置，但可以配置为位于不同的位置。
<a name="tobeO"></a>
#### 已知答案测试
KAT 的目的是对密码模块进行健康检查，以识别在电源周期之间的模块的严重故障或变化，而不是验证实现是否正确。<br />[FIPS 140-2 IG](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/fips140-2/fips1402ig.pdf) 的规则指定了需要测试每个已支持的算法（不是每个模式），如果一个算法作为另一个测试的组成部分进行了测试，则不需要单独的测试，以下是需要进行测试的算法列表：

- 加密/解密算法
   - AES_128_GCM2
   - TDES_CBC
- 哈希算法
   - SHA1
   - SHA256 在其他地方已经要求测试
   - SHA512
   - SHA3-256
- 签名/验证测试
   - DSA_2048
   - RSA_SHA256（使用PKCS #1 v1.5填充）
   - ECDSA P256
- 任何支持的 DRBG 机制的 DRBG 健康测试
   - CTR（AES_128_CTR）
   - HASH - SHA256
   - HMAC - SHA256
- 派生测试（计算Z）
   - ECDSA P256
   - ECDH
   - KDF（密钥派生函数）
   - KBKDF（用于 TLS 的 HKDF）

注意：完整性测试使用 HMAC-SHA-256，因此不需要单独进行 HMAC 测试。
<a name="uvwMf"></a>
#### 接口访问
为了方便修改和更改运行的自测试，自测试应该是数据驱动的，POST 测试在注册任何方法之前运行，但方法表仍然可以间接使用，仍然需要较低级别的 API 来设置密钥（参数、公钥/私钥），密钥加载代码应该在一个单独的函数中进行隔离。<br />需要一个初始化方法来为高级函数设置所需的依赖项，例如在执行基本调用之前可能需要调用 set_cpuid。<br />应为摘要、密码、签名、DRBG、KDF、HMAC 提供不同类型的自测试 API。<br />传递给这些测试的参数是 KAT 数据。
<a name="kBxaP"></a>
### 安全强度
[SP 800-131A rev2](https://csrc.nist.gov/publications/detail/sp/800-131a/rev-2/draft) 在某些日期之后禁止使用特定的算法和密钥长度，这些项目与安全强度相关联。<br />允许具有至少112位安全强度的算法。<br />对于签名验证，为了遗留目的，允许使用安全强度至少为80且低于112的算法。<br />这两个值可以在FIPS模块中定义和执行，也可以在安全策略文档中更简单地处理。<br />可以通过公共API定义这些最小值，并允许设置它们。<br />还应添加目标安全强度的概念，该值将在密钥生成算法中使用，这些算法通过其标准指定了目标安全强度参数。
<a name="zaqsN"></a>
### SP800-56A & 56B
这些标准包含密钥协商协议，为了测试这些协议，加密模块需要包含以下低级原语。

- 计算密钥方法 - 这些已经存在。（例如 `DH_compute_key()`）。
- 密钥生成 - （目前缺少 [RSA FIPS 186-4](https://csrc.nist.gov/publications/detail/fips/186/4/final) 密钥生成）。
- 密钥验证 - （大部分已经实现）。
<a name="PUVea"></a>
#### FIPS 186-4 RSA 密钥生成

- 已经编写了 RSA 密钥生成的初始代码（[https://github.com/openssl/openssl/pull/6652](https://github.com/openssl/openssl/pull/6652)）；

尚未完成的工作是将其整合到 FIPS 模块中，OpenSSL FIPS Provider 将具有强制密钥大小限制的逻辑；

- 对于 RSA、DSA 和 ECDSA 密钥对生成，需要进行成对一致性测试（条件自检）。由于在密钥生成过程中无法确定密钥的用途，[FIPS 140-2 IG](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/fips140-2/fips1402ig.pdf) 规定相同的成对测试可以用于签名和加密两种模式；
- 不允许使用 1024 位的 RSA 密钥生成；
- 密钥生成算法具有目标安全强度的概念。例如，RSA 的密钥生成代码需要进行以下检查：
```c
if (target_strength < 112
    || target_strength > 256
    || BN_security_bits(nbits) < target_strength)
    return 0;
```
<a name="mCH5I"></a>
#### DH 密钥生成

- DH 密钥生成 - 可能需要将其拆分为符合标准步骤的形式。目前它是一个相当复杂用时也用于验证的整体函数。
<a name="aVUv2"></a>
#### 密钥验证

- RSA [SP 800-56B](https://csrc.nist.gov/publications/detail/sp/800-56b/rev-1/final) 密钥验证 - 公钥、私钥和密钥对的检查已添加到 [PR＃6652](https://github.com/openssl/openssl/pull/6652)，符合标准的要求；
- 需要检查 DH 密钥验证是否符合标准；
- EC 密钥验证符合标准要求；
- AES-XTS 模式需要进行扭曲密钥检查。

对于KAS DH参数，支持两种类型：

1. 已批准的安全素数群如下：

（其中 g=2，q=(p-1)/2，priv=[1, q-1]，pub=[2, p-2]）<br />TLS：(ffdhe2048, ffdhe3072, ffdhe4096, ffdhe6144, ffdhe8192)<br />IKE：(modp-2048, modp-3072, modp-4096, modp-6144, modp-8192)<br />只有上述安全素数可以进行验证-其他任何素数都应该失败。<br />安全素数可用于至少112位的安全强度，可能需要进行 FIPS 特定的检查以验证群组。

2. [FIPS 186-4](https://csrc.nist.gov/publications/detail/fips/186/4/final) 参数集仅用于向后兼容性，安全强度仅为112位。群组为 FB (2048, 224) 和 FC (2048, 256)。这需要保存种子和计数器以进行验证。

如果需要同时支持两种类型，则需要不同的密钥验证代码。<br />现有的`DH_Check()`需要进行 FIPS 特定的检查来验证批准的类型。<br />密钥生成对于两者来说是相同的（安全强度和私钥的最大位长度是输入）。<br />DSA 在 [FIPS 186-4](https://csrc.nist.gov/publications/detail/fips/186/4/final) 中定义为 “FFC”，DSA 密钥生成/密钥验证可以重新设计，以更好地匹配标准步骤，这将有助于密钥验证，并且如果需要，可以在 DH 情况下进行重用。
<a name="nO2en"></a>
### GCM IV 生成
对于 FIPS 模块，AES GCM 有与唯一密钥/IV 对相关的要求，即：

- 加密时密钥/IV 对必须是唯一的；
- IV 必须在 FIPS 边界内生成；
- 对于 TLS，IV 的计数部分必须由模块设置，模块必须确保当计数用尽时返回错误；
- 对于给定的密钥（对于任何 IV 长度），认证加密函数的总调用次数必须小于 ![](https://cdn.nlark.com/yuque/__latex/967952ba4e1b290af0e1170cb711f7f9.svg#card=math&code=2%5E%7B32%7D&id=Ontj2)；
- 模块断电不应导致 IV 的重复使用。

IV 生成将使用随机构造方法（来自[SP 800-38D](https://csrc.nist.gov/publications/detail/sp/800-38d/final)），其中包括一个自由字段（将为 NULL）和一个随机字段，随机字段将使用一个能提供至少96位熵强度的 DRBG，该 DRBG 需要由模块进行种子生成。<br />现有代码需要修改，以便在`init()`阶段未设置 IV 时生成 IV，然后可以使用`do_cipher()`方法在需要时生成 IV。
```c
int aes_gcm_cipher()
{
    ....
    /* old code just returned -1 if iv_set was zero */
    if (!gctx->iv_set) {
        if (ctx->encrypt) {
           if (!aes_gcm_iv_generate(gctx, 0))
               return -1;
           } else {
               return -1;
           }
        }
    }
}
```
生成代码如下所示：
```c
#define AES_GCM_IV_GENERATE(gctx, offset)                   \
    if (!gctx->iv_set) {                                    \
        int sz = gctx->ivlen - offset;                      \
        if (sz <= 0)                                        \
            return -1;                                      \
        /* Must be at least 96 bits */                      \
        if (gctx->ivlen < 12)                               \
            return -1;                                      \
        /* Use DRBG to generate random iv */                \
        if (RAND_bytes(gctx->iv + offset, sz) <= 0)         \
            return -1;                                      \
        gctx->iv_set = 1;                                   \
    }
```
生成的 IV 可以通过`EVP_CIPHER_CTX_iv()`方法获取，因此不需要使用 ctrl id。<br />理想情况下，在 FIPS 模式下尝试设置 GCM IV 参数将导致错误。实际上，可能仍然有一些应用程序需要设置 IV，因此建议将其指定为安全策略项。<br />安全策略还需要说明以下内容：（参见 [FIPS 140-2 IG](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/fips140-2/fips1402ig.pdf) A.5）

- 当电源断开然后恢复时，应建立一个新的用于 AES GCM 加密的密钥；
- 使用相同密钥的总调用次数必须小于![](https://cdn.nlark.com/yuque/__latex/967952ba4e1b290af0e1170cb711f7f9.svg#card=math&code=2%5E%7B32%7D&id=GChMW)；
- 场景1：IV 生成符合 TLS 协议；
- 场景2：使用 [NIST SP 800-38D](https://csrc.nist.gov/publications/detail/sp/800-38d/final)（第8.2.2节）的 IV 生成方法。
<a name="TCvUp"></a>
### CSP/Key 清零
当不再需要临界安全参数（CSP）时，我们必须将其全部设置为零，这可能会在不同的上下文中发生：

- 临时的 CSP 副本可能是栈或堆分配的，并且将在其使用范围内的相关函数中被清零；
- 一些 CSP 将与 OpenSSL 对象（如 EVP_PKEY 或 EVP_CIPHER_CTX）关联，具有与之相关联的生命周期，在这种情况下，当这些对象被释放时，CSP 将在该点被清零。在某些情况下，对象可能会被复用（例如，EVP_CIPHER_CTX 可以用于多个加密操作），在这种情况下，对象中仍存在的任何 CSP 将在其重新初始化为新操作时被清零。
- 一些 CSP（例如内部 DRBG 状态）可能会在加载 OpenSSL FIPS 模块的整个时间内存在，在这种情况下，状态将封装在 OpenSSL 对象中。所有 OpenSSL Provider（包括 FIPS 模块 Provider）都具有注册“卸载”函数的能力，该函数在关闭 OpenSSL 时（或因其他原因卸载模块）时将被调用，该卸载函数将释放（因此将清零）包含 CSP 的对象。
- 根据 [FIPS 140-2 IG](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/fips140-2/fips1402ig.pdf) 4.7 的规定：仅用于执行 [FIPS 140-2](https://csrc.nist.gov/publications/detail/fips/140/2/final) 第4.9.1节上电测试的密码模块使用的密码密钥不被视为 CSP，因此不需要满足 [FIPS 140-2](https://csrc.nist.gov/publications/detail/fips/140/2/final) 第4.7.6节的清零要求。

OpenSSL FIPS 模块将包含其自己的标准`OPENSSL_cleanse()`函数的副本来执行清零操作，这是使用特定于平台的汇编语言实现的。	
<a name="CpvET"></a>
### DRBG
在旧的 FIPS 模块中存在以下 API，可能需要重新添加：

- **FIPS_drbg_health_check**：按需运行 DRBG KAT 测试，我们需要使其可用。
- **FIPS_drbg_set_check_interval**：设置运行 DRBG KAT 测试之间的间隔（生成调用的次数）。这似乎不是必需的，这些测试在上电时运行，但后续不需要运行，但这个调用对于故障测试很有用。
<a name="uONNi"></a>
#### 推导函数
根据 [FIPS 140-2 IG 14.5](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/fips140-2/fips1402ig.pdf) 中的第2点要求，CTR DRBG 将无条件支持派生函数，如果禁用派生函数，当前的代码在重新生成种子时会出现问题。此外，如果没有派生函数，需要从实验室获得额外的理由支持。
<a name="XpBmS"></a>
#### 测试要求

- 在`uninstantiate()`函数中需要证明内部状态已被清零；
- 故障测试需要一个函数，使 DRBG 始终产生相同的输出。
<a name="y5JKO"></a>
#### 其他需要考虑的事项
除了下面描述的熵之外，还需要考虑以下几点：

- 应考虑实施 [NIST SP 800-90C](https://csrc.nist.gov/publications/detail/sp/800-90c/draft) 10.1.2 中的熵扩展；
- 需要更好的 DRBG 选择机制，以在可用的 DRBG 之间进行选择；
- 支持预测抵抗，即在请求时尝试从我们的来源收集更多熵。
- 我们需要弄清楚 DRBG 层将是什么样子，代码的很大一部分需要放在 FIPS 模块内。目前，此代码访问 EVP 功能，而这些功能可能不会在模块内部公开。例如，`drbg_ctr_init()` 从 NID 解析 EVP_CIPHER，然后设置一个 EVP_CIPHER_CTX。
<a name="T7y8t"></a>
### 熵
对于所有平台，操作系统将提供熵。对于某些平台，还可以使用内置的硬件随机数生成器，但这将引入额外的验证需求。<br />对于类 UNIX 系统，将使用系统调用`getrandom`、`getentropy`或随机设备`/dev/random`作为熵源，优先考虑使用系统调用。可以替代`/dev/random`的其他强随机设备包括：`/dev/srandom`和`/dev/hwrng`。请注意，`/dev/urandom`、`/dev/prandom`、`/dev/wrandom`和`/dev/arandom`在没有额外的证明的情况下不能用于 FIPS 操作。<br />在 Windows 上，将使用`BCryptGenRandom`或`CryptGenRandom`作为熵源。<br />在 VMS 上，将使用各种系统状态信息作为熵源。请注意，这将需要证明和分析以证明源的质量。<br />对于 iOS，将使用 [SecRandomCopyBytes](https://developer.apple.com/documentation/security/1399291-secrandomcopybytes) 生成具有[密码学安全性的随机字节](https://developer.apple.com/documentation/security/secrandomref)。<br />FIPS 仅允许将一个熵源归因于模块，因此 FIPS 模块将完全依赖于前述的操作系统源，不会使用其他来源，例如 egd、硬件设备等。
<a name="aOHgi"></a>
#### 完成熵解决方案的工作
需要将 DRBG 健康检测添加到随机框架中，以检查输入到 DRBG 的种子材料，检查的目的是确保没有连续的两个种子材料块是相同的，该检查在所有熵源被合并在一起后进行，如果检查失败，则 DRBG 的种子重新生成操作将永远失败。我们可以定义使用的块大小为64位，这是在意外接收到重复块的概率（![](https://cdn.nlark.com/yuque/__latex/a437106bd01d30b2af4c65182a767997.svg#card=math&code=2%5E%7B-64%7D&id=dRUNg)）和从操作系统中获取过多熵量之间的平衡（因为第一个块会被丢弃），其他明显可用的块大小包括128位和256位。<br />使用后，初始数据块必须被清零和丢弃。
<a name="BXqI7"></a>
#### GCM的初始化向量（IV）
[FIPS 140-2 IG](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/fips140-2/fips1402ig.pdf) A.5 的最新更新指出，如果模块声称为 GCM 生成随机 IV，则需要提供证明，我们需要证明模块能够从操作系统获取所需的96位熵，如果使用阻塞调用操作系统的随机性源，并且至少使用这么多熵素材作为 DRBG 的种子材料，那么这应该不是无法解决的问题。
<a name="hA49g"></a>
### FIPS 模块边界
一旦进入 FIPS 模块提供的算法，在任何其他的加密操作中我们必须仍然保持在 FIPS 模块内部。根据 FIPS 规则，允许一个 FIPS 模块使用另一个 FIPS 模块。然而，在3.0设计中，为了简化起见，我们假设不允许这样做。例如，`EVP_DigestSign*`实现同时使用签名算法和摘要算法，我们不允许其中一个算法来自 FIPS 模块，另一个来自其他 Provider。<br />所有 Provider 在初始化时都被分配一个唯一的`OSSL_PROVIDER`对象，当 FIPS 模块被要求使用某个算法时，它会验证该算法的实现`OSSL_PROVIDER`对象是否与自己的`OSSL_PROVIDER`对象相同（即传递给`OSSL_provider_init`的对象），例如，考虑使用 RSA 和 SHA256 的`EVP_DigestSign*`的情况，两个算法都会通过 Core 在 FIPS 模块外部进行查找，RSA 签名算法是第一个入口点，"init" 调用将被传递给要使用的 SHA256 算法的引用，FIPS 模块的实现将检查与其被要求使用的 SHA256 实现关联的`OSSL_PROVIDER`对象是否也在 FIPS 模块边界内，如果不是，则 "init" 操作将失败。下面的图示从 FIPS 模块的角度说明了这个操作。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687162145466-4ad0d3e3-cc69-453f-a0fe-12a2b1d59bfb.png#averageHue=%23fafafa&clientId=ud9c33b12-ba37-4&from=paste&id=u36822a5d&originHeight=884&originWidth=1526&originalType=binary&ratio=2&rotation=0&showTitle=false&size=92838&status=done&style=none&taskId=u5a0a1bc3-5e03-4110-95e2-673defa21c7&title=)<br />请注意，在 FIPS 模块内部，我们使用了EVP的概念（如`EVP_MD_CTX`、`EVP_PKEY_CTX`等）来实现这一点，这些是 libcrypto 中 EVP 实现的副本，FIPS 模块没有与 libcrypto 进行链接，这是为了确保完整的操作都在 FIPS 模块的边界内进行，而不调用外部的代码。
<a name="Cmhtb"></a>
### ASN.1 代码
ASN.1 DER（Distinguished Encoding Rules）用于：

- 序列化**密钥**和**参数**
- 序列化由两个值 r 和 s 组成的 **DSA 和 ECDSA 签名**
- 编码放置在 **RSA PKCS #1** 填充中的签名摘要 OBJECT IDENTIFIER（OID）
- 序列化 X.509 证书和证书撤销列表（CRL）
- 其他 PDU，如 PKCS #7/CMS、OCSP、PKCS #12 等。

FIPS 模块不会包含 ASN.1 DER 编码器/解析器的副本，也不会要求任何 Provider 对由 OpenSSL 实现的算法执行 ASN.1 序列化/反序列化。<br />所有 ASN.1 序列化/反序列化操作将在 libcrypto 中执行，复合值的**密钥**、**参数和签名**结构将作为项数组越过 Core/Provider 边界，使用 [附录2 - 参数传递](#Q6UPj) 中定义的公共数据结构进行传递。<br />用于 **RSA PKCS #1** 填充的编码摘要 OID 将预先生成（与旧 FIPS 模块使用 SHA_DATA 宏相同）或根据需要使用简单函数生成，这个函数仅为 PKCS #1 填充支持的小型摘要集合生成编码的 OID，这些摘要 OID 在“OID 树”下的一个公共节点下，验证填充时将获取预期摘要的编码 OID，并将其字节与填充中的字节进行比较；不需要进行 DER 解析/解码。
<a name="RaFOc"></a>
## 代码维护
<a name="fYEtO"></a>
### 源代码结构/目录树的清理
密码学实现（`crypto/evp/e_*.c`和大部分`crypto/evp/m_*.c`，以及任何定义`EVP_CIPHER`、`EVP_MD`、`EVP_PKEY_METHOD`、`EVP_MAC`或`EVP_KDF`的代码）必须移出 evp 目录，它们最终将成为一个或两个 Provider 的一部分，因此它们应该位于特定 Provider 的子目录中。<br />将创建一个新的目录`providers/`，用于存放特定 Provider 的代码。`providers/build.info`定义了哪些源文件在哪些 Provider 模块中使用。
<a name="h9mnq"></a>
### 共享源代码
FIPS Provider 模块和默认 Provider 将共享相同的源代码，在不同的条件下，例如不同的`#include`路径或定义的宏不同（后者需要在构建系统中添加支持）。下面是一个示例`build.info`文件，实现了这一点：
```c
PROVIDERS=p_fips p_default

SOURCE[p_fips]=foo.c
INCLUDE[p_fips]=include/fips

SOURCE[p_default]=foo.c
INCLUDE[p_default]=include/default
```
或者，使用宏：
```c
PROVIDERS=p_fips p_default

SOURCE[p_fips]=foo.c
DEFINE[p_fips]=FIPS_MODE

SOURCE[p_default]=foo.c
```
注意：一些关键字还不是 build.info 语言的一部分。
<a name="chOsw"></a>
### 条件代码
我们需要对编译时包含 FIPS 特定代码的方法进行一致处理，并在某些情况下排除 FIPS 不允许的代码。<br />编译时的控制将通过`#ifdef FIPS_MODE`来进行，这确保所有相关的文件都明确地为非 FIPS 或在 FIPS 模块内部进行编译，由于每个文件都将被编译两次（在默认 Provider 和 FIPS 模块中各一次），一次使用每个设置，因此使用具有恒定值的运行时if语句没有好处。（此外，运行时设置并不总是有效（例如在扩展诸如`BLOCK_CIPHER_custom`之类的宏时，会创建全局变量或函数指针。）<br />构建系统将通过使用`-DFIPS_MODE`编译 FIPS Provider 对象文件，以及不带命令行定义的来自相同源的默认 Provider 对象文件来支持此操作。<br />对于运行时检查，将需要检查 TLS 连接是否处于 FIPS 模式，这可以通过以通用方式检查与特定`SSL_CTX` 或`SSL`对象关联的属性查询字符串来完成，以查看是否设置了“fips”属性。
<a name="J3tY4"></a>
## FIPS 测试
需要进行以下类型的测试：

- 用于 CMVP 验证算法的 CAVS 测试；
- 能够运行所有 FIPS 模块算法的 FIPS 测试套件；
- 启动后故障测试；
- Acumen 将编写使用 libcrypto 的应用程序，通过 EVP 层访问 FIPS Provider。

任何需要返回中间值（例如 CAVS 密钥生成）以显示信息（自检状态）或更改 FIPS 模块代码的正常流程的特殊情况代码（例如自检失败或在提供固定随机值的密钥生成循环中失败），将通过将回调函数嵌入到FIPS模块代码中进行控制。<br />建议将这些回调代码以条件编译的方式编入模块中中，因为其中一些值不应该被返回（例如，FIPS 模块不应该输出密钥生成中的中间值）。<br />针对需要使用固定 rand_bytes 的测试，将对`rand_bytes()`进行重写。
<a name="wwRYr"></a>
### FIPS 测试回调
应用程序可以选择提供一个回调函数，用于处理从 FIPS 模块接收到的值（如果需要，可以注册多个回调函数）。<br />可选的应用程序回调函数的形式如下：
```c
static int fips_test_callback(const char *type, void *arg)
{
    return 1;
}
```
回调函数的返回值可用于控制 FIPS 模块代码中的特殊情况的流程。<br />类型由 FIPS 模块钩子传入，FIPS模块中的每个不同的钩子应具有唯一的类型，类型决定了参数 arg 的内容（可以是结构体（例如中间值）、名称或整数）。<br />FIPS 模块中的回调函数的形式如下：
```c
MY_STRUCT  data;   /* values that need to be returned to the application */
data.i = 1;
.....
if (FIPS_test_cb != NULL)
    FIPS_test_cb(FIPS_TEST_CB_RSA_KEYGEN_GET, (void *)&data);
```
<a name="XuHLv"></a>
### POST 故障测试和日志记录
为了支持多个测试的失败，所有测试将始终运行而不提前退出（只是标记失败），在所有测试完成后，将返回失败状态。<br />用于日志记录或失败的参数将为：
```c
struct {
    const char *desc;
    const char *state;
    const char *fail_reason;
};
```
其中：

- type 是“post_integrity”、“post_cipher”、“post_digest”、“post_signature”、“post_drbg”等之一；
- desc 是标识性名称，例如 AES_128_CBC；
- state 是以下之一：
   - “start” - 表示测试开始
   - “corrupt” - 如果返回值为0，则测试将失败
   - “pass” - 表示测试通过
   - “fail” - 表示测试失败
- fail_reason - 是失败的具体原因（例如，无法读取完整性模块文件或完整性校验和文件）。
<a name="VV4ed"></a>
### CAVS测试
CAVS 测试将由实验室执行。<br />然而，每个 CAVS 测试文件也可以进行抽样，并添加到单元测试中，这意味着可以将单个测试的文件数据转换为单元测试内的二进制数据。<br />（DRBG_ctr 是已经实现了此功能的示例）。<br />这将确保以下内容：

- CAVS 测试将可访问所需的接口（某些 CAVS 测试需要访问通常不需要的内部接口）；
- 算法的正常工作；
- 覆盖率。

如果与实验室之间有良好的沟通，我们可以跳过此步骤，但如果实验室发现缺少对内部访问器的访问，可能需要在代码中添加一些额外的回调钩子。
<a name="Ku8zo"></a>
## 遗留问题
<a name="lxdcw"></a>
### EVP 到低级 API 的桥接
在某些地方，低级 API 结构被赋值给`EVP_PKEY`对象，对公共的`EVP_PKEY`的影响是它必须保留指向可能的低级结构的指针，并且在 libcrypto 内部必须知道该低级结构的类型，每当具有这种指针的`EVP_PKEY`用于任何计算时，都必须检查低级结构是否发生了变化，并将其数据转换为可以与新的 Provider 一起使用的参数。<br />确定如何检查低级结构的内容是否发生了变化的具体机制有待确定，一种可能的方法是在低级结构中设置一个 dirty 计数器，并在`EVP_PKEY`结构中复制一个副本，每当低级结构发生变化时（例如，`RSA_set0_key`等函数必须执行递增操作），每当`EVP_PKEY`用于计算时，就会将其副本与低级脏计数器进行比较，如果它们不同，则`EVP_PKEY`的 Provider 参数将使用低级结构的数据进行修改。<br />（另一个想法是通过遗留函数在`EVP_PKEY`中放置一个回调函数，如果检测到低级别的更改，则更新参数）
<a name="H4TQg"></a>
### EVP方法创建者
在 OpenSSL 1.1.x 中有可以轻松创建各种EVP方法结构的功能，可以像这样找到：
```c
grep EVP_CIPHER_meth util/libcrypto.num
grep EVP_MD_meth util/libcrypto.num
grep EVP_PKEY_meth util/libcrypto.num
```
<a name="v2Q0n"></a>
### 相关类型
低级 API 相当独立，因此除了在某些类型中添加了一个 dirty 标志外，所有低级 API 类型将保持不变。关联的`EVP_CIPHER`、`EVP_MD`、`EVP_PKEY_METHOD`、`EVP_MAC`或`EVP_KDF`实例通过在遗留 Provider 模块中通过实现调度表来单独处理（见下文）。
<a name="ZlbPZ"></a>
## 遗留 Provider 模块
一些被认为是“遗留”的算法（例如 IDEA）且具有当前的`EVP_CIPHER`、`EVP_MD`、`EVP_PKEY_METHOD`、`EVP_MAC`或`EVP_KDF`实现将移至一个名为 "Legacy" 的 Provider 模块，而不是我们的默认 Provider 模块。<br />以下算法的方法将成为 "Legacy" Provider 模块中的调度表：

1. Blowfish
2. CAST
3. DES（但不包括3DES）
4. DSA
5. IDEA
6. MD2
7. MD4
8. MDC2
9. RC2
10. RC4
11. RC5
12. RIPEMD160
13. SEED
14. Whirlpool

（注意：这不是一个详尽无遗的列表，尽管对目前来说相当完整）
<a name="oNAdQ"></a>
## ENGINE API
整个 ENGINE API 将在此版本之后的主要发布中被弃用和移除，到那时，人们必须学会如何创建 Provider 模块。与此同时，它将被转变为一个工具，帮助实现者从 ENGINE 模块实现过渡到 Provider 模块实现。<br />由于算法构造函数将被更改为构造调度表，因此 ENGINE 类型将变为一组调度表，并且 ENGINE 构造函数的功能将更改为将它们收集到给定的 ENGINE 中。<br />通过这种注册方式注册的调度表将添加一个名为 "engine" 的属性，并将 ENGINE 标识作为 Provider 名称属性。这将使`ENGINE_by_id`和类似功能能够找到正确的 Provider。<br />ENGINE 模块的入口点 bind_engine 将被替换为 Provider 模块的入口点，并且宏`IMPLEMENT_DYNAMIC_BIND_FN`将被更改为构造此类入口点。此入口点将创建一个类似 Provider 的 ENGINE 结构，调用绑定函数，该函数将使用与之前相同的方法创建函数将其填充到调度表中，然后使用与以前相同的方法设置函数将所有这些收集到的调度表在 ENGINE 结构中注册，就像任何 Provider 模块一样。<br />与此版本的其余部分一样，我们的目标是源代码级的兼容性。<br />在 OpenSSL 1.1.x 及更早版本中，可以使用诸如`ENGINE_get_default_RSA`和`ENGINE_get_RSA`等函数，将 ENGINE 提供的方法挂钩以替代内置于 libcrypto 中的函数，前者无需修改，而后者将被更改为根据附加到引擎的相应调度表创建旧式方法（例如`RSA_METHOD`）。

---

<a name="NDwuz"></a>
## 附录1 - 属性语法
属性定义和查询具有明确定义的语法。本节将以 eBNF 和铁路图的形式介绍该语法。几乎是 eBNF，但在某些地方使用了正则表达式扩展。

| <a name="XyUbq"></a>
### 定义
 | ![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687184382300-2fafd6e1-1808-469e-89af-994a3d30447e.png#averageHue=%232a2514&clientId=ud9c33b12-ba37-4&from=paste&id=u84ecc9bf&originHeight=102&originWidth=338&originalType=binary&ratio=2&rotation=0&showTitle=false&size=13598&status=done&style=none&taskId=ubf4bed96-14ec-4959-bb16-028ac8ce9a7&title=) |  |
| --- | --- | --- |
| ```c
Definition
      ::= SingleDefinition ( ',' SingleDefinition )*

SingleDefinition
      ::= PropertyName ( '=' Value )?
```

---

 |  |  |
| <a name="m5TJS"></a>
### 查询
 | ![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687184889083-3fb2a630-2269-4233-a040-27e00e702cc8.png#averageHue=%23252111&clientId=ud9c33b12-ba37-4&from=paste&id=ud3519776&originHeight=181&originWidth=377&originalType=binary&ratio=2&rotation=0&showTitle=false&size=21705&status=done&style=none&taskId=ud92b198b-f21c-406b-a712-ad29c7a08ba&title=) |  |
| ```c
Query ::= SingleQuery ( ',' SingleQuery )*

SingleQuery
      ::= '-'? PropertyName
        | PropertyName ( '=' | '!=' ) Value )
```

---

 |  |  |
| <a name="w7dyn"></a>
### 值
 | ![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687185031974-1c20479b-2bad-42df-946d-60397a8e6dd3.png#averageHue=%23605837&clientId=ud9c33b12-ba37-4&from=paste&id=u76067d99&originHeight=73&originWidth=186&originalType=binary&ratio=2&rotation=0&showTitle=false&size=9976&status=done&style=none&taskId=ud2cd6bea-7930-4601-82a8-780fd556f8f&title=) |  |
| ```c
Value ::= NumberLiteral
        | StringLiteral
```
 |  |  |
| <a name="Nuawj"></a>
### 字符串
 | ![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687185286611-3e21eb01-f2db-4c81-90e5-3bcf24682203.png#averageHue=%23635b39&clientId=ud9c33b12-ba37-4&from=paste&id=ubf254c77&originHeight=73&originWidth=197&originalType=binary&ratio=2&rotation=0&showTitle=false&size=10768&status=done&style=none&taskId=ua32488b8-f8b0-4b3e-8c51-b0878b6fbb2&title=) |  |
| ```c
StringLiteral
      ::= QuotedString
        | UnquotedString
```

---

 |  |  |
| <a name="S8Rfo"></a>
### 引号字符串
 | ![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687185331577-4d10226f-92b9-4f63-95e1-e5d468cc4446.png#averageHue=%23211e0f&clientId=ud9c33b12-ba37-4&from=paste&id=u785a42ff&originHeight=136&originWidth=294&originalType=binary&ratio=2&rotation=0&showTitle=false&size=18231&status=done&style=none&taskId=ua0ae442c-a9a8-4373-bdb9-1b3d18f3a82&title=) |  |
| ```c
QuotedString
      ::= '"' [^"]* '"'
        | "'" [^']* "'"
```

---

 |  |  |
| <a name="tpXhL"></a>
### 无引号字符串
 | ![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687185356350-45adfad8-5775-45c1-a265-d5e5fb10a06d.png#averageHue=%234c4835&clientId=ud9c33b12-ba37-4&from=paste&id=u5dcc6a7c&originHeight=48&originWidth=186&originalType=binary&ratio=2&rotation=0&showTitle=false&size=8700&status=done&style=none&taskId=u8caf1b0a-b8ac-464f-81be-96821b25080&title=) |  |
| ```c
UnquotedString
      ::= [^{space},]+
```

---

 |  |  |
| <a name="Da0e8"></a>
### 数字
 | ![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687185378749-28670bc2-9271-4bba-968a-76a307e1905a.png#averageHue=%231d1a11&clientId=ud9c33b12-ba37-4&from=paste&id=u8649cc5c&originHeight=275&originWidth=339&originalType=binary&ratio=2&rotation=0&showTitle=false&size=31379&status=done&style=none&taskId=ua32f13a3-8cc8-4d5a-9c16-238bb90489b&title=) |  |
| ```c
NumberLiteral
      ::= '0' ( [0-7]+ | 'x' [0-9A-Fa-f]+ )
        | '-'? [1-9] [0-9]+
```

---

 |  |  |
| <a name="is6ie"></a>
### 属性名称
 | ![image.png](https://cdn.nlark.com/yuque/0/2023/png/26770235/1687185408762-d0127fee-a66c-4e20-ab1e-c1b6a5b4704c.png#averageHue=%23201d12&clientId=ud9c33b12-ba37-4&from=paste&id=u6e9c53a9&originHeight=183&originWidth=255&originalType=binary&ratio=2&rotation=0&showTitle=false&size=18038&status=done&style=none&taskId=u3b3d2ce0-2310-47f3-ae64-aa7ad9cf3af&title=) |  |
| ```c
PropertyName
      ::= [A-Z] [A-Z0-9_]* ( '.' [A-Z] [A-Z0-9_]* )*
```
 |  |  |


---

<a name="Q6UPj"></a>
## 附录2 - 参数传递
Core 或 Provider 对象应该对外部所有内容保持不透明，然而，我们仍然需要能够以统一的方式从中获取参数或向其传递参数。因此，我们需要一个中间的非透明结构来支持此操作。<br />传递的数据类型需要保持简单：

- 数字（任意大小的整数）
- 字符串（假定为UTF-8编码）
- 八位字节串（任意大小的字节数组）

传递值给模块的任何参数都需要包含以下项：

- 标识符，用于指示传递的参数是什么
- 值的类型（来自上述列表）
- 值的大小
- 值本身

用于请求模块的值的任何参数都需要包含以下项：

- 标识符，用于指示请求的是什么
- 值的类型（来自上述列表）
- 缓冲区的大小
- 用于填充值的缓冲区
- 结果输出大小，由我们请求参数的函数填充

这两个结构非常相似，可以表示为同一个结构：
```c
typedef struct ossl_param_st {
    const char *key;
    unsigned char data_type;    /* declare what kind of content is sent or
                                   expected */
    void *buffer;               /* value being passed in
                                   or out */
    size_t buffer_size;         /* buffer size */
    size_t *return_size;        /* OPTIONAL: address to
                                   content size */
} OSSL_PARAM;
```
使用示例：
```c
    /* passing parameters to a module */
    unsigned char *rsa_n = /* memory allocation */
#if __BYTE_ORDER == __LITTLE_ENDIAN
    size_t rsa_n_size = BN_bn2lebinpad(N, rsa_n, BN_num_bytes(rsa_n));
#else
    size_t rsa_n_size = BN_bn2bin(N, rsa_n);
#endif
    struct OSSL_PARAM rsa_params[] = {
        { RSA_N, OSSL_PARAM_INTEGER, rsa_n, rsa_n_size, NULL },
        { 0, 0, 0, 0, 0 },
    };

    EVP_set_params(pkey, rsa_params);

    /* requesting parameters from a module */
    size_t rsa_n_buffer_size = BITS / 2 / 8 + 1;
    unsigned char *rsa_n_buffer =
       OPENSSL_malloc(rsa_n_size);
    size_t rsa_n_size = 0;
    OSSL_PARAM rsa_params[] = {
        { RSA_N, OSSL_PARAM_INTEGER, rsa_n_buffer, rsa_n_buffer_size,
          &rsa_n_size },
        { 0, 0, 0, 0, 0 },
    };

    EVP_get_params(pkey, rsa_params);
    
    /*
     * Note: we could also have a ctrl functionality:
     * EVP_ctrl(pkey, EVP_CTRL_SET_PARAMS, rsa_params);
     * EVP_ctrl(pkey, EVP_CTRL_GET_PARAMS, rsa_params);
     *
     * This would allow other controls using the same API.
     * For added flexibility, the signature could be something like:
     *
     * int EVP_ctrl(EVP_CTX *ctx, int cmd, ...);
     */
```
<a name="sU04N"></a>
### 数据类型
该规范支持以下参数类型：

- INTEGER（整数）
- UNSIGNED_INTEGER（无符号整数）
   - 这些类型可以是任意长度，可能需要一个任意大小的缓冲区。
   - 数字以本机形式存储，即在大端系统上以最高有效位（MSB）为先，而在小端系统上以最低有效位（LSB）为先。这意味着可以在缓冲区中存储任意本机整数，只需确保缓冲区大小正确且对齐。
- REAL（实数）
   - 以其本机格式和对齐方式存储 C 的二进制浮点值。
- UTF8_STRING（UTF-8字符串）
   - 该字符串类型预期可以直接打印。
- OCTET_STRING（字节字符串）
   - 在打印时，预期该字符串以十六进制形式呈现。

我们还支持相同类型的指针变体（这意味着`OSSL_PARAM`缓冲区只需具有`void *`的空间），这种用法在指向的值随时间变化时是脆弱的，除非这些值是恒定的。<br />我们有一些宏来声明`data_type`中内容的类型：
```c
#define OSSL_PARAM_INTEGER              1
#define OSSL_PARAM_UNSIGNED_INTEGER     2
#define OSSL_PARAM_UTF8_STRING          3
#define OSSL_PARAM_OCTET_STRING         4
/*
 * This one is combined with one of the above, i.e. to get a string pointer:
 * OSSL_PARAM_POINTER | OSSL_PARAM_UTF8_STRING
 */
#define OSSL_PARAM_POINTER           0x80
```
<a name="boiiu"></a>
### 实现细节
<a name="egWcq"></a>
#### 确定缓冲区的大小
在请求参数值时，调用者可以选择在一个或多个参数结构中将 buffer 赋值为 NULL。被调用的 getter 应该通过填充 return_size 指向的大小并返回来回应这样的请求，此时调用者可以分配适当大小的缓冲区，并进行第二次调用，此时 getter 可以毫无问题地填充缓冲区。<br />如果程序员希望，return_size 可以指向同一个 OSSL_PARAM 中的 buffer_size。
<a name="ujkjp"></a>
### 潜在的其他用途
<a name="xk6rY"></a>
#### 将 CONF/NCONF 值作为参数
配置值对于 Provider 来说是有兴趣的！然而，仅仅在 Core 和 Provider 之间传递一个 CONF 指针可能是不可行的，尽管它当前是一个非透明的结构。<br />另一种方法是将 CONF/NCONF 值转换为参数，其中的灵感来自于 git 配置值的命名方式。<br />让我们首先设想一个与当前 ENGINE 配置模块类似的 Provider 配置：
```c
[provider_section]
# Configure provider named "foo"
foo = foo_section
# Configure provider named "bar"
bar = bar_section

[foo_section]
provider_id = myfoo
module_path = /usr/lib/openssl/providers/foo.so
selftests = foo_selftest_section
algorithms = RSA, DSA, DH

[foo_selftest_section]
doodah = 1
cookie = 0
```
对于 Provider "foo"，Core 侧的 Provider 结构可以响应以下请求的参数 keys：

- "provider_id"（值为"myfoo"）
- "module_path"（值为"/usr/lib/openssl/providers/foo.so"）
- "selftests.doodah"（值为1）
- "selftests.cookie"（值为0）
- "algorithms"（值为"RSA, DSA, DH"）

请注意，section 名称本身从未出现在参数 key 中，而是出现在导致该 section 的 key 中。这是因为 OpenSSL 允许任意命名的 section 名称。
<a name="h4jF7"></a>
### 时间齿轮
上述定义的参数结构并非即兴发明，它受到了长期以来被证明稳定的 OpenVMS 编程范例的启发，实际上启发这种结构的是一个名为 "item_list_3" 的结构，在这里有相关的文档：[《OpenVMS编程概念手册，第一卷》](https://support.hpe.com/hpsc/doc/public/display?docId=emr_na-c04621447)。
<a name="CCzK4"></a>
## 附录3 - 算法
| **要求** |  | **标准** | **说明** |
| --- | --- | --- | --- |
| TDES | CBC | [FIPS 81](https://csrc.nist.gov/publications/detail/fips/81/archive/1980-12-02) | 另请参阅 [SP 800-67rev2](https://csrc.nist.gov/publications/detail/sp/800-67/rev-2/final)。<br />TDES 仅支持解密（从 2020 年开始）和禁止解密（从 2025 年开始）。 |
|  | ECB | [FIPS 81](https://csrc.nist.gov/publications/detail/fips/81/archive/1980-12-02) | 对数据长度施加限制。<br />需要有关 [SP 800-67rev1](https://csrc.nist.gov/publications/detail/sp/800-67/rev-1/archive/2012-01-23) 转换和限制的安全策略声明。 |
| AES | CBC | [SP 800-38A](https://csrc.nist.gov/publications/detail/sp/800-38a/final) | 支持 128、192 和 256 位的所有 AES 密码模式。 |
|  | CBC CTS |  |  |
|  | CCM | [SP 800-38C](https://csrc.nist.gov/publications/detail/sp/800-38c/final) |  |
|  | CFB | [SP 800-38A](https://csrc.nist.gov/publications/detail/sp/800-38a/final) |  |
|  | CTR | [SP 800-38A](https://csrc.nist.gov/publications/detail/sp/800-38a/final) |  |
|  | ECB | [SP 800-38A](https://csrc.nist.gov/publications/detail/sp/800-38a/final) |  |
|  | GCM | [SP 800-38D](https://csrc.nist.gov/publications/detail/sp/800-38d/final) |  |
|  | GMAC | [SP 800-38D](https://csrc.nist.gov/publications/detail/sp/800-38d/final) |  |
|  | OFB | [SP 800-38A](https://csrc.nist.gov/publications/detail/sp/800-38a/final) |  |
|  | XTS | [SP 800-38E](https://csrc.nist.gov/publications/detail/sp/800-38e/final) | 请参阅 [FIPS 140-2 I.G](https://csrc.nist.gov/CSRC/media/Projects/Cryptographic-Module-Validation-Program/documents/fips140-2/FIPS1402IG.pdf). A.9。需要添加密钥检查。此模式不支持 192 位。检查在 [#7120](https://github.com/openssl/openssl/pull/7120) 中已添加。 |
|  | KW | [SP 800-38F](https://csrc.nist.gov/publications/detail/sp/800-38f/final) | 与标准有差异，但仍在标准范围内。 |
|  | KWP | [SP 800-38F](https://csrc.nist.gov/publications/detail/sp/800-38f/final) |  |
| HASH | SHA-1 | [FIPS 180-4](http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf) |  |
|  | SHA-2 | [FIPS 180-4](http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf) | 224, 256, 384, 512, 512/224, 512/256. |
|  | SHA-3 | [FIPS 202](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf) | 224, 256, 384, 512. |
| HMAC | SHA-1 | [FIPS 198-1](https://www.nist.gov/publications/keyed-hash-message-authentication-code-hmac-0?pub_id=901614) |  |
|  | SHA-2 | [FIPS 198-1](https://www.nist.gov/publications/keyed-hash-message-authentication-code-hmac-0?pub_id=901614) | 224, 256, 384, 512. |
|  | SHA-3 | [FIPS 198-1](https://www.nist.gov/publications/keyed-hash-message-authentication-code-hmac-0?pub_id=901614) |  |
| CMAC |  |  |  |
| GMAC |  |  |  |
| KMAC |  |  |  |
| DRBG | AES CTR | [SP 800-90A](https://csrc.nist.gov/publications/detail/sp/800-90a/rev-1/final) | [SP 800-90C](https://csrc.nist.gov/publications/detail/sp/800-90c/draft) 存在一些问题，但所有内容都符合[SP 800-90A](https://csrc.nist.gov/publications/detail/sp/800-90a/rev-1/final)的要求。 |
|  | HASH | [SP 800-90A](https://csrc.nist.gov/publications/detail/sp/800-90a/rev-1/final) |  |
|  | HMAC | [SP 800-90A](https://csrc.nist.gov/publications/detail/sp/800-90a/rev-1/final) |  |
| RSA |  | [FIPS 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf) | 还请参考[SP 800-56B](https://csrc.nist.gov/publications/detail/sp/800-56b/rev-2/draft)。涉及到 PKCS#1.5、PSS 和密钥对生成方面的内容，还有模数大小的变化。 |
|  | Key wrap (transport) | [SP 800-56B](https://csrc.nist.gov/publications/detail/sp/800-56b/rev-2/draft) | OAEP。更新到[SP 800-56B rev-1](https://csrc.nist.gov/publications/detail/sp/800-56b/rev-2/draft)的标准。 |
| DH | KAS | [SP 800-56A](https://csrc.nist.gov/publications/detail/sp/800-56a/rev-3/final) | 更新到[SP 800-56A rev-3](https://csrc.nist.gov/publications/detail/sp/800-56a/rev-3/final)的标准。 |
|  | KAS | [KASVS](https://csrc.nist.gov/CSRC/media/Projects/Cryptographic-Algorithm-Validation-Program/documents/keymgmt/KASVS.pdf) | 额外的测试以满足ZZonly、CVL/KAS要求。 |
| DSA |  | [FIPS 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf) | PQG 生成和验证、签名生成和验证、密钥对生成。 |
| ECDSA |  | [FIPS 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf) | 密钥对生成、公钥生成、签名生成和验证。 |
| ECC | KAS | [SP 800-56A](https://csrc.nist.gov/publications/detail/sp/800-56a/rev-3/final) | B-233、283、409、571；K-233、283、409、571；P-224、256、384、521。更新到[SP 800-56A rev-3](https://csrc.nist.gov/publications/detail/sp/800-56a/rev-3/final)标准。 |
|  | KAS | [KASVS](https://csrc.nist.gov/CSRC/media/Projects/Cryptographic-Algorithm-Validation-Program/documents/keymgmt/KASVS.pdf) | 额外的测试以满足ZZonly、CVL/KAS要求。 |
| KDF | PBKDF2 | [SP 800-132](https://csrc.nist.gov/publications/detail/sp/800-132/final) | 验证符合标准。详见 [#6674](https://github.com/openssl/openssl/pull/6674)。 |
|  | HKDF |  |  |
|  | SSKDF |  |  |
|  | SSHKDF |  |  |
|  | X9.42 KDF |  |  |
|  | X9.63 KDF |  |  |
|  | KBKDF |  |  |
|  | TLS PRF |  |  |
| TLS | PRF |  | 适用于 TLS 1.2 和 1.3。 |

<a name="JI5ez"></a>
## 备注

1. 由于[FIPS 140-2 IG](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/fips140-2/fips1402ig.pdf) 9.8的规定，不需要对 DRBG 的输出进行测试。然而，仍需要对输入到主 DRBG 的种子材料进行 RCT 或固定位测试。[↩︎](#ucc3fdbb5)
2. 草案指南已经发生了变化，可选的算法有：AES_GMAC，AES_128_CCM，AES_256_GCM 和 AES_256_CCM，可以认为 GMAC 是这三者中最简单的，因此可能更可取。[↩︎](#uab28ec78)
3. 对于 HASH 和 HMAC DRBGs 使用不同的摘要算法将消除对摘要的独立测试的需求。[↩](#u70148537)
4. 引号字符串可以包含 UTF-8 字符。[↩︎](#S8Rfo)
5. 无引号字符串会经过小写转换，并且只能包含ASCII字符。[↩︎](#tpXhL)
6. 属性名称是不区分大小写的，即使这里只显示了大写形式。[↩︎](#u1012dcc3)
