# Exactly Protocol Price Manipulation Exploit 

- **Project**: Exactly Protocol
- **Exploit_type**: Insufficient Input Validation + Permit Bypass
- **Loss**: ~$7.6 million 
- **Entry_point**: leverage() in the DebtManager contract
- **Exploit_vector**: Attacker supplied a malicious market contract address to bypass permit checks, re-entered the DebtManager via a fake market contract, and siphoned user collateral via manipulated crossDeleverage() calls.
- **Severity**: Critical
- **Attack_steps**:
    - The attacker invoked leverage() with a corrupted market address and forged permit parameters.
    - Bypassed the permit check due to improper ordering that prioritized permits over market validation.
    - The protocol set _msgSender to the victim’s address, enabling the attacker to act on behalf of the victim.
    - External call to the malicious market’s deposit() executed, triggering a reentrancy into crossDeleverage().
    - Victim collateral was withdrawn via the attacker-built Uniswap pool and swapped for attacker-controlled tokens.
    - Attacker repeated the sequence across multiple users and tokens (USDC, WETH, wstETH, OP).
    - A secondary actor (copycat) performed similar operations using the same bug.
- **Impact**: ~$7.6 million stolen, affecting 117 victims, including two whale wallets each losing > $1.5 million 
- **Exploitability**: High
- **Root_cause**: DebtManager lacked validation on market input before permit() checks; insufficient authorization logic and flawed reentrancy protection enabled control hijack.
- **Resource**:[Link](https://medium.com/@exactly_protocol/exactly-protocol-incident-post-mortem-b4293d97e3ed) 
