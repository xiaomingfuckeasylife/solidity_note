# solidity_note

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
  // 定义了一个所谓的事件，在调用他的时候，实际上调用的是最后的一个方法，send注意这个地方的使用规范，我们的接口使用的方法名字是大写
  // 调用的方法名字对应的名字只不过首字母小写。用户界面（或者服务器）能够监听那些在区块链上执行的时间并且不用对内存的很大的消耗。只要方法执行以后
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

