# Kokomo Finance Exit Scam 

- **Project**: Kokomo Finance (Optimism-based lending protocol)
- **Exploit_type**: Smart contract loophole → Insider Exit Scam
- **Loss**: ~$4 million 
- **Entry_point**: Wrapped Bitcoin token contract (cBTC) via reward speed and borrow controls
- **Exploit_vector**: The deployer manipulated contract logic to withdraw user-deposited wrapped Bitcoin, swapping it out for profits and immediately abandoning platform
- **Severity**: Critical
- **Attack_steps**:
    - Team reset the rewardSpeed and paused borrowing in the cBTC contract
    - Upgraded the deployed cBTC implementation to malicious code
    - Approved over 7,000 Sonne Wrapped Bitcoin (So-WBTC) for transfer from the contract 
    - Transferred ~7,010 So-WBTC to attacker-controlled address (0x5a2d…)
    - Swapped So-WBTC to 141 WBTC (~$4 million) via DEX
    - Simultaneously wiped social media and website (‘fly off radar’) 
- **Impact**: Approximately $4 million stolen; KOKO token crashed ~95–97%; TVL plummeted 
- **Exploitability**: High (privileged control over token contracts and upgrades)
- **Root_cause**: Full control retained by deployer over proxy contracts, enabling contract logic manipulation and fund extraction without oversight
- **Resource**:[Link](https://cointelegraph.com/news/4m-exit-scam-suspected-as-kokomo-finance-flies-off-radar-token-plunges)
