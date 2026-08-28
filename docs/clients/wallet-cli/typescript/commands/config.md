# wallet-cli config

查看 / 读取 / 设置配置项。

## 用法

```
wallet-cli config [<key>] [<value>] [options]
```

## 参数

- `key`——要读取或设置的配置键；省略则显示完整的生效配置
- `value`——新值；省略则表示读取该键

## 选项

仅[全局选项](index.md)。

## 注意事项

已知的配置键：

| 键 | 取值 | 内置默认值 | 含义 |
|---|---|---|---|
| `defaultNetwork` | 网络 id | `tron:mainnet` | 省略 `--network` 时使用的网络 |
| `defaultOutput` | `text` \| `json` | `text` | 省略 `-o` 时使用的输出格式 |
| `timeoutMs` | 整数毫秒 | `60000` | 单次 RPC/设备调用的默认超时（`--timeout` 可覆盖） |
| `waitTimeoutMs` | 整数毫秒 ≥ 0 | `60000` | 广播类命令 `--wait` 轮询的默认时间上限 |
| `gasfreeApiKey` | string | （未设置） | GasFree API key（[`gasfree`](gasfree/index.md)） |
| `gasfreeApiSecret` | string | （未设置） | GasFree API secret |
| `tronlinkSecretId` | string | （未设置） | TronLink 多签服务的 secretId（[`tx multisig`](tx/multisig.md)） |
| `tronlinkSecretKey` | string | （未设置） | TronLink 多签服务的 secretKey |
| `tronlinkChannel` | string | （未设置） | TronLink 多签服务的 channel |
| `networks` | —— | —— | 已知网络（只读列表） |

同时存在命令行选项和配置键时的优先级（从高到低）：命令行选项 > 配置值 > 内置默认值——例如 `--timeout` > 配置 `timeoutMs` > 内置的 60000。

外部服务的凭据是**按环境区分的**：GasFree（`gasfreeApiKey` / `gasfreeApiSecret`）和 TronLink（`tronlinkSecretId` / `tronlinkSecretKey` / `tronlinkChannel`）的凭据必须与当前 `--network` 所属的服务环境（主网 vs 测试网）匹配；不匹配会以 `provider_error` 失败，所以切换环境时记得同步更换。某个键未设置时，需要它的命令会给出明确的错误——[`gasfree`](gasfree/index.md) 报 `gasfree_credentials_missing`，[`tx multisig`](tx/multisig.md) 报 `tronlink_credentials_missing`。

取值非法时返回 `invalid_value`（退出码 2）。

## 示例

查看完整的生效配置：

```bash
wallet-cli config
```

```console
defaultNetwork     tron:mainnet
defaultOutput      text
timeoutMs          60000
waitTimeoutMs      60000
gasfreeApiKey      ak_9f2c71d0e8b64a53
gasfreeApiSecret   sk_e37a90c412f85b6d
tronlinkSecretId   TEST
tronlinkSecretKey  TESTTESTTEST
tronlinkChannel    test
networks           tron:mainnet, tron:nile, tron:shasta
```

先读取某个键，再设置它：

```bash
wallet-cli config timeoutMs
```

```console
timeoutMs  60000
```

```bash
wallet-cli config timeoutMs 120000
```

```console
✅ Set config
  Key    timeoutMs
  Value  120000
```

```bash
wallet-cli config timeoutMs 120000 -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"config","data":{"key":"timeoutMs","value":120000,"input":"120000"},"meta":{"durationMs":3,"warnings":[]}}
```

## 输出

`data` 随使用模式而异。本地命令——没有 `chain` 块。

| 模式 | `data` 字段 |
|---|---|
| 全部显示（无参数） | 每个配置键一个字段：`defaultNetwork`、`defaultOutput`、`timeoutMs`、`waitTimeoutMs`、`gasfreeApiKey`、`gasfreeApiSecret`、`tronlinkSecretId`、`tronlinkSecretKey`、`tronlinkChannel`、`networks`（网络 id 数组） |
| 读取（`<key>`） | `key`、`value` |
| 设置（`<key> <value>`） | `key`、`value`、`input`（用户输入的原始字符串） |

## 退出码

`0` 成功 · `1` 执行失败（`invalid_config`——`config.yaml` 不可读或不是合法 YAML；`insecure_config`——文件中存有服务凭据，却是符号链接或对同组/其他用户可读，请对其执行 `chmod 600`）· `2` 用法错误（`invalid_value`——未知的键）。参见 [machine-interface](../machine-interface.md)。

## 另请参见

[网络](../concepts/networks.md) · [machine-interface](../machine-interface.md)
