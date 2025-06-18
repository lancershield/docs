# Nomad Bridge Exploit 

- **Project**: Nomad Bridge Exploit
- **Exploit_type**: Faulty Initialization + Signature Verification Bypass
- **Loss**: ~$190 million
- **Entry_point**: process() function in the Nomad Replica contract
- **Exploit_vector**: A misconfigured contract set the trusted root to 0x00, causing the system to accept any arbitrary message as valid without proper verification.
- **Severity**: Critical
- **Attack_steps**:
    - A contract upgrade inadvertently initialized the trusted root in the Replica contract to zero (0x00).
    - This allowed any message with a valid format (not necessarily a valid signature) to pass verification.
    - The first attacker crafted a fake withdrawal message and called `process()` successfully.
    - News spread and hundreds of copycat users began submitting their own fake messages.
    - No signature or ownership checks blocked these unauthorized withdrawals.
    - In total, over $190M was drained from the bridge by 300+ unique addresses.
- **Impact**: ~$190M in ETH, USDC, WBTC, and other assets drained from the Nomad cross-chain bridge.
- **Exploitability**: High
- **Root_cause**: Failure to initialize the trusted root correctly during a smart contract upgrade, bypassing all signature validation for cross-chain messages.
- **Resource**:[Link](https://bravenewcoin.com/insights/nomad-bridge-hacker-arrested-in-israel-over-190m-crypto-exploit)