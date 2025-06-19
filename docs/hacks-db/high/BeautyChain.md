# BeautyChain Token Bug

- **Project**: BeautyChain (BEC)
- **Exploit_type**: Integer Overflow in batchTransfer()
- **Loss**: US $900,000
- **Entry_point**: batchTransfer(address[] _receivers, uint256 _value) in the token contract 
- **Exploit_vector**: Malicious call with _value ≈ 2²⁵⁵ and two receivers. Multiplying _value * cnt overflowed to zero, bypassing balance checks, allowing arbitrary token generation 
- **Severity**: High 
- **Attack_steps**:
    - Crafted batchTransfer([addr1, addr2], hugeValue) with hugeValue = 2²⁵⁵.
    - Computation cnt * value overflowed to zero.
    - require(balances[msg.sender] >= 0) passed.
    - Looping transfer added hugeValue tokens to each recipient’s balance.
    - Inflated supply causing dumping/manipulation and market freeze 
- **Impact**: Unlimited mint of BEC tokens, dramatic dilution, >90% price collapse, trading suspended on major exchanges 
- **Exploitability**: High 
- **Root_cause**: Single unguarded multiplication (amount = cnt * _value) bypassing SafeMath; insecure custom logic outside audited standards 
- **Resource**:[Link](https://medium.com/secbit-media/a-disastrous-vulnerability-found-in-smart-contracts-of-beautychain-bec-dbf24ddbc30e)