# Golem Network Token Approve Front-Running Exploit

- **Project**: Golem Network Token (GNT)
- **Exploit_type**: Front-running attack on approve() function
- **Loss**: ~$400,000 (est. over multiple attacks)
- **Entry_point**: approve() function in the GNT ERC20 contract
- **Exploit_vector**: Attacker monitored the mempool, front-ran users increasing allowances to malicious contracts before updating their approvals
- **Severity**: Medium
- **Attack_steps**:
    - Users approved a new allowance to a third-party (DApp, exchange, etc.)
    - Attacker saw the approve() transaction in the mempool
    - Quickly submitted a transferFrom() using previously approved allowance before the new approval was confirmed
    - Drained the user’s tokens before the new approval could overwrite it
- **Impact**: Multiple users lost their token balances due to poor allowance management pattern
- **Exploitability**: Medium — relied on user behavior and timing
- **Root_cause**: ERC20 standard allowed changing approvals without resetting to zero first; no protection against front-running
- **Resource**:[Link](https://www.golem.network/)