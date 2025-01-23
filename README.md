# DEX Arbitrage Smart Contract and Trading Bot

## Introduction

Decentralized Exchange (DEX) arbitrage is a strategy to capitalize on price discrepancies between two or more decentralized exchanges. This method involves buying a token on one DEX where it is undervalued and selling it on another where it is overvalued, thereby profiting from the price difference. This repository provides an implementation of a smart contract and a trading bot to perform DEX arbitrage effectively.

## How It Works

### Arbitrage Mechanics

1. **Price Imbalance:** Large trades can create slippage and price imbalances in liquidity pools.
2. **Restoring Balance:** Arbitrage bots exploit these imbalances to restore price equilibrium by transferring liquidity between markets.
3. **Profitability Check:** The smart contract ensures the entire transaction is profitable; otherwise, it reverts the trade to avoid losses.

### Key Features

- **Batch Transactions:** Combine multiple swaps into a single transaction.
- **Profit Validation:** Transactions are reverted if no profit is made.
- **Customizable Routes:** Flexible architecture to support different tokens, routes, and DEXes.
- **Supports Dual and Triangular Arbitrage:** Includes functionality for two-token and three-token arbitrage strategies.

---

## Smart Contract Highlights

### Checking Prices and Trade Profitability

The `getAmountOutMin` function queries the router to get the minimum output for a given input amount:

```solidity
function getAmountOutMin(address router, address _tokenIn, address _tokenOut, uint256 _amount) public view returns (uint256) {
  address[] memory path = new address[](2);
  path[0] = _tokenIn;
  path[1] = _tokenOut;
  uint256[] memory amountOutMins = IUniswapV2Router(router).getAmountsOut(_amount, path);
  return amountOutMins[path.length - 1];
}
```

For dual DEX trades, we estimate profitability with:

```solidity
function estimateDualDexTrade(address _router1, address _router2, address _token1, address _token2, uint256 _amount) external view returns (uint256) {
  uint256 amtBack1 = getAmountOutMin(_router1, _token1, _token2, _amount);
  uint256 amtBack2 = getAmountOutMin(_router2, _token2, _token1, amtBack1);
  return amtBack2;
}
```

### Executing Trades

The `dualDexTrade` function batches trades to ensure profitability:

```solidity
function dualDexTrade(address _router1, address _router2, address _token1, address _token2, uint256 _amount) external onlyOwner {
  uint startBalance = IERC20(_token1).balanceOf(address(this));
  swap(_router1, _token1, _token2, _amount);
  uint tradeableAmount = IERC20(_token2).balanceOf(address(this)) - IERC20(_token2).balanceOf(address(this));
  swap(_router2, _token2, _token1, tradeableAmount);
  require(IERC20(_token1).balanceOf(address(this)) > startBalance, "Trade Reverted, No Profit Made");
}
```

---

## Setting Up

### Requirements

- Node.js
- Hardhat
- Solidity compiler
- Aurora testnet/mainnet RPC URL
- A wallet private key

### Installation

Clone this repository and install dependencies:

```bash
git clone https://github.com/jamesbachini/DEX-Arbitrage.git
cd DEX-Arbitrage
npm install
```

### Deployment

1. Add your private key to the `.env` file.
2. Deploy the smart contract:
   ```bash
   npx hardhat run --network aurora ./scripts/deploy.js
   ```
3. Add the deployed contract address to `.env`.

---

## Trading Bot Controller

### Overview

The bot queries routes and tokens for opportunities and executes trades when profitable.

### Key Functions

- **Checking Profitability:**
  ```javascript
  const amtBack = await arb.estimateDualDexTrade(router1, router2, token1, token2, amount);
  if (amtBack.gt(profitTarget)) {
    await dualTrade(router1, router2, token1, token2, amount);
  }
  ```
- **Executing Trades:**
  ```javascript
  const tx = await arb.connect(owner).dualDexTrade(router1, router2, token1, token2, amount);
  await tx.wait();
  ```

### Usage

1. Configure your routes and tokens in the `config/aurora.json` file.
2. Run the bot:
   ```bash
   node ./scripts/trade.js
   ```

---

## Research & Optimization

### Gathering Data

- Use tools like [DeFiLlama](https://defillama.com/) to identify active DEXes.
- Query token lists (e.g., [Aurora token list](https://raw.githubusercontent.com/aurora-is-near/bridge-assets/master/assets/aurora.tokenlist.json)) to find tradeable pairs.

### Expanding Opportunities

- Explore triangular arbitrage with:
  ```solidity
  function estimateTriDexTrade(address _router1, address _router2, address _router3, address _token1, address _token2, address _token3, uint256 _amount) external view returns (uint256) {
    uint amtBack1 = getAmountOutMin(_router1, _token1, _token2, _amount);
    uint amtBack2 = getAmountOutMin(_router2, _token2, _token3, amtBack1);
    uint amtBack3 = getAmountOutMin(_router3, _token3, _token1, amtBack2);
    return amtBack3;
  }
  ```

---

## Results and Observations

### Testing on Aurora

- Small-scale tests with \~\$20 capital showed profitable returns:
  - **wNEAR:** 9.66% in under 12 hours.
  - **USDT:** 1.24%.
- Scaling to \$300 reduced profitability due to slippage.

### Key Takeaways

- DEX arbitrage on low-volume chains can yield excellent returns with low risk.
- High-volume chains require gas optimization and advanced strategies like MEV (Miner Extractable Value).

---

## Disclaimer

This code is for educational purposes only. It is unaudited and not production-ready. Use at your own risk.

---

## Contributing

Feel free to fork and contribute to the project. Pull requests are welcome for improvements and optimizations.

---

## License

This project is licensed under the MIT License. See the LICENSE file for details.

