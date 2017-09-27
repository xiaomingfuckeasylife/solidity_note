#### web3.js库是一个针对EVM包含的函数定制化的一个javascript库
* web3-eth是针对以太坊区块链何智能合约的模块.
* web3-shh是用于连接p2p以及广播的whisper协议.
* web3-bzz用于去中心化文件存储的swarm协议.
* web3-utils对于去中心化app的开发者，包含一些有用的帮助项.

#### 添加web3.js
* npm : npm install web3 
* meteor : meteor add ethereum:web3
* pure js: link the dist/web3.min.js

完成以后，你需要创建一个web3的实例，并且设置一个rpc的服务提供者。如果发生连接错误。我们可以通过打开本地节点的rpc连接。`admin.startRPC("127.0.0.1", 8545, "*", "web3,net,eth")`
```
var web3 = new Web3(Web3.givenProvider || "ws://localhost:8545")
```
这样
