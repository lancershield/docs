# Defrost Finance Admin Key Exploit 

- **Project**: Defrost Finance
- **Exploit_type**: Admin Key Compromise + Fake Collateral Oracle Injection
- **Loss**: ~$12 million
- **Entry_point**: admin function updating price oracle and collateral configuration (v1 contract)
- **Exploit_vector**: The attacker gained control over the admin private key, added an unauthorized collateral token and malicious oracle, enabling mass liquidations to drain protocol funds
- **Severity**: Critical
- **Attack_steps**:
    - Initial flash loan exploit targeted v2, draining ~$173k from LSWUSDC pool 
    - Attacker or insider accessed the admin key for v1 contract.
    - Added a fake collateral token and linked it to a malicious price oracle. 
    - Triggered mass liquidations of user collateral (including H2O token minting and liquidation loop). 
    - Stole ~$12 million from protocol treasury and users.
    - Platform later offered bounty and recovered some funds. 
- **Impact**: ~$12 million drained; user vaults evacuated; TVL collapsed.
- **Exploitability**: High – full admin access granted unilateral control.
- **Root_cause**: Compromise or misuse of protocol admin private key, combined with insufficient privilege control and lack of timelock/proposal governance on admin functions.
- **Resource**:[Link](https://www.halborn.com/blog/post/explained-the-defrost-hack-december-2022)