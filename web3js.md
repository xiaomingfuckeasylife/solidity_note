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
* name:"inputs" 一个数组对象，包含着：
  - name : 参数名称
  - type : 参数类型
  - indexed:true 
* anonymous:true 如果事件被定义成匿名的。


#### web3
web3是一个用来和以太坊相关模块交互的对象
```js
>var Web3 = require('web3');
>Web3.utils // utils module
>Web3.version // web3 version 
>Web3.modules // show all the modules

>var web3 = new Web3('ws://localhost:8546'); 
>web3.eth  // eth module 
>web3.shh  // shh module
>web3.bzz  // bzz module
>web3.utils// utils module
>web3.version // show version

```
#### 使用provider连接web3不同的模块
```js
// you can 
var Web3 = require('web3');
var web3 = new Web3("http://localhost:8545");

// or 
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:8545'));

// change provider 
web3.setProvider('http://localhost:8545');

// or
var web3 = new Web3(new Web3.providers.WebsocketProvider('ws://localhost:8646'));

// using the IPC Provider in node.js
var net = require('net');
var web3 = new Web3('/Users/clark/Library/Ethereum/geth.ipc',net); // mac os path

// or 
var web3 = new Web3(new Web3.providers.IpcProvider('/Users/clark/Library/Ethereum/geth.ipc'),net);
```
#### prodivers 
* Object-HttpProvider: httpProvider is deprecated , as it won't work for subscriptions.
* Object-WebsocketProvider:The Websocket provider is the standard for usage in legacy browsers.
* Object-IpcProvider:The IPC provider is used node.js dapps when running a local node , Gives the most secure connection.

##### currentProvider
当前的连接提供者,如果没有返回null
```
web3.currentProvider
web3.eth.currentProvider
web3.shh.currentProvider
web3.bzz.currnetProvider
```
##### BatchRequest
批量请求
```
web3.batchRequest();
web3.eth.batchRequest();
web3.shh.batchRequest();
..
```
不需要任何的参数，返回一个对象可以接入下面两种方法
* add(request): add a request object to the batch call . 
* execute(): will execute the batch request. 
```js
var contract = new web3.eth.Contract(abi,address);
var batch = new web3.BatchRequest();
batch.add(web3.eth.getBalance.request('0x00000','latest',callback));
batch.add(contract.methods.balance(address).call.request({from:'0x000'},callback2));
batch.execute();
```
##### extend 
继承模块
```js
web3.extend(methods);
web3.eth.extend(methods);
web3.shh.extend(methods);
web3.bzz.extend(methods);
```
参数
1.methods - Object:Extension object with array of methods description objects as follows:
* property - String: the name of the property to add to the module. If no propert is set it willl be added to the module directly . 
* methods-Array:the array of method descriptions:
  -name - String : Name of the method to be added .
  -call - String : The RPC method name . 
  -params - Number:(optional) The number of parameters for that function. defualt().
  -inputFormatter  - Array:
  -outputFormatter - function
2 returns : the extended modules  
```js
web3.extend({
  property:'myModule',
  methods:[{
    name : 'getBalance',
    call : 'eth_getBalance',
    params:2,
    inputFormatter:[web3.extend.formatters.inputAddressFormatter,web3.extend.formatters.inputDefaultBlockNumberFormatter],
    outputFormatter:web3.utils.hexToNumberString
  }]
});

web3.extend({
 method:[{
  name:'directCall',
  call:'eth_callForFun'
 }]
})
```

#### web3.eth
web3-eth can allow us interact with ethereum blockchain and smart contract . 
```
var Eth = require('web3-eth');
var eth = new Eth('http://localhost:8545');
```

#### checksum addresses 
because all ethereum address returned by functions of this package are returned as checksum addresses . this means some letters are uppercase and some are lowercase. Based on that it will calculate a checksum for the address and prove its correctness.  

#### setProvider 
```
web3.setProvider(...);
web3.eth.setProvider(..);
... // this will change the provider for the module. 
```

#### providers
```
web3.eth.defaultAccount 
```
// 用于一些方法的默认from属性。 比如下面的一些方法
* web3.eth.sendTransaction()
* web3.eth.call()
* new web3.eth.Contract() -> myContract.methods.myMethod().call()
* new web3.eth.Contract() -> myContract.method.myMethod().send()

```
web3.eth.defaultBlock
```
the default value is 'latest' 
can be used in the following methods
```
eth.getBalance()
eth.getCode()
eth.getTransactionCount()
eth.getStorageAt()
eth.call();
```
the property can be `latest , pending , genesis`

