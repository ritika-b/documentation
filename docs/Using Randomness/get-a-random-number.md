---
layout: nodes.liquid
section: smartContract
date: Last Modified
title: "Get a Random Number"
permalink: "docs/get-a-random-number/"
whatsnext: {"Contract Addresses":"/docs/vrf-contracts/"}
metadata: 
  description: "How to generate a random number inside a smart contract using Chainlink VRF."
  image: 
    0: "/files/OpenGraph_V3.png"
---
This page explains how to get a random number inside a smart contract using Chainlink VRF.

Chainlink VRF is subscription-based. The subscription owner manages the subscription LINK balance as well as the set of 
addresses (consumers) that are allowed to use that balance for VRF requests. The requests follow the 
[Request & Receive Data](../../request-and-receive-data/) cycle. Upon fulfillment, the gas used to fulfill the request is calculated, converted
to link using an ETH/LINK feed, and charged to the subscription including a flat per request fee. To learn more about the fee, see `fulfillmentFlatFeeLinkPPM` in the [Coordinator Parameters](#coordinator-parameters) list. 

# Static Parameters 
These parameters are the same for all VRF users. You can find the respective values for your network in the [VRF Contracts page](../vrf-contracts).
- `address link` - LINK token address on the corresponding network (Ethereum, Polygon, BSC, etc).
- `address vrfCoordinator` - Address of the Chainlink VRF Coordinator.
- `bytes32 keyHash` - Hash of the public key used to verify the VRF proof. It functions as an ID of the offchain VRF job to be run in response to requests. 
It is very important this is correct, if not, your request will not be processed. 

# User Parameters
These parameters are configurable based on the users needs. 
- `uint16 requestConfirmations` - How many confirmations you'd like the Chainlink node to wait before responding. The longer the node waits the more secure
the random value is. It must be greater than the coordinators `minimumRequestBlockConfirmations`.
- `uint32 callbackGasLimit` - How much gas you would like in your callback to do work with the random words provided. It must be less than the coordinators `maxGasLimit`.
- `uint16 numWords` - How many random words you would like in your request. If you are able to make use of several random words in the same callback,
you can achieve significant gas savings. 

# Coordinator Parameters
These parameters are configured by the coordinator owner, which currently is Chainlink itself until the threshold VRF is released. You can view their values by running `getConfig` on the coordinator contract (see [VRF Contracts](../vrf-contracts) for the coordinator address).
- `uint16 minimumRequestBlockConfirmations` - A minimum number of confirmation blocks on VRF requests before oracles should respond. 
- `uint32 fulfillmentFlatFeeLinkPPM`- The charge per request on top of the gas fees. Its flat fee specified in millionths of LINK.
- `uint32 maxGasLimit` - The maximum gas limit supported for a fulfillRandomWords callback.
- `uint32 stalenessSeconds` - How long we wait until we consider the ETH/LINK price (used for converting gas costs to LINK) is stale and use `fallbackWeiPerUnitLink`
- `uint32 gasAfterPaymentCalculation` - How much gas is used outside of the payment calculation. Specifically its the gas outside of here (TODO link) and here (TODO link).
- `uint96 minimumSubscriptionBalance` - The minimum subscription balance required to make a request. Its set to be about 300% of the cost of a single request to handle
fluctuations in ETH/LINK price between request and fulfillment time.
- `uint256 MAXIMUM_CONSUMERS` - A fixed maximum of 100 consumers per subscription. Use multiple separate subscriptions if you need more.

# Example Configurations

How you manage the subscription is up to you and depends on your randomness needs. Here are a few example configurations.

## Single consumer/subscription owner
A simple example subscription with only one consumer who is also the subscription owner. It also sets the 
request config to be static, so each request uses the same parameters.

```solidity
pragma solidity ^0.8.0;

import "path/to/LinkTokenInterface.sol";
import "path/to/VRFCoordinatorV2Interface.sol";
import "path/to/VRFConsumerBaseV2.sol";

contract VRFSingleConsumerExample is VRFConsumerBaseV2 {

    VRFCoordinatorV2Interface COORDINATOR;
    LinkTokenInterface LINKTOKEN;

    struct RequestConfig {
        uint64 subId;
        uint32 callbackGasLimit;
        uint16 requestConfirmations;
        uint32 numWords;
        bytes32 keyHash;
    }
    RequestConfig s_requestConfig;
    uint256[] s_randomWords;
    uint256 s_requestId;

    constructor(
        address vrfCoordinator,
        address link,
        uint32 callbackGasLimit,
        uint16 requestConfirmations,
        uint32 numWords,
        bytes32 keyHash
    )
    VRFConsumerBaseV2(vrfCoordinator)
    {
        COORDINATOR = VRFCoordinatorV2Interface(vrfCoordinator);
        LINKTOKEN = LinkTokenInterface(link);
        s_requestConfig = RequestConfig({
            subId: 0, // Unset
            callbackGasLimit: callbackGasLimit,
            requestConfirmations: requestConfirmations,
            numWords: numWords,
            keyHash: keyHash
        });
    }

    function fulfillRandomWords(
        uint256 requestId,
        uint256[] memory randomWords
    )
        internal
        override
    {
        s_randomWords = randomWords;
    }

    function requestRandomWords()
        external
    {
        RequestConfig memory rc = s_requestConfig;
        // Will revert if subscription is not set and funded.
        s_requestId = COORDINATOR.requestRandomWords(
            rc.keyHash,
            rc.subId,
            rc.requestConfirmations,
            rc.callbackGasLimit,
            rc.numWords);
    }

    // Assumes this contract owns link
    function topUpSubscription(
        uint256 amount
    )
        external
    {
        LINKTOKEN.transferAndCall(
            address(COORDINATOR),
            amount,
            abi.encode(s_requestConfig.subId));
    }

    function unsubscribe()
        external
    {
        // Returns funds to this address
        COORDINATOR.cancelSubscription(s_requestConfig.subId, address(this));
        s_requestConfig.subId = 0;
    }

    function subscribe()
        external
    {
        address[] memory consumers = new address[](1);
        consumers[0] = address(this);
        s_requestConfig.subId = COORDINATOR.createSubscription(consumers);
    }
}
``` 


## Multiple consumers, external subscription owner

In this example, the subscription for multiple consumers is managed by an external account. Steps to set up this configuration:

1. Create a subscription with no consumers `createSubscription()`. Make a note of the subscriptionId emitted in the `SubscriptionCreated` log. TODO: metamask screen shots
2. Deploy your applications which expose a `setSubscriptionId(uint64 subId)` like `VRFConsumerExternalSubOwner` does and record all their addresses. 
3. Register all the applications `addConsumer(uint64 subId, address consumer)` . TODO: metamask screen shots
4. Fund the subscription with `LINKTOKEN.transferAndCall(address(COORDINATOR), amount, abi.encode(subId));`


```solidity
pragma solidity ^0.8.0;

import "path/to/LinkTokenInterface.sol";
import "path/to/VRFCoordinatorV2Interface.sol";
import "path/to/VRFConsumerBaseV2.sol";

contract VRFConsumerExternalSubOwnerExample is VRFConsumerBaseV2 {

    VRFCoordinatorV2Interface COORDINATOR;
    LinkTokenInterface LINKTOKEN;

    struct RequestConfig {
        uint64 subId;
        uint32 callbackGasLimit;
        uint16 requestConfirmations;
        uint32 numWords;
        bytes32 keyHash;
    }
    RequestConfig s_requestConfig;
    uint256[] s_randomWords;
    uint256 s_requestId;

    constructor(
        address vrfCoordinator,
        address link,
        uint32 callbackGasLimit,
        uint16 requestConfirmations,
        uint32 numWords,
        bytes32 keyHash
    )
    VRFConsumerBaseV2(vrfCoordinator)
    {
        COORDINATOR = VRFCoordinatorV2Interface(vrfCoordinator);
        LINKTOKEN = LinkTokenInterface(link);
        s_requestConfig = RequestConfig({
            subId: 0, // Initially unset
            callbackGasLimit: callbackGasLimit,
            requestConfirmations: requestConfirmations,
            numWords: numWords,
            keyHash: keyHash
        });
    }

    function fulfillRandomWords(
        uint256 requestId,
        uint256[] memory randomWords
    )
        internal
        override
    {
        s_randomWords = randomWords;
    }

    function requestRandomWords()
        external
    {
        RequestConfig memory rc = s_requestConfig;
        // Will revert if subscription is not set and funded.
        s_requestId = COORDINATOR.requestRandomWords(
            rc.keyHash,
            rc.subId,
            rc.requestConfirmations,
            rc.callbackGasLimit,
            rc.numWords);
    }

    function setSubscriptionID(
        uint64 subId
    )
        public
    {
        s_requestConfig.subId = subId;
    }
}
```


## Advanced

The above examples are by no means the only way to make use of the VRF. The subscription can be managed dynamically as follows: 

- Change the consumer set with `addConsumer(uint64 subId, address consumer)`/`removeConsumer(uint64 subId, address consumer)` 
- Transfer the subscription ownership with `requestSubscriptionOwnerTransfer(uint64 subId, address newOwner)`/`acceptSubscriptionOwnerTransfer(uint64 subId)`.
- Withdraw from the subscription balance with `defundSubscription(uint64 subId, address to, uint96 amount)`
- Top up the subscription balance with `LINKTOKEN.transferAndCall(address(COORDINATOR), amount, abi.encode(subId));`
- View the subscription with `getSubscription(uint64 subId)`
- Cancel the subscription with  `cancelSubscription(uint64 subId)`

The full coordinator interface is available here (TODO link). You can use the subscription management functions however you see fit. 

> 🚧 Security Considerations
>
> Be sure to look your contract over with [these security considerations](../vrf-security-considerations/) in mind!

>❗️ Remember to fund your contract with LINK!
>
> Requesting randomness will fail unless your subscription contract has enough LINK to pay for it. **Learn how to [Acquire testnet LINK](../acquire-link/)**.