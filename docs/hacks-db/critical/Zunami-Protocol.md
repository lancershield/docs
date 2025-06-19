# Zunami Protocol Price Manipulation Exploit 

- **Project**: Zunami Protocol
- **Exploit_type**: Flash Loan + Price Manipulation
- **Loss**: ~$2.1 million
- **Entry_point**: cacheAssetPrice() and on-chain price computation logic in Stable Pool contracts (totalHoldings(), assetPriceCached())
- **Exploit_vector**: The attacker used flash loans to manipulate pool asset prices, then triggered faulty price caching to inflate their token balance and drained funds.
- **Severity**: Critical
- **Attack_steps**:
    - Borrowed ~7 M USDT (Uniswap V3), ~7 M USDC & 10,011 WETH (Balancer) via flash loans
    - Deposited USDC into Curve pools to mint crvFRAX, then swapped for UZD and zETH 
    - Bought SDT on Curve and SushiSwap, then donated SDT to inflate assetPriceCached via cacheAssetPrice() 
    - Cached inflated price caused attacker’s token balance to jump dramatically
    - Redeemed inflated UZD/zETH back to underlying assets, repaid flash loans, and pocketed ~$2.1 M 
- **Impact**: Drained ~$2.1 million from zStables (UZD, zETH) Curve liquidity pools; collateral depegged significantly (UZD → 99% drop; zETH → 85% drop) 
- **Exploitability**: High
- **Root_cause**: Unsafely cached price logic—cacheAssetPrice() relied on a mutable pool balance and allowed flash-loan influenced price snapshots without adequate validation
- **Resource**:[Link](https://www.halborn.com/blog/post/explained-the-zunami-protocol-hack-august-2023) 
