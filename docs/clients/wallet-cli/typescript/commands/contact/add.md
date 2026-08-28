# wallet-cli contact add

向联系人簿中添加一个收款方。

## 用法

```
wallet-cli contact add <name> <address> [--note <text>]
```

## 说明

把一个收款方（名称 → 地址）保存到本地地址簿。之后凡是需要填收款方的地方都可以使用这个名称——[`tx send --to`](../tx/send.md) 和 [`gasfree transfer --to`](../gasfree/transfer.md)。地址校验和在本地验证，不访问节点。

名称必须为 1–64 个字符，且不能长得像一个地址（以免与直接写在 `--to` 里的地址混淆）。

## 选项

| 选项 | 说明 |
|---|---|
| `--note <text>` | 自由格式备注（例如“交易所充值地址”），最多 128 个字符 |

此外还有[全局选项](../index.md#global-options-every-command)。 `name` 和 `address` 是位置参数。

## 示例

```bash
wallet-cli contact add alice TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub --note "Alice mainnet"
```

```console
✅ Contact added
  Name     alice
  Address  TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub
  Note     Alice mainnet
```

```bash
wallet-cli contact add alice TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub --note "Alice mainnet" -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"contact.add","data":{"name":"alice","address":"TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub","note":"Alice mainnet","family":"tron"},"meta":{"durationMs":4,"warnings":[]}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `name` | string | 联系人名称 |
| `address` | string | 收款方地址 |
| `note` | string \| null | 备注，未设置时为 `null` |
| `family` | string | 链族（`tron`） |

## 退出码

`0` 成功 · `1` 执行失败（`already_exists`——名称已被占用、`limit_exceeded`——地址簿已满）。地址簿是一个本地文件：无法解码时为 `encoding_error`，若它是符号链接或对同组/其他用户可读则为 `insecure_permissions`（请 `chmod 600`）。 · `2` 用法错误（`invalid_value`——地址校验和错误，或名称不合法）。

## 另请参见

[`contact list`](list.md) · [`contact remove`](remove.md) · [`tx send`](../tx/send.md)
