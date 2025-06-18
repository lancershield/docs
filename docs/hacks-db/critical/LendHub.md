# LendHub Double-Spending Exploit 

- **Project**: LendHub
- **Exploit_type**: Double-Spending via Legacy Contract Conflict
- **Loss**: ~$6 million
- **Entry_point**: withdraw() function across two versions of LendHub’s lending contracts
- **Exploit_vector**: The attacker exploited a duplication in token accounting between an old and new contract. By interacting with both, they withdrew more tokens than were legitimately supplied.
- **Severity**: Critical
- **Attack_steps**:
    - LendHub deployed a new lending market contract but failed to deprecate the old one properly.
    - Both old and new contracts tracked the same token balance without shared state.
    - Attacker supplied tokens to the old contract and triggered logic to withdraw from the new one.
    - Due to inconsistent accounting, the same tokens were counted as collateral in both systems.
    - Repeated this process to withdraw ~$6M worth of tokens (mostly stablecoins and ETH derivatives).
    - Funds were laundered through multiple wallets and bridges.
- **Impact**: ~$6 million in assets were stolen due to token over-withdrawal exploiting mismatched logic.
- **Exploitability**: High
- **Root_cause**: Coexistence of two independently accounting contract versions for the same asset, allowing the same collateral to be counted twice — no migration safeguards or asset lockout enforced.
- **Resource**:[Link](https://cointelegraph.com/news/lendhub-protocol-exploiters-spotted-shifting-3-85m-into-tornado-cash)