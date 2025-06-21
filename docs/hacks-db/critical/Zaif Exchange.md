# Zaif Exchange hack

- **Project**: Zaif Exchange
- **Exploit_type**: Hot Wallet Breach (Infrastructure Compromise)
- **Loss**: ~$60 million
- **Entry_point**: Hot wallet private key/API system
- **Exploit_vector**: Attacker accessed the hot wallet via compromised credentials and drained assets
- **Severity**: Critical
- **Attack_steps**:
    - Attacker gained unauthorized access to Zaif’s hot wallet
    - Initiated large withdrawals including 5,966 BTC, BCH, and MonaCoin
    - Transferred funds to attacker-controlled wallets
    - Exfiltrated assets without triggering alerts
- **Impact**: ¥7 billion ($60M) in BTC, BCH, and MONA stolen
- **Exploitability**: High
- **Root_cause**: Poor hot wallet security and lack of real-time monitoring or anomaly detection
- **Resource**:[Link](https://www.coindesk.com/markets/2018/09/20/crypto-exchange-zaif-hacked-in-60-million-bitcoin-theft)