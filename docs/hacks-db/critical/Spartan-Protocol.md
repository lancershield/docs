# Spartan Protocol hack

- **Project**: Spartan Protocol
- **Exploit_type**: Flash-loan + Liquidity share miscalculation
- **Loss**: ~$30 million
- **Entry_point**: removeLiquidity() / calcLiquidityShare()
- **Exploit_vector**: Flash-loan inflated pool balance before liquidity removal
- **Severity**: Critical
- **Attack_steps**:
    - Took flash-loan (~100k WBNB) from PancakeSwap
    - Added liquidity with WBNB + SPARTA and minted LP tokens
    - Manipulated pool balance via asset transfers and swaps
    - Burned LP tokens to withdraw inflated share
    - Repeated the cycle multiple times
    - Repaid flash-loan and retained profits
- **Impact**: ~2.6M SPARTA and 21k WBNB stolen ($30M)
- **Exploitability**: High
- **Root_cause**: Liquidity calculation used live balance instead of a fixed reserve snapshot
- **Resource**:[Link](https://www.halborn.com/blog/post/explained-the-spartan-protocol-hack-may-2021)