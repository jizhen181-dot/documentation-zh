# wallet-cli typed-data sign

签名 EIP-712 / TIP-712 结构化数据。

## 用法

```
wallet-cli typed-data sign --typed-data <json> [options]
```

## 说明

使用当前账户（或 `--account` 指定账户）的密钥对结构化（typed）数据签名，并打印签名、被签名的 digest 以及 primary type。只签名——不会广播任何内容；`--network` 是可选的，省略后即可完全离线签名。

`--typed-data` 的取值是形如 `{"domain":…,"types":…,"primaryType"?:…,"message":…}` 的 EIP-712 / TIP-712 JSON。解析时有三点便利处理：

- `types` 中的 `EIP712Domain` 会被**忽略**（写不写都可以）。
- `value` 可作为 `message` 的别名。
- `address` 字段中可以使用 TRON 的 **base58** 地址。

`primaryType` 是可选的——省略时会自动推断。

与 [`message sign`](../message/sign.md) 不同，本命令**绝不会**交互式提示输入 master password：软件账户必须使用 `--password-stdin`。仅观察账户无法签名（`watch_only_no_signer`）。

## 选项

| 选项 | 说明 |
|---|---|
| `--typed-data <json>` | **必填。** EIP-712/TIP-712 JSON: `{"domain":…,"types":…,"primaryType"?:…,"message":…}` |
| `--password-stdin` | 从 stdin 读取 master password（软件账户） |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

示例中的 `$PW` 是你的 master password（来自环境变量、密码管理器等），通过 `--password-stdin` 从 stdin 传入。

```bash
echo "$PW" | wallet-cli typed-data sign --typed-data "$(cat permit.json)" --password-stdin --network tron:nile
```

```console
✅ Signed typed data
  Address    TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ
  Type       Permit
  Digest     0x1e0f...
  Signature  0x9f3c...
```

```bash
echo "$PW" | wallet-cli typed-data sign --typed-data "$(cat permit.json)" --password-stdin --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"typed-data.sign","data":{"address":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ","primaryType":"Permit","digest":"0x1e0f...","signature":"0x9f3c..."},"meta":{"durationMs":15,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

Ledger 账户——在设备上确认，不需要 master password：

```bash
wallet-cli typed-data sign --typed-data "$(cat permit.json)" --network tron:nile
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `address` | string | 签名者的 base58 地址 |
| `primaryType` | string | 被签名的 primary type |
| `digest` | string | 被签名的 digest，带 `0x` 前缀的 hex |
| `signature` | string | 签名，带 `0x` 前缀的 hex |

## 退出码

`0` 已签名 · `1` 执行失败（`watch_only_no_signer`、`auth_failed`、`signing_rejected`——在 Ledger 上被拒绝、`ledger_setting_required`——需在设备应用中开启盲签 / EIP-712 支持、`ledger_unsupported`——设备应用无法签名此负载） · `2` 用法错误（缺少 `--typed-data` → `missing_option`，或其取值不是 JSON → `invalid_value`）。

## 另请参见

[`message sign`](../message/sign.md)——签名纯文本消息 · [安全模型](../../concepts/security.md) · [machine-interface → 敏感信息处理](../../machine-interface.md#secret-handling)
