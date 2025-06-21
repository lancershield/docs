# Vulcan Forged hack

- **Project**: Vulcan Forged
- **Exploit_type**: Server-side private key compromise (not smart contract bug)
- **Loss**: ~$140 million (4.5 million PYR tokens)
- **Entry_point**: Semi-custodial wallet system (MyForge, managed by Venly)
- **Exploit_vector**: Attackers gained access to backend infrastructure, extracted private keys, and drained user wallets
- **Severity**: Critical
- **Attack_steps**:
    - Attackers compromised Vulcan Forged's backend systems
    - Accessed wallet credentials and private keys from ~96 users
    - Transferred ~4.5M PYR tokens from compromised wallets
    - Used platform's treasury to refund affected users post-exploit
- **Impact**: 4.5M PYR (~9% of token supply) stolen; ~100 wallets drained
- **Exploitability**: High
- **Root_cause**: Insecure server-side wallet management and lack of proper key custody safeguards
- **Resource**:[Link](https://decrypt.co/88177/hacker-steals-140-million-polygon-gaming-platform-nft-marketplace-vulcan-forged)