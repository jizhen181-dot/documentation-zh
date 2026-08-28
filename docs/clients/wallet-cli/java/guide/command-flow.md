# 命令行操作流程

一个完整的端到端交互式会话示例：构建并运行、注册、备份、查看、发行资产并转移它。

```console
$ cd wallet-cli
$ ./gradlew build
$ ./gradlew run
> RegisterWallet 123456      (password = 123456)
> login 123456
> getAddress
address = TRfwwLDpr4excH4V4QzghLEsdYwkapTxnm'  # 请备份它！
> BackupWallet 123456
priKey = 1234567890123456789012345678901234567890123456789012345678901234  # 请备份它！！！（也可选用 BackupWallet2Base64）
> getbalance
Balance = 0
> AssetIssue TestTRX TRX 75000000000000000 1 1 2 "2019-10-02 15:10:00" "2020-07-11" "just for test121212" www.test.com 100 100000 10000 10 10000 1
> getaccount TRfwwLDpr4excH4V4QzghLEsdYwkapTxnm
(打印余额: 9999900000
"assetV2": [
    {
        "key": "1000001",
        "value": 74999999999980000
    }
],)
  # （assetIssue 消耗 1000 trx）
  # （你可以查询任意账户的 trx 余额和其他资产余额）
> TransferAsset TWzrEZYtwzkAxXJ8PatVrGuoSNsexejRiM 1000001 10000
```

## 另请参见

- [getting-started](getting-started.md)——更简短的快速上手
- [commands/transfer-trc10](../commands/transfer-trc10.md)——`AssetIssue` / `TransferAsset` 的细节
