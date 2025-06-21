# FTX hack

- **Project**: FTX
- **Exploit_type**: SIM-swap / Private Key Access
- **Loss**: ~$400–600 million
- **Entry_point**: Hot wallets accessed post-bankruptcy filing
- **Exploit_vector**: Attackers used SIM-swap to gain access to multi-factor authentication (MFA), retrieved private keys, and transferred funds from wallets
- **Severity**: Critical
- **Attack_steps**:
    - Attackers performed SIM-swap on FTX executives
    - Used intercepted MFA codes to access backend systems
    - Retrieved private keys to FTX’s hot wallets
    - Initiated unauthorized withdrawals post-bankruptcy announcement
- **Impact**: ~$400–600 million in assets drained, immediate loss of user funds, erosion of trust during an already critical time
- **Exploitability**: High
- **Root_cause**: Weak operational security — reliance on SMS-based MFA, lack of proper key isolation, and absence of post-bankruptcy wallet freezing
- **Resource**:[Link](https://www.elliptic.co/blog/the-477-million-ftx-hack-following-the-blockchain-trail)