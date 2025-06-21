# SpankChain Reentrancy Exploit

- **Project**: SpankChain
- **Exploit_type**: Reentrancy
- **Loss**: ~$38,000
- **Entry_point**: withdraw() function in the payment channel smart contract
- **Exploit_vector**: Attacker triggered a reentrant call during an ETH.withdraw() before internal balance update
- **Severity**: High
- **Attack_steps**:
    - Discovered vulnerable withdraw() function that sent ETH before updating user balance
    - Created a malicious contract with a fallback function to call withdraw() recursively
    - Re-entered repeatedly, draining funds from the payment channel
- **Impact**: 165 ETH (~$38K at the time) stolen from both users and the contract’s own treasury
- **Exploitability**: High — no reentrancy guard or pattern protection
- **Root_cause**: Violation of Checks–Effects–Interactions pattern; failure to secure external ETH transfers
- **Resource**:[Link](https://medium.com/swlh/how-spankchain-got-hacked-af65b933393c)