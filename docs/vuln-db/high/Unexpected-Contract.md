# Unexpected Contract Inheritance Order Leads to Overridden Logic 

```YAML
id: TBA
title: Unexpected Contract Inheritance Order Leads to Overridden Logic 
severity: H
category: inheritance
language: solidity
blockchain: [ethereum]
impact: Overwritten functions, broken modifiers, or misconfigured access control
status: draft
complexity: medium
attack_vector: internal
mitigation_difficulty: medium
versions: [">=0.6.0", "<=0.8.25"]
cwe: CWE-681
swc: SWC-125
```

## 📝 Description

- In Solidity, multiple inheritance is linearized using C3 linearization, which defines the order in which base contracts are resolved when functions or modifiers are overridden. When contracts are inherited in the wrong order, it can result in:
- Incorrect function logic being selected
- Modifiers or access controls being overridden unintentionally
- Storage layout mismatches in upgradeable contracts
- Logic from security-critical parents (like Ownable or Pausable) being ignored
- This can lead to subtle but critical errors that often pass tests and static analysis but cause unexpected runtime behavior or broken access control.

## 🚨 Vulnerable Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/Ownable.sol";

contract Pausable {
    bool public paused;

    modifier whenNotPaused() {
        require(!paused, "Paused");
        _;
    }

    function pause() external {
        paused = true;
    }
}

contract MyContract is Pausable, Ownable {
    function withdraw() external onlyOwner whenNotPaused {
        // transfer funds
    }

    function pause() public override {
        // ❌ This overrides Ownable's pause logic if any
        paused = true;
    }
}
```

## 🧪 Exploit Scenario

1. A contract inherits Pausable and Ownable in the wrong order.
2. A function like pause() is overridden in Pausable without respecting onlyOwner.
3. Anyone can now call pause() and disable the contract.
4. Alternatively, the wrong version of withdraw() is executed, ignoring modifiers like whenNotPaused.

**Assumptions:**

- Multiple base contracts define the same function name or modifier.
- Developer assumes base contracts are resolved in a certain order, but C3 linearization overrides expectations.

## ✅ Fixed Code

```solidity

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/Pausable.sol";

contract MyContract is Ownable, Pausable {
    constructor() {
        _pause(); // optional: start paused
    }

    function withdraw() external onlyOwner whenNotPaused {
        // safe withdrawal logic
    }

    function pause() public onlyOwner override {
        _pause(); // ✅ Uses Pausable's implementation with owner check
    }

    function unpause() public onlyOwner override {
        _unpause();
    }
}
```

## 🛡️ Prevention

### Primary Defenses

- Always understand and control the linearization order of multiple inheritance.
- Use explicit override declarations for clarity and compiler enforcement.
- Avoid defining functions with the same name across base contracts unless intentional.

### Additional Safeguards

- Audit all overridden functions and modifiers.
- Use interfaces or abstract base contracts for clarity when composing logic.

### Detection Methods

- Review is A, B, C inheritance chains for possible linearization mismatches.
- Run Slither’s shadowing-state and multiple-inheritance checks.
- Use compiler warnings for override declarations (solc --via-ir).

## 🕰️ Historical Exploits

- **Name:** MDTCrowdsale Incorrect Inheritance 
- **Date:** 2018 
- **Loss:** ~$500,000 in misallocated bonus tokens due to incorrect inheritance structure 
- **Post-mortem:** [Link to post-mortem](https://www.vidma.io/blog/unraveling-the-perils-of-incorrect-inheritance-order-in-smart-contracts) 
- **Name:** ownersCanKill Role Misconfiguration 
- **Date:** 2021 
- **Loss:** ~$1.3 million lost after unauthorized contract self-destruction due to faulty override logic 
- **Post-mortem:** [Link to post-mortem](https://blog.solidityscan.com/incorrect-inheritance-order-in-smart-contracts-ddcc75ed472c)
 
## 📚 Further Reading

- [SWC-125: Incorrect Inheritance Order](https://swcregistry.io/docs/SWC-125/) 
- [Solidity Docs – Inheritance and Linearization](https://docs.soliditylang.org/en/latest/contracts.html#inheritance)
- [OpenZeppelin Contracts – Best Practices](https://docs.openzeppelin.com/contracts/4.x/) 
- [Slither Detector: Shadowing State Variables](https://github.com/crytic/slither/wiki/Detector-Documentation#shadowing-state-variables)

--- 
  
## ✅ Vulnerability Report
```markdown
id: TBA
title: Unexpected Contract Inheritance Order Leads to Overridden Logic 
severity: H 
score:
impact: 4 
exploitability: 3 
reachability: 4  
complexity: 3 
detectability: 3 
finalScore: 3.55
```

---

## 📄 Justifications & Analysis

- **Impact**: If access control or core logic is bypassed, attackers can pause, mint, or drain funds.
- **Exploitability**: Exploitable if attacker can trigger a mis-prioritized function.
- **Reachability**: Found in most complex contracts using OpenZeppelin or multi-inheritance.
- **Complexity**: Understanding C3 linearization is non-trivial.
- **Detectability**: Not caught by most static tools unless explicitly configured.