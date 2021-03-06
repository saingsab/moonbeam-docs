---
title: Using Truffle
description: Learn how to deploy a Solidity-based smart contract to a Moonbeam node using Truffle.
---

#Interacting with Moonbeam Using Truffle
<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed//RD5MefSPNeo' frameborder='0' allowfullscreen></iframe></div>
<style>.caption { font-family: Open Sans, sans-serif; font-size: 0.9em; color: rgba(170, 170, 170, 1); font-style: italic; letter-spacing: 0px; position: relative;}</style><div class='caption'>You can find all of the relevant code for this tutorial on the [code snippets page](/resources/code-snippets/)</div>

##Introduction
This guide walks through the process of deploying a Solidity-based smart contract to a Moonbeam node using [Truffle](https://www.trufflesuite.com/).  Truffle is one of the commonly used development tools for smart contracts on Ethereum.  Given Moonbeam’s Ethereum compatibility features, Truffle can be used directly with a Moonbeam node.

!!! note
    This tutorial was created using the pre-alpha release of [Moonbeam](https://github.com/PureStake/moonbeam/tree/moonbeam-tutorials). The Moonbeam platform, and the [Frontier](https://github.com/paritytech/frontier) components it relies on for Substrate-based Ethereum compatibility, are still under very active development.  We have created this tutorial so you can test out Moonbeam’s Ethereum compatibility features.  Even though we are still in development, we believe it’s important that interested community members and developers have the opportunity to start to try things with Moonbeam and provide feedback.

This guide is based on an Ubuntu 18.04 installation and assumes that you have a running local Moonbeam node running in `--dev` mode.  You can find instructions for running a local Moonbeam node [here](/getting-started/setting-up-a-node/).

##Checking Prerequisites and Setting Up Truffle  
If you followed the previous guides, you should have a local Moonbeam node producing blocks that looks like this:

![Local Moonbeam node that's producing blocks](/images/using-truffle-1.png)

In addition, for this tutorial, we need to install Node.js (we'll go for v14.x) and the npm package manager. You can do this by running in your terminal:

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

As of the writing of this guide, versions used were 14.6.0 and 6.14.6, respectively.

Next, navigate to the folder where you built Moonbeam and go into the `tools/truffle` directory.  In our case, this is `/home/purestake/moonbeam/tools/truffle`, but replace this with the correct path for your environment.  This directory has a Truffle configuration that is designed to work with a locally running Moonbeam `--dev` node.

Let’s take a look at the `truffle-config.js` file:

``` javascript
const PrivateKeyProvider = require ('./private-provider')
var privateKey = "99B3C12287537E38C90A9219D4CB074A89A16E9CDB20BF85728EBD97C343E342";

module.exports = {
  networks: {
    development: {
      provider: () => new PrivateKeyProvider(privateKey, "http://localhost:9933/", 43),
      network_id: 43
    },
    live: {
      provider: () => new PrivateKeyProvider(privateKey, "http://35.203.125.209:9933/", 43),
      network_id: 43
    },
    ganache: {
      provider: () => new PrivateKeyProvider(privateKey, "http://localhost:8545/", 43),
      network_id: 43
    }
  }
}
```

!!! note
    We are using a `PrivateKeyProvider` as our Web3 provider (instantiation included in `private-provider.js`).  This provider is being set up in a very specific way to work around some issues we are currently working on related to `chainid`, `nonce handling`, and the `skipCache: true` setting when using the default Truffle-provided Web3 provider with Moonbeam nodes.  We also are using the same private key that we have been using in other guides, which comes pre-funded with tokens via the genesis config of a Moonbeam node running in `--dev` mode.  The public key for this account is: 0x6Be02d1d3665660d22FF9624b7BE0551ee1Ac91b.

The contract we will be deploying with Truffle is a simple ERC-20 contract.  You can find this contract under `moonbeam/tools/truffle/contracts/MyToken.sol`.  The content of this file is:

```solidity
pragma solidity ^0.5.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20Detailed.sol";

// This ERC-20 contract mints the specified amount of tokens to the contract creator.
contract MyToken is ERC20, ERC20Detailed {
    constructor(uint256 initialSupply)
        public
        ERC20Detailed("MyToken", "MYTOK", 18)
    {
        _mint(msg.sender, initialSupply);
    }
}
```

This is a simple ERC-20 contract based on the OpenZepplin ERC-20 contract that creates MyToken and assigns the created initial token supply to the contract creator.

If we take a look at the Truffle migration script under `migrations/2_deploy_contracts.js`, it contains the following:

```javascript
var MyToken = artifacts.require("MyToken");

module.exports = function (deployer) {
  // deployment steps
  deployer.deploy(MyToken, "8000000000000000000000000", { gas: 4294967295 });
};
```

"8000000000000000000000000" is the number of tokens to initially mint with the contract, that is, 8 million with 18 decimal places.

!!! note
    We are specifying the gas to send with the contract deployment transaction.  This is needed as we are still working on some of the gas estimation functionality in Moonbeam.

The last setup step is to install all of the dependencies.  Do this by running the following command in the `moonbeam/tools/truffle` folder: 

```
npm install
```

![Running the npm install command](/images/using-truffle-2.png)

As the installation proceeds, you may see errors related to the compilation of keccak, which can be safely ignored for purposes of this walkthrough.

At the end of the process you should see a note about the number of packages which have been added:

![Confirmation message with number of packages added](/images/using-truffle-3.png)

##Deploying a Contract to Moonbeam Using Truffle  
Before we can deploy our contracts, let's compile them. You can do this with the following command:

```
node_modules/.bin/truffle compile
```

If successful, you should see output like the following:

![Truffle compile success message](/images/using-truffle-4.png)

Now we are ready to deploy the compiled contracts.  You can do this with the following command:

```
node_modules/.bin/truffle migrate --network development
```

If successful, you will see deployment actions including the address of the deployed contract:

![Successful contract deployment actions](/images/using-truffle-5.png)

Once you have followed the [MetaMask guide](/getting-started/using-metamask/) and [Remix guide](/getting-started/using-remix/), you will be able to take the deployed contract address that is returned and load it into MetaMask or Remix.

##We Want to Hear From You
This is obviously a simple example, but it provides context for how you can start working with Moonbeam and how you can try out its Ethereum compatibility features.  We are interested in hearing about your experience following the steps in this guide or your experience trying other Ethereum-based tools with Moonbeam.  Feel free to join us in the [Moonbeam Discord here](https://discord.gg/PfpUATX).  We would love to hear your feedback on Moonbeam and answer any questions that you have.  
