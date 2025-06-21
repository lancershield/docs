# Wemix hack

- **Project**: Wemix (Play Bridge Vault)
- **Exploit_type**: Private Key/Authentication Key Compromise
- **Loss**: ~$6.1 million 
- **Entry_point**: Play Bridge Vault (cross-chain withdrawal system) 
- **Exploit_vector**: Attackers stole authentication key for NFT monitoring system (“NILE”), then executed unauthorized withdrawals from bridge. 
- **Severity**: Critical
- **Attack_steps**:
    - Steal auth key from shared developer repository
    - Plan over ~2 months
    - Execute ~15 withdrawal attempts via Play Bridge Vault
    - Successfully withdraw ~8.65 M WEMIX tokens
- **Impact**: Drained ~6.1 M USD worth of tokens; token price dropped ~40%; operations suspended temporarily. 
- **Exploitability**: High — key compromise allowed direct unauthorized bridge access
- **Root_cause**: Poor off-chain key management; auth key publicly accessible in repo, no multi-sig or vault protection 
- **Resource**:[Link](https://www.halborn.com/blog/post/explained-the-wemix-hack-march-2025)