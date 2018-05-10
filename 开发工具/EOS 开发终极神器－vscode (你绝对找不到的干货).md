# EOS开发终极神器－vscode(你绝对找不到的干货)
> 声明：本文由EOS中文社区，原创首发，转载请注明原文地址，谢谢。

前言：最近一直苦于EOS开发没有好用的IDE，用了很多，试了很多，都让人觉得有些差强人意。于是乎笔者在经过，长时间的查找实践中，终于找到了eos开发终极神器－vscode。当然这个只是笔者经过测试开发尝试后的一家之言。

话不多说下面进入正题。

## vscode　安装

- 下载
  
大家开一去官网下载vscode：https://code.visualstudio.com/Download。下载安装都很方便。

- 安装
  - win
    windows下的安装，相信大家都能顺利完成
  - linux
    linux下，官网下载的都是linux可执行文件.deb,.rpm。直接打开安装即可，也很方便。
  - Mac os下，这个本人没有试过，应该安装很简单。

## 配置
vscode安装起来非常简单，配置起来也非常容易。

首先，EOS是用c++开发的，所以打开vscode之后先装c++ 插件：
```
ms-vscode.cpptools
```
这个插件是必须的，其他的，也有很多插件非常有用，大家可以自己积极去发现。

## 运行测试

首先打开本地的eos，然后会看到最下面有许多选项，如图所示：
![](https://eosfans-static.strahe.com/photo/2018/3280d6d4-ccaf-48a0-bafb-1d04a966d733.png?x-oss-process=image/resize,w_1920)

### build all 测试
1. 点击build：后的`[all]`，可以选择构建的区域.选择`[all]`可以构建整个eos项目.
2. 点击build[all]进行构建。
如图所示：

![](https://eosfans-static.strahe.com/photo/2018/a950534f-0ea8-4987-9abf-e925c5c93df6.png?x-oss-process=image/resize,w_1920)

### 智能合约构建
１．创建智能合约，这里我用的是｀hello｀的例子，来说明。
首先是`hello.cpp`。
```
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>

//用eosio命名空间
using namespace eosio;

//所有的智能合约都继承自contract类
class hello : public eosio::contract {

  public:
      using contract::contract;

      /// @abi action
      void hi( account_name user ) {
         print( "Hello, ", name{user} );
      }

};
EOSIO_ABI( hello, (hi) )
```

`hello.abi`:
```
{
  "types": [],
  "structs": [{
      "name": "hi",
      "base": "",
      "fields": [{
          "name": "user",
          "type": "account_name"
        }
      ]
    }
  ],
  "actions": [{
      "name": "hi",
      "type": "hi"
    }
  ],
  "tables": []
}
```
`CMakeLists.txt`:
```
file(GLOB ABI_FILES "*.abi")
configure_file("${ABI_FILES}" "${CMAKE_CURRENT_BINARY_DIR}" COPYONLY)

add_wast_executable(TARGET hello
  INCLUDE_FOLDERS "${STANDARD_INCLUDE_FOLDERS}"
  LIBRARIES libc++ libc eosiolib
  DESTINATION_FOLDER ${CMAKE_CURRENT_BINARY_DIR}
)
```
然后在｀eos/contracts｀目录下的`CMakeLists.txt`中加入`hello`:
添加命令：

```
add_subdirectory(hello)
```
然后build[all].后根据cmake文件来自动构建。构建完成以后，你可以在`build/contracts`目录下，看到构建好的`hello`，如图所示：

![](https://eosfans-static.strahe.com/photo/2018/42310465-6425-44de-8739-3fefa1259633.png?x-oss-process=image/resize,w_1920)


构建完成以后，你就可以在build后选择[hello],单独进行编译。


## 代码提示
在左下角点击设置，加入以下配置,并且保存，就会出现代码提示了：
```
"[cpp]": {
    "editor.autoIndent": true,
    "editor.quickSuggestions":true
  },
"[c]": {
    "editor.quickSuggestions":true
},
"cmake-tools-helper.auto_set_cpptools_target": true,
```
代码提示效果如图所示：
![](https://eosfans-static.strahe.com/photo/2018/c4d3bdda-deac-4463-bef1-10a477af923f.png?x-oss-process=image/resize,w_1920)

## vscode　Debug 
debug方式很简单，选择`debug:`后,你所debug的代码，打上断点，然后点击`debug`就可以调试了，效果如图所示：
![](https://eosfans-static.strahe.com/photo/2018/90b91cb7-d476-4e81-b072-31f37e3703f8.png?x-oss-process=image/resize,w_1920)
![](https://eosfans-static.strahe.com/photo/2018/7843a239-5487-479f-afd0-9cdf63a563cd.png?x-oss-process=image/resize,w_1920)

一切都是如此简单，一切都是如此便捷。
