# Hundred Finance Hack 

- **Project**:  Hundred Finance Hack
- **Exploit_type**: Price Manipulation via Oracle Misuse
- **Loss**: ~$7.4 million
- **Entry_point**: borrow() function interacting with manipulated collateral valuation
- **Exploit_vector**: The attacker manipulated the price oracle by inflating the value of collateral, allowing them to borrow significantly more than they should, then drained the protocol.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker deployed a token and added it to a Hundred Finance market as collateral.
    - Supplied a large amount of this token as collateral.
    - Manipulated the token’s price using a low-liquidity pair that influenced the oracle.
    - Borrowed high-value assets (USDC, ETH, etc.) based on the artificially high collateral valuation.
    - Repeated the manipulation across multiple transactions to maximize extraction.
    - Drained ~$7.4M in assets and funneled the funds through multiple wallets.
- **Impact**: ~$7.4 million in stablecoins and blue-chip tokens stolen from the protocol.
- **Exploitability**: High
- **Root_cause**: Oracle manipulation via low-liquidity token pricing allowed attackers to over-leverage their position and drain liquidity without proper collateral backing.
- **Resource**:[Link](https://cointelegraph.com/news/hundred-finance-hacker-moves-assets-year-after-exploit)
