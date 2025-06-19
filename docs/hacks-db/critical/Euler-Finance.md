# Euler Finance Flash Loan Exploit

- **Project**: Euler Finance
- **Exploit_type**: Flash Loan + Logic Flaw in donateToReserves()
- **Loss**: ~$197 million (DAI, wBTC, stETH, USDC, etc.) 
- **Entry_point**: EToken::donateToReserves in the EToken collateral contracts
- **Exploit_vector**: Attacker used flash-loaned funds to self-over-leverage, manipulate health score, donate collateral, and then liquidate themselves under favorable terms
- **Severity**: Critical
- **Attack_steps**:
    - Acquired a ~$30 million DAI flash loan from Aave. 
    - Deposited ~20M DAI into Euler to mint eDAI.
    - Borrowed against collateral, minting large amounts of eDAI and dDAI multiple times. 
    - Donated eDAI to reserves via donateToReserves(), reducing collateral without adjusting debt. 
    - Self-liquidated: protocol liquidator contract purchased discounted collateral. 
    - Withdrew assets, repaid flash loan, pocketed ~$8–9 million per asset type; repeated across tokens (wBTC, stETH, USDC, DAI). 
- **Impact**: ~$197 million drained across multiple tokens; protocol TVL dropped from ~$264M to ~$10M; significant liquidation of protocol reserves. 
- **Exploitability**: High
- **Root_cause**: Missing solvency checks in donateToReserves() allowed collateral donation that didn't reduce debt, enabling self-liquidation under generous protocol discount logic.
- **Resource**:[Link](https://www.chainalysis.com/blog/euler-finance-flash-loan-attack/) 
