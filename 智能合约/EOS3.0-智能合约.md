 # EOSIO 智能合约
<!-- TOC -->

- [EOSIO 智能合约](#eosio-%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6)
  - [EOSIO 智能合约介绍](#eosio-%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%E4%BB%8B%E7%BB%8D)
    - [必须的背景知识](#%E5%BF%85%E9%A1%BB%E7%9A%84%E8%83%8C%E6%99%AF%E7%9F%A5%E8%AF%86)
    - [EOSIO 智能合约基础知识](#eosio-%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86)
  - [智能合约文件](#%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%E6%96%87%E4%BB%B6)
    - [hpp](#hpp)
    - [cpp](#cpp)
    - [wast](#wast)
    - [abi](#abi)
  - [调试智能合约](#%E8%B0%83%E8%AF%95%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6)
    - [方法](#%E6%96%B9%E6%B3%95)
    - [打印](#%E6%89%93%E5%8D%B0)
    - [例子](#%E4%BE%8B%E5%AD%90)

<!-- /TOC -->

## EOSIO 智能合约介绍

### 必须的背景知识

**C / C++ 经验**

基于EOSIO的区块链使用 [WebAssembly](http://webassembly.org/) (WASM) 执行用户生成的应用程序和代码。WASM是一项新兴的网络标准，得到了谷歌，微软，苹果等公司的广泛支持。目前，用于构建编译为WASM的应用程序的最成熟工具链是使用 C/C++ 编译器的 [clang/llvm](https://clang.llvm.org/)。

其他第三方开发的工具链包括：Rust，Python和Solidity。虽然这些其他语言看起来可能更简单，但它们的性能可能会影响你可以构建的应用程序的规模。我们预计 C++ 将成为开发高性能和安全智能合约的最佳语言，并计划在可预见的将来使用 C++。

**Linux / Mac OS 经验**

EOSIO软件支持以下环境:
- Amazon 2017.09 and higher
- Centos 7
- Fedora 25 and higher (Fedora 27 recommended)
- Mint 18
- Ubuntu 16.04 (Ubuntu 16.10 recommended)
- MacOS Darwin 10.12 and higher (MacOS 10.13.x recommended)

**命令行知识**

与EOSIO一起提供的各种工具，要求你具有基本的命令行知识才能与之交互。

### EOSIO 智能合约基础知识

**交互模型**

EOSIO智能合约以动作（actions）和共享内存数据库访问的形式彼此交互，
例如，合约可以读取其他合约数据库的状态，只要它包含在具有异步事务的读取范围内即可。
异步通信可能会导致资源限制算法会处理的垃圾邮件（spam）。
在合约中可以定义两种通信模式：

- **内联**。内联保证与当前交易一起执行或展开; 无论成功或失败，都不会通知任何通知。内联与原有交易拥有相同的作用范围和权限。

- **延期**。延期交互将由出块人酌情决定如何执行; 可以传递交互结果或者可以简单地超时。延期交互可以有不同的作用范围，并带有发送它们的合约指定的权限。

**动作 vs 交易**

一个动作表示单个操作，而一个交易是一个或多个动作的集合。合约和账户以动作的形式进行交流。动作可以单独发送，也可以组合的形式发送，如果它们打算作为一个整体来执行。

*1个动作的交易*.

```json
{
  "expiration": "2018-04-01T15:20:44",
  "region": 0,
  "ref_block_num": 42580,
  "ref_block_prefix": 3987474256,
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
      "data": "00000000007015d640420f000000000004454f5300000000046d656d6f"
    }
  ],
  "signatures": [
    ""
  ],
  "context_free_data": []
}
```

*多动作交易*, 这些动作会同时成功或失败.
```json
{
  "expiration": "...",
  "region": 0,
  "ref_block_num": ...,
  "ref_block_prefix": ...,
  "net_usage_words": ..,
  "kcpu_usage": ..,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "...",
      "name": "...",
      "authorization": [{
          "actor": "...",
          "permission": "..."
        }
      ],
      "data": "..."
    }, {
      "account": "...",
      "name": "...",
      "authorization": [{
          "actor": "...",
          "permission": "..."
        }
      ],
      "data": "..."
    }
  ],
  "signatures": [
    ""
  ],
  "context_free_data": []
}
```

**动作名称限制**

动作类型实际上是 **base32编码的64位整数** 。这意味着对于前12个字符它们仅限于字符a-z，1-5和 '.' 。如果有第13个字符，则它仅限于前16个字符（'.'和a-p）。

**交易确认**

接收交易哈希并不意味着交易已被确认，它只意味着节点认为没有错误并接受了它，这也意味着其他出块人很可能会接受它。

通过确认，你应该在交易历史中看到包含确认交易所属区块的交易。

## 智能合约文件

为了简单起见 ，我们创建了一个名为 **[eosiocpp](https://github.com/EOSIO/eos/wiki/Programs-&-Tools#eosiocpp)** 的工具，可以用来启动一个新的合约。eosiocpp也将为你创建3个智能合约文件，并提供基本框架。

```base
$ eosiocpp -n ${contract}
```

上面的命令将新建一个空项目，项目目录下有3个文件:
```base
${contract}.abi ${contract}.hpp ${contract}.cpp
```

### hpp

`${contract}.hpp` 是被 `.cpp` 引用的，包含变量，常量和函数定义的头文件。

### cpp

`${contract}.cpp` 文件是包含智能合约功能函数的源文件。

如果你使用 `eosiocpp` 工具生成 `.cpp` 文件，生成的 .cpp 文件与下面的相似:

```cpp
#include <${contract}.hpp>

/**
 *  The init() and apply() methods must have C calling convention so that the blockchain can lookup and
 *  call these methods.
 */
extern "C" {

    /**
     *  This method is called once when the contract is published or updated.
     */
    void init()  {
       eosio::print( "Init World!\n" ); // Replace with actual code
    }

    /// The apply method implements the dispatch of actions to this contract
    void apply( uint64_t code, uint64_t action ) {
       eosio::print( "Hello World: ", eosio::name(code), "->", eosio::name(action), "\n" ); 
    }

} // extern "C"
```
在这个例子中，你可以看到有两个函数， `init` 和 `apply`。
他们所做的只是记录动作，不做其他检查。只要出块人允许，任何人都可以随时提供任何操作。在没有任何所需的签名的情况下，合约将按照消耗的带宽收费。

**init**

`init` 函数只会在初始部署时执行一次。用来初始化智能合约的变量，例如，代币合约的代币发行量。

**apply**

`apply` 是动作处理器，它监听所有传入的动作并根据函数内的逻辑作出反应。该 apply 函数需要两个输入参数，`code` 和 `action`。

**代码过滤器**

为了应对特定的动作，`apply` 函数按以下方式编写。你也可以通过省略代码过滤器来编写对通用动作的响应。

```cpp
if (code == N(${contract_name}) {
    // your handler to respond to particular action
}
```

你也可以在代码块中定义对各个操作的响应。

**动作过滤器**

为了响应某个特定动作，`apply` 函数按照以下方式编写。这通常与代码过滤器结合使用。

```cpp
if (action == N(${action_name}) {
    //your handler to respond to a particular action
}
```

### wast

任何要部署到EOSIO区块链的程序都必须编译为WASM格式。这是区块链接受的唯一格式。

准备好CPP文件后，可以使用 `eosiocpp` 工具将其编译为WASM（.wast）的文本版本。


```base
$ eosiocpp -o ${contract}.wast ${contract}.cpp
```
### abi

应用程序二进制接口（ABI）是一种基于JSON的描述，介绍如何将用户动作在JSON和二进制表达之间转换。ABI还介绍了如何将数据库状态转换为JSON或从JSON转换数据库状态。通过ABI描述了智能合约，开发人员和用户就可以通过JSON无缝地与你的合约进行交互。

ABI文件可以通过使用 `eosiocpp` 工具从 `.hpp` 文件生成：

```base
$ eosiocpp -g ${contract}.abi ${contract}.hpp
```

以下是框架合约ABI的示例：
```json
{
  "types": [{
      "new_type_name": "account_name",
      "type": "name"
    }
  ],
  "structs": [{
      "name": "transfer",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "quantity": "uint64"
      }
    },{
      "name": "account",
      "base": "",
      "fields": {
        "account": "name",
        "balance": "uint64"
      }
    }
  ],
  "actions": [{
      "action": "transfer",
      "type": "transfer"
    }
  ],
  "tables": [{
      "table": "account",
      "type": "account",
      "index_type": "i64",
      "key_names" : ["account"],
      "key_types" : ["name"]
    }
  ]
}
```

你会注意到这个ABI定义了一个 `transfer` 类型的动作 `transfer` 。这告诉EOSIO，当 `${account}->transfer` 被看到时，交易的负载（payload）类型是 `transfer` 。动作类型 `transfer` 在 `structs` 数组中被定义，`structs` 数组对象中，`name` 属性的值为 `transfer`。

```json
...
  "structs": [{
      "name": "transfer",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "quantity": "uint64"
      }
    },{
...
```

该ABI 有好几个字段，包括 `from`, `to` 和 `quantity` 。
这些字段有相应的类型  `account_name`, 和 `uint64` 。
`account_name` 是一个内置的类型使用 `uint64` 来表示 base32 字符串。
要详细了解可用的内置类型，请点击[此处](https://github.com/EOSIO/eos/blob/master/libraries/chain/contracts/abi_serializer.cpp)。

```json
{
  "types": [{
      "new_type_name": "account_name",
      "type": "name"
    }
  ],
...
```

在上面的 `types` 数组中，我们为已存在类型定义了一个别名列表。在这里，我们定义 `name` 为 `account_name` 的一个别名。

## 调试智能合约

为了能够调试你的智能合约，你需要设置本地 nodeos 节点。这个本地 nodeos 节点可以作为独立的私人测试网或作为公共测试网（或官方测试网）的扩展来运行。

当你首次创建智能合约时，建议先在私人测试网上测试并调试你的智能合约，因为你完全控制了整个区块链。这使你可以拥有无​​限量你所需要的eos，你可以随时重置区块链状态。当准备发布到生产环境时，可以通过将本地节点连接到公共测试网（或官方测试网）来在公共测试网（或官方测试网）上进行调试，以便你可以在本地节点中看到测试网的日志。

下面的教程，将在私人测试网上进行调试。

如果你尚未设置自己的本地节点，请按照 [启动指南](https://github.com/eosfansio/eos-tutorials/blob/master/EOS3.0-%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97.md) 进行操作。默认情况下，除非你按照 [Testnet指南](Testnet%3A%20Public) 中所述修改config.ini文件以便与公共testnet（或官方testnet）节点连接，否则你的本地节点将仅运行在私有测试网络中。

### 方法
用于调试智能合约的主要方法是 **穴居人调试**（**Caveman Debugging**），我们利用打印功能来检查变量的值并检查合约的流程。在智能合约中打印可以通过打印API ([C](https://github.com/EOSIO/eos/blob/master/contracts/eoslib/print.h) 和 [C++](https://github.com/EOSIO/eos/blob/master/contracts/eoslib/print.hpp)) 完成。C++ API是C API的封装器，因此大多数情况下我们只会使用C++ API。

### 打印
打印 C API支持你可以打印的以下数据类型：
- prints - 一个带null终止符的字符数组（字符串）
- prints_l - 给定大小的任何字符数组（字符串）
- printi - 64位无符号整数
- printi128 - 128位无符号整数
- printd - 编码为64位无符号整数的浮点类型
- printn - 编码为64位无符号整数的base32字符串
- printhex - 给出二进制数据及其大小的十六进制

打印 C++ API通过重写print() 函数来封装一些上述C API，因此用户不需要确定他需要使用哪种特定的打印功能。

打印 C++ API支持：
- 一个带null终止符的字符数组（字符串）
- 整数（128位无符号，64位无符号，32位无符号，有符号，无符号）
- 编码为64位无符号整数的base32字符串
- 具有 print() 方法的结构体


### 例子
我们来写一个新的合约作为调试的例子
- debug.hpp
```cpp
#include <eoslib/eos.hpp>
#include <eoslib/db.hpp>

namespace debug {
    struct foo {
        account_name from;
        account_name to;
        uint64_t amount;
        void print() const {
            eosio::print("Foo from ", eosio::name(from), " to ",eosio::name(to), " with amount ", amount, "\n");
        }
    };
}
```
- debug.cpp
```cpp
#include <debug.hpp>

extern "C" {

    void init()  {
    }

    void apply( uint64_t code, uint64_t action ) {
        if (code == N(debug)) {
            eosio::print("Code is debug\n");
            if (action == N(foo)) {
                 eosio::print("Action is foo\n");
                debug::foo f = eosio::current_message<debug::foo>();
                if (f.amount >= 100) {
                    eosio::print("Amount is larger or equal than 100\n");
                } else {
                    eosio::print("Amount is smaller than 100\n");
                    eosio::print("Increase amount by 10\n");
                    f.amount += 10;
                    eosio::print(f);
                }
            }
        }
    }
} // extern "C"
```
- debug.hpp
```cpp
{
  "structs": [{
      "name": "foo",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "amount": "uint64"
      }
    }
  ],
  "actions": [{
      "action_name": "foo",
      "type": "foo"
    }
  ]
}

```

让我们部署它并发送一条消息给它。假设你已经创建 `debug` 帐户并在将私钥导入你的钱包中。
```bash
$ eosiocpp -o debug.wast debug.cpp
$ cleos set contract debug debug.wast debug.abi
$ cleos push message debug foo '{"from":"inita", "to":"initb", "amount":10}' --scope debug
```

当你检查你的本地 `nodeos` 节点日志时，你将在发送上述消息后看到以下行。
```
Code is debug
Action is foo
Amount is smaller than 100
Increase amount by 10
Foo from inita to initb with amount 20
```
这里，你可以确认你的消息正在进入正确的控制流程并且金额已正确更新。你可能会看到上述消息至少2次，这很正常，
因为每个交易在验证、块生成和块应用阶段都会被执行。
