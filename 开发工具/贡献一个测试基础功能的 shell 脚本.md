脚本说明：其实密钥是可以在脚本中生成的。我这里直接生成，在脚本里写死了，有些地方不严谨，不成熟，凑合能用，欢迎指正，谢谢。

一、单实例测试

1、前置条件

1.1 配置文件 config.ini
```
genesis-json = genesis.json
block-log-dir = blocks
enable-stale-production = true
shared-file-dir = blockchain
shared-file-size = 8192
http-server-address = 127.0.0.1:8888
p2p-listen-endpoint = 0.0.0.0:9876
p2p-server-address = localhost:9876
allowed-connection = any
required-participation = 33
private-key = ["EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV","5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"]
producer-name = eosio

plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::account_history_plugin
plugin = eosio::account_history_api_plugin
plugin = eosio::wallet_plugin
plugin = eosio::wallet_api_plugin
plugin = eosio::net_plugin
plugin = eosio::net_api_plugin
plugin = eosio::chain_plugin
```

1.2 目前只能一个producer工作  eosio

1.3 规划4个钱包：default,wallet_1,wallet_2,wallet_3

1.4 生成12对公私钥对，备用

1.5  规划6个账户：    walt1acc1,walt1acc2
                                    walt2acc1,walt2acc2
                                    walt3acc1,walt3acc2
2、测试步骤和命令

2.1创建default 钱包

2.2初始化eosio基础配置：eosio.bios

2.3创建eosio自带的智能合约系统:eosio.system

2.4发行EOS合约币：100000.0000 EOS  给eosio账户

2.5创建currency账户：使用系统公钥对 keypair0

2.6创建设置currency智能合约

2.7创建currency智能合约币，最大发行量：1000000.0000 CUR

2.8 发行CUR合约币：10000.0000给currency账户

2.9使用创建钱包的命令创建：三个钱包 wallet_1,wallet_2,wallet_3

2.10使用12对公私钥创建6个账户

2.10.1把新创建的6个账户对应的私钥分别导入到三个钱包，这样钱包和账户就绑定了
    
![](https://eosfans-static.strahe.com/photo/2018/235bb8a9-efd4-4f5b-9335-6289d41d3918.jpg?x-oss-process=image/resize,w_1920)

2.10.2 查看钱包的列表  ./cleos wallet list

2.10.3 查看钱包的keys  ./cleos wallet keys

2.10.4 锁定wallet_1,查看钱包列表  wallet list,查看钱包密钥对  wallet keys

2.10.5解锁wallet_1,查看钱包列表  wallet list,查看钱包密钥对  wallet keys

2.11转账测试：转EOS和CUR以及读余额的测试

-------手动分割线(以下是脚本部分)----------------ubuntu1604 和MACOSX都能跑通----------------------------------------------

```shell
#!/bin/bash

#1 Define keypais
#keypair0
privatekey0=5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
pbulickkey0=EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

#keypair1   
Privatekey1=5JZ5Wwb8uQbi3A7DmMsD2zevcKCYw1pxmitij1x4xCjU8gv7ucj
Publickey1=EOS6a5pr4DS4CksCQSHqTdKMPbAdCyrE4b7QExDwTuCxH1vbkYMqG

#keypair2   
Privatekey2=5JKkei9CFtawsvnHt728DUQaahcjHm5nqJsNgZzna9XZKq8eA5c
Publickey2=EOS5NiFNF4bG7T49S6f7qVXMAt4RN2WM211s77UZrwD4cz2Xu6gw9

#keypair3  
Privatekey3=5JBDtjPbUeV2Hte6ZuFE5ny9RtuUujWEKG1u2yYPw2jmkCR7A4Y
Publickey3=EOS59rjXxZLjRnUEdErjtCEN8fihQnMmdsWYSz7jaeruPEoSeyCHz

#keypair4   
Privatekey4=5JQPYAtWxdzGsJkBpHyWBV18N2rzFtMjcBwxvfndS3KXe4oQu3L
Publickey4=EOS5psRxWMGyQS4HPNY8fa4PDhgP53vD4AZ6w24Z9HUCTxXKEH7Ey

#keypair5 
Privatekey5=5K8tbrJbsC1nK588jLBjZwDUdRtARiLWery69QybgpEK7JNGZX8
Publickey5=EOS6HHgXNrrjRTDmM12Ym6kehMNLrao4JpqQ5JWx1unVX7SFF296V

#keypair6  
Privatekey6=5HrmFypKi9ES1pnLYL43rJK5n3nZh7jVSVa7PqTrbHeCu5HYUr6
Publickey6=EOS5wquAffiaMMQsXyibBGXpB7WWhZYFtwmSUGZcfkFE49pjsB28C

#keypair7  
Privatekey7=5HpwW81Wj3zx3VwpzitsBvjT3dTNKzffrj1BBtjgbsV2DbKJqgq
Publickey7=EOS8EFSo72zEhgYyq5eZKrStabJegGZ5koNdr3o3iS9QcEyNnKEuo

#keypair8  
Privatekey8=5HwLFxXXF8rvbFLiQXD2rE9HTWfppYiqK6G9e3uMWv78nRWkMVw
Publickey8=EOS5HTYTunH1vApRdoF3bVWQaMmbh2BtaNnTYYVBdvHEQpAE4PDCh

#keypair9
Privatekey9=5JRzVojEi46ntQeiFR2G6xFSNBMqHDgKMqxiQmzomLGWzAwQs5F
Publickey9=EOS7o4vLAhMsT5caP2Pv8c4Yha3tSG5EMYSzVwjRiLpQRZ9LsTyKY

#keypair10
Privatekey10=5JAR12zZYs745RnHzXAEUik38SP73oPF2UNqyzxoJ8ySbe7Ry6C
Publickey10=EOS5ar66dCYpQC715kZhmYf69YCq2Ht1X8AY4Zp31nfvnqZZRLmoK

#keypair11   
Privatekey11=5J4oA7KCzx9MejkXmNgNqeSiD5DXccyYCQs4j9YuvqGcuexy5Ei
Publickey11=EOS7sCjRdmL7c5khiLwwstU7LKPY3wGWx8XFfwviGjaobzzU2vP9q

#keypair12  
Privatekey12=5KKERr29AtW2jgW4911PLG61ezG6DLSteCKcS8NFpfVwDtGbBwg
Publickey12=EOS5Wd9zeYJKCSxsmwaQvpkjs8PDeHozjzkZpqTxvCPPeckfFV6YG


waitnsec=1

#2.1 create default wallet and save pin
CMDRES=`./cleos wallet create`
if [[ $? -eq 1 ]];then
  echo "create wallet default failed!"
  exit 1
fi
echo "create wallet default ok!"
temppin=${CMDRES#*\"}
DefaultWalletPin=${temppin%\"*}

sleep ${waitnsec}

#2.2初始化eosio基础配置：eosio.bios
./cleos set contract eosio ../../contracts/eosio.bios -p eosio
if [[ $? -eq 1 ]];then
  echo "bios initialize failed!"
  exit 1
fi
echo "bios initialize ok!"

sleep ${waitnsec}

#2.3创建eosio自带的智能合约系统:eosio.system
./cleos set contract eosio ../../contracts/eosio.system
if [[ $? -eq 1 ]];then
  echo "set contract eosio failed!"
  exit 1
fi
echo "set contract eosio ok!"

sleep ${waitnsec}

#2.4发行EOS合约币：100000.0000 EOS  给eosio账户
./cleos push action eosio issue '{"to":"eosio","quantity":"10000.0000 EOS"}' --permission eosio@active
if [[ $? -eq 1 ]];then
  echo "issue contract eosio EOS coin  failed!"
  exit 1
fi
echo "issue contract eosio EOS coin ok!"

sleep ${waitnsec}


#2.5创建currency账户：使用系统公钥对 keypair0
./cleos create account eosio currency $pbulickkey0 $pbulickkey0
if [[ $? -eq 1 ]];then
  echo "issue contract eosio EOS coin  failed!"
  exit 1
fi
echo "issue contract eosio EOS coin ok!"

sleep ${waitnsec}

#2.6创建设置currency智能合约
./cleos set contract currency ../../contracts/currency

sleep ${waitnsec}

#2.7创建currency智能合约币，最大发行量：1000000.0000 CUR
./cleos push action currency create '{"issuer":"currency","maximum_supply":"1000000.0000 CUR","can_freeze":0,"can_recall":0,"can_whitelist":0}' --permission currency@active

sleep ${waitnsec}

#2.8 发行CUR合约币：10000.0000给currency账户
./cleos push action currency issue '{"to":"currency","quantity":"10000.0000 CUR","memo":""}' --permission currency@active

sleep ${waitnsec}


#2.9使用创建钱包的命令创建：三个钱包 wallet_1,wallet_2,wallet_3

CMDRES=`./cleos wallet create -n wallet_1`
if [[ $? -eq 1 ]];then
  echo "create wallet wallet_1 failed!"
  exit 1
fi
echo "create wallet wallet_1 ok!"
temppin=${CMDRES#*\"}
Wallet1Pin=${temppin%\"*}

sleep ${waitnsec}

CMDRES=`./cleos wallet create -n wallet_2`
if [[ $? -eq 1 ]];then
  echo "create wallet wallet_2 failed!"
  exit 1
fi
echo "create wallet wallet_2 ok!"
temppin=${CMDRES#*\"}
Wallet2Pin=${temppin%\"*}

sleep ${waitnsec}

CMDRES=`./cleos wallet create -n wallet_3`
if [[ $? -eq 1 ]];then
  echo "create wallet wallet_3 failed!"
  exit 1
fi
echo "create wallet wallet_3 ok!"
temppin=${CMDRES#*\"}
Wallet3Pin=${temppin%\"*}

sleep ${waitnsec}

#2.10使用12对公私钥创建6个账户
./cleos create account eosio walt1acc1 ${Publickey1} ${Publickey2}

./cleos create account eosio walt1acc2 ${Publickey3} ${Publickey4}

./cleos create account eosio walt2acc1 ${Publickey5} ${Publickey6}

./cleos create account eosio walt2acc2 ${Publickey7} ${Publickey8}

./cleos create account eosio walt3acc1 ${Publickey9} ${Publickey10}

./cleos create account eosio walt3acc2 ${Publickey11} ${Publickey12}


sleep ${waitnsec}

#2.10.1把新创建的6个账户对应的私钥分别导入到三个钱包，这样钱包和账户就绑定了
 
#./cleos wallet import ${privatekey0} 

./cleos wallet import -n wallet_1 ${Privatekey1}

./cleos wallet import -n wallet_1 ${Privatekey2}

./cleos wallet import -n wallet_1 ${Privatekey3}

./cleos wallet import -n wallet_1 ${Privatekey4}

./cleos wallet import -n wallet_2 ${Privatekey5}

./cleos wallet import -n wallet_2 ${Privatekey6}

./cleos wallet import -n wallet_2 ${Privatekey7}

./cleos wallet import -n wallet_2 ${Privatekey8}

./cleos wallet import -n wallet_3 ${Privatekey9}

./cleos wallet import -n wallet_3 ${Privatekey10}

./cleos wallet import -n wallet_3 ${Privatekey11}

./cleos wallet import -n wallet_3 ${Privatekey12}

sleep ${waitnsec}


#2.10.2 查看钱包的列表  ./cleos wallet list
./cleos wallet list

sleep ${waitnsec}

#2.10.3 查看钱包的keys  ./cleos wallet keys
./cleos wallet keys

sleep ${waitnsec}

#2.10.4 锁定wallet_1,查看钱包列表  wallet list,查看钱包密钥对  wallet keys
./cleos wallet lock -n wallet_1
./cleos wallet list
./cleos wallet keys

sleep ${waitnsec}

#2.10.5解锁wallet_1,查看钱包列表  wallet list,查看钱包密钥对  wallet keys
echo $Wallet1Pin | ./cleos wallet unlock -n wallet_1
./cleos wallet list
./cleos wallet keys

sleep ${waitnsec}

#2.11转账测试：
#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

echo "------------------------------get all account balance first---------------------------------------------"

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------eosio pay to other 7 accounts 100.0000-100.0006 EOS--------------------------------------"

# eosio pay to other 7 accounts 100.0000-100.0006 EOS
amount=1000000
accounts=("currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")
for account in ${accounts[@]};do
./cleos transfer eosio ${account} ${amount}
if [[ $? -eq 1 ]];then
  echo "EOS transfer failed!"
  exit 1
fi
let "amount += 1"
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------currency pay to other 7 accounts 110.0000 CUR--------------------------------------"

#./cleos push action currency transfer '{"from":"currency","to":"eosio","quantity":"20.0000 CUR","memo":"my first transfer"}' --permission currency@active

# currency pay to other 7 accounts 110.0000 CUR

accounts=("eosio" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")
amount=10000
for account in ${accounts[@]};do
./cleos push action currency transfer '{"from":"currency","to":"'${account}'","quantity":"110.0000 CUR","memo":""}' --permission currency@active
if [[ $? -eq 1 ]];then
  echo "currency transfer failed!"
  exit 1
fi
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
if [[ $? -eq 1 ]];then
  echo "currency transfer failed!"
  exit 1
fi
done

for account in ${accounts[@]};do
echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
if [[ $? -eq 1 ]];then
echo "currency transfer failed!"
exit 1
fi
done

sleep ${waitnsec}

echo "-------------------------walt1acc1 pay to other 7 accounts 1.0000-7.0000 EOS--------------------------------------"

# walt1acc1 pay to other 7 accounts 1.0000-7.0000 EOS
amount=10000
accounts=("eosio" "currency" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")
for account in ${accounts[@]};do
  ./cleos transfer walt1acc1 ${account} ${amount}
  if [[ $? -eq 1 ]];then
    echo "eos transfer failed!"
    exit 1
  fi
  let "amount += 10000"
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
  if [[ $? -eq 1 ]];then
    echo "get balance failed!"
    exit 1
  fi
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
  if [[ $? -eq 1 ]];then
    echo "get balance failed!"
    exit 1
  fi
done

sleep ${waitnsec}

echo "-------------------------walt1acc2 pay to other 7 accounts 1.0001-1.0007 EOS--------------------------------------"

# walt1acc2 pay to other 7 accounts 1.0001-1.0007 EOS
amount=10001
accounts=("eosio" "currency" "walt1acc1" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")
for account in ${accounts[@]};do
./cleos transfer walt1acc2 ${account} ${amount}
 if [[ $? -eq 1 ]];then
    echo "eos transfer failed!"
    exit 1
  fi
let "amount += 1"
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------walt2acc1 pay to other 7 accounts 2.0001-2.0007 EOS--------------------------------------"

# walt2acc1 pay to other 7 accounts 2.0001-2.0007 EOS
amount=20001
accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc2" "walt3acc1" "walt3acc2")
for account in ${accounts[@]};do
./cleos transfer walt2acc1 ${account} ${amount}
  if [[ $? -eq 1 ]];then
    echo "eos transfer failed!"
    exit 1
  fi
let "amount += 1"
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------walt2acc2 pay to other 7 accounts 2.0011-2.0017 EOS--------------------------------------"

# walt2acc2 pay to other 7 accounts 2.0011-2.0017 EOS
amount=20011
accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt3acc1" "walt3acc2")
for account in ${accounts[@]};do
./cleos transfer walt2acc2 ${account} ${amount}
if [[ $? -eq 1 ]];then
    echo "eos transfer failed!"
    exit 1
fi
let "amount += 1"
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------walt3acc1 pay to other 7 accounts 3.0001-3.0007 EOS--------------------------------------"

# walt3acc1 pay to other 7 accounts 3.0001-2.0007 EOS
amount=30001
accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc2")
for account in ${accounts[@]};do
./cleos transfer walt3acc1 ${account} ${amount}
  if [[ $? -eq 1 ]];then
    echo "eos transfer failed!"
    exit 1
  fi
let "amount += 1"
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------walt3acc2 pay to other 7 accounts 3.0011-3.0017 EOS--------------------------------------"

# walt3acc2 pay to other 7 accounts 3.0011-3.0017 EOS
amount=30011
accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1")
for account in ${accounts[@]};do
./cleos transfer walt3acc2 ${account} ${amount}
  if [[ $? -eq 1 ]];then
    echo "eos transfer failed!"
    exit 1
  fi
let "amount += 1"
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}


echo "-------------------------currency pay test form accounts--------------------------------------"

echo "-------------------------walt1acc1 pay to other 7 accounts 1.1111 CUR--------------------------------------"

# walt1acc1 pay to other 7 accounts 1.1111 CUR

accounts=("eosio" "currency" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")
for account in ${accounts[@]};do
./cleos push action currency transfer '{"from":"walt1acc1","to":"'${account}'","quantity":"1.1111 CUR","memo":""}' --permission walt1acc1@active
  if [[ $? -eq 1 ]];then
    echo "currency transfer failed!"
    exit 1
  fi
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------walt1acc2 pay to other 7 accounts 2.2222 CUR--------------------------------------"

# walt1acc2 pay to other 7 accounts 2.2222 CUR

accounts=("eosio" "currency" "walt1acc1" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")
for account in ${accounts[@]};do
./cleos push action currency transfer '{"from":"walt1acc2","to":"'${account}'","quantity":"2.2222 CUR","memo":"currency->eosio 2.2222 CUR"}' --permission walt1acc2@active
  if [[ $? -eq 1 ]];then
    echo "currency transfer failed!"
    exit 1
  fi
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------walt2acc1 pay to other 7 accounts 3.3333 CUR--------------------------------------"

# walt2acc1 pay to other 7 accounts 3.3333 CUR

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc2" "walt3acc1" "walt3acc2")
for account in ${accounts[@]};do
./cleos push action currency transfer '{"from":"walt2acc1","to":"'${account}'","quantity":"3.3333 CUR","memo":""}' --permission walt2acc1@active
  if [[ $? -eq 1 ]];then
    echo "currency transfer failed!"
    exit 1
  fi
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

sleep ${waitnsec}

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------walt2acc2 pay to other 7 accounts 4.4444 CUR--------------------------------------"

# walt2acc2 pay to other 7 accounts 4.4444 CUR
amount=11110
accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt3acc1" "walt3acc2")
for account in ${accounts[@]};do
./cleos push action currency transfer '{"from":"walt2acc2","to":"'${account}'","quantity":"4.4444 CUR","memo":""}' --permission walt2acc2@active
  if [[ $? -eq 1 ]];then
    echo "currency transfer failed!"
    exit 1
  fi
let "amount += 10"
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

sleep ${waitnsec}

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------walt3acc1 pay to other 7 accounts 5.5555 CUR--------------------------------------"

# walt3acc1 pay to other 7 accounts 5.5555 CUR

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc2")
for account in ${accounts[@]};do
./cleos push action currency transfer '{"from":"walt3acc1","to":"'${account}'","quantity":"5.5555 CUR","memo":""}' --permission walt3acc1@active
  if [[ $? -eq 1 ]];then
    echo "currency transfer failed!"
    exit 1
  fi
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done

sleep ${waitnsec}

echo "-------------------------walt3acc2 pay to other 7 accounts 6.6666 CUR--------------------------------------"

# walt3acc2 pay to other 7 accounts 6.6666 CUR

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1")
for account in ${accounts[@]};do
./cleos push action currency transfer '{"from":"walt3acc2","to":"'${account}'","quantity":"6.6666 CUR","memo":""}' --permission walt3acc2@active
  if [[ $? -eq 1 ]];then
    echo "currency transfer failed!"
    exit 1
  fi
done

sleep ${waitnsec}

#读取各账户的EOS和CUR余额：eosio  currency   walt1acc1   walt1acc2   walt2acc1   walt2acc2  walt3acc1 walt3acc2

accounts=("eosio" "currency" "walt1acc1" "walt1acc2" "walt2acc1" "walt2acc2" "walt3acc1" "walt3acc2")

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance eos ${account}`"
done

sleep ${waitnsec}

for account in ${accounts[@]};do
  echo "account ${account} balance is `./cleos get currency balance currency ${account}`"
done```
