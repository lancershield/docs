# StableMagnet Exit Scam 

- **Project**: StableMagnet
- **Exploit_type**: Hardcoded Backdoor / Rug Pull
- **Loss**: ~$27 million
- **Entry_point**: Liquidity removal function in malicious contract controlled by developers
- **Exploit_vector**: Smart contracts included a hidden function allowing developers to drain user funds
- **Severity**: Critical
- **Attack_steps**:
    - StableMagnet launched as a new “stablecoin yield farm” platform on Binance Smart Chain and Ethereum.
    - Users deposited ~$27 million worth of stablecoins (USDT, USDC, BUSD, etc.) into the protocol.
    - The developers deployed contracts containing an obfuscated backdoor in the liquidity logic.
    - Once TVL peaked, the devs triggered the hidden function to transfer all stablecoins to their wallets.
    - The contracts were drained in a single transaction and user funds were permanently lost.
    - Website and social channels went offline shortly after the theft.
- **Impact**: ~$27M in stablecoins stolen; total loss for users
- **Exploitability**: Low (external attackers couldn’t exploit it; only insiders could)
- **Root_cause**: Obfuscated malicious code deployed by protocol developers with no audits or community scrutiny
- **Resource**:[Link](https://www.certik.com/resources/blog/60ejqmsLzP8cVRXFTqkHqK-stablemagnet-rugpull)

