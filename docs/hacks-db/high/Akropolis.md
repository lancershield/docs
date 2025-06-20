# Akropolis

- **Project**: Akropolis
- **Exploit_type**: Reentrancy via Curve Y Pool integration
- **Loss**: ~$2 million
- **Entry_point**: withdraw() function in Akropolis' SavingsModule
- **Exploit_vector**: Reentrancy exploit during withdraw() that allowed repeated extraction of funds
- **Severity**: High
- **Attack_steps**:
    - Attacker created a contract that interacted with Akropolis’ SavingsModule.
    - Used the contract to deposit funds into Akropolis using supported stablecoins.
    - Triggered the withdraw() function, which interacted with Curve’s Y pool for liquidity.
    - Exploited a reentrancy vulnerability in the callback path during liquidity withdrawal.
    - Reentered the withdraw() logic multiple times before state was properly updated.
    - Drained ~$2 million in DAI across multiple transactions.
- **Impact**: ~$2 million in DAI drained from Akropolis yield pools
- **Exploitability**: Medium
- **Root_cause**: Incomplete reentrancy protection around external Curve pool calls
- **Resource**:[Link](https://rekt.news/akropolis-rekt/)