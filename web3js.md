#### web3.js库是一个针对EVM包含的函数定制化的一个javascript库
* web3-eth是针对以太坊区块链何智能合约的模块.
* web3-shh是用于连接p2p以及广播的whisper协议.
* web3-bzz用于去中心化文件存储的swarm协议.
* web3-utils对于去中心化app的开发者，包含一些有用的帮助项.

#### 添加web3.js
* npm : npm install web3 
* meteor : meteor add ethereum:web3
* pure js: link the dist/web3.min.js

完成以后，你需要创建一个web3的实例，并且设置一个rpc的服务提供者。如果发生连接错误。我们可以通过打开本地节点的http rpc连接。`admin.startRPC("127.0.0.1", 8545, "*", "web3,net,eth")`
```
var web3 = new Web3()
web3.setProvider(new web3.providers.HttpProvider());
```
这样你就可以使用web3这个对象了。

#### 回调保证事件
为了帮助web3集成到各种项目中，我们提供多种方式完成异步的函数。大多数web3.js对象都允许在函数的末尾添加一个回调参数。也是一个返回到链上函数的一个保障。
以太坊区块链需要不同级别的输出，因此对于同一个行为可能有不同级别的返回值。因为这个需求，我们返回一个可靠事件，对于像web3.eth.sendTransaction或者是合约方法。这种可靠事件是一个保证可以产生不同级别的行为返回值在区块链上，比如一个事物。

保证事件就像普通保证一样只不过添加了`on`,`once`,`off`函数。这样开发着观察额外的事件，例如"receipt" 或者是 "transactionHash"
```
web3.eth.sendTransaction({from:'address1',data:'data1'}).once('transactionHash',function(hash){...})
.once('receipt',function(receipt){...})
.on('confirmation',function(confNumber,receipt){...})
.on('error',function(error){...})
.then(function(receipt){
  // 一旦交易确认下面就会执行
});
```

#### json interface
json interface 是针对以太坊智能合约描述其接口的一个json对象。

具体来讲：
function：
* type：“function” or "constructor";
* name: the name function
* constant : true , 如果保证不改变区块链的状态
* payable : true , 如果函数可以接受ether , default false;
* stateMutability: pure/view(constant)/nonpayable/payable
* inputs:一个数组对象，包含着
  -name:参数名字
  -type:参数类型
* outputs:一个数组对象，同inputs

events:
* type:“event”
* name:"inputs"
