
# 1. dice 玩法说明

两个人玩的猜数游戏，两个出一样额度的赌注，然后A写下一个数，B如果在一定时间内猜对，则赢得赌注。（理解可能有误，请懂的大佬指出错误）

# 2. 操作大致流程

概要：创建钱包，发行货币，创建两个玩家，上传押金，上传同样赌注，开始游戏，结束游戏，拿走货币。

首先启动nodes

```python
nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::account_history_api_plugin
```

然后创建钱包，然后导入eosio（默认导入）和dice 私钥

```python
cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5HpvzFiU7kLx6FXcq5RjEyU33mWLEFJuZQPz4znBFDwb6DFFac6"

cleos wallet import 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
imported private key for: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
```

上传bios合约，用于创建eosio.token账号，为eosio发行货币

```python
cleos set contract eosio build/contracts/eosio.bios -p eosio
Reading WAST/WASM from build/contracts/eosio.bios/eosio.bios.wast...
Assembling WASM...
Publishing contract...
executed transaction: 88355c1ebc31960b521b992f9410f95a5a0921c790f424356453cf5f84edb077 3288 bytes 2200576 cycles
# eosio <= eosio::setcode {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001581060037f7e7f0060057f7e7e7e7e...
# eosio <= eosio::setabi {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```

创建eosio.token账号

```python
cleos create account eosio eosio.token EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 46c68f270dfceb36ce2270a6bad89ca2228b9750a19a3a3bc92578a46637fbb1 352 bytes 102400 cycles
# eosio <= eosio::newaccount {"creator":"eosio","name":"eosio.token","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7D...
```

为eosio.token账号部署eosio.token合约

```python
cleos set contract eosio.token build/contracts/eosio.token -p eosio.token
Reading WAST/WASM from build/contracts/eosio.token/eosio.token.wast...
Assembling WASM...
Publishing contract...
executed transaction: 5a0305bcab99b9d9f8590f51ce4d55bb0a0e60c2d79f4d61cd1ab210571171e2 8288 bytes 2200576 cycles
# eosio <= eosio::setcode {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d010000000183011560067f7e7f7f7f7f00...
# eosio <= eosio::setabi {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...
```

创建dice账号，用来掌管玩家的赌注

```python
cleos create account eosio dice EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 7d144219ebc5e2098af0b621221ea7d56572037a6adef53b5dd5691999818d18 352 bytes 102400 cycles
# eosio <= eosio::newaccount {"creator":"eosio","name":"dice","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...
```

部署dice合约

```python
cleos set contract dice build/contracts/dice -p dice
Reading WAST/WASM from build/contracts/dice/dice.wast...
Assembling WASM...
Publishing contract...
executed transaction: c810aa75fe5f00520c16fb7464919756cf8bc7a1f424061a0de2a41dc6c3b6e1 11744 bytes 2200576 cycles
# eosio <= eosio::setcode {"account":"dice","vmtype":0,"vmversion":0,"code":"0061736d0100000001ad011a60047f7f7e7f0060027f7f006...
# eosio <= eosio::setabi {"account":"dice","abi":{"types":[],"structs":[{"name":"offer","base":"","fields":[{"name":"id","typ...
```

为eosio创建货币，dice只支持EOS

```python
cleos push action eosio.token create '[ "eosio", "1000000000.0000 EOS", 0, 0, 0]' -p eosio.token
executed transaction: b6ffc0fbb62ff4c23b7d24b8b55a34bf0c74e7c710b18d16af326b939ae9ecfd 248 bytes 104448 cycles
# eosio.token <= eosio.token::create {"issuer":"eosio","maximum_supply":"1000000000.0000 EOS","can_freeze":0,"can_recall":0,"can_whitelis...
```

创建玩家alice和bob

```python
leos create account eosio alice EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 5e4b7f4a687db33afa025ab7a5ea86c0e13cab68455b109c5346b02c1c62652f 352 bytes 102400 cycles
# eosio <= eosio::newaccount {"creator":"eosio","name":"alice","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZ...

cleos create account eosio bob EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 141d6eabefc7e9fa84261559970e54d9674d5278a89a507bce3228dae7169be2 352 bytes 102400 cycles
# eosio <= eosio::newaccount {"creator":"eosio","name":"bob","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZe...
```

为alice和bob发行1000 EOS

```python
cleos push action eosio.token issue '[ "alice", "1000.0000 EOS", "" ]' -p eosio
executed transaction: 8e33d754aa7dd8987522c3c76ab9a1bffada096363f71169e6da71d444552631 248 bytes 124928 cycles
# eosio.token <= eosio.token::issue {"to":"alice","quantity":"1000.0000 EOS","memo":""}
>> issue
# eosio.token <= eosio.token::transfer {"from":"eosio","to":"alice","quantity":"1000.0000 EOS","memo":""}
>> transfer
# eosio <= eosio.token::transfer {"from":"eosio","to":"alice","quantity":"1000.0000 EOS","memo":""}
# alice <= eosio.token::transfer {"from":"eosio","to":"alice","quantity":"1000.0000 EOS","memo":""}

cleos push action eosio.token issue '[ "bob", "1000.0000 EOS", "" ]' -p eosio
executed transaction: 6cac95b553404bc98cd7cd88088c4a9368b8ab1be3cd7dde0a337cb3c775a919 248 bytes 124928 cycles
# eosio.token <= eosio.token::issue {"to":"bob","quantity":"1000.0000 EOS","memo":""}
>> issue
# eosio.token <= eosio.token::transfer {"from":"eosio","to":"bob","quantity":"1000.0000 EOS","memo":""}
>> transfer
# eosio <= eosio.token::transfer {"from":"eosio","to":"bob","quantity":"1000.0000 EOS","memo":""}
# bob <= eosio.token::transfer {"from":"eosio","to":"bob","quantity":"1000.0000 EOS","memo":""}
```

激活alice和bob转账权限

```python
cleos set account permission alice active '{"threshold": 1,"keys": [{"key": "EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4","weight": 1}],"accounts": [{"permission":{"actor":"dice","permission":"active"},"weight":1}]}' owner -p alice
executed transaction: 49275f38369b6966293a9dcd2dd2c76818d6513fbf4f5590ede50840db9337b4 312 bytes 102400 cycles
# eosio <= eosio::updateauth {"account":"alice","permission":"active","parent":"owner","data":{"threshold":1,"keys":[{"key":"EOS7...

cleos set account permission bob active '{"threshold": 1,"keys": [{"key": "EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4","weight": 1}],"accounts": [{"permission":{"actor":"dice","permission":"active"},"weight":1}]}' owner -p bob
executed transaction: 78aee9021742cb71e3d0dd0683a10037c4001f1381334ed401ff4d060bb29b2e 312 bytes 102400 cycles
# eosio <= eosio::updateauth {"account":"bob","permission":"active","parent":"owner","data":{"threshold":1,"keys":[{"key":"EOS7ij...
```

进行游戏前，两人要上交100EOS押金（有点小贵哦）

```python
cleos push action dice deposit '[ "alice", "100.0000 EOS" ]' -p alice
executed transaction: 235e78118f80b11eb295815ebc2558019c1ac633f7c1c22dc8a0afd9e1a4babb 248 bytes 123904 cycles
# dice <= dice::deposit {"from":"alice","a":"100.0000 EOS"}
# eosio.token <= eosio.token::transfer {"from":"alice","to":"dice","quantity":"100.0000 EOS","memo":""}
>> transfer
# alice <= eosio.token::transfer {"from":"alice","to":"dice","quantity":"100.0000 EOS","memo":""}
# dice <= eosio.token::transfer {"from":"alice","to":"dice","quantity":"100.0000 EOS","memo":""}

cleos push action dice deposit '[ "bob", "100.0000 EOS" ]' -p bob
executed transaction: d309b2090d2a5e73d97f0d198b2967b20dcfa9ced2d80cc75e445e10b0f43c6f 248 bytes 122880 cycles
# dice <= dice::deposit {"from":"bob","a":"100.0000 EOS"}
# eosio.token <= eosio.token::transfer {"from":"bob","to":"dice","quantity":"100.0000 EOS","memo":""}
>> transfer
# bob <= eosio.token::transfer {"from":"bob","to":"dice","quantity":"100.0000 EOS","memo":""}
# dice <= eosio.token::transfer {"from":"bob","to":"dice","quantity":"100.0000 EOS","memo":""}
```

alice 随机生成一个32长度的串，hex显示，在通过sha256加密后，生成一个加密串，然后将赌注和加密串同时上传到dice监管

```python
openssl rand 32 -hex
15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12

echo -n '15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12' | xxd -r -p | sha256sum -b | awk '{print $1}'
50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129
cleos push action dice offerbet '[ "3.0000 EOS", "alice", "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883" ]' -p alice
executed transaction: 2acc94e4d1fb6937fc5686740855230f3ad45f15d2406cf76daa55a57fe2a1af 280 bytes 106496 cycles
# dice <= dice::offerbet {"bet":"3.0000 EOS","player":"alice","commitment":"d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793...
```

然后让bob干同样的事情，然后游戏已经开始了

```python

openssl rand 32 -hex
15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12

echo -n '15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12' | xxd -r -p | sha256sum -b | awk '{print $1}'
50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129

cleos push action dice offerbet '[ "3.0000 EOS", "bob", "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129" ]' -p bob
executed transaction: bcb28be69afd0eab7d125fb4fae79019f55bb18046fb4e6ee7e7982a8aed4309 280 bytes 112640 cycles
# dice <= dice::offerbet {"bet":"3.0000 EOS","player":"bob","commitment":"50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d...
```

检查一下EOS是不是真确操作了，alice和bob，dice

```python
cleos get table dice dice account
{
"rows": [{
"owner": "alice",
"eos_balance": "97.0000 EOS",
"open_offers": 0,
"open_games": 1
},{
"owner": "bob",
"eos_balance": "97.0000 EOS",
"open_offers": 0,
"open_games": 1
}
],
"more": false
}

cleos get table dice dice game
{
"rows": [{
"id": 1,
"bet": "3.0000 EOS",
"deadline": "1970-01-01T00:00:00",
"player1": {
"commitment": "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883",
"reveal": "0000000000000000000000000000000000000000000000000000000000000000"
},
"player2": {
"commitment": "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129",
"reveal": "0000000000000000000000000000000000000000000000000000000000000000"
}
}
],
"more": false
}
```

bob揭示他的字符串

```python
cleos push action dice reveal '[ "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129", "15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12" ]' -p bob
executed transaction: 8bd988a929970e4f110d4c9500dc13c81110eff4a8a02743bcf433f5bd8709e1 288 bytes 105472 cycles
# dice <= dice::reveal {"commitment":"50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129","source":"15fe76d25...
```

再来看一下dice

```python
cleos get table dice dice game
{
"rows": [{
"id": 1,
"bet": "3.0000 EOS",
"deadline": "2018-04-19T07:58:56",
"player1": {
"commitment": "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883",
"reveal": "0000000000000000000000000000000000000000000000000000000000000000"
},
"player2": {
"commitment": "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129",
"reveal": "15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12"
}
}
],
"more": false
}
```

alice 揭示他的code，游戏就结束了，赢得是alice（没看懂怎么赢得）

```python
cleos push action dice reveal '[ "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883", "28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905" ]' -p alice
executed transaction: 16400e6850ea39d3afec89cf257e840248ea2d515533b785d72e0dc8e431f8ef 288 bytes 110592 cycles
# dice <= dice::reveal {"commitment":"d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883","source":"28349b1d4...
```

对一下账号信息

```python
cleos get table dice dice account
{
"rows": [{
"owner": "alice",
"eos_balance": "103.0000 EOS",
"open_offers": 0,
"open_games": 0
},{
"owner": "bob",
"eos_balance": "97.0000 EOS",
"open_offers": 0,
"open_games": 0
}
],
"more": false
}
```

alice拿走 押金和赢的赌注

```python
cleos push action dice withdraw '[ "alice", "103.0000 EOS" ]' -p alice
executed transaction: 6e9524dfe59dc79c0016c7afa5c080368b1f09eacd9b0823340051183aca10a7 248 bytes 124928 cycles
# dice <= dice::withdraw {"to":"alice","a":"103.0000 EOS"}
# eosio.token <= eosio.token::transfer {"from":"dice","to":"alice","quantity":"103.0000 EOS","memo":""}
>> transfer
# dice <= eosio.token::transfer {"from":"dice","to":"alice","quantity":"103.0000 EOS","memo":""}
# alice <= eosio.token::transfer {"from":"dice","to":"alice","quantity":"103.0000 EOS","memo":""}
```

最后再查询一下alice的balance（未通过）

```python
cleos get currency balance eosio.token alice eos
1003.0000 EOS
```
