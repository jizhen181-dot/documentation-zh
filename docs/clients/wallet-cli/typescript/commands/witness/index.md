# wallet-cli witness

注册并运营超级代表（SR）候选资格。

注册会把一个普通账户变成**超级代表候选人**——它可以被投票（[`vote cast`](../vote/cast.md)），如果得票进入前 27 名就会开始出块。候选资格同时也是治理权限的前提：只有已注册的见证人才能创建或批准[提案](../proposal/index.md)。

链上关于候选人保存的信息极少：owner 地址，以及唯一的一个 **url**——浏览器中显示在该 SR 旁边的信息页——这也是本命令组唯一能修改的字段。SR 的其他一切（排名、得票、出块）都是投票的结果，而不是可设置项。

唯一的经济参数是**佣金比例**（brokerage）：SR 自己保留的出块奖励份额，其余部分分配给它的投票人。默认为 20 %。

注册会燃烧一笔费用（目前约 9,999 TRX），且无法撤销。

## 用法

```
wallet-cli witness COMMAND
```

## 子命令

| 命令 | 页面 | 说明 |
|---|---|---|
| `witness create` | [create.md](create.md) | 将账户注册为 SR 候选人 |
| `witness update` | [update.md](update.md) | 修改候选人信息页 URL |
| `witness set-brokerage` | [set-brokerage.md](set-brokerage.md) | 设置 SR 保留的出块奖励份额 |

候选人、他们的得票和佣金比例可用 [`vote list`](../vote/list.md) 查看——它默认只显示当选的 27 位 SR，因此要看不在其中的候选人需要加 `vote list --candidates`。

## 另请参见

[`proposal`](../proposal/index.md) · [`vote list`](../vote/list.md) · [`reward balance`](../reward/balance.md)
