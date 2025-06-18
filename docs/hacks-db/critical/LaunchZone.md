# LaunchZone Token Drain Exploit 

- **Project**: LaunchZone 
- **Exploit_type**: Excessive Token Allowance + Arbitrary Swap Exploit
- **Loss**: ~$700,000
- **Entry_point**: SwapX router contract
- **Exploit_vector**: The attacker abused pre-approved token allowance by calling arbitrary swap functions through an upgradeable router, draining liquidity from user-pools.
- **Severity**: Critical
- **Attack_steps**:
    - The attacker identified that the SwapX implementation contract (an upgradeable router) had excessive token allowances granted by users.
    - They crafted malicious swap transactions targeting SwapX, which pushed tokens from user wallets.
    - Used the compromised router to manipulate token flows in liquidity pools (e.g., DND, LZ).
    - Drained approximately $700K from the liquidity pool—about 80% of pool funds.
    - Funds were swapped out through PancakeSwap in a rapid series of transactions. 
    - Post-exploit, LaunchZone halted token transfers and paused trading to prevent further losses. 
- **Impact**: ~$700K worth of user funds drained from liquidity pools; native token dropped ~80%.
- **Exploitability**: High
- **Root_cause**: Failure to restrict spending permissions on the router (SwapX), plus lack of access control and auditing on upgradeable contract logic which permitted exploitation of allowance privileges.
- **Resource**:[Link](https://cointelegraph.com/news/700-000-drained-from-bnb-chain-based-defi-protocol-launchzone) 
