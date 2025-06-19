# zkSync Era Circuit Bug Exploit 

- **Project**: zkSync (Era)
- **Exploit_type**: ZK‑Circuit Soundness Flaw 
- **Loss**: ~$100 million 
- **Entry_point**: ZK‑Proof generation logic in zkSync Era circuits (verify_withdrawal() in prover)
- **Exploit_vector**: A soundness bug in the zk‑SNARK verification circuit allowed forging a proof authorizing token withdrawals without actual layer‑1 backing.
- **Severity**: Critical
- **Attack_steps**:
    - ChainLight researchers developed a proof-of-concept exploit targeting the Era zk-circuit soundness flaw. 
    - Used a forked environment to test a malicious block with forged zk-SNARK proof claiming a 100k ETH withdrawal.
    - Submitted the fake block to the zkSync sequencer, which accepted the withdrawal in simulation (pre-patch version).
    - Proof validated in absence of actual L1 deposit state, bypassing core security logic.
    - If executed in production, would drain tokens without collateral on Ethereum.
- **Impact**: Hypothetically >100k ETH stolen; actual incident prevented at PoC level.
- **Exploitability**: High – crafted zk proof passes soundness check ideally in one block.
- **Root_cause**: Cryptographic circuit incorrectly implemented assumptions, allowing arbitrary balance claims without state inclusion proofs in zk verification.
- **Resource**:[Link](https://github.com/chainlight-io/zksync-era-write-query-poc )

