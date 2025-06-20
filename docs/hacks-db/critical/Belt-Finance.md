# Belt Finance Exploit 

- **Project**: Belt Finance
- **Exploit_type**: Flash Loan & Oracle Manipulation
- **Loss**: ~$6.3 million
- **Entry_point**: BeltBUSD vault strategy and price oracle logic
- **Exploit_vector**: Flash loan attack manipulating PancakeSwap pool pricing to exploit vault withdrawal mechanism
- **Severity**: Critical
- **Attack_steps**:
    - Attacker took a large flash loan from PancakeSwap in multiple stablecoins.
    - Swapped tokens across various liquidity pools to manipulate the price ratio used in Belt Finance’s oracle.
    - Deposited manipulated assets into the BeltBUSD vault, locking in an inflated share price.
    - Immediately withdrew from the vault, receiving more assets than deposited.
    - Repeated the attack across multiple stablecoin vaults (BUSD, USDT, USDC, DAI).
    - Repaid the flash loan and retained ~$6.3M in profit.
- **Impact**: Vault funds drained across multiple stablecoin strategies, user funds indirectly affected
- **Exploitability**: High
- **Root_cause**: Oracle read spot price from manipulable DEX pools without time-averaging or resistance
- **Resource**:[Link](https://rekt.news/belt-rekt/)