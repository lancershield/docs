# WOOFi Swap sPMM Flash Loan Exploit

- **Project**: WOOFi (Arbitrum-based DEX)
- **Exploit_type**: Flash Loan + Price Manipulation in sPMM Logic
- **Loss**: ~$8.75 million 
- **Entry_point**: swap() function in the WOOFi Swap v2 contract
- **Exploit_vector**: Attacker manipulated the synthetic proactive market maker (sPMM) price by executing flash-loaned token swaps, causing the algorithm to misprice and extract funds
- **Severity**: Critical
- **Attack_steps**:
    - Attacker took out a flash loan of ~7.7 million WOO tokens. 
    - Swapped WOO tokens into the pool, triggering the sPMM logic to adjust price precipitously to near zero. 
    - Oracle fallback was missing for WOO, allowing sPMM price to remain artificially low.
    - Attacker swapped ~10 million WOO for near-zero output, exiting with large ETH or stablecoin receipts. 
    - Flash loan repaid; attacker netted ~$8.75 million profit—all within ~13 minutes before detection.
- **Impact**: ~$8.75 M stolen; WOOFi Swap v2 contract frozen for two weeks; liquidity severely impacted 
halborn.com
- **Exploitability**: High — attacker used flash loan to manipulate pricing in a single transaction
- **Root_cause**: sPMM algorithm lacked protection against large arbitrary trades and didn’t enforce oracle price fallback, enabling flash-loan-induced price distortion
- **Resource**:[Link](https://www.halborn.com/blog/post/explained-the-woofi-hack-march-2024)
