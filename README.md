# 3nodes: Private transactions on Azure Blockchain Service

Follow this instruction to perform private transactions on Azure Blockchain Service (Quorum). Uses `````` in a similar style to the classic [7nodes](https://github.com/jpmorganchase/quorum-examples/tree/master/examples/7nodes) example.

## Required Setup

* 3 member consortium on Azure Blockchain Service: 
    * [Create an Azure Blockchain Service blockchain member](https://docs.microsoft.com/en-us/azure/blockchain/service/create-member)
    * [Inviting more members](https://docs.microsoft.com/en-us/azure/blockchain/service/manage-consortium-powershell#new-blockchainmemberinvitation)
* [](https://.ethereum.org/downloads/)

In setting up the member Azure Blockchain Service instances, be sure to take note of the node password that you use:

<img width="700" src="https://github.com/vitoc/3nodes/blob/master/node_password.PNG" />

This instruction uses this node password, identified by *_NODE_PASSWORD below.

## Example Constants

The next steps in this instruction will use example constants that you should replace with your values (usually in hash form). These example constants will appear CAPITALISED.

## Naming and assumptions

It is assumed that you have the required setup of 3 Azure Blockchain Service instances, each belonging theoretically to a different member of the consortium and each with their own transaction nodes.

We shall name these 3 separate nodes Alice, Bob and Carol.

## Connection Strings and Public Keys

While Azure Blockchain Service takes care of the provisioning and setup of the nodes, you will need to have the public keys and connection strings of these nodes in order to follow the rest of the steps. These public keys can be obtained conveniently via the Azure portal as follows:

<img width="700" src="https://github.com/vitoc/3nodes/blob/master/pk_and_cs.PNG" />

Click on the specific transaction node to get to the blade for the node where you will also be able to access the connection strings. Use ```HTTPS (Access key 1)```.

## Step 1: Sending a private transaction from Alice to Carol

First, replace ```CAROL_PUBLIC_KEY``` in ```private-contract.js``` with Carol node's public key obtained from the preparatory step above.

Secondly, replace ```ALICE_NODE_PASSWORD``` in ```private-contract.js``` with Alice's node password (specified while creating Alice's Azure Blockchain Service instance).

Once that is done, send a private transaction from Alice to Carol. Within this code repository, do:

```
 --exec "loadScript('private-contract.js')" attach ALICE_CONNECTION_STRING
```

> The ```private-contract.js``` source is mostly similar to the one on 7nodes except for line 3 where we need to unlock the account used as it is locked by default on Azure Blockchain Service.

> CAROL_CONNECTION_STRING is in the form https://...

Executing the above, you should get the following:

```
Contract transaction send: TransactionHash: TRANSACTION_HASH waiting to be mined...
true
```
## Step 2: Check that Carol can see the value set by Alice

To check that Carol can see the value in the private transaction, [we use  to connect](https://docs.microsoft.com/en-us/azure/blockchain/service/connect-) to Carol and ask:

```
 attach CAROL_CONNECTION_STRING
```

> 

Once connected, obtain the ADDRESS of the contract that's in the transaction:

```
> eth.getTransactionReceipt("TRANSACTION_HASH");
{
  blockHash: "0xcfdff87dbf015d3d19e026322bd9b7a5cf5f271a00034d9ded4ec3ddd89c35e9",
  blockNumber: 1156847,
  contractAddress: "ADDRESS",
  cumulativeGasUsed: 0,
  from: "0xe1123522b2d75a616aa1ab7a61b03a6d60ca18b0",
  gasUsed: 0,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x1",
  to: null,
  transactionHash: "0x1147d30a1cb261a10a15af88a0d434f624fc76e06ef03c0a8964936d007d83a4",
  transactionIndex: 0
}
```

Once you have the ADDRESS, load up the contract and get the value set by Alice:

```
> var abi = [{"constant":true,"inputs":[],"name":"storedData","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"x","type":"uint256"}],"name":"set","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"retVal","type":"uint256"}],"payable":false,"type":"function"},{"inputs":[{"name":"initVal","type":"uint256"}],"type":"constructor"}];
> var private = eth.contract(abi).at("ADDRESS")
> private.get()
42
```
## Step 3: Check that Bob cannot see the value set by Alice

As the transaction is private only between Alice and Carol, Bob should not be able to see the value ```42``` set by Alice.

Connect to Bob via  with steps similar to above and repeat the following:

```
> var abi = [{"constant":true,"inputs":[],"name":"storedData","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"x","type":"uint256"}],"name":"set","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"retVal","type":"uint256"}],"payable":false,"type":"function"},{"inputs":[{"name":"initVal","type":"uint256"}],"type":"constructor"}];
> var private = eth.contract(abi).at("ADDRESS")
> private.get()
0
```

Note the value 0 returned vs 42 for Carol.

## Exploring further

### Updating the state of the contract

You can further try to change the value of the private contract between Alice and Carol by doing:

```
private.set(88, {from:eth.accounts[0],privateFor:["ALICE_PUBLIC_KEY||CAROL_PUBLIC_KEY"]});
```

Note that both Alice and Carol can continue changing the value as long as privateFor is set to the other party in the private contract. 

Neither Alice and Carol can change the ```privateFor``` above to Bob at this point without establishing a new contract.

### 