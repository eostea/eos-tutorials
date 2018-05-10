#　EOSIO 转帐详解

> 前言：最近有许多小伙伴问关于转账的一些操作，笔者在这里写一个教程进行详细说明。

## EOS和EOS的不同之处

在EOS网络中存在两种货币，一种是EOS，还有一种是EOS网络中的代币。说到这里大家似乎有点疑惑，举个简单的例子，就好比ETH网络中的ETH，ETH网络中的其他代币。这样大家或许都清除了吧。

在目前EOS网络中可以通过合约`eosio.token`产生多种名称为`EOS`的代币。但是还有一种通过合约`eosio.system`合约发布的代币，它是`EOS`网络中真正的EOS,他会存储在用户的账户中。可以通过`cleos transfer`来交易。


## 通过`eosio.token`发布的`EOS`代币
1. 发布`bios`合约

```
cleos set contract eosio eosio.bios/
```
2. 创建账户

```
cleos create account eosio eostea EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
>result
executed transaction: c10ba7625be38c823426ac9c974a7c3a774594ea80f600d95c88dc9d1053a3c6  352 bytes  102400 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eostea","owner":{"threshold":1,"keys":[{"key":"EOS6MRyAjQq8ud7hVNYcfnVPJq...

```
3. `eostea`发布代币EOS
发布合约：

```
> cleos set contract eostea eosio.token/
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 7908bd47ae2c68ffa8f0f51bc2401e9deda2e06e16fc60356afa27f316ef529f  8032 bytes  2200576 cycles
#         eosio <= eosio::setcode               {"account":"eostea","vmtype":0,"vmversion":0,"code":"0061736d01000000018a011660067f7e7f7f7f7f0060057...
#         eosio <= eosio::setabi                {"account":"eostea","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name":"fro...
```
创建代币：

```
>cleos push action eostea create '["eostea","10000000 EOS",0,0,0]' -p eostea
executed transaction: ae707244932ccd9c3b5a579d1e3875de6c0188d2024447c90df9ad716ad5ac41  248 bytes  104448 cycles
#        eostea <= eostea::create               {"issuer":"eostea","maximum_supply":"10000000 EOS","can_freeze":0,"can_recall":0,"can_whitelist":0}
```
发布代币：

```
$ cleos push action eostea issue '["eostea","100000 EOS","issue"]' -p eostea
executed transaction: 8e37f71b607d4ec0fd9ef7582b296e0b738a13948fe9cc82090cb96c1db8054e  256 bytes  107520 cycles
#        eostea <= eostea::issue                {"to":"eostea","quantity":"100000 EOS","memo":"issue"}
>> issue
```
查看代币：

```
$cleos get currency balance eostea eostea
10000 TEA
100000 EOS
```

下面笔者来做一个实验，用eosio再创建一种`EOS`代币,得到的结果：

```
cleos get currency balance eosio eosio
10000 EOS
```

给eostea转账：
```
cleos push action eosio transfer '["eosio","eostea","100 EOS",""]' -p eosio
```
查看eostea的代币：
```
lome@lome:~/eos/build/contracts$ cleos get currency balance eostea eostea
10000 TEA
100000 EOS
lome@lome:~/eos/build/contracts$ cleos get currency balance eosio eostea
100 EOS
```
这样我发型了两种`EOS`代币。但是.....
看看数据库里面是这样的：
```
{
    "_id" : ObjectId("5af41c653c27103f203a6beb"),
    "name" : "eostea",
    "eos_balance" : "0.0000 EOS",
    "staked_balance" : "0.0000 EOS",
    "unstaking_balance" : "0.0000 EOS",
    "createdAt" : "2018-05-10T10:18:13.008Z",
    "updatedAt" : "2018-05-10T10:19:34.007Z",
    "abi" : {
        ......
    }
}
```
eos_balance是０；

## 发型`EOS`

```
$ cleos push action eosio issue '["eosio","10000000.0000  EOS",""]' -p eosio
executed transaction: ad53e2b11f1b90f8cb3c5edff982fde7e87f4011773e179ca5a963df14c7227c  248 bytes  120832 cycles
#         eosio <= eosio::issue                 {"to":"eosio","quantity":"10000000.0000 EOS"}
#         eosio <= eosio::transfer              {"from":"eosio","to":"eosio","quantity":"10000000.0000 EOS","memo":""}
```
然后查看账户`eosio`:
```
{
    "_id" : ObjectId("5af41b903c27103f203a6392"),
    "name" : "eosio",
    "eos_balance" : "10000000.0000 EOS",
    "staked_balance" : "0.0000 EOS",
    "unstaking_balance" : "0.0000 EOS",
    "createdAt" : "2018-05-10T10:14:40.258Z",
    "updatedAt" : "2018-05-10T11:44:28.506Z",
    "abi" : {
        ........
    }
}
```
转账：
```
cleos transfer eosio eostea 1000000
executed transaction: 76a8b7f4ab67d8205661c0848d6dd4566830e84ca2b86a2ae44cef58c6cea4e1  256 bytes  107520 cycles
#         eosio <= eosio::transfer              {"from":"eosio","to":"eostea","quantity":"100.0000 EOS","memo":""}
#        eostea <= eosio::transfer              {"from":"eosio","to":"eostea","quantity":"100.0000 EOS","memo":""}
```
账户余额：

```
{
    "_id" : ObjectId("5af4336c3c27100f643add5f"),
    "name" : "eostea",
    "eos_balance" : "100.0000 EOS",
    "staked_balance" : "0.0000 EOS",
    "unstaking_balance" : "0.0000 EOS",
    "createdAt" : "2018-05-10T11:56:28.501Z",
    "updatedAt" : "2018-05-10T12:05:38.507Z",
    "abi" : {
        ........
    }
}
```
相信看完这些，大家都非常清楚了。
