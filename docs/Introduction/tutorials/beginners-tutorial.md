---
layout: nodes.liquid
section: smartContract
date: Last Modified
title: "The Basics: Using Data Feeds"
permalink: "docs/beginners-tutorial/"
excerpt: "Smart Contracts and Chainlink"
whatsnext: {"Get the Latest Price":"/docs/get-the-latest-price/", "Deploy your first contract":"/docs/deploy-your-first-contract/", "Random Numbers Tutorial":"/docs/intermediates-tutorial/"}
metadata:
  title: "The Basics Tutorial"
  description: "Learn what smart contracts are, how to write them, and how to use Chainlink data feeds to deploy your very own Chainlink smart contract."
  image:
    0: "/files/1a63254-link.png"
---

> 👍 New to smart contracts?
>
> If you're new to smart contract development this is a great place to start. We'll walk you through developing your first smart contract that interacts with Chainlink.

<p>
  https://www.youtube.com/watch?v=rFXSEEQG9YE
</p>


# Overview

In this tutorial, you will write and deploy a Chainlink smart contract to an Ethereum testnet.

**Table of Contents**

1. [What are smart contracts? What are data feeds?](#q1)
2. [What language is a smart contract written in?](#q2)
3. [What does a smart contract look like?](#q3)
  a. [Define the version `pragma solidity ...`](#q3a)
  b. [Start your contract `contract ... {}`] (#q3b)
  c. [State variables `string public ...`] (#q3c)
  d. [The `constructor`] (#q3d)
  e. [Using functions `function name(type paramName) ...`] (#q3e)
4. [What does "deploying" mean?] (#q4)
5. [What are oracles? Why are they important?] (#q5)
6. [How do smart contracts use oracles?] (#q6)
  a. [Information about interfaces] (#q6a)
  b. [Using Chainlink data feeds] (#q6b)
7. [How do I deploy to testnet?] (#q7)
  a. [The Remix IDE] (#q7a)
  b. [Metamask wallet] (#q7b)
  c. [Obtaining testnet ETH] (#q7c)
  d. [Compiling your smart contract] (#q7d)
  e. [Deploying] (#q7e)
  f. [Get the price] (#q7f)

  
# 1. What are smart contracts? What are data feeds? <a name="q1"></a>

When deployed to a blockchain, a **smart contract** is a set of instructions that can be executed without intervention from third parties. The code of a smart contract determines how it responds to input, just like the code of any other computer program.

A valuable feature of smart contracts is that they can store and manage on-chain assets (like ETH or ERC20 tokens), just like you or I can with an Ethereum wallet. Because they have an on-chain address, like a wallet, they can do everything any other address can. This opens the door for programming automated actions when receiving and transferring assets.

Smart contracts can connect to real-world market prices of assets to produce powerful applications. Chainlink's **[Data Feeds](../using-chainlink-reference-contracts/)** feature allows users to quickly and securely connect smart contracts to such assets in a single call. You can read more about the applications of data feeds [here](/docs/other-tutorials/#data-feeds-tutorials).

# 2. What language is a smart contract written in? <a name="q2"></a>

The most popular language for writing smart contracts on Ethereum is [Solidity](https://docs.soliditylang.org/en/v0.8.7/?target=_blank). It was created by the Ethereum Foundation specifically for smart contract development and is constantly being updated.

If you've ever written Javascript, Java, or other object-oriented scripting languages, Solidity should be easy to understand. Similar to object-oriented langauges, Solidity is considered to be a *contract*-oriented language.

# 3. What does a smart contract look like? <a name="q3"></a>

The structure of a smart contract is similar to that of a _class_ in Javascript, with a few differences. Let's take a look at this `HelloWorld` example.

```solidity
pragma solidity 0.8.7;

contract HelloWorld {
    string public message;

    constructor(string memory initialMessage) {
        message = initialMessage;
    }

    function updateMessage(string memory newMessage) public {
        message = newMessage;
    }
}
```

## 3a. Define the version `pragma solidity ...` <a name="q3a"></a>

The first thing that every solidity file must have is the Solidity version definition. The version HelloWorld.sol is using is 0.8.7, defined by `pragma solidity 0.8.7;`

You can see the latest versions of the Solidity compiler [here](https://github.com/ethereum/solc-bin/blob/gh-pages/bin/list.txt/?target=_blank). You may also notice Solidity files containing definitions with multiple versions of Solidity:

```solidity
pragma solidity >=0.7.0 <0.9.0;
```
This means that the code is written for Solidity version 0.7.0, or a newer version of the language up to, but not including version 0.9.0. In short, `pragma` is used to instruct the compiler as how to treat the code.

## 3b. Start your contract `contract ... {}` <a name="q3b"></a>

Next, the `HelloWorld` contract is defined by using the keyword `contract`. Think of this as being similar to declaring a `class` in Javascript. The implementation of `HelloWorld` is inside this definition, denoted with curly braces.

```solidity
pragma solidity 0.8.7;

contract HelloWorld {

}
```

## 3c. State variables `string public ...` <a name="q3c"></a>

Again, like Javascript, contracts can have state variables and local variables. *State variables* are variables with values that are permanently stored in contract storage. The values of *local variables*, however, are present only until the function is executing. There are also different *types* of variables you can use within Solidity, such as `string`, `uint256`, etc. Check out the [Solidity documentation](https://docs.soliditylang.org/en/v0.8.7/) to learn more about the different kinds of variables and types.

_Modifiers_ are used to change the level of access to these variables. Here are some examples of state variables with different modifiers:

```solidity
string public message;
uint256 internal internalVar;
uint8 private privateVar;
bool external isTrue;
```

## 3d. The `constructor` <a name="q3d"></a>

Another familiar concept to programmers is the constructor. It is called upon deploying the contract, so as to set the state of the contract once created.

In `HelloWorld`, the constructor takes in a `string` as a parameter and sets the `message` state variable to that string.

```solidity
pragma solidity 0.8.7;

contract HelloWorld {
  constructor(string memory initialMessage) {
        message = initialMessage;
  }
}
```

## 3e. Using functions `function name(type paramName) ...` <a name="q3e"></a>

Functions are used to access and modify the state of the contract, and call functions on external contracts. `HelloWorld` has a function called `updateMessage`, which updates the current message stored in the state.

```solidity
pragma solidity 0.8.7;

contract HelloWorld {
  constructor(string memory initialMessage) {
        message = initialMessage;
  }

  function updateMessage(string memory newMessage) public {
        message = newMessage;
    }
}
```

# 4. What does "deploying" mean? <a name="q4"></a>

Deploying a smart contract is the process of pushing the code to the blockchain, at which point it resides with an on-chain address. Once it's deployed, the code cannot be changed and is said to be *immutable*.

As long as the address is known, its functions can be called through an interface, on [Etherscan](https://etherscan.io/?target=_blank), or through a library like [web3js](https://web3js.readthedocs.io/en/v1.3.0/?target=_blank), [web3py](https://web3py.readthedocs.io/?target=_blank), [ethers](https://docs.ethers.io/v5/?target=_blank), and more. Contracts can also be written to interact with other contracts on the blockchain.

# 5. What are oracles? Why are they important? <a name="q5"></a>

Oracles provide a bridge between the real-world and on-chain smart contracts, by being a source of data that smart contracts can rely on, and act upon.

Oracles play an extremely important role in facilitating the full potential of smart contract utility. Without a reliable connection to real-world conditions, smart contracts are unable to effectively serve the real-world.

# 6. How do smart contracts use oracles? <a name="q6"></a>

Oracles are most popularly used with [Data Feeds](../using-chainlink-reference-contracts/) . DeFi platforms like [AAVE](https://aave.com/?target=_blank) and [Synthetix](https://www.synthetix.io/?target=_blank) use Chainlink data feed oracles to obtain accurate real-time asset prices in their smart contracts.

Chainlink data feeds are sources of data [aggregated from many independent Chainlink node operators](../architecture-decentralized-model/). Each data feed has an on-chain address and functions that enable contracts to read from that address. For example, the [ETH / USD feed](https://feeds.chain.link/eth-usd/?target=_blank).

![Chainlink Feeds List](/images/contract-devs/price-aggr.png)

## 6a. Information about interfaces <a name="q6a"></a>
Before we go into using data feeds, it's important to know how interfaces work in Solidity. An _interface_ is another concept that will be familiar to programmers of other languages. 

Interfaces define functions without their implementation, leaving inheriting contracts to define the actual implementation themselves. This make it easier to know what functions to call in a contract. Here's an example of an interface:

```solidity
pragma solidity 0.8.7;

interface numberComparison {
   function isSameNum(uint a, uint b) external view returns(bool);
}

contract Test is numberComparison {
    
   constructor() public {}
   
   function isSameNum(uint a, uint b) external view returns(bool){
      if (a == b) {
        return true;
      } else {
        return false;
      }
   }
}
```

## 6b. Using Chainlink data feeds <a name="q6b"></a>

The following code is from the [Get the Latest Price](../get-the-latest-price/) page. It describes a contract which obtains the latest ETH / USD price using the Kovan testnet.

```solidity
{% include samples/PriceFeeds/PriceConsumerV3.sol %}
```

Notice how the code imports an interface called `AggregatorV3Interface`. In this case, `AggregatorV3Interface` defines that all V3 Aggregators will have the function `latestRoundData`. We can see all of the functions that a V3 Aggregator exposes in the[`AggregatorV3Interface` file on Github](https://github.com/smartcontractkit/chainlink/blob/master/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol/?target=_blank).

Our contract is initialized with the hard-coded address of the Kovan data feed for ETH / USD prices. Then in `getLatestPrice` it uses `latestRoundData` to obtain the most recent round of price data. We're interested in the price, so the function returns that.

# 7. How do I deploy to testnet? <a name="q7"></a>

There are a few things that are needed to deploy a contract to a testnet:

- [x] Smart contract code
- [ ] A Solidity compiler
- [ ] An address to deploy from
- [ ] Some ETH (in our case, testnet ETH, which is free)

We have the code. What we need next is a compiler.

## 7a. The Remix IDE <a name="q7a"></a>

[Remix](https://remix.ethereum.org/?target=_blank) is an online IDE which enables anyone to write, compile and deploy smart contracts from the browser.

Fortunately for us, Remix also has support for gist. This means that Remix can load code from Github, and in this case, `PriceConsumerV3.sol` Click the button below to open a new tab, then once Remix has loaded, find the `gists` folder in the File Explorer on the left-hand side, and click on the file to open the code in the editor.

<div class="remix-callout">
  <a href="https://remix.ethereum.org/#url=https://docs.chain.link/samples/PriceFeeds/PriceConsumerV3.sol" target="_blank" class="cl-button--ghost">Deploy this contract using Remix ↗</a>
</div>

![Remix Select PriceConsumerV3.sol](/files/11d7052-Screenshot_2020-11-27_at_10.16.47.png)

Get familiar with the layout of Remix and play around with the contract. This is what we'll use for the compiler.

- [x] Smart contract code
- [x] A Solidity compiler
- [ ] An address to deploy from
- [ ] Some ETH

Now we need an address to deploy from.

## 7b. Metamask wallet <a name="q7b"></a>

Contracts are deployed by addresses on the network, so to deploy our own we need an address. Not only that, but we need one which we can easily use with Remix. Fortunately, Metamask is just what is needed. Metamask allows anyone to create an address, store funds and interact with Ethereum compatible blockchains from a browser extension.

Head to the [Metamask website](https://metamask.io/?target=_blank) to download, install and create an account.

Once that's done, navigate to the Kovan testnet inside Metamask extension, as seen in the image below.

![Metamask Select Kovan Screen](/files/de9b81c-kovan.gif)

We now have an address to deploy to the Kovan testnet from.

- [x] Smart contract code
- [x] A Solidity compiler
- [x] An address to deploy from
- [ ] Some ETH

We'll finally need to obtain ETH to deploy our smart contract to a testnet.

## 7c. Obtaining testnet ETH <a name="q7c"></a>

Making transactions on Ethereum blockchains are not free, they cost ETH. Deploying a contract is no exception to this rule. Fortunately, testnet ETH doesn't actually cost anything, since the purpose of testnets is to test smart contracts publically before they are deployed to the mainnet.

Connect your Metamask wallet and request ETH from one of the available faucets on [LINK Token Contracts page](../link-token-contracts/).

- [x] Smart contract code
- [x] A Solidity compiler
- [x] An address to deploy from
- [X] Some ETH

Great, we have all the necessary components to compile and deploy our smart contract!

## 7d. Compiling your smart contract <a name="q7d"></a>

We have all the pieces needed to deploy our price consumer to Kovan. To start the process we need to compile it first. Head back to the Remix tab.

Under the logo in the top left-hand corner, there's a vertical menu, made up of icons. Hovering over each button shows a tooltip explaining what each item is. The first is "File explorer", which shows us all the files loaded into Remix. The second is "Solidity compiler". Click this item to open up a side menu where we can compile our contract.

Remix should automatically detect the correct compiler version depending on the version specified in the contract, and you should see a button that looks like this:

![Remix Click Complile PriceConsumerV3.sol](/files/99af570-Screenshot_2020-11-27_at_10.45.44.png)

Click it, and you will see some details below it by scrolling down. There might be a few yellow warnings, but don't worry about those for now. If the warnings are red, revisit your code and troubleshoot any bugs or errors which may be present.

## 7e. Deploying <a name="q7e"></a>

Looking at the icon menu on the far-left again. Below the compiler icon should be "Deploy & run transactions". Click on that.

This screen might seem a little more intimidating, but do not fret. This is where we hook up Metamask to Remix so that it knows which account to deploy from.

In the first dropdown, named _ENVIRONMENT_, the selected value should currently be "Javascript VM". Instead, select "Injected Web3". This should trigger a Metamask notification asking for permission to connect. Accept it, and your address should be automatically loaded into the _ACCOUNT_ dropdown below _ENVIRONMENT_.

Once that's done, check that the _CONTRACT_ dropdown shows the name of our contract, then click "Deploy". Another Metamask notification will pop up asking for permission, and detailing how much GAS it will cost in testnet ETH. Confirm the transaction and await confirmation! This may take a few seconds depending on the network, so be patient.

## 7f. Get the price <a name="q7f"></a>

Once your smart contract is deployed, an item will appear in the "Deployed Contracts" section underneath the "Deploy" button. This is the deployed contract with all its address.

![Remix Deployed Contracts Section](/files/ca77c39-Screenshot_2020-11-27_at_10.56.56.png)

Click on the caret to see a list of all the functions available to call.

Click "getLatestPrice", and voilà! The latest price appears just underneath the button. We have successfully deployed a smart contract, which uses Chainlink data feeds, to the Kovan Ethereum testnet!
