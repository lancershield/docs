# Sonne Finance Precision-Loss Exploit 

- **Project**: Sonne Finance
- **Exploit_type**: Flash Loan + Precision-Loss (Compound V2 fork donation bug)
- **Loss**: ≈ $20 million 
- **Entry_point**: Multiple market initialization transactions (VELO lending pool creation + collateral factor assignment) in timelocked multisig
- **Exploit_vector**: The attacker hijacked permissionless transaction execution post-timelock to enable known Compound V2 precision-loss attack via flash loans
- **Severity**: Critical
- **Attack_steps**:
    - Scheduled VELO-based lending markets via a 2-day timelock proposal
    - Attacker executed multiple series of permissionless execution right after timelock ended
    - Created empty pools with zero initial liquidity
    - Conducted flash-loan-driven deposit-redeem loops (donation attack), exploiting rounding imprecision
    - Drained ~$20M before pools were seeded or properly secured
    - A security contributor managed to rescue ~$6.5M by quickly seeding VELO mid-attack 
- **Impact**: ~$20M stolen from Optimism-based lending pools (USDC/WETH), markets paused immediately
- **Exploitability**: High
- **Root_cause**: Separation of pool deployment and collateral-factor adjustment allowed attacker to front-run setup; use of empty market + known Compound V2 rounding bug
- **Resource**:[Link](https://www.halborn.com/blog/post/explained-the-sonne-finance-hack-may-2024)

