# EOS3.0测试网络组建-EOS中文社区特辑
> 声明：本文由EOS中文社区原创首发，转载请标明原文出处．

## 摘要
EOS 3.0已经发布，EOS中文社区决定组建一个测试网络用于EOS3.0的开发．这篇文章主要讲解EOS测试网络组建的过程，其中包括EOS的节点配置，运行，组网等，关于EOS3.0的安装，EOS中文社区(eosfans.io)会上传国内Docker镜像，及相关教程，本文不再赘述．

## EOS3.0配置

### config.ini的设置

由于整个网络的运行需要一个节点先产生创世区块，进行区块链网络的初始化。所以该节点在链静默条件下需要能生产区块, 所以要将`enable-stale-production`配置项设置为`true`.

EOS区块链的创始账号是eosio，由于`eosio`的密匙，已经公布出来，所以在开启网络之前我们要修改eosio的密匙对。

密匙对可以在本地生成，也可以使用[生成工具](https://eosfans.io/tools/generate/)生成。把密匙对改为所生成的密匙对。

例如你生成的密匙对为：
```
私钥：xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
公钥：EOS5tfo9VMQcpjK3BWwzKQDP9Z4Xb3G5SWQVBZu2yP6e4hpH2A9iw
```
将密匙加入配置文件：

```
private-key = ["公钥","私钥"]
```
其他的配置如下：
```
get-transactions-time-limit = 3
genesis-json = “genesis.json”
block-log-dir = "blocks"
shared-file-dir = "blockchain"
shared-file-size = 8192
http-server-address = 0.0.0.0:8888
access-control-allow-credentials = false
# 这里填写你自己的服务器地址和监听的端口, 一般是 IP:9876
p2p-server-address =
agent-name = ""
send-whole-blocks = 1
allowed-connection = any
log-level-net-plugin = info
max-clients = 25
connection-cleanup-period = 30
network-version-match = 0
enable-stale-production = true
required-participation = 33
# 这里填写一对密钥对, 建议用全新的: https://eosfans.io/tools/generate/
private-key = ["EOS5tfo9VMQcpjK3BWwzKQDP9Z4Xb3G5SWQVBZu2yP6e4hpH2A9iw","xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"]
plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::wallet_api_plugin
plugin = eosio::account_history_api_plugin
plugin = eosio::http_plugin
# 这里填写节点用户名
producer-name =
```
### genesis.json配置
genesis.json的配置如下：

```
{
  "initial_timestamp": "2018-04-06T12:00:00.000",
  "initial_key": "EOS5tfo9VMQcpjK3BWwzKQDP9Z4Xb3G5SWQVBZu2yP6e4hpH2A9iw",
  "initial_configuration": {
    "base_per_transaction_net_usage": 100,
    "base_per_transaction_cpu_usage": 500,
    "base_per_action_cpu_usage": 1000,
    "base_setcode_cpu_usage": 2097152,
    "per_signature_cpu_usage": 100000,
    "per_lock_net_usage": 32,
    "context_free_discount_cpu_usage_num": 20,
    "context_free_discount_cpu_usage_den": 100,
    "max_transaction_cpu_usage": 10485760,
    "max_transaction_net_usage": 104857,
    "max_block_cpu_usage": 104857600,
    "target_block_cpu_usage_pct": 1000,
    "max_block_net_usage": 1048576,
    "target_block_net_usage_pct": 1000,
    "max_transaction_lifetime": 3600,
    "max_transaction_exec_time": 0,
    "max_authority_depth": 6,
    "max_inline_depth": 4,
    "max_inline_action_size": 4096,
    "max_generated_transaction_count": 16
  },
  "initial_chain_id": "0000000000000000000000000000000000000000000000000000000000000000"
}
```
#### 注意：genesis.json文件的`initial_key`必须和config.ini文件中的eosio的公匙一样，不然会导致网络运行不起来。

#### 到此准备工作已经完成。

> 接下来把网络运行起来，我们来进行下一步的设置。

## 测试网络的初始化

### 创建钱包导入密匙
首先我们需要创建一个钱包来存eosio账户的密匙。　　　
```
cleos wallet create -n eosfans
```
执行结果会返回钱包的密匙如下：
```
Creating wallet: eosfans
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5KJMZ8mneheKXuneuVHL6YWxsipsqmr5yPvmm4nFG5N5bQRvswN"
```
这里需要保存好钱包的密匙，然后把eosio的密匙导入钱包：
```
cleos wallet import -n eosfans xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

###　发布eosio.bios智能合约

发布智能合约时请保持钱包在打开状态，如果钱包锁了可以用，需先打开钱包，命令如下：
```
cleos wallet open -n eosfans
cleos wallet unlock -n eosfans
Password:这里输入密码
```

接下来我们就可以发布智能合约，命令如下：
```
cleos set contract eosio contracts/eosio.bios
```
此时，智能合约发布完成，可以用eosio主账号来创建新的账号。

### 发行代币

1. 创建eosio.token账号，命令如下：

```
cleos create account eosio eosio.token public_key public_key
```
2. 将eosio.token的私匙导入钱包。
3. 发布代币合约：
```
cleos set contract eosio.token contracts/eosio.token
```
结果如下：
```
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 924fc886136a604f36e726e36cea4324e4949c457077181d30667af5e5a696fd  8024 bytes  2200576 cycles
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d01000000018a011660067f7e7f7f7f7f0060057f7...
#         eosio <= eosio::setabi                {"account":"test","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name":"from"...
```
4. 创建代币：

```
cleos push action eosio.token create '["eosio", "10000000000.0000 EOS",0,0,0]' -p eosio.token
```

出现以下结果，就说明代币成功了：
```
executed transaction: 0a00766ada9f0dee409defce661ad70d9beb2d78ca2f745c7c7ab0efec6649fc  248 bytes  104448 cycles
#          eosio <= eosio.token::create                 {"issuer":"eosio","maximum_supply":"10000000000.0000 EOS","can_freeze":0,"can_recall":0,"can_whitelis...
```
5. 发布代币：

```
cleos push action eosio.token issue '["eosio","1000000000.0000 EOS", "issue"]' -p eosio
```
此时在网络中已经存在了十亿个EOS代币。

### 设置BP

1. 创建BP（Block Producer）账号

创建BP账号，账号是用来设BP的，创建账号这里就不在赘述。（BP：区块生产者）

2. 设置BP

设置BP的命令如下：
```
cleos push action eosio setprods '{"version":6, "producers":[{"producer_name":"eostea","block_signing_key":"EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV"}]}' -p eosio
```
结果如下：
```
executed transaction: 9b05ff1d246d44cc6c3757f16de093bf2346531d04314d0378cbd2c155d94cae  568 bytes  117760 cycles
#         eosio <= eosio::setprods              {"version":6,"producers":[{"producer_name":"eostea2","block_signing_key":"EOS54stgvUjdSwn8jpLiW1AucR...
```

### 给BP发放代币
```
cleos push action eosio transfer '["eostea","100000.0000 EOS","mome"]' -p eosio 
```
#### 查看账户余额
```
cleos get currency balance eosio.token eostea 
```
返回结果：
```
100000.0000 EOS
```
至此EOS测试网络的设置已经完毕，想要了解给更多BP相关的部署和设置等内容请到[EOS中文社区wiki](https://eosfans.io/wiki/eos-party-testnet)查看。
