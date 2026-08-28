# wallet-cli message sign

对任意消息签名（TIP-191/V2 · EIP-191）。

## 用法

```
wallet-cli message sign (--message <text> | --message-stdin) [options]
```

## 说明

使用当前账户（或 `--account` 指定账户）的密钥，按 TRON 的 TIP-191/V2 个人消息方案（兼容 EIP-191）对消息签名。只签名——不会广播任何内容；`--network` 是可选的，省略后即可完全离线签名。

**stdin 只有一个消费者（fd 0）**：`--message-stdin` 和 `--password-stdin` 不能在同一次运行中同时使用（否则报 `secret_source_error`）。实际取舍如下：

- **Ledger 账户**在设备上确认，不需要 master password——fd 0 是空闲的，因此可以用 `--message-stdin` 通过管道传入消息。
- **软件账户**在非交互场景下需要 `--password-stdin`——此时消息只能通过 `--message` 直接写在命令行上。

仅观察账户无法签名（`watch_only_no_signer`）。

## 选项

| 选项 | 说明 |
|---|---|
| `--message <string>` | 要签名的消息文本；`--message` / `--message-stdin` 二者必居其一 |
| `--message-stdin` | 从 stdin（fd 0）读取消息 |
| `--password-stdin` | 从 stdin 读取 master password（软件账户） |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

示例中的 `$PW` 是你的 master password（来自环境变量、密码管理器等），通过 `--password-stdin` 从 stdin 传入。

软件账户——密码走 stdin，消息写在命令行上：

```bash
echo "$PW" | wallet-cli message sign --message "hello" --password-stdin
```

```console
✅ Signed
  Address    TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ
  Signature  0x9f3c...
```

```bash
echo "$PW" | wallet-cli message sign --message "hello" --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"message.sign","data":{"address":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ","message":"hello","signature":"0x9f3c..."},"meta":{"durationMs":15,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

Ledger 账户——消息走 stdin，在设备上确认：

```bash
cat challenge.txt | wallet-cli message sign --message-stdin --network tron:nile
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `address` | string | 签名者的 base58 地址 |
| `message` | string | 被签名的消息 |
| `signature` | string | 签名，带 `0x` 前缀的 hex |

## 退出码

`0` 已签名 · `1` 执行失败（`watch_only_no_signer`、`auth_failed`；同时使用两个 stdin 选项 → `secret_source_error`） · `2` 用法错误（同时给出或都不给出消息来源 → `invalid_option` / `missing_option`）。

## 另请参见

[安全模型](../../concepts/security.md) · [machine-interface → 敏感信息处理](../../machine-interface.md#secret-handling)
