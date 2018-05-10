# cleos 钱包命令
## cleos设置

1. nodeos服务地址参数

```
cleos -H nodeos.httpserver(nodeos服务的地址) -p nodeos.httpport(nodeos服务的端口) wallet ....
```

2. cleos钱包地址参数

```
cleos --wallet-host keosd.httpserver (钱包服务的地址) --wallet-port keosd.httpport(钱包服务的端口) wallet ....
```

## cleos wallet 

### 若未指定名称则统一为操作钱包`default`

1. 钱包列表

```
cleos wallet list
```

2. 创建钱包

```
cleos wallet create -n walletname(创建指定名称的钱包)
```

3. 打开钱包

```
cleos wallet open -n walletname(打开指定名称的钱包)
```

4. 解锁钱包

```
cleos wallet unlock -n walletname (解锁指定名称的钱包)
```

5. 锁定钱包

```
cleos wallet lock -n walletname (锁定指定名称的钱包)
```

6. 锁定所有钱包

```
cleos wallet lock_all 
```

## 钱包状态

```
cleos wallet list
```

通过以上命令可以查看当前所有本地钱包的状态， 返回如下：

```
Wallets:
[
  "default *",
  "lome"
]
```

###  * 表示解锁状态，其他为锁定状态， 只有解锁状态的钱包的密匙才能用来签名。

## 本文地址：https://eosfans.io/topics/351