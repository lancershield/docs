# Contracts That Lock Ether Permanently 

```YAML
id: LS15M
title: Contracts That Lock Ether Permanently
baseSeverity: M
category: funds-locking
language: solidity
blockchain: [ethereum]
impact: Irretrievable ETH, protocol unusability, or capital deadlock
status: draft
complexity: low
attack_vector: internal
mitigation_difficulty: easy
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-703
swc: SWC-132
```

## 📝 Description

- Smart contracts that receive Ether but do not implement withdrawal mechanisms effectively trap ETH permanently. This occurs when:
- receive() or fallback() functions exist, allowing ETH deposits
- There is no function to transfer, call, or otherwise send ETH out
- No self-destruct or upgrade path exists to recover funds
- This is often an unintentional design flaw rather than a security vulnerability. However, in DeFi protocols or NFT marketplaces, this can cause protocol degradation, locked user funds, or operational dead-ends.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract EtherSink {
    receive() external payable {} // ✅ Accepts ETH
    // ❌ No withdraw(), no send, no fallback with logic
}
```

## 🧪 Exploit Scenario

1. A user mistakenly sends ETH to a contract that has a receive() function but no withdrawal logic.
2. ETH is accepted and stored in the contract’s balance.
3. The contract has no way to send ETH back out—no owner, no selfdestruct, no withdrawal function.
4. The ETH remains locked indefinitely and cannot be recovered.

**Assumptions:**

- Contract accepts Ether through receive() or fallback().
- No call, transfer, or selfdestruct logic is implemented.
- Deployer or admin cannot access funds.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract RecoverableEther {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    receive() external payable {}

    function withdraw() external {
        require(msg.sender == owner, "Not authorized");
        payable(owner).transfer(address(this).balance);
    }
}
```

## 🧭 Contextual Severity

```yaml

- context: "Default"
  severity: M
  reasoning: "Permanent fund loss can harm users, even if accidental."
- context: "Protocols where users interact by sending Ether"
  severity: H
  reasoning: "Many users may unknowingly send ETH and lose it."
- context: "Internal utility contract with no public interface"
  severity: L
  reasoning: "Low risk as no external Ether expected."
```

## 🛡️ Prevention

### Primary Defenses

- Never include a receive() or fallback() function unless you intend to manage Ether.
- If ETH is accepted, implement a secure withdrawal mechanism or controlled router.

### Additional Safeguards

- Use revert() in receive() if Ether should not be accepted.
- Monitor contract balances during testing and include refund paths.

### Detection Methods

- Identify contracts with receive() or fallback() and no ETH-out functions.
- Check for lack of call, .transfer(), or .send() usage.
- Tools: Slither (ether-lock), MythX, manual bytecode inspection

## 🕰️ Historical Exploits

- **Name:** King of the Ether Throne 
- **Date:** 2016 
- **Loss:** ~10,000 ETH 
- **Post-mortem:** [Link to post-mortem](https://ethereum.stackexchange.com/questions/19341/what-happened-with-the-king-of-the-ether-throne-contract) 
  
## 📚 Further Reading

- [SWC-132: Unexpected Ether Balance](https://swcregistry.io/docs/SWC-132/) 
- [Solidity Docs – receive() and fallback()](https://docs.soliditylang.org/en/latest/contracts.html#receive-ether-function)
- [Slither Detector – Locked Ether](https://github.com/crytic/slither/wiki/Detector-Documentation#contracts-that-lock-ether) 
  
---

## ✅ Vulnerability Report

```markdown
id: LS15M
title: Contracts That Lock Ether Permanently Due to Missing Withdrawal Logic
severity: M
score:
impact: 3        
exploitability: 2 
reachability: 3   
complexity: 1     
detectability: 4  
finalScore: 2.75
```

---

## 📄 Justifications & Analysis

- **Impact**: While not exploitable directly, ETH becomes unrecoverable, affecting users or the project.
- **Exploitability**: Not maliciously triggered, but user error can send ETH to a dead-end.
- **Reachability**: Any contract with a receive() or fallback() is exposed if not managed.
- **Complexity**: Mistake often stems from minimal contract design or negligence.
- **Detectability**: Missed unless explicitly auditing ETH inflow/outflow paths.

