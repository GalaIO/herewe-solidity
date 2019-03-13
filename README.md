solidity是以太坊的智能合约语言，随着solidity的流行，现在很多区块链的vm虚拟机都兼容solidity开发合约，但是区块链的开发不同于普通开发，在合约安全、随机数、权限控制、合约升级、gas消耗、存储优化等有很多不同的地方，同时由于合约一旦部署无法更改，所以一定要小心。这样一来一篇solidity的最佳实践和开发的startup脚手架的必须的。

### 智能合约的永固性

你把智能协议传上以太坊之后，它就变得不可更改, 这种永固性意味着你的代码永远不能被调整或更新。

你编译的程序会一直，永久的，不可更改的，存在以太坊上。这就是 Solidity 代码的安全性如此重要的一个原因。如果你的智能协议有任何漏洞，即使你发现了也无法补救。你只能让你的用户们放弃这个智能协议，然后转移到一个新的修复后的合约上。

但这恰好也是智能合约的一大优势。代码说明一切。如果你去读智能合约的代码，并验证它，你会发现，一旦函数被定义下来，每一次的运行，程序都会严格遵照函数中原有的代码逻辑一丝不苟地执行，完全不用担心函数被人篡改而得到意外的结果。

同样对于合约我们可以升级或者提前设计，利用中心化手段进行合约动作熔断，中心化和非中心化需要自行权衡。

### 基本元素

solidity的语法和js很像，首先遍历一下solidity的关键字以及它们的用途，注意这里是大部分，不是全部；

```solidity

// 声明solidity的编译版本

pragma solidity >0.4.19;



// 通过相对目录引入合约

//import "./ownable.sol";



// 申明一个合约接口，可以申明通用的，升级合约也不会变的接口级结构，比如ERC20

// 声明与abi的接口一致

contract KittyInterface {

  function getKitty(uint256 _id) external view returns (

    bool isGestating,

    bool isReady

  );

}



// 声明一个合约，在以太中属于合约账户，存在一个地址，可以存储、发送余额

// is 声明继承自哪个合约，可以复用代码

contract ZombieFactory is Ownable {



    // 声明事件，可以使用emit触发事件

    uint=uint256，evm默认字长为256位，除了在结构体申明的非uint256，其他都会转为uint256

    // string 可以存储任意长字符串，但是为了节约存储，最好保存少点

    event NewZombie(uint zombieId, string name, uint dna);



    // 通过合约地址实例化合约，可以通过函数动态生成合约

    // 调用 (,isReady) = kittyContract.getKitty(_kittyId);

    address _address = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;

    KittyInterface kittyContract = KittyInterface(_address);



    // 声明状态变量，所有在合约内声明的变量都是状态变量，存储在区块链中的storage，持久化在区块链中

    // 很神奇吧，除了在第一次初始创建时，是用合约部署费用创建的，其他对状态变量的修改，都会触发存储fee，存储的fee比较高

    // ether为以太坊余额单位 1 ether = 1000 finney = 10**18 wei

    uint balance = 1 ether;

    // days 表示时间 其实是语法糖 1 days = 3600 * 24

    uint cooldownTime = 1 days;



    // struct表示结构体，可以自定义类型，

    // 同时struct中的uint16 uint32会优化为占用较小的存储，节约gas

    // 存储的string最好在初始保存时，校验string大小，避免用超

    struct Zombie {

      string name;

      uint dna;

      uint32 level;

    }



    // 数组声明，声明为public，外部可见，部署存在一个默认生成getter函数，外部合约可以直接访问变量

    Zombie[] public zombies;



    // solidity不能返回自定义状态变量，只能返回memory变量

    function zombies(uint index) view public returns (

      string memory name,

      uint dna,

      uint32 level

    ){

        return (zombies[index].name, zombies[index].dna, zombies[index].level);

    }



    // 没有描述访问权限的状态变量，默认权限是internal，只能被当前合约和子合约访问

    // mapping结构定义了类似字典结构，可以定义一种映射关系，可以用于快速查找

    // 利用一个数组 Zombie[]，两个mapping存储Zombie的index，来提高查找效率，同时避免一个地址映射到一个数组，如果删除会导致数组整体移动，耗费过高存储

    mapping (uint => address) public zombieToOwner;

    mapping (address => uint) ownerZombieCount;



    // 定义函数修饰器，可以声明在函数后，共享方法入参，执行该函数时，首先执行修饰器，然后在_占位符执行被修饰函数

    // 对于入参或者internal的函数，推荐以_开头命名，职责清楚

    modifier ownerOf(uint _zombieId) {

        // 使用断言来进行条件判断，如果校验失败会报错

        // 使用require(bool condition, string message)返回错误message

        require(msg.sender == zombieToOwner[_zombieId]);

        _;

    }



    // 定义一个函数，常用定义如：

    // function (funcName) (<parameter types>) {public|external|internal|private} [view|pure|payable] [returns (<return types>)]

    // 函数访问权限修饰符 public内部、子合约及外部合约可调用、external只允许外部合约调用、internal只允许内部或子合约调用、private只允许内部合约调用

    // 函数行为修饰符 view承若只查看状态变量不修改，pure承诺不和合约状态变量交互，payable承诺会存在以太坊转账操作，注意如果转账没有payable修饰会报错

    // 函数返回值，可以返回多个值

    function queryZombie(uint _zombieId) public view ownerOf(_zombieId) returns (string memory name, uint dna, uint32 level) {

        Zombie memory zombie = zombies[_zombieId];

        return (zombie.name, zombie.dna, zombie.level);

    }

    

    // 向合约支付一定余额，使用payable修饰

    function paysomething(uint balabce) external payable {

        this.transfer(balabce);

    }



    // 合约向owner支付所有余额

    function withdraw() external onlyOwner {

        owner.transfer(this.balance);

    }

}

```

### 节约gas

- 优化存储结构，比如在struct中使用uint32等较小的类型，避免使用大类型；

- 在存储用户string或者变长数组时，注意限制用户的输入，避免存储成本剧增；

- 避免频繁变更存储，如果要操作数组或者自定义结构体可以先在内存建立memory变量，随后考虑是否写入storage；

- 在明确不会写入数据时，使用view和pure修饰的函数，不会造成额外的gas；


### 内置对象

全局命名空间中总是存在一些特殊的变量和函数，它们主要用于提供关于区块链的信息，或者是通用的实用工具函数。



- 块和交易属性



block.blockhash(uint blockNumber) returns (bytes32): hash of the given block - only works for 256 most recent, excluding current, blocks - deprecated in version 0.4.22 and replaced by blockhash(uint blockNumber).

block.coinbase (address): current block miner’s address

block.difficulty (uint): current block difficulty

block.gaslimit (uint): current block gaslimit

block.number (uint): current block number

block.timestamp (uint): current block timestamp as seconds since unix epoch

gasleft() returns (uint256): remaining gas

msg.data (bytes): complete calldata

msg.gas (uint): remaining gas - deprecated in version 0.4.21 and to be replaced by gasleft()

msg.sender (address): sender of the message (current call)

msg.sig (bytes4): first four bytes of the calldata (i.e. function identifier)

msg.value (uint): number of wei sent with the message

now (uint): current block timestamp (alias for block.timestamp)

tx.gasprice (uint): gas price of the transaction

tx.origin (address): sender of the transaction (full call chain)



>注意：msg 的所成员变量，包括msg.sender和msg.value可以随着每次外部函数调用而改变。这包括对库函数的调用。

不要依赖 block.timestamp,now 和 blockhash 作为机性的来源，除非你知道自己在做什么。

时间戳和块散列在某种程度上都可以受到挖掘程序的影响。例如，采矿社区中的不良行为者可以在选定的散列上运行赌场支付函数，如果他们没有收到任何钱，就重新尝试不同的散列。

当前块时间戳必须严格大于上一个块的时间戳，但唯一的保证是它将介于标准链中两个连续块的时间戳之间。

由于可伸缩性的原因，块散列不能用于所有块。您只能访问最近的256块的散列，所有其他值都为零。



- 账户相关 Address Related



<address>.balance (uint256): 返回Wei单位的余额

<address>.transfer(uint256 amount): 给该Address发送 Wei单位的amount，默认消耗2300gas ，不可调整

<address>.send(uint256 amount) returns (bool):

给该Address发送 Wei单位的amount， 失败的时候返回false，默认消耗2300gas ，不可调整

<address>.call(...) returns (bool): 失败的时候返回false，转发所有gas，可调整

<address>.callcode(...) returns (bool):失败的时候返回false，转发所有gas，可调整

<address>.delegatecall(...) returns (bool):失败的时候返回false，转发所有gas，可调整

- 内置函数



以下函数可以根据方法和返回值的字面意思理解，因为这里翻译很怪 ，或者参考资料

addmod(uint x, uint y, uint k) returns (uint):

mulmod(uint x, uint y, uint k) returns (uint):

keccak256(...) returns (bytes32): 计算keccak256摘要

sha256(...) returns (bytes32):

sha3(...) returns (bytes32):

ripemd160(...) returns (bytes20):

ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address):



### 代码复用

is继承

modify修饰器

内部函数

合约调用



### ERC20与ERC741



### Oracle预言机



### 开发脚手架工程