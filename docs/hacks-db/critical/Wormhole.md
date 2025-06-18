# Wormhole Bridge Hack 

- **Project**: Wormhole
- **Exploit_type**: Signature Verification Bypass (Cross-Chain Bridge Logic Flaw)
- **Loss**: ~$325 million (120,000 wETH)
- **Entry_point**: verify_signatures() in Wormhole’s Solana smart contract
- **Exploit_vector**: The attacker exploited a bug in the Wormhole bridge's Solana-side implementation that failed to properly verify guardian signatures before minting wrapped ETH on Solana.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker reverse-engineered the Solana-side Wormhole bridge logic.
    - They discovered a missing signature verification check in the `verify_signatures()` function.
    - Crafted a fake message claiming 120,000 ETH was deposited on Ethereum.
    - Submitted the forged message to the Solana contract.
    - Wormhole bridge minted 120,000 wETH on Solana without real ETH backing.
    - Attacker bridged assets out and began laundering the funds.
- **Impact**: 120,000 wETH minted on Solana without backing, resulting in ~$325M loss
- **Exploitability**: High
- **Root_cause**: The Solana smart contract failed to properly enforce signature checks from the guardian network, allowing forged cross-chain messages to be accepted as valid.
- **Resource**:[Link](https://www.theverge.com/2022/2/3/22916111/wormhole-hack-github-error-325-million-theft-ethereum-solana)