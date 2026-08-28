# 发送 TRX 和 token

一条命令即可发送全部三种资产——原生 TRX、TRC20、TRC10——由选择器参数决定发送哪一种。命令示例都在 Nile 上运行。

> **密码**：用**软件账户**真正签名的 `tx send` 需要从 stdin 传入 master password，且签名过程不会有任何提示（`--dry-run` / `--build-only` 不签名、Ledger 账户在设备上签名，都不需要密码）。下面的示例省略了这部分，以便聚焦 token 相关参数——请自行在前面加上 `printf '%s' "$PW" |`、在后面加上 `--password-stdin`，或者从密码管理器管道传入（见[快速上手](getting-started.md#3-send-your-first-transaction)）。

## 原生 TRX

```bash
wallet-cli tx send --to TSx72ViULFepRGCS4PM5dP4FqD1d8qggCc --amount 1 --network tron:nile
```

`--amount` 是人类可读的 TRX 数量（`1` = 1 TRX = 1,000,000 SUN）。想用精确的基础单位？改用 `--raw-amount 1000000`——两者只能选其一，绝不能同时使用。

## TRC20 token

TRC20 token 由它的**合约地址**标识。你可以直接传入该地址，也可以传入一个简短的**符号**，由 wallet-cli 帮你解析成对应合约：

```bash
# 用合约地址——总是可用
wallet-cli tx send --to T... --contract TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t --amount 5 --network tron:nile

# 用符号——需要该 token 已在地址簿中（见下文）
wallet-cli tx send --to T... --token USDT --amount 5 --network tron:nile
```

TRC20 转账会执行合约代码，因此会消耗**能量**；`--fee-limit` 限制为此最多可燃烧多少 TRX（默认 100000000 SUN = 100 TRX）。如果一次转账因触及费用上限而失败，先弄清原因再考虑调高它——见[能量与带宽](../concepts/energy-bandwidth.md)。

### token 地址簿（符号 → token）

`--token USDT` 的工作方式是在按网络划分的 **token 地址簿**中查找该符号：这是一张本地表，把符号映射到链上的 token——可以是一个 TRC20 合约，也可以是一个 TRC10 资产 id。条目有两个来源，显示在 `token list` 的 `Source` 列中：

- **official**——针对知名 token 内置的条目。在**主网**上预置了 `USDT` 和 `USDC`；**测试网**（Nile、Shasta）不附带任何官方条目。
- **user**——你自己添加的 token。

```bash
wallet-cli token list --network tron:mainnet
```

```console
| Symbol | Name       | Source   | Contract / ID                      |
| ------ | ---------- | -------- | ---------------------------------- |
| USDT   | Tether USD | official | TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t |
| USDC   | USD Coin   | official | TEkxiTehnzSmSe2XqrBj4w32RUN966rdz8 |
```

由于测试网没有任何官方条目，你需要先添加一次 token，`--token` 才能在那里解析出来。`token add` 会从 token 上读取符号和精度，并存为一条 **user** 条目——TRC20 用 `--contract`，TRC10 用 `--asset-id`——且作用范围仅限于那一个网络：

```bash
wallet-cli token add --contract T... --network tron:nile      # TRC20，按合约地址
wallet-cli token add --asset-id 1000001 --network tron:nile   # TRC10，按数字资产 id
```

其余管理操作都在同一个命令组里：`token list` 查看全部条目，`token remove` 删除一条 user 条目，`token balance` / `token info` 在不添加 token 的前提下直接查询它。

## TRC10 token

TRC10 资产用的是数字 id，而不是合约：

```bash
wallet-cli tx send --to T... --asset-id 1002000 --raw-amount 1000000 --network tron:nile
```

`--token`、`--contract` 和 `--asset-id` 三者互斥；一个都不给就表示发送原生 TRX。

## 先演练，再发送

`--dry-run` 只构建交易并估算费用，不签名也不广播——资产绝不会离开你的钱包：

```bash
wallet-cli tx send --to T... --token USDT --amount 5 --network tron:nile --dry-run -o json
```

检查输出中的 `fee` 块，然后去掉 `--dry-run` 重跑一次。仅仅提交并不代表成功——之后要么用 [`tx status`](../commands/tx/status.md) 确认，要么给 `tx send` 命令加上 `--wait`，让它阻塞到交易被确认或失败为止。

> **主网**：同样的命令换成 `--network tron:mainnet` 就会转移真实资产，且不可逆。请反复核对 `--to`（转错地址且已确认的转账就再也拿不回来了），并先做 dry run。

## 另请参见

[`tx send` 参考](../commands/tx/send.md)——每个参数和输出字段 · [`token` 命令](../commands/token/index.md)——token 地址簿 · [快速上手](getting-started.md) · [脚本编写](scripting.md)——安全地自动化发送
