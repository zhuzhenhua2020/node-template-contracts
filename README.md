# 第五课
完成并部署一个erc20的智能合约
## 安装先决条件
```
rustup component add rust-src --toolchain nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```
## 安装 binaryen
方法1：
```
sudo apt install binaryen
wasm-opt --version
```
编译需要version_99以上版本，如果apt方式版本太低，使用源代码编译方式

方法2：
```
sudo apt install cmake
wget https://github.com/WebAssembly/binaryen/archive/refs/tags/version_102.tar.gz
tar zvxf version_102.tar.gz && cd binaryen-version_102/
cmake . && make
echo 'PATH=$PATH:/home/vico/binaryen-version_102/bin' >> ~/.bashrc
source ~/.bashrc
echo $PATH 
wasm-opt --version
```
## 安装 Substrate Contracts 节点
```
cargo install contracts-node --git https://github.com/paritytech/substrate-contracts-node.git --tag v0.1.0 --force --locked
```
## 安装cargo-contract
```
cargo install cargo-contract --vers ^0.14 --force --locked
ll ~/.cargo/bin/cargo-contract
```
## 编译erc20合约
- 合约的存储
```
pub struct Erc20 {
    total_supply: Lazy<Balance>,
    balances: StorageHashMap<AccountId, Balance>,
    allowances: StorageHashMap<(AccountId, AccountId), Balance>,
}
```
- 初始化方法
```
#[ink(constructor)]
pub fn new(initial_supply: Balance) -> Self {
    let caller = Self::env().caller();
    let mut balances = StorageHashMap::new();
    balances.insert(caller, initial_supply);
    let instance = Self {
        total_supply: Lazy::new(initial_supply),
        balances,
        allowances: StorageHashMap::new(),
    };
    Self::env().emit_event(Transfer {
        from: None,
        to: Some(caller),
        value: initial_supply,
    });
    instance
}
```
- 获取总发行量
```
#[ink(message)]
pub fn total_supply(&self) -> Balance {
    *self.total_supply
}
```
- 获取指定账号的Token
```
#[ink(message)]
pub fn balance_of(&self, owner: AccountId) -> Balance {
    self.balances.get(&owner).copied().unwrap_or(0)
}
```
- 转账
```
#[ink(message)]
pub fn transfer(&mut self, to: AccountId, value: Balance) -> Result<()> {
    let from = self.env().caller();
    self.transfer_from_to(from, to, value)
}
```
- 授权额度
```
#[ink(message)]
pub fn allowance(&self, owner: AccountId, spender: AccountId) -> Balance {
    self.allowances.get(&(owner, spender)).copied().unwrap_or(0)
}
```
- 完整代码

[https://gitee.com/vicowong/node-template-contracts/blob/master/erc20/lib.rs](http://https://gitee.com/vicowong/node-template-contracts/blob/master/erc20/lib.rs)

### 合约测试
```
cargo +nightly test
```
![Erc20合约测试](https://gitee.com/vicowong/node-template-contracts/raw/master/assets/Erc20%E5%90%88%E7%BA%A6%E6%B5%8B%E8%AF%95.png "Erc20合约测试")
### 合约编译
```
cargo +nightly contract build
```
![Erc20合约编译](https://gitee.com/vicowong/node-template-contracts/raw/master/assets/Erc20%E5%90%88%E7%BA%A6%E7%BC%96%E8%AF%91.png "Erc20合约编译")

## 合约部署：
### 启动contracts节点
```
substrate-contracts-node --dev --tmp
```
![contract节点运行](https://gitee.com/vicowong/node-template-contracts/raw/master/assets/contract%E8%8A%82%E7%82%B9%E8%BF%90%E8%A1%8C.png "contract节点运行")
### 打开前端 
https://paritytech.github.io/canvas-ui/#/
### 上传已经编译好的Erc20
![上传Erc20合约](https://gitee.com/vicowong/node-template-contracts/raw/master/assets/%E4%B8%8A%E4%BC%A0Erc20%E5%90%88%E7%BA%A6.png "上传Erc20合约")
### 初始化合约，设置总发行量
![初始化Erc20](https://gitee.com/vicowong/node-template-contracts/raw/master/assets/%E5%88%9D%E5%A7%8B%E5%8C%96Erc20.png "初始化Erc20")
![合约调用](https://gitee.com/vicowong/node-template-contracts/raw/master/assets/%E5%90%88%E7%BA%A6%E8%B0%83%E7%94%A8.png "合约调用")
### 调用合约，查询总发行量
![查询总发行量](https://gitee.com/vicowong/node-template-contracts/raw/master/assets/%E8%B0%83%E7%94%A8%E5%90%88%E7%BA%A6%E6%9F%A5%E8%AF%A2%E6%80%BB%E5%8F%91%E8%A1%8C%E9%87%8F.png "查询总发行量")
### 调用合约，转账
![调用合约转账](https://gitee.com/vicowong/node-template-contracts/raw/master/assets/%E8%B0%83%E7%94%A8%E5%90%88%E7%BA%A6%E8%BD%AC%E8%B4%A6.png "调用合约转账")