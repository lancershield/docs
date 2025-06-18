# CryptoNova Exploit 

- **Project**: CryptoNova
- **Exploit_type**: Private Key Compromise (Social Engineering or Credential Leak)
- **Loss**: ~$3 million (NFTs and ETH)
- **Entry_point**: Project’s deployer wallet and NFT contract permissions
- **Exploit_vector**: An attacker gained access to the project’s admin wallet and used it to transfer high-value NFTs and ETH from the CryptoNova treasury and user-owned assets.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker gained access to CryptoNova's deployer/admin wallet (likely via leaked private key or phishing).
    - Used admin permissions to transfer rare NFTs from the official CryptoNova contract to their own wallet.
    - Drained ETH from the project’s treasury wallet.
    - Also targeted some community wallets connected to project interactions.
    - Sold stolen NFTs and laundered ETH through mixers and anonymous wallets.
    - The CryptoNova team confirmed the compromise and had no way to reverse the damage.
- **Impact**: ~$3M in NFTs and ETH stolen, including rare pieces and treasury funds; significant damage to community trust.
- **Exploitability**: High
- **Root_cause**: Single point of failure due to compromised private key for a privileged wallet controlling both NFTs and funds — no multisig or backup recovery strategy in place.
- **Resource**:[Link](https://cryptonews.com/news/nobitex-loses-73m-in-tron-exploit-is-irans-top-crypto-exchange-under-threat/)