# dForce Lendf.Me Reentrancy Exploit 

- **Project**: dForce Lendf.Me
- **Exploit_type**: Reentrancy via ERC‑777 callback (read‑only reentrancy)
- **Loss**: ~$25 million 
- **Entry_point**: deposit() for imBTC collateral in the lending contract
- **Exploit_vector**: Attacker used the ERC‑777 token callback to re‑enter deposit logic, inflating collateral balance and borrowing far beyond actual funds
- **Severity**: Critical
- **Attack_steps**:
    - Attacker deposited imBTC as collateral (ERC‑777) via deposit().
    - During the callback, triggered reentrancy to instantly withdraw the original collateral—before internal state was updated.
    - Protocol state updated erroneously to reflect collateral retention and removal, effectively doubling collateral balance.
    - Repeated this flash‑loan loop multiple times to amplify the collateral record.
    - Borrowed maximum across markets based on inflated collateral and drained the vault.
    - Flash loan repaid, attacker pocketed borrowed funds.
- **Impact**: ~$25M summoned from asset pools; platform paused; funds later returned by exploiter.
- **Exploitability**: High — misuse of ERC‑777 callback and no reentrancy guard
- **Root_cause**: Reliance on ERC-777 transfer hooks without accounting for reentrancy; lack of secure state update patterns
- **Resource**:[Link](https://quantstamp.com/blog/how-the-dforce-hacker-used-reentrancy-to-steal-25-million)
  