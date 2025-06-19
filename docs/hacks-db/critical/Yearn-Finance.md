# Yearn Finance

- **Project**: Yearn Finance
- **Exploit_type**: Token Misconfiguration → Supply Mint Logic Bug
- **Loss**: ~$11.5 million  
- **Entry_point**: yUSDT vault contract referencing iUSDC instead of iUSDT
- **Exploit_vector**: A copy-paste error caused yUSDT vault to mint a disproportionate number of yUSDT tokens since it tracked the wrong interest-bearing asset. This enabled attackers to mint yUSDT massively using minimal USDT deposits.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker obtained flash loans (~$12 M across DAI, USDC, USDT).
    - Deposited into misconfigured yUSDT vault.
    - Vault minted over 1 quadrillion yUSDT due to incorrect iUSDC reference.
    - Redeemed minted yUSDT for underlying assets via Curve swaps.
    - Returned flash loans; swapped residual tokens for profit.
- **Impact**: ~$11.5 M drained from legacy Yearn vault; contract since deprecated.
- **Exploitability**: High – simple logic/config error exploited in a single transaction.
- **Root_cause**: Vault constructor used wrong interest-bearing asset address (iUSDC), breaking the value backing for yUSDT token.
- **Resource**:[Link](https://quillaudits.medium.com/decoding-yearn-finance-11-million-hack-quillaudits-c9a75ac7e68b)