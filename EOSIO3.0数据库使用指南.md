
## 注：由于翻译仓促，英语水平有限，觉得看不懂的朋友可以去文章末尾，有英文原文链接。谢谢！！

EOSIO区块链上持久数据的多索引容器抽象

## 目录

- 概览
  - 持久性服务的需求
- EOS多索引API
  - EOS多索引迭代器
- 使用Vehicle
  - 怎样创建自己的EOSIO多索引表
  - 怎样去用自己的EOSIO多索引表
- 保持传输的跟踪器示例
- C++ API 介绍
  - eosio::multi_index
  - eosio::indexed_by
  - eosio::multi_index::index 

## 概览

EOSIO提供了一系列服务和接口，使合同开发人员能够跨越行动持续状态，从而实现交易，边界。如果没有持久性，处理过程中产生的状态将在处理超出范围时丢失。 持久性组件包括：

1. 用于在数据库中保持状态的服务
2. 增强查询功能以查找和检索数据库内容
3. C ++ API提供给这些服务，供合同开发人员使用
4. 用于访问核心服务的C API，是图书馆和系统开发人员感兴趣的

本文件涵盖前三个主题。

## 持久性服务的需求
Action调用EOSIO合约，Action在被称为Action上下文的环境中运作。如下图所示，Action上下文提供执行Action所需的几件事情。其中一件事是Action的工作内存。这是Action执行的地方。在处理一个Action之前，EOSIO为该Action进行一次内存清理工作。在新操作的上下文中当另一个操作执行时可能已经被设置的变量不可用。在行动中传递状态的唯一方法是将其持久存储并从EOSIO数据库中检索。

![](https://eosfans-static.strahe.com/photo/2018/19b6de52-8d10-484b-bc64-aead47b24164.png?x-oss-process=image/resize,w_1920)

## EOSIO Multi-Index API
EOSIO Multi-Index API 提供了关于EOSIO数据库的C++接口。EOSIO Multi-Index API 来自　Boost Multi-Index Containers　https://www.boost.org/doc/libs/1_66_0/libs/multi_index/doc/index.html 。该API为具有丰富检索功能的对象存储提供了一个模型,支持使用不同排序和访问语义的多个索引.Multi-Index API 通过`eosio::multi_index`类提供，在文件`contracts/eosiolib`下。该类使用C++编写的合约能够读取和修改EOSIO数据库中的持久状态。

Multi-Index容器接口`eosio::multi_index`提供了一个任意C++类型的同类容器。它保存了一个从对象派生的各种类型的键的多索引的分类。他可以很容易区分 Boost Multi-index Containers。实际上许多的成员函数签名是`eosio::multi_index`仿照`boost::multi_index`,尽管有一些重要的不同。


`eosio::multi_index`可以在概念上被视为传统数据库中的表格，其中行是容器中的单个对象，列是容器中对象的成员属性，并且索引通过与一个键兼容的键提供对对象的快速查找 对象成员属性。

传统的数据库表允许索引成为表中某些列数的用户定义函数。`eosio::multi_index`同样允许索引是任何用户定义的函数。但其返回值仅限于受支持的一组受限密钥类型之一。

传统数据库表通常有一个唯一的主键，它允许明确标识表中的特定行，并为表中的行提供标准排序顺序。`eosio::multi_index`支持类似的语义，但是该对象的主键在`eosio::multi_index`容器必须是唯一的无符号64位整数。`eosio::multi_index`中的对象容器按主键索引按无符号64位整数主键的升序排序。

## EOSIO Multi-Index 迭代器

与其他区块链基础架构相比，EOSIO持久性服务的一个关键区别在于其多指标迭代器。 与仅提供键值存储的其他区块链不同，EOSIO Multi-Index表允许合约开发人员保存按照各种不同键类型排序的对象集合，这些键类型可以从对象内的数据派生。这使得丰富的检索功能。最多可以定义16个二级索引，每个索引都有自己的排序和检索表格内容的方式。

EOSIO多指数迭代器遵循C++迭代器通用的模式。所有迭代器都是双向常量，可以是const_iterator或const_reverse_iterator。 迭代器可以取消引用以提供对多索引表中的对象的访问。

# 使用

## 如何创建您的EOSIO Multi-Index表
以下是使用EOSIO多指标表创建您自己的持久性数据的步骤摘要。

- 使用C ++类或结构定义你的对象。每个对象都将位于其自己的Multi-Index表中。
- 在`class/struct`定义一个`const`成员函数调用`primary_key`返回你的`uint64_t`primary_key关于你的对象的。
- 确定二级索引。最多支持16个附加索引。辅助索引支持以下列出的几种密钥类型。
  - idx64 - 原始的64位无符号整数密钥.
  - idx128 - 原始128位无符号整数密钥.
  - idx256 - 256位固定大小的字典键.
  - idx_double - 双精度浮点键.
  - idx_long_double - 四倍精度浮点键.
- 为每个二级索引定义一个键提取器。密钥提取器是一个函数，用于从多索引表的元素中获取密钥。请参阅下面的 Multi-Index Constructor和indexed_by部分。

## 如何使用您的EOSIO Multi-Index表
- 实例化您的多索引表.
- 按照您的合约要求插入（emplace）到表中，然后修改或清除表中的对象。
- 使用get，find和iterator操作查找和遍历表中的对象。

# 保持传输的跟踪器示例
我们将调用包含单个服务事务的表的服务表。该表格将用于创建服务记录报告。该表的记录将包含以下属性：

- Primary key - 客户ID不能成为主键，因为客户可以有很多记录。事实上，我们不需要直接使用主键，所以我们可以让系统为我们生成一个。

- Customer ID - 这将对应于客户的帐户名称（uint64_t value）
- 服务日期 - 服务执行的日期.
- 里程表 - 在服务之日传输的里程表读数

我们希望能够按客户搜索此表，因此我们将在客户资产上创建二级索引。下图说明了客户的服务表及其二级索引。

![](https://eosfans-static.strahe.com/photo/2018/411b20fc-7aea-4de3-b81d-63d53c6f89a5.png?x-oss-process=image/resize,w_1920)

我们将用这些属性声明一个struct service_rec来表示我们的服务记录对象：

```
struct service_rec {
    uint64_t        pkey;           // opaque, will use available_primary_key()
    account_name    customer;       // will create a secondary index on this
    uint32_t        service_date;
    uint32_t        odometer;
    
    auto            primary_key()const { return pkey; }
    account_name    get_customer()const { return customer; }

    EOSLIB_SERIALIZE( service_rec, (pkey)(customer)(service_date)(odometer) )
};
```

为了实例化我们的服务表（我们将调用我们的实例service_table），我们在合同中写下以下内容，其中mechanic是机制的帐户名称。
```
using service_table_type = multi_index<service, service_rec,
    indexed_by< N(bycustomer), const_mem_fun<service_rec, account_name, &service_rec::get_customer> >
>;
service_table_type service_table( current_receiver(), mechanic );
```
传递给构造函数的两个参数为表建立访问权限。 第一个参数（代码参数，见下文）决定动作是否访问合约自己的持久状态（即code == current_receiver（）），在这种情况下，动作上下文同时具有对表的读取和写入访问权限，或者 如果它正在访问某个其他合同的持久状态并因此是只读的。

要将记录添加到service_table，我们可以使用类似于以下的C ++（假定服务记录内容对应的本地变量）。 变量customer_name包含客户可读的字符串名称，我们必须将其转换为存储在表格中的客户的account_name编码。

```
service_table.emplace(mechanic, [&]( auto& s_rec ) {
    s_rec.pkey = service_table.available_primary_key();
    s_rec.customer = eosio::chain::string_to_name(customer_name);
    s_rec.service_date = service_date;
    s_rec.odometer = odometer;
});
```
要找到客户的所有服务记录，我们首先获取客户二级索引，
```
auto customer_index = service_table.template get_index<N(bycustomer)>();
```
那么我们可以使用二级索引找到想要的客户。
```
account_name customer_acct = eosio::chain::string_to_name(customer_name);
auto cust_itr = customer_index.find(customer_acct);
```
我们可以遍历二级索引来获取客户的所有服务记录。

```
while (cust_itr != service_table.end() && cust_itr->customer == customer_acct) {
    // code to process customer record...
    ...
    cust_itr++;
} 
```
## 传输的跟踪器customer Table

我们的第二张表格将包含客户的当前状态。我们将这称为客户表。此状态由以下属性组成：

- 客户ID - 他对客户表格将是唯一的，所以我们可以将其作为
我们的主要关键.
- 上次服务日期 - 上次进行日常服务的时间.
- 在最后服务日期的里程表 - 每次客户或技工更新最新的里程表值时都会计算此值.
- 自上次服务以来的里程 - 每次客户或技工更新最新的里程表值时都会计算此值

我们将在上次服务日期和上次服务以来的里程中为客户表编制索引。 下图说明了customer表和它的两个二级索引，bydate和bymiles。 在这个例子中注意到，Sue自愿提供最新的里程表读数，大概是使用应用程序提供的接口。 每当客户输入里程时，就会计算相应的Miles Since Last Service，并将此值编入索引。 这使机械师能够确定需要按日期或按里程服务的客户，至少对于选择参与的客户而言。 下面给出了使用这些值的例子。

![](https://eosfans-static.strahe.com/photo/2018/699bd38d-a9cb-466b-bb54-b0657a94eddf.png?x-oss-process=image/resize,w_1920)

我们将用这些属性声明一个struct customer_rec来表示我们的客户记录对象：
```
struct customer_rec {
    account_name    customer_id             // primary key
    uint64_t        last_service_date;      // will create a secondary index on this
    unit32_t        odometer_at_last_service;
    uint32_t        latest_odometer;
    
    auto            primary_key()const { return customer_id; }
    uint64_t        get_last_service_date()const { return last_service_date; }
    uint64_t        get_miles_since_service()const {
                        return latest_odometer > odometer_at-last_service ? latest_odometer - odometer_at_last_service : 0;
                    }

    EOSLIB_SERIALIZE( customer_rec, (customer_id)(last_service_date)(odometer_at_last_service)(latest_odometer) )
};
```
为了实例化我们的客户表（我们将其称为customer_table），我们在合同中写下以下内容，其中mechanic是机械师的帐户名称。
```
using customer_table_type = multi_index<customer, customer_rec,
   indexed_by< N(bydate), const_mem_fun<customer_rec, uint64_t, &customer_rec::get_last_service_date> >,
   indexed_by< N(bymiles), const_mem_fun<customer_rec, uint64_t, &customer_rec::get_miles_since_service> >
>;
customer_table_type customer_table( current_receiver(), mechanic );
```
要将记录添加到customer_table，我们可以使用类似于以下内容的C ++（假设客户记录内容的相应本地变量）。 变量customer_name包含客户可读的字符串名称，我们必须将其转换为存储在表格中的客户的account_name编码。

```
customer_table.emplace(mechanic, [&]( auto& c_rec ) {
    c_rec.customer_id = eosio::chain::string_to_name(customer_name);
    c_rec.last_service_date = current_date;
    c_rec.odometer_at_last_service = current_odometer;
    c_rec.latest_odometer = current_odometer;
});
```
我们可以使用主键customer_id找到客户。
```
account_name customer_id = eosio::chain::string_to_name(customer_name);
auto cust_itr = customer_table.find(customer_id);
```
要更新latest_odometer属性，请使用类似于以下内容的内容。

```
account_name customer_id = eosio::chain::string_to_name(customer_name);
customer_table.modify(customer_table.get(customer_id), mechanic, [&]( auto& c_rec) {
    c_rec.latest_odometer = customer_provided_odometer_reading;     // customer provided input
});
```
参数mechanic是更新操作的付款人。我们的示例允许机械师或客户更新latest_odometer属性，但我们不希望客户仅为志愿者最新的里程表读数付费。

请注意，以上要求计算miles_since_service属性。 该示例假定变量customer_odometer_at_last_service保存客户上次服务的里程表读数。 我们可以通过使用服务表的bycustomer二级索引来获取客户的最新服务记录。 有关使用此索引的信息，请参阅上述服务表示例。

要获取customer_table的二级索引，请使用类似于以下内容的内容。
```
auto last_service_date_index = customer_table.template get_index<N(bydate)>();
auto miles_since_service_index = customer_table.template get_index<N(bymiles)>();
```
我们可以通过bydate辅助索引向后迭代，以获取所有到期服务的客户，例如，因为他们的最后一项服务超过3个月前。
```
auto overdue_date_itr = last_service_date_index.upper_bound(three_months_ago);  // date for three months earlier than today
// code to iterate the customer table backwards from three months ago
...
```
我们可以通过bymiles二级索引来迭代，以获得所有到期服务的客户，例如，因为他们自上次服务以来的里程数大于3000。
```
auto over_miles_itr = last_service_date_index.lower_bound(3000);  // all records 3000 or more miles
// code to iterate the customer table forward for greater than 3000 miles
...
```

# C++ API
## eosio::multi_index

本节介绍`eosio::multi_index C ++ API`。 该接口实际上是Boost Multi-Index容器库的改编版本。 它直接使用`boost/multi_index/const_mem_fun`类模板进行密钥提取。 它还使用Boost Hana库进行元编程。 有关其他概念信息和详细信息，请参阅www.boost.org上的Boost文档。

在下面的描述中，以下别名将用于通用模板声明。

|别号|描述|
|---|---|
|object_type|Multi-Index表中的对象的类型|
|secondary_index|多索引表的相应二级索引的类型|

## 构造函数

|方法|描述|
|---|---|
|multi_index( uint64_t code, uint64_t scope )|构造一个多索引表的实例|

|参数|描述|
|---|---|
|code|拥有表格的帐户|
|scope|代码层次结构中的范围标识符|
|后置条件|多索引表的新的空实例被创建|
|`code`和`scope`成员属性被初始化|多索引表的新的空实例被创建|
|每个二级索引表初始化||
|代码层次结构中的范围标识符|||
#### 注意：`eosio::multi_index`模板具有模板参数<uint64_t TableName,typename T，typename ... Indices>，其中：

- `TableName` 是表格的名称，最长12个字符，名称中的字符由小写字母，数字1至5和“.”组成。
- T 是对象类型.
- Indices 是最多16个二级指标的清单。
  - 每个必须是默认的可构造类或结构
  - 每个函数都必须有一个函数调用操作符，它接受对表格对象类型的const引用并返回辅助键类型或对辅助键类型的引用
  - 建议使用eosio :: const_mem_fun模板，它是boost :: multi_index :: const_mem_fun的一个类型别名。有关更多详细信息，请参阅Boost const_mem_fun密钥提取器的文档。

## 复制和分配

不支持复制构造函数和赋值运算符。

## 表操作

### emplace
|方法|描述|
|----|----|
|const_iterator emplace( unit64_t payer, Lambda&& constructor )|向表中添加一个新对象（即row）。|

|参数|描述|
|----|----|
|payer|新对象的存储使用情况的`payer`帐户名称|
|constructor|lambda函数，它可以在表中就地创建要创建的对象|

|返回|描述|
|----|----|
|返回|一个主键迭代器到新创建的对象|

|前提|描述|
|----|----|
|前提|`payer`是授权当前操作的有效帐户（因此允许为存储使用收费）|


|后置条件|描述|
|----|----|
|在多索引表中创建一个新对象，并使用唯一的主键（如对象中指定的）。该对象被序列化并写入表中。如果该表不存在，则创建该表。||
|二级索引被更新以引用新添加的对象。如果辅助索引表不存在，则会创建它们。||
|`payer`收取新对象的存储使用费，并且如果必须创建表（和二级索引表），则为表创建的开销。||
|Exceptions|当前接收者不是拥有该表格的帐户（multi_index的代码）。|

### get

|方法|描述|
|----|----|
|const object_type& get( uint64_t primary )const|使用主键从表中检索现有对象。|

|参数|描述|
|----|----|
|primary|对象的主键值|

|返回|描述|
|----|----|
|返回|对包含指定主键的对象的常量引用。|

|前提|描述|
|----|----|
|前提|||


|后置条件|描述|
|----|----|
|||
|Exceptions|没有任何对象与给定的键匹配|

### find 

|方法|描述|
|----|----|
|const_iterator find( uint64_t primary )const|使用主键在表中搜索现有对象。|

|参数|描述|
|----|----|
|primary|对象的主键值|

|返回|描述|
|----|----|
|返回|找到的对象的迭代器，其主键等于主键OR|
||`end`被引用表的迭代器，如果一个对象主键`primary`未找到|

|前提|描述|
|----|----|
|前提|None||


|后置条件|描述|
|----|----|
|后置条件|None|
|Exceptions|None|

### modify

|方法|描述|
|----|----|
|void modify( const_iterator itr, uint64_t payer, Lambda&& updater )|修改表中的现有对象。|
|void modify( const object_type &obj, uint64_t payer, Lambda&& updater )|||

|参数|描述|
|----|----|
|itr|指向要更新的对象的迭代器|
|obj|对要更新的对象的引用|
|payer|`payer`的账户名称用于更新行的存储使用情况;值为0表示修改后的行的`payer`与现有`payer`相同。|
|updater|用于更新目标对象的lambda函数|

|前提|描述|
|----|----|
|前提|itr指向，或obj是表中的现有对象。|
|前提|`payer`是授权当前操作的有效帐户（因此允许为存储使用收费）|


|后置条件|描述|
|----|----|
|后置条件|修改的对象被序列化，然后替换表中的现有对象。|
||二级指数更新;更新对象的主键不会更改。|
||`payer`收取更新对象的存储使用费。|
|后置条件|如果付款人与现有付款人相同，付款人仅支付现有和更新对象之间的使用差额（如果差额为负数，则退款）。|
|后置条件|如果付款人与现有付款人不同，现有付款人将退还现有物品的存储使用权。|
|Exceptions|如果用无效的前提条件调用，则中​​止执行。|
|Exceptions|当前接收者不是拥有该表格的帐户（multi_index的代码）。|

## erase

|方法|描述|
|----|----|
|const_iterator erase( const_iterator itr )|使用主键从表中删除现有的对象。|
|void erase( const object_type& obj )	|||

|参数|描述|
|----|----|
|itr|指向要删除的对象的迭代器|
|obj|对要删除的对象的引用|

|返回|描述|
|----|----|
|返回|对于使用const_iterator的签名，返回指向移除对象后面的对象的指针。|

|前提|描述|
|----|----|
|前提|None|


|后置条件|描述|
|----|----|
|后置条件|该对象将从表格中删除，并且所有关联的存储都将被回收。|
||与表格相关的二级索引被更新。|
||对于该对象的存储使用情况的现有付款人将退还已移除对象的表和次级索引使用情况，并且如果移除了表和索引，则为相关联的开销。|
|后置条件|如果付款人与现有付款人相同，付款人仅支付现有和更新对象之间的使用差额（如果差额为负数，则退款）。|
|Exceptions|要删除的对象不在表格中。|
|Exceptions|该操作无权修改表格。|
|Exceptions|给定的迭代器是无效的。|

# 成员访问

## get_code
|方法|描述|
|----|----|
|uint64_t get_code() const|返回代码成员属性。|
|Returns|拥有主表的代码的帐户名称|

## get_scope

|方法|描述|
|----|----|
|uint64_t get_scope() const|返回作用域成员属性。|
|Returns|在当前接收器的代码内的范围的范围ID，在该范围内可以找到期望的主表实例|

# Utilities

## available_primary_key

|方法|描述|
|----|----|
|uint64_t available_primary_key()const|返回一个可用（未使用的）主键值，打算用于表中主键必须为自动递增的表中，因此决不会将该键设置为自定义值。 由于无法分配可用的主键，违反此期望可能会导致表格显示为已满。|

# Iterators
多指数迭代器遵循C ++迭代器常用的模式。 所有迭代器都是双向常量，可以是const_iterator或const_reverse_iterator。 迭代器可以取消引用以提供对多索引表中的对象的访问。

## begin and cbegin

|方法|描述|
|----|----|
|const_iterator begin()const|返回指向多指标表中具有最低主键值的object_type的迭代器。|
|const_iterator cbegin()const	|||

## end and cend

|方法|描述|
|----|----|
|const_iterator end()const|返回一个迭代器，指向表示刚刚过去表的最后一行的虚拟行（假设存在）;不能被解除引用;可以向后推进，但不能向前推进。|
|const_iterator cend()const	|||

## rbegin and crbegin

|方法|描述|
|----|----|
|const_iterator rbegin()const	|返回指向多指标表中具有最高主键值的object_type的反向迭代器。|
|const_iterator crbegin()const|||

## rend and crend

|方法|描述|
|----|----|
|const_iterator rend()const|返回一个迭代器，指向表示刚刚过去表的最后一行的虚拟行（假设存在）;不能被解除引用;可以向前推进，但不能向后推进。|
|const_iterator crend()const|||

## lower_bound

|方法|描述|
|----|----|
|const_iterator lower_bound( uint64_t primary )const	|使用大于或等于给定主键的最低主键搜索object_type。|
|Parameters||
|primary|为下限搜索建立目标值的主键|
|Returns||
|指向具有大于或等于主键的最低主键OR的object_type的迭代器
||
|如果找不到对象，则包括`end`迭代器，包括如果表不存在|||

## upper_bound

|方法|描述|
|----|----|
|const_iterator upper_bound( uint64_t primary )const	|使用大于给定主键的最低主键搜索object_type。|
|Parameters||
|primary|为上限搜索建立目标值的主键|
|Returns||
|一个迭代器，指向具有大于主键的最低主键OR的object_type||
|如果找不到对象，则包括`end`迭代器，包括如果表不存在|||

## get_index

|方法|描述|
|----|----|
|secondary_index get_index<IndexName>()	|返回一个适当类型的二级索引。|
|secondary_index get_index<IndexName>() const	||
|Parameters||
|IndexName|所需的二级索引的ID|
|Returns|适当类型的索引，即以下之一:|
|idx64|原始的64位无符号整数密钥|
|idx128|原始128位无符号整数密钥|
|idx256|256位固定大小的字典键|
|idx_double|双精度浮点键|
|idx_long_double|四倍精度浮点键|

## iterator_to

|方法|描述|
|----|----|
|const_iterator iterator_to( const object_type& obj )const|返回多索引表中给定对象的迭代器。|
|secondary_index get_index<IndexName>() const	||
|Parameters||
|obj|对所需对象的引用|
|Returns|给定对象的迭代器|

## indexed_by

indexed_by结构用于实例化多索引表的索引。在EOSIO中，最多可以指定16个二级索引。
```
template<uint64_t IndexName, typename Extractor>
struct indexed_by {
   enum constants { index_name   = IndexName };
   typedef Extractor secondary_extractor_type;
};
```
### 参数

- IndexName 是索引的名称。 该名称必须作为EOSIO base32编码的64位整数提供，并且必须符合最多13个字符的EOSIO命名要求，前十二个来自小写字符az，数字0-5和“。”，并且if 有第13个字符，它仅限于小写字母ap和“.”。

- Extractor 是一个函数调用操作符，它接受对表格对象类型的const引用并返回辅助键类型或对辅助键类型的引用。 建议使用eosio :: const_mem_fun模板，它是boost :: multi_index :: const_mem_fun的一个类型别名。 有关更多详细信息，请参阅Boost const_mem_fun密钥提取器的文档

## 例子

```
multi_index<mytable, record,
     indexed_by< N(bysecondary), const_mem_fun<record, uint128_t, &record::get_secondary> >
  > table( code, scope);
```
在此示例中，多索引表名为mytable，具有对象（行）类型记录，由名为bysecondary的辅助索引编制索引，其名称使用N宏转换为所需的编码格式。 为记录对象类指定const_mem_fun键提取器; 键类型是uint128_t，使用record :: get_secondary函数获得。

# 该 eosio::multi_index::index API（二级指标）

EOSIO多指标表最多可以有16个二级指标。 索引可以使用get_index方法从multi_index对象中检索，然后可以使用通用的C ++迭代器模式进行迭代。

这里描述的索引API在许多方面都与上述的multi_index相同，但它在一些重要方面也有所不同。

- 索引旨在提供替代手段来访问多索引表。
- 二级索引表的内容基本上是映射到主键的键。该映射对用户是透明的。
- 有些操作不适用于二级索引，最值得注意的是，一种操作不使用二级索引来放置内容。可以使用辅助索引修改和删除对象。
- 使用辅助表检索主要内容的操作实际上是相同的，主要区别在于使用的键值（您无法将对象传递到辅助索引以在表中找到它）。 键值将是支持的二级索引类型之一：
  - idx64 - 原始的64位无符号整数密钥.
  - idx128 - 原始128位无符号整数密钥.
  - idx256 - 256位固定大小的字典键.
  - idx_double - 双精度浮点键.
  - idx_long_double - 四倍精度浮点键.

- 二级索引具有`code`和`scope`成员属性，与多索引表上的相应属性相同。

|别号|描述|
|---|---|
|object_type|Multi-Index表中的对象的类型|
|secondary_index|多索引表的相应二级索引的类型|

## 构造函数
辅助索引是作为多指数表构造的一部分而创建的。不支持直接构建索引。

## 复制和分配

不支持复制构造函数和赋值运算符。

## 表操作

### get

|方法|描述|
|----|----|
|const object_type& get( secondary_key_type&& secondary )const|使用其辅助键从表中检索现有对象。多个签名可用。|
|const object_type& get( const secondary_key_type& secondary )const|||

|参数|描述|
|----|----|
|secondary|对象的辅助键值|

|返回|描述|
|----|----|
|返回|对包含指定主键的对象的常量引用。|

|前提|描述|
|----|----|
|前提|||


|后置条件|描述|
|----|----|
|||
|Exceptions|没有任何对象与给定的键匹配|

### find 

|方法|描述|
|----|----|
|const_iterator find( secondary_key_type&& secondary )const|使用辅助键在表格中搜索现有对象。|
|const_iterator find( const secondary_key_type&& secondary )const|||

|参数|描述|
|----|----|
|secondary|对象的辅助键值|

|返回|描述|
|----|----|
|返回|找到的对象的迭代器，它具有等于辅助键的二级键，或未找到具有辅助键辅助键的对象时，结束迭代器|

|前提|描述|
|----|----|
|前提|None||


|后置条件|描述|
|----|----|
|后置条件|None|
|Exceptions|None|

### modify

|方法|描述|
|----|----|
|void modify( const_iterator itr, uint64_t payer, Lambda&& updater )|修改表中的现有对象。|
|void modify( const object_type &obj, uint64_t payer, Lambda&& updater )|||

|参数|描述|
|----|----|
|itr|指向要更新的对象的迭代器|
|obj|对要更新的对象的引用|
|payer|`payer`的账户名称用于更新行的存储使用情况;值为0表示修改后的行的`payer`与现有`payer`相同。|
|updater|用于更新目标对象的lambda函数|

|前提|描述|
|----|----|
|前提|itr指向，或obj是表中的现有对象。|
|前提|`payer`是授权当前操作的有效帐户（因此允许为存储使用收费）|


|后置条件|描述|
|----|----|
|后置条件|修改的对象被序列化，然后替换表中的现有对象。|
||二级指数更新;更新对象的主键不会更改。|
||`payer`收取更新对象的存储使用费。|
|后置条件|如果付款人与现有付款人相同，付款人仅支付现有和更新对象之间的使用差额（如果差额为负数，则退款）。|
|后置条件|如果付款人与现有付款人不同，现有付款人将退还现有物品的存储使用权。|
|Exceptions|如果用无效的前提条件调用，则中​​止执行。|
|Exceptions|当前接收者不是拥有该表格的帐户（multi_index的代码）。|

## erase

|方法|描述|
|----|----|
|const_iterator erase( const_iterator itr )|使用主键从表中删除现有的对象。|
|void erase( const object_type& obj )	|||

|参数|描述|
|----|----|
|itr|指向要删除的对象的迭代器|
|obj|对要删除的对象的引用|

|返回|描述|
|----|----|
|返回|对于使用const_iterator的签名，返回指向移除对象后面的对象的指针。|

|前提|描述|
|----|----|
|前提|None|


|后置条件|描述|
|----|----|
|后置条件|该对象将从表格中删除，并且所有关联的存储都将被回收。|
||与表格相关的二级索引被更新。|
||对于该对象的存储使用情况的现有付款人将退还已移除对象的表和次级索引使用情况，并且如果移除了表和索引，则为相关联的开销。|
|后置条件|如果付款人与现有付款人相同，付款人仅支付现有和更新对象之间的使用差额（如果差额为负数，则退款）。|
|Exceptions|要删除的对象不在表格中。|
|Exceptions|该操作无权修改表格。|
|Exceptions|给定的迭代器是无效的。|

# 成员访问

## get_code
|方法|描述|
|----|----|
|uint64_t get_code() const|返回代码成员属性。|
|Returns|拥有次要索引表的代码的账户名称|

## get_scope

|方法|描述|
|----|----|
|uint64_t get_scope() const|返回作用域成员属性。|
|Returns|范围在当前接收器的代码范围内，在该范围内可找到所需的二级索引表实例|

## name

|方法|描述|
|----|----|
|constexpr static uint64_t name()|返回index_table_name成员常量。|
|Returns|二级索引的表名|

## number

|方法|描述|
|----|----|
|constexpr static uint64_t number()|返回index_number成员常量。|
|Returns|二级索引的number|

# Iterators
EOSIO二级索引迭代器遵循C++迭代器常用的模式。

## const_iterator

|方法|描述|
|----|----|
|struct const_iterator : public std::iterator<std::bidirectional_iterator_tag, const object_type>|多指标表中的对象的双向迭代器。|
|参数||
|object_type|表中对象的类型名称|

## const_reverse_iterator

|方法|描述|
|----|----|
|typedef std::reverse_iterator<const_iterator> const_reverse_iterator|多指数表中的对象的反向双向迭代器。|


## begin and cbegin

|方法|描述|
|----|----|
|const_iterator begin()const|返回指向次级索引表中最低次级键值的迭代器。|
|const_iterator cbegin()const	|||

## end and cend

|方法|描述|
|----|----|
|const_iterator end()const|返回一个迭代器，指向表示刚刚过去表的最后一行的虚拟行（假设存在）;不能被解除引用;可以向后推进，但不能向前推进。|
|const_iterator cend()const	|||

## rbegin and crbegin

|方法|描述|
|----|----|
|const_iterator rbegin()const	|返回指向次级索引表中最高次级键值的反向迭代器。|
|const_iterator crbegin()const|||

## rend and crend

|方法|描述|
|----|----|
|const_iterator rend()const|返回一个迭代器，指向表示刚刚过去表的最后一行的虚拟行（假设存在）;不能被解除引用;可以向前推进，但不能向后推进。|
|const_iterator crend()const|||

## lower_bound

|方法|描述|
|----|----|
|const_iterator lower_bound( secondary_key_type&& secondary )const|搜索具有大于或等于给定次级密钥的最低次级密钥的object_type。|
|const_iterator lower_bound( const secondary_key_type&& secondary )const	||
|Parameters||
|secondary|建立下界搜索的目标值的辅助键|
|Returns||
|指向次级索引中具有大于或等于次级的最低次级关键字的对象的迭代器，或者如果找不到对象，则包括结束迭代器，包括如果表不存在||

## upper_bound

|方法|描述|
|----|----|
|const_iterator upper_bound( secondary_key_type&& secondary )const|使用大于给定次级密钥的最低次级密钥搜索object_type。|
|const_iterator upper_bound( const secondary_key_type&& secondary )const	||
|Parameters||
|secondary|为上限搜索建立目标值的辅助键|
|Returns||
|指向次级索引中具有大于次级的最低次级关键字的对象的迭代器，或者
如果找不到对象，则包括`end`迭代器，包括如果表不存在|||

## iterator_to

|方法|描述|
|----|----|
|const_iterator iterator_to( const object_type& obj )const|返回Multi-Index表中给定对象的二级索引迭代器。|
|Parameters||
|obj|对所需对象的引用|
|Returns|给定对象的迭代器|

# Utilities

## extract_secondary_key

|方法|描述|
|----|----|
|static auto extract_secondary_key(const object_type& obj)|从辅助索引中返回对象的辅助键值。|
|参数||
|obj|对所需对象的const引用|
|Return|该索引的给定对象的关键值|

## 本文地址：https://eosfans.io/topics/410
## 翻译自https://github.com/EOSIO/eos/wiki/Persistence-API