# Scroll Pre-alpha Testnet

## 一、准备
#### 登录：https://scroll.io/portal
#### 将网络（Goerli 测试网和 Scroll Alpha 测试网）添加到 metamask 钱包
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/a04fc52b-6f5d-4772-afd1-fb6898523969)

#### 想办法得到GETH，并将 ETH 从 goerli testnet 发送到 Scroll Alpha testnet 网络
#### 登录：https://scroll.io/alpha/bridge
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/dcf537d9-3b68-40b4-94ff-26a5e3957d64)



## 二、部署智能合约
#### https://remix.ethereum.org/

#### 新建合约文件，填写文件名称
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/47f6f537-8199-4be8-96ae-7823f2d01fea)

#### 将如下代码复制到右侧空白处
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.4;

    contract FunctionTypes{
        uint256 public number = 5;
        
        constructor() payable {}
        // 函数类型
        // function (<parameter types>) {internal|external} [pure|view|payable] [returns (<return types>)]
        // 默认function
        function add() external{
            number = number + 1;
        }
        // pure
        function addPure(uint256 _number) external pure returns(uint256 new_number){
            new_number = _number+1;
        }
        
        // view
        function addView() external view returns(uint256 new_number) {
            new_number = number + 1;
        }
        // internal: 内部
        function minus() internal {
            number = number - 1;
        }
        // 合约内的函数可以调用内部函数
        function minusCall() external {
            minus();
        }
        // payable: 能给合约支付eth的函数
        function minusPayable() external payable returns(uint256 balance) {
            minus();    
            balance = address(this).balance;
        }
    }

#### 编译合约，版本选择soljson-v0.8.18+commit.87f61d96，再点击编译按钮
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/5c97cdcb-155d-41a2-9b81-f479f3aa3991)

#### 部署合约，选择 Injected provider (metamask)，确保小狐狸钱包在Scroll Alpha 网络上并检查地址是否正确，点击部署按钮并在小狐狸钱包中确认
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/d4f0ae04-7dba-41b3-8aad-dba6bb07c838)

#### 如果合约部署成功，将看到如下所示的消息
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/6e8fcc75-da38-4305-84fb-8748e7396568)



## 三、创建自定义token
#### 新建合约文件，填写文件名称
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/deb2a8db-ca89-4a3e-a788-9e39a909b6e1)
#### 将如下代码复制到右侧空白处
    // SPDX-License-Identifier: MIT
    // WTF Solidity by 0xAA
    pragma solidity ^0.8.4;
        interface IERC20 {
        /**
         * @dev 释放条件：当 `value` 单位的货币从账户 (`from`) 转账到另一账户 (`to`)时.
         */
        event Transfer(address indexed from, address indexed to, uint256 value);
        /**
         * @dev 释放条件：当 `value` 单位的货币从账户 (`owner`) 授权给另一账户 (`spender`)时.
         */
        event Approval(address indexed owner, address indexed spender, uint256 value);
        /**
         * @dev 返回代币总供给.
         */
        function totalSupply() external view returns (uint256);
        /**
         * @dev 返回账户`account`所持有的代币数.
         */
        function balanceOf(address account) external view returns (uint256);
        /**
         * @dev 转账 `amount` 单位代币，从调用者账户到另一账户 `to`.
         *
         * 如果成功，返回 `true`.
         *
         * 释放 {Transfer} 事件.
         */
        function transfer(address to, uint256 amount) external returns (bool);
        /**
         * @dev 返回`owner`账户授权给`spender`账户的额度，默认为0。
         *
         * 当{approve} 或 {transferFrom} 被调用时，`allowance`会改变.
         */
        function allowance(address owner, address spender) external view returns (uint256);
        /**
         * @dev 调用者账户给`spender`账户授权 `amount`数量代币。
         *
         * 如果成功，返回 `true`.
         *
         * 释放 {Approval} 事件.
         */
        function approve(address spender, uint256 amount) external returns (bool);
        /**
         * @dev 通过授权机制，从`from`账户向`to`账户转账`amount`数量代币。转账的部分会从调用者的`allowance`中扣除。
         *
         * 如果成功，返回 `true`.
         *
         * 释放 {Transfer} 事件.
         */
        function transferFrom(
            address from,
            address to,
            uint256 amount
        ) external returns (bool);
    }
    contract ERC20 is IERC20 {
        mapping(address => uint256) public override balanceOf;
        mapping(address => mapping(address => uint256)) public override allowance;
        uint256 public override totalSupply;   // 代币总供给
        string public name;   // 名称
        string public symbol;  // 符号
        
        uint8 public decimals = 18; // 小数位数
        address public owner;
        // @dev 在合约部署的时候实现合约名称和符号
        constructor(string memory name_, string memory symbol_){
            name = name_;
            symbol = symbol_;
            owner = msg.sender;
        }
        // @dev 实现`transfer`函数，代币转账逻辑
        function transfer(address recipient, uint amount) external override returns (bool) {
            balanceOf[msg.sender] -= amount;
            balanceOf[recipient] += amount;
            emit Transfer(msg.sender, recipient, amount);
            return true;
        }
        // @dev 实现 `approve` 函数, 代币授权逻辑
        function approve(address spender, uint amount) external override returns (bool) {
            allowance[msg.sender][spender] = amount;
            emit Approval(msg.sender, spender, amount);
            return true;
        }
        // @dev 实现`transferFrom`函数，代币授权转账逻辑
        function transferFrom(
            address sender,
            address recipient,
            uint amount
        ) external override returns (bool) {
            allowance[sender][msg.sender] -= amount;
            balanceOf[sender] -= amount;
            balanceOf[recipient] += amount;
            emit Transfer(sender, recipient, amount);
            return true;
        }
        // @dev 铸造代币，从 `0` 地址转账给 调用者地址
        function mint(uint amount) external {
            require(owner == msg.sender);
            balanceOf[msg.sender] += amount;
            totalSupply += amount;
            emit Transfer(address(0), msg.sender, amount);
        }
        // @dev 销毁代币，从 调用者地址 转账给  `0` 地址
        function burn(uint amount) external {
            balanceOf[msg.sender] -= amount;
            totalSupply -= amount;
            emit Transfer(msg.sender, address(0), amount);
        }
    }

#### 编译合约，版本选择soljson-v0.8.18+commit.87f61d96，再点击编译按钮
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/51f31c49-bc1d-49bb-80a4-081987ee6583)

#### 部署合约，选择 Injected provider (metamask)，确保小狐狸钱包在Scroll Alpha 网络上并检查地址是否正确。点开【DEPLOY】右侧箭头，填写【NAME、symbol】，点击【transact】按钮并在小狐狸钱包中确认
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/816e9b79-ca84-44b6-844b-177fd3f024d9)

#### Mint Token，点开【ERC20 AT...】左侧的箭头，打开【mint】右侧的箭头，输入金额 1800000000000000000000，点击交易按钮并在小狐狸钱包中确认
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/9042440b-a1f7-4b07-b14a-e35335dca9f9)

#### 将token导入小狐狸钱包，复制合约地址，打开小狐狸钱包，自定义添加token
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/c389e738-7c52-4c42-b8ee-c7bc48c241eb)

#### 将合约地址粘添到输入框中，添加token
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/374a3f45-de69-4e28-b1af-6e3af5fd957f)

#### 使用自定义token添加流动性
#### 登录：https://uniswap-v3.scroll.io/#/pool
#### 点击【new position】按钮，点击【Select a token】，在弹出窗口里的输入框中，填写自定义token的合约地址
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/2ef34c78-3e5d-4df9-ab81-81041f30f573)

#### 选择查询出来的token，确认风险提示
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/b5e66230-a902-44a6-b55b-ee437394b077)

#### 添加流动性，可按照下图数据填写
![image](https://github.com/ZoeDTiger/Scroll-Alpha-Testnet/assets/100336530/50c63362-0dc5-4cc7-b90a-7b5ec56b040e)

## 自定义token的交易，想办法交易100次！！！





































