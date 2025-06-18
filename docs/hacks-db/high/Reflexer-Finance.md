# RAI Reflex Bonding Curve Bug 

- **Project**: Reflexer Finance (RAI)
- **Exploit_type**: Bonding Curve Mispricing Bug
- **Loss**: ~$3.3 million
- **Entry_point**: buy() and sell() functions in the bonding curve contract
- **Exploit_vector**: A misconfiguration in the bonding curve logic allowed users to buy RAI from the curve at a lower price and immediately sell it back at a higher price, effectively creating an infinite arbitrage loop.
- **Severity**: High
- **Attack_steps**:
    - A bug was introduced during a bonding curve update that inverted the buy/sell pricing logic.
    - Attacker bought large amounts of RAI at a low price from the curve contract.
    - Immediately sold the same RAI tokens back to the contract at a higher price.
    - Repeated this buy-low/sell-high cycle within a single transaction.
    - Drained ~$3.3M in ETH and stablecoins from the bonding curve’s liquidity.
    - Reflexer team paused the bonding curve and patched the pricing bug.
- **Impact**: ~$3.3M drained from protocol reserves due to incorrect price calculation logic.
- **Exploitability**: Medium
- **Root_cause**: Inverted pricing logic in the bonding curve smart contract allowed users to profit infinitely from buy-sell cycles within the same block, due to a math bug in the curve calculation.
- **Resource**:[Link](https://github.com/reflex-frp/reflex-platform/blob/develop/HACKING.md)