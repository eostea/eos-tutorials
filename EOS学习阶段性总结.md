
> dawn3.0编译过程参考https://github.com/eosio/eos  readme.md

```shell
# 创建默认钱包
./cleos wallet create

# 设置基础配置智能合约：eosio.bios
./cleos set contract eosio ../../contracts/eosio.bios -p eosio -j

# 设置系统智能合约：eosio.system 部署了该智能合约，eosio账户才可以改包括自己在内的所有账户发行EOS token，才可以执行，注册producer,投票者下注，投票等操作
./cleos set contract eosio ../../contracts/eosio.system -j

# 给eosio账户发行EOS token 10个亿
./cleos push action eosio issue '{"to":"eosio","quantity":"1000000000.0000 EOS"}' --permission eosio@active -j

# 创建两组密钥对
./cleos create key

#keypair1   
#Private key: 5JZ5Wwb8uQbi3A7DmMsD2zevcKCYw1pxmitij1x4xCjU8gv7ucj
#Public key: EOS6a5pr4DS4CksCQSHqTdKMPbAdCyrE4b7QExDwTuCxH1vbkYMqG

#keypair2   
#Private key: 5JKkei9CFtawsvnHt728DUQaahcjHm5nqJsNgZzna9XZKq8eA5c
#Public key: EOS5NiFNF4bG7T49S6f7qVXMAt4RN2WM211s77UZrwD4cz2Xu6gw9

# 导入密钥到默认钱包
./cleos wallet import 5JZ5Wwb8uQbi3A7DmMsD2zevcKCYw1pxmitij1x4xCjU8gv7ucj
./cleos wallet import 5JKkei9CFtawsvnHt728DUQaahcjHm5nqJsNgZzna9XZKq8eA5c

# 查看钱包中的密钥对
./cleos wallet keys

# 创建账户 currency
./cleos create account eosio currency EOS6a5pr4DS4CksCQSHqTdKMPbAdCyrE4b7QExDwTuCxH1vbkYMqG EOS5NiFNF4bG7T49S6f7qVXMAt4RN2WM211s77UZrwD4cz2Xu6gw9 -j

# 设置currency的智能合约
./cleos set contract currency ../../contracts/currency -j

# currency 创建合约token CUR 最大发行量10个亿
./cleos push action currency create '{"issuer":"currency","maximum_supply":"1000000000.0000 CUR","can_freeze":0,"can_recall":0,"can_whitelist":0}' --permission currency@active -j

# 给currency账户发行 10万 CUR
./cleos push action currency issue '{"to":"currency","quantity":"100000.0000 CUR","memo":""}' --permission currency@active -j
```


```shell
# 查询 currency 账户中CUR tokenr的总量
./cleos get currency balance currency currency

# currency token 转账
./cleos push action currency transfer '{"from":"currency","to":"eosio","quantity":"200.0000 CUR","memo":"my first transfer"}' --permission currency@active -j

# 查询余额 CUR
./cleos get currency balance currency eosio
./cleos get currency balance currency currency

# EOS token 转账有问题，转账可以成功，余额查询为空，原来这是EOS的一个BUG，
#eosio 给currency 转账 EOS币后，currency余额查不到，eosio有两个余额
#然后currency 再给eosio转帐0个EOS，再查余额，一切正常。
./cleos transfer eosio currency 100000000 -j

# 查询余额 EOS，eosio账户会有两个余额，currency账户是空的
./cleos get currency balance eos eosio
./cleos get currency balance eos currency

# 执行这个回转0，后面查询就正确了
./cleos transfer currency eosio  0 -j
# 查询余额 EOS 一切正常
./cleos get currency balance eos eosio
./cleos get currency balance eos currency
```

如果还是不正常，可能需要修改chain_plugin.cpp文件  321行开始，get_currency_balance修改成下面这样

```java
vector<asset> read_only::get_currency_balance( const read_only::get_currency_balance_params& p )const {
   vector<asset> results;
   if (p.code.to_string() == "eos") {
     const auto& d = db.get_database();
     auto &kvi = d.get_index<contracts::key_value_index>().indices();
     for (auto ite = kvi.begin(); ite != kvi.end(); ++ite) {
       if (ite->id._id > 1){
         if (p.account == ite->payer){
           share_type balance;
           fc::datastream<const char *> ds(ite->value.data(), ite->value.size());
           fc::raw::unpack(ds, balance);
           auto cursor = asset(balance, symbol(ite->primary_key));
           if (cursor.symbol_name() == "EOS"){
             results.emplace_back(balance, symbol(ite->primary_key));
           }
           //ilog("account:${payer}",("payer",p.account));
           //ilog("${payer}:${cursor};id:${id};t_id:${t_id}",
           //   ("payer",ite->payer)("cursor",cursor)("id",ite->id._id)("t_id",ite->t_id._id));
         }
       }
     }
   } else {
     walk_table<contracts::key_value_index, contracts::by_scope_primary>(p.code, p.account, N(accounts), [&](const contracts::key_value_object& obj){
       share_type balance;
       fc::datastream<const char *> ds(obj.value.data(), obj.value.size());
       fc::raw::unpack(ds, balance);
       auto cursor = asset(balance, symbol(obj.primary_key));

       if( !p.symbol || cursor.symbol_name().compare(*p.symbol) == 0 ) {
         results.emplace_back(balance, symbol(obj.primary_key));
       }

       // return false if we are looking for one and found it, true otherwise
       return !p.symbol || cursor.symbol_name().compare(*p.symbol) != 0;
     });
   }
   return results;
}
```


```shell
# 下面的命令的注册矿工、押注、投票、使矿工开始工作，并撤销押注，矿工停止工作的命令，只是命令实例
# 可以写脚本生成52个账户，26个投票，另外26个注册成producer
# 然后 投票账户押注
# 最后一对一投票，可实现21个矿工轮流出块。
# 如需笔者提供脚本，可以@我 caokun_8341@sina.com


# 创建三个账户，一个用来投票vo.a，另外两个是矿工 bp.a,bp.b
# 先导入三对私钥，用来创建三个账户,使用默认钱包
# 导入bp.a的私钥
./cleos wallet import 5KXWvLDQnqyDkGwSNhpJc9rWdYdxGHMXSCfyRsaezfWhNxGT8ag
./cleos wallet import 5JaWKNP1uyfxNey6U2T1rJ1GuqakNQCeJ7pmsomL2vf67HQS9nE

# 导入bp.b的私钥
./cleos wallet import 5J8YadJ1Y2HhSRamPMDUNQnSM95BfDiCBiEf1NrAooi9LQAsDUC
./cleos wallet import 5JXUfyVAmMh6sSxNXAdLzk3rDwjyMh8cspWcpD7rPSvHy4c3QFV

# 导入vo.a的私钥
./cleos wallet import 5JPkqi92pc7B6CH2Hftn27BvA6cu1vswUKANLWJ2Fobjg9X3CCq
./cleos wallet import 5JbioArDrpG6EnUp6qtgxHUFTTjBNYiLsr3wzPuCxBzDy7U3B3y

# 创建三个账户 bp.a  bp.b  vo.a
./cleos create account eosio bp.a EOS8dtXWZQWqSv4mk3WPrpMKGywA5pBY1MWxbpSGSLVvV4k98kvxs EOS7fhfs1j5BBQC9bsM5c8c2WZy6NPG6mbAGMuuYJCnKNHfb3vrND
./cleos create account eosio bp.b EOS7PcVKaTP9sLPv8Hff55y3qiLyNtz1vtDtrSdF2DGigmwiMgqCp EOS5nqrTB9Tj4B41AUrHPUf7sfnN1r25b6v6qtYM57n74bpqwXd3e
./cleos create account eosio vo.a EOS6aGsSSs36TytGdwaSb6WN6Jayr5uL5FqqJi97oDSSz7puqNtCu EOS7zKxE4HNRboD2SeZHUcKGbinquBim4FtAqoWJtcQFPQjiZ83bh

# 给三个账户发放EOS token
./cleos push action eosio issue '{"to":"bp.a","quantity":"10000.0000 EOS"}' --permission eosio@active
./cleos push action eosio issue '{"to":"bp.b","quantity":"10000.0000 EOS"}' --permission eosio@active
./cleos push action eosio issue '{"to":"vo.a","quantity":"10000.0000 EOS"}' --permission eosio@active

# bp.a  bp.b 注册矿工 producer_key 需要把对应账户的producer_key active 公钥转换成16进制编码
./cleos push action eosio regproducer  '{"producer":"bp.a","producer_key":"00036e0d69f6d267173ff478815c077bc8f34b24bc687e81f4dec22c4dd6e2fb37b3","prefs":{"base_per_transaction_net_usage":102,"base_per_transaction_cpu_usage":102,"base_per_action_cpu_usage":102,"base_setcode_cpu_usage":102,"per_signature_cpu_usage":102,"per_lock_net_usage":102,"context_free_discount_cpu_usage_num":3,"context_free_discount_cpu_usage_den":102,"max_transaction_cpu_usage":1000002,"max_transaction_net_usage":1000002,"max_block_cpu_usage":10000002,"target_block_cpu_usage_pct":12,"max_block_net_usage":10000002,"target_block_net_usage_pct":12,"max_transaction_lifetime":3602,"max_transaction_exec_time":9902,"max_authority_depth":8,"max_inline_depth":6,"max_inline_action_size":4098,"max_generated_transaction_count":12,"percent_of_max_inflation_rate":52,"storage_reserve_ratio":102}}' --permission {eosio@active,bp.a@active}
./cleos push action eosio regproducer  '{"producer":"bp.b","producer_key":"000276dff979ee0ab622c57577e5ce70041eeae33cdf1d4a00803bebc6dbeaaa5a13","prefs":{"base_per_transaction_net_usage":102,"base_per_transaction_cpu_usage":102,"base_per_action_cpu_usage":102,"base_setcode_cpu_usage":102,"per_signature_cpu_usage":102,"per_lock_net_usage":102,"context_free_discount_cpu_usage_num":3,"context_free_discount_cpu_usage_den":102,"max_transaction_cpu_usage":1000002,"max_transaction_net_usage":1000002,"max_block_cpu_usage":10000002,"target_block_cpu_usage_pct":12,"max_block_net_usage":10000002,"target_block_net_usage_pct":12,"max_transaction_lifetime":3602,"max_transaction_exec_time":9902,"max_authority_depth":8,"max_inline_depth":6,"max_inline_action_size":4098,"max_generated_transaction_count":12,"percent_of_max_inflation_rate":52,"storage_reserve_ratio":102}}' --permission {eosio@active,bp.b@active}

# vo.a 押注
./cleos push action eosio delegatebw '{"from":"vo.a","receiver":"vo.a","stake_net":"100.0000 EOS","stake_cpu":"100.0000 EOS","stake_storage":"0.0000 EOS"}' --permission vo.a@active

# vo.a给 bp.a 投票
./cleos push action eosio voteproducer '{"voter":"vo.a","proxy":"","producers":["bp.a"]}' --permission vo.a@active

# vo.a给 bp.b 投票
./cleos push action eosio voteproducer '{"voter":"vo.a","proxy":"","producers":["bp.b"]}' --permission vo.a@active

# 查vo.a的余额
./cleos get currency balance eos vo.a

#减少押注
./cleos push action eosio undelegatebw '{"from": "vo.a","receiver": "vo.a","unstake_net": "299.0000 EOS","unstake_cpu": "299.0000 EOS","unstake_bytes": 0}' --permission vo.a@active

# 查vo.a的余额
./cleos get currency balance eos vo.a

#减少押注
./cleos push action eosio undelegatebw '{"from": "vo.z","receiver": "vo.z","unstake_net": "1.0000 EOS","unstake_cpu": "1.0000 EOS","unstake_bytes": 0}' --permission vo.z@active

# 取消矿工注册
./cleos push action eosio unregprod '{"producer":"bp.a"}' --permission {eosio@active,bp.a@active}

./cleos push action eosio unregprod '{"producer":"bp.b"}' --permission {eosio@active,bp.b@active}
```

结论性总结：
1. 首先需要创建个钱包，才可以做其它操作
2. 如果要进行EOS token 发行,转账,超级节点注册，投票人押注，投票等操作，部署eosio.bios和eosio.system这两个系统合约是必要前提
3. 需要生成密钥，并导入到钱包，才可创建账户
4. procucer 工作前要现创建账户，再注册成为producer，可以脚本实现批量生成密钥和创建账户
5.投票人投票前需要先押注
6. 使用相关命令使21个producer正常工作
7. 在prodcuer满21个工作的时候，投票人撤销下注，对应的producer停止工作；在只有一个producer工作的情况下，投票人撤销下注，prodcuer的工作不会停止
8. 21个producer 工作过程中，按照对应投票人押注的多少，确定打块顺序，一人12打个块，打完就走，等待下轮
9. 投票人可以随时增加押注，也可以减少押注，减少的部分代币不会立即回到账户
10. 随着投票人押注多少的变化打块顺序会有相应调整
11. 如果备选的producers中有某一个支持者（投票人）的押注数量超过目前当选的21个producer中的至少一个prodcuer,被超过的那个当前producer会被超过他的备选producer替换
12. 总之投票可以是一直在进行的，当前21个producer,随时有可能被替换
13. 有人说EOS 只有21个超级节点产生区块数据，并非真正的去中心化，其实我觉得：投票不止，替换不息。

## 本文地址：https://eosfans.io/topics/418

