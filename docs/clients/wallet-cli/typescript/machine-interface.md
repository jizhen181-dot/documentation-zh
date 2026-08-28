# 机器接口

本页定义脚本、CI 流水线和 AI 智能体调用 wallet-cli 时应遵循的接口规范，包括 JSON 响应结构、
退出码、错误码和敏感信息处理方式。除非另有说明，这些约定均受 `wallet-cli.result.v1` 的稳定性承诺保护。

## 调用约定 {#calling-convention}

```bash
wallet-cli <command> -o json [--network <id>] [--timeout <ms>] [--account <id|label>]
```

- 始终传 `-o json`。text 输出是给人看的，不带任何稳定性承诺。
- 在 JSON 模式下，stdout **只输出一个完整的 JSON 响应对象**，不会混入其他内容；诊断信息写入 stderr。
- 每次 RPC / 设备调用都受 `--timeout` 限制（毫秒，默认取 `config.timeoutMs`，内置默认值 60000）。

## 退出码 {#exit-codes}

| 退出码 | 含义 | JSON 响应 |
|---|---|---|
| `0` | 成功 | `success: true` |
| `1` | 执行失败——运行时错误：RPC 失败、超时、链上拒绝、钱包错误 | `success: false` |
| `2` | 用法错误——参数写错、缺少必填选项、取值非法、family 不匹配 | `success: false` |

退出码只有以上三种，含义固定。在 JSON 模式下，非零退出码总会同时在 stdout 输出错误响应。

## JSON 响应结构 {#the-result-envelope}

Schema id：`wallet-cli.result.v1`。

**成功：**

```json
{
  "schema": "wallet-cli.result.v1",
  "success": true,
  "command": "account.balance",
  "data": { "address": "TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ", "balance": "1976489000", "decimals": 6, "symbol": "TRX" },
  "meta": { "durationMs": 1114, "warnings": [] },
  "chain": { "family": "tron", "network": "tron:nile", "chainId": "nile" }
}
```

**出错：**

```json
{
  "schema": "wallet-cli.result.v1",
  "success": false,
  "command": "tx.info",
  "error": { "code": "rpc_error", "message": "TRON getTransaction failed: Transaction not found" },
  "meta": { "durationMs": 1033, "warnings": [] },
  "chain": { "family": "tron", "network": "tron:nile", "chainId": "nile" }
}
```

| 字段 | 类型 | 出现时机 | 说明 |
| ----------------- | ------------------------ | ------------------- | -------------------------------------------------------------------------------- |
| `schema`          | `"wallet-cli.result.v1"` | 始终 | schema 版本标识；调用方应据此选择解析逻辑 |
| `success`         | boolean | 始终 | 与退出码一致（`true` ⇔ 0） |
| `command`         | string | 始终 | 规范命令 id，例如 `tx.send`、`list`。它标识的是**操作**，而不是你输入的字面词：`backup --records` 报告为 `backup.records`，`import keystore` 报告为 `import.keystore` |
| `data`            | object/array | 仅成功时 | 命令返回的数据；见各命令的参考页 |
| `error.code`      | string | 仅出错时 | 机器可读；见[错误码](#error-codes) |
| `error.message`   | string | 仅出错时 | 面向人类可读；**不**稳定——绝不要解析它 |
| `error.details`   | object | 可选 | 可用时提供的结构化附加信息 |
| `meta.durationMs` | number | 始终 | 实际耗时（毫秒） |
| `meta.warnings`   | `(string \| {code, message})[]` | 始终 | 非致命提示；**元素类型并不统一**——见下文 |
| `meta.pagination` | object | 仅分页命令 | `offset` / `limit` / `total`；在支持 `--limit` / `--offset` 的地方出现——见[分页](#pagination) |
| `chain`           | object | 仅链上命令 | `family` / `network` / `chainId`；与链无关的命令（`list`、`config` 等）不带它 |

编码规则：`bigint` 值序列化为十进制**字符串**（例如 `"balance": "1976489000"`），二进制数据序列化为 hex。请把所有链上金额都当作字符串处理。

### 读取 `meta.warnings` {#reading-metawarnings}

每个条目可能是普通字符串，也可能是 `{code, message}` 对象。需要程序据此分支处理的警告使用对象形式——
目前包括 [`permission update`](commands/permission/update.md) 的安全警告和确认后的检查；其他警告使用
普通字符串。展示前请先统一处理两种形式，不要假设所有元素类型相同：

```bash
# 给人看的文本——两种形式都适用
jq -r '.meta.warnings[] | if type == "string" then . else .message end'

# 针对特定情况分支——仅对象形式适用
jq -e '.meta.warnings[] | select(type == "object" and .code == "owner_lockout")' >/dev/null && exit 1
```

假定元素是字符串的辅助写法（`.meta.warnings | join("\n")`、`Array.prototype.join`）在遇到对象形式时会报错或打印出 `[object Object]`。警告的 `code` 取值在 v1 内是稳定且只增的——可能出现新的 code，已有的 code 会保持原有含义。警告的 `message` 文本**不**稳定；请像对待 `error.message` 一样，绝不要解析它。

### 分页 {#pagination}

接受 `--limit` / `--offset` 的命令会在 `meta.pagination` 中报告当前分页信息，而不是放在 `data` 中：

| 键 | 类型 | 含义 |
|---|---|---|
| `offset` | number | 本页的起始索引——回显 `--offset` |
| `limit` | number \| **null** | 每页大小；`null` = 不限（未给 `--limit`） |
| `total` | number \| **null** | 匹配记录的总数；`null` 表示**不存在这个计数**，而不是"被省略了" |

这三个键始终存在，因此 `null` 是唯一的"未知"信号，也就不需要把"缺失"和 null 区分开。

对于由 TRON 分页节点端点提供的命令——[`asset list`](commands/asset/list.md) 和 [`exchange list`](commands/exchange/list.md)——`total` 永久为 `null`。这些端点不返回计数，而要算出一个计数就意味着传输全部记录（主网上有 5,187 个资产，2.7 MB）。请一直翻页直到返回一个不足整页的结果为止，而不要去和总数比较：

```bash
offset=0
while :; do
  page=$(wallet-cli asset list --limit 50 --offset "$offset" -o json)
  n=$(jq '.data.assets | length' <<<"$page")
  jq -c '.data.assets[]' <<<"$page"
  [ "$n" -lt 50 ] && break
  offset=$((offset + 50))
done
```

对本地有限数据集分页的命令（[`backup --records`](commands/backup.md)），以及先取回全部数据再在客户端分页的命令（[`proposal list`](commands/proposal/list.md)），会报告 `total`。

text 模式会为同一页数据添加标题（`Assets (limit 50, offset 0)`、`Proposals (showing 2 of 4)`、`Backup records (showing 3 of 12)`），但 text 输出不属于稳定接口，请使用并解析 `-o json`。

## 错误码 {#error-codes}

**退出码是稳定接口**：`2` 表示命令参数或调用方式有误（直接重试仍会失败），`1` 表示执行过程中发生
网络、设备、链或钱包错误。`error.code` 用于进一步区分具体原因；程序应先按退出码分类，再按需要处理
`error.code`。错误码并非封闭枚举，后续可能新增；少数字符串（如 `invalid_value`、`aborted`）也可能
出现在两种退出码下，具体取决于错误发生阶段。调用方应兼容未知错误码，并回退到对应的退出码类别。

退出码 **2**（用法——修正调用方式）下的常见错误码：

| 错误码 | 含义 |
|---|---|
| `usage_error` | 参数未知 / 缺失 / 冲突（由解析器抛出） |
| `missing_option` | 未提供某个必填参数 |
| `invalid_option` | 某个参数被用在了非法组合中 |
| `invalid_value` | 某个参数取值未通过校验（例如 `config defaultOutput xml`） |
| `invalid_amount` | 金额格式错误或超出范围 |
| `invalid_secret` | 提供的助记词 / 私钥格式错误 |
| `weak_password` | master password 未达策略要求（≥8 个字符；大写 + 小写 + 数字 + 特殊字符） |
| `tty_required` | 需要交互式提示，但没有挂载 TTY——请传对应的 `*-stdin` 标志 |
| `missing_network` / `unsupported_network` | 缺少 `--network`，或它不是已知的规范 id |
| `unknown_command` | 没有这条命令 |
| `output_exists` | 目标文件已存在且绝不会被覆盖（`backup --out`、`address generate --out`）。这是确定性的——用同一路径重试永远会失败 |
| `file_not_found` | 某个参数指定的输入文件不存在（`contract create2 --code-file`） |
| `keystore_not_found` | `import keystore`：给定路径上没有文件 |
| `invalid_keystore` | `import keystore`：不是合法的 Web3 V3 keystore——JSON 有误、`version` ≠ 3、使用了不支持的 cipher/KDF，或解密结果不是 32 字节私钥 |
| `invalid_config` | `config.yaml` 无法读取或不是合法的 YAML——请修复或删除该文件。解析器的具体细节被隐去：它会引用出错的那一行，而那行可能带有凭据 |
| `insecure_config` | `config.yaml` 保存了服务凭据，但它是符号链接或对同组/所有人可读——请对它执行 `chmod 600`（仅 POSIX；Windows 上不强制） |
| `token_not_in_book` / `token_is_official` / `token_metadata_unavailable` | token 地址簿相关的状况 |
| `unknown_parameter` | 不存在该名称或 id 的链参数（`proposal create --set`） |
| `invalid_asset_name` | TRC10 名称或缩写不在 1–32 个可见 ASCII 字符范围内 |

退出码 **1**（执行——运行时失败）下的常见错误码：

| 错误码 | 含义 |
|---|---|
| `rpc_error` | TRON 节点拒绝了请求或请求执行失败 |
| `invalid_node_response` | 节点的应答与请求或协议相矛盾：TRC10/exchange 记录的 id 并非所请求的那个、`precision` 超出 0..6，或汇率对不是正的 int32。这些值决定了签名金额，因此命令会直接停止而不是照单执行。列表读取则会丢弃有问题的记录并保留该页 |
| `timeout` | 等待网络或设备时被中止（超过 `--timeout`） |
| `auth_required` | 需要 master password 但未提供 |
| `auth_failed` | master password 错误（解密失败） |
| `signing_rejected` / `transaction_rejected` | 签名或广播被拒绝（设备或链） |
| `watch_only_no_signer` | 该账户是仅观察账户，无法签名 |
| `wrong_device_seed` | 连接的 Ledger 与已注册的账户不匹配 |
| `tx_integrity` / `invalid_transaction` | 预签名交易未通过完整性 / 合法性检查 |
| `insufficient_balance` / `insufficient_token_balance` | TRX / token 不足以覆盖金额加手续费 |
| `provider_error` | 外部服务（GasFree、TronLink 多签）返回错误或做了限流 |
| `gasfree_credentials_missing` / `tronlink_credentials_missing` | 未配置所需的服务凭据（用 `config` 设置） |
| `tx_expired` | 签名收齐之前交易就已过期 |
| `history_not_supported` | 该端点不支持 TronGrid 历史查询 |
| `not_found` | 所寻址的对象不存在——未激活的账户、联系人、链参数、GasFree 或 TronLink 资源。有专属分组的查询使用下面更具体的错误码 |
| `proposal_not_found` / `contract_not_found` / `asset_not_found` / `exchange_not_found` | 链上没有该提案 id、合约地址、TRC10 引用或交易对 id 对应的对象 |
| `ambiguous_asset_name` | 某个 TRC10 名称匹配到多个 token；`error.details` 中带有候选项——见 [`error.details.matches`](#errordetailsmatches) |
| `ledger_unsupported` | Ledger TRON app 无法为该合约类型签名——在触碰设备之前就会被拒绝（`asset` 写操作、`witness` 写操作） |
| `not_a_witness` / `already_witness` / `not_proposal_owner` | 治理身份不满足该操作的规则 |
| `already_approved` / `not_approved` / `proposal_expired` / `already_canceled` | 提案投票的状态条件 |
| `account_not_active` / `chain_parameter_unavailable` | `witness create`：账户未在链上激活，或节点没有返回 `getAccountUpgradeCost` |
| `not_contract_deployer` | 该账户并非那个合约的部署者 |
| `already_issued_asset` / `not_an_issuer` | 该账户已经发行过 TRC10，或者从未发行过 |
| `not_in_ico_window` / `self_participation` | TRC10 ICO 参与条件 |
| `no_frozen_supply` / `not_yet_unfreezable` | 没有冻结的部分，或还没有到期的部分（`asset unfreeze`） |
| `not_exchange_creator` / `token_not_in_exchange` / `exchange_closed` / `same_token` | 交易对的访问权限与状态条件 |
| `insufficient_reserve` | `exchange withdraw`：超过了交易对那一侧所持有的量 |
| `precision_loss` / `slippage_exceeded` / `exchange_trading_disabled` | 从一份很窄的白名单中命名的节点拒绝原因——储备比例无法整洁换算的金额、低于下限的回报，或者该网络根本不接受 Bancor 交易 |
| `not_exportable` | 该账户不持有可导出的密钥材料（仅观察或 Ledger）——`backup` |
| `account_exists` / `wrong_keystore_password` | `import keystore`：该地址已在钱包中，或文件自身的密码不对（区别于 `auth_failed`，后者指的是 master password）。`mac` 缺失或不是 hex 的文件属于 `invalid_keystore`，而不是密码错误——hex 不区分大小写 |
| `internal_error` | 预期之外的内部失败；消息刻意保持通用 |

预期之外的异常会先经过**脱敏处理**，再以 `internal_error` 和通用消息返回，避免第三方库回显的敏感信息
进入响应。以上列表仅列出当前具有代表性的错误，v1 版本中仍可能新增错误码。

### `error.details.matches` {#errordetailsmatches}

部分错误表示输入存在歧义，需要调用方从多个候选项中选择，而不是简单重试。这类错误会把候选项放在
`error.details.matches` 中，其值是字段结构一致的对象数组：

```json
{"code":"ambiguous_asset_name","message":"2 TRC10 tokens are named MyToken; re-run with the id","details":{"name":"MyToken","assetIds":["1000123","1000488"],"matches":[{"assetId":"1000123","issuerAddress":"TQkXm4vN...","totalSupply":"1000000000000000","precision":6},{"assetId":"1000488","issuerAddress":"TZx9kP2m...","totalSupply":"5000000000","precision":2}]}}
```

`matches` 是通用字段，并不限定于某个错误码。任何带有该字段的错误都应按相同方式处理。text 模式会在
stderr 的 `error [...]` 行下方以表格显示候选项。`matches` 中的数量保留最小单位，与对应成功响应的
表示方式一致；某行包含 `precision` 时，text 表格会换算为便于阅读的数值。

与它并列，错误还可能携带一个只包含标识符的标量列表，供你据此重试——就是上面的 `assetIds`。脚本请优先用它；`matches` 的存在是为了让人能分辨这些候选项。

## 敏感信息处理 {#secret-handling}

CLI 不从 argv 或专用环境变量读取密码、助记词和私钥等敏感信息，因为这些位置可能通过 shell 历史、
进程信息或 CI 日志造成泄露。敏感信息只能通过以下两种方式传入 CLI：

1. **stdin 标志**——`--password-stdin`、`--tx-stdin`、`--message-stdin`。**每次运行只能有一个 `*-stdin` 标志消费 stdin。**（助记词和私钥没有 stdin 路径——`import mnemonic` / `import private-key` / `change-password` 只能交互执行，使用隐藏的 TTY 输入。）
2. **交互式 TTY 提示**——在挂载了终端的情况下运行时。

```bash
# 非交互式解锁
printf '%s' "$MASTER_PASSWORD_FROM_YOUR_VAULT" | wallet-cli tx send \
  --to TSx72ViULFepRGCS4PM5dP4FqD1d8qggCc --amount 1 \
  --network tron:nile --password-stdin -o json
```

## 脚本安全：绝不要把"已提交"当作"已确认" {#script-safety-never-mistake-submitted-for-confirmed}

这是一个钱包；成功判断写错就意味着丢钱。规则如下：

1. 广播类（✍️）命令**默认在提交后就返回**，而不是等待确认。`data` 是一个扁平对象，其中包含表示操作类型的 `kind`（`send`、`stake-freeze`、`permission-update`、`account-activate`、`proposal-create`、`asset-issue`、`exchange-trade` 等）、`stage` 和 `txId`；`submitted` 阶段不包含区块、手续费或执行结果（这些字段只有在 `--wait` 确认后才会出现）：

   ```json
   { "kind": "send", "stage": "submitted", "txId": "7d9b6a08…", "rawAmount": "1000000", "to": "TSx72…" }
   ```

   **由链分配的 id 只有随确认才会出现。** 新提案的 `proposalId`、TRC10 的 `assetId`、交易对的 `exchangeId` 在提交时并不存在——它们不会出现在提交回执中，而是在 `--wait`（或之后的查询）在链上看到该交易之后才出现。创建这些对象的脚本必须等待。

2. 想阻塞到结果已知为止，请传 `--wait`（轮询直到 confirmed/failed，上限由 `--wait-timeout` 控制，默认 60000 毫秒；触及上限时返回提交回执）。

   **`--wait` 通过 `data.stage` 报告交易结果，而不是通过 `success`。** 如果交易已提交并入块，但链上执行失败，CLI 调用本身仍算成功：响应中的 `success` 为 `true`、退出码为 `0`，而 `data.stage` 为 `"failed"`。退出码表示 CLI 是否完成了请求，并不表示链上执行是否成功。因此使用 `--wait` 后，程序必须检查 `data.stage`（`confirmed` / `failed` / `submitted`），再决定如何记录本次操作。

3. 或者你自己用 `tx status` 轮询，它有一个**四状态模型**：

   | `data.state` | 含义 | 能否停止轮询？ |
   |---|---|---|
   | `confirmed` | 已入块并取得执行回执（带有 `blockNumber`） | 可以——视为成功 |
   | `failed` | 已入块，但链上执行失败 | 可以——视为失败 |
   | `pending` | 已被看到但尚未入块 | 不能——继续轮询 |
   | `not_found` | 被查询的节点不知道它 | 不能——超过截止时间后转为**状态未知**，而不是失败 |

   > **`confirmed` 表示已入块，不表示已固化。** 本命令与 `--wait` 读取的都是 FullNode 的
   > *未确认*视图：`confirmed` 说明交易已在区块中、回执已确定，**不代表已固化、不可回滚**。
   > 需要最终性时请另行查询 SolidityNode。

   > **`not_found` 超时不等于失败。** 它只说明*被查询的那个节点*不知道这笔交易，交易可能仍在
   > 其他节点的内存池里。切勿据此重发——那是重复付款的来源。超过截止时间后应视为**状态未知**，
   > 先通过 SolidityNode 或区块浏览器对账再决定。

   `data.confirmed` 和 `data.failed` 以布尔值提供，便于直接分支。

   **GasFree 转账是个例外。** `gasfree transfer` 提交给的是一个服务提供方，而不是直接提交给节点：提交回执中带的是 `traceId`（而不是 `txId`），进度遵循提供方的状态——`WAITING` → `INPROGRESS` → `CONFIRMING` → `SUCCEED` / `FAILED`。请用 `--wait` 或 [`gasfree trace <traceId>`](commands/gasfree/trace.md) 跟踪它，而不是 `tx status`；`txId` 只有在提供方把它送上链之后才会出现。

```bash
# 先保存完整输出和退出码再解析：管道的退出码来自 jq，而错误响应中的 .data.txId 求值为 null，
# jq 仍会以 0 退出，wallet-cli 的失败会被吞掉。
out=$(wallet-cli tx send --to T... --amount 1 --network tron:nile --password-stdin -o json < pw.fifo)
code=$?
[ "$code" -eq 0 ] || { printf '%s\n' "$out" | jq -r '.error.code' >&2; exit "$code"; }

txid=$(printf '%s' "$out" | jq -er '.data.txId') || exit 1

deadline=$(( $(date +%s) + 120 ))       # 自行设定截止时间
while :; do
  # 查询本身同样要先保存输出和退出码：写成管道会取到 jq 的退出码，
  # tx status 返回错误响应时 .data.state 求值为 null，会被当成"空状态"无限轮询。
  status_out=$(wallet-cli tx status --txid "$txid" --network tron:nile -o json)
  status_code=$?
  if [ "$status_code" -ne 0 ]; then
    printf '%s\n' "$status_out" | jq -r '.error.code' >&2
    exit "$status_code"                      # 查询失败，不要静默继续
  fi
  state=$(printf '%s' "$status_out" | jq -er '.data.state') || exit 1

  case "$state" in
    confirmed) echo "已入块: $txid"; break ;;
    failed)    echo "执行失败: $txid" >&2; exit 1 ;;   # 必须中止，不能继续循环
    *)         # pending / not_found
      if [ "$(date +%s)" -ge "$deadline" ]; then
        # 超时不代表失败：状态未知，对账后再决定，切勿盲目重发
        echo "状态未知，需人工对账: $txid" >&2; exit 2
      fi
      sleep 3 ;;
  esac
done
```

4. **批量操作**：每条命令对应一笔交易和一个退出码。默认的安全做法是遇到第一个失败就停止；如果选择继续，请逐项记录 txid，并在报告成功前用 `tx status` 逐一核对。

## 稳定性承诺（v1） {#stability-promise-v1}

只要 `schema` 是 `wallet-cli.result.v1`，以下内容保证稳定：

- 上表中的响应字段名称与语义；
- 0/1/2 的退出码映射；
- JSON 模式下 stdout 只输出一个完整 JSON 对象的约定；
- 已有的 `error.code` 取值保持其含义（可能新增 code）；
- 规范命令 id 和网络 id（`tron:mainnet`、`tron:nile`、`tron:shasta`）。

不在承诺范围内：text 模式输出、`error.message` 的措辞、字段顺序、`meta.durationMs` 的取值，以及任何在命令参考页上标记为尽力而为（best-effort）的字段（例如 `account portfolio` 中的 `priceUsd`）。

## 另请参见

- [脚本编写指南](guide/scripting.md)——更通俗的入门
- [命令参考](commands/index.md)——每条命令的 `data` 返回数据
- [故障排查](troubleshooting.md)——按上述错误码组织的、面向人的处理办法
