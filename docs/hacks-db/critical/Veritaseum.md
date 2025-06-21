# Veritaseum Token Contract Exploit

- **Project**: Veritaseum
- **Exploit_type**: Privileged Access Token Drain
- **Loss**: ~$8.4 million
- **Entry_point**: Veritaseum smart contract via compromised wallet access
- **Exploit_vector**: Attacker exploited admin-level access to the contract and transferred tokens out using elevated privileges
- **Severity**: Critical
- **Attack_steps**:
    - Attacker gained access to Veritaseum wallet or contract keys
    - Initiated unauthorized transfers of VERI tokens
    - Sent stolen tokens to exchange wallets
    - Liquidated tokens on exchanges during low-liquidity periods
- **Impact**: 36,000+ VERI tokens (worth ~$8.4M at the time) stolen and partially sold
- **Exploitability**: Medium — required privileged access or compromised keys
- **Root_cause**: Insecure key management and lack of operational security around privileged accounts
- **Resource**:[Link](http://csidb.net/csidb/incidents/7981da1e-c60b-4c5b-ac53-12c9c557f35d/)