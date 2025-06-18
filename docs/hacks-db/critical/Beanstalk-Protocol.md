# Beanstalk Flash Loan Governance Exploit 

- **Project**: Beanstalk Protocol
- **Exploit_type**: Flash Loan + Governance Manipulation
- **Loss**: ~$182 million 
- **Entry_point**: emergencyCommit() via governance proposal (BIP‑18)
- **Exploit_vector**: Attacker used a flash loan to amass voting power, passed a malicious proposal, and triggered immediate fund extraction via emergencyCommit().
- **Severity**: Critical
- **Attack_steps**:
    - Took out a massive flash loan (~$1 billion) from Aave in stablecoins and DAI 
    - Converted into governance tokens (Stalk) and new LP tokens (BEAN3CRV‑f and BEANLUSD‑f) to secure >67% voting power 
    - Submitted BIP‑18 (malicious transfer) and BIP‑19 (Ukraine donation), then used emergencyCommit() to bypass the 7-day waiting period 
    - Executed fund transfer, drained ~$182 M from the silo, repaid the flash loan, yielded ~76 M profit
    - Attacker laundered proceeds via Tornado Cash and donated ~$250K to Ukraine.
- **Impact**: ~$182 million drained from protocol reserves; stablecoin peg broke; governance integrity compromised.
- **Exploitability**: High
- **Root_cause**: Governance design allowed flash loan–based voting manipulation; emergencyCommit() lacked anti–flash loan protections or delays.
- **Resource**:[Link](https://www.merklescience.com/blog/hack-track-analysis-of-beanstalk-flash-loan-attack)