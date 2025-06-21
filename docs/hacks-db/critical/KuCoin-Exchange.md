# KuCoin Exchange

- **Project**: KuCoin Exchange
- **Exploit_type**: Private Key Compromise
- **Loss**: ~$280 million
- **Entry_point**: Hot wallet infrastructure
- **Exploit_vector**: Attackers obtained private keys of KuCoin's hot wallets, enabling unauthorized withdrawal of various tokens across multiple blockchains
- **Severity**: Critical
- **Attack_steps**:
    - Attackers accessed KuCoin's hot wallet keys
    - Drained assets including BTC, ETH, USDT, and dozens of ERC-20 tokens
    - Swapped or mixed stolen assets to avoid traceability
    - Certain tokens were frozen by their issuers post-attack (e.g., Tether, Ocean, Orion)
- **Impact**: $280 million in tokens lost, though a large portion ($240M) was recovered or frozen
- **Exploitability**: High — direct key access gave complete fund control
- **Root_cause**: Inadequate protection of private keys associated with hot wallets
- **Resource**:[Link](https://www.kucoin.com/)

