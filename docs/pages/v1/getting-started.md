---
layout: docs-content
title: Bitlend | Docs - Getting Started
permalink: /
docs_version: v1

## Element ID: In-page Heading
sidebar_nav_data:
  getting-started: Getting Started
  guides: Guides
  networks: Networks
  protocol-math: Protocol Math

deployments:
  Ethereum Mainnet - Bitlend: ## this becomes the header text
    tab_text: Mainnet
    blockscan_origin: 'https://bttcscan.io/'
    contracts:
      bUSDC: '0x0000000000000000000000000000000000000000'
      bTRX: '0x0000000000000000000000000000000000000000'
      bWBTC: '0x0000000000000000000000000000000000000000'
      bWETH: '0x0000000000000000000000000000000000000000'
      BLEND: '0x0000000000000000000000000000000000000000'
      Comptroller: '0x0000000000000000000000000000000000000000'
      Governance: '0x0000000000000000000000000000000000000000'
      Timelock: '0x0000000000000000000000000000000000000000'

  Ethereum Donau Testnet - Bitlend:
    tab_text: Donau
    blockscan_origin: 'https://testnet.bttcscan.io/'
    contracts:
      bUSDC: '0x0000000000000000000000000000000000000000'
      bTRX: '0x0000000000000000000000000000000000000000'
      bWBTC: '0x0000000000000000000000000000000000000000'
      bWETH: '0x0000000000000000000000000000000000000000'
      BLEND: '0x0000000000000000000000000000000000000000'
      Comptroller: '0x0000000000000000000000000000000000000000'
      Governance: '0x0000000000000000000000000000000000000000'
      Timelock: '0x0000000000000000000000000000000000000000'
---

# Getting Started

## Introduction to Bitlend

<div class="new-docs-banner">
  <div class="center">
    <span class="message">Bitlend is now live, you're currently viewing Bitlend documentation.</span>
    <a href="/">
      <span class="button">Bitlend Documentation</span>
    </a>
  </div>
</div>

The Bitlend protocol is based on the [Bitlend Whitepaper](https://compound.finance/documents/Bitlend.Whitepaper.pdf){:target="_blank"} (2019); the codebase is [open-source](https://github.com/Bitlend-protocol/compound-protocol){:target="_blank"}, and maintained by the community.

The app.bitlend.fi interface is [open-source](https://github.com/Bitlend-protocol/palisade){:target="_blank"}, and maintained by the community.

Please join the #development room in the Bitlend community [Discord](https://discord.com/invite/compound){:target="_blank"} server; Bitlend Labs and members of the community look forward to helping you build an application on top of Bitlend. Your questions help us improve, so please don't hesitate to ask if you can't find what you are looking for here.

## Guides

1. [Supplying Assets to the Bitlend Protocol](){:target="_blank"}
2. [Borrowing Assets from the Bitlend Protocol](){:target="_blank"}
3. [Create a Bitlend API with Infura](){:target="_blank"}
4. [Building a Governance Interface](){:target="_blank"}
5. [Delegation & Voting](){:target="_blank"}
6. [Contributing to the Protocol](){:target="_blank"}
{: .mega-ordered-list }

## Networks

The Bitlend Protocol is currently deployed on the following networks:

<div id="networks-widget-container"></div>

You can also see a full list of all deployed contract addresses [here](https://github.com/Bitlend-protocol/compound-config){:target="_blank"}.

## Protocol Math

The Bitlend protocol contracts use a system of exponential math, [ExponentialNoError.sol](https://github.com/Bitlend-protocol/compound-protocol/blob/master/contracts/ExponentialNoError.sol){:target="_blank"}, in order to represent fractional quantities with sufficient precision.

Most numbers are represented as a *mantissa*, an unsigned integer scaled by `1 * 10 ^ 18`, in order to perform basic math at a high level of precision.

### bToken and Underlying Decimals

Prices and exchange rates are scaled by the decimals unique to each asset; bTokens are ERC-20 tokens with 8 decimals, while their underlying tokens vary, and have a public member named *decimals*.

| bToken | bToken Decimals | Underlying | Underlying Decimals |
| ------ | --------------- | ---------- | ------------------- |
| bBTT   | 8               | BTT        | 18                  |
| bWETH  | 8               | WETH       | 18                  |
| bWBTC  | 8               | WBTC       | 8                   |
| bUSDC  | 8               | USDC       | 6                   |
| bTRX   | 8               | TRX        | 6                   |
{: .decimals-events-table }

### Interpreting Exchange Rates

The bToken [Exchange Rate](/btokens#exchange-rate) is scaled by the difference in decimals between the bToken and the underlying asset.

```
oneBTokenInUnderlying = exchangeRateCurrent / (1 * 10 ^ (18 + underlyingDecimals - bTokenDecimals))
```

Here is an example of finding the value of 1 cBAT in BAT with Web3.js JavaScript.

```js
const bTokenDecimals = 8; // all bTokens have 8 decimal places
const underlying = new web3.eth.Contract(erc20Abi, batAddress);
const bToken = new web3.eth.Contract(bTokenAbi, cBatAddress);
const underlyingDecimals = await underlying.methods.decimals().call();
const exchangeRateCurrent = await bToken.methods.exchangeRateCurrent().call();
const mantissa = 18 + parseInt(underlyingDecimals) - bTokenDecimals;
const oneBTokenInUnderlying = exchangeRateCurrent / Math.pow(10, mantissa);
console.log('1 cBAT can be redeemed for', oneBTokenInUnderlying, 'BAT');
```

There is no underlying contract for ETH, so to do this with cETH, set `underlyingDecimals` to 18.

To find the number of underlying tokens that can be redeemed for bTokens, multiply the number of bTokens by the above value `oneBTokenInUnderlying`.

```
underlyingTokens = bTokenAmount * oneBTokenInUnderlying
```

### Calculating Accrued Interest

Interest rates for each market update on any block in which the ratio of borrowed assets to supplied assets in the market has changed. The amount interest rates are changed depends on the interest rate model smart contract implemented for the market, and the amount of change in the ratio of borrowed assets to supplied assets in the market.

See the interest rate data visualization notebook on [Observable](https://observablehq.com/@jflatow/compound-interest-rates){:target="_blank"} to visualize which interest rate model is currently applied to each market.

Historical interest rates can be retrieved from the [MarketHistoryService API](/api#MarketHistoryService).

Interest accrues to all suppliers and borrowers in a market when any Ethereum address interacts with the market’s bToken contract, calling one of these functions: mint, redeem, borrow, or repay. Successful execution of one of these functions triggers the `accrueInterest` method, which causes interest to be added to the underlying balance of every supplier and borrower in the market. Interest accrues for the current block, as well as each prior block in which the `accrueInterest` method was not triggered (no user interacted with the bToken contract). Interest compounds only during blocks in which the bToken contract has one of the aforementioned methods invoked.

Here is an example of supply interest accrual:

Alice supplies 1 ETH to the Bitlend protocol. At the time of supply, the `supplyRatePerBlock` is 37893605 Wei, or 0.000000000037893605 ETH per block. No one interacts with the cEther contract for 3 Ethereum blocks. On the subsequent 4th block, Bob borrows some ETH. Alice’s underlying balance is now 1.000000000151574420 ETH (which is 37893605 Wei times 4 blocks, plus the original 1 ETH). Alice’s underlying ETH balance in subsequent blocks will have interest accrued based on the new value of 1.000000000151574420 ETH instead of the initial 1 ETH. Note that the `supplyRatePerBlock` value may change at any time.

### Calculating the APY Using Rate Per Block

The Annual Percentage Yield (APY) for supplying or borrowing in each market can be calculated using the value of `supplyRatePerBlock` (for supply APY) or `borrowRatePerBlock` (for borrow APY) in this formula:

```
Rate = bToken.supplyRatePerBlock(); // Integer
Rate = 37893566
ETH Mantissa = 1 * 10 ^ 18 (ETH has 18 decimal places)
Blocks Per Day = 6570 (13.15 seconds per block)
Days Per Year = 365

APY = ((((Rate / ETH Mantissa * Blocks Per Day + 1) ^ Days Per Year)) - 1) * 100
```

Here is an example of calculating the supply and borrow APY with Web3.js JavaScript:

```js
const ethMantissa = 1e18;
const blocksPerDay = 6570; // 13.15 seconds per block
const daysPerYear = 365;

const bToken = new web3.eth.Contract(cEthAbi, cEthAddress);
const supplyRatePerBlock = await bToken.methods.supplyRatePerBlock().call();
const borrowRatePerBlock = await bToken.methods.borrowRatePerBlock().call();
const supplyApy = (((Math.pow((supplyRatePerBlock / ethMantissa * blocksPerDay) + 1, daysPerYear))) - 1) * 100;
const borrowApy = (((Math.pow((borrowRatePerBlock / ethMantissa * blocksPerDay) + 1, daysPerYear))) - 1) * 100;
console.log(`Supply APY for ETH ${supplyApy} %`);
console.log(`Borrow APY for ETH ${borrowApy} %`);
```