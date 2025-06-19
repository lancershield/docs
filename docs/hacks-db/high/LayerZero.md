# LayerZero Proxy Refund Bug 

- **Project**: LayerZero
- **Exploit_type**: Proxy Contract Misconfiguration (Missing receive() handler for native tokens)
- **Loss**: Indeterminate—risk of locked refunds and failed cross-chain fee reimbursements
- **Entry_point**: Proxy contracts forwarding LayerZero refunds to implementation contracts
- **Exploit_vector**: Users received excess native token refunds that were reverted by proxy contracts lacking receive() — refund and retry operations consistently failed
- **Severity**: High (risk of user funds being permanently locked)
- **Attack_steps**:
    - LayerZero refunded excess fees to the proxy contract via native token transfer.
    - Proxy delegatecalls the refund to its implementation contract.
    - The implementation lacks a receive() function, so ERC-20 native refunds cannot be accepted.
    - The transfer reverts, failing the refund delivery transaction.
    - Refunds become trapped in proxy, potentially unrecoverable and user funds locked indefinitely.
- **Impact**: Potential loss of user refunds, degraded UX, and funds stuck in contract due to failed refund mechanism
- **Exploitability**: Medium (not a direct financial theft but causes locked funds and protocol trust damage)
- **Root_cause**: Implementation contracts lacked a receive() payable fallback; proxy pattern forwarded native transfers via delegatecall, causing reverts when implementation couldn’t handle Ether
- **Resource**:[Link](https://coinsbench.com/understanding-the-vulnerability-in-cross-chain-refunds-a-deep-dive-into-layerzero-and-proxy-0d4f8be458b1?gi=63bd058f442b1)

