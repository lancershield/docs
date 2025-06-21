# Cryptopia Exchange 

- **Project**: Cryptopia Exchange
- **Exploit_type**: Private Key Compromise
- **Loss**: ~$16 million
- **Entry_point**: User wallet access via compromised private keys
- **Exploit_vector**: Attackers accessed wallet private keys and initiated transfers across multiple Ethereum-based assets
- **Severity**: Critical
- **Attack_steps**:
    - Attacker breached Cryptopia’s internal systems
    - Extracted private keys for hot and warm wallets
    - Systematically drained wallets in two major waves (January 14 & 16, 2019)
    - Continued siphoning funds for days even after discovery
- **Impact**: ~$16 million in Ethereum and ERC-20 tokens stolen
- **Exploitability**: High — once private keys were compromised, no on-chain defense possible
- **Root_cause**: Poor key management and delayed incident response after first signs of compromise
- **Resource**:[Link](https://cryptodamus.io/en/articles/news/cryptopia-s-downfall-what-every-crypto-investor-needs-to-know)

