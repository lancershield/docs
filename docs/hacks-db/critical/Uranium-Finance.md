# Uranium Finance Exploit 

- **Project**: Uranium Finance
- **Exploit_type**: Misconfigured Math Logic (Rounding Bug in Swap Function)
- **Loss**: ~$57 million
- **Entry_point**: swap() function in the custom AMM pair contract
- **Exploit_vector**: The attacker exploited incorrect token amount calculation due to a flawed divisor in the constant product formula. 
- The contract used 10000 instead of 10000 - fee for output amount calculation, leading to significant over-withdrawals.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker identified that the AMM pair contract used an incorrect formula for swap calculations.
    - Specifically, the divisor used was `10000` instead of `9975`, failing to account for swap fees.
    - This let the attacker receive significantly more tokens than intended for each swap.
    - They performed multiple swaps, draining liquidity pools without triggering errors.
    - They routed funds through various addresses and ultimately to Tornado Cash for laundering.
- **Impact**: Drained ~$57M in liquidity from various pools (including ETH, BTCB, BUSD, USDT).
- **Exploitability**: High
- **Root_cause**: Arithmetic miscalculation in swap output logic — failure to deduct fee from the divisor, which allowed extraction of more tokens than deposited in trades.
- **Resource**:[Link](https://medium.com/immunefi/building-a-poc-for-the-uranium-heist-ec83fbd83e9f)