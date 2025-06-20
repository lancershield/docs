# bZx Oracle Manipulation Exploit 

- **Project**: bZx
- **Exploit_type**: Price Oracle Manipulation via Flash Loans
- **Loss**: ~$350,000 (first of multiple exploits)
- **Entry_point**: iETH and Fulcrum contracts relying on manipulated oracles
- **Exploit_vector**: Manipulated token price using Uniswap via a flash loan to borrow more than entitled
- **Severity**: High
- **Attack_steps**:
    - Attacker took a 10,000 ETH flash loan from dYdX.
    - Sent half to Compound to borrow WBTC and manipulated ETH/wBTC price on Uniswap.
    - The manipulated price caused bZx’s oracle to read a false ETH/WBTC ratio.
    - Used inflated ETH price to take an undercollateralized loan from bZx.
    - Exited positions after profiting and repaid the flash loan.
    - Extracted ~$350,000 profit from the protocol.
- **Impact**: Substantial undercollateralized loan drained from bZx’s lending pool
- **Exploitability**: High
- **Root_cause**: Dependence on a single on-chain price source (Uniswap) without time-averaging or manipulation resistance
- **Resource**:[Link](https://www.palkeo.com/en/projets/ethereum/bzx.html)