# CoinDash Token Sale Website Attack

- **Project**: CoinDash
- **Exploit_type**: Website Frontend Hijack
- **Loss**: ~$7.5 million
- **Entry_point**: Token sale website (coindash.io)
- **Exploit_vector**: Attacker modified the Ethereum address displayed on the CoinDash ICO page to redirect funds to their own wallet
- **Severity**: Critical
- **Attack_steps**:
    - Gained unauthorized access to CoinDash’s website
    - Replaced the official ETH contribution address with the attacker’s address
    - Investors unknowingly sent ETH to the wrong address during the token sale
    - CoinDash halted the ICO but funds were already drained
- **Impact**: ~$7.5M in ETH sent to attacker address; trust in ICO process damaged
- **Exploitability**: High — due to centralization and poor website infrastructure security
- **Root_cause**: Centralized web infrastructure compromised; no address verification or cryptographic attestation
- **Resource**:[Link](https://www.securityweek.com/hacker-steals-7-million-ethereum-coindash/)

