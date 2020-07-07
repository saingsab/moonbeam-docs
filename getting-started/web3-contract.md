---
title: Using Web3 for Contracts
description: Learn how to deploy Solidity-based contracts on Moonbeam with a simple script using web3.
---

#Setting Using Web3 to Deploy Smart Contracts on Moonbeam 
##Introduction  
This guide walks you through the process of using the solidity compiler and Web3 to deploy and interact with a Solidity-based smart contract on a Moonbeam dev node. Given Moonbeam’s Ethereum compatibility features, the Web3 library can be used directly with a Moonbeam node.

The examples on this guide are based on a Ubuntu 18.04 environment and assume that you have a local Moonbeam node running in --dev mode, you can find instructions to set up a local Moonbeam node [here](/getting-started/setting-up-a-node/).

!!! note 
    This tutorial was created using the pre-alpha release of [Moonbeam](https://github.com/PureStake/moonbeam/tree/moonbeam-tutorials). The Moonbeam platform, and the [Frontier](https://github.com/paritytech/frontier) components it relies on for Substrate-based Ethereum compatibility, are still under very active development. We have created this tutorial so you can test out Moonbeam’s Ethereum compatibility features. Even though we are still in development, we believe it’s important that interested community members and developers have the opportunity to start to try things with Moonbeam and provide feedback.

##Checking Prerequisites 

If you followed setting up a local Moonbeam node tutorial, you should have a local Moonbeam node producing blocks that looks like this:

![Moonbeam local node](/images/web3-contract-1.png)

In addition, for this tutorial, we need to install Node.js and the npm package manager. You can do this by running in your terminal:

```
sudo apt install nodejs
sudo apt install npm
```

We can verify if everything installed correctly by querying the version for each package:

```
node -v
npm -v
```

As of the writing of this guide, versions used were 8.10.0 and 3.5.2, respectively. Next, we can create a directory to store all our relevant files (in a separate path from the local Moonbeam node files), and create a simple package.json file by running:

```
mkdir incrementer
cd incrementer/
npm init --yes
```

With the package.json file created, we can then install both the Web3 and the Solidity compiler packages, by executing:

```
npm install --save web3
npm install --save solc
```

To verify the installed version of Web3 or the Solidity compiler you can use the ls command:

```
npm ls web3
npm ls solc
```

As of the writing of this guide, versions used were 1.2.9 and 0.6.10, respectively. Our setup for this example is going to be pretty simple. We are going to have the following files:

-  _Incrementer.sol_: the file with our Solidity code
-  _compile.js_: it will compile the contract with the Solidity compiler
-  _deploy.js_: it will handle the deployment to our local Moonbeam node
-  _get.js_: it will make a call to the node to get the current value of the number
-  _increment.js_: it will make a transaction to increment the number stored on the Moonbeam node
-  _reset.js_: the function to call that will reset the number stored to zero

##The Contract File 

The contract we will use is a very simple incrementer (arbitrarly named _Incrementer.sol_, and which you can find [here](/code-snippets/web3-contract/Incrementer.sol)), the Solidity code is the following:

```js
pragma solidity ^0.6.0;

contract Incrementer{
    uint public number ;

    constructor(uint _initialNumber) public {
        number = _initialNumber;
    }

    function increment(uint _value) public {
        number = number + _value;
    }

    function reset() public {
        number = 0;
    }
}
```

Our `constructor` function, that runs when the contract is deployed, sets the initial value of the number variable that is stored in the Moonbeam node (default is 0). The `increment` function adds `_value` provided to the current number, but a transaction needs to be sent as this modifies the stored data. And lastly, the `reset` function resets the stored value to zero.

!!! note
    This contract is just a simple example that does not handle values wrapping around, and it is only for illustration purposes.

##The Compile File

The only purpose of the _compile.js_ file (arbitrarily named, and which you can find [here](/code-snippets/web3-contract/compile.js)), is to use the Solidity compiler to output the bytecode and interface of our contract. First, we need to load the different modules that we will use for this process. The _path_ and _fs_ modules are included by default in Node.js (that is why we didn't have to install it before). Next, we have to read the content of the Solidity file (in UTF8 encoding). Then, we build the input object for the solidity compiler. And finally, we run the compiler and extract the data related to our incrementer contract, because for this simple example, is all we need.

```js
const path = require('path');
const fs = require('fs');
const solc = require('solc');

// Compile contract
const contractPath = path.resolve(__dirname, 'Incrementer.sol');
const source = fs.readFileSync(contractPath, 'utf8');
const input = {
   language: 'Solidity',
   sources: {
      'Incrementer.sol': {
         content: source,
      },
   },
   settings: {
      outputSelection: {
         '*': {
            '*': ['*'],
         },
      },
   },
};
const tempFile = JSON.parse(solc.compile(JSON.stringify(input)));
const contractFile = tempFile.contracts['Incrementer.sol']['Incrementer'];
module.exports = contractFile;
```

## The Deploy File

The deployment file (which you can find [here](/code-snippets/web3-contract/deploy.js)) is divided into two subsections: the initialization and the deploy contract. First, we need to load our web3 module and the export of the _compile.js_ file, from which we will extract the `bytecode` and `abi`. Next, define the privKey variable as the private key of our genesis account, where all the funds are stored when deploying your local Moonbeam node, and this is also used to sign the transactions. The address is needed to specify the form value of the transaction. And lastly, create a local web3 instance, where we set the provider to connect to our local Moonbeam node.

To deploy the contract, we create an asynchronous function to handle the transaction promises. First, we need to create a local instance of our contract using the `web3.eth.Contract(abi)`, from which we will call the deploy function. For this function, provide the `bytecode` and the arguments input of the constructor function, in our case was just one that was arbitrarily set to five. Then, to create the transaction, we use the `web3.eth.accounts.signTransaction(tx, privKey)` command, where we have to define the tx object with some parameters such as: from address, the encoded abi from the previous step, and the gas limit. The private key must be provided as well to sign the transaction.


```js
const Web3 = require('web3');
const contractFile = require('./compile');

// Initialization
const bytecode = contractFile.evm.bytecode.object;
const abi = contractFile.abi;
const privKey =
   '99B3C12287537E38C90A9219D4CB074A89A16E9CDB20BF85728EBD97C343E342'; // Genesis private key
const address = '0x6Be02d1d3665660d22FF9624b7BE0551ee1Ac91b';
const web3 = new Web3('http://localhost:9933');

// Deploy contract
const deploy = async () => {
   console.log('Attempting to deploy from account:', address);

   const incrementer = new web3.eth.Contract(abi);

   incrementerTx = incrementer.deploy({
      data: bytecode,
      arguments: [5],
   });

   const createTransaction = await web3.eth.accounts.signTransaction(
      {
         from: address,
         data: incrementerTx.encodeABI(),
         gas: '4294967295',
      },
      privKey
   );

   const createReceipt = await web3.eth.sendSignedTransaction(
      createTransaction.rawTransaction
   );
   console.log('Contract deployed at address', createReceipt.contractAddress);
};

deploy();
```

Note that the value "4294967295" for gas (referred to as the gas limit) needs to be manually set. As of the writing of this guide, we are working through some issues related to gas estimation in Moonbeam. Once these are fixed, this manual setting of the gas limit shouldn’t be necessary.

With the transaction message created and signed (you can `console.log(createTransaction)` to see the v-r-s values), we can now deploy it using the `web3.eth.sendSignedTransaction(signedTx)` by providing the `rawTransaction` from the createTransaction object. Lastly, we run our deploy function.

!!! note
    The _deploy.js_ script provides the contract address as an output. This comes handy as it is used for the contract interaction files.

##Files to Interact with the Contract

In this section, we will quickly go over the files that interact with our contract, either by making calls or sending transactions to it. First, let's overview the _get.js_ file (the simplest of them all, which you can find [here](/code-snippets/web3-contract/get.js)), that fetches the current value stored in the Moonbeam node. We need to load our web3 module and the export of the _compile.js_ file, from which we will extract the `abi`. Next, we define our address from which we are going to make the call to the contract and create a local web3 instance. And lastly, we need to provide the contract address (which is log in the console by the _deploy.js_ file).

The following step is to create a local instance of the contract by using the `web3.eth.Contract(abi)` command. Then, wrapped in an async function, we can write the contract call by running `web3.methods.myMethods()`, where we set the method or function that we want to call and provide the inputs for this call. This promise returns the data that we can log in the console. And lastly, we run our get function.

```js
const Web3 = require('web3');
const { abi } = require('./compile');

// Initialization
const address = '0x6Be02d1d3665660d22FF9624b7BE0551ee1Ac91b';
const web3 = new Web3('http://localhost:9933');
const contractAddress = '0xC2Bf5F29a4384b1aB0C063e1c666f02121B6084a';

// Contract Call
const incrementer = new web3.eth.Contract(abi, contractAddress);
const get = async () => {
   console.log(`Making a call to contract at address ${contractAddress}`);
   const data = await incrementer.methods
      .number()
      .call({ from: address, gas: '4294967295' });
   console.log(`The current number stored is: ${data}`);
};

get();
```

Let's now define the file to send a transaction that will add the value provided to our number. The _increment.js_ file (which you can find [here](/code-snippets/web3-contract/increment.js)) is somewhat different to the previous example, and that is because here we are modifying the stored data, and for this, we need to send a transaction that pays gas. However, the initialization part of the file is similar. The only differences are that the private key must be defined for signing and that we've defined a `_value` that corresponds to the value to be added to our number. The contract transaction starts by creating a local instance of the contract as before, but when we call the corresponding `incrementer(_value).encodedABI` method where we pass in `_value`. Then, as we did when deploying the contract, we need to create the transaction with the corresponding data (wrapped in a async function), sign it with the private key, and send it. Lastly, we run our incrementer function.

```js
const Web3 = require('web3');
const { abi } = require('./compile');

// Initialization
const privKey =
   '99B3C12287537E38C90A9219D4CB074A89A16E9CDB20BF85728EBD97C343E342'; // Genesis private key
const address = '0x6Be02d1d3665660d22FF9624b7BE0551ee1Ac91b';
const web3 = new Web3('http://localhost:9933');
const contractAddress = '0xC2Bf5F29a4384b1aB0C063e1c666f02121B6084a';
const _value = 3;

// Contract Tx
const incrementer = new web3.eth.Contract(abi);
const encoded = incrementer.methods.increment(_value).encodeABI();

const increment = async () => {
   console.log(
      `Calling the increment by ${_value} function in contract at address ${contractAddress}`
   );
   const createTransaction = await web3.eth.accounts.signTransaction(
      {
         from: address,
         data: encoded,
         gas: '4294967295',
         to: contractAddress,
      },
      privKey
   );

   const createReceipt = await web3.eth.sendSignedTransaction(
      createTransaction.rawTransaction
   );
   console.log(`Tx successfull with hash: ${createReceipt.transactionHash}`);
};

increment();
```

The _reset.js_ file (which you can find [here](/code-snippets/web3-contract/reset.js)), is almost identical to the previous example. The only difference is that we need to call the `reset()` method which takes no input.

```js
const Web3 = require('web3');
const { abi } = require('./compile');

// Initialization
const privKey =
   '99B3C12287537E38C90A9219D4CB074A89A16E9CDB20BF85728EBD97C343E342'; // Genesis private key
const address = '0x6Be02d1d3665660d22FF9624b7BE0551ee1Ac91b';
const web3 = new Web3('http://localhost:9933');
const contractAddress = '0xC2Bf5F29a4384b1aB0C063e1c666f02121B6084a';

// Contract Tx
const incrementer = new web3.eth.Contract(abi);
const encoded = incrementer.methods.reset().encodeABI();

const reset = async () => {
   console.log(
      `Calling the reset function in contract at address ${contractAddress}`
   );
   const createTransaction = await web3.eth.accounts.signTransaction(
      {
         from: address,
         data: encoded,
         gas: '4294967295',
         to: contractAddress,
      },
      privKey
   );

   const createReceipt = await web3.eth.sendSignedTransaction(
      createTransaction.rawTransaction
   );
   console.log(`Tx successfull with hash: ${createReceipt.transactionHash}`);
};

reset();
```

##Interacting with the Contract

With all the files ready we can proceed to deploy our contract the local Moonbeam node. To do this, we execute the following command in the directory where all the files are:

```
node deploy.js
```

After a successful deployment, you should get the following output:

![Moonbeam local node](/images/web3-contract-2.png)

First, let's check and confirm that that the value stored is equal to the one we passed in as the input of the constructor function (that was 5), we do this by running:

```
node get.js
```

With the following output:

![Moonbeam local node](/images/web3-contract-3.png)

Then, we can use our incrementer file, remember that `_value = 3`. We can immediately use our getter file to prompt the value after the transaction:

```
node incrementer.js
node get.js
```

With the following output:

![Moonbeam local node](/images/web3-contract-4.png)


Lastly, we can reset our number by using the reset file:

```
node reset.js
node get.js
```

With the following output:

![Moonbeam local node](/images/web3-contract-5.png)

##We Want to Hear From You
This example provides context on how you can start working with Moonbeam and how you can try out its Ethereum compatibility features such as the Web3 library. We are interested in hearing about your experience following the steps in this guide or your experience trying other Ethereum-based tools with Moonbeam. Feel free to join us in the [Moonbeam Riot room here](https://matrix.to/#/!dzULkAiPePEaverEEP:matrix.org?via=matrix.org&via=web3.foundation). We would love to hear your feedback on Moonbeam and answer any questions that you have.