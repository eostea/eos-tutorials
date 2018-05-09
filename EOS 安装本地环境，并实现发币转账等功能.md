第一步：很重要：买个服务器或者装个虚拟机，推荐用linux服务器
注意：由于国内网络环境原因，建议使用国外服务器搭建。
 

下面为环境搭建过程

首先保证你服务器安装有git，没有自行百度安装

001 获取代码
克隆EOS存储库及子模块

git clone https://github.com/EOSIO/eos --recursive

002 安装EOSIO
这里我们使用自动构建脚本安装：

cd eos

./eosio_build.sh
询问是否安装这些包，输入1确认。安装开始。中间大概运行两个小时左右，根据服务器性能判断，可放下电脑做其他的

003 运行系统
首先，需要运行mongod数据库，然后运行test，测试一下，操作如下

~/opt/mongodb/bin/mongod -f ~/opt/mongodb/mongod.conf &

然后运行test

cd build

make test

我在运行test测试时和运行mongod数据库是发现了错误，需要验证大概32项，其中8项运行失败。(原因未知，但最终我在测试失败的情况下依然良好的完成了安装和转账，在完成后，回来再次测试，依然是8运行失败。当然后续我会排查失败原因。）

004 安装可执行文件

cd build

sudo make install


005 创建单一测试节点

cd build/programs/nodeos

nodeos

这时候会报错正常的  这时候需要修改config.ini，config.ini位于这个目录下

cd ~/.local/share/eosio/nodeos/config

# Enable production on a stale chain, since a single-node test chain is pretty much always stale

enable-stale-production = true

# Enable block production with the testnet producers

producer-name = eosio

# Load the block producer plugin, so you can produce blocks

plugin = eosio::producer_plugin

# Wallet plugin

plugin = eosio::wallet_api_plugin

# As well as API and HTTP plugins

plugin = eosio::chain_api_plugin

plugin = eosio::http_plugin

# This will be used by the validation step below, to view account history

plugin = eosio::account_history_api_plugin

贴上去以后  注意缩进，排列。需要注释  文件上放已存在的enable-stale-production = false，这个如果文件上方不存在就不需要注释plugin = eosio::chain_api_plugin


这时候我们尝试启动一个单一测试节点：

cd build/programs/nodeos

nodeos

运行成功。

006 “货币”合同演练


在演练中，我们会尝试建立两个账户currency和eosio，然后发行一种叫做MGD（随意名称，因为我在发行此名称时候出错，可能是因为币已存在）的代币，然后尝试一次转账操作，最后再查询余额，确定转账成功。

所有的操作，都是基于cleos完成的。

首先我们需要保持nodeos的运行。

然后，用下面的命令创建一个钱包。



cd build/programs/cleos



./cleos wallet create

正常情况下会创建一个钱包，还会展示私钥。


加载BIOS合约（注意要到/eos/build/programs/cleos目录下操作）

./cleos set contract eosio ../../contracts/eosio.bios -p eosio

为货币合约创建一个账户currency，首先生成两组key，分别对应OwnerKey和ActiveKey

在cleos目录下：生成的key做好备份，两个key每一个都会有公钥私钥，分别备份

./cleos create key  # OwnerKey

./cleos create key  # ActiveKey


然后，将key导入到钱包,import 后的代码不执行

./cleos wallet import <private-OwnerKey>（导入OwnerKey的私钥）

./cleos wallet import <private-ActiveKey> （导入ActiveKey的私钥）

接下来，用cleos create account命令，创建账户currency导入两个公钥，空格隔开currency 后的代码替换成生成的两个公钥

./cleos create account eosio currency <public-OwnerKey> <public-ActiveKey>

我们使用 get account命令，看以下currency是否已经创建成功：

./cleos get account currency

接下来，将示例货币合约上传至区块链

在上传合约前，确认一下当前合约还未创建 返回的code hash如果全是0就没创建

./cleos get code currency


使用货币账户上传样本货币合约，响应包含一个transaction_id的JSON，代表合同上传成功

./cleos set contract currency ../../contracts/currency

接下来，可以再试一次，看看是否成功：

./cleos get code currency

然后就是发币的环节，要先创造货币，这一步，之前版本是没有的

cleos push action currency create '{"issuer":"currency", "maximum_supply": "1000000000.0000 CUR", "can_freeze": 1, "can_recall": 1, "can_whitelist": 1}' -p currency@active 


然后在发行货币

./cleos push action currency issue '{"to":"currency","quantity":"1000.0000 CUR","memo":""}' --permission currency@active 

还有一个坑，就是获取账号信息的时候，这里文档写的是这个样子的

./cleos get table currency currency account 

然而实际使用是这个：

./cleos get table currency currency accounts 


下面我们使用currency合约来转移资金：

这个命令现实发送到货币合约的转账操作，将20.0000 MGD从货币账户转移到eosio账户


./cleos push action currency transfer'{"from":"currency","to":"eosio","quantity":"20.0000 MGD","memo":"my first transfer"}'--permission currency@active

上面代码一起执行

来看一下余额的变化

./cleos get table currency eosio accounts  //eosio 账户

./cleos get table currency currency accounts  currency 账户


接下来，要做的就是合约的开发使用。由于不知道eos开发支持什么语言要先去了解更多eos基础

下面贴出 eos官方文档，和官方 wiki，和官方代码，和eos的中文社区，和eos浏览器

https://eosio.github.io/eos/

https://github.com/EOSIO/eos/wiki

https://github.com/EOSIO/eos

https://eosfans.io/

https://eostracker.io/producers
