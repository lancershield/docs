# Orbit Bridge Exploit 

- **Project**: Orbit Bridge (Orbit Chain cross‑chain bridge)
- **Exploit_type**: Private Key Compromise of Vault Multisig
- **Loss**: ~$82 million 
- **Entry_point**: Bridge vaults governed by a multisig wallet that authorized withdrawals
- **Exploit_vector**: Attackers gained access to one or more private keys tied to the vault multisig, enabling unauthorized cross-chain withdrawals
- **Severity**: Critical
- **Attack_steps**:
    - Compromised private key(s) of the multisig controlling bridge vaults (likely via insider or social-engineering). 
    - Created and signed multiple withdrawal transactions across Ethereum vaults.
    - Extracted assets in 6–14 batches between Dec 31, 2023 and Jan 1, 2024—draining ETH, USDT, USDC, DAI, WBTC. 
    - Converted stolen funds off-chain, with ~$48M later moved to Tornado Cash. 
    - Orbit Bridge paused vault operations and engaged law enforcement and security firms. 
- **Impact**: $82M stolen; TVL dropped from ~$152M to ~$71M; cross-chain services halted; stolen funds remain in mixers. 
- **Exploitability**: High — private key/multisig compromise allows full vault control
- **Root_cause**: Centralized multisig key management lacking isolation (e.g. keys stored on same system, no secure key rotation/backups); insufficient operational separation and detection systems
- **Resource**:[LInk](https://www.halborn.com/blog/post/explained-the-orbit-bridge-hack-december-2023)