
## 前言
> 在此表示抱歉，最近确实忙的晕头转向，之前说的不旧之后结果推到了现在，但是今天终于抽出时间看了一下，结果还是比较何意的，虽然晚了点。

## 话不多说，进入正题：
关于之前的操作笔者在这里就不多说了大家可以在2.0版本看到详细的说明https://eosfans.io/topics/62

### 比之2.0,3.0的代码还是做了很多的优化，代码很少，在这里不做过多说明，都在代码里，介绍在代码中间穿插。重点看代码演示。

## .cpp 文件
.cpp代码如下：
```
/**
 *  @eosfans.io
 *  @author lome
 */

#include <lover/lover.hpp>


namespace eosio{

//定义函数
void love::say(account_name lover, string letter)
{
    //验证权限
    require_auth( _self );
    // 定义loves类型
    loves lovetable( _self, lover);
    //查询lover是否存在， 如果存在则assert
    auto exit = lovetable.find(lover);
    eosio_assert(exit == lovetable.end(), "You love has been said!");
    //构造loves
    lovetable.emplace( _self, [&]( auto& s ) {
        s.lover = lover;
        s.letter = letter;
    });

}
}
/// namespace eosio
```
代码逻辑非常的简单，就不在多加说明。
## .hpp 文件
.hpp代码如下：
```
/**
 *  @eosfans.io
 *  @author lome
 */

#include <eosiolib/eosio.hpp>
#include <string>

namespace eosio {

   using std::string;
   //所有智能合约继承contract类
   class love : public contract {
      public:
         //love类构造函数
         love( account_name self ):contract(self){}
         // @abi action 成员函数声明
         void say(account_name lover, string letter);

      private:
         // @abi table 私有数据成员声明
         struct lovel {
            account_name    lover;
            string          letter;
            //私有函数成员
            account_name primary_key()const { return lover; }
         };
         //数据库类型定义
         typedef eosio::multi_index<N(love), lovel> loves;
   };

} /// namespace eosio
EOSIO_ABI( eosio::love, (say) )
```
## .abi 文件
```
{
    "types": [],
    "structs": [{
        "name": "lovel",
        "base": "",
        "fields": [
            {"name": "lover", "type": "account_name"},
            {"name": "letter","type": "string"}
            ]
        },{
            "name": "say",
            "base": "",
            "fields": [
                {"name": "lover","type": "account_name"},
                {"name": "letter","type": "string"}
                ]
        }
        ],
    "actions": [
        {"name": "say","type": "say"}
        ],
    "tables": [{
        "name": "love",
        "type": "lovel",
        "index_type": "i64",
        "key_names" : ["lover"],
        "key_types" : ["uint64"]
    }
        ],
    "clauses": []
}
```
## 代码执行演示
在发布合约之前讲一下我的合约书写步骤：
1. 通过vscode书写合约，vscode的使用社区里面已经提供，这里不在多说。
2. 通过vscode来build项目，build完成以后`build/contracts`目录下会多一个`lover`合约。这个合约是直接可以使用的，合约目录如下所示：

![](https://eosfans-static.strahe.com/photo/2018/d084ad28-e266-46bd-b3b9-68634054f3c6.png?x-oss-process=image/resize,w_1920)


### 发布合约
发布合约通过通常的发布合约命令即可,具体代码如下：
```
cleos set contract lome lover/
```
结果如下：

![](https://eosfans-static.strahe.com/photo/2018/c937d651-576b-4337-a8a8-f1eb42aa0a28.png?x-oss-process=image/resize,w_1920)


### 合约使用
本合约也是只定义了一个Action和一个table，后续如果有需求的话可以添加，合约使用代码如下:
```
cleos push action lome say '{"lover":"eostea","letter":"i love you"}' -p lome
```
结果如下：
![](https://eosfans-static.strahe.com/photo/2018/7decf2e4-dfdb-4471-b81a-9321e51c688f.png?x-oss-process=image/resize,w_1920)

如果你再执行一次，智能合约限制只能对同一个人执行一次say操作。会报错结果如下：



![](https://eosfans-static.strahe.com/photo/2018/332c2893-28a3-4177-bbaf-a72d3fb7cb08.png?x-oss-process=image/resize,w_1920)

### table 数据
执行一下命令，你讲看到你的数据：
```
cleos get table lome eostea love
```
结果如下：

![](https://eosfans-static.strahe.com/photo/2018/1f1818ed-42bd-41ff-b2a2-06b0fae388c3.png?x-oss-process=image/resize,w_1920)



### 测试环境
已将合约发布到`party`测试网络，有兴趣的同学可以去测试网络浏览器(party.eosmonitor.io)看看一下合约详情：

１．部署合约的账户lome


## 本文地址：https://eosfans.io/topics/407

