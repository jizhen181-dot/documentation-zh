# wallet-cli token

管理 token 地址簿并查询 token。

## 用法

```
wallet-cli token COMMAND
```

## 子命令

| 命令 | 页面 | 说明 |
|---|---|---|
| `token balance` | [balance.md](balance.md) | 查询单个 token 的余额（--contract / --asset-id） |
| `token info` | [info.md](info.md) | 查看 token 元数据（name/symbol/decimals/totalSupply） |
| `token add` | [add.md](add.md) | 向地址簿中添加 token（自动读取 symbol/decimals） |
| `token list` | [list.md](list.md) | 列出地址簿（official + user） |
| `token remove` | [remove.md](remove.md) | 删除一条自行添加的 token |

## 另请参见

[发送 token](../../guide/send-tokens.md) · [`tx send`](../tx/send.md)
