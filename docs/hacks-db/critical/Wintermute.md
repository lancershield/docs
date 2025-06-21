# Wintermute hack

- **Project**: Wintermute
- **Exploit_type**: Private Key Compromise (Profanity vanity-address vulnerability)
- **Loss**: ~$160 million
- **Entry_point**: compromised hot-wallet private key and vault permissions
- **Exploit_vector**: Attacker brute-forced vanity-address private key generated via Profanity, then used admin privileges to drain assets from Wintermute’s DeFi vault 
- **Severity**: Critical
- **Attack_steps**:
    - Vanity address generated via insecure Profanity tool
    - Attacker brute-forced private key
    - Used compromised key to call vault admin functions
    - Drained ~$160M in tokens to attacker-controlled address
- **Impact**: $160 million lost across stablecoins (e.g. USDC, USDT, DAI), ETH, WBTC, others ($120 M stablecoins, ~$20 M ETH/WBTC, ~$20 M altcoins) 
- **Exploitability**: High
- **Root_cause**: Use of vulnerable vanity-address generator (Profanity) resulting in weak private keys and insufficient revocation of admin rights on compromised address 
- **Resource**:[Link](https://halborn.com/blog/post/explained-the-wintermute-hack-september-2022)