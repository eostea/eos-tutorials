# EOSIO 智能合约数据库演练

> 前几天翻译了一篇关于EOS智能合约数据库的内容，今天来演示一下数据库的使用方法。

## 目录
1. 增
2. 查
3. 改
4. 删


## 新增
新增内容往往用到`emplace`构造函数，来进行数据库对象的新增。

.cpp
```
void test_da::create(account_name user, string title, string content)
    {
        require_auth( user ); //验证权限
        das datable( _self, user);　//定义数据库对象
        datable.emplace(user, [&]( da & d){
            d.title = title;
            d.content = content;
            d.post_id = datable.available_primary_key();
            d.poster = user;
        });　//数据库内容创建
    }
```
这里需要注意的是：   

1. 定义数据库对象，　其中第一个参数是合约的拥有者_self，第二个变量就是数据库的`payer`，也就是数据库是谁的，数据库存储在谁的账户下。
2. emplace函数接收两个参数，一个`payer`,和一个`lamada`表达式。这个结构是固定的。

那么我们现在来看一下我们的`test_da`类是怎么定义的。

.hpp　
```
class test_da : public contract {
     public:
           test_da( account_name self ):contract(self){}

           // @abi action
           void create(account_name user, string title, string content);

     private:
           // @abi table data i64
           struct da {
                 uint64_t     post_id;
                 account_name poster;
                 string       title;
                 string       content;

                 uint64_t primary_key()const { return post_id; }
                 account_name get_poster() const { return poster; }

                 EOSLIB_SERIALIZE(da, (post_id)(poster)(title)(content))
           };
           typedef eosio::multi_index<N(data), da, indexed_by<N(byposter), const_mem_fun<da, account_name, &da::get_poster>> > das;
  };
} /// namespace eosio
```

1. 所有的智能合约都继承自`contract`合约。`test_da( account_name self ):contract(self){}
`是`test_da`合约的构造函数。
2. 下面是对`create`函数的声明。
3. 接下来是对数据字段的定义。这里我们定义了数据结构`da`.
4. `primary_key`函数是定义主键的函数。　
5. 接下来我们定义了辅助主键返回`poster`。
6. `EOSLIB_SERIALIZE`宏的第一个参数是数据结构，其他参数是数据结构中的数据成员。
7. `typedef`我们在这里定义了一个名字为`das`的类型，它用来定义数据库对象。这里我们定义的是一个具有主键及一个辅助键的数据库对象。
.abi　
abi 非常重要，错误的`abi`会导致合约执行失败。
```
{
  "types": [],
  "structs": [{
      "name": "da",
      "base": "",
      "fields": [{
          "name": "post_id",
          "type": "uint64"
        },{
          "name": "poster",
          "type": "account_name"
        },{
          "name": "title",
          "type": "string"
        },{
          "name": "content",
          "type": "string"
        }
      ]
    },{
      "name": "create",
      "base": "",
      "fields": [{
          "name": "user",
          "type": "account_name"
        },{
          "name": "title",
          "type": "string"
        },{
          "name": "content",
          "type": "string"
        }
      ]
    }}
  ],
  "actions": [{
          "name": "create",
          "type": "create",
        }
  ],
  "tables": [{
      "name": "data",
      "index_type": "i64",
      "key_names": [
        "post_id"
      ],
      "key_types": [
        "uint64"
      ],
      "type": "da"
    }
  ],
  "ricardian_clauses": []
}
```
接下来来演示一下:
1. 发布合约
```
cleos set contract eosio test_da/
Reading WAST/WASM from test_da/test_da.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 3d6f04278617d3807fe876a33057f1155acf9c9e5a392ac6ed8ad51e79506009  6752 bytes  24679 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ad011a60037f7e7e0060057f7e7e7f...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"da","base":"","fields":[{"name":"post_id","...
```
2. 创建数据
```
cleos push action eosio create '{"user":"eosio","title":"first","content":"create a first one"}' -p eosio
executed transaction: 830057f270fa499b1d61b82e80ad8cda1774cdc1786c1e786f558a3e0a48974c  216 bytes  17229 us
#         eosio <= eosio::create                {"user":"eosio","title":"first","content":"create a first one"}
```
下面我们来查一下数据表：
```
cleos get table eosio eosio data
{
  "rows": [{
      "post_id": 0,
      "poster": "eosio",
      "title": "first",
      "content": "create a first one"
    }
  ],
  "more": false
}
```
创建信息成功，那么我们用别的账号创建会怎样呢？
```
cleos push action eosio create '{"user":"eostea","title":"eostea first","content":"eostea create a first one"}' -p eostea
executed transaction: 8542a87e563a9c62b7dbe46ae09ccf829c7821f8879167066b658096718de148  232 bytes  2243 us
#         eosio <= eosio::create                {"user":"eostea","title":"eostea first","content":"eostea create a first one"}
```
查看数据表：
```
cleos get table eosio eostea data
{
  "rows": [{
      "post_id": 0,
      "poster": "eostea",
      "title": "eostea first",
      "content": "eostea create a first one"
    }
  ],
  "more": false
}
```
到这里，相信大家对创建数据已经没有什么疑惑了。

## 查询
对于数据库，最重要的功能就是查询，如果没有查询功能，数据库里的数据就不能呈现，也就没有意义。查询数据库主要分为两方面，一方面是主键查询，一方面是通过二级索引查询。
这里为了使表中数据多样化我会做一些修改，将所有数据都合到一张表中。
我将以上`.cpp`中的`das datable( _self, user);`改为`das datable( _self, _self);`.这样数据都存在合约账户的表中。

### 主键查询
这里我添加了一个方法来查询数据并打印：
```
void test_da::getd(uint64_t post_id){
        das datable(_self, _self);
        auto post_da = datable.find( post_id);
        eosio::print("Post_id: ", post_da->post_id, "  Post_Tile: ", post_da->title.c_str(), " Content: ", post_da->content.c_str());
    }
```
abi文件也做了相应的调整：


执行：
```
cleos push action eosio getd '{"post_id":1}' -p eosio
executed transaction: ac8663235462d947c74542af848cca54a059c3991d193237025da7d4767d6725  192 bytes  1724 us
#         eosio <= eosio::getd                  {"post_id":1}
>> Post_id: 1  Post_Tile: first Content: eosio create a first one
```

### 二级索引查询
添加二级索引查询代码如下：
```
auto poster_index = datable.template get_index<N(byposter)>();
auto pos = poster_index.find( user );

for (; pos != poster_index.end(); pos++)
{
    eosio::print("content:", pos->content.c_str(), " post_id:", pos->post_id, " title:", pos->title.c_str());
}
```
获取二级索引并获取数据，这里我只用了find查询，其他查询就不在一一介绍。

执行`getd`操作：
```
cleos push action eosio getd '{"post_id":2,"user": "eostea"}' -p eosio
executed transaction: 2370e1fb1ee8a581f7321f02fb40645e51269e579d183c33ef470dba0b3afdbc  200 bytes  5403 us
#         eosio <= eosio::getd                  {"post_id":2,"user":"eostea"}
>> Post_id: 2  Post_Tile: eostea first Content: eostea create a first onecontent:eostea create a first one post_id:2 title:eostea first
```
数据库中数据如下：
```
cleos get table eosio eosio data
{
 "rows": [{
     "post_id": 0,
     "poster": "eosio",
     "title": "first",
     "content": "eostea create a first one"
   },{
     "post_id": 1,
     "poster": "eosio",
     "title": "first",
     "content": "eostea create a first one"
   },{
     "post_id": 2,
     "poster": "eostea",
     "title": "eostea first",
     "content": "eostea create a first one"
   }
 ],
 "more": false
}
```
## 更改
更改数据库内容。
这里我们先看一下目前的数据库内容。
```
cleos get table eosio eosio data
{
 "rows": [{
     "post_id": 0,
     "poster": "eosio",
     "title": "first",
     "content": "eostea create a first one"
   },{
     "post_id": 1,
     "poster": "eosio",
     "title": "first",
     "content": "eostea create a first one"
   },{
     "post_id": 2,
     "poster": "eostea",
     "title": "eostea first",
     "content": "eostea create a first one"
   }
 ],
 "more": false
}
```
写一个更改数据的`action`代码如下：
```
void test_da::change(account_name user, uint64_t post_id, string title, string content)
    {
        require_auth(user);
        das datable( _self, user);
        auto post = datable.find(post_id);
        eosio_assert(post->poster == user, "yonghucuowu");
        datable.modify(post, user, [&](auto& p){
            if (title != "")
                p.title = title;
            if (content != "")
                p.content = content;
        });
    }
```
1.　前几行代码已经之前讲解过，现在直接说`modify`方法，他的第一个参数是你查询出的要更改的对象，第二个参数是`payer`,其他的不用多说。
下面我们执行一下命令：
```
cleos push action eosio change '{"user":"eosio","post_id":1,"title":"change","content":"change action"}' -p eosio
executed transaction: 8cb561a712f2741560118651aefd49efd161e3d73c56f6d24cf1d699c265e2dc  224 bytes  2130 us
#         eosio <= eosio::change                {"user":"eosio","post_id":1,"title":"change","content":"change action"}
```
下面我们看一下数据库：
```
cleos get table eosio eosio data
{
  "rows": [{
      "post_id": 0,
      "poster": "eosio",
      "title": "first",
      "content": "eostea create a first one"
    },{
      "post_id": 1,
      "poster": "eosio",
      "title": "change",
      "content": "change action"
    },{
      "post_id": 2,
      "poster": "eostea",
      "title": "eostea first",
      "content": "eostea create a first one"
    }
  ],
  "more": false
}
```
post_id=1的记录已经被改变，说明我们成功了。

## 删除数据
删除数据我又加了一个`action`,如下所示：
```
void test_da::dele(account_name user, uint64_t post_id)
    {
        require_auth(user);
        das datable( _self, user);
        auto post = datable.find(post_id);
        eosio::print(post->title.c_str());

        eosio_assert(post->poster == user, "yonghucuowu");
        datable.erase(post);
    }
```
这里调用了`erase`方法删除数据，参数为一个数据对象。下面我们来看一下执行结果：
```
cleos push action eosio dele '{"user":"eosio","post_id":1}' -p eosioexecuted transaction: 3affbbbbd1da328ddcf37753f1f2f6c5ecc36cd81a0e12fea0c789e75b59714e  200 bytes  2383 us
#         eosio <= eosio::dele                  {"user":"eosio","post_id":1}
```
现在再来看一下我们的数据库：
```
cleos get table eosio eosio data
{
  "rows": [{
      "post_id": 0,
      "poster": "eosio",
      "title": "first",
      "content": "eostea create a first one"
    },{
      "post_id": 2,
      "poster": "eostea",
      "title": "eostea first",
      "content": "eostea create a first one"
    }
  ],
  "more": false
}
```
post_id=1的数据已经被我们删除。

到这里，数据库的增删改查已经讲解完毕。
本文内容为原创内容，如需转载请联系作者。谢谢！！！
