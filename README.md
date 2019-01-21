# Luckywin Node

[TOC]

## 一、LUCKYWIN主链资料

### 1.1 主网节点

http://mainnet.luckywin.io

### 1.2 主网资料

```
共识机制： IBFT
出块速度：约5S一个块
确定性共识，一般情况下一个确认就可以了。
LUCKY精度：最小单位为wei, 1LUCKY=10^18wei
交易模型： 同ETH的account

私钥管理方式：
keystore同以太坊

交易类型（交易是否只能一对一交易，或者可以多对多交易，是否支持多重签名）
基础交易只支持一对一，多对多需要通过合约实现，不支持多重签名

失败的交易是否会上链？手续费被全部扣掉？
失败的交易会上链，无手续费

有无分红机制：无
```

### 1.3 节点部署

Luckywin节点部署步骤

1. 准备工作
    准备服务器，服务器要求环境如下：

```
操作系统：CentOS7
内存：8GB以上
存储空间：200GB以上
网络带宽：5MB以上
```
2. 下载可执行文件
	获取Luckywin的节点可执行程序，创世配置文件genesis.json，许可节点配置文件static-nodes.json，存放在~/download目录下（该目录为用户下载目录，下同）
3. 配置工作目录

```
mkdir Luckywin	// 在合适的目录下创建Luckywin目录
cd Luckywin		// 进入Luckywin目录
cp ~/download/luckywin . 		// 拷贝下载的Luckywin节点可执行程序到Luckywin目录
cp ~/download/genesis.json .	// 拷贝下载的genesis.json到Luckywin目录

mkdir -p qdata/node/{keystore,geth}
cp ~/download/static-nodes.json qdata/node/
geth --datadir qdata/node init genesis.json
```
4. 配置启动脚本
启动节点脚本 startnode.sh

```
#!/bin/bash
set -u
set -e
NETWORK_ID=$(cat genesis.json | grep chainId | awk -F " " '{print $2}' | awk -F "," '{print $1}')

mkdir -p qdata/logs

set -v
ARGS="--nodiscover  --networkid $NETWORK_ID --syncmode full --rpc --rpcaddr 0.0.0.0 --rpcapi eth,net,txpool,web3"

nohup geth --datadir qdata/node $ARGS --rpcport 8545 2>>qdata/logs/1.log &
set +v
echo
echo "See 'node/logs' for logs, and run e.g. 'geth attach qdata/node/geth.ipc' to attach to node."

脚本文件赋予执行权限
chmod +x startnode.sh
```
4. 启动节点
./startnode

5. 检查节点工作状态
通过命令连接节点：geth attach qdata/node/geth.ipc
检查节点是否工作正常

## 二、RPC API

### 查询网络ID net_version

返回当前连接网络的ID。

1. 参数

无

2. 返回值

`String` - 网络的chainID

3. 示例

请求：

```
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"net_version","params":[],"id":1}'
```

响应：

```
{"jsonrpc":"2.0","id":1,"result":"2018"}
```

### 同步状态 eth_syncing

对于已经同步的客户端，该调用返回一个描述同步状态的对象；对于未同步客户端，返回false。

1. 参数

无

2. 返回值

`Object|Boolean`, 同步状态对象或false。同步对象的结构如下：

- startingBlock: - 开始块
- currentBlock: - 当前块，同eth_blockNumber
- highestBlock:  - 预估最高块

3. 示例

```
请求
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'

未同步响应：
{"jsonrpc":"2.0","id":2,"result":false}

同步中响应：
{"jsonrpc":"2.0","id":2,"result":{"currentBlock":"0xf536c","highestBlock":"0x112cad","knownStates":"0x0","pulledStates":"0x0","startingBlock":"0xf4240"}}
```

### eth_gasPrice

返回当前的gas价格，单位：wei。

1. 参数

无

2. 返回值

`QUANTITY` - 整数，以wei为单位的当前gas价格, 恒为0

3. 示例

```
请求：
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":2}' 
响应：
{"jsonrpc":"2.0","id":2,"result":"0x0"}
```

### 查询余额 eth_getBalance

返回指定地址账户的余额。

1、参数

- `DATA` - 20字节，要检查余额的地址
- `QUANTITY|TAG` - 整数块编号，或者字符串"latest", "earliest" 或 "pending"

2. 返回值

   `QUANTITY` - 当前余额，单位：wei

3. 示例

```
请求：
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0xca843569e3427144cead5e4d5999a3d0ccf92b8e", "latest"],"id":1}' 
响应：
{"jsonrpc":"2.0","id":1,"result":"0x2000000000000000"}
```

### 查询交易数量eth_getTransactionCount

返回指定地址发生的交易数量。

1、参数

- `DATA` - 20字节，要检查余额的地址
- `QUANTITY|TAG` - 整数块编号，或者字符串"latest", "earliest" 或 "pending"

1. 返回值

   `QUANTITY` - 从指定地址发出的交易数量，整数

2. 示例

```
请求：
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionCount","params":["0xca843569e3427144cead5e4d5999a3d0ccf92b8e","latest"],"id":1}'
响应：
{"jsonrpc":"2.0","id":1,"result":"0x2"}
```

### 构造交易

1. 查询交易数量 eth_getTransactionCount

2. 构造原始交易

   ```
   let rawTx = {
     nonce: '0x30', // Replace by nonce for your account on geth node
     gasPrice: '0x0',  // GasPrice值为0，没有手续费
     gasLimit: '0x30000',
     to: '0x77beb894fc9b0ed41231e51f128a347043960a9d', 
     value: '0x100000000',
     data: 0x0,
     chainId: 2018   // 固定chainID
   };
   ```

3. 签名交易数据，同以太坊
4. 用eth_sendRawTransaction发送数据

### 发送交易eth_sendRawTransaction

为签名数据创建一个新的交易。

1. 参数

   `DATA` - 签名的交易数据

2. 返回值

   ``DATA` - 32字节，交易哈希，如果交易未生效则返回全0哈希。

   当创建合约时，在交易生效后，使用`eth_getTransactionReceipt`获取合约地址。

3. 示例

   ```
   请求：
   curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_sendRawTransaction","params":[{see above}],"id":1}'
   响应:
   {
     "id":1,
     "jsonrpc": "2.0",
     "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
   }
   ```

### 查询交易 eth_getTransactionByHash

返回指定哈希对应的交易。

1. 参数

   `DATA`, 32 字节 - 交易哈希

2. 返回值

   `Object` - 交易对象，如果没有找到匹配的交易则返回null。结构如下：

   - hash: DATA, 32字节 - 交易哈希
   - nonce: - 本次交易之前发送方已经生成的交易数量
   - blockHash: DATA, 32字节 - 交易所在块的哈希
   - blockNumber: QUANTITY - 交易所在块的编号
   - transactionIndex: QUANTITY - 交易在块中的索引位置
   - from: DATA, 20字节 - 交易发送方地址
   - to: DATA, 20字节 - 交易接收方地址，对于合约创建交易，该值为null
   - value: QUANTITY - 发送的以太数量，单位：wei
   - gasPrice: QUANTITY - 发送方提供的gas价格，单位：wei
   - gas: QUANTITY - 发送方提供的gas可用量
   - input: DATA - 交易发送的数据

3. 示例：

   ```
   请求：
   curl -H "Content-Type:application/json"  -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByHash","params":["0x3a1de5a4cbc49f846e206be6b34966f048756bd5a417c1b2dee71b2bebb6681e"],"id":1}'
   响应：
   {
       "jsonrpc":"2.0",
       "id":1,
       "result":{
           "blockHash":"0x87014f25428e1922a3f4fb36750279abd6c759508220dc48dea8003c24589a95",
           "blockNumber":"0x113666",
           "from":"0xca843569e3427144cead5e4d5999a3d0ccf92b8e",
           "gas":"0x15f90",
           "gasPrice":"0x0",
           "hash":"0x3a1de5a4cbc49f846e206be6b34966f048756bd5a417c1b2dee71b2bebb6681e",
           "input":"0x",
           "nonce":"0x2",
           "to":"0xca843569e3427144cead5e4d5999a3d0ccf92b8e",
           "transactionIndex":"0x0",
           "value":"0x0",
           "v":"0xfe7",
           "r":"0x14756a9ba008146e4291a619a15026a078323d9a75421df8ef123b50d0c115ac",
           "s":"0x1438eb70e7fd39b69208badc8ba712064808ecea23e95d6c9f64b8c6af9e5a64"
       }
   }
   ```


### 交易收据eth_getTransactionReceipt

返回指定交易的收据

1. 参数

   `DATA`, 32字节 - 交易哈希

2. 返回值

   `Object` - 交易收据对象，如果收据不存在则为null。交易对象的结构如下：

   - transactionHash: DATA, 32字节 - 交易哈希
   - transactionIndex: QUANTITY - 交易在块内的索引序号
   - blockHash: DATA, 32字节 - 交易所在块的哈希
   - blockNumber: QUANTITY - 交易所在块的编号
   - from: DATA, 20字节 - 交易发送方地址
   - to: DATA, 20字节 - 交易接收方地址，对于合约创建交易该值为null
   - cumulativeGasUsed: QUANTITY - 交易所在块消耗的gas总量
   - gasUsed: QUANTITY - 该次交易消耗的gas用量
   - contractAddress: DATA, 20字节 - 对于合约创建交易，该值为新创建的合约地址，否则为null
   - logs: Array - 本次交易生成的日志对象数组
   - logsBloom: DATA, 256字节 - bloom过滤器，轻客户端用来快速提取相关日志
   - status: QUANTITY ，1 (成功) 或 0 (失败)

3. 示例

   ```
   请求：
   curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionReceipt","params":["0x3a1de5a4cbc49f846e206be6b34966f048756bd5a417c1b2dee71b2bebb6681e"],"id":1}'
   响应：
   {
       "jsonrpc":"2.0",
       "id":1,
       "result":{
           "blockHash":"0x87014f25428e1922a3f4fb36750279abd6c759508220dc48dea8003c24589a95",
           "blockNumber":"0x113666",
           "contractAddress":null,
           "cumulativeGasUsed":"0x5208",
           "from":"0xca843569e3427144cead5e4d5999a3d0ccf92b8e",
           "gasUsed":"0x5208",
           "logs":[
   
           ],
           "logsBloom":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
           "status":"0x1",
           "to":"0xca843569e3427144cead5e4d5999a3d0ccf92b8e",
           "transactionHash":"0x3a1de5a4cbc49f846e206be6b34966f048756bd5a417c1b2dee71b2bebb6681e",
           "transactionIndex":"0x0"
       }
   }
   ```

### 查询区块高度 eth_blockNumber

返回最新块的编号。

1. 参数

   无

2. 返回值

   `QUANTITY` - 节点当前块编号

3. 示例

   ```
   请求
   curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
   响应
   {"jsonrpc":"2.0","id":1,"result":"0x1133c6"}
   ```

## 三、查询区块信息 eth_getBlockByNumber

返回指定编号的块。

1. 参数

   - `QUANTITY|TAG` - 整数块编号，或字符串"earliest"、"latest" 或"pending"

2. 返回值

   `Object` - 匹配的块对象，如果未找到块则返回null，结构如下：

   - number:  - 块编号
   - hash: DATA, 32 Bytes - 块哈希
   - parentHash: DATA, 32 Bytes - 父块的哈希
   - nonce: DATA, 8 Bytes - 生成的pow哈希，
   - sha3Uncles: DATA, 32 Bytes - 块中叔伯数据的SHA3哈希
   - logsBloom: DATA, 256 Bytes 块日志的bloom过滤器，
   - transactionsRoot: DATA, 32 Bytes - 块中的交易树根节点
   - stateRoot: DATA, 32 Bytes - 块最终状态树的根节点
   - receiptsRoot: DATA, 32 Bytes - 块交易收据树的根节点
   - miner: DATA, 20 Bytes - 挖矿奖励的接收账户
   - difficulty: QUANTITY - 块难度，整数
   - totalDifficulty: QUANTITY - 截止到本块的链上总难度
   - extraData: DATA - 块额外数据
   - size: QUANTITY - 本块字节数
   - gasLimit: QUANTITY - 本块允许的最大gas用量
   - gasUsed: QUANTITY - 本块中所有交易使用的总gas用量
   - timestamp: QUANTITY - 块时间戳
   - transactions: Array - 交易对象数组，或32字节长的交易哈希数组
   - uncles: Array - 空

3. 示例

```
请求
curl -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x1", true],"id":1}' 
响应
{
    "jsonrpc":"2.0",
    "id":1,
    "result":{
        "difficulty":"0x1",
        "extraData":"0xd883010810846765746888676f312e31312e32856c696e757800000000000000f90164f854946571d97f340c8495b661a823f2c2145ca47d63c294d8dba507e85f116b1f7e231ca8525fc9008a696694e36cbeb565b061217930767886474e3cde903ac594f512a992f3fb749857d758ffda1330e590fa915eb841d8c2dc52e9409b51cf06ac607422ec806eb8bd69b471ad98d63cc75544f116732d73f9b9cd5a65ee23dd7df5c553a22d4f6af344b4c2e01278b70bd7bbdd69a601f8c9b8416d47583a7a3ff38c394a8da9cb16fd0cd7cc33356283b635a03cd38ccf4fa0192f8bfd9e2d516d01bbff6c8b516b7374fe385118a6c8956d17218555595befbb01b8411765e61c361159b022087708c7458560f2e068b47a411bafb32e3b3d931746a82f65d23c7d899aa1b7bec2271070477e5a73a79c722a3fa44487e5a01b025ad901b84132598cdeecebd49ccf7bc094d2d50efc0f17e772f92d4d8ccf00d666880d018d4db7999a40816bfa58b5f1b14e367f2b54b77665b63994bbf0188c9a9603fd8b01",
        "gasLimit":"0x7a1200",
        "gasUsed":"0x0",
        "hash":"0x357a793ce35a1d36a6d85be8df93bae275b7f0908d20124dc1712b57b3b3fa70",
        "logsBloom":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        "miner":"0x0000000000000000000000000000000000000000",
        "mixHash":"0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365",
        "nonce":"0x0000000000000000",
        "number":"0x1",
        "parentHash":"0x984b16bb3fd1f204e3bc125a6c9654d82ea6264f2797f3ffc67a557a857c78b3",
        "receiptsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
        "sha3Uncles":"0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
        "size":"0x385",
        "stateRoot":"0x07fe2c0d3dc368ed1b9d3677d4a07b790d677463d16c1b07cfee20ef58e79054",
        "timestamp":"0x5bf4cedb",
        "totalDifficulty":"0x2",
        "transactions":[

        ],
        "transactionsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
        "uncles":[

        ]
    }
}
```

### 根据hash查询区块信息 eth_getBlockByHash

返回具有指定哈希的块。

1. 参数

- `DATA`, 32字节 - 块哈希
- `Boolean` - 为true时返回完整的交易对象，否则仅返回交易哈希

2. 返回值

   同eth_getBlockByNumber
