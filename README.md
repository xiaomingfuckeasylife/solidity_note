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

#### 燃料
在创建的时候，每一个事物都被要求支付一定的燃料费用，目的是为了支付EVM的执行以及限制在执行事物的复杂度。当EVM在执行交易的过程中，燃料通过特定的规则逐渐消耗殆尽。
燃料价格可以由事物的创建者设定，并且必须先支付gas_price * gas的燃料费。如果燃料在最后没有用完，会返回到事物发生者的账户中。如果燃料在某个时刻用完了，一个out-of-gas异常会被触发，那么之前做的所有的动作都会回滚。

#### 存储，内存和栈
每一个账户都有一个持续的内存区域被称为`存储区`。存储区是一个Key-value对，存储的key和value都是32byte的。我们不可能吧合约中的存储区域穷尽。为了修改存储区域的内容，合约必须要能够读取或者写入他的所有的存储区域。

第二个部分被称之为`内存`，通过内存合约每次都能回去最新的调用信息。内存是线性存在的，并且能够在字节层面进行操作。但是读取不能超过32个字节，写入要么是1个字节，要么是32个字节。内存会扩张一个字节，当我们获取之前没有缓存的内容。在扩张的过程中，燃料的费用需要支付。随着内存的增大燃料费用也是指数上涨。

第三个部分，EVM不是一个注册机，相对而言是一个栈机器。所以所有的计算全部是在栈上进行的。他最大有1024个元素，并且包含32位的字符。可以将顶部的16个元素与后面的16个元素进行置换。当然我们也可以将`栈元素`移动到`存储区域`或者`内存`，但是我们不可能将任意一个元素进行随意的移动，并且不讲元素移动到栈的顶端。

#### 指令集
为了避免不正确的实现而导致的共识问题，EVM将指令集合设置到最小。所有的操作都是在最小的数据类型上面32个字节。常见的逻辑操作，bit操作，比较操作都是存在的，条件以及非条件跳跃也是可能的。更重要的，合约可以获取区块的一些相关的属性，例如区块号以及时间戳。

#### 消息调用
智能合约可以通过`消息调用`其他的智能合约或者发送Ether给其他非智能合约的账户。消息调用和事物很像。他们都包含源头，目标，数据input，ether，gas，以及返回数据，实际上，所有的事物都包含一个顶级的消息调用，并且基于他，进行递归的消息调用。

合约能够决定到底多少的gas需要发送在进行消息调用的时候。如果gas用完了，在内部调用的时候。那么他会提醒所有的并且放在栈上面。然后逐层的向上抛出。正如我们所说的被调用的智能合约会获取一个干净的内存区域，并且能够获取到输入参数。在某些地方被称为回调参数。在执行完毕后，返回值会被存储在调用这提前分配的内存区域。

递归消息调用的深度在1024以内，意味着对于复杂的操作，我们更倾向使用循环调用而不是递归调用。

#### 代理调用/调用代码以及代码库
存在很多类型的消息调用，代理调用其实和消息调用基本一样的。例如 ： msg.sender msg.value. 

这些功能意味着合约可以在运行时候动态的加载代码，并且运行。

以上的一些特性可以让我们在solidity中实现所谓`库`的概念， 可重用的库代码可以被运用在合约的存储区域，进而实现复杂的数据结构。

#### 日志
我们可以使用索引将数据存在区块上，日志被solidity在实现事件的时候会用到。合约在创建以后就没有权限看到日志了。

#### 创建
合约也可以创建合约用一些特殊的code。可以用管道的想法来理解这个过程。

#### 自我毁灭
只有通过selfdestruct操作将智能合约代码从区块链上删除掉。那个合约储存的ether。会被发送到目标目的地。然后存储空间以及代码会从区块链中删除掉。
警告：就算我们的智能合约中没有包含方法selfdestruct，他仍然可以执行delegateCall或者callCode方法来进行调用。

## Solidity by Excample 

### 投票
下面的智能合约相对来说会比较复杂，但是也展示了很多的Solidity的特性。他实现了一个投票的智能合约。当然，主要的问题如何将正确的选票投给正确的人。并且如何预防人工操作。我们不会在这个地方解决掉所有的问题。但是至少我们会展示一个优雅的透明且自动化的选票系统。

大致的想法是：为选票创建一个合约，提供每个选项的简称，然后合约的创建者会作为主席会对每个地址单独给予权利进行投票，拥有地址的那个人可以选择投给自己，或者将选票投递给自己信任的人

在选票结束后，winningProposal 会返回得到票数最多的人。

```go
pragma solidity ^0.4.11;
/// @title Voting with delegation.
contract Ballot {
    // 定义了一个结构体类型代表一票
    struct Voter {
       uint weight ; // 投票的权重，由那些下方投票权重的人累加
       bool voted  ; // 是否已经投票
       address delegate ; // 代表人地址
       uint vote ; //投票的选项的索引
    }
    
    // 定义了一个投票选择项的结构体
    struct Proposal {
        bytes32 name ;  // 名字最大只能有32个字节 也就是最大只能有32个因为单词
        uint voteCount; // 选项获得的选票
    }

    address public chairperson ; // 投票主席

    // 定义一个表存储所有地址对应的所有的选票
    mapping(address=>Voter) public voters ; 

    // 定义一个可变长的选项切片
    Proposal[] public proposals;

    // 为下面的这些选项创建一个选票
    function Ballot(bytes32[] proposalNames) {
    	chairperson = msg.sender;
    	voters[chairperson].weight = 1;
    	// 初始化选项切面
    	// `Proposal({...})` 创建一个临时选项。使用`proposals.push(...)`将选型放在选项数组的最后面
    	for (uint i=0;i<proposalNames.length;i++){
    		proposals.push(Proposal({
    			name:proposalNames[i],
    			voteCount:0
    		}));
    	}
    }

    function giveRightToVote(address voter){
    	// 如果require得出的结果是false 那么调用会停止然后退回所有到原始状态以及Ether的余额，
    	// 这个通常是一个很好使用这个函数来验证但是也要注意，这个函数目前也是需要消耗gas的但是
    	// 以后可能会取消gas的消耗。
    	require ((msg.sender != chairperson) && (!voters[voter].voted) && (voters[voter].weight == 0));
    	voters[voter].weight = 1;
    }

    function delegate(address to) {
    	// 声明一个变量 并且分配引用地址
    	Voter storage sender = voters[msg.sender];
    	require (!sender.voted);

    	// 不能自己让自己投票，形成死循环
    	require (to != msg.sender);

    	// 找到最终的投票代理，通常来说这样的循环是很危险的，因为如果循环的时间太久那么很可能造成gas消耗殆尽
    	// 这个地方会导致delegate方法执行不成功，但是在其他地方很可能导致智能合约完全的卡死。
    	while(voters[to].delegate != address(0)){
    		to = voters[to].delegate;

    		// 不能形成闭环
    		require (to != msg.sender);
    	}

    	sender.voted = true;
    	sender.delegate = to;
    	Voter storage delegate = voters[to];
    	// 如果代理已经投了的话，直接把权重加到投入的选项上。
    	if(delegate.voted){
    		proposals[delegate.vote].voteCount += sender.weight;
    	}else {
    		//如果没有投的话，把权重加到代理的身上
    		delegate.weight += sender.weight;
    	}
    }

    // 对选项进行投票
    function vote(uint proposal) {
    	Voter storage sender = voters[msg.sender];
    	require (!sender.voted);
    	sender.voted = true;
    	sender.vote  = proposal;
    	// 如果没有这个proposal那么会自动抛出一个异常，并且回滚已经发生的所有发生的改变
    	proposals[proposal].voteCount += sender.weight;
    }

    // 比较看那个选项的投票最多
    function winningProposal() constant returns(uint winningProposal){
    	uint winningVoteCount = 0;
    	for(uint i=0;i<proposals.length ;i++){
    		if (proposals[i].voteCount > winningVoteCount){
    			winningVoteCount = proposals[i].voteCount;
    			winningProposal = i;
    		}
    	}
    }

    // 通过调用winningProposal()方法获取获胜的选项下标。
    // 然后返回选项的名字
    function winnerName() constant returns (bytes32 winnerName) {
    	winnerName = proposals[winningProposal()].name;
    }
}
```
目前来说，我们所有的参与者都需要被分配权限才能够参与投票，你能想到一个很好的解决方案吗？

### 匿名拍卖

在这个部分，我们会向你们展示创建一个完全的匿名拍卖合约在EVM上面。我们首先会创建一个公开的拍卖每个人都可以看到当前的出价，然后继承那个合约形成一个完全匿名的拍卖我们可以在交易结束借钱看到真正的投标价格。

#### 简单的公开拍卖

简单的公开售卖合约是每个人都可以发送他们的投标价格在投标时段内，投标包含有发送金额或者ETH从而将他们的投标与投标人绑定。如果最高的投标价格出现了，那么
之前的最高的投资人会得到他之前投标的钱。在投标的结束阶段，合约必须手动调用让拍卖方获取到钱。合约不能自动激活。
```go
pragma solidity ^0.4.11;

contract SimpleAuction{
	// 拍卖合约的参数，时间要么是从1970年1月1日到现在的绝对时间或者时间段
	// 受益人
	address public beneficiary ; 
	// 合约开始时间
	uint public auctionStart;
	// 合约进行时间
	uint public biddingTime;

	// 当前最高的投标者
	address public highestBidder;
	// 最高的价格
	uint public highestBid;

	// 允许没有中标的投资者取回自己的投标金额
	mapping(address => uint) pendingReturns;

	// 投标是否结束
	bool ended;

	//定义投标价格增长事件 事件以大写字母开始
	event HighestBidIncreased(address bidder , uint amount);
	//定义投标结束事件  
	event AuctionEnded(address winner , uint amount);

	//构造器
	function SimpleAuction(uint _biddingTime,address _beneficiary){
		beneficiary = _beneficiary;
		auctionStart = now;
		biddingTime = _biddingTime;
	}

	// 投标 如果没有中标 金额可被赎回
	// 关键字payable必须的，让这个函数能够接受ether
	function bid() payable {

		// 不能超过投标时间，如果超过了投标时间取消所有的事物操作
		require(now <= (auctionStart + biddingTime));
		// 必须要求投标金额大于目前的最高金额，不然返回所有惭怍
		require(msg.value > highestBid);
		// 如果最高的投标者已经被初始化
		if (highestBidder != 0) {
			// 将原有的最高者加入到退回列表
			pendingReturns[highestBidder] += highestBid;
		}
		highestBidder = msg.sender;
		highestBid = msg.value;
		HighestBidIncreased(msg.sender,msg.value);
	}

	// 被推翻的金额可以手动退回
	function withdraw() returns (bool) {
		// 应退回的金额
		uint amount = pendingReturns[msg.sender];
		// 如果金额大于0
		if(amount > 0){
			// 清0处理
			pendingReturns[msg.sender] = 0;
			// 发送金额给自己
			if (!msg.sender.send(amount)){
				// 如果发送失败，重置原始的退回金额
				pendingReturns[msg.sender] = amount;
				return false;
			}
		}
		return true;
	}

	// 结束竞标 并且将酬劳给予受益者
	function auctionEnd(){
		// 下面的例子是一个很好的范例，给我们展示了如何与其他的合约进行沟通
		// 分为以下三个部分：
		// 1 . 状态检查。
		// 2 . 发生行为（通常改变状态）。
		// 3 . 和其他合约进行交互。
		// 如果这些顺序反了，其他的合约可能在回调当前的合约的时候造成状态的修改，或者多次支付ether等情况

		// 1 Conditions
		// 判断竞标时间是否已经结束
		require ( now >= (auctionStart + biddingTime));
		// 判断是否竞标已经结束 
		require (!ended); 

		// 2 
		// 执行动作 改变状态 开启事件
		ended = true;
		AuctionEnded(highestBidder,highestBid);

		// 3. Interaction  
		//发送ether给受益者
		beneficiary.transfer(highestBid);
	}
}
```

#### 匿名拍卖

```go
  // not now . 
```

####  安全远程购买
```go
    pragma solidity ^0.4.11;

// 用于购买的智能合约
contract Purchase{
	// 商品价值
	uint public value;
	// 卖方
	address public seller;
	// 买方
	address public buyer;
	// 购买状态枚举
	enum State {Created , Locked , Inactive}
	// 记录当前合约状态
	State public state;

	// 确保合约生成者的value是偶数
	function Purchase() payable {
		// 合约生成者为卖方
		seller = msg.sender;
		// 取合约的1/2价格作为购买的价值
		value  = msg.value/2;
		// 要求合约价值必须是偶数值
		require((2 * value) == msg.value);
	}

	// 修饰符 在程序执行前判断，减少gas的使用
	modifier condition(bool _condition) {
		require(_condition);
		_;
	}

	// 只允许买方
	modifier onlyBuyer(){
		require(msg.sender == buyer);
		_;
	}

	modifier onlySeller(){
		require(msg.sender == seller);
		_;
	}

	// 是否在我们需要的状态
	modifier inState(State _state){
		require(state == _state);
		_;
	}

	// 事件用于跟踪事物发生
	event Aborted();
	event PurchaseConfirmed();
	event ItemReceived();

	function abort() onlySeller inState(State.Created) {
		Aborted();
		// 终止后改变状态
		state = State.Inactive;
		// 将合约包含的ether转回给卖方
		seller.transfer(this.balance);
	}

	// 确定买方，买方的金额必须有两倍的 value 值得ether 。买完之后锁定金额
	function confirmPurcharse() inState(State.Created) condition(msg.value == (2 * value)) payable{
		PurchaseConfirmed();
		buyer = msg.sender ; 
		state = State.Locked;
	}

	// 买方确定收款项 这个会释放锁定的ether
	function confirmReceived() onlyBuyer inState(State.Locked){
		ItemReceived();
		// 改状态
		state = State.Inactive;
		// 给予买方价值
		buyer.transfer(value);
		// 转移卖方价值
		seller.transfer(this.balance);
	}
}
```

### 深入Solidity
这个章节会提供给你所有你应该知道的关于Solidity的一切。

#### solidity的文件结构
源文件能够包含任意数量的合约定义，包括指令和编译指令。

#### 编译版本
```go
pragma solidity ^0.4.0;
```
这种文件抬头不会被编译在0.4.0之前，也不会被编译在0.5.0以后，因为在0.5.0之后可能会有打的变动，可能使得我们的代码执行结果和我们的预期不太一样。

#### 引用其他的源文件

#### 语法以及语义
solidity支持import表达式，这个JavaScript的import很像。
在全局级别，你可以引用语句像这样。
```
import "filename";
```
这个语句引用了所有在filename中声明的全局变量，以及他的import文件，并且把它放到当前的全局变量域中。

```go 
import * as symbolName from  "filename";
import "filename" as symbolName;
```
以上的两种是一样的意思，都是创建一个全局变量symbolName然后他的变量都在文件filename中，

#### 文件路径 
在solidity中所有的路径默认是绝对路径，比如上面的"filename"其实是"/filename"，如果我们想要使用当前路径或者上一级目录，可以使用"." , ".." 
通常依靠编译器如何获取路径中的内容，通常情况下，目录的层级不需要严格的匹配到文件系统，他可以通过git，http等网络资源配置获取。
对于solc来说，这些文件的路径可以通过`context:prefix=target`进行参数匹配其中context和target都是不必须的，target默认就是prefix
例子
```
solc github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ source.sol

solc module1:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ \
     module2:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin_old/ \
     source.sol
```
#### comments
```go
// This is a single-line comment.

/*
This is a multi-line comment.
*/

/** @title Shape calculator. */
```

### 合约结构
在solidity中contract和在面向对象语言中的class很类似。每一个合约可以进行变量的声明，函数声明modifier声明，事件声明，结构体类型，枚举类型。更重要的是合约可以继承其他的合约。

#### 变量状态
状态变量会永久的存在合约存储空间中，
```
pragma solidity ^0.4.0;
contract SimpleStorage{
    uint storedData; // 状态变量
}
```
#### 函数
函数是合约中能够独立的执行单元
```
pragma solidity ^0.4.0;

contract SimpleAuction{
    function bid() payable {
        // ... 
    }
}
```
具体状态变量和函数的获取权限，我们在后面会考虑进去。

#### 函数修饰器
函数修饰器可以在声明阶段修改函数语义，具体内容我们在后面会说到
```
pragma solidity ^0.4.0;
contract Purchase{
    address public buyer;
    modifier onlyBuyer(){
        require(msg.sender == buyer);
	-;
    }
}
```

#### 事件
对于EVM日志特性，事件是一个很方便的接口。
```
pragma solidity ^0.4.0;
contract SimpleAuction{
    event HighestBidIncreased(address bidder , uint amount); // Event 
    
    function bid() payable{
        // ...
	HighestBidIncreased(msg.sender,msg.value); // Triggering event 
    }
}
```
#### 结构体
结构体和Go中基本一致，但是这里都是定义在contract中，定义实体的数据结构。
```
pragma solidity ^0.4.0;
contract Ballot {
    struct Voter{
    	uint weight;
	bool voted;
	address delegate;
	uint vote;
    }
}
```
#### 枚举类型
用法同JAVA
```
pragma solidity ^0.4.0;
contract Purchase {
    enum State { Created , Locked , Inactive}
}
```

#### 类型
solidity是一个静态类型语言，和go是一样的。也就是说所有的变量类型在编译阶段就已经决定了。solidity提供了一些类型可以组合成复杂的类型。

#### 值类型
之所以叫做值类型，是因为他们通过值传递。他们在传递的过程中，总是通过复制值得内容，然后传递。

#### Boolean
bool 可以存储true或者false两种其中之一。
操作符包含"!" "&&" "||" "==" "!="

#### Integers
int/uint "u"代表无符号。uint是uint256的别名。在go solidity中无符号的整形使用的较为普遍。因为大部分情况下我们都不需要负数。
uint8/uint256 8-256代表8到256个bits。 

#### Address
address : 保存着一个包含着20个字节的值，address类型也有成员作为所有合约的基准变量。
可以对address使用的操作符号包括，<= , < , == , != , >= , > . 

#### Address Member 
* balance / transfer 
我们可以使用使用balance方法获取地址中的余额。使用transfer来发送ether到地址
```
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
```
笔记： 如果x是一个合约地址，会有一个回调函数和transfer以及被调用，这个是被EVM约束的，如果这个在执行过程中没有了gas或者失败了，那么ether转移会被回滚，并且当前的合约会停止执行并且抛出异常。

* send
发送方法是一个底层的和transfer相反的操作，如果执行失败，那么合约不会停止，send会返回false。
警告：send有一些潜在的风险，如果栈的深度超过1024，或者接受者没有了gas，那么转账就会失败。所以为了安全的ether转账，总是检查send的返回值。或只使用transfer或者一些更好的方式。

* call / callcode / delegatecall 
call函数提供了一些参数的类型，这些参数会被分割为32个字节。除了当第一个参数是刚好4个字节的时候，这种情况函数的签名不会被分割。
```go
address nameReg = "0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2";
nameReg.call("register","MyName");
nameReg.call(byte4(keccak256("fun(uint256)")),a);
```
call会返回一个布尔值，标志是否调用成功。还是产生了一个EVM异常。我们不可能有权限获取实际的返回值。
我们可以通过.gas修饰器来修改gas的值。
```
nameReg.call.gas(1000000)("register","MyName");
```
相同的，提供的Ether值也是可以被设置的
```
nameReg.call.value(1 ether)("register","MyName");
```
笔记：目前来说我们不能使用gas或者value修饰器对重载的方法进行设值。

同样的delegateCal,callcodel也可以同样被使用，这三种函数都是很底层的函数应该最后不得不使用的时候才使用，因为他们打破了solidity的安全性问题。有点像java中的反射机制。
`.gas`可以被三种使用，`.value`选项不支持delegateCall。 callcode不建议使用在以后的代码中会被去掉
我们可以使用this.balance 获取合约的余额。

#### 固定大小的byte数组
bytes1 bytes2 .. bytes32. byte是bytes1的别名。 有length方法
我们也可以使用数组用byte[], 但是它很浪费空间，每一个实例会使用31个字节准确来说。最好使用bytes

#### 动态数组
bytes , string 

#### 函数类型
定义
```sol
function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)] 
```

#### 数据存储位置
每一个复制的类型不论是数组还是结构体都有一个附带的标签，也就是存储位置。关于他是存储在memory中还是存储在storage中，根据上下文，总是有默认的存储位置。但是也可以通过手动的添加`storage` `memory`来覆盖位置类型。 对于函数的参数默认的地址在memory，本地变量默认存储在storage中，对于状态变量是强制存储在storage区域中的。

除了上面的两个位置以外还有一个位置`calldata` ， 这是一个不持久的也不可改变的区域用于存储函数参数。 对于外部函数的函数参数以及返回参数，都强制放在calldata位置，和memory类似。

```
pragma solidity ^0.4.0;

contract C {
    uint[] x; // the data location of x is storage

    // the data location of memoryArray is memory
    function f(uint[] memoryArray) {
        x = memoryArray; // works, copies the whole array to storage
        var y = x; // works, assigns a pointer, data location of y is storage
        y[7]; // fine, returns the 8th element
        y.length = 2; // fine, modifies x through y
        delete x; // fine, clears the array, also modifies y
        // The following does not work; it would need to create a new temporary /
        // unnamed array in storage, but storage is "statically" allocated:
        // y = memoryArray;
        // This does not work either, since it would "reset" the pointer, but there
        // is no sensible location it could point to.
        // delete y;
        g(x); // calls g, handing over a reference to x
        h(x); // calls h and creates an independent, temporary copy in memory
    }

    function g(uint[] storage storageArray) internal {}
    function h(uint[] memoryArray) {}
}
```

##### Summary
Forced data location:
1. parameters (not return) of external functions: calldata
2. state variables: storage
Default data location:
1. parameters (also return) of functions: memory
2. all other local variables: storage

#### Arrays
bytes is almost the same as byte[]
bytes should always be preferred over byte[] because it is cheaper.  it is packed tightly in calldata. 

#### Allocation Memory Arrays.
```
pragma solidity ^0.4.0;
contract C{
    function f(uint len) {
        uint[] memory a = new uint[](7);
	bytes memory b = new bytes(len);
	// Here we have a.length == 7 and b.length == len 
    }
}
```

Array Literals / Inline Arrays  
```
pragma solidity ^0.4.0;
contract C{
    fuction f() {
    	g([uint,2,3]);
    }
    function g(uint[3] _data) {} // this is cool 
}
```

```
function f() {
    // The next line creates a type error becuase uint[3] memory cannot be converted to uint[] memory. 
    uint[] x = [uint(1),3,4];
}
```

