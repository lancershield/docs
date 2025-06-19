# vETH Token Smart Contract Hack

- **Project**: vETH Token
- **Exploit_type**: Business Logic Flaw in Token Mint/Borrow Functionality
- **Loss**: ~$450,000
- **Entry_point**: takeLoan() function within the vETH factory architecture
- **Exploit_vector**: A logic error allowed the attacker to manipulate the Uniswap pair state, minting vETH without proper backing
- **Severity**: Critical
- **Attack_steps**:
    - Attacker called takeLoan() via the factory contract to borrow vETH with minimal collateral.
    - The function modified Uniswap pair balances incorrectly, breaking the invariant 
    - This let the attacker mint vETH without actually transferring proportional assets.
    - Multiple cycles were executed to inflate their vETH balance fraudulently.
    - Accumulated vETH was swapped on Uniswap to extract ~$450K worth of value.
- **Impact**: ~$450K drained; vETH price collapsed; confidence in project token mechanics severely damaged
- **Exploitability**: High – the logic flaw was exploitable in a single transaction
- **Root_cause**: Lack of input validation and misuse of Uniswap pair logic in business flow—failed to ensure the invariant held during internal loans
- **Resource**:[Link](https://blog.solidityscan.com/veth-token-hack-analysis-828f6b8f5fd7)
