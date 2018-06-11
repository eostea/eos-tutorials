# EOS 帐户权限 3.0
## 关于2.0权限问题请移步https://eosfans.io/topics/28

# 目录
* 查看权限
* 改变权限
* 增加权限
* 删除权限

## 查看权限
有人说查看权限非常简单，不就是看看用户信息嘛！
其实不然，EOS用户的权限是与key相关联的，所以确定你有没有权限的其实是要看你有没有这些公匙对应的私匙：
1. 查看自己帐户的详细信息：

> cleos get account lome

```
{
  "account_name": "lome",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS8FbfG31mJELxUS4Jj9Xv3tsNRzFMg2uP2h9b5hCkvHomsatVYw",
            "weight": 1
          }
        ],
        "accounts": []
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}

```

## 更改授权

### 更改权限的key
初始化的帐户是有两种权限的，这一步我们来给帐户加一个权限(群组)。
1.这里我就拿我在测试的公网上的一个帐户，
初始化的帐户是这样的：

2. 下面讲解一下命令：

> cleos set account permission ${account_name} ${permission} ${JSON}  ${permission}

```
cleos set account permission lome  active '{"threshold": 1, "keys": [{"key": "EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg", "weight": 1}], "accounts": []}' owner
```
更改以后的账号如下：

```
{
  "account_name": "lome",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg",
            "weight": 1
          }
        ],
        "accounts": []
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}
```
>参数说明：
##### threshold权限阀值，权限等于阀值才能获取该权限，否则授权失败
##### keys 是该权限已授权的密匙 ： keys中的key为授权的密匙对的公匙，weight 为授权密匙对的权重 ps： 若阀值是2，权重是1，是不能够完成授权的，操作将失败.
##### accounts 是该权限已授权的帐户： permission是被授权用户的权限这里指的就是lome的active权限，weight指的也是权重。
#### ps：重要：授权用户权限其实跟授权key是一个道理，其实授权帐户权限，实质上就是授权该权限的密匙对权限。
##### 最后owner是权限，只有owner权限才能改变用户的权限

#####  值得一提的是：EOS3.0对权限及权重做了校验。如果我的命令是这样的：

```
cleos set account permission lome active '{"threshold":2,"keys":[{"key":"EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg","weight":1}],"accounts":[]}' owner
```

> 该操作将会失败，因为这个权限根本就达不到伐值，2.0在这里是没有校验的，错误提示如下:

```
Error 3040000: message validation exception
Error Details:
Invalid authority: {"threshold":2,"keys":[{"key":"EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg","weight":1}],"accounts":[]}
```
### 增加权限的授权账户

```
cleos set account permission lome active '{"threshold":2,"keys":[{"key":"EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg","weight":1}],"accounts":[{"permission":{"actor":"eosio","permission":"active"},"weight":1}]}' owner
```
这里更改的是lome账户权限的accounts权限,下面看一下更改完成后我的账户：

```
{
  "account_name": "lome",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 2,
        "keys": [{
            "key": "EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg",
            "weight": 1
          }
        ],
        "accounts": [{
            "permission": {
              "actor": "eosio",
              "permission": "active"
            },
            "weight": 1
          }
        ]
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}

```
在这里我的权重伐值为2，key的权重为1，`eosio@active`的权重为1.那么要使用lome@active权限就需要key和eosio@active所对应的key的权限。

### 增加多个授权key
> 增加授权的命令如下：

```
cleos set account permission lome active '{"threshold":1,"keys":[{"key":"EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg","weight":1},{"key":"EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV","weight":1}],"accounts":[]}' owner
```
> 结果如下：

```
{
  "account_name": "lome",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg",
            "weight": 1
          },{
            "key": "EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV",
            "weight": 1
          }
        ],
        "accounts": []
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS5hepJxGjP3X93wVmXZeBSNavGWeJvwWtev5ak8oARqwrXjehXd",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}

```
### 增加权限多个授权账户
> 增加授权的命令如下：

```
cleos set account permission lome active '{"threshold":1,"keys":[],"accounts":[{"permission":{"actor":"test","permission":"active"},"weight":1},{"permission":{"actor":"eosio","permission":"active"},"weight":1}]}' owner
```
> 结果如下：

```
{
  "account_name": "lome",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [],
        "accounts": [{
            "permission": {
              "actor": "test",
              "permission": "active"
            },
            "weight": 1
          },{
            "permission": {
              "actor": "eosio",
              "permission": "active"
            },
            "weight": 1
          }
        ]
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS5hepJxGjP3X93wVmXZeBSNavGWeJvwWtev5ak8oARqwrXjehXd",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}

```

### 增加多个授权key && 增加权限多个授权账户
> 增加授权的命令如下：

```
cleos set account permission lome active '{"threshold":1,"keys":[{"key":"EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg","weight":1},{"key":"EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV","weight":1}],"accounts":[{"permission":{"actor":"test","permission":"active"},"weight":1},{"permission":{"actor":"eosio","permission":"active"},"weight":1}]}' owner
```
> 结果如下：

```
{
  "account_name": "lome",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg",
            "weight": 1
          },{
            "key": "EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV",
            "weight": 1
          }
        ],
        "accounts": [{
            "permission": {
              "actor": "test",
              "permission": "active"
            },
            "weight": 1
          },{
            "permission": {
              "actor": "eosio",
              "permission": "active"
            },
            "weight": 1
          }
        ]
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS5hepJxGjP3X93wVmXZeBSNavGWeJvwWtev5ak8oARqwrXjehXd",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}
```

### 增加权限

> 代码如下：
```
cleos set account permission lome test '{"threshold":1,"keys":[{"key":"EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg","weight":1}],"accounts":[]}' active
```
> 结果如下：

```
{
  "account_name": "lome",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg",
            "weight": 1
          },{
            "key": "EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV",
            "weight": 1
          }
        ],
        "accounts": [{
            "permission": {
              "actor": "test",
              "permission": "active"
            },
            "weight": 1
          },{
            "permission": {
              "actor": "eosio",
              "permission": "active"
            },
            "weight": 1
          }
        ]
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS5hepJxGjP3X93wVmXZeBSNavGWeJvwWtev5ak8oARqwrXjehXd",
            "weight": 1
          }
        ],
        "accounts": []
      }
    },{
      "perm_name": "test",
      "parent": "active",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}
```

### 删除权限
终于来到了最后一节，恭喜你！
有人会说这个权限或者群组没用了怎么办？这节我就来教你，我们来删除它：
执行代码如下
```
cleos set account permission lome test 'NULL' active
```

命令就不再多赘述什么意思。这个命令中只有NULL前面没见过，这里是用来专门删除权限或者群组用的。执行结果如下：

```
{
  "account_name": "lome",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS6ePVMSdSkGn4wDMqcCSTSN3GnRgEPxDPEioQQXUkfYxT8jrudg",
            "weight": 1
          },{
            "key": "EOS5wMFMPiD6qbKSZQpJFdEpzvY2yC2o6XLsg97gPPrYHbW4KovjV",
            "weight": 1
          }
        ],
        "accounts": [{
            "permission": {
              "actor": "test",
              "permission": "active"
            },
            "weight": 1
          },{
            "permission": {
              "actor": "eosio",
              "permission": "active"
            },
            "weight": 1
          }
        ]
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS5hepJxGjP3X93wVmXZeBSNavGWeJvwWtev5ak8oARqwrXjehXd",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}
```

***若转载请注明出处。谢谢！！！***
