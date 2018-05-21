# EOSIO 4.0测试网络启动教程
> EOS4.0在昨天已经发布，本片文章将介绍EOS4.0网络的搭建过程。

与EOS3.0相比，EOS4.0无疑是具有重要意义的预发布版。其中对`eosio.system`合约的更改相当大。下面就来说一下搭建网络的具体步骤。

## 节点简介
首先说明一下，搭建测试网络的节点信息，以免后文更加清晰的讲述整个过程。

1. node1(启动节点ip=111.11.11.111):
```
producer-name = eosio
http-server-address = 0.0.0.0:8888
p2p-listen-endpoint = 0.0.0.0:9876
p2p-peer-address = 222.22.22.222:9876
p2p-peer-address = 333.33.33.333:9876
enable-stale-production = true
```
2. node2(启动节点:ip=222.22.22.222):
```
producer-name = eostea
http-server-address = 0.0.0.0:8888
p2p-listen-endpoint = 0.0.0.0:9876
p2p-peer-address = 111.11.11.111:9876
p2p-peer-address = 333.33.33.333:9876
enable-stale-production = false
```
2. node3(启动节点:ip=333.33.33.333):
```
producer-name = eostea1
http-server-address = 0.0.0.0:8888
p2p-listen-endpoint = 0.0.0.0:9876
p2p-peer-address = 111.11.11.111:9876
p2p-peer-address = 222.22.22.222:9876
enable-stale-production = false
```
账号：lome, strahe, chare.

## 创建token
由于`eosio.system`已经没有发布代币的操作了，所有创建和发放代币只能通过`eosio.token`合约。

1. 创建`eosio.token`账号
```
cleos  create account eosio eosio.token EOS67Cjewqdy9kiS3uAy8RRxuYFzc4BuhMZk1crw69tdH2cVQSMw5 EOS67Cjewqdy9kiS3uAy8RRxuYFzc4BuhMZk1crw69tdH2cVQSMw5
```
输出结果如下：
```
executed transaction: 4a96ee307283ca589fa9640f6e059b47bb25fae61c5b0cdb6a8810fa5a231131  288 bytes  218 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.token","owner":{"threshold":1,"keys":[{"key":"EOS67Cjewqdy9kiS3uAy8...
warning: transaction executed locally, but may not be confirmed by the network yet
```
2. 发布`eosio.token`合约
```
cleos  set contract eosio.token eosio.token
```
结果如下：
```
Reading WAST/WASM from eosio.token/eosio.token.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 04e7ef9d7434c9726b01ed964dfffc0d908f040993cb67c5a0138437e8f18e7f  8848 bytes  1421 us
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d010000000182011560067f7e7f7f7f7f00...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...
warning: transaction executed locally, but may not be confirmed by the network yet
```
3. 创建`EOS`代币
```
cleos  push action eosio.token create '["eosio","1000000000.0000 EOS",0,0,0]' -p eosio.token
```
结果如下：
```
executed transaction: 49fda5d0a205d867c5f92dfe1ca272bc399957c50e34340b1526fa2016da693f  208 bytes  4586 us
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 EOS","can_freeze":0,"can_recall":0,"can_whitelis...
warning: transaction executed locally, but may not be confirmed by the network yet
```

4. 发布代币
```
cleos  push action eosio.token issue '["eosio","1000000000.0000 EOS","issue"]' -p eosio
```
结果如下：
```
executed transaction: 33e6433957063ccea8c0afeb26c101b3064079622416b7e471e1e4a91d962124  216 bytes  632 us
#   eosio.token <= eosio.token::issue           {"to":"eosio","quantity":"1000000000.0000 EOS","memo":"issue"}
>> issueeosio balance: 1000000000.0000 EOS
warning: transaction executed locally, but may not be confirmed by the network yet
```

## 发布`eosio.system`合约
> 注意：　    
    > 1. 必须创建代币后才能发布合约
    > 2. 运行节点的主机速度必须块否则，发布合约可能会失败最后内存在16G以上。              


```
cleos  set contract eosio eosio.system
```
运行结果：
```
Reading WAST/WASM from eosio.system/eosio.system.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: ff766928a0488566628c2ff37260937ab7b6ff2c235d65e844606754d11284a8  40368 bytes  5581 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001c1023060027f7e0060067f7e7e7f7f...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"buyrambytes","base":"","fields":[{"name":"p...
warning: transaction executed locally, but may not be confirmed by the network yet
```
## 创建账号
> 由于用`cleos create account`会出现以下错误：
```
$ cleos create account eosio lome2 EOS67Cjewqdy9kiS3uAy8RRxuYFzc4BuhMZk1crw69tdH2cVQSMw5 EOS67Cjewqdy9kiS3uAy8RRxuYFzc4BuhMZk1crw69tdH2cVQSMw5
Error 3080001: account using more than allotted RAM usage
Error Details:
account lome2 has insufficient ram bytes; needs 2996 has 0
```
错误原因是账户的ram不足，至于解决办法，笔者没有深究。所以直接用`eosio.system`中的`newaccount`操作创建账户。

创建账户代码如下：
```
cleos  system newaccount eosio eostea EOS8Ar1fUGtZxcJ8Rdkh3rc55Vqg3ariR6fdBV8zxz4WgTW3nT73L EOS8Ar1fUGtZxcJ8Rdkh3rc55Vqg3ariR6fdBV8zxz4WgTW3nT73L --stake-net '50.00 EOS' --stake-cpu '50.00 EOS'
```
执行结果如下：
```
1170388ms thread-0   main.cpp:418                  create_action        ] result: {"binargs":"0000000000ea3055000000001895315500200000"} arg: {"code":"eosio","action":"buyrambytes","args":{"payer":"eosio","receiver":"eostea","bytes":8192}}
1171112ms thread-0   main.cpp:418                  create_action        ] result: {"binargs":"0000000000ea3055000000001895315580f0fa020000000004454f530000000080f0fa020000000004454f530000000000"} arg: {"code":"eosio","action":"delegatebw","args":{"from":"eosio","receiver":"eostea","stake_net_quantity":"50.0000 EOS","stake_cpu_quantity":"50.0000 EOS","transfer":false}}
executed transaction: 8eae5e1e2d3ce126920fe2aa6e44376eb7c901a34dc820b0b95b0f33f8618375  424 bytes  2472 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eostea","owner":{"threshold":1,"keys":[{"key":"EOS8Ar1fUGtZxcJ8Rdkh3rc55V...
>> eosio created eostea
#         eosio <= eosio::buyrambytes           {"payer":"eosio","receiver":"eostea","bytes":8192}
>> eosout: 0.1192 EOS 68719484928. RAM 999999.8808 EOS
#         eosio <= eosio::delegatebw            {"from":"eosio","receiver":"eostea","stake_net_quantity":"5000.0000 EOS","stake_cpu_quantity":"5000....
>> from: eosio to: eostea net: 5000.0000 EOS cpu: 5000.0000 EOStotalsvoters
warning: transaction executed locally, but may not be confirmed by the network yet
```
## 注册BP
> 注意这里的`url`是节点的`http-server-address`。

代码如下：
```
cleos  system regproducer eostea EOS8Ar1fUGtZxcJ8Rdkh3rc55Vqg3ariR6fdBV8zxz4WgTW3nT73L http://http-server-address:8888
```
执行结果如下：
```
1381671ms thread-0   main.cpp:418                  create_action        ] result: {"binargs":"00000000189531550003b03a1cb0fa44023cbdb8cda51c2e0f08baf39742a6f50a241f4eee7b92c78a831a687474703a2f2f3134392e3132392e38312e3136333a36363636"} arg: {"code":"eosio","action":"regproducer","args":{"producer":"eostea","producer_key":"EOS8Ar1fUGtZxcJ8Rdkh3rc55Vqg3ariR6fdBV8zxz4WgTW3nT73L","url":"http://149.129.81.163:6666"}}
executed transaction: 835e6a4f8a892b607deac57beb64ccd558e95344ead9218a8cd8f6b51e8b6b15  256 bytes  879 us
#         eosio <= eosio::regproducer           {"producer":"eostea","producer_key":"EOS8Ar1fUGtZxcJ8Rdkh3rc55Vqg3ariR6fdBV8zxz4WgTW3nT73L","url":"h...
warning: transaction executed locally, but may not be confirmed by the network yet
```
## 转账
>由于要投票必须要有`EOS`账号，所以在这之前通过`newaccount`创建投票账号。然后进行转账。

代码如下：
```
cleos  push action eosio.token transfer '["eosio", "lome","1000000000.0000 EOS","vote"]' -p eosio
```
执行结果如下：
```
executed transaction: 160c0e2bbd405f567dcf28e0db5c6eff12e80cb08a58dd66b5115bde27a7f44c  224 bytes  736 us
#   eosio.token <= eosio.token::transfer        {"from":"eosio","to":"eostea","quantity":"10000.0000 EOS","memo":"vote"}
>> transfer from eosio to eostea 10000.0000 EOS
#         eosio <= eosio.token::transfer        {"from":"eosio","to":"eostea","quantity":"10000.0000 EOS","memo":"vote"}
#        eostea <= eosio.token::transfer        {"from":"eosio","to":"eostea","quantity":"10000.0000 EOS","memo":"vote"}
warning: transaction executed locally, but may not be confirmed by the network yet
```
## 锁定代币
代码如下：
```
cleos system delegatebw lome lome '25000000.0000 EOS' '25000000.0000 EOS' --transfer
```
执行结果如下：
```
1717800ms thread-0   main.cpp:1002                 operator()           ] act_payload: {"from":"eostea","receiver":"eostea","stake_net_quantity":"5000.0000 EOS","stake_cpu_quantity":"5000.0000 EOS","transfer":true}
1718159ms thread-0   main.cpp:418                  create_action        ] result: {"binargs":"0000000018953155000000001895315580f0fa020000000004454f530000000080f0fa020000000004454f530000000001"} arg: {"code":"eosio","action":"delegatebw","args":{"from":"eostea","receiver":"eostea","stake_net_quantity":"5000.0000 EOS","stake_cpu_quantity":"5000.0000 EOS","transfer":true}}
executed transaction: 2eece2e97c122bb8403137290f492508b0ae0d625bc1cf56fe01348125658139  232 bytes  1996 us
#         eosio <= eosio::delegatebw            {"from":"eostea","receiver":"eostea","stake_net_quantity":"5000.0000 EOS","stake_cpu_quantity":"5000...
>> from: eostea to: eostea net: 5000.0000 EOS cpu: 5000.0000 EOStotalsvoters
#   eosio.token <= eosio.token::transfer        {"from":"eostea","to":"eosio","quantity":"10000.0000 EOS","memo":"stake bandwidth"}
>> transfer from eostea to eosio 10000.0000 EOS
#        eostea <= eosio.token::transfer        {"from":"eostea","to":"eosio","quantity":"10000.0000 EOS","memo":"stake bandwidth"}
#         eosio <= eosio.token::transfer        {"from":"eostea","to":"eosio","quantity":"10000.0000 EOS","memo":"stake bandwidth"}
warning: transaction executed locally, but may not be confirmed by the network yet
```
## 投票
> 激动人心的时刻到了，我们要进行投票了，在这之前，我已经在`lome`,`strahe`,`chare`三个投票账户分别抵押了`５千万EOS`,可能许多朋友都猜到了，是的，需要`150,000,000.0000 TOKENS`投票才能解锁网络。

投票代码如下：
```
cleos system voteproducer prods lome eostea
```
结果如下：
```
2581429ms thread-0   main.cpp:418                  create_action        ] result: {"binargs":"0000000000a0248d0000000000000000010000000018953155"} arg: {"code":"eosio","action":"voteproducer","args":{"voter":"lome","proxy":"","producers":["eostea"]}}
executed transaction: 5a752a45a5b26613394fecd082cdd26b0726168731e37c85f1777dbba6dff093  208 bytes  1372 us
#         eosio <= eosio::voteproducer          {"voter":"lome","proxy":"","producers":["eostea"]}
>> orig total_votes: 4.034073698549329e+22 delta: 2.017036849274665e+26
warning: transaction executed locally, but may not be confirmed by the network yet
```
当你的投票数超过`150M`时，你的节点`BP`将开始产生区块。

> 后记：　这就是一个`EOS4.0`测试网络搭建的全过程，但其中还有许多的东西，笔者并未深究。
例如：
１．是否投票后需要锁定三天，取消抵押需要三天后才生效。
２．创建账号是否只能够使用`eosio.system`合约中的`newaccount`
等等。。
希望后来者提出更多的问题，也期望后来者能解决问题，谢谢～
