# Inverse Finance Hack

- **Project**: Inverse Finance
- **Exploit_type**: Price Oracle Manipulation (via Flash Loan)
- **Loss**: ~$15.6 million
- **Entry_point**: anchorRouter contract and price oracle dependency
- **Exploit_vector**: Manipulated token price oracle using flash loans to borrow undercollateralized assets
- **Severity**: Critical
- **Attack_steps**:
    - Attacker flash loaned large amounts of WBTC from a lending protocol.
    - Used WBTC to manipulate the price of INV/wETH on decentralized exchanges.
    - This skewed the price oracle used by Inverse’s lending system.
    - Deposited manipulated collateral (e.g., INV) into Inverse Finance’s Anchor lending protocol.
    - Borrowed large quantities of DOLA, ETH, WBTC, and YFI against inflated collateral.
    - Exited positions and repaid flash loans, extracting ~$15.6M in profit.
- **Impact**: DOLA stablecoin depegged temporarily; several core assets drained from Anchor
- **Exploitability**: High
- **Root_cause**: Use of non-TWAP oracle relying on easily manipulable on-chain token pairs
- **Resource**:[Link](https://www.vidma.io/blog/inverse-finance-hack-a-15-6-million-lesson-in-defi-oracle-manipulation) 

