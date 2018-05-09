# 1.  支持类型说明和调用实例

自动生成abi文件
函数：添加注释//@abi action ，最新版本可以不写
需要EOSIO_ABI中添加对应函数（必须）。

结构体：需要 //@abi table name type（必须）

## 1.1 int类型相关说明

生成abi可以支持的int类型

```python
uint64_t uint32_t uint16_t uint8_t

int64_t int32_t int16_t int8_t
```

eosio::print 输出只支持以下int类型

```python
uint64_t uint32_t int64_t int32_t
```
## 1.2 函数书写和合约调用 

下面介绍一下，合约中支持哪些类型和怎么传参数的。

### 1.2.1  account_name 类型

继承类自己定义的，还有其他类型，这里不一一列举了

函数，这里eosio::name 暂时用不了了，可能是最近改动过了，但是我没找到相关log


```python
// @abi action 
void hi(account_name user ) { 
require_auth( user );
eosio::print( "hi, ", user); 
//eosio::print( "hi, ", eosio::name(user)); 
}
```

合约调用


```python
cleos push action user hi '[user]' -p user 
executed transaction: 9b904829cb37416b283148c0579e8081403feca2306a39ab40470f163bfd0d08 232 bytes 102400 cycles
# user <= user::hi {"user":"user"}
>> hi, 15426359243929812992
```

### 1.2.2 std::string 类型


```python
// @abi action
void abiprint(std::string str) {
eosio::print("abiprint, ",str.c_str() );
}
```

合约调用


```python
cleos push action user abiprint '["OuterTS"]' -p user
executed transaction: f6cf3d91c4aba8c71b096557615fcfce24a000a361503abc27d0201b6646f28d 232 bytes 102400 cycles
# user <= user::abiprint {"str":"OuterTS"}
>> abiprint, OuterTS
```

### 1.2.3 int相关类型

这里声明一下，带有返回参数的，不能生成abi文件


```python
// @abi action
void abiadd (uint64_t num1, uint64_t num2) {
eosio::print( "add:, ", add(num1 , num2) ); 
}

uint64_t add (uint64_t num1, uint64_t num2){
return num1 + num2; 
} 

void mulnum( uint64_t num1,uint32_t num2,uint16_t num3,uint8_t num4,
int64_t num5,int32_t num6,int16_t num7,int8_t num8) {
eosio::print("eosio::print dont support others");
eosio::print("num1:",num1," num2:",num2,"\n");
eosio::print("num5:",num5," num6:","\n");

}
```

合约调用


```python
cleos push action user abiadd '[9527,9527]' -p user
executed transaction: 65e9c964929bcc4851fba6dbcbf5bdfef57dc55170b19a091c9c9699a14cce47 240 bytes 102400 cycles
# user <= user::abiadd {"num1":9527,"num2":9527}
>> add:, 19054



cleos push action user mulnum '[9527,9527,9527,9527,9527,9527,9527,9527]' -p user
executed transaction: d068edbd729346e8e58038997c850875de7ec13517de7a301e4336217b9b0b2d 256 bytes 102400 cycles
# user <= user::mulnum {"num1":9527,"num2":9527,"num3":9527,"num4":55,"num5":9527,"num6":9527,"num7":9527,"num8":55}
>> eosio::print dont support othersnum1:9527 num2:9527
```

### 1.2.4  vector类型和多种类型传参


```python
// @abi action
void mulargs(int64_t num, std::string str,std::vector<uint64_t> iarr) {
eosio::print("num:",num,"\n");
eosio::print("str:",str.c_str(),"\n");
for (int i = 0; i < iarr.size(); ++i)
{
eosio::print(iarr[i]);
}

}
```

合约调用


```python
cleos push action user mulargs '[9527,"OuterST",[9,5,2,7]]' -p user
executed transaction: a84344447e58bbe3fe6b27a7cebe750bde802ed8d57904db2965e03c1c0840de 272 bytes 104448 cycles
# user <= user::mulargs {"num":9527,"str":"OuterST","iarr":[9,5,2,7]}
>> num:9527
```

# 2. cpp源码和abi文件

## 2.1 生成abi文件 


```python
feng@feng-B250-HD3P:~/work/eos/build/contracts/hellocopy$ eosiocpp -g hellocopy.abi hellocopy.cpp 
```

## 2.2 生成wast文件


```python
feng@feng-B250-HD3P:~/work/eos/build/contracts/hellocopy$ eosiocpp -o hellocopy.wast hellocopy.cpp
```

## 2.3 部署合约

合约更新的时候，需要重新生成abi文件和wast文件。

每次部署合约需要保证code的变化：所以部署A合约的时候，再一次部署A合约会失败。

要想更新A合约，需要先部署B合约，再一次部署A合约。


```python
feng@feng-B250-HD3P:~/work/eos/build$ cleos set contract user contracts/hellocopy/
```

## 2.4 cpp文件

EOSIO_ABI （func,(a)(b)(c)） a,b,c的顺序要对应

```python


#include <vector>
#include <eosiolib/print.hpp>

#include <eosiolib/contract.hpp>
#include <eosio.system/eosio.system.hpp>

// using eos_currency = eosiosystem::contract<N(eosio)>::currency;
// using eosio::key256;
// using eosio::indexed_by;
// using eosio::const_mem_fun;
// using eosio::asset;


class hellocopy : public eosio::contract { 
	public: 
		hellocopy(account_name self)
		:eosio::contract(self)
		{}
		// @abi action 
		void hi(account_name user ) { 
			require_auth( user );
			eosio::print( "hi, ", user); 
			//eosio::print( "hi, ", eosio::name(user)); 
		} 

		// @abi action
		void abiprint(std::string str) {
			eosio::print("abiprint, ",str.c_str() );
		}
		// @abi action
	    void abinum (uint64_t num) {
			eosio::print( "abinum: ",num ); 	
		}

		// @abi action
		void abiadd (uint64_t num1, uint64_t num2) {
			eosio::print( "add:, ", add(num1 , num2) ); 	
		}

		uint64_t add (uint64_t num1, uint64_t num2){
			return num1 + num2; 	
		} 
		
		void mulnum( uint64_t num1,uint32_t num2,uint16_t num3,uint8_t num4,
				int64_t num5,int32_t num6,int16_t num7,int8_t num8) {
			eosio::print("eosio::print dont support others");
			eosio::print("num1:",num1," num2:",num2,"\n");
			eosio::print("num5:",num5," num6:","\n");
			
		}

		// @abi action
		void mulargs(int64_t num, std::string str,std::vector<uint64_t> iarr) {
			eosio::print("num:",num,"\n");
			eosio::print("str:",str.c_str(),"\n");
			for (int i = 0; i < iarr.size(); ++i)
			{
				eosio::print(iarr[i]);
			}
			
		}


		//@abi table test i64
		struct test
		{
			std::string str;
		};
}; 

EOSIO_ABI( hellocopy, (hi)(abiprint)(abinum)(abiadd)(mulnum)(mulargs))

```

## 2.5 abi文件


```python
{
  "____comment": "This file was generated by eosio-abigen. DO NOT EDIT - 2018-04-18T10:08:36",
  "types": [],
  "structs": [{
      "name": "test",
      "base": "",
      "fields": [{
          "name": "str",
          "type": "string"
        }
      ]
    },{
      "name": "hi",
      "base": "",
      "fields": [{
          "name": "user",
          "type": "account_name"
        }
      ]
    },{
      "name": "abiprint",
      "base": "",
      "fields": [{
          "name": "str",
          "type": "string"
        }
      ]
    },{
      "name": "abinum",
      "base": "",
      "fields": [{
          "name": "num",
          "type": "uint64"
        }
      ]
    },{
      "name": "abiadd",
      "base": "",
      "fields": [{
          "name": "num1",
          "type": "uint64"
        },{
          "name": "num2",
          "type": "uint64"
        }
      ]
    },{
      "name": "mulnum",
      "base": "",
      "fields": [{
          "name": "num1",
          "type": "uint64"
        },{
          "name": "num2",
          "type": "uint32"
        },{
          "name": "num3",
          "type": "uint16"
        },{
          "name": "num4",
          "type": "uint8"
        },{
          "name": "num5",
          "type": "int64"
        },{
          "name": "num6",
          "type": "int32"
        },{
          "name": "num7",
          "type": "int16"
        },{
          "name": "num8",
          "type": "int8"
        }
      ]
    },{
      "name": "mulargs",
      "base": "",
      "fields": [{
          "name": "num",
          "type": "int64"
        },{
          "name": "str",
          "type": "string"
        },{
          "name": "iarr",
          "type": "uint64[]"
        }
      ]
    }
  ],
  "actions": [{
      "name": "hi",
      "type": "hi",
      "ricardian_contract": ""
    },{
      "name": "abiprint",
      "type": "abiprint",
      "ricardian_contract": ""
    },{
      "name": "abinum",
      "type": "abinum",
      "ricardian_contract": ""
    },{
      "name": "abiadd",
      "type": "abiadd",
      "ricardian_contract": ""
    },{
      "name": "mulnum",
      "type": "mulnum",
      "ricardian_contract": ""
    },{
      "name": "mulargs",
      "type": "mulargs",
      "ricardian_contract": ""
    }
  ],
  "tables": [{
      "name": "test",
      "index_type": "i64",
      "key_names": [
        "str"
      ],
      "key_types": [
        "string"
      ],
      "type": "test"
    }
  ],
  "ricardian_clauses": []
}
```
