# 为 wallet-cli 编写脚本

本页介绍如何从 shell 脚本和 CI 中调用 wallet-cli；完整的接口规范见
[machine-interface.md](../machine-interface.md)。

## 三项基本规则

**1. 始终添加 `-o json`。** text 输出面向人工阅读，格式可能变化；JSON 输出才是稳定的程序接口。
每次运行只会在 stdout 输出一个 JSON 对象：

```bash
wallet-cli account balance --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"account.balance","data":{"address":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ","balance":"1976489000","decimals":6,"symbol":"TRX"},"meta":{"durationMs":1114,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

**2. 先看退出码，再看 `error.code`。** `0` 成功，`1` 运行时失败，`2` 说明你把命令写错了：

```bash
if out=$(wallet-cli account balance --network tron:nile -o json); then
  bal=$(jq -r '.data.balance' <<<"$out")     # 原始 SUN，是*字符串*
else
  code=$(jq -r '.error.code' <<<"$out")      # e.g. timeout, rpc_error
fi
```

**3. 通过 stdin 传入敏感信息，不要放在 argv 中。** 参数中的密码、助记词或私钥可能出现在 shell 历史
和 `ps` 输出中：

```bash
printf '%s' "$PW" | wallet-cli tx send --to T... --amount 1 \
  --network tron:nile --password-stdin -o json
```

（shell 变量可以作为 stdin 的数据来源，但 `$PW` 应来自密钥管理服务或密码管理器，不应来自仓库中的
文件，也不建议长期 `export`。每次运行只能使用一个 `*-stdin` 标志。）

## 等待确认

`tx send` 在提交时就返回。对脚本来说，最简单且安全的写法是 `--wait`：

```bash
wallet-cli tx send --to T... --amount 1 --network tron:nile \
  --password-stdin --wait --wait-timeout 90000 -o json
```

也可以解耦：先取出 `data.txId`，再轮询 [`tx status`](../commands/tx/status.md) 直到 `data.state` 为 `confirmed`（遇到 `failed` 则中止）。完整的四状态轮询模式，以及批量操作的相关规则，见 [machine-interface.md → 脚本安全](../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)。

## 分离签名与广播 {#sign-here-broadcast-there}

`--sign-only` 和 `tx broadcast` 把签名与提交拆开，这样持有密钥的机器不必执行广播：

```bash
# 在持有密钥的机器上（它仍需联网来构建和估算）
out=$(printf '%s' "$PW" | wallet-cli tx send --to T... --amount 1 --network tron:nile \
        --password-stdin --sign-only -o json) || exit 1
printf '%s' "$out" | jq -ce '.data.signed' > signed.json || exit 1

# 在联网机器上广播
wallet-cli tx broadcast --tx-stdin --network tron:nile -o json < signed.json
```

`.data.signed` 是**已签名的交易 JSON 对象**，因此要配合接收 JSON 的 `--tx-stdin`；`--hex` /
`--file` 收的是交易 **hex**（即 `tx sign` 输出的那种形式），两者不能混用。

不要写成 `wallet-cli … | jq … > signed.json`：管道的退出码来自 `jq`，`wallet-cli` 的失败会被
吞掉，最终得到一个内容为 `null` 的文件。`jq -e` 会让缺失或为 `null` 的字段返回非零退出码。

> **`--sign-only` 不等于离线签名。** `tx send --sign-only` 在签名之前仍要构建并估算交易，两步
> 都会调用 RPC，因此这台机器**必须联网**。它省掉的是广播动作，不是网络依赖。
>
> 真正的离线签名是三段式：**联网机器 [`tx send --build-only`](../commands/tx/send.md) 产出未签名
> 交易 → 离线机器 [`tx sign`](../commands/tx/sign.md) 签名 →
> 联网机器 [`tx broadcast`](../commands/tx/broadcast.md) 广播**。

## 超时与重试

每次 RPC 或设备调用都受 `--timeout`（毫秒）限制。出现 `error.code = "timeout"` 时，读取类命令可以
适当增大超时时间后重试。对于 `tx send`，应先使用已有的 txid 查询 `tx status`；未确认状态就直接
重发可能造成重复付款。

## 另请参见

- [machine-interface](../machine-interface.md)——响应 schema、错误码、稳定性承诺
- [命令参考](../commands/index.md)——每条命令的 `data` 返回数据
