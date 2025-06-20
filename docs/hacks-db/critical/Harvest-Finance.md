# Harvest Finance

- **Project**: Harvest Finance
- **Exploit_type**: Price Oracle Manipulation (Flash Loan Attack)
- **Loss**: ~$33.8 million
- **Entry_point**: withdraw() function in the fUSDT and fUSDC vaults
- **Exploit_vector**: Manipulated Curve pool prices via flash loans to withdraw more than deposited
- **Severity**: Critical
- **Attack_steps**:
    - Attacker took a large flash loan from Uniswap to obtain capital.
    - They swapped funds repeatedly in the Curve Y pool to manipulate the price of USDT/USDC.
    - This manipulation caused Harvest’s internal price oracle to report inflated vault token values.
    - Attacker deposited stablecoins into Harvest’s fUSDT and fUSDC vaults at manipulated high prices.
    - Immediately withdrew funds after manipulation reverted, extracting excess value.
    - Repaid the flash loan and routed profits through Tornado Cash for obfuscation.
- **Impact**: ~$33.8M drained from Harvest vaults (mostly USDT and USDC)
- **Exploitability**: High
- **Root_cause**: Reliance on manipulable on-chain price data without time-weighted average price (TWAP) or oracle safeguards
- **Resource**:[Link](https://crystalintelligence.com/investigations/defi-hacks-case-study-harvest-finance-protocol/)

