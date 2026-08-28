# wallet-cli gasfree

通过 GasFree 服务进行免 gas 的 token 转账。

`gasfree` 让你在不持有任何 TRX 的情况下转移 token：你用 EIP-712 结构化数据签名对转账签名，再由 GasFree 服务（[open.gasfree.io](https://open.gasfree.io)）代你上链。费用直接从所转的那种 token 里扣——每笔转账一笔服务费，首次转账另加一次性的激活费——所以**完全不需要 TRX**。

## 用法

```
wallet-cli gasfree COMMAND
```

## 子命令

| 命令 | 页面 | 说明 |
|---|---|---|
| `gasfree info` | [info.md](info.md) | 你的 GasFree 地址、激活状态、`nonce` 与费率表 |
| `gasfree transfer` | [transfer.md](transfer.md) | 签署一笔免 gas 转账并提交给服务方 |
| `gasfree trace` | [trace.md](trace.md) | 用 `traceId` 跟踪已提交的转账 |

## 工作原理

- 每个账户都有一个**确定性派生出的 GasFree 地址**。资产在该地址上收取和支付——要通过 GasFree 收 USDT，就把你的 GasFree 地址（见 `gasfree info`）给付款方。
- **首次**转账时，服务方会在链上激活这个 GasFree 地址，并在每笔转账服务费之外额外收取一次性的激活费。两笔费用都从 token 中扣除。
- 签名按每个地址各自的 **`nonce`** 排序，以防重放。
- 需要服务方的 **API 凭据**——用 [`config`](../config.md) 设置 `gasfreeApiKey` / `gasfreeApiSecret`。`--network` 用于选择服务环境（主网 / 测试网）。

与 [`tx send`](../tx/send.md) 相比：`tx send` 在链上广播，会消耗带宽/能量或燃烧 TRX；`gasfree transfer` 走服务方的 API——不花 TRX，但每笔转账要付一笔以 token 计价的费用。手上有 TRX 或能量时，`tx send` 通常更便宜；`gasfree` 针对的是“完全没有 TRX”的情形。

## 另请参见

[`gasfree info`](info.md) · [`gasfree transfer`](transfer.md) · [`gasfree trace`](trace.md) · [`config`](../config.md) · [`tx send`](../tx/send.md)
