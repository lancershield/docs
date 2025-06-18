# BNB Chain Exploit 

- **Project**: BNB Chain 
- **Exploit_type**: Cross-Chain Bridge Logic Flaw
- **Loss**: ~$570 million
- **Entry_point**: verifyProof() in the BSC Token Hub bridge contract
- **Exploit_vector**: The attacker forged a message that passed the verifyProof() validation, allowing them to mint 2 million BNB without depositing collateral on the source chain.
- **Severity**: Critical
- **Attack_steps**:
    - Attacker identified a flaw in the BSC Token Hub bridge's IAVL proof verification logic.
    - Crafted a forged cross-chain message that appeared valid to the bridge smart contract.
    - Submitted the fake message to the `verifyProof()` function.
    - Smart contract accepted the proof and minted 2 million BNB on BNB Chain.
    - Attacker moved ~$100 million worth of BNB to other chains before BNB Chain was halted.
    - Validators halted the chain temporarily to prevent further damage and initiated a hard fork.
- **Impact**: 2 million BNB (~$570M) minted out of thin air; ~$100M successfully exfiltrated before halt.
- **Exploitability**: High
- **Root_cause**: Improper validation of IAVL Merkle proof logic in the Token Hub bridge contract, allowing fake messages to be treated as legitimate without verifying source chain state.
- **Resource**: [Link](https://www.merklescience.com/blog/hack-track-analysis-of-the-bnb-smart-chain-exploit)

