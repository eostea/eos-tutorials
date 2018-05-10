# EOSIO 3.0 使用指南

## 运行本地节点
### 方法1
```shell
nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::account_history_api_plugin
```
### 方法2
* step1:运行nodeos，节点会创建nodeos的config.ini 文件，文件地址可能在两个地方：
    1.```eos/build/etc/node_00/```
    2.```~/.local/share/eosio/nodeos/config/```
* step2: 更改config文件，然后运行nodeos，就可以正常运行。

## 创建钱包

创建钱包的操作跟2.0一样，这里就不再多说基本的步骤：
```shell
cleos wallet create -n wallet_name //创建钱包
cleos wallet import -n wallet_name key //导入私匙
cleos wallet open -n wallet_name //
cleos wallet unlock -n wallet_name --password password
cleso wallet lock_all
cleos wallet lock -n wallet_name
```

## 使用系统默认的eosio.bios合约
现在我们有一个带有eosio加载密钥的钱包，我们可以设置默认的系统合同。 为了开发目的，可以使用默认的eosio.bios合约。 此合约使您可以直接控制其他帐户的resoruce分配并访问其他特权api调用。 在公共区块链中，该合同将管理令牌的放样和取消操作，以预留cpu和网络活动的带宽以及合同的内存。

执行结果如下：
```shell
lome@lome:~/eos/build$ cleos set contract eosio contracts/eosio.bios -p eosio
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 0bb7ae021dffb931200c20ed6f4cb81fefd089bebbce7346cf41c86e554f57f6  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```
## 创建账户
创建账户的代码跟2.0也是一样的：
```shell
lome@lome:~/eos/build$ cleos create account eosio lome EOS5sU4bj1UhjNNLzP4CYmmoHPm9rpUkqX74ra5FiFwBGwznHt59P EOS5sU4bj1UhjNNLzP4CYmmoHPm9rpUkqX74ra5FiFwBGwznHt59P
executed transaction: 957e5d8ff2e9d98db6ba6e1efc6dc6e2e57e1123ff81f78cadf6bf9fa11ef4ea  364 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"lome","owner":{"threshold":1,"keys":[{"key":"EOS5sU4bj1UhjNNLzP4CYmmoHPm9...
```
## 通过key获取账户

1.在创建一个账户：
```shell
lome@lome:~/eos/build$ cleos create account eosio lome2 EOS5sU4bj1UhjNNLzP4CYmmoHPm9rpUkqX74ra5FiFwBGwznHt59P EOS5sU4bj1UhjNNLzP4CYmmoHPm9rpUkqX74ra5FiFwBGwznHt59P
executed transaction: deab457f2c4f4e98f7447fd87ccb92149766f809dcd3577525e03c87fa4d55c5  364 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"lome2","owner":{"threshold":1,"keys":[{"key":"EOS5sU4bj1UhjNNLzP4CYmmoHPm...
```
2. 获取使用该密匙的账户：
```shell
lome@lome:~/eos/build$ cleos get accounts EOS5sU4bj1UhjNNLzP4CYmmoHPm9rpUkqX74ra5FiFwBGwznHt59P
{
  "account_names": [
    "lome",
    "lome2"
  ]
}
```
## 创建一个token 合约
1. 创建账户
```shell
lome@lome:~/eos/build$ cleos create account eosio eosio.token EOS8mvgDwRv8kiX213BN38UeTE8ga2TckKyKUaAGzE3LguCKVETzu EOS8mvgDwRv8kiX213BN38UeTE8ga2TckKyKUaAGzE3LguCKVETzu
executed transaction: 2cd3d9266a5cf250c7d8900eacfa16c58791b5c442c305fe90d32743bfe11af2  364 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.token","owner":{"threshold":1,"keys":[{"key":"EOS8mvgDwRv8kiX213BN3...
```
2. 部署合约
```shell
lome@lome:~/eos/build$ cleos set contract eosio.token contracts/eosio.token -p eosio.token
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: f0f4a179c15f6655fd79a11b62c1a948c82ade13e1e44a5c35e5f79cf00af470  8708 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d0100000001ce011d60067f7e7f7f7f7f00...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...
```
## 创建EOS token
我们可以在eosio.token合约中看到,合约有三个方法如下所示:
```c++
void create( account_name issuer,
                      asset        maximum_supply,
                      uint8_t      issuer_can_freeze,
                      uint8_t      issuer_can_recall,  
                      uint8_t      issuer_can_whitelist );


         void issue( account_name to, asset quantity, string memo );

         void transfer( account_name from,
                        account_name to,
                        asset        quantity,
                        string       memo );
```
一个创建\一个发布\一个转账.

### 创建EOS token
```shell
lome@lome:~/eos/build$ cleos push action eosio.token create '["eosio","10000000000.0000 EOS",0,0,0]' -p eosio.token
executed transaction: a160153e605f047e67a02a330a564cdd1a5cdb53c0b4245a65c0145e80e0c19e  260 bytes  1000 cycles
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"10000000000.0000 EOS","can_freeze":0,"can_recall":0,"can_whiteli...
```
另外一种更详细的方式来调用这个方法，使用命名参数：
```shell
lome@lome:~/eos/build$ cleos push action eosio.token create '{"issuer":"eosio","maximum_supply":"10000000000.0000 EOS","can_freeze":0,"can_recall":0,"can_whitelist":0}' -p eosio.token
executed transaction: a160153e605f047e67a02a330a564cdd1a5cdb53c0b4245a65c0145e80e0c88e  260 bytes  1000 cycles
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"10000000000.0000 EOS","can_freeze":0,"can_recall":0,"can_whiteli...
```

## 发放token
```shell
lome@lome:~/eos/build$ cleos push action eosio.token issue '[ "lome", "100.0000 EOS", "memo" ]' -p eosio
executed transaction: fff713e1bebdd34b91e08de07e4df8e3060356ca79d902226b83f2fe9198d507  268 bytes  1000 cycles
#   eosio.token <= eosio.token::issue           {"to":"lome","quantity":"100.0000 EOS","memo":"memo"}
>> issue
#   eosio.token <= eosio.token::transfer        {"from":"eosio","to":"lome","quantity":"100.0000 EOS","memo":"memo"}
>> transfer
#         eosio <= eosio.token::transfer        {"from":"eosio","to":"lome","quantity":"100.0000 EOS","memo":"memo"}
#          lome <= eosio.token::transfer        {"from":"eosio","to":"lome","quantity":"100.0000 EOS","memo":"memo"}
```
从显示的日志可以看到具体的转账信息.

如果你想看到详情可以使用`-d -j`命令,他会以json的形式返回传账信息.

```shell
lome@lome:~/eos/build$ cleos push action eosio.token issue '[ "lome", "100.0000 EOS", "memo" ]' -p eosio -d -j
{
  "expiration": "2018-04-02T09:13:18",
  "region": 0,
  "ref_block_num": 15589,
  "ref_block_prefix": 3298773986,
  "net_usage_words": 21,
  "kcpu_usage": 1000,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio.token",
      "name": "issue",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000a0248d40420f000000000004454f5300000000046d656d6f"
    }
  ],
  "signatures": [
    "EOSK3xkbqHF8aUYSm7o8KeH5tMsibgFAChiBccDh9tsnvKMsn7XQ1hq3otGapebivptwuFJBEs5Cj79yVxLsTHtBsFyw54VJ6"
  ],
  "context_free_data": []
}
```
### 转账给用户
将账户余额转给其它用户:

```shell
lome@lome:~/eos/build$ cleos push action eosio.token transfer '[ "lome", "lome2", "25.0000 EOS", "m" ]' -p lome
executed transaction: b1295caa79423f35e6710bbffc5221ab0980b959aeff052592cb1c491b09e28a  268 bytes  1000 cycles
#   eosio.token <= eosio.token::transfer        {"from":"lome","to":"lome2","quantity":"25.0000 EOS","memo":"m"}
>> transfer
#          lome <= eosio.token::transfer        {"from":"lome","to":"lome2","quantity":"25.0000 EOS","memo":"m"}
#         lome2 <= eosio.token::transfer        {"from":"lome","to":"lome2","quantity":"25.0000 EOS","memo":"m"}
```
# 在EOS3.0发布代币成功
