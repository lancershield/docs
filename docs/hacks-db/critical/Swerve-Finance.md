# Swerve Finance hack

- **Project**: Swerve Finance
- **Exploit_type**: Governance Exploit
- **Loss**: ~ $1.3 million (DAI/USDC/USDT pool assets)
- **Entry_point**: On‑chain governance voting mechanism
- **Exploit_vector**: Acquired majority governance token power to pass malicious proposal
- **Severity**: Critical
- **Attack_steps**:
    - Attacker (“Exploiter A”) gathered 348 k Swerve governance tokens.
    - Submitted malicious proposal to transfer ~$1.3 M pool funds.
    - Fell short of quorum.
    - Added “Exploiter B” with extra 102 k tokens to boost voting power.
    - Continued voting to attempt proposal passage.
- **Impact**: ~ $1.3 M at risk of being transferred to attacker-controlled contract
- **Exploitability**: High
- **Root_cause**: Centralized token governance power remained transferable—gave potential attacker quorum control
- **Resource**:[Link](https://www.theblock.co/post/222744/defunct-swerve-finance-still-subject-of-1-3-million-live-governance-hack)