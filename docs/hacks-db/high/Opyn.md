# Opyn ETH Put Bug

- **Project**: Opyn (ETH Put oTokens)
- **Exploit_type**: Double-Spend via Loop Misuse / Improper msg.value Handling
- **Loss**:371,260 USDC 
- **Entry_point**: exercise() function in Opyn's ETH Put contract — specifically the internal _exercise() loop that re-used msg.value improperly 
- **Exploit_vector**: Attacker called exercise() with multiple vaults, reusing one ETH payment (msg.value) across vaults and withdrawing USDC collateral more than once 
- **Severity**: High 
- **Attack_steps**:
    - Attacker obtained oETH Put tokens (via purchase or mint).
    - Constructed a single exercise(oTokens, [vault1, vault2, …]) call supplying one ETH._exercise() loop processed multiple vaults but reused the single msg.value.
    - With only one ETH paid, attacker drained USDC collateral from each vault.Resulted in duplicate withdrawals totaling 371,260 USDC 
- **Impact**: $371k USDC stolen from ETH Put options sellers; protocol liquidity disrupted; Uniswap liquidity removed to mitigate further damage 
- **Exploitability**: High – the bug was trivial to execute by any holder of oETH Put tokens, requiring no complex setup 
- **Root_cause**: Incorrect reuse of msg.value across multiple _exercise() calls — payment guard only checked once, enabling multiple collateral drains per single payment
- **Resource**:[Link](https://medium.com/opyn/opyn-eth-put-exploit-c5565c528ad2)

