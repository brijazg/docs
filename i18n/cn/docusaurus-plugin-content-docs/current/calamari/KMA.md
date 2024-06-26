# 🐙 Calamari 代币 KMA

我们将关于 Calamari 代币的一些基本特性列于下表：

| 属性                 | 数值               | 标注                                       |
|---------------------|--------------------|-------------------------------------------|
| 代币名称             | `Calamari`         |                                           |
| 代币符号             | `KMA`              |                                           |
| 总供应量             | 10,000,000,000 KMA | 10 Billion KMA                            |
| Decimal             | 12                 | 1 KMA = 1,000,000,000,000 基本单位         |
| 最小余额             | 0.1 KMA            | 用来维持账户活跃的最小余额                    |

## 最小余额

**最小余额** 是用户为了维持钱包活跃而需要在账户保留的最小余额。举个例子，如果 Bob 创建了一个余额为0的账户，默认情况下，该账户是不会在账本状态中显示的。现在，如果 Alice 给 Bob 发送不足`0.1 KMA`，那么 Bob 的账户将被加入 `pallet_balances` as part of ledger state。然而，如果 Alice 试图给 Bob 发送低于 `0.1 KMA`，这笔转账将被拒绝，因为一旦账本接受该笔转账，将违反  **最小余额** 规则。同样的，Alice 也不能转出余额，因为账户余额不足`0.1 KMA` 。她可以选择在账户中预留超过 `0.1 KMA` 或者将所有的余额转出并在账本状态中移除账户。

## Calamari 代币应用

作为 Calamari 的原生通证, `KMA` 有以下几项主要应用：

- 存留押金： 作为 Calamari 的原生代币，每一个账户都需要保留最小数量的`KMA` 作为存留押金
- 外部费用支付：`KMA` 可被用户支付外部费用，用来防止 Calamari 网络上的 DDoS 攻击
- 手续费：作为隐私支付/隐私兑换的手续费
- 链上治理：`KMA` 持有者可参与链上治理，为治理提案进行投票

## Calamari 代币分配

`KMA` 的分配以社区主导为原则。绝大部分的`KMA`代币将由 Calamari 社区成员持有。

![Calamari Supply](/img/calamari-supply.png)

无团队分配; 也没有私募额度。代币首次社区分配是面向 Calamari 第一次 Kusama 众贷参与者。