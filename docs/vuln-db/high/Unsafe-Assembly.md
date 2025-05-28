# Unsafe Assembly Usage

```YAML
id: TBA
title: Unsafe Assembly Usage 
baseSeverity: H
category: low-level
language: solidity
blockchain: [ethereum]
impact: Memory corruption, access control bypass, or gas griefing
status: draft
complexity: high
attack_vector: internal
mitigation_difficulty: hard
versions: [">=0.4.0", "<=0.8.25"]
cwe: CWE-242
swc: SWC-128
```

## 📝 Description

- Solidity allows low-level operations using the assembly block, enabling direct access to EVM instructions. 
- While powerful, improper use of inline assembly introduces serious risks such as:
- Memory corruption
- Type misinterpretation
- Bypassing overflow checks
- Circumventing access control
- Reentrancy vulnerabilities
- Unintended code execution paths
- Assembly instructions are not subject to Solidity’s safety mechanisms like bounds checking, type enforcement, or access protection. This makes them error-prone and dangerous if not carefully sandboxed or verified.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

contract UnsafeAssembly {
    function overwriteStorage() external {
        assembly {
            sstore(0, caller()) // ❌ Overwrites slot 0 with caller address
        }
    }
}
```

## 🧪 Exploit Scenario

1. A contract uses inline assembly to optimize a low-level operation.
2. It directly modifies a storage slot or uses mstore/sstore unsafely.
3. An attacker triggers this function and rewrites a privileged slot (e.g., owner, implementation, or balances[0]).
4. The attacker becomes the new admin or bypasses critical restrictions.

**Assumptions:**

- Assembly is used without bounds or slot protection.
- Callers can reach the function or override inputs.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

contract SafeStorage {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function safeSet(address newOwner) external {
        require(msg.sender == owner, "Not authorized");
        owner = newOwner; // ✅ Uses safe Solidity assignment
    }
}
```

## 🧭 Contextual Severity

```yaml
- context: "Default"
  severity: H
  reasoning: "Unsafe assembly can result in serious but developer-dependent vulnerabilities."
- context: "Highly optimized protocol (gas-focused)"
  severity: C
  reasoning: "Widespread use of assembly increases risk of critical bugs or misuse."
- context: "Simple contracts with minimal inline assembly"
  severity: L
  reasoning: "Minimal use of assembly reduces likelihood of major issues."
```

## 🛡️ Prevention

### Primary Defenses

- Avoid using assembly unless necessary for gas optimization or interoperability.
- Validate all pointer math, storage slot usage, and return sizes manually when using inline assembly.

### Additional Safeguards

- Encapsulate assembly into internal/private functions.
- Use OpenZeppelin’s StorageSlot or SafeCast libraries for managing storage directly.

### Detection Methods

- Search for assembly  blocks or usage of mstore, sstore, delegatecall, calldatacopy.
- Tools: Slither (assembly-usage, low-level-calls), MythX, manual logic review

## 🕰️ Historical Exploits

- **Name:** Parity Wallet `delegatecall` Bug 
- **Date:** 2017 
- **Loss:** ~$280M  
- **Post-mortem:** [Link to post-mortem](https://paritytech.io/blog/security-alert.html) 
  
## 📚 Further Reading

- [SWC-128: DoS via Unexpected Revert](https://swcregistry.io/docs/SWC-128/) 
- [Solidity Docs – Inline Assembly](https://docs.soliditylang.org/en/latest/assembly.html) 
- [OpenZeppelin StorageSlot Library](https://docs.openzeppelin.com/contracts/4.x/api/utils#StorageSlot) 
- [Slither – Assembly Usage Detection](https://github.com/crytic/slither/wiki/Detector-Documentation#assembly-usage) 

---

## ✅ Vulnerability Report

```markdown
id: TBA
title: Unsafe Assembly Usage 
severity: H
score:
impact: 4      
exploitability: 3 
reachability: 3 
complexity: 3   
detectability: 4  
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: Can corrupt contract state, violate access control, or crash logic.
- **Exploitability**: High if attacker controls execution path into unsafe assembly.
- **Reachability**: Found in optimization logic, proxies, and legacy DeFi contracts.
- **Complexity**: Requires low-level knowledge; misuse is easy, debugging is hard.
- **Detectability**: Static tools can flag usage, but correctness depends on manual review.