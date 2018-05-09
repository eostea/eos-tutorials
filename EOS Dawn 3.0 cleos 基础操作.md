# 创建缺省wallet
./cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JFmD7M6PzLQNgaXjeBWmGvJvFFtK5H4uRuAUSydJ8AqyyXq51Q"

# 加载 Bios 智能合约
./cleos set contract eosio ../../contracts/eosio.bios -p eosio

# 创建user账户
./cleos create key
Private key: 5K9Njtg7nLtmgMQHCxnggeZz2KZtSWt6mfR7wJM86aqHe731zeH
Public key: EOS8kBCd9riWHFMf3tmSsf57soNaYngoeYW5UyfXptQaAWaZ9qLei

./cleos wallet import 5K9Njtg7nLtmgMQHCxnggeZz2KZtSWt6mfR7wJM86aqHe731zeH

./cleos create account eosio user EOS8kBCd9riWHFMf3tmSsf57soNaYngoeYW5UyfXptQaAWaZ9qLei EOS8kBCd9riWHFMf3tmSsf57soNaYngoeYW5UyfXptQaAWaZ9qLei

# 创建tester账户
./cleos create key
Private key: 5K2i48pZTWQVtrBbWDGWi4PqXanzN7LYLjdSHVataMkrAoN74o8
Public key: EOS8Q4HcWNnCjXng4pShUTc7mfkewm7D5ysPBeY4iah2cht2Exq4T

./cleos wallet import 5K2i48pZTWQVtrBbWDGWi4PqXanzN7LYLjdSHVataMkrAoN74o8

./cleos create account eosio tester EOS8Q4HcWNnCjXng4pShUTc7mfkewm7D5ysPBeY4iah2cht2Exq4T EOS8Q4HcWNnCjXng4pShUTc7mfkewm7D5ysPBeY4iah2cht2Exq4T

# 创建eosio.token账户
./cleos create key
Private key: 5K5PXoAm1WjGeDCyqXuepkZaXk6xzxaEsZKp3nYY4pKCuNRDDvE
Public key: EOS7WjY1GcmPD9qNYXtqu55CrCeKi77fj5w3VtJhyBG1k6xGuB2Wc

./cleos wallet import 5K5PXoAm1WjGeDCyqXuepkZaXk6xzxaEsZKp3nYY4pKCuNRDDvE

./cleos create account eosio eosio.token EOS7WjY1GcmPD9qNYXtqu55CrCeKi77fj5w3VtJhyBG1k6xGuB2Wc EOS7WjY1GcmPD9qNYXtqu55CrCeKi77fj5w3VtJhyBG1k6xGuB2Wc

# 加载合约
./cleos set contract eosio.token ~/eos/build/contracts/eosio.token --permission eosio.token@active

# 创建Token
./cleos push action eosio.token create '[ "eosio", "1000000000.0000 EOS", 0, 0, 0]' -p eosio.token

# issue资产
./cleos push action eosio.token issue '["user", "100.0000 EOS", "memo"]' -p eosio
./cleos push action eosio.token issue ‘["tester", "100.0000 EOS", "memo"]' -p eosio

# 查看资产
./cleos get table eosio.token tester accounts
./cleos get table eosio.token user accounts
./cleos get currency balance eosio.token tester EOS


# transfer资产
./cleos push action eosio.token transfer '{"from":"user","to":"tester","quantity”:"1.1200 EOS","memo":"my-first-transfer"}’ -p user
./cleos push action eosio.token transfer '[ "user", "tester", "1.2340 EOS", "m" ]' -p user