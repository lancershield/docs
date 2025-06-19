# LeetSwap LP Drain Exploit 

- **Project**: LeetSwap (Base‑chain DEX)
- **Exploit_type**: Access Control Misconfiguration + Price Manipulation
- **Loss**: 340 ETH ($630,000) 
- **Entry_point**: _transferFeesSupportingTaxTokens() function in the LeetSwapV2 pair contract
- **Exploit_vector**: The attacker exploited the publicly accessible fee-transfer function to drain token balances, manipulate price, sync reserves, then extract ETH via a reverse swap
- **Severity**: Critical
- **Attack_steps**:
    - Discovered _transferFeesSupportingTaxTokens() was declared public instead of internal, allowing external invocation 
    - Performed a small token swap (e.g., WETH → TokenA) to gain tokens in the pool
    - Called the public fee-transfer function to move TokenA from the contract to the fee recipient, emptying the fee-balance
    - Executed sync() to re-balance reserves based on the modified state
    - Swapped TokenA back to WETH, draining the pool at inflated rates
    - Repeated this attack across multiple LP pools, extracting 340 ETH (~$630K) 
- **Impact**: ~$630K drained from multiple liquidity pools; LeetSwap paused trading; TVL dropped from ~$41M to ~$7M 
- **Exploitability**: High — the vulnerable function needed only a simple public call
- **Root_cause**: A mis-specified function visibility (public vs internal) in fee-transfer logic enabled unauthorized external calls and price manipulation
- **Resource**:[Link](https://www.coindesk.com/business/2023/08/01/leetswap-halts-trading-after-630k-drained-from-liquidity-pairs) 
