# 资源：带宽、能量与份额

TRON 账户通过冻结（质押）TRX 来获得资源。本页汇总各操作页面所引用的机制。

## 冻结产生的份额与带宽

资金冻结之后，会获得相应数量的份额（share）和带宽（Bandwidth）。份额可用于投票，带宽可用于交易。

- **份额（Share）**——每冻结 1 TRX 可获得 1 单位份额。份额用于[投票](../commands/vote-reward.md#how-to-vote)。
  解冻之后，此前的投票将失效。
- **带宽（Bandwidth）**——由合约消耗（转账、资产转移、投票、冻结等）。查询不消耗带宽。

## 如何计算带宽

带宽的计算规则是：

```
constant * FrozenFunds * days
```

假设冻结 1 TRX（1_000_000 Sun）3 天，则获得的带宽 = 1 * 1_000_000 * 3 = 3_000_000。

所有合约都会消耗带宽，包括转账、资产转移、投票、冻结等。查询不消耗带宽。每个合约需要消耗
**100_000 带宽**。

如果一个合约超过一定时间（**10s**），该操作不消耗带宽。

发生解冻操作时，带宽不会被清空。下一次执行冻结时，新增的带宽会累加。

## 资源价格

带宽和能量的历史单价以及备注费均可查询——参见 [commands/resources](../commands/resources.md)。

## 另请参见

- [commands/stake-v2](../commands/stake-v2.md)——当前的质押模型
- [commands/stake-v1-legacy](../commands/stake-v1-legacy.md)——旧版冻结
- [concepts/staking-models](staking-models.md)
