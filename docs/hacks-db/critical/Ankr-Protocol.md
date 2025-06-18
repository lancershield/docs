# Ankr Protocol Exploit (2022)

- **Project**: Ankr Protocol
- **Exploit_type**: Private Key Compromise + Unlimited Token Minting
- **Loss: ~$5 million**; $15M+ impact due to liquidity disruption and token dumps
- **Entry_point**: mint() function of the aBNBc token contract
- **Exploit_vector**: The attacker gained access to the deployer’s private key and used it to call the mint() function, creating 6 quadrillion aBNBc tokens without authorization.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker compromised the private key of the deployer wallet for the aBNBc contract.
    - Used the `mint()` function to create 6 quadrillion aBNBc tokens.
    - Swapped a portion of the fake aBNBc for other tokens (USDC, BNB, etc.) via DEXes.
    - Price of aBNBc collapsed due to the massive oversupply.
    - Several liquidity providers and LPs were left holding worthless aBNBc.
    - Ankr paused trading, burned excess supply, and launched compensation plans.
- **Impact**: Price collapse of aBNBc, ~$5M in stolen assets, and widespread market disruption.
- **Exploitability**: High
- **Root_cause**: Critical failure in key management—deployer’s private key was exposed, enabling direct access to token minting functionality without any safeguard (e.g., multisig or timelock).
- **Resource**:[Link](https://www.merklescience.com/blog/hack-track-analysis-of-ankr-exploit)