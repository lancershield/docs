# Yam Finance Rebase Bug 

- **Project**: Yam Finance
- **Exploit_type**: Overflow Bug in Governance-Relevant Logic
- **Loss**: ~$750,000 (in YAM-curve LP tokens)
- **Entry_point**: rebase() function affecting governance calculations
- **Exploit_vector**: A miscalculation during the rebase process caused minted YAM tokens to overflow and be permanently locked in the governance contract, breaking governance and making it impossible to pass proposals (like pausing or fixing the protocol).
- **Severity**: Critical
- **Attack_steps**:
    - Yam Finance launched with unaudited contracts and a rebase mechanism.
    - A bug in the `rebase()` function incorrectly calculated excess YAM supply.
    - This caused a large number of YAM tokens to be sent to the governance contract.
    - Due to integer overflow or lack of validation, these tokens became unusable.
    - As a result, governance proposals could no longer reach quorum.
    - prevented fixes to the protocol, locking it into a broken state.
- **Impact**: ~$750K worth of YAM-curve LP tokens lost and protocol governance permanently disabled in v1.
- **Exploitability**: Medium (not maliciously exploited, but had critical downstream effects)
- **Root_cause**: Flawed rebase logic and lack of bounds checking caused unintended minting of governance-breaking tokens—amplified by lack of audits and time to patch.
- **Resource**:[Link](https://www.theblock.co/post/156796/yam-finance-thwarts-governance-attack-aimed-at-hijacking-its-treasury)