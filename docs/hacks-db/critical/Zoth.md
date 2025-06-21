# Zoth Hack

- **Project**: Zoth
- **Exploit_type**: Access Control (Admin Key Compromise)
- **Loss**: ~$8.4 million
- **Entry_point**: USD0PPSubVaultUpgradeable proxy contract
- **Exploit_vector**: Attacker gained access to the deployer's private key, upgraded the proxy to a malicious implementation, and drained funds.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker obtained the deployer’s private key.
    - Upgraded the proxy contract (USD0PPSubVaultUpgradeable) to a malicious version.
    - Called malicious functions to withdraw all funds (USD0++ tokens).
    - Swapped USD0++ to DAI and then to ETH via Uniswap.
- **Impact**: ~$8.4M drained from the vault (USD0++ tokens converted to ETH).
- **Exploitability**: High
- **Root_cause**: Centralized single-key admin control without multisig or time-lock protections enabled upgrade abuse.
- **Resource**:[Link](https://www.halborn.com/blog/post/explained-the-zoth-hack-march-2025)

