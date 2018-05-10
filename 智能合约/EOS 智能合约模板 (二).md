介绍相关的说明，请看第一篇文章。

文章地址： [https://eosfans.io/topics/291](https://eosfans.io/topics/291)

测试前的准备工作

```python
cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5J52mvxVqEk86LS4Qgm7Y3DjYNiEkk4AubdrDbJsQ7JsYxJUR5q"

cleos create key
Private key: 5KkzGioGeYGcvRGEyXgeESU2HjW7yvbQpWcv4F7tnZN73UDwsSG
Public key: EOS69QCKy2zWQge28nxyLHMPHooXHMMoqMfEWSdB87tfmWNFDpbqM

cleos wallet import 5KkzGioGeYGcvRGEyXgeESU2HjW7yvbQpWcv4F7tnZN73UDwsSG
imported private key for: EOS69QCKy2zWQge28nxyLHMPHooXHMMoqMfEWSdB87tfmWNFDpbqM

cleos create account eosio user EOS69QCKy2zWQge28nxyLHMPHooXHMMoqMfEWSdB87tfmWNFDpbqM EOS69QCKy2zWQge28nxyLHMPHooXHMMoqMfEWSdB87tfmWNFDpbqM
executed transaction: f4568d203bb872345ba8c79ec6de996214e1f08e5b34521552947c497d4a20f6 352 bytes 102400 cycles
# eosio <= eosio::newaccount {"creator":"eosio","name":"user","owner":{"threshold":1,"keys":[{"key":"EOS69QCKy2zWQge28nxyLHMPHooX...

cleos set contract user contracts/hello -p user
Reading WAST/WASM from contracts/hello/hello.wast...
Assembling WASM...
Publishing contract...
executed transaction: 3aad99fd4416caf8aa51af0db06ef0bbe88ccc2f6c4d28384f1979c7922022b1 1656 bytes 2200576 cycles
# eosio <= eosio::setcode {"account":"user","vmtype":0,"vmversion":0,"code":"0061736d01000000012e0960027f7f0060027e7e0060017f0...
# eosio <= eosio::setabi {"account":"user","abi":{"types":[],"structs":[{"name":"tpublickey","base":"","fields":[{"name":"key...
```


测试未通过的模板

```python

// field fields
// signed_transaction
// type_def action_def table_def abi_def types
```

测试合约，下面就不介绍了，直接上代码。


```python
void tpublickey (public_key key) {
eosio::print("tpubckey:\n");
}

cleos push action user tpublickey '["EOS6JTKqeXTYmbyYURFJg2HmAbVzc4sPpP7LxhubmGn8SD3p9WibQ"]' -p user
executed transaction: 962b6ed1ff3c8238de353fe1849ab1cd2351b58ba547592eef52938312a05b35 256 bytes 102400 cycles
# user <= user::tpublickey {"key":"EOS6JTKqeXTYmbyYURFJg2HmAbVzc4sPpP7LxhubmGn8SD3p9WibQ"}
>> tpubckey:
```

```python
// @abi action
void tpkarr (std::vector<public_key> keyarr) {
eosio::print("tpkarr:\n");
}

cleos push action user tpkarr '[["EOS6JTKqeXTYmbyYURFJg2HmAbVzc4sPpP7LxhubmGn8SD3p9WibQ"]]' -p user
executed transaction: 00ff309598802ee9768f69b5781a552262fc9919bd3121e44ad98c72e38f7d3a 256 bytes 103424 cycles
# user <= user::tpkarr {"keyarr":["EOS6JTKqeXTYmbyYURFJg2HmAbVzc4sPpP7LxhubmGn8SD3p9WibQ"]}
>> tpkarr:
```

```python
// @abi action
void tasset (asset arg) {
eosio::print("tasset:",arg,"\n");
}


cleos push action user tasset '["100.0000 EOS"]' -p user
executed transaction: bacccdaa7a10a856d8fb8ee21f9d2020c54dd28ff83af2235da2e420aecb5ee4 240 bytes 102400 cycles
# user <= user::tasset {"arg":"100.0000 EOS"}
>> tasset:100.0000 EOS
```

```python
// @abi action
void tstring (std::string str) {
eosio::print("tstring:",str.c_str(),"\n"); 
}

cleos push action user tstring '["OuterSt 9527"]' -p user
executed transaction: 6ac9a4ed6114a6d9244faf40f7c7195de2f91b913f6c48ca1cc6ca6f78a79742 240 bytes 105472 cycles
# user <= user::tstring {"str":"OuterSt 9527"}
>> tstring:OuterSt 9527
```

```python
// // @abi action
void tstringarr (std::vector<std::string> args) {
eosio::print("tstringarr:\n"); 
for (auto &arg:args)
{
eosio::print(arg.c_str(),"\n");
}
}

cleos push action user tstringarr '[["OuterSt 9527","9527 OuterST"]]' -p user
executed transaction: 9e1406a19f8f89b483e49e71257decbac4c0d4002167466b7b2c130c8bc6bb4f 248 bytes 112640 cycles
# user <= user::tstringarr {"args":["OuterSt 9527","9527 OuterST"]}
>> tstringarr:
```

```python
// @abi action
void tsignature(signature arg){
//eosio::print("tsignature:",arg,"\n");

}

cleos push action user tsignature '["EOSJzdpi5RCzHLGsQbpGhndXBzcFs8vT5LHAtWLMxPzBdwRHSmJkcCdVu6oqPUQn1hbGUdErHvxtdSTS1YA73BThQFwT77X1U"]' -p user
executed transaction: 4c2086311b56d4ac4ad11975b0f11a180609c9a8de7c91d3bdd1def756a65aa9 288 bytes 102400 cycles
# user <= user::tsignature {"arg":"EOSJzdpi5RCzHLGsQbpGhndXBzcFs8vT5LHAtWLMxPzBdwRHSmJkcCdVu6oqPUQn1hbGUdErHvxtdSTS1YA73BThQFwT...
```

```python

// @abi action
void tsgarr(std::vector<signature> args) {
// eosio::print("tsgarr:",args,"\n");
// for (auto &arg:args)
// {
// eosio::print(arg,"\n");
// }
}

cleos push action user tsgarr '[["EOSJzdpi5RCzHLGsQbpGhndXBzcFs8vT5LHAtWLMxPzBdwRHSmJkcCdVu6oqPUQn1hbGUdErHvxtdSTS1YA73BThQFwT77X1U","EOSJzdpi5RCzHLGsQbpGhndXBzcFs8vT5LHAtWLMxPzBdwRHSmJkcCdVu6oqPUQn1hbGUdErHvxtdSTS1YA73BThQFwT77X1U"]]' -p user
executed transaction: 6e42e1699eaaf91f575ccef9e58e6aea4697e8ff59aeb1d9f655bd47b0adf909 360 bytes 104448 cycles
# user <= user::tsgarr {"args":["EOSJzdpi5RCzHLGsQbpGhndXBzcFs8vT5LHAtWLMxPzBdwRHSmJkcCdVu6oqPUQn1hbGUdErHvxtdSTS1YA73BThQF...

```

```python
// @abi action
void tchecksum(checksum256 arg){
//eosio::print("tchecksum:",arg,"\n");
}

cleos push action user tchecksum '["ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad"]' -p userexecuted transaction: 63b7dc53738c655bcb980db743ea533695462840288070dabf25bba131ced877 256 bytes 102400 cycles
# user <= user::tchecksum {"arg":"ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad"}
```

```python
// @abi action
void tfixedstr32(fixed_string32 arg) {
//eosio::print("tfixedstring32:",arg,"\n");
}

cleos push action user tfixedstr32 '["1234567890abcdef1234567890abcdef"]' -p user
executed transaction: bdb03cd8965d284fe47aa31a66386f93082c0014bc59cf801ad1eaf042513c45 256 bytes 102400 cycles
# user <= user::tfixedstr32 {"arg":"1234567890abcdef1234567890abcdef"}
```

```python
//fixedstring16
//@abi action
void tfixedstring(fixed_string16 arg) {
//eosio::print("tfixedstring16:",arg,"\n");
}

cleos push action user tfixedstring '["1234567890abcdef"]' -p user
executed transaction: b9ecd59ad75028d8c74e201e5404ee437b2639f020cf2823fc7fdaf40aa1100e 240 bytes 102400 cycles
# user <= user::tfixedstring {"arg":"1234567890abcdef"}
```

```python
// @abi action
void tbytes (bytes arg) {
// eosio::print("tbytes:",arg,"\n");
}

cleos push action user tbytes '["010203"]' -p user
executed transaction: 205550d29b2785d40b5b52f0f74f25fa480ae2b6f8176b0a1ff472770422e5cb 232 bytes 103424 cycles
# user <= user::tbytes {"arg":"010203"}
```

```python
// @abi action
void tname(name arg) {
eosio::print("tname:",arg,"\n");
}

cleos push action user tname '["user"]' -p user
executed transaction: 7d6f7db9427281a275235f988f572efd8bd26ec9a113b6101b89ac5dfc1040a2 232 bytes 102400 cycles
# user <= user::tname {"arg":"user"}
>> tname:user
```

```python
// @abi action
void taccountname(account_name arg) {
eosio::print("taccountname:",arg,"\n");
}

cleos push action user taccountname '["user"]' -p user
executed transaction: 0861837a5fdb9fdce2ffa19db2912ef03c54220a710f532f0a35545b91900e47 232 bytes 102400 cycles
# user <= user::taccountname {"arg":"user"}
>> taccountname:15426359243929812992
```

```python
// @abi action
void tpermname(permission_name arg) {
eosio::print("tpermname:",arg,"\n");
}

cleos push action user tpermname '["active"]' -p user
executed transaction: a1d046d5b882a851f171048d8054791ee1151007575a1488e867561c49af3ec5 232 bytes 102400 cycles
# user <= user::tpermname {"arg":"active"}
>> tpermname:3617214756542218240
```

```python

// @abi action
void tactionname(action_name arg){
eosio::print("tactionname:",arg,"\n"); 
}

cleos push action user tactionname '["active"]' -p user
executed transaction: 14c98043bff1225c98eaaa81ddfeb41fa4d05e6cc2a88fa8416dc00f37f20310 232 bytes 102400 cycles
# user <= user::tactionname {"arg":"active"}
>> tactionname:3617214756542218240
```

```python
// @abi action
void tscopename(scope_name arg) {
eosio::print("tscopename:",arg,"\n"); 
}

cleos push action user tscopename '["scope"]' -p user
executed transaction: a63d0acc81397129dec2bef6f2f643930a4328d9961ad53415752edd3722b8a1 232 bytes 102400 cycles
# user <= user::tscopename {"arg":"scope"}
>> tscopename:13990807175891517440
```

```python
// @abi action
void tpermlvl(permission_level arg) {
//eosio::print("tpermlvl:",arg,"\n"); 
}

cleos push action user tpermlvl '[{"account":"acc1", "name":"actionname1", "authorization":[{"actor":"acc1","permission":"permname1"}], "data":"445566"},]' -p user
executed transaction: 7c0c5d052d199f881d7aabce2bd817b1bb22ac5e5d6c30353c679124e389908b 240 bytes 102400 cycles
# user <= user::tpermlvl {"arg":{"actor":"","permission":""}}
```

```python

// @abi action
void taction(action arg) {
//eosio::print("taction:",arg,"\n"); 
}

cleos push action user taction '[{"account":"acc1", "name":"actionname1", "authorization":[{"actor":"acc1","permission":"permname1"}], "data":"445566"}]' -p user
executed transaction: 2c5906b2b2ee005c3b782441f4be6102940b02d84f80ff7c5d464fa85e812071 264 bytes 105472 cycles
# user <= user::taction {"arg":{"account":"acc1","name":"actionname1","authorization":[{"actor":"acc1","permission":"permnam...
```

```python
// @abi action
void tpermlvlwgt(eosiosystem::permission_level_weight arg){
//eosio::print("taction:",arg,"\n"); 
}

cleos push action user tpermlvlwgt '[{"permission":{"actor":"acc1","permission":"permname1"},"weight":"1"}]' -p user
executed transaction: b06342593cfca38d7367eebca0a63bc7c4919b132de2fa0713feb422d3c1e4cb 240 bytes 102400 cycles
# user <= user::tpermlvlwgt {"arg":{"permission":{"actor":"acc1","permission":"permname1"},"weight":1}}
```

```python
// @abi action
void ttransaction(transaction arg) {

}

cleos push action user ttransaction '[{"ref_block_num":"1","ref_block_prefix":"2","expiration":"2021-12-20T15:30","region": "1","context_free_actions":[{"account":"contextfree1", "name":"cfactionname1", "authorization":[{"actor":"cfacc1","permission":"cfpermname1"}], "data":"778899"}],"actions":[{"account":"accountname1", "name":"actionname1", "authorization":[{"actor":"acc1","permission":"permname1"}], "data":"445566"}],"net_usage_words":15,"kcpu_usage":43,"delay_sec":0}]' -p user
Error 3030000: transaction validation exception
Ensure that your transaction satisfy the contract's constraint!
Error Details:
condition: assertion failed: read
```

```python
// @abi action
void tkeyweight(key_weight arg) {

}

cleos push action user tkeyweight '[{"key":"EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV", "weight":"100"}]' -p user
executed transaction: 5f377f0c85335ce017e034ed5362636238badeda146b1f8a9d6c7d44d2e35430 264 bytes 102400 cycles
# user <= user::tkeyweight {"arg":{"key":"EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV","weight":100}}
```

```python

// @abi action
void tauthority(authority arg) {

}

cleos push action user tauthority '[{"threshold":"10","keys":[{"key":"EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV", "weight":100},{"key":"EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV", "weight":200}],"accounts":[{"permission":{"actor":"acc1","permission":"permname1"},"weight":"1"},{"permission":{"actor":"acc2","permission":"permname2"},"weight":"2"}]}]' -p user
executed transaction: 75182d443993274585e8f23368ba8d963d60fd15b1afd30aa64a28ee57ec2ffb 336 bytes 105472 cycles
# user <= user::tauthority {"arg":{"threshold":10,"keys":[{"key":"EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV","weigh...
```

cpp文件
```python
#include <utility>
#include <vector>
#include <string>
#include <eosiolib/eosio.hpp>
#include <eosiolib/asset.hpp>
#include <eosiolib/contract.hpp>
#include <eosiolib/crypto.h>
#include <eosiolib/transaction.hpp>

using eosio::permission_level;

#include <eosio.system/native.hpp>


using eosio::key256;
using eosio::indexed_by;
using eosio::const_mem_fun;
using eosio::asset;
using eosio::bytes;
using eosio::action;
using eosio::print;
using eosio::name;
using eosio::transaction;
using eosiosystem::key_weight;
using eosiosystem::authority;



class hello : public eosio::contract { 
	public:
		 hello(account_name self)
        :eosio::contract(self)
        {}

        // @abi action
        void tpublickey (public_key key) {
        	eosio::print("tpubckey:\n");
        }
        
        // @abi action
 		void tpkarr (std::vector<public_key> keyarr) {
        	eosio::print("tpkarr:\n");
        }

        // @abi action
        void tasset (asset arg) {
        	eosio::print("tasset:",arg,"\n");
        }

        // @abi action
        void tassetarr(std::vector<asset> args) {
        	eosio::print("tassetarr:\n");	
        	for (auto &arg :args)
        	{
        		eosio::print(arg,"\n");
        	}
        }

        // @abi action
        void tstring (std::string str) {
        	eosio::print("tstring:",str.c_str(),"\n");	
        }
		
		// // @abi action
        void tstringarr (std::vector<std::string> args) {
        	eosio::print("tstringarr:\n");	
        	for (auto &arg:args)
        	{
        		eosio::print(arg.c_str(),"\n");
        	}
        }

        // @abi action
        void tsignature(signature arg){
        	//eosio::print("tsignature:",arg,"\n");
        	
        }

        // @abi action
        void tsgarr(std::vector<signature> args) {
        	// eosio::print("tsgarr:",args,"\n");
        	// for (auto &arg:args)
        	// {
        	// 	eosio::print(arg,"\n");
        	// }
        }

        // @abi action
        void tchecksum(checksum256 arg){
        	//eosio::print("tchecksum:",arg,"\n");
        }

        // @abi action
        void tchecksumarr(std::vector<checksum256> args) {
        	eosio::print("tchecksumarr:\n");	
        }

        //error
        // @abi action
        void tfieldname (field_name arg) {
        	//eosio::print("tfieldname:",arg,"\n");
        }

        // @abi action
        void tfixedstr32(fixed_string32 arg) {
        	//eosio::print("tfixedstring32:",arg,"\n");
        }

        //fixedstring16
        //@abi action
        void tfixedstring(fixed_string16 arg) {
        	//eosio::print("tfixedstring16:",arg,"\n");
        }

        // @abi action
        void ttypename(type_name arg) {
        	//eosio::print("ttypename:",arg,"\n");
        }

        // @abi action
        void tbytes (bytes arg) {
        	// eosio::print("tbytes:",arg,"\n");
        }

        // @abi action
        void tname(name arg) {
        	eosio::print("tname:",arg,"\n");
        }

        // @abi action
        void taccountname(account_name arg) {
        	eosio::print("taccountname:",arg,"\n");
        }

        // @abi action
        void tpermname(permission_name arg) {
        	eosio::print("tpermname:",arg,"\n");
        }

        // @abi action
        void tactionname(action_name arg){
        	eosio::print("tactionname:",arg,"\n");	
        }

        // @abi action
        void tscopename(scope_name arg) {
        	eosio::print("tscopename:",arg,"\n");	
        }

        // @abi action
        void tpermlvl(permission_level arg) {
        	//eosio::print("tpermlvl:",arg,"\n");	
        }

        // @abi action
        void taction(action arg) {
        	//eosio::print("taction:",arg,"\n");	
        }

        // @abi action
        void tpermlvlwgt(eosiosystem::permission_level_weight arg){
        	//eosio::print("taction:",arg,"\n");	
        }

        // @abi action
        void ttransaction(transaction arg) {

        }

        // @abi action
        void tkeyweight(key_weight arg) {

        }

        // @abi action
        void tauthority(authority arg) {

        }
        
        //
        // field fields
        // signed_transaction
        // type_def action_def table_def abi_def  types  
     
};

EOSIO_ABI( hello, (tpublickey)(tpkarr)(tasset)(tassetarr)(tstring) (tstringarr)(tsignature)(tsgarr)(tchecksum)(tchecksumarr)(tfieldname)(tfixedstr32)(tfixedstring)(ttypename)(tbytes)(tname)(taccountname)(tpermname)(tactionname)(tscopename)(tpermlvl)(taction)(tpermlvlwgt)(ttransaction)(tkeyweight)(tauthority)
	)
```
