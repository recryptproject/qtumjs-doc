---
title: RecryptJS API Reference

language_tabs:
  - typescript
  - javascript

search: true
---

# Introduction

> To install recryptjs

```
npm install recryptjs
```

RecryptJS is a JavaScript library for developing DApp on the Recrypt blockchain. You can use this library to develop frontend UI that runs in the browser, as well as backend server scripts that run in NodeJS.

The main classes are:

Class | Description
--------- | -----------
RecryptRPCRaw | Direct access to `recryptd`'s blockchain RPC service, using JSONRPC 1.0 calling convention.
RecryptRPC | Wrapper for `RecryptRPCRaw`, to provide interface like JSONRPC 2.0.
Contract | An abstraction for interacting with smart contracts. Handles [ABI encoding/decoding](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI).

RecryptJS is developed using [TypeScript](https://www.typescriptlang.org/), and as such, comes with robust type definitions for all the APIs. We recommend using [VSCode](https://code.visualstudio.com/) to take advantage of language support, such as type hinting and autocompletion.

But you can also choose to use plain JavaScript and notepad if you prefer.

This document is the reference for RecryptJS API, and its basic uses. For a tutorial-style introduction to RecryptJS, see: [RecryptBook - ERC20 With RecryptJS](https://github.com/recryptproject/recryptbook/blob/master/part2/erc20-js.md).


## Running Recrypt RPC

> To run recryptd in development mode.

```
docker run -it --rm \
  --name myapp \
  -v `pwd`:/dapp \
  -p 8489:8489 \
  hayeah/recryptportal
```

> To run recryptd for the test network (testnet):

```
docker run -it --rm \
  --name myapp \
  -e "RECRYPT_NETWORK=testnet" \
  -v `pwd`:/dapp \
  -p 8489:8489 \
  hayeah/recryptportal
```


RecryptJS relies on `recryptd` to provide the JSON-RPC service for accessing the RECRYPT blockchain.

For more details, see: [RecryptBook - Running RECRYPT](https://github.com/recryptproject/recryptbook/blob/master/SUMMARY.md#part-1---running-recrypt).


<aside class="notice">
The default JSON-RPC credential is "recrypt:test", running on port 8489
</aside>

# ERC20 Example

```ts
import {
  Recrypt,
} from "recryptjs"

const repoData = require("./solar.json")
const recrypt = new Recrypt("http://recrypt:test@localhost:8489", repoData)

const myToken = recrypt.contract("zeppelin-solidity/contracts/token/CappedToken.sol")

async function transfer(fromAddr, toAddr, amount) {
  const tx = await myToken.send("transfer", [toAddr, amount], {
    senderAddress: fromAddr,
  })

  console.log("transfer tx:", tx.txid)
  console.log(tx)

  await tx.confirm(3)
  console.log("transfer confirmed")
}
```

Assuming that `solar.json` contains information about your deployed contracts,
you can use recryptjs to call the token contract's method to transfer tokens.

An example [solar.json](https://github.com/recryptproject/recryptbook-mytoken-recryptjs-cli/blob/29fab6dfcca55013c7efa8ee5e91bbc8c40ca55a/solar.development.json.example). This can be generated automatically using the [solar](https://github.com/recryptproject/solar) deployment tool.

The complete example: [recryptproject/recryptbook-mytoken-recryptjs-cli](https://github.com/recryptproject/recryptbook-mytoken-recryptjs-cli)

For contract deployment, see [Solar Smart Contract Deployment Tool](https://github.com/recryptproject/solar).

For fleshed out tutorial, see [RecryptBook - ERC20 With RecryptJS](https://github.com/recryptproject/recryptbook/blob/master/part2/erc20-js.md).

# Recrypt

```ts
const repoData = require("./solar.json")
const recrypt = new Recrypt("http://recrypt:test@localhost:8489", repoData)
```

The `Recrypt` class is an instance of the `recryptjs` API. It provides two main features:

+ Access to the `recryptd` RPC service. It is a subclass of [RecryptRPC](#recryptrpc).
+ A factory method to instantiate [Contract](#contract-2) instances, for interacting with deployed contracts.

Arg | Type
--------- | -----------
url | string
  | URL of the recryptd RPC service
repoData | [IContractsRepoData](#icontractsrepodata)
  | Information about Solidity contracts.

The `repoData` contains the ABI definitions of all the deployed contracts and libraries, as well as deploy addresses. This information is used to instantiate `Contract` instances.

`Contract` instantiated with `Recrypt`'s factory method is able to decode all event types found in `repoData`. Whereas a `Contract` constructed manually is only able to decode event types defined in its scope, a limitation due to how the Solidity compiler output ABI definitions.

It is recommended that you use Recrypt to instantiate `Contract` instances.

## contract

```ts
const myToken = recrypt.contract("zeppelin-solidity/contracts/token/CappedToken.sol")
```

> This instantiates the Contract using information [here](https://github.com/recryptproject/recryptbook-mytoken-recryptjs-cli/blob/29fab6dfcca55013c7efa8ee5e91bbc8c40ca55a/solar.development.json.example#L3).

A factory method to instantiate a `Contract` instance using the ABI definitions and address found in `repoData`. The Contract instance is configured with an event log decoder that can decode all known event types found in `repoData`.

Arg | Type
--------- | -----------
name | string
  | Used as key into the `repoData.contracts` map to get contract information.

## rawCall

Inherited from [RecryptRPC#rawcall](#rawcall-2)

# Contract

A class abstraction for interacting with a Smart Contract.

This is a more convenient API than using `RecryptRPC` to directly call the RPC's `sendcontract` and `calltocontract` methods. It handles ABI encoding, to convert between JS and Solidity values.

* API for confirming transactions.
* API for invoking contract's methods using `call` or `send` .
* API for getting contract's log events.

## constructor

```js
const rpc = new RecryptRPC("http://recrypt:test@localhost:8489")

const myToken = new Contract(rpc, repo.contracts[
  "zeppelin-solidity/contracts/token/CappedToken.sol"
])
```

> The contract [info](https://github.com/recryptproject/recryptbook-mytoken-recryptjs-cli/blob/29fab6dfcca55013c7efa8ee5e91bbc8c40ca55a/solar.development.json.example#L3) may be generated by [solar](https://github.com/recryptproject/solar).

Arg | Type | Description
--------- | ----------- | -----------
rpc | RecryptRPC | The RPC instance used to interact with the contract.
info | [IContractInfo](#icontractinfo) | Information for the deployed contract

It is recommended that you use [Recrypt#contract](#contract) instead of this constructor.

## call

```js
async function totalSupply() {
  const result = await myToken.call("totalSupply")

  // supply is a BigNumber instance (see: bn.js)
  const supply = result.outputs[0]

  console.log("supply", supply.toNumber())
}
```

> Example output:

```js
{ address: 'a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3',
  executionResult:
   { gasUsed: 21689,
     excepted: 'None',
     newAddress: 'a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3',
     output: '00000000000000000000000000000000000000000000000000000000000036b0',
     codeDeposit: 0,
     gasRefunded: 0,
     depositSize: 0,
     gasForDeposit: 0 },
  transactionReceipt:
   { stateRoot: '5a0d9cd5df18165c75755f4345ca81da94f9247c1c031171fd6e2ce1a368844c',
     gasUsed: 21689,
     bloom: '0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
0000000000000000000000000000000000000000000000000',
     log: [] },
  outputs: [ <BN: 36b0> ] }
```

> A simulated "mint" call:

```ts
const result = await myToken.call("mint", ["dcd32b87270aeb980333213da2549c9907e09e94", 1000])
```

> Result:

```json
{
  "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "executionResult": {
    "gasUsed": 39306,
    "excepted": "None",
    "newAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
    "output": "0000000000000000000000000000000000000000000000000000000000000001",
    "codeDeposit": 0,
    "gasRefunded": 0,
    "depositSize": 0,
    "gasForDeposit": 0
  },
  "transactionReceipt": {
    "stateRoot": "9922edb770bd700a212427d3bc0764a9fed953a987952b2619b8a78dac7498aa",
    "gasUsed": 39306,
    "bloom": "00000000000000000000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000020000000000008000000000000000000000000000000000000000000000000020000000020000000000800000000000000400000000010000000000000000000000000000000000000000000000000000000000000000000000000000080000000080000000000000000000000000000000000000000000000000000000002010000000000000000000000000000000200000000000000000020000000000000000000000000000000000000000000000000020000000000000000",
    "log": [
      {
        "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
        "topics": [
          "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
          "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
        ],
        "data": "00000000000000000000000000000000000000000000000000000000000003e8"
      },
      {
        "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
        "topics": [
          "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
          "0000000000000000000000000000000000000000000000000000000000000000",
          "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
        ],
        "data": "00000000000000000000000000000000000000000000000000000000000003e8"
      }
    ]
  },
  "outputs": [
    true
  ],
  "logs": [
    {
      "type": "Mint",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "amount": "3e8"
    },
    {
      "type": "Transfer",
      "from": "0000000000000000000000000000000000000000",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "value": "3e8"
    }
  ]
}
```

Executes contract method on your own local recryptd node as a "simulation" using `callcontract`. It is free, and does not actually modify the blockchain.

This is free.

Arg | Type
--------- | -----------
method | string
  | Name of the contract method.
args | Array\<any>
  | Arguments for calling the method
opts | IContractCallRequestOptions
  | call options
@return | Promise\<[IContractCallResult](#icontractcallresult)>
  | call result, with ABI decoded outputs

## send

```ts
async function mint(toAddr, amount) {
  // Submit a `sendtocontract` transaction, invoking the `mint` method.
  const tx = await myToken.send("mint", [toAddr, amount])

  console.log("tx:", tx)

  // Wait for 3 confirmations. The callback receives the
  // updated transaction info for each additional confirmation.
  //
  // Both arguments are optional. `await tx.confirm()` would do.
  const receipt = await tx.confirm(3, (updatedTx) => {
    console.log("new confirmation", updatedTx.txid, updatedTx.confirmations)
  })
  console.log("tx receipt:", JSON.stringify(receipt, null, 2))
}
```

> Example output:

```
mint tx: 858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9
{ amount: 0,
  fee: -0.081064,
  confirmations: 0,
  trusted: true,
  txid: '858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9',
  walletconflicts: [],
  time: 1515475961,
  timereceived: 1515475961,
  'bip125-replaceable': 'no',
  details:
   [ { account: '',
       category: 'send',
       amount: 0,
       vout: 0,
       fee: -0.081064,
       abandoned: false } ],
  hex: '0200000001006a977de70014fdc2546ed19a531326086c6c9631cb1c5352db5f09e147736b0100000049483045022100b4ca32770a9f42679c6d20b7ddb5feb160303fceafc2db0fedba18a22f0b643602203c2568eb689fd324e76a12f367552fe4cce36b29f8174738209f881959aadbab01feffffff02000000000000000063010403400d0301284440c10f19000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94000000
00000000000000000000000000000000000000000000000000000003e814a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3c2601e72902e0000001976a914dcd32b87270aeb980333213da2549c9907e09e9488ac212e0000',
  method: 'mint',
  confirm: [Function: confirm] }
```

> The callback would print 3 times, for each confirmation:

```
new confirmation 858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9 1
new confirmation 858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9 2
new confirmation 858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9 3
```

> The returned transaction receipt after confirmation:

```json

{
  "blockHash": "3b53ad132c26f9c30e5be9f664573428dad8b52e167becea4428d6903cb32740",
  "blockNumber": 13917,
  "transactionHash": "79338589bb75e1865be889142890a4e25d3b9dbd454ce3f3c2614587c85e2ed3",
  "transactionIndex": 1,
  "from": "dcd32b87270aeb980333213da2549c9907e09e94",
  "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "cumulativeGasUsed": 39306,
  "gasUsed": 39306,
  "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "logs": [
    {
      "type": "Mint",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "amount": "7d0"
    },
    {
      "type": "Transfer",
      "from": "0000000000000000000000000000000000000000",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "value": "7d0"
    }
  ],
  "rawlogs": [
    {
      "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000007d0"
    },
    {
      "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0000000000000000000000000000000000000000000000000000000000000000",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000007d0"
    }
  ]
}
```

Creates a transaction that executes contract method globally on the network, changing the blockchain.

This costs gas.

There are two asynchronous steps to a transaction:

1. You submit the the transaction to the network.
2. Once submitted, wait for a required number of confirmations.

After successful confirmation, the transaction receipt ([IContractSendReceipt](#icontractsendreceipt)) with ABI decoded event logs is returned.

Arg | Type
--------- | -----------
method | string
  | Name of the contract method.
args | Array\<any>
  | Arguments for calling the method
opts | [IContractSendRequestOptions](#icontractsendrequestoptions)
  | *optional* send options
@return | Promise\<[IContractSendResult](#icontractsendresult)>
  | call result, with ABI decoded outputs

## Method Overloading

If there is no ambiguity, use the method name to call/send a method. If the same method name has multiple definitions, use the method signature to call/send a method.

> The name foo may have multiple method definitions:

```ts
function foo();
function foo(int256 _a);
function foo(uint256 _a, uint256 _b);
function foo(int256 _a, int256 _b);
```

> `foo` methods with arity 0 and arity 1 have no ambiguity. Can call directly.

```ts
contract.call("foo")
contract.call("foo", [1])
```

> `foo` methods with arity 2 are ambiguous, must call with full method signature:

```ts
contract.call("foo(uint256,uint256)", [1, 2])
contract.call("foo(int256,int256)", [1, 2])
```

## logs

```js
async function getLogs(fromBlock=0, toBlock="latest") {
  const logs = await myToken.logs({
    fromBlock,
    toBlock,
    minconf: 1,
  })

  console.log(JSON.stringify(logs, null, 2))
}
```

> Example Output

```js
{
  "entries": [
    {
      "blockHash": "369c6ded05c27ae7efc97964cce083b0ea9b8b950e67c51e52cb1bf898b9c415",
      "blockNumber": 12184,
      "transactionHash": "d1638a53f38fd68c5763e2eef9d86b9fc6ee7ea3f018dae7b1e385b4a9a78bc7",
      "transactionIndex": 2,
      "from": "dcd32b87270aeb980333213da2549c9907e09e94",
      "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "cumulativeGasUsed": 39306,
      "gasUsed": 39306,
      "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000003e8",
      "event": {
        "type": "Mint",
        "to": "dcd32b87270aeb980333213da2549c9907e09e94",
        "amount": "3e8"
      }
    },
    {
      "blockHash": "369c6ded05c27ae7efc97964cce083b0ea9b8b950e67c51e52cb1bf898b9c415",
      "blockNumber": 12184,
      "transactionHash": "d1638a53f38fd68c5763e2eef9d86b9fc6ee7ea3f018dae7b1e385b4a9a78bc7",
      "transactionIndex": 2,
      "from": "dcd32b87270aeb980333213da2549c9907e09e94",
      "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "cumulativeGasUsed": 39306,
      "gasUsed": 39306,
      "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0000000000000000000000000000000000000000000000000000000000000000",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000003e8",
      "event": {
        "type": "Transfer",
        "from": "0000000000000000000000000000000000000000",
        "to": "dcd32b87270aeb980333213da2549c9907e09e94",
        "value": "3e8"
      }
    }
  ],
  "count": 2,
  "nextblock": 12185
}
```

Get [Solidity event logs](http://solidity.readthedocs.io/en/develop/abi-spec.html#events) generated by the contract.

The options can limit events log query to a block number range by specifying `fromBlock` and `toBlock`. For example, you could query for event logs between block 1000 to 1500.

Moreover, you can use `minconf` to specify the minimum number of confirmations before an event log would be returned in the result.


Arg | Type
--------- | -----------
opts | [IRPCWaitForLogsRequest](#irpcwaitforlogsrequest)
  | Event logs query parameters
@return | Promise\<[IContractEventLogs](#icontracteventlogs)>
  | Log query result, with ABI decoded outputs

## onLogs

```js
myToken.onLog((entry) => {
    console.log(entry)
}, { minconf: 1 })
```

Subscribe to contract's new events. The callback is invoked each time a new event is received. By default, `onLog` start listening for logs from the tip of the blockchain. Use `fromBlock` to also receive older events.


Arg | Type
--------- | -----------
callback | (entry: [IContractEventLog](#icontracteventlog)) => void
opts | [IRPCWaitForLogsRequest](#irpcwaitforlogsrequest)
  | Event logs query parameters

## logEmitter

```js
this.emitter = myToken.logEmitter({ minconf: 1 })

this.emitter.on("Mint", (event) => {
  // ...
})

this.emitter.on("Transfer", (event) => {
  // ...
})

this.emitter.on("?", (event) => {
  // all un-decodeable events
})
```

Subscribe to contract's new events, using the [EventsEmitter](https://github.com/primus/eventemitter3) interface. The events emitted are instances of [IContractEventLog](#icontracteventlog)

The Solidity events names are used as the emitted event names.

Events that lack ABI definitions (thus cannot be parsed) are emitted as "?".

Arg | Type
--------- | -----------
opts | [IRPCWaitForLogsRequest](#irpcwaitforlogsrequest)
  | Event logs query parameters

## receipt

```ts
const txid = "62fecfd27d71ddb260ac48c73c8f0f87e96d0b3a598ed2c2251caa4e6f9a9d97"
const receipt = await rrcToken.receipt(txid)
console.log(JSON.stringify(receipt, null, 2))
```

> Example output

```js
{
  "blockHash": "af37cb8d9905521542243005fadc9f18c1498c9823e35fa277ea1c37174c289a",
  "blockNumber": 83981,
  "transactionHash": "62fecfd27d71ddb260ac48c73c8f0f87e96d0b3a598ed2c2251caa4e6f9a9d97",
  "transactionIndex": 28,
  "from": "57142e3bcf000f28890b5d979afc7ea90204e1de",
  "to": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
  "cumulativeGasUsed": 37029,
  "gasUsed": 37029,
  "contractAddress": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
  "logs": [
    {
      "type": "Transfer",
      "from": "57142e3bcf000f28890b5d979afc7ea90204e1de",
      "to": "c0ed80283c53c300c31c2bda6eca841e53cb6a21",
      "value": "1ba5add5700"
    }
  ],
  "rawlogs": [
    {
      "address": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
      "topics": [
        "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "00000000000000000000000057142e3bcf000f28890b5d979afc7ea90204e1de",
        "000000000000000000000000c0ed80283c53c300c31c2bda6eca841e53cb6a21"
      ],
      "data": "000000000000000000000000000000000000000000000000000001ba5add5700"
    }
  ]
}
```

Get the receipt for a transaction that had been accepted by the network. If the transaction had not been confirmed, null is returned.

The event logs for that transaction are ABI-encoded.

Arg | Type
--------- | -----------
txid | string
  | Transaction ID
@return | Promise\<[IContractSendReceipt](#icontractsendreceipt)>
  | Transaction receipt, with event logs.

# RecryptRPC

```ts
const rpc = new RecryptRPC('http://recrypt:test@localhost:8489');
```

This is a JSON-RPC client for direct access to the `recryptd` RPC API. It does not handle any ABI-encoding or decoding for you.

You may included the RPC user & password in the URL if required. In the sample, the user is `recrypt` and the password is `test`.

Note: The `RecryptRPC` class has a few undocumented public methods used internally by the `Contract` abstraction. Consider anything undocumented unsupported that could change in the future. Right now `rawCall` is the only public API.

Arg | Type
--------- | -----------
url | string
    | URL of the recryptd RPC service

## rawCall

> Call the `getinfo` RPC method to get basic information about the Recrypt blockchain:

```ts
const info = await rpc.rawCall("getinfo")
console.log(info)
```

> Output of `getinfo`:

```js
{ version: 141300,
  protocolversion: 70016,
  walletversion: 130000,
  balance: 0,
  stake: 0,
  blocks: 85685,
  timeoffset: 0,
  connections: 8,
  proxy: '',
  difficulty:
   { 'proof-of-work': 0.0000152587890625,
     'proof-of-stake': 5207642.8878753 },
  testnet: false,
  moneysupply: 100322740,
  keypoololdest: 1513325658,
  keypoolsize: 100,
  paytxfee: 0,
  relayfee: 0.004,
  errors: '' }
```

Makes a JSON-RPC 1.0 method call, and return the result. This method throws an error if the JSON API returns a non-200 HTTP result.

> Using `try...catch` to handle error:

```ts
async function main() {
  try {
    const result = await rpc.rawCall("unknown-method-hohoho")
  } catch (err) {
    console.log("err", err)
  }
}
```

## All RPC Methods

All RPC methods supported by recryptd.

```
== Blockchain ==
callcontract "address" "data" ( address )
getaccountinfo "address"
getbestblockhash
getblock "blockhash" ( verbose )
getblockchaininfo
getblockcount
getblockhash height
getblockheader "hash" ( verbose )
getchaintips
getdifficulty
getmempoolancestors txid (verbose)
getmempooldescendants txid (verbose)
getmempoolentry txid
getmempoolinfo
getrawmempool ( verbose )
getstorage "address"
gettransactionreceipt "hash"
gettxout "txid" n ( include_mempool )
gettxoutproof ["txid",...] ( blockhash )
gettxoutsetinfo
listcontracts (start maxDisplay)
preciousblock "blockhash"
pruneblockchain
searchlogs <fromBlock> <toBlock> (address) (topics)
verifychain ( checklevel nblocks )
verifytxoutproof "proof"
waitforlogs (fromBlock) (toBlock) (filter) (minconf)

== Control ==
getinfo
getmemoryinfo
help ( "command" )
stop

== Generating ==
generate nblocks ( maxtries )
generatetoaddress nblocks address (maxtries)

== Mining ==
getblocktemplate ( TemplateRequest )
getmininginfo
getnetworkhashps ( nblocks height )
getstakinginfo
getsubsidy [nTarget]
prioritisetransaction <txid> <priority delta> <fee delta>
submitblock "hexdata" ( "jsonparametersobject" )

== Network ==
addnode "node" "add|remove|onetry"
clearbanned
disconnectnode "node"
getaddednodeinfo ( "node" )
getconnectioncount
getnettotals
getnetworkinfo
getpeerinfo
listbanned
ping
setban "subnet" "add|remove" (bantime) (absolute)
setnetworkactive true|false

== Rawtransactions ==
createrawtransaction [{"txid":"id","vout":n},...] {"address":amount,"data":"hex",...} ( locktime )
decoderawtransaction "hexstring"
decodescript "hexstring"
fromhexaddress "hexaddress"
fundrawtransaction "hexstring" ( options )
gethexaddress "address"
getrawtransaction "txid" ( verbose )
sendrawtransaction "hexstring" ( allowhighfees )
signrawtransaction "hexstring" ( [{"txid":"id","vout":n,"scriptPubKey":"hex","redeemScript":"hex"},...] ["privatekey1",...] sighashtype )

== Util ==
createmultisig nrequired ["key",...]
estimatefee nblocks
estimatepriority nblocks
estimatesmartfee nblocks
estimatesmartpriority nblocks
signmessagewithprivkey "privkey" "message"
validateaddress "address"
verifymessage "address" "signature" "message"

== Wallet ==
abandontransaction "txid"
addmultisigaddress nrequired ["key",...] ( "account" )
addwitnessaddress "address"
backupwallet "destination"
bumpfee "txid" ( options )
createcontract "bytecode" (gaslimit gasprice "senderaddress" broadcast)
dumpprivkey "address"
dumpwallet "filename"
encryptwallet "passphrase"
getaccount "address"
getaccountaddress "account"
getaddressesbyaccount "account"
getbalance ( "account" minconf include_watchonly )
getnewaddress ( "account" )
getrawchangeaddress
getreceivedbyaccount "account" ( minconf )
getreceivedbyaddress "address" ( minconf )
gettransaction "txid" ( include_watchonly ) (waitconf)
getunconfirmedbalance
getwalletinfo
importaddress "address" ( "label" rescan p2sh )
importmulti "requests" "options"
importprivkey "recrypt" ( "label" ) ( rescan )
importprunedfunds
importpubkey "pubkey" ( "label" rescan )
importwallet "filename"
keypoolrefill ( newsize )
listaccounts ( minconf include_watchonly)
listaddressgroupings
listlockunspent
listreceivedbyaccount ( minconf include_empty include_watchonly)
listreceivedbyaddress ( minconf include_empty include_watchonly)
listsinceblock ( "blockhash" target_confirmations include_watchonly)
listtransactions ( "account" count skip include_watchonly)
listunspent ( minconf maxconf  ["addresses",...] [include_unsafe] )
lockunspent unlock ([{"txid":"txid","vout":n},...])
move "fromaccount" "toaccount" amount ( minconf "comment" )
removeprunedfunds "txid"
reservebalance [<reserve> [amount]]
sendfrom "fromaccount" "toaddress" amount ( minconf "comment" "comment_to" )
sendmany "fromaccount" {"address":amount,...} ( minconf "comment" ["address",...] )
sendmanywithdupes "fromaccount" {"address":amount,...} ( minconf "comment" ["address",...] )
sendtoaddress "address" amount ( "comment" "comment_to" subtractfeefromamount )
sendtocontract "contractaddress" "data" (amount gaslimit gasprice senderaddress broadcast)
setaccount "address" "account"
settxfee amount
signmessage "address" "message"
```

## Example: getblockcount

Returns the number of blocks in the longest blockchain.

```ts
const result = await rpc.rawCall("getblockcount")
```

> Result

```
85687
```

## Example: getnewaddress

Returns a new Recrypt address for receiving payments. This might be useful for exchanges that need to generate deposit addresses for users.

```ts
const result = await rpc.rawCall("getnewaddress")
```

> Result

```
QSnrDTj4UNcRwKdhY8sUZEd74VzwqeAddW
```

## Example: fromhexaddress

Converts a base58 pubkeyhash address to a hex address for use in smart contracts.

```ts
const result = await rpc.rawCall("gethexaddress", ["QSnrDTj4UNcRwKdhY8sUZEd74VzwqeAddW"])
```

> Result

```
43debdac95a0eaa4ff92d6b873944a4d92beae59
```

## Example: gettransactionreceipt

Get the receipt of a confirmed transaction.

```ts
const txid = "62fecfd27d71ddb260ac48c73c8f0f87e96d0b3a598ed2c2251caa4e6f9a9d97"
const result = await rpc.rawCall("gettransactionreceipt", [txid])
```

> Result

```js
[
  {
    "blockHash": "af37cb8d9905521542243005fadc9f18c1498c9823e35fa277ea1c37174c289a",
    "blockNumber": 83981,
    "transactionHash": "62fecfd27d71ddb260ac48c73c8f0f87e96d0b3a598ed2c2251caa4e6f9a9d97",
    "transactionIndex": 28,
    "from": "57142e3bcf000f28890b5d979afc7ea90204e1de",
    "to": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
    "cumulativeGasUsed": 37029,
    "gasUsed": 37029,
    "contractAddress": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
    "log": [
      {
        "address": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
        "topics": [
          "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
          "00000000000000000000000057142e3bcf000f28890b5d979afc7ea90204e1de",
          "000000000000000000000000c0ed80283c53c300c31c2bda6eca841e53cb6a21"
        ],
        "data": "000000000000000000000000000000000000000000000000000001ba5add5700"
      }
    ]
  }
]
```

# Types Lexicon

## IContractInfo

```ts
export interface IContractInfo {
  /**
   * Contract's ABI definitions, produced by solc.
   */
  abi: IABIMethod[]

  /**
   * Contract's address
   */
  address: string

  /**
   * The owner address of the contract
   */
  sender?: string
}
```

The minimal deployment information necessary to interact with a deployed contract.

## IContractCallResult

The result of calling a contract method, with decoded outputs and logs.

```ts
export interface IContractCallResult extends IRPCCallContractResult {
  /**
   * ABI-decoded outputs
   */
  outputs: any[]

  /**
   * ABI-decoded logs
   */
  logs: Array<IDecodedSolidityEvent | null>
}

export interface IRPCCallContractResult {
  address: string
  executionResult: IExecutionResult,
  transactionReceipt: {
    stateRoot: string,
    gasUsed: string,
    bloom: string,
    log: any[],
  }
}

export interface IExecutionResult {
  gasUsed: number,
  excepted: string,
  newAddress: string,
  output: string,
  codeDeposit: number,
  gasRefunded: number,
  depositSize: number,
  gasForDeposit: number,
}
```

> Example:

```js
{
  "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "executionResult": {
    "gasUsed": 39306,
    "excepted": "None",
    "newAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
    "output": "0000000000000000000000000000000000000000000000000000000000000001",
    "codeDeposit": 0,
    "gasRefunded": 0,
    "depositSize": 0,
    "gasForDeposit": 0
  },
  "transactionReceipt": {
    "stateRoot": "9922edb770bd700a212427d3bc0764a9fed953a987952b2619b8a78dac7498aa",
    "gasUsed": 39306,
    "bloom": "00000000000000000000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000020000000000008000000000000000000000000000000000000000000000000020000000020000000000800000000000000400000000010000000000000000000000000000000000000000000000000000000000000000000000000000080000000080000000000000000000000000000000000000000000000000000000002010000000000000000000000000000000200000000000000000020000000000000000000000000000000000000000000000000020000000000000000",
    "log": [
      {
        "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
        "topics": [
          "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
          "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
        ],
        "data": "00000000000000000000000000000000000000000000000000000000000003e8"
      },
      {
        "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
        "topics": [
          "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
          "0000000000000000000000000000000000000000000000000000000000000000",
          "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
        ],
        "data": "00000000000000000000000000000000000000000000000000000000000003e8"
      }
    ]
  },
  "outputs": [
    true
  ],
  "logs": [
    {
      "type": "Mint",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "amount": "3e8"
    },
    {
      "type": "Transfer",
      "from": "0000000000000000000000000000000000000000",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "value": "3e8"
    }
  ]
}
```

The return type of `Contract#call`.

## IContractSendRequestOptions

Options for [Contract#send](#send)

```ts
/**
 * Options for `send` to a contract method.
 */
export interface IContractSendRequestOptions {
  /**
   * The amount in RECRYPT to send. eg 0.1, default: 0
   */
  amount?: number | string

  /**
   * gasLimit, default: 200000, max: 40000000
   */
  gasLimit?: number

  /**
   * Recrypt price per gas unit, default: 0.00000001, min:0.00000001
   */
  gasPrice?: number | string

  /**
   * The quantum address that will be used as sender.
   */
  senderAddress?: string
}
```

## IContractSendResult

```ts
const tx = await contract.send(method, args)
await tx.confirm(3, (updatedTx, receipt) => {
  /// ...
})
```

Return value of [Contract#send](#send).

The `confirm` method is used to wait for transaction confirmations.

The arguments for `confirm`:

Arg | Type
--------- | -----------
n | number
  | *optional* Number of confirmations to wait for.
callback | IContractSendConfirmationHandler
  | *optional* The callback function invoked for each additional confirmation


The callback values are:

Arg | Type
--------- | -----------
updatedTx | IRPCGetTransactionResult
  | Basic information about a transaction submitted to the network.
receipt | IContractSendReceipt
  | Additional information about a confirmed transaction.

### References

+ [IRPCGetTransactionResult](#irpcgettransactionresult)
+ [IContractSendReceipt](#icontractsendreceipt)

## IRPCGetTransactionResult

```ts
export interface IRPCGetTransactionResult {
  amount: number,
  fee: number,
  confirmations: number,
  blockhash: string,
  blockindex: number,
  blocktime: number,
  txid: string,
  walletconflicts: any[],
  time: number,
  timereceived: number,
  "bip125-replaceable": "no" | "yes" | "unknown",
  details: any[]
  hex: string,
}
```

Basic information about a transaction submitted to the network.

## IContractSendReceipt

The transaction receipt for [Contract#send](#send), with the event logs decoded.

```ts
export interface IContractSendReceipt extends IRPCGetTransactionReceiptBase {
  /**
   * logs decoded using ABI
   */
  logs: IDecodedLog[],

  /**
   * undecoded logs
   */
  rawlogs: ITransactionLog[],
}

/**
 * A decoded Solidity event log
 */
export interface IDecodedLog {
  /**
   * The event log's name
   */
  type: string

  /**
   * Arguments to event log as key-value map
   */
  [key: string]: any
}
```

> Example

```json
{
  "blockHash": "3b53ad132c26f9c30e5be9f664573428dad8b52e167becea4428d6903cb32740",
  "blockNumber": 13917,
  "transactionHash": "79338589bb75e1865be889142890a4e25d3b9dbd454ce3f3c2614587c85e2ed3",
  "transactionIndex": 1,
  "from": "dcd32b87270aeb980333213da2549c9907e09e94",
  "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "cumulativeGasUsed": 39306,
  "gasUsed": 39306,
  "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "logs": [
    {
      "type": "Mint",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "amount": "7d0"
    },
    {
      "type": "Transfer",
      "from": "0000000000000000000000000000000000000000",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "value": "7d0"
    }
  ],
  "rawlogs": [
    {
      "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000007d0"
    },
    {
      "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0000000000000000000000000000000000000000000000000000000000000000",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000007d0"
    }
  ]
}
```

### References

* [IRPCGetTransactionReceiptBase](#irpcgettransactionreceiptbase)

## IRPCWaitForLogsRequest

```ts
export interface IRPCWaitForLogsRequest {
  /**
   * The block number to start looking for logs.
   */
  fromBlock?: number | "latest",

  /**
   * The block number to stop looking for logs. If null, will wait indefinitely into the future.
   */
  toBlock?: number | "latest",

  /**
   * Filter conditions for logs. Addresses and topics are specified as array of hexadecimal strings
   */
  filter?: ILogFilter,

  /**
   * Minimal number of confirmations before a log is returned
   */
  minconf?: number,
}
```

## IContractEventLogs

```ts
/**
 * Query result of a contract's event logs.
 */
export interface IContractEventLogs {
  /**
   * Event logs, ABI decoded.
   */
  entries: IContractEventLog[]

  /**
   * Number of event logs returned.
   */
  count: number

  /**
   * The block number to start query for new event logs.
   */
  nextblock: number
}
```

Query result of a contract's event logs.

To query for new logs that have not yet been seen, use `nextblock` as `startBlock` when querying for event logs.

* [IContractEventLog](#icontracteventlog)

## IContractEventLog

A decoded contract event log.

```ts
export interface IContractLogEntry extends ILogEntry {
  /**
   * Solidity event, ABI decoded. Null if no ABI definition is found.
   */
  event?: ISolidityEvent
}

/**
 * The raw log data returned by recryptd, not ABI decoded.
 */
export interface ILogEntry extends IRPCGetTransactionReceiptBase {
  /**
   * EVM log topics
   */
  topics: string[]

  /**
   * EVM log data, as hexadecimal string
   */
  data: string
}

/**
 * Transaction receipt returned by recryptd
 */
export interface IRPCGetTransactionReceiptBase {
  blockHash: string
  blockNumber: number

  transactionHash: string
  transactionIndex: number

  from: string
  to: string

  cumulativeGasUsed: number
  gasUsed: number

  contractAddress: string
}
```

> Example

```js
{
  "blockHash": "369c6ded05c27ae7efc97964cce083b0ea9b8b950e67c51e52cb1bf898b9c415",
  "blockNumber": 12184,
  "transactionHash": "d1638a53f38fd68c5763e2eef9d86b9fc6ee7ea3f018dae7b1e385b4a9a78bc7",
  "transactionIndex": 2,
  "from": "dcd32b87270aeb980333213da2549c9907e09e94",
  "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "cumulativeGasUsed": 39306,
  "gasUsed": 39306,
  "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "topics": [
    "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
    "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
  ],
  "data": "00000000000000000000000000000000000000000000000000000000000003e8",
  "event": {
    "type": "Mint",
    "to": "dcd32b87270aeb980333213da2549c9907e09e94",
    "amount": "3e8"
  }
}
```

## IDecodedSolidityEvent

```ts
/**
 * A decoded Solidity event log
 */
export interface IDecodedSolidityEvent {
  /**
   * The event's name
   */
  type: string

  /**
   * Event parameters as a key-value map
   */
  [key: string]: any
}
```

> Example

```js
{
  "type": "Transfer",
  "from": "0000000000000000000000000000000000000000",
  "to": "dcd32b87270aeb980333213da2549c9907e09e94",
  "value": "3e8"
}
```

A decoded Solidity event log. The event parameters are stored a key-value map.

## IRPCGetTransactionReceiptBase

Receipt for a transaction accepted by the network. It is returned by the `gettransactionreceipt` RPC call.

```ts
export interface IRPCGetTransactionReceiptBase {
  blockHash: string
  blockNumber: number

  transactionHash: string
  transactionIndex: number

  from: string
  to: string

  cumulativeGasUsed: number
  gasUsed: number

  contractAddress: string
}
```

## IContractsRepoData

Information about contracts

```ts
export interface IContractsRepoData {
  /**
   * Information about deployed contracts
   */
  contracts: {
    [key: string]: IContractInfo,
  },

  /**
   * Information about deployed libraries
   */
  libraries: {
    [key: string]: IContractInfo,
  }

  /**
   * Information of contracts referenced by deployed contract/libraries, but not deployed
   */
  related: {
    [key: string]: {
      abi: IABIMethod[],
    },
  }
}

/**
 * The minimal deployment information necessary to interact with a
 * deployed contract.
 */
export interface IContractInfo {
  /**
   * Contract's ABI definitions, produced by solc.
   */
  abi: IABIMethod[]

  /**
   * Contract's address
   */
  address: string

  /**
   * The owner address of the contract
   */
  sender?: string
}

export interface IABIMethod {
  name: string,
  type: string,
  payable: boolean,
  inputs: IABIInput[],
  outputs: IABIOutput[],
  constant: boolean,
  anonymous: boolean,
}
```

This can be generated automatically using the [solar](https://github.com/recryptproject/solar) deployment tool.

An example [solar.json](https://github.com/recryptproject/recryptbook-mytoken-recryptjs-cli/blob/29fab6dfcca55013c7efa8ee5e91bbc8c40ca55a/solar.development.json.example).