Conflux JSON-RPC API是一组接口的集合，允许您使用JSON-RPC协议中的HTTP连接与本地或远程的Conflux节点进行交互。

以下是带有示例的API参考文档。

## JSON-RPC
JSON是一种轻量级的数据交换格式。 它可以表示数字，字符串，有序数值队列，和键值对集合。

JSON-RPC是一种无状态、轻量级的远程过程调用（RPC）协议。该规范主要定义了几种数据结构以及与之相关的处理规则。它是与传输方式无关的，可以用在socket、HTTP 或各种消息传送环境。 它使用JSON（RFC 4627）作为数据格式。


## JavaScript API
我们还提供了一个JavaScript库[js-conflux-sdk](https://github.com/Conflux-Chain/js-conflux-sdk) ，供您从JavaScript应用程序内部与Conflux节点进行交互，为RPC提供了方便的接口。

## JSON-RPC 节点和支持
目前，Conflux拥有一个支持JSON-RPC 2.0和HTTP的[Rust 实现](https://github.com/Conflux-Chain/conflux-rust) 。

## 十六进制值的编码
当前，通过JSON传输，有两种关键数据类型：无格式字节数组和数量。 两者都以十六进制编码传输，但有不同的格式要求：

当编码**数量**（整数，数字）时：编码为十六进制，前缀为“ 0x”，这是最紧凑的表现形式（小例外：零表示为“ 0x0”）。 例子：

* 0x41 （十进制65）
* 0x400 （十进制1024）
* WRONG: 0x（至少有一位数字 - 零为“ 0x0”）
* WRONG: 0x0400（不允许以零开头）
* WRONG: ff（必须以0x为前缀）

当编码 **无格式数据**（字节数组，帐户地址，哈希值，字节码数组）时：十六进制编码，前缀为“ 0x”，每个字节包含两个十六进制数字。 例子：

* 0x41 (长度为1，“A”)
* 0x004200 (长度为3，“ \ 0B \ 0”)
* 0x (长度为0，“”)
* WRONG: 0xf0f0f (必须为偶数位数)
* WRONG: 004200 (必须以0x开头)

## 纪元号参数（Epoch Number）
以下方法拥有纪元号参数：

* cfx_call
* cfx_epochNumber
* cfx_estimateGasAndCollateral
* cfx_getAccumulateInterestRate
* cfx_getAdmin
* cfx_getBalance
* cfx_getBlockByEpochNumber
* cfx_getBlocksByEpoch
* cfx_getCode
* cfx_getCollateralForStorage
* cfx_getInterestRate
* cfx_getNextNonce
* cfx_getSponsorInfo
* cfx_getStakingBalance
* cfx_getStorageAt
* cfx_getStorageRoot

当请求Conflux树图区块链的状态时，参数Epoch Number决定了Epoch的高度。
以下是参数Epoch Number的可选项:

* `HEX String` - 整型的纪元号
* `String "earliest"` - 创始区块所在的最早纪元
* `String "latest_mined"` - 最新挖掘的区块所在的纪元
* `String "latest_state"` - 可执行状态的最新区块所在的纪元

注意，出于性能优化的考虑，最新的纪元是不可执行的，所以在这些纪元里没有可用的状态。对于大多数与状态查询有关的RPCs，建议使用`"latest_state"`。


## Curl 范例解释
Curl选项可能会返回一个关于content type报错的响应，这是因为 --data 选项会将content type设置为 application/x-www-form-urlencoded 。如果你的节点无法执行，手动设置header -H "Content-Type: application/json "。

范例也包含了URL/IP和端口的组合，它们必须作为最后的参数传给curl。
例如 ```http://localhost:12345```

例子： [cfx_getbestblockhash](#cfx_getbestblockhash)

```
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getBestBlockHash","params":[],"id":1}' -H "Content-Type: application/json" http://localhost:12345
```

## 从 Ethereum JSON-RPC 移植
以太坊和Conflux之间，一些JSON-RPCs存在对应关系。

尽管JSON-RPC的细节可能彼此不同，以下映射表有助于从以太坊迁移到Conflux：
* eth_gasPrice => cfx_gasPrice
* eth_blockNumber => cfx_epochNumber
* eth_getBalance => cfx_getBalance
* eth_getStorageAt => cfx_getStorageAt
* eth_getTransactionCount => cfx_getNextNonce
* eth_getCode => cfx_getCode
* eth_sendRawTransaction => cfx_sendRawTransaction
* eth_call => cfx_call
* eth_estimateGas => cfx_estimateGasAndCollateral
* eth_getBlockByHash => cfx_getBlockByHash
* eth_getBlockByNumber => cfx_getBlockByEpochNumber
* eth_getTransactionByHash => cfx_getTransactionByHash
* eth_getTransactionReceipt => cfx_getTransactionReceipt
* eth_getLogs => cfx_getLogs

## JSON-RPC 方法
#### cfx_getTransactionByHash 
通过交易哈希值，返回交易的信息。
##### 参数
 1. DATA, 32 Bytes - 交易的哈希值
```
params: [
    '0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b',
]
```
##### 返回值
`Object` - 交易对象，如果未找到任何交易，则为`null`：
* `blockHash`: `DATA`, 32字节-此交易存在并已执行的区块的哈希。交易等待状态时为`null`。
* `contractCreated`: `DATA`，20字节-已创建的合约地址。当为不创建交易的合约时为`null`。
* `data`: `DATA` - 与交易一起发送的数据。
* `from`: `DATA`, 20字节-发送方地址。
* `gas`: `QUANTITY` - 发送者提供的gas。
* `gasPrice`: `QUANTITY` - 发送方提供的gas价格（以Drip为单位）。
* `hash`: `DATA`, 32字节-交易的哈希。
* `nonce`: `QUANTITY` - 发送方在此之前进行的交易次数。
* `r`: `DATA`, 32字节-ECDSA签名r。
* `s`: `DATA`, 32字节-ECDSA签名s。
* `status`: `QUANTITY` - 0代表成功，1代表发生错误，当交易被跳过或未打包时为`null` 。
* `to`: `DATA`, 20字节-接收者的地址。当其为合约创建的交易时为`null`。
* `transactionIndex`: `QUANTITY` - 区块中交易的索引位置的整数。交易等待状态时为`null`。
* `v`: `QUANTITY` - ECDSA恢复ID。
* `value`: `QUANTITY` - 传输的值（以Drip为单位）。

##### 使用范例
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getTransactionByHash","params":["0x53fe995edeec7d241791ff32635244e94ecfd722c9fe90f34ddf59082d814514"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": {
    "blockHash": "0xbb1eea3c8a574dc19f7d8311a2096e23a39f12e649a20766544f2df67aac0bed",
    "contractCreated": null,
    "data": "0x",
    "from": "0xb2988210c05a43ebd76575f5421ef84b120ebf80",
    "gas": "0x5208",
    "gasPrice": "0x174876e800",
    "hash": "0x53fe995edeec7d241791ff32635244e94ecfd722c9fe90f34ddf59082d814514",
    "nonce": "0x1",
    "r": "0x27e5cb110dd198b8fc963d4741ec0840400a6351d9e0c458eeda6af5e0a12760",
    "s": "0x2c486d8e26da3c867fbcf4ab242af1265a5036c5e23ea42c8ab23437ce4c0bca",
    "status": "0x0",
    "to": "0xb2988210c05a43ebd76575f5421ef84b120ebf80",
    "transactionIndex": "0x0",
    "v": "0x1",
    "value": "0x3635c9adc5dea00000"
  },
  "id": 1
}
```
---

#### cfx_getBlockByHash 
按哈希值返回区块信息。
##### 参数
  1. `DATA`, 32 字节 - 区块的哈希值。
  2. `Boolean` - 如果为 `true` ，则返回完整的交易对象；如果为`false`，则仅返回交易的哈希值。
```
params: [
    '0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331',
    true
]
```
##### 返回值
`Object` - 区块对象，如果未找到任何区块，则为`null`:
* `adaptive`: `Boolean` - 如果为`true` ，说明区块的权重符合GHAST规则，否则为`false`。
* `blame`: `QUANTITY` - 如果为0，说明在其引用路径上没有blame区块；如果大于0，则说明在其引用路径上最近的blame区块距离为`blame`。
* `deferredLogsBloomHash`: `DATA`, 32字节-延迟日志的Bloom哈希。
* `deferredReceiptsRoot`: `DATA`, 32字节-延迟执行后的区块的接收的哈希值。
* `deferredStateRoot`: `DATA`, 32字节-延迟执行后，区块的最终状态排列的根。
* `difficulty`: `QUANTITY` - 此区块的难度的整数。
* `epochNumber`: `QUANTITY` - 用户视图中的当前区块纪元号。如果它不处在最佳区块集合或未确定纪元号，则为`null` 。
* `gasLimit`: `QUANTITY` - 此区块中允许的最大gas值。
* `hash`: `DATA`, 32 Bytes - 区块链的哈希值。当状态为等待的时为`null`。
* `height`: `QUANTITY` - 区块的高度。当状态为等待的时为`null`。
* `miner`: `DATA`, 20 Bytes - 获得采矿奖励的受益人地址。
* `nonce`: `DATA`, 8 Bytes - 生成的工作量证明的哈希。当状态为等待的时为`null`。 
* `parentHash`: `DATA`, 32 Bytes - 父区块的哈希。
* `powQuality`: , `DATA`, Bytes - 生成的工作量证明的哈希。当状态为等待的时为`null`。
* `refereeHashes`: `Array` - referee哈希数组。
* `size`: `QUANTITY` - 此区块的大小（以字节为单位）。
* `timestamp`: `QUANTITY` - 该区块被收录时的UNIX时间戳。
* `transactions`: `Array` - 交易对象的数组，或32字节的交易哈希，具体取决于最后给定的参数。
* `transactionsRoot`: `DATA`, 32 Bytes - 区块交易的哈希。

##### 使用范例
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getBlockByHash","params":["0x692373025c7315fa18b2d02139d08e987cd7016025920f59ada4969c24e44e06", false],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": {
    "adaptive": false,
    "blame": 0,
    "deferredLogsBloomHash": "0xd397b3b043d87fcd6fad1291ff0bfd16401c274896d8c63a923727f077b8e0b5",
    "deferredReceiptsRoot": "0x522717233b96e0a03d85f02f8127aa0e23ef2e0865c95bb7ac577ee3754875e4",
    "deferredStateRoot": "0xd449df4ba49f5ab02abf261e976197beecf93c5198a6f0b6bd2713d84115c4ec",
    "difficulty": "0xeee440",
    "epochNumber": "0x1394cb",
    "gasLimit": "0xb2d05e00",
    "hash": "0x692373025c7315fa18b2d02139d08e987cd7016025920f59ada4969c24e44e06",
    "height": "0x1394c9",
    "miner": "0x000000000000000000000000000000000000000b",
    "nonce": "0x329243b1063c6773",
    "parentHash": "0xd1c2ff79834f86eb4bc98e0e526de475144a13719afba6385cf62a4023c02ae3",
    "powQuality": "0x2ab0c3513",
    "refereeHashes": [
      "0xcc103077ede14825a5667bddad79482d7bbf1f1a658fed6894fa0e9287fc6be1"
    ],
    "size": "0x180",
    "timestamp": "0x5e8d32a1",
    "transactions": [
      "0xedfa5b9c38ba51e791cc72b8f75ff758533c8c38f426eddee3fd95d984dd59ff"
    ],
    "transactionsRoot": "0xfb245dae4539ea49812e822adbffa9dd2ee9b3de8f3d9a7d186d351dcc9a6ed4"
  },
  "id": 1
}
```

---

#### cfx_getBlockByEpochNumber
按纪元号返回区块的信息。
##### Parameters
  1. `QUANTITY|TAG` - 纪元号, 或者为字符串 "latest_mined",  "latest_state" 或 "earliest", 详见 [epoch number parameter](#the-epoch-number-parameter).
  2. `Boolean` - 如果为`true`，则返回完整的交易对象；如果为`false`，则仅返回交易的哈希值。
```
params: [
    'latest_mined',
    true
]
```
##### 返回值
见 [cfx_getBlockByHash](#cfx_getblockbyhash).

##### 使用范例
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getBlockByEpochNumber","params":["latest_mined", false],"id":1}'
```
结果 见 [cfx_getBlockByHash](#cfx_getblockbyhash).

---

#### cfx_getBestBlockHash
返回最佳区块的哈希值。
##### 参数
`none`
##### 返回值
`DATA` , 32 Bytes - 最佳区块的哈希值。
##### 使用范例
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getBestBlockHash","params":[],"id":1}'

// Result
{
    "result" : "0x7d54c03f4fe971d5c45d95dddc770a0ec8d5bd27d57c049ce8adc469269e35a4",
    "id" : 1,
    "jsonrpc" : "2.0"
}
```

---


#### cfx_epochNumber
返回给定参数的纪元号。
##### 参数
`TAG` - (可选, 默认值: "latest_mined") String "latest_mined",  "latest_state" 或 "earliest", 详见 [epoch number parameter](#the-epoch-number-parameter).
##### 返回值
`QUANTITY` - 给定参数的当前纪元号。
##### 使用范例
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_epochNumber","params":[],"id":1}'

// Result
{
    "jsonrpc" : "2.0",
    "id" : 1,
    "result" : "0x49"
}
```

---

#### cfx_gasPrice
返回当前的gas价格（以Drip为单位）。
##### 参数
`none`
##### 返回值
`QUANTITY` - 当前的gas价格（以Drip为单位）.
##### 使用范例
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_gasPrice","params":[],"id":1}'

// Result
{
    "jsonrpc" : "2.0",
    "id" : 1,
    "result" : "0x09184e72a000"
} 
```

---

#### cfx_getBlocksByEpoch
返回位于某个纪元的区块的哈希值。
##### 参数
`QUANTITY|TAG` - 纪元号, 或者字符串 "latest_mined",  "latest_state" 或 "earliest", 详见 [epoch number parameter](#the-epoch-number-parameter).
##### 返回值
`Array` - 区块哈希数组，按执行（拓扑）顺序排序。
##### 使用范例
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getBlocksByEpoch","params":["0x11"],"id":1}'

// Result
{
    "jsonrpc": "2.0",
    "result": [
        "0x618e813ed93f1020bab13a1ab77e1550da6c89d9c69de837033512e91ac46bd0",
        "0x0f6ac81dcbc612e72e0019681bcec32254a34bd29a6bbab91e5e8dc37ecb64d5",
        "0xad3238c00456adfbf847d251b004c1e306fe637227bb1b9917d77bd5b207af68",
        "0x0f92c2e796be7b016d8b74c6c270fb1851e47fabaca3e464d407544286d6cd34",
        "0x5bcc2b8d2493797fcadf7b80228ef5b713eb9ff65f7cdd86562db629d0caf721",
        "0x7fcdc6fff506b19a2bd72cd3430310915f19a59b046759bb790ba4eeb95e9956",
        "0xf4f33ed08e1c625f4dde608eeb92991d77fff26122bab28a6b3a2037511dcc83",
        "0xa3762adc7f066d5cb62c683c2655be3bc3405ff1397f77d2e1dbeff2d8522e00",
        "0xba7588476a5ec7e0ade00f060180cadb7430fd1be48940414baac48c0d39556d",
        "0xe4dc4541d07118b598b2ec67bbdaa219eb1d649471fe7b5667a0001d83b1e9b6",
        "0x93a15564544c57d6cb68dbdf60133b318a94439e1f0a9ccb331b0f5a0aaf8049"
    ],
    "id": 1
}
```
---

#### cfx_getBalance
返回给定地址的帐户余额。
##### 参数
1. `DATA`, 20 Bytes - 待查询余额的地址。
2. `QUANTITY|TAG` - 纪元号, 或字符串 "latest_mined",  "latest_state", "earliest", 详见 [epoch number parameter](#the-epoch-number-parameter)
```
params: [
   '0xc94770007dda54cF92009BFF0dE90c06F603a09f',
   'latest_state'
]
```
##### 返回值
`QUANTITY` - 当前地址帐户的余额（以Drip为单位）。
##### 使用范例
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getBalance","params":["0xc94770007dda54cF92009BFF0dE90c06F603a09f", "latest_state"],"id":1}'

// Result
{
    "id":1,
    "jsonrpc": "2.0",
    "result": "0x0234c8a3397aab58" // 158972490234375000
}
```

---

#### cfx_getStakingBalance
返回给定地址的抵押帐户的余额。
##### 参数
1. `DATA`, 20 Bytes - 待检查抵押余额的地址。
2. `QUANTITY|TAG` - 纪元号, 或字符串 "latest_mined",  "latest_state", "earliest", 详见 [epoch number parameter](#the-epoch-number-parameter)
```
params: [
   '0xc94770007dda54cF92009BFF0dE90c06F603a09f',
   'latest_state'
]
```
##### 返回值
`QUANTITY` - 当前抵押账户的余额（以Drip为单位）。
##### 使用范例
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getStakingBalance","params":["0xc94770007dda54cF92009BFF0dE90c06F603a09f", "latest_state"],"id":1}'

// Result
{
    "id":1,
    "jsonrpc": "2.0",
    "result": "0x0234c8a3397aab58" // 158972490234375000
}
```

---


#### cfx_getCollateralForStorage
Returns the size of the collateral storage of given address, in Byte.
##### Parameters
1. `DATA`, 20 Bytes - address to check for collateral storage.
2. `QUANTITY|TAG` - integer epoch number, or the string "latest_mined",  "latest_state", "earliest", see the [epoch number parameter](#the-epoch-number-parameter)
```
params: [
   '0xc94770007dda54cF92009BFF0dE90c06F603a09f',
   'latest_state'
]
```
##### Returns
`QUANTITY` - integer of the collateral storage in Byte.
##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getCollateralForStorage","params":["0xc94770007dda54cF92009BFF0dE90c06F603a09f", "latest_state"],"id":1}'

// Result
{
    "id":1,
    "jsonrpc": "2.0",
    "result": "0x0234c8a8"
}
```

---

#### cfx_getAdmin
Returns the admin of given contract.
##### Parameters
1. `DATA`, 20 Bytes - address to contract.
2. `QUANTITY|TAG` - integer epoch number, or the string "latest_mined",  "latest_state", "earliest", see the [epoch number parameter](#the-epoch-number-parameter)
```
params: [
    '0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f',
    'latest_state'
]
```
##### Returns
`DATA` - 20 Bytes - address to admin, or `0x0000000000000000000000000000000000000000` if the contract does not exist.
##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getAdmin","params":["0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": "0x144aa8f554d2ffbc81e0aa0f533f76f5220db09c",
  "id": 1
}
```

---

#### cfx_getCode
Returns the code of given contract.
##### Parameters
1. `DATA`, 20 Bytes - address to contract.
2. `QUANTITY|TAG` - integer epoch number, or the string "latest_mined",  "latest_state", "earliest", see the [epoch number parameter](#the-epoch-number-parameter)
```
params: [
    '0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f',
    'latest_state'
]
```
##### Returns
`DATA` - byte code of contract, or `0x` if the contract does not exist.
##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getCode","params":["0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f","latest_state"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": "0x",
  "id": 1
}
```

---

#### cfx_getStorageAt
Returns storage entries from a given contract.
##### Parameters
1. `DATA`, 20 Bytes - address to contract.
2. `DATA`, 32 Bytes - the given position.
3. `QUANTITY|TAG` - integer epoch number, or the string "latest_mined",  "latest_state", "earliest", see the [epoch number parameter](#the-epoch-number-parameter)
```
params: [
    '0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f',
    '0x6661e9d6d8b923d5bbaab1b96e1dd51ff6ea2a93520fdc9eb75d059238b8c5e9',
    'latest_state'
]
```
##### Returns
`DATA` - 32 Bytes - storage entry of given query, or `null` if the it does not exist.
##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getStorageAt","params":["0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f","0x6661e9d6d8b923d5bbaab1b96e1dd51ff6ea2a93520fdc9eb75d059238b8c5e9","latest_state"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": "0x000000000000000000000000000000000000000000000000000000000000162e",
  "id": 1
}
```

---

#### cfx_getStorageRoot

Returns the storage root of a given contract.

##### Parameters
1. `DATA`, 20 Bytes - address to contract.
2. `QUANTITY|TAG` - integer epoch number, or the string "latest_mined", "latest_state", "earliest", see the [epoch number parameter](#the-epoch-number-parameter)
```
params: [
    '0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f',
    'latest_state'
]
```

##### Returns

`Object` - A storage root object, or `null` if the contract does not exist:

* `delta`: `DATA`, 32 Bytes - storage root in the delta trie.
* `intermediate`: `DATA`, 32 Bytes - storage root in the intermediate trie.
* `snapshot`: `DATA`, 32 Bytes - storage root in the snapshot.

If all three of these fields match for two invocations of this RPC, the contract's storage is guaranteed to be identical.
If they do not match, storage has likely changed (or the system transitioned into a new era).

<!---
TODO: Add links to snapshot/checkpoint documentation.
-->

##### Example

```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getStorageRoot","params":["0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f","latest_state"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": {
    "delta": "0x0240a5a3486ac1cee71db22b8e12f1bb6ac9f207ecd81b06031c407663c20a94",
    "intermediate": "0x314a41f277b678a1dc811a1fc0393b6d30c35e900cb27762ec9e9042bfdbdd49",
    "snapshot": "0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470"
  },
  "id" :1
}
```

---

#### cfx_getSponsorInfo
Returns the sponsor info of given contract.
##### Parameters
1. `DATA`, 20 Bytes - address to contract.
2. `QUANTITY|TAG` - integer epoch number, or the string "latest_mined",  "latest_state", "earliest", see the [epoch number parameter](#the-epoch-number-parameter)
```
params: [
    '0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f',
    'latest_state'
]
```
##### Returns
`Object` - A sponsor info object, if the contract doesn't have a sponsor, then the all fields in returned object will be `0`:

   * `sponsorBalanceForCollateral`: `QUANTITY` - the sponsored balance for storage.
   * `sponsorBalanceForGas`: `QUANTITY` - the sponsored balance for gas.
   * `sponsorGasBound`: `QUANTITY` - the max gas could be sponsored for one transaction.
   * `sponsorForCollateral`: `DATA`, 20 Bytes - the address of the storage sponsor.
   * `sponsorForGas`: `DATA`, 20 Bytes - the address of the gas sponsor.

##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getSponsorInfo","params":["0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": {
    "sponsorBalanceForCollateral": "0x0",
    "sponsorBalanceForGas": "0x0",
    "sponsorForCollateral": "0x0000000000000000000000000000000000000000",
    "sponsorForGas": "0x0000000000000000000000000000000000000000",
    "sponsorGasBound": "0x0"
  },
  "id": 1
}
```

---
#### cfx_getNextNonce
Returns the next nonce should be used by given address.

##### Parameters
1. `DATA`, 20 Bytes - address.
2. `QUANTITY|TAG` - integer epoch number, or the string "latest_mined",  "latest_state", "earliest", see the [epoch number parameter](#the-epoch-number-parameter)
```
params: [
    '0xfbe45681ac6c53d5a40475f7526bac1fe7590fb8',
    'latest_state' // state at the latest executed epoch
]
```
##### Returns
`QUANTITY` - integer of the next nonce should be used by given address.

##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getNextNonce","params":["0xfbe45681ac6c53d5a40475f7526bac1fe7590fb8", 'latest_state'],"id":1}'

// Result
{
    "jsonrpc" : "2.0",
    "result" : "0x1",
    "id" : 1
}
```

---
#### cfx_sendRawTransaction
Creates new message call transaction or a contract creation for signed transactions.
##### Parameters
`DATA`, The signed transaction data.
```
params: [
    '0xf86eea8201a28207d0830f4240943838197c0c88d0d5b13b67e1bfdbdc132d4842e389056bc75e2d631000008080a017b8b26f473820475edc49bd153660e56b973b5985bbdb2828fceacb4c91f389a03452f9a69da34ef35acc9c554d7b1d63e9041141674b42c3abb1b57b9f83a2d3'
]
```
##### Returns
`DATA`, 32 Bytes - the transaction hash, or the zero hash if the transaction is not yet available.
##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_sendRawTransaction","params":[{see above}],"id":1}'

// Result
{
    "id":1,
    "jsonrpc": "2.0",
    "result": "0xf5338a6cb85d10acc9108869f94fe322b2dfa2715d16d264676c91f6a0404b61"
}
```

---

#### cfx_call
Virtually call a contract, return the output data.

##### Parameters
1. `Object` - A call request object:

      * `from`: `DATA`, 20 Bytes - (optional, default: random address) address of sender.
* `to`: `DATA`, 20 Bytes - (optional, default: `null` for contract creation) address of receiver.
* `gasPrice`: `QUANTITY` - (optional, default: `0`) gas price provided by the sender in Drip.
* `gas`: `QUANTITY` - (optional, default: `500000000`) gas provided by the sender.
* `value`: `QUANTITY` - (optional, default: `0`) value transferred in Drip.
* `data`: `DATA` - (optional, default: `0x`) the data send along with the transaction.
* `nonce`: `QUANTITY` - (optional, default: `0`) the number of transactions made by the sender prior to this one.

2. `QUANTITY|TAG` - integer epoch number, or the string "latest_mined",  "latest_state", "earliest", see the [epoch number parameter](#the-epoch-number-parameter)

```
params: [
    {
        "from":"0xf6B7219AF657e14B5103e915839dD12f51cDBA68",
        "to":"0x63428378C5D7d168c9Ef2809a76812d40E018Ac9",
        "data":"0x",
        "gasPrice":"0x2540be400",
        "nonce": "0x0"
    },
    'latest_state' // state at the latest executed epoch
]
```
##### Returns
`DATA`, Bytes - the output data.
##### Example
```
// Request
curl -X POST --data '{"method":"cfx_call","id":1,"jsonrpc":"2.0","params":[{"from":"0xf6B7219AF657e14B5103e915839dD12f51cDBA68","to":"0x63428378C5D7d168c9Ef2809a76812d40E018Ac9","data":"0x","gasPrice":"0x2540be400", "nonce": "0x0"}]}'

// Result
{
  "jsonrpc": "2.0",
  "result": "0x",
  "id": 1
}
```

---

#### cfx_estimateGasAndCollateral
Virtually call a contract, return the output data.

##### Parameters
See [cfx_call](#cfx_call).

```
params: [
    {
        "from":"0xf6B7219AF657e14B5103e915839dD12f51cDBA68",
        "to":"0x63428378C5D7d168c9Ef2809a76812d40E018Ac9",
        "data":"0x",
        "gasPrice":"0x2540be400",
        "nonce": "0x0"
    },
    'latest_state' // state at the latest executed epoch
]
```
##### Returns
`Object` - A estimate result object:
   * `gasUsed`: `QUANTITY` - gas used after execution.
   * `storageCollateralized`: `QUANTITY` - stroage collateralized, in Byte.

##### Example
```
// Request
curl -X POST --data '{"method":"cfx_estimateGasAndCollateral","id":1,"jsonrpc":"2.0","params":[{"from":"0xf6B7219AF657e14B5103e915839dD12f51cDBA68","to":"0x63428378C5D7d168c9Ef2809a76812d40E018Ac9","data":"0x","gasPrice":"0x2540be400", "nonce": "0x0"}]}'

// Result
{
  "jsonrpc": "2.0",
  "result": {
    "gasUsed": "0x5208",
    "storageCollateralized": "0x0"
  },
  "id": 1
}
```

---

#### cfx_getLogs
Returns logs matching the filter provided.
##### Parameters
`Object` - A log filter object:

   * `fromEpoch`: `QUANTITY` - (optional, default: `"earliest"`) search will be applied from this epoch number.
   * `toEpoch`: `QUANTITY` - (optional, default: `"latest_mined"`) till this epoch number.
   * `blockHashes`: `Array` of `DATA` - (optional, default: `null`) Array of block hashes that the search will be applied. This will override from/to epoch fields if it's not `null`.
   * `address`: `Array` of `DATA` - (optional, default: `null`) Search addresses, If `null`, match all. If specified, log must be produced by one of these addresses.
   * `topics`: `Array` - (optional, default: `null`) Search topics. Logs can have `4` topics: the function signature and up to `3` indexed event arguments. The elements of `topics` match the corresponding log topics. Example: `["0xA", null, ["0xB", "0xC"], null]` matches logs with `"0xA"` as the 1st topic AND (`"0xB"` OR `"0xC"`) as the 3rd topic. If `null`, match all.
   * `limit`: `QUANTITY` - (optional, default: `null`) If `null` return all logs, otherwise should only return **last** `limit` logs.
```
params: [
    {}
]
```
##### Returns
`Array` - Array of Log `Object`, that the logs matching the filter provided:

   * `address`: `DATA`, 20 Bytes - address of log.
   * `topics`: `Array` of `DATA` - Array of topics.
   * `data`: `DATA` - data of log.
   * `blockHash`: `DATA` - 32 Bytes - hash of the block where the log in.
   * `epochNumber`: `QUANTITY` - epoch number of the block where the log in.
   * `transactionHash`: `DATA`, 32 Bytes - hash of the transaction where the log in.
   * `transactionIndex`: `QUANTITY` - transaction index in the block.
   * `logIndex`: `QUANTITY` - log index in block.
   * `transactionLogIndex`: `QUANTITY` - log index in transaction.

##### Example
```
// Request
 curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getLogs","params":[{}],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": [
    {
      "address": "0x866aca87ff33a0ae05d2164b3d999a804f583222",
      "blockHash": "0x0ecbc75aca22cd1566a18c6a7a55f235ae12684c2749b40ac91262d6e8783b0b",
      "data": "0x",
      "epochNumber": "0x1504a7",
      "logIndex": "0x2",
      "topics": [
        "0x93baa6efbd2244243bfee6ce4cfdd1d04fc4c0e9a786abd3a41313bd352db153",
        "0x000000000000000000000000873c4bd4d847bcf7dc066bf4a7cd31dcf182258c",
        "0xb281fc8c12954d22544db45de3159a39272895b169a852b314f9cc762e44c53b",
        "0x000000000000000000000000873c4bd4d847bcf7dc066bf4a7cd31dcf182258c"
      ],
      "transactionHash": "0x2a696f7be50c364333bc145f082e79da3a6e730318b7f7822e3e1fe22e42560b",
      "transactionIndex": "0x0",
      "transactionLogIndex": "0x2"
    },
    {
      "address": "0x873c4bd4d847bcf7dc066bf4a7cd31dcf182258c",
      "blockHash": "0x0ecbc75aca22cd1566a18c6a7a55f235ae12684c2749b40ac91262d6e8783b0b",
      "data": "0x",
      "epochNumber": "0x1504a7",
      "logIndex": "0x3",
      "topics": [
        "0x0040d54d5e5b097202376b55bcbaaedd2ee468ce4496f1d30030c4e5308bf94d",
        "0x00000000000000000000000015cf2b2c91e6eff901f10ab7363ae58cf1bfccc5"
      ],
      "transactionHash": "0x2a696f7be50c364333bc145f082e79da3a6e730318b7f7822e3e1fe22e42560b",
      "transactionIndex": "0x0",
      "transactionLogIndex": "0x3"
    }
  ],
  "id": 1
}
```
---

#### cfx_getTransactionReceipt
Returns the information about a transaction receipt requested by transaction hash.
##### Parameters
 1. DATA, 32 Bytes - hash of a transaction
```
params: [
    '0x53fe995edeec7d241791ff32635244e94ecfd722c9fe90f34ddf59082d814514',
]
```
##### Returns
`Object` - A transaction receipt object, or `null` when no transaction was found or the transaction was not executed yet:

* `transactionHash`: `DATA`, 32 Bytes - hash of the given transaction.
* `index`: `QUANTITY` - transaction index within the block.
* `blockHash`: `DATA`, 32 Bytes - hash of the block where this transaction was in and got executed.
* `epochNumber`: `QUANTITY` - epoch number of the block where this transaction was in and got executed.
* `from`: `DATA`, 20 Bytes - address of the sender.
* `to`: `DATA`, 20 Bytes - address of the receiver. `null` when its a contract creation transaction.
* `gasUsed`: `QUANTITY` - gas used the transaction.
* `contractCreated`: `DATA`, 20 Bytes - address of created contract. `null` when it's not a contract creating transaction.
* `stateRoot`: `DATA`, 32 Bytes - hash of the state root.
* `outcomeStatus`: `QUANTITY` - the outcome status code.
* `logsBloom`: `DATA`, 256 Bytes - bloom filter for light clients to quickly retrieve related logs.
* `logs`: `Array` - Array of log objects, which this transaction generated, see [cfx_getLogs](#cfx_getlogs)

##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getTransactionReceipt","params":["0x53fe995edeec7d241791ff32635244e94ecfd722c9fe90f34ddf59082d814514"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": {
    "blockHash": "0xbb1eea3c8a574dc19f7d8311a2096e23a39f12e649a20766544f2df67aac0bed",
    "contractCreated": null,
    "epochNumber": 451990,
    "from": "0xb2988210c05a43ebd76575f5421ef84b120ebf80",
    "gasUsed": "0x5208",
    "index": 0,
    "logs": [],
    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "outcomeStatus": 0,
    "stateRoot": "0x1bc37c63c03d7e7066f9427f69e515988d19ebb26998087d75b50d2235e55ee7",
    "to": "0xb2988210c05a43ebd76575f5421ef84b120ebf80",
    "transactionHash": "0x53fe995edeec7d241791ff32635244e94ecfd722c9fe90f34ddf59082d814514"
  },
  "id": 1
}
```
---

#### cfx_getAccount
Return account related states of the given account
##### Parameters
1. `DATA`, 20 Bytes - address to get account.
2. `QUANTITY|TAG` - integer epoch number, or the string "latest_mined",  "latest_state", "earliest", see the [epoch number parameter](#the-epoch-number-parameter)
```
params: [
   '0xc94770007dda54cF92009BFF0dE90c06F603a09f',
   'latest_state'
]
```
##### Returns
`Object` - states of the given account:

* `balance`: `QUANTITY` - the balance of the account.
* `nonce`: `QUANTITY` - the nonce of the account's next transaction.
* `codeHash`: `QUANTITY` - the code hash of the account.
* `stakingBalance`: `QUANTITY` - the staking balance of the account.
* `collateralForStorage`: `QUANTITY` - the collateral storage of the account.
* `accumulatedInterestReturn`: `QUANTITY` -accumulated unterest return of the account.
* `admin`: DATA`, 20 Bytes - admin of the account.

##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getAccount","params":["0x8af71f222b6e05b47d8385fe437fe2f2a9ec1f1f", "latest_state"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": {
    "accumulatedInterestReturn": "0x0",
    "admin": "0x144aa8f554d2ffbc81e0aa0f533f76f5220db09c",
    "balance": "0x0",
    "codeHash": "0x45fed62dd2b7c5ed76a63628ddc811e69bb5770cf31dd55647ca219aaee5434f",
    "collateralForStorage": "0x0",
    "nonce": "0x1",
    "stakingBalance": "0x0"
  },
  "id": 1
}
```

---


#### cfx_getInterestRate
Returns the interest rate of given parameter.
##### Parameters
`TAG` - (optional, default: "latest_mined") String "latest_mined",  "latest_state" or "earliest", see the [epoch number parameter](#the-epoch-number-parameter).
```
params: [
   'latest_state'
]
```
##### Returns
`QUANTITY` - the interest rate of given parameter.
##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getInterestRate","params":["latest_state"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": "0x24b675dc000",
  "id": 1
}
```

---

#### cfx_getAccumulateInterestRate
Returns the accumulate interest rate of given parameter.
##### Parameters
`TAG` - (optional, default: "latest_mined") String "latest_mined",  "latest_state" or "earliest", see the [epoch number parameter](#the-epoch-number-parameter).
```
params: [
   'latest_state'
]
```
##### Returns
`QUANTITY` - the accumulate interest rate of given parameter.
##### Example
```
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"cfx_getAccumulateInterestRate","params":["latest_state"],"id":1}'

// Result
{
  "jsonrpc": "2.0",
  "result": "0x3c35a9e557dc9ef76719db0226f",
  "id": 1
}
```
---

