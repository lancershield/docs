# Saddle Finance Exploit 

- **Project**: Saddle Finance
- **Exploit_type**: Incomplete Reentrancy Protection
- **Loss**: ~$10 million
- **Entry_point**: swap() function in the Saddle swap contract
- **Exploit_vector**: The attacker exploited a reentrancy bug by calling back into the protocol during a swap() operation, enabling them to manipulate pool state mid-transaction and withdraw more than they should.
- **Severity**: Critical
- **Attack_steps**:
    - Deployed a malicious contract to interact with Saddle’s liquidity pool.
    - Initiated a swap() call that triggered internal balance updates.
    - Exploited reentrancy by re-entering the swap() logic before the contract state was updated properly.
    - Manipulated the pool’s virtual price and token balances during the reentrant call.
    - Extracted significantly more funds than entitled to through repeated abuse in a single transaction.
    - Laundered funds across chains and through mixers.
- **Impact**: ~$10M in stablecoins and LP tokens drained from Saddle’s liquidity pools.
- **Exploitability**: High
- **Root_cause**: Reentrancy guard was not correctly implemented across all vulnerable code paths—particularly in custom swap logic—allowing state to be manipulated during execution.
- **Resource**:[Link](https://pexx.com/chaindebrief/snowdog-avalanche-rugpull/)