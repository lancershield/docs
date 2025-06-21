# EXMO Exchange Private Key Compromise Exploit

- **Project**: EXMO Exchange
- **Exploit_type**: Private Key Compromise
- **Loss**: ~$10.5 million
- **Entry_point**: Hot wallet system (multiple assets)
- **Exploit_vector**: Attackers gained access to EXMO's hot wallet private keys and initiated unauthorized withdrawals to their own addresses
- **Severity**: Critical
- **Attack_steps**:
    - Attacker breached EXMO’s infrastructure (suspected phishing or insider vector)
    - Transferred crypto assets (including BTC, ETH, XRP, ZEC, USDT, etc.) from hot wallets to attacker-controlled wallets
    - Funds were laundered through known mixing services like Tornado Cash
- **Impact**: ~$10.5 million in digital assets withdrawn from EXMO hot wallets
- **Exploitability**: High — due to full control of hot wallet private keys
- **Root_cause**: Lack of operational security on hot wallets, insufficient segregation of customer assets
- **Resource**:[Link](https://www.bleepingcomputer.com/news/security/exmo-cryptocurrency-exchange-hacked-loses-5-percent-of-total-assets/)