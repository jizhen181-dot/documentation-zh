# 交易所（Bancor）

内置的链上交易所。交易对的交易和价格波动遵循 Bancor 协议，相关内容可参见 TRON 的
[相关文档](../../../../mechanism-algorithm/dex.md)。

## ExchangeCreate

创建交易对。

```console
> exchangeCreate [OwnerAddress] first_token_id first_token_balance second_token_id second_token_balance
```

- `OwnerAddress`（可选）——发起交易的账户地址。默认：登录账户的地址。
- `first_token_id`、`first_token_balance`——第一个 token 的 ID 和数量。
- `second_token_id`、`second_token_balance`——第二个 token 的 ID 和数量。ID 是已发行 TRC10 token
  的 ID。如果是 TRX，ID 为 `_`。数量必须大于 0，且小于 1,000,000,000,000,000。

示例：

```console
> exchangeCreate 1000001 10000 _ 10000
    # 创建 ID 为 1000001 与 TRX 的交易对，两者数量均为 10000。
```

## GetExchange

按 id 查询交易对（已确认状态）。

```console
> getExchange 1
```

## ExchangeInject

注资。进行注资时，会根据其数量（`quant`），按比例从账户中提取交易对中的每种 token 并注入交易对。
由于交易余额存在差异，同一 token 的相同金额也会有所不同。

```console
> exchangeInject [OwnerAddress] exchange_id token_id quant
```

- `OwnerAddress`（可选）——发起交易的账户地址。默认：登录账户的地址。
- `exchange_id`——要注资的交易对 ID。
- `token_id`、`quant`——注资的 tokenId 和数量（单位为基础单位）。

## ExchangeTransaction

```console
> exchangeTransaction [OwnerAddress] exchange_id token_id quant expected
```

- `OwnerAddress`（可选）——发起交易的账户地址。默认：登录账户的地址。
- `exchange_id`——交易对的 ID。
- `token_id`、`quant`——被兑换 token 的 ID 和数量，相当于卖出。
- `expected`——期望获得的另一种 token 的数量。`expected` 必须小于 `quant`，否则会报错。

示例：

```console
> ExchangeTransaction 1 1000001 100 80
```

期望在 ID 为 1 的交易对中，用数量为 100 的 1000001 兑换得到 80 TRX。（相当于在交易对 ID 为 1 中，
以 80 TRX 的价格卖出数量为 100 的 tokenID 1000001。）

## ExchangeWithdraw

撤资。进行撤资时，会根据其数量（`quant`），按比例从交易对中提取每种 token 并注入账户。由于交易
余额存在差异，同一 token 的相同金额也会有所不同。

```console
> exchangeWithdraw [OwnerAddress] exchange_id token_id quant
```

- `OwnerAddress`（可选）——发起交易的账户地址。默认：登录账户的地址。
- `Exchange_id`——要撤资的交易对 ID。
- `Token_id`、`quant`——撤资的 tokenId 和数量（单位为基础单位）。

## 获取交易对信息 {#obtain-information-on-trading-pairs}

- `ListExchanges`——列出交易对。
- `ListExchangesPaginated`——分页列出交易对。

## 另请参见

- [proposals](proposals.md)——链上治理提案
- [dex](dex.md)——TRON-DEX 订单市场
- [transfer-trc10](transfer-trc10.md)——发行被交易的 TRC10 资产
