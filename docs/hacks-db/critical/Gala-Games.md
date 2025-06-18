# Gala Games Token Mint Bug 

- **Project**: Gala Games
- **Exploit_type**: Private Key Compromise → Excessive Token Minting
- **Loss**: ~$206 million 
- **Entry_point**: mint() function on GALA token contract via privileged minter address
- **Exploit_vector**: An attacker (or compromised admin) used a privileged account to mint 5 billion GALA tokens, then sold part of them before the account was blocklisted.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker gained access to a privileged GALA minter address due to poor access control or key management 
    - Minted 5 billion new GALA tokens (~$200 million), suddenly inflating token supply
    - Sold 600 million tokens ($21 million) via Uniswap in batches of ~100 ETH each 
    - Gala team detected the abnormal mint, activated emergency blocklisting mechanism on the exploiter address within ~45 minutes 
    - The remaining 4.4 billion minted tokens were effectively frozen or burned to prevent further damage 
    - Some exploit proceeds (~$21 million ETH) were returned to Gala Games by the exploiter 
- **Impact**:Market cap temporarily destabilized, GALA price plunged ~20%.Approximately $21 million worth of ETH was liquidated and later partially recovered.
- **Exploitability**: High
- **Root_cause**:Excessive trust in single private key for minter privileges, no multisig or time-lock safeguards.No real-time monitoring or transaction limits on mint operations.
- **Resource**:[Link](https://cryptoslate.com/gala-games-says-it-resolved-exploit-within-45-minutes-identified-culprit/)
