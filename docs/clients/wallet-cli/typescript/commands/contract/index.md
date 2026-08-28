# wallet-cli contract

调用、发送、部署、查看并管理智能合约。

其中“管理”的部分属于部署者：一次调用的能量由谁支付，以及合约是否在链上保留 ABI。这些设置归属于部署该合约的账户，交易一确认便立即生效。`create2` 与这些无关——它是针对一个尚不存在的地址做的本地运算。

## 用法

```
wallet-cli contract COMMAND
```

## 子命令

| 命令 | 页面 | 说明 |
|---|---|---|
| `contract call` | [call.md](call.md) | 只读调用（triggerConstantContract） |
| `contract send` | [send.md](send.md) | 改变链上状态的调用（triggerSmartContract） |
| `contract deploy` | [deploy.md](deploy.md) | 部署智能合约 |
| `contract info` | [info.md](info.md) | 查看合约 ABI 与元数据 |
| `contract clear-abi` | [clear-abi.md](clear-abi.md) | 清除链上 ABI（不可逆） |
| `contract set-origin-energy-limit` | [set-origin-energy-limit.md](set-origin-energy-limit.md) | 部署者为每次调用承担的能量 |
| `contract set-user-resource-percent` | [set-user-resource-percent.md](set-user-resource-percent.md) | 调用方承担的能量比例 |
| `contract create2` | [create2.md](create2.md) | 本地计算 CREATE2 地址 |

## 另请参见

[能量与带宽](../../concepts/energy-bandwidth.md) · [`tx status`](../tx/status.md)
