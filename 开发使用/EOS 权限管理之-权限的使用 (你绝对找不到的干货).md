EOS 权限管理之-权限的使用 (你绝对找不到的干货)
----
作者: lome


> 首先，跟大家说声抱歉，由于之前一直在准备EOS上线的一些工作，所以，很长时间没有更新内容。今天正好有时间，也想到了一些题材，就来说一下这个话题。本文完全是个人见解，如有不当之处，欢迎指出。

前提回顾：
相信看这篇教程的人，对我之前的一些账户操作已经进行了了解，如果不了解的，可以移步 https://eosfans.io/topics/372.

之前讲了单个账户的权限的增、删、改、查及一系列操作，并没有讲到权限的使用，那么如果我们增加一个权限，但是不会用它，那又有什么意义呢？
所以，今天就来学习一下权限的使用。

> 说明： 本教程，通过`eosio.token`合约的`transfer`来进行演示。

## 目录

1. 创建`eosio.token`并发放`eosio.token`合约
2. 创建测试账号`eostea`
3. 测试账号`eostea`发放代币
4. 测试账号转账给账号`hello`
5. `hello`账号添加权限`hello`
6. `hello`账号转账给`eostea`
7. `hello`账号权限`hello`绑定action
8. `hello`账户通过`hello`权限转账

### 钱包中中的密匙：
我所有钱包中的密匙如下：

![](https://static.eosfans.tc.ink/photo/2018/7c95e5c0-daec-4842-bc5e-208246580a4a.png?x-oss-process=image/resize,w_1920)

### 创建测试账号

```
cleos create account eosio eosio.token EOS8Ar1fUGtZxcJ8Rdkh3rc55Vqg3ariR6fdBV8zxz4WgTW3nT73L EOS8Ar1fUGtZxcJ8Rdkh3rc55Vqg3ariR6fdBV8zxz4WgTW3nT73L
cleos create account eosio hello EOS5G13KtUHdsqbeuR2fcoRW4bUzJhigTcX37Aw39xKdy4NMQD9hZ EOS5G13KtUHdsqbeuR2fcoRW4bUzJhigTcX37Aw39xKdy4NMQD9hZ
cleos create account eosio eostea EOS8aQ3bgYKsipwsuPGzimnH5be9AkHr3N6Y7Knh6pSPoLAV2y6Ab EOS8aQ3bgYKsipwsuPGzimnH5be9AkHr3N6Y7Knh6pSPoLAV2y6Ab
```
下面我创建本文的所有测试账号,创建情况如下：
![](https://static.eosfans.tc.ink/photo/2018/7fd7c14d-6db3-4b5f-b135-52de6473e4ae.png?x-oss-process=image/resize,w_1920)
![](https://static.eosfans.tc.ink/photo/2018/bef27605-a2ea-4560-b042-044583ee038a.png?x-oss-process=image/resize,w_1920)

### 发布`eosio.token`合约
```
cleos set contract eosio.token eosio.token/
```
![](https://static.eosfans.tc.ink/photo/2018/7fafbc9e-e2d2-4c07-aacf-a71dd095bd92.png?x-oss-process=image/resize,w_1920)

### 发放并转账代币
```
cleos push action eosio.token create '["eostea","10000000000.0000 TEA","create"]' -p eosio.token
cleos push action eosio.token issue '["eostea", "10000000000.0000 TEA","issue"]' -p eostea
cleos push action eosio.token transfer '["eostea","hello","100000000.0000 TEA","transfer"]' -p eostea
```
如图所示：

![](https://static.eosfans.tc.ink/photo/2018/1920319d-1921-4746-9253-b69a01fea55a.png?x-oss-process=image/resize,w_1920)
![](https://static.eosfans.tc.ink/photo/2018/651abf60-af4e-4c86-8487-f5fcfc8ee771.png?x-oss-process=image/resize,w_1920)

### 转账测试
```
cleos push action eosio.token transfer '["hello","eostea","100.0000 TEA","transfer"]' -p hello
```
![](https://static.eosfans.tc.ink/photo/2018/b05951a9-0b9b-4c55-a833-b0d7e6c77050.png?x-oss-process=image/resize,w_1920)

可能大家看到这里还是一头雾水，别着急重点马上就来。

### 给账户`hello`添加`hello`权限
```
cleos set account permission hello hello '{"threshold": 1, "keys":[{"key":"EOS5dFqCCX8nhV5e2RZWTDGFtAw4mJcCjiQC9Fe6zquKwKky2aAEm","weight":1}],"accounts":[],"waits":[]}' active
```
![](https://static.eosfans.tc.ink/photo/2018/518d2018-2cd2-4294-a315-b476908c365f.png?x-oss-process=image/resize,w_1920)

### 新增权限`hello`绑定`transfer`动作
```
cleos set action permission hello eosio.token transfer hello
```
![](https://static.eosfans.tc.ink/photo/2018/0ae512a2-3908-4e04-bfa4-e9124ef542f2.png?x-oss-process=image/resize,w_1920)

权限绑定`action`成功，那么有什么用呢？注意见证奇迹的时刻到了。

### 新增权限的使用

#### 我们先来尝试用`active`权限转账，然后用`hello`权限转账：
```
cleos push action eosio.token transfer '["hello","eostea","100.0000 TEA","transfer"]' -p hello@active
cleos push action eosio.token transfer '["hello","eostea","100.0000 TEA","transfer"]' -p hello@hello
```
![](https://static.eosfans.tc.ink/photo/2018/3970ee60-1fc1-4db8-9d7d-97fe1bc5345b.png?x-oss-process=image/resize,w_1920)

大家可以看到我们用`hello`权限也成功执行了转账操作。

#### 可能一些小伙伴就要说，你的钱包里有`active`权限对应的密匙。好，那么我把钱包锁起来，只留`hello`权限的密匙。

![](https://static.eosfans.tc.ink/photo/2018/15ea66c4-2337-41d5-b239-35e519664ec8.png?x-oss-process=image/resize,w_1920)

大家可以看到，现在我的钱包里只剩下`hello`权限对应的密匙了。再次进行转账：
```
cleos push action eosio.token transfer '["hello","eostea","100.0000 TEA","transfer"]' -p hello@hello
```
![](https://static.eosfans.tc.ink/photo/2018/d3a8c6eb-a155-487c-a867-19d3b79f3f44.png?x-oss-process=image/resize,w_1920)

大家可以看到，这次转账也是成功的。

#### 可能有些同学又要找茬了，那说不定`hello`这个权限本来就能转账呢？

那么，我们现在解除`hello`和`transfer`绑定关系(注意这里是需要`active`权限来接触绑定关系),再次转账
```
cleos set action permission hello eosio.token transfer NULL
cleos push action eosio.token transfer '["hello","eostea","100.0000 TEA","transfer"]' -p hello@hello
```
![](https://static.eosfans.tc.ink/photo/2018/d5f2da7b-4795-43cb-8cd0-6f25687923df.png?x-oss-process=image/resize,w_1920)

大家可以看到，没有绑定权限，是操作不成功的。

## 应用场景

权限和`action`的绑定关系，极大的增加了eos网络权限的灵活性，通过单个权限的绑定，我们可以将一个账户的权限分层管理，甚至一个公司的所有人都可以使用一个`EOS`账户来进行权限分分离。

下面我拿`hello`账号举个例子：
![](https://static.eosfans.tc.ink/photo/2018/18aad458-2eb5-46f1-9273-05f20544ce9d.png?x-oss-process=image/resize,w_1920)

1. owner：　公司的所有者，或者股东，根据权限分配给每个股东相应的权限。
2. active:　管理阶层，可添加部门如`active`，转账，等除拥有者以外的所有操作。
3. hello：财务部门，具有转账权限。
4. oo: 部门小组,可以有相应的转账权限
5. o: 员工，权限不详。

这样成功的将整个公司所有成员的账户都容纳进一个账户，足见`EOS`账户权限的灵活性。

好了！今天就到这里！！
如需转载请联系作者！！谢谢！！！
