---
date: '2022-10-20T20:09:28+09:00'
title: '名词解释：Web3 账户相关概念大梳理'
slug: 'web3-account-concepts'
categories: ['Web3', 'Crypto']
tags: ['account abstraction', 'wallet', '4337']
isCJKLanguage: true
---
刚刚结束的 Devcon 上，账户抽象算是是最热的几个话题之一，最近可以经常看到 AA / EOA / SCW / 4337 等缩写和代号在各种 talk、panel 和信息流里出现。再加上叙事开始往「Onboarding next billion users」的方向发展，一些新的形容词也开始出现在产品之前，比如 seedless / gasless / social recovery / non-custodial。相信看完这两句的你已经开始脑壳疼了，那么接下来就让我尽自己所能来帮大家梳理一下这些名词概念到底代表什么。

>阅前提示：本文不是严肃的技术文档，可能会用不精确但容易理解的语言进行阐述或比喻，欢迎大家以此为起点深入探索这些技术的细节。

## EOA - Externally Owned Accounts

EOA 中文叫做 **外部账户**，我们最熟悉的 MetaMask 生成的地址就是 EOA。它的特点是原理简单，比如生成规则是：

`私钥 → 公钥 → Keccak256 哈希 → 最后 20 Bytes → 十六进制字符串（EOA 地址）`

可以看出这个规则非常直接，全是由数学变换计算出来的，生成的地址内部没有任何结构和逻辑。节点验证一笔交易是否被地址 owner 授权的时候也是固定的规则：

`交易签名 → ec_recover → 公钥 → （用上面的规则生成）地址 → 对比要操作的地址`

对比结果一致那么验签通过，进行后续流程；不通过则直接打回，不会进一步广播交易。

EOA 的另一个设定是作为交易的发起方并支付 gas，相对应的 CA（合约账户） 只能被其他 CA 或者 EOA 调用。也就是说，EOA 是交易的触发器，一笔交易无论后面有多少合约调用，一开始都必须由一个 EOA 发起并且支付足够的 gas 才可以进行。

需要指出的是，EOA 是以太坊以及其他 EVM 兼容链（或类 EVM 链）才有的概念，严格来说包括 BTC 在内的主流非 EVM 链都没有这个设定。

## CA - Contract Accounts

CA 中文叫做 **合约账户**（也曾被称为 **内部账户**），我们常见的 ERC-20 代币合约、DeFi 业务合约等都有一个跟 EOA 长得很像的地址，这就是 CA。

在设定上，CA 是以太坊世界的原住民，EOA 和 ETH 是为 CA 的业务逻辑准备的触发器和燃料；实际使用下来，以太坊上除 ETH 之外的所有资产都是由 CA 承载，DeFi 等业务逻辑就更是全都由 CA 来实现。然而 CA 无法主动进行操作和支付 gas 的设定也限制了它的能力，早在[2016 年就有提案](https://github.com/ethereum/EIPs/issues/61)希望能让 CA 自己支付 gas。

简单来说，CA 是具备内部逻辑的以太坊账户，里面既可以是业务逻辑（Token 合约用来记账，质押合约用来放贷和清算），也可以是账户逻辑（比如 gnosis safe 的多签逻辑），而后者就是我们即将提到的「SCW - 智能合约钱包」概念。

CA 的地址规则是通过计算生成的，有 CREATE 和 CREATE2 两种方式，这里不再展开。大家只需要记住 CA 和公钥没有必然对应关系即可，比如 gnosis safe 创建的 CA 里可以设定任意多把公钥来解锁它的地址对应的资产；当然 CA 也可以不设定任何密钥，而是由其他 CA 的逻辑决定是否可以解锁，比如 DeFi 的借贷合约，只要还了钱就能取回质押的资产。

## SCW/A - Smart Contract Wallet/Account

**智能合约钱包** 应该是字面意思最好理解的了，也就是用 CA 作为地址的钱包方案，而我们常用的 EOA 钱包方案是用前述的公钥变换结果作为地址。由于具备内部逻辑，智能合约钱包可以实现很多 EOA 无法实现的功能，比如 gas 代付，批量交易，权限管理，离线授权，社交恢复等等。

这里举几个例子来展示一下智能合约钱包的扩展潜力：

1. Gnosis safe 利用智能合约钱包架构实现多签逻辑；
2. 用户可以在一笔上链交易中同时给多个地址发送不同的 token，也可以在用 uniswap 时让 approve 和 swap 在一笔交易里完成，从而做到需要多少授权多少，避免因为过度授权造成安全隐患。
3. 用户可以给不同资产设定不同的操作权限，比如给 PFP 设定比普通 ERC-20 token 更高的操作门槛（例如需要一把由硬件钱包管理的 admin key 才能转移），这样即便日常使用的环境发生密钥泄露，黑客也无法将高价值资产转走，在安全和便利中间取得平衡。
4. 用户可以签署一个离线授权「谁能给我 100 ETH，就可以转走我的某个 BAYC」，这样不需要授权给第三方合约，用户就可以跟其他人 P2P 地完成原子交易。

## AA - Account Abstraction

**账户抽象** 其实不是一个新概念了，最早可以追溯到 2015 年的一些讨论，当时 Vitalik 认为至少要让以太坊用来验证交易的密码学算法做到可替换，比如换成性能更优的 ed25519（[详见这里](https://blog.ethereum.org/2015/07/05/on-abstraction/)），可以说 7 年来 Vitalik 和 EF 都没有停止对账户抽象方案的讨论和探索，这里有个整理好的 [linktree](https://hackmd.io/@matt/r1neQ_B38) 可以帮大家回顾一下历史。

那么账户抽象怎么理解呢？这里我引用一下 [ERC-4337](https://eips.ethereum.org/EIPS/eip-4337) 里对其目标的描述：

> Achieve the key goal of account abstraction: allow users to use smart contract wallets containing arbitrary verification logic instead of EOAs as their primary account. Completely remove any need at all for users to also have EOAs (as status quo SC wallets and [EIP-3074](https://eips.ethereum.org/EIPS/eip-3074) both require)

可以看出以太坊对于账户抽象的期望是改变目前大多数人都在使用 EOA 的现状，希望用户转向 SCW，并且把生态对 EOA 的依赖完全去除。除了里面提到的 [EIP-3074](https://eips.ethereum.org/EIPS/eip-3074) 之外，还有一个更为激进和远期的 [EIP-5003](https://eips.ethereum.org/EIPS/eip-5003)，这里同样引述几段原文（有省略）：

> EOAs … are limited by the protocol in a variety of critical ways. These accounts do not support rotating keys for security, batching to save gas, or sponsored transactions to reduce the need to hold ether yourself. There are countless other benefits that come from having a contract account or account abstraction, like choosing one’s own authentication algorithm, setting spending limits, enabling social recovery, allowing key rotation, arbitrarily and transitively delegating capabilities, and just about anything else we can imagine.

> …This EIP provides a path not to enshrine EOAs, but to provide a migration path off of them, once and for all.

不难看出，EIP-5003 的目标是一次性将 EOA 转换为 CA，让所有用户用上 SCW，彻底解决向前兼容的问题。（经过上面的名词解释，看这些缩写是不是顺畅了些？）

到这里大家对 AA 的来龙去脉和未来目标应该有所了解了。但需要指出的是，AA 这个概念不是以太坊和 EVM 专属的，很多链原生已经具备了不同程度的 AA 特性。比如 EOS / Polkadot / Near / Solona / Flow / Aptos … 甚至 BTC（单签 / 多签 / Taproot），这些链在设计时就已经将账户做成了有内部结构甚至具备权限管理能力的状态，还有 StarkNet / CKB 等具备更完善的账户抽象能力。说到这里大家不难发现，以太坊的 AA 是在解决 EOA 意外地流行带来的历史遗留问题，从而在账户层面上变得更加先进和灵活。

## 4337 - ERC 4337

从上面对 AA 的讨论里不难看出，ERC-4337 只是这个方向众多提案中的一个，但是为什么大家一提到 AA 或者 SCW 就会说到它呢？我们来看这个文档的副标题：

> An account abstraction proposal which completely avoids consensus-layer protocol changes, instead relying on higher-layer infrastructure.

也就是说，ERC-4337 是 AA 的路线第一次从「暴力革命」转向「和平演变」，不再追求利用共识层的改变实现 AA，而是转而使用 SCW 这种用户层的方案。并且为了实现更好的互操作性，ERC-4337 定义了一些 SCW 应该实现的接口，以及元交易打包、gas 代付等基础设施的框架。它的出现让目前差异极大的各种 SCW 方案能够拥有统一的用户交互界面以及共用一些生态层面搭建的开放基础设施，有助于各种场景快速实现自己需要的 SCW 方案。另一方面，ERC-4337 的推动有助于促进生态其他参与方提升对 SCW 的兼容性，比如验签需要的 [EIP-1271](https://eips.ethereum.org/EIPS/eip-1271) 和有些 DeFi 协议里定义的禁止 CA 交互的一些规则。

## Seedless

这里的 seed 指的是 seed phrase，就是我们创建钱包的时候经常被要求备份的助记词。那么 seedless 的意思就是「无助记词的」，或者也可以说成「无私钥的」。注意这个「无」并不是实际意义上的没有密钥，而是指不需要用户备份助记词 / 私钥或者感知到它们的存在。

一个常见的问题是，如果用户不备份助记词，用户是不是就没有账户的控制权了？一旦用户切换新设备环境，账户不就无法访问了吗？没错，只是把用户备份助记词的功能砍掉的话只能算是产品设计失误，而 seedless 追求的是用户「不需要」知道助记词的存在，同时依然拥有账户的完全控制权。也就是说，用户（且只有用户自己）拥有在新设备自主恢复账户控制的能力，只是不再依赖助记词这种 UX 很差、过于 geek 的方式，比如下面要讲到的社交恢复就是非常好的一种。

## Gasless

这里的 gas 指的是 gas fee，所以 gasless 的意思是「免 gas fee 的」。同样的，gasless 也不是真的不需要支付 gas fee，而是指用户不需要被迫去了解 gas 概念，更不用提前购买各种原生代币来支付 gas。

那么 gas 谁来付？分两种情况：

一种是用户账户里已经有 crypto asset 的时候，比如 play to earn 得到 token，或者领到的空投，亦或是别人的转账，只要这些 token 有一定的价值和流动性，就会有 relayer 愿意接受它们并帮用户支付 gas，以此赚取收益。

另一种是用户账户里没有有价 token，比如刚刚创建的账户。如果此时需要链上交互，应用方可以选择资助用户一些「定向」用途的 gas 来帮他们 bootstrap，从而降低用户流失，这时即便算上 gas 补贴的消耗，整体的用户获取成本反而可能会更低；或者可以通过让用户观看广告等方式来换取一些 gas。这两种策略在 gas 成本较低的 L2 上都非常有效。

## Social Recovery

**社交恢复** 是指利用社交关系帮助用户在丢失密钥的情况下重新获得账户访问权的机制。如果你用微信登录过新设备，应该有过「让你的两个朋友发送 xxx 给你的账号以登录」的体验——这就是社交恢复想达到的效果，只不过验证方从微信变成了智能合约。

一种常见的误区是把利用社交账号来创建 / 登录钱包的方案称为社交恢复，这是错把「社交关系」与「社交平台账号」划了等号。老牌智能合约钱包 Argent 就内置了社交恢复能力，它要求你的 guardian 提供一个以太坊地址，从而在你需要登陆新设备时提供签名来进行授权，然而这一方案的潜在设定就是：你的 guardian 一定比你在管理以太坊账户上更专业，否则当你需要他们签名的时候，如果他们自己的账户已经无法访问，你的账户也会连带遭殃。所以一种更加可行的办法是利用 email 的密码学证明（DKIM Signature）或者电子护照等生活中常见的密码学工具来增强社交恢复方案的实用性。

## Non-custodial

**非托管** 可以说是 crypto 行业最政治正确、也是被滥用最多的概念之一了，因为很多时候各家都会有自己的定义。这里我也分享一下我们对非托管的定义，主要有两方面：

1. 钱包开发商**无法擅自操作**用户的账户

2. 钱包开发商**无法阻止用户操作**自己的账户

如果你也认同这两点，那么判断一个钱包是托管、半托管还是非托管就可以直接拿这两个规则去检验了：

`不满足 1 → 托管；满足 1 不满足 2 → 半托管；1、2 都满足 → 非托管`

那么知道了是哪种托管程度有什么用吗，用户可能并不 care 背后的原理，只要好用就行了呗！没错，其实我也部分认同这种观点，至少在现在的阶段，行业还没有发展到发生用户认知范式转移的程度。其实我认为三种类型的方案分别适用于不同的场景：

1. **托管方案** - 适用于交易所、大机构金服、强合规等场景，比如 coinbase 提供的一些服务。特点是用户量少，不需要应对高频交互，而且客单价高，能支撑服务商花费大成本来维护一系列高防系统。

2. **半托管方案** - 适用于相对高端的个人用户群体。他们明白服务方可以审查自己的交易，并且有能力提前准备备份方案（比如导出私钥），在服务方主动或被动拒绝服务时可以不影响自己的资产安全。这样日常使用时可以享受安全和便利，极端情况下可以保全资产。注意这种方案对服务商的运维能力要求也非常高，毕竟个人用户量大，日常跟各种应用的交互需求也更高，再就是对数据可用性要求高，毕竟一旦丢失服务端保存的数据有可能导致所有没备份的用户永远无法访问账户。

3. **非托管方案** - 适用于面向 mass adoption 的场景。初听上去可能是反直觉的，但是从成本上讲，非托管方案是唯一能够在低客单价的场景里保证足够的安全性和可用性的方案。如果一个面向大规模用户场景的应用方打算选择上面两种方案，就一定要考虑对方能否为自己的用户群提供足够安全可用的服务，否则一旦内部人员作恶、黑客入侵或不可抗力导致服务停摆，自己的所有用户都会受到牵连，自己的业务也可能因此一蹶不振。历史上的无数次案例都在讲述一个故事，安全无小事，为用户负责就是为自己负责。

## MPC - Multi-Party Computation

**多方安全计算** 跟零知识证明（ZKP）可以并称当下 Web3 两大「魔法」，一旦跟它们沾边，似乎原来做不到的事情 somehow 就能做了。实际上有些情况是这样的，尤其是 ZKP，可以利用概率换可行性；MPC 则是通过分散控制权来达成风控或者灾备能力。

MPC 其实是一种范式，包含很多技术方案，在目前 Web3 的语境下大都指的是 tss。

### TSS - Threshold Signature Scheme

**门限签名** 是一种分布式多方签名协议，包含分布式密钥生成、签名，以及在不改变公钥的情况下更换私钥碎片的 re-sharing 等算法。

一个 m-n 的 tss 指的是一个公钥对应了 n 个私钥碎片，其中 m 个碎片的联合签名可以被公钥验签成功。不难发现这个逻辑类似于多签（multi-sig），他们的区别主要在公钥的数量上。

举例来说，2-2 的多签是一个门上挂了 2 把锁，必须用两个钥匙把它们都打开才能开门；2-2 的 tss 是一个门上挂了 1 把锁，但是钥匙有两片，合起来用才能打开门。这里为了好理解，描述并不严谨，两把钥匙合成一把其实更符合 Shamir Secret Sharing 算法的情况；tss 算法下的密钥碎片是不会相遇的，而是它们分别签名之后，通过特定算法可以用对应的公钥验签通过。

那么 tss 是不是一定是托管或者非托管的？其实没有必然联系，主要看最终的方案如何设计和取舍。非托管方案要求用户拥有独立操作账户的能力，所以用户必须掌握不少于门限数量的密钥碎片，例如 2-3 的话用户需要掌握 2 片，而 2-2 的方案无法达成非托管，最多可以做到半托管（比如 ZenGo）；但是如果用户管理最多的私钥碎片，那么势必会提高对用户能力的要求，很难做到 mass adoption。

写到这里应该把常见的 Web3 账户相关的名词都覆盖到了，数了一下字数也有差不多 5k 了。这么多的内容难免有错误和疏漏的地方，还请大家不吝拍砖，发现问题或者有不同观点直接来 Twitter 找我提就好（[@frank_lay2](https://twitter.com/frank_lay2)），后面有内容增改或更新我也会在 Twitter 上及时跟大家同步。