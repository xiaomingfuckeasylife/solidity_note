## solidity_note

A simple smart contract 
```go
pragma solidity ^0.4.0; // pragma solidity 说明用哪一种编译器编译代码。 0.4.0 表明使用的编译器版本 ^表示只要版本在4以上5以下的都是可以编译的

contract SimpleStorage{// 一个合约是一串代码的集合其中包含有函数也有数据，由某一个地址发布的，并且存放在区块链上。

  uint storedData;     // storedData 是一个uint类型的（无类型整型，大小为256bits，32byte）你可以把这个变量当做数据库中的某一个表。你可以通过调用方法对这个值进行修改和获取。
  
  function set(uint x){ // 修改storedData的值
    storedData = x;
  }
  
  function get() constant returns (uint){ // 获取storedData的值
    return storedData;
  }
  // 提醒： 所有的标识符（合约名字，函数名字，变来名字）被要求使用ASCII编码。对于string 变量也是可以存储UTF-8字符的。
  // 警告： 不要使用UNICODE编码符，因为他们的被编码成字节数组的方式是不一样的。
}
```

subcurrency exmaple
```go
pragma solidity ^0.4.0;

contract Coin {
  // public 使得变量minter可以在外部进行访问 minter是一个地址类型 地址类型是一个160-bit值并且不允许进行任何的算术操作。
  // 他通常用于存储只能合约的地址或者属于外部人员们的键值对
  // public 会自动生成一个函数让你可以访问这个变量.不用使用关键字this，其他的智能合约不能够访问这个变量
  address public minter; 
  // mapping 类型 key为地址，value为uint类型 用于存储地址对应的
  // public 自动生成的函数大致像这个样子。
  /*
  function balances(address _account) return (uint) {
    return balances[_account];
  }
  可以看到我们可以使用这个函数轻松的访问到这个账户的余额。
  */
  mapping(address=>uint) public balances;
  // 事件可以让轻客户端进行有效的调用
  // 定义了一个所谓的事件，在调用他的时候，实际上调用的是send最后的一行
  // 用户界面（或者服务器）能够监听那些在区块链上执行的时间并且不用对内存的很大的消耗。只要方法执行以后
  // 监听器也能够接收到这些参数包括from,to,amount。这样就能很容易的跟踪交易事物。如果想要这个事件进行监听我们可以执行下面的代码
  /*
  Coin.Sent().watch({},function(error,result){
      if (!error){
        console.log("Coin transfer: " +result.args.amount + " coins were sent from " + result.args.from + " to " + result.args.to);
        console.log("Balances now:\n" + "Sender: " + Coin.balances.call(result.args.from) + "Receivers: " + Coin.balances.call(result.args.to));
      }
    }
  )
  // 我们可以看到我们是如何调用balances的方法，使用的是类似JavaScript的原型链的call方法。
  */
  event Sent(address from,address to,uint amount);
  // 构造器只在智能合约创建的时候会执行。
  function Coin(){
    minter = msg.sender;
  }
  // 造币，只准合约拥有者造币
  function mint(address receiver,uint amount){
    if(msg.sender != minter) return;
    balances[receiver]+=amount;
  }
  // 发钱
  function send(address receiver,uint amount){
    if(balances[msg.sender] < amount) return;
    balances[msg.sender] -= amount;
    balances[receiver] += amount;
    Sent(msg.sender,receiver,amount);
  }
  
  // 后续： msg （tx , block）是一个很魔性的变量，让我们能够获取区块链的一些内部特性。msg.sender总是指向执行当前方法的地址。
}
```

## 区块链常识

### 事物
类似于Oracle的事物，因为都是数据库，所以这样类比起来，容易理解。

### 区块
什么是区块呢，区块是存放交易的地方，在比特币中有一种被称为“双花”的东西。也就是说，如果有两个事物都存在网络中，都想清空账户，是否会产生冲突呢？答案是你不用关心，其中一笔会被选出来并加入到区块链中，另外一笔会被拒绝掉，并且不会被加入到区块链中。
区块按照线性加入到区块中，这也是区块链的由来，区块链加入到区块中成相对固定的时间段，ETH大概每隔17秒会形成一个区块。
作为自然选择的一部分区块可能会是不是的回滚，但是等待的时间越长回滚的可能性就月底，一般ETH在达到30个确认之后回滚的可能性就极地了。

### 以太坊虚拟机

#### 概述
EVM是让智能合约在以太坊上面运行的虚拟机。他不仅是沙河并且完全隔离，运行在EVM的代码不能连接网络文件系统或者其他线程。就算是智能合约也只有部分权限访问
其他的智能合约

#### 账户
在ETH中有两种账户类型，一种被称为外部账户，通过键值对的方式保存，另一种被称为合约账户，通过保存代码的账户决定。
外部账户通过公钥决定，合约地址是在和合约被创建时候的创建合约的地址，以及在创建时候有多少笔交易发生决定的。
不管有没有存储代码，这两种类型的账户都被同等对待
每一个账户都是64byte的key存储64byte的value，并且每一个账户都有一个余额在Ether中被称之为“Wei”，这个值在交易Ether的时候会发生变化

#### 事物
一个事物是一条消息从一个账户发送到另外一个账户。
如果目标账户包含有代码，那么代码会被执行发送方的发送信息回座位输入数据。
如果目标账户是一个0类型账户，那么这个事物会创建一个新的智能合约，正如我们提到的，智能合约的地址并不是0地址，它是由合约创建地址和创建时候的事物决定的，payload会作为EVM的字节码被执行。这个执行的输出会永久的存放在智能合约中。所以为了创建一个智能合约，你不需要真的把代码存起来，只需要存着能返回那些代码的代码就可以了。






