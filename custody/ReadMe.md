# DappLink 资金托管系统

## 一.系统介绍

### 1.系统架构

[![Dapplink](./images/arc.png)](https://github.com/dapplink-baas/baas-docs)

基于链上/链下协同调度的去中心化 MPC（多方安全计算）网络。整个系统由以太坊主链、MPC 动态委员会、MPC 节点集群、mpc-manager 服务以及上层托管钱包共同组成。

首先，所有 MPC 节点需由 Operator 在链上进行注册和质押，链上的 onchain scheduler 与 MpcManagerService 会根据 staking protocol 提供的节点状态，对节点进行管理与调度。链下的 offchain scheduler 与节点实时同步状态，确保节点可用性和委员会成员的动态更新。

在执行 keygen 或签名任务时，MPC 动态委员会会从已注册节点中选取若干节点组成子委员会。mpc-manager 负责向这些节点广播 keygen 指令或 messageHash，并收集节点返回的部分签名。最终聚合出的公钥将生成托管钱包地址，而聚合签名将用于用户的链上交易。

用户通过托管钱包申请地址、充值和提现，所有签名均由去中心化 MPC 节点生成，从而形成高安全性、可审计、去中心化的签名体系。整个架构确保节点可替换、任务可调度、签名可信且密钥永不落地。

### 2.签名风控

[![Dapplink](./images/rist.png)](https://github.com/dapplink-baas/baas-docs)

结合风控密钥（RiskKey） 与 钱包密钥（WalletKey） 的双层安全 TSS（Threshold Signature Scheme）交易签名体系。整个流程由业务层、风控服务、钱包服务（wallet-services）、TSS 管理层（tss-manager）以及多个 TSS 节点（tss-node）共同协作完成。

业务层发起提现交易后，交易首先进入风控服务。风控模块维护独立的 RiskKey，并对交易生成 Hash(RiskKey + tx)。若交易通过风控验证，交易被推送到 wallet-services。钱包服务负责管理 WalletKey，对交易体生成 Hash(WalletKey + tx)，并构造统一的 MessageHash，随后把三项数据 —— 交易体、Hash(RiskKey + tx)、Hash(WalletKey + tx) —— 一并发送给 tss-manager。

tss-manager 验证所有 Hash 是否一致后，将交易广播至各个 TSS 节点。每个节点先检验 Hash(RiskKey + tx) 与 Hash(WalletKey + tx) 是否正确，确保数据在传输中未被篡改；验证通过后，各节点进入多轮 TSS 协同计算流程并生成部分签名；最终由 tss-manager 聚合为完整的 Signature 返回给 wallet-services。

wallet-services 获得签名后将其附加到交易体并广播到链上。通过双 Key 体系和分布式 TSS，系统实现了高等级安全、可控风控、密钥不落地的可信签名架构。

### 3.资管系统链路风控

在整个钱包体系中，风控模块作为第一道安全防线，对每一笔交易进行全生命周期的动态追踪与校验。每当业务层发起提现或敏感操作，交易首先进入风控中心，由风控维护的独立 RiskKey 对交易体进行签名级别的哈希校验（Hash(RiskKey + tx)），确保风险验证不可伪造。

风控模块不仅执行额度、频次、地址画像、黑名单等基础策略，还同步接入链上链下数据，实现交易路径溯源与实时反洗钱（AML）分析。例如，风控可识别资金是否来自高风险地址、是否存在分散再聚合（structuring）行为、是否触发地缘或 KYC 风险，从而在交易进入 TSS 签名流程前提前阻断不合规行为。

所有交易的风控结果都会完整记录，形成可查询的审计链路，支持事后追踪、合规检查和风险回溯。一旦交易通过风控验证，其哈希将与钱包侧生成的 Hash(WalletKey + tx) 一同送入 tss-manager，进入分布式 TSS 签名流程。

依托该风控体系，每笔交易都具备 可跟踪、可审计、可反洗钱、可追溯 的特性，在不暴露密钥的前提下大幅增强钱包的安全性与合规能力，满足机构级数字资产托管要求。

## 二.api 文档

- https://dlwallet.dapplink.xyz/swagger/index.html#/
