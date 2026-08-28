# 投票、奖励与见证人

为超级代表投票、管理佣金比例并领取奖励，以及创建 / 更新见证人。

## 如何投票 {#how-to-vote}

投票需要份额（share）。份额可以通过冻结资金获得。

- 份额的计算方式是：每冻结 **1 TRX** 可获得 **1** 单位份额。
- 解冻之后，此前的投票将失效。你可以通过重新冻结并投票来避免投票作废。

**注意** TRON 网络只记录你最后一次投票的状态，也就是说，你的每一次投票都会覆盖此前的全部投票
结果。

例如：

```console
> freezeBalance 100000000 3 1 address  # 冻结 10TRX，获得 10 单位份额

> votewitness 123455 witness1 4 witness2 6  # 同时为 witness1 投 4 票、为 witness2 投 6 票

> votewitness 123455 witness1 10  # 为 witness1 投了 10 票
```

上述命令的最终结果是 witness1 得 10 票，witness2 得 0 票。

## 佣金比例（Brokerage） {#brokerage}

为见证人投票后，你将获得奖励。见证人有权决定佣金比例。默认比例为 20%，见证人可以调整它。

默认情况下，如果见证人获得奖励，他们将得到全部奖励的 20%，其余 80% 分配给他们的投票者。

### GetBrokerage

查看见证人的佣金比例。

```console
> getbrokerage OwnerAddress
```

`OwnerAddress`——见证人账户的地址，base58check 类型的地址。

### GetReward

查询未领取的奖励。

```console
> getreward OwnerAddress
```

`OwnerAddress`——投票者账户的地址，base58check 类型的地址。

### UpdateBrokerage

更新佣金比例。该命令通常由见证人账户使用。

```console
> updateBrokerage OwnerAddress brokerage
```

- `OwnerAddress`——见证人的账户地址，base58check 类型的地址。
- `brokerage`——你想更新到的佣金比例，取值 0 到 100。如果输入 10，表示总奖励的 10% 分配给 SR，
  其余部分奖励给所有投票者，在这个例子中即 90%。

示例：

```console
> getbrokerage TZ7U1WVBRLZ2umjizxqz3XfearEHhXKX7h

> getreward  TNfu3u8jo1LDWerHGbzs2Pv88Biqd85wEY

> updateBrokerage TZ7U1WVBRLZ2umjizxqz3XfearEHhXKX7h 30
```

## WithdrawBalance

提取投票奖励或出块奖励。

每个区块产生后，出块奖励会发送到账户的 allowance，允许每 **24 小时**从 allowance 提取一次到
balance。allowance 中的资金不能被锁定或交易。

```console
> WithdrawBalance [owner_address]
```

```console
> WithdrawBalance TEDapYSVvAZ3aYH7w8N9tMEEFKaNKUD5Bp
```

## 如何创建见证人 {#how-to-create-witness}

申请成为见证人账户需要消耗 **100_000 TRX**。这部分资金将被直接燃烧。

### CreateWitness

申请成为超级代表候选人。

```console
> CreateWitness [owner_address] url
```

```console
> CreateWitness TEDapYSVvAZ3aYH7w8N9tMEEFKaNKUD5Bp 007570646174654e616d6531353330363038383733343633
```

### UpdateWitness

编辑 SR 官方网站的 URL。

```console
> UpdateWitness TEDapYSVvAZ3aYH7w8N9tMEEFKaNKUD5Bp 007570646174654e616d6531353330363038383733343633
```

## ListWitnesses

获取所有矿工节点信息。

## GetPaginatedNowWitnessList

分页获取当前见证人列表。

```console
wallet> getPaginatedNowWitnessList 0 2
{
	"witnesses": [
		{
			"address": "TJmka325yjJKeFpQDwKSQAoNwEyNGhsaEV",
			"voteCount": 5405926918,
			"url": "http://sr-8.com",
			"totalProduced": 1801675,
			"totalMissed": 456,
			"latestBlockNum": 64577529,
			"latestSlotNum": 590063589,
			"isJobs": true
		},
		{
			"address": "TFFLWM7tmKiwGtbh2mcz2rBssoFjHjSShG",
			"voteCount": 2322244615,
			"url": "http://sr-27.com",
			"totalProduced": 1807756,
			"totalMissed": 619,
			"latestBlockNum": 64577530,
			"latestSlotNum": 590063590,
			"isJobs": true
		}
	]
}
```

## 另请参见

- [concepts/resources](../concepts/resources.md)——份额如何获得
- [stake-v2](stake-v2.md)——冻结 TRX 以获得投票份额
