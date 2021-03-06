---
title: Precompiled Contracts
description:  Learn about precompile contracts on Moonbase Alpha, the Moonbeam Network TestNet for its Ethereum-compatible chain. 
---

# Precompiled Contracts on Moonbase Alpha

## Introduction

Another feature added with the [release of Moonbase Alpha v2](http://www.purestake.com/blog/new-in-moonbase-alpha-v2-contract-events-and-pub-sub-capabilities/), is the inclusion of some of the [precompiled contracts](https://docs.klaytn.com/smart-contract/precompiled-contracts) that are natively available on Ethereum. Currently, the first four precompiles are included, which are: ecrecover, sha256, ripemd-160 and the identity function.

In this guide, we will show how to use and/or verify these four precompiles.

## Checking Prerequisites

For this tutorial, we need to install Node.js (we'll go for v14.x) and the npm package manager. You can do this by running in your terminal:

```
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
```
```
sudo apt install -y nodejs
```

We can verify that everything installed correctly by querying the version for each package:

```
node -v
```
```
npm -v
```

As of the writing of this guide, versions used were 14.6.0 and 6.14.6, respectively. Also, we need to install the Web3 package by executing:

```
npm install --save web3
```

To verify the installed version of Web3, you can use the `ls` command:

```
npm ls web3
```

As of the writing of this guide, the version used was 1.3.0. 

## Verify Signatures with ECRECOVER

The main function of this precompile is to verify the signature of a message. In general terms, you feed `ecrecover` the transaction's signature values, and it returns an address. The signature is verified if the address returned is the same as the public address who sent the transaction.

Let's jump into a small example to showcase how we can leverage this precompiled function. To do so we need to retrieve the transaction's signature values (v, r, s). To do so, we'll sign and retrieved the signed message were these values are as well:

```js
const Web3 = require('web3');

// Provider
const web3 = new Web3('https://rpc.testnet.moonbeam.network');

// Address and Private Key
const address = '0x6Be02d1d3665660d22FF9624b7BE0551ee1Ac91b';
const pk1 = '99B3C12287537E38C90A9219D4CB074A89A16E9CDB20BF85728EBD97C343E342';
const msg = web3.utils.sha3('supercalifragilisticexpialidocious');

async function signMessage(pk) {
   try {
   // Sign and get Signed Message
      const smsg = await web3.eth.accounts.sign(msg, pk);
      console.log(smsg);
   } catch (error) {
      console.error(error);
   }
}

signMessage(pk1);
```

This code will return the following object in the terminal:

```js
{
  message: '0xc2ae6711c7a897c75140343cde1cbdba96ebbd756f5914fde5c12fadf002ec97',
  messageHash: '0xc51dac836bc7841a01c4b631fa620904fc8724d7f9f1d3c420f0e02adf229d50',
  v: '0x1b',
  r: '0x44287513919034a471a7dc2b2ed121f95984ae23b20f9637ba8dff471b6719ef',
  s: '0x7d7dc30309a3baffbfd9342b97d0e804092c0aeb5821319aa732bc09146eafb4',
  signature: '0x44287513919034a471a7dc2b2ed121f95984ae23b20f9637ba8dff471b6719ef7d7dc30309a3baffbfd9342b97d0e804092c0aeb5821319aa732bc09146eafb41b'
}
```
With the necessary values, we can go to Remix to test the precompiled contract. Note that this can be verified as well with the Web3 JS library, but in our case, we'll go to Remix to be sure that this is using the precompiled contract on the blockchain. The Solidity code we can use to verify the signature is the following:

```solidity
pragma solidity ^0.7.0;

contract ECRECOVER{
    address addressTest = 0x12Cb274aAD8251C875c0bf6872b67d9983E53fDd;
    bytes32 msgHash = 0xc51dac836bc7841a01c4b631fa620904fc8724d7f9f1d3c420f0e02adf229d50;
    uint8 v = 0x1b;
    bytes32 r = 0x44287513919034a471a7dc2b2ed121f95984ae23b20f9637ba8dff471b6719ef;
    bytes32 s = 0x7d7dc30309a3baffbfd9342b97d0e804092c0aeb5821319aa732bc09146eafb4;
    
    
    function verify() public view returns(bool) {
        // Use ECRECOVER to verify address
        return (ecrecover(msgHash, v, r, s) == (addressTest));
    }
}
```

Using the [Remix compiler and deployment](/getting-started/local-node/using-remix), and with [MetaMask pointing to Moonbase Alpha](/getting-started/testnet/metamask), we can deploy the contract and call the `verify()` method that returns _true_ if the address returned by `ecrecover` is equal to the address used to signed the message (related to the private key and needs to be manually set in the contract).

## Hashing with SHA256

This hashing function returns the SHA256 hash from the given data. To test this precompile, you can use this [online tool](https://md5calc.com/hash/sha256) to calculate the SHA256 hash of any string you want, in our case we'll do so with `Hello World!`. We can head directly to Remix and deploy the following code, where the calculated hash is set for the `expectedHash` variable:

```solidity
pragma solidity ^0.7.0;

contract Hash256{
    bytes32 public expectedHash = 0x7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069;

    function calculateHash() internal pure returns (bytes32) {
        string memory word = 'Hello World!';
        bytes32 hash = sha256(bytes (word));
        
        return hash;        
    }
    
    function checkHash() public view returns(bool) {
        return (calculateHash() == expectedHash);
    }
}

```
Once the contract is deployed, we can call the `checkHash()` method that returns _true_ if the hash returned by `calculateHash()` is equal to the hash provided.

## Hashing with RIPEMD-160

This hashing function returns a RIPEMD-160 hash from the given data. To test this precompile, you can use this [online tool](https://md5calc.com/hash/ripemd160) to calculate the RIPEMD-160 hash of any string you want, in our case we'll do so again with `Hello World!`. We'll reuse the same code as before but using the `ripemd160` function, note that it returns a `bytes20` type variable:

```solidity
pragma solidity ^0.7.0;

contract HashRipmd160{
    bytes20 public expectedHash = hex'8476ee4631b9b30ac2754b0ee0c47e161d3f724c';

    function calculateHash() internal pure returns (bytes20) {
        string memory word = 'Hello World!';
        bytes20 hash = ripemd160(bytes (word));
        
        return hash;        
    }
    
    function checkHash() public view returns(bool) {
        return (calculateHash() == expectedHash);
    }
}
```
With the contract deployed, we can call the `checkHash()` method that returns _true_ if the hash returned by `calculateHash()` is equal to the hash provided.

## The Identity Function

Also known as datacopy, his function serves as a cheaper way to copy data in memory. The Solidity compiler does not support it, so it needs to be called with inline assembly. The [following code](https://docs.klaytn.com/smart-contract/precompiled-contracts#address-0x-04-datacopy-data) (adapted to Solidity), can be used to call this precompiled contract. We can use this [online tool](https://web3-type-converter.brn.sh/) to get the bytes from any string, as this is the input of the method `callDataCopy()`.

```solidity
pragma solidity ^0.7.0;

contract Identity{
    
    bytes public memoryStored;

    function callDatacopy(bytes memory data) public returns (bytes memory) {
    bytes memory result = new bytes(data.length);
    assembly {
        let len := mload(data)
        if iszero(call(gas(), 0x04, 0, add(data, 0x20), len, add(result,0x20), len)) {
            invalid()
        }
    }
    
    memoryStored = result;

    return result;
    }
}
```
With the contract deployed, we can call the `callDataCopy()` method that and verify if `memoryStored` checks with the bytes that you pass in as an input of the function.

## We Want to Hear From You

If you have any feedback regarding Moonbase Alpha, the precompile contracts, or any other Moonbeam related topic, feel free to reach out through our official development [Discord channel](https://discord.gg/PfpUATX).



